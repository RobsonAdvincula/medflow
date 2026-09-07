# Assessment de Segurança e Privacidade — MedFlow

**Data:** 2026-03-27  
**Classificação de dados:** Dados pessoais de saúde (RGPD Categoria Especial — Art.º 9.º)

---

## Lethal Trifecta

O MedFlow **não atinge o Lethal Trifecta** (dados sensíveis + internet exposure + low control). No entanto, lida com dados pessoais de categorias especiais (dados de saúde — nome, telefone, data de consulta, médico, especialidade), pelo que se aplicam as medidas abaixo.

---

## Classificação de Dados

| Dado | Categoria RGPD | Presente nos workflows |
|------|---------------|----------------------|
| Nome do paciente | Dados pessoais | Sim — mensagem WhatsApp |
| Número de telefone | Dados pessoais | Sim — routing Evolution API |
| Data/hora da consulta | Dados de saúde (indirectamente) | Sim |
| Médico e especialidade | Dados de saúde (indirectamente) | Sim |
| Diagnóstico / resultado | Dados de saúde directos | **Não** — fora de âmbito |

---

## Superfície de Ataque

| Componente | Exposição | Mitigação |
|-----------|-----------|-----------|
| Webhook n8n (WF-001) | Internet-facing, recebe dados de pacientes | Autenticação por header secreto; HTTPS obrigatório |
| Evolution API | Internet-facing | API key em variável de ambiente; não exposta no código |
| n8n credentials | Armazenado em n8n | Usar n8n Credentials nativas, não variáveis de texto |
| Dashboard HTML | Estático, sem autenticação de utilizador | Servir apenas em rede interna da clínica ou com Basic Auth nginx |
| API da clínica | Depende da clínica | Bearer token; HTTPS; validar certificado |

---

## Ameaças Identificadas e Controlos

### T1 — Intercepção de dados em trânsito
- **Risco:** Baixo (HTTPS em todos os endpoints)
- **Controlo:** TLS 1.2+ obrigatório em Evolution API, n8n e API da clínica; verificação de certificado activa

### T2 — Acesso não autorizado ao webhook
- **Risco:** Médio
- **Controlo:** Header `X-MedFlow-Secret` validado no início do WF-001; pedidos sem o header são rejeitados com 401

### T3 — Exposição de dados em logs n8n
- **Risco:** Médio
- **Controlo:** Configurar `N8N_LOG_LEVEL=warn`; desactivar persistência de execuções com dados pessoais após 7 dias (`EXECUTIONS_DATA_MAX_AGE=7`)

### T4 — Acesso ao dashboard por pessoal não autorizado
- **Risco:** Médio (se exposto na internet)
- **Controlo:** Dashboard deve ser servido apenas em rede interna; se exposição externa necessária, adicionar Basic Auth no servidor web

### T5 — Comprometimento da API key Evolution API
- **Risco:** Baixo
- **Controlo:** API key em n8n Credentials (não em texto nos nodes); rotação trimestral

---

## RGPD — Obrigações

| Obrigação | Estado |
|-----------|--------|
| Base legal para tratamento (consentimento ou execução de contrato) | Responsabilidade da clínica (controlador); MedFlow é subcontratante |
| Minimização de dados — apenas o necessário para o contacto | Cumprido — sem diagnósticos ou dados clínicos |
| Retenção limitada — dados não guardados além do necessário | n8n configurado para purga de execuções após 7 dias |
| Acordo de subcontratação (DPA) com operador n8n / Evolution API | A assinar pela clínica com os fornecedores |
| Registo de actividades de tratamento | A incluir no registo RGPD da clínica |

---

## Recomendações para Produção

1. Activar `X-MedFlow-Secret` no webhook antes de expor ao exterior
2. Configurar n8n com `EXECUTIONS_DATA_SAVE_ON_SUCCESS=none` para não guardar payloads de execuções bem-sucedidas
3. Servir o dashboard em HTTPS com autenticação (Basic Auth ou SSO)
4. Rever a política de retenção de logs com o DPO da clínica
5. Documentar o MedFlow no registo RGPD da clínica como subcontratante de dados de saúde
