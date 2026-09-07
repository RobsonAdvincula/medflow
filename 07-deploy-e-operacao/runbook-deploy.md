# Runbook de Deploy — MedFlow

**Data:** 2026-03-27

---

## Pré-requisitos

- Instância n8n activa (Railway, self-hosted, ou n8n Cloud)
- Instância Evolution API com número WhatsApp configurado e aquecido
- API da clínica disponível com endpoint de atendimentos
- Acesso SSH ao servidor (se self-hosted)

---

## Deploy Inicial

### 1. Configurar variáveis de ambiente no n8n

No painel n8n → **Settings → Variables**, criar:

| Variável | Valor |
|----------|-------|
| `WEBHOOK_SECRET` | `openssl rand -hex 32` |
| `MEDFLOW_API_URL` | URL base da API da clínica |

No painel n8n → **Credentials**, criar:
- **HTTP Header Auth** com nome `Evolution API Key` → header `apikey`, valor = API key da instância
- **Bearer Token Auth** com nome `API Clínica` → token fornecido pela clínica

### 2. Importar workflows

```
n8n UI → Workflows → Import from File
```

Importar por ordem:
1. `WF-001-evolution-confirmacao.json`
2. `WF-002-evolution-status.json`
3. `WF-003-evolution-lembrete.json`

Após importar cada um, abrir e substituir as credentials pelos nomes criados no passo 1.

### 3. Activar workflows

Activar pela ordem: WF-002 → WF-003 → WF-001 (WF-001 expõe o webhook; activar por último após os outros estarem prontos)

### 4. Configurar webhook Evolution API (para WF-002)

No painel Evolution API → **Webhooks**, configurar:
- URL: `https://n8n.{dominio}/webhook/medflow-status`
- Eventos: `messages.upsert`

### 5. Servir o dashboard

```bash
# Opção A — nginx (produção)
cp clinica-atendimento.html /var/www/medflow/index.html
cp n8n-integracoes.html /var/www/medflow/integracoes.html
# Adicionar Basic Auth no bloco location se exposição externa

# Opção B — servidor local simples (teste)
python3 -m http.server 8080
```

### 6. Validar

```bash
# Testar webhook
curl -X POST https://n8n.{dominio}/webhook/medflow-confirmar \
  -H "X-MedFlow-Secret: {WEBHOOK_SECRET}" \
  -H "Content-Type: application/json" \
  -d '{"paciente":"Teste","telefone":"351900000000","data":"2026-04-01","horario":"10:00","medico":"Dr. Teste","especialidade":"Geral","atendimento_id":"TEST-001"}'

# Verificar no n8n Executions que WF-001 correu sem erros
```

---

## Actualizar um Workflow

1. **Desactivar** o workflow no n8n antes de editar
2. Aplicar a alteração
3. Testar em staging com dados sintéticos
4. **Reactivar** o workflow
5. Monitorizar as primeiras 3 execuções reais

Não juntar múltiplas alterações no mesmo deploy.

---

## Rollback

```
n8n UI → Workflows → {workflow} → ... → Execution History
```

O n8n guarda versões anteriores do workflow. Para reverter:
1. Desactivar o workflow actual
2. Abrir execução anterior bem-sucedida
3. Copiar a definição do workflow desse estado
4. Importar como nova versão

---

## Variáveis de Ambiente Completas

```env
# n8n (se self-hosted)
N8N_LOG_LEVEL=warn
EXECUTIONS_DATA_SAVE_ON_ERROR=all
EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
EXECUTIONS_DATA_MAX_AGE=7
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD={senha forte}
WEBHOOK_URL=https://n8n.{dominio}
```

---

## Operação Diária

| Hora | Acção automática |
|------|-----------------|
| 08:00 | WF-003 cron dispara — envia lembretes do dia seguinte |
| Qualquer hora | WF-001 processa novas marcações em tempo real |
| Qualquer hora | WF-002 processa respostas de pacientes em tempo real |

**Verificação semanal (manual):**
- Abrir n8n → Executions → verificar que não há erros acumulados
- Verificar taxa de entrega no Evolution API
