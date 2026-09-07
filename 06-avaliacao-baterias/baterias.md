# Baterias de Avaliação — MedFlow

**Data:** 2026-03-27

---

## Estratégia de Testes

O MedFlow é um sistema de automação sem lógica de negócio complexa no código. Os testes são maioritariamente de integração e end-to-end, executados com dados reais em ambiente de staging antes de produção.

---

## Bateria 1 — WF-001 Confirmação

### Teste 1.1 — Confirmação enviada com sucesso

**Pré-condição:** n8n activo, Evolution API online, número WhatsApp de teste disponível

```bash
curl -X POST https://n8n.{dominio}/webhook/medflow-confirmar \
  -H "Content-Type: application/json" \
  -H "X-MedFlow-Secret: {secret}" \
  -d '{
    "paciente": "João Silva",
    "telefone": "351912345678",
    "data": "2026-04-10",
    "horario": "14:30",
    "medico": "Dra. Ana Costa",
    "especialidade": "Medicina Geral",
    "atendimento_id": "ATD-001-TEST"
  }'
```

**Resultado esperado:**
- HTTP 200 com `{ "status": "ok" }`
- Mensagem WhatsApp recebida no número de teste em < 5 s
- Status `ATD-001-TEST` actualizado para `confirmacao_enviada` na API da clínica

### Teste 1.2 — Rejeição sem header secreto

```bash
curl -X POST https://n8n.{dominio}/webhook/medflow-confirmar \
  -H "Content-Type: application/json" \
  -d '{"paciente": "Teste"}'
```

**Resultado esperado:** HTTP 401 `{ "error": "Unauthorized" }`

---

## Bateria 2 — WF-002 Status em Tempo Real

### Teste 2.1 — Paciente confirma

Enviar do número de teste: "Sim, confirmo"

**Resultado esperado:** Status do atendimento muda para `confirmado` na API da clínica em < 10 s

### Teste 2.2 — Paciente cancela

Enviar do número de teste: "Preciso cancelar"

**Resultado esperado:** Status muda para `cancelado`

### Teste 2.3 — Resposta ambígua

Enviar do número de teste: "Talvez"

**Resultado esperado:** Status muda para `pendente` (não bloqueia o fluxo)

---

## Bateria 3 — WF-003 Lembrete 24h

### Teste 3.1 — Lembrete enviado para marcação de amanhã

**Pré-condição:** Criar marcação na API da clínica para o dia seguinte com status `confirmado`

Forçar execução manual do WF-003 no n8n.

**Resultado esperado:**
- Mensagem WhatsApp de lembrete recebida no número de teste
- Log de execução n8n mostra 1 mensagem enviada

### Teste 3.2 — Sem marcações amanhã

**Pré-condição:** Nenhuma marcação para o dia seguinte

**Resultado esperado:** Execução conclui sem erros; 0 mensagens enviadas; log regista "Nenhuma marcação encontrada"

---

## Bateria 4 — Dashboard

### Teste 4.1 — Atendimentos visíveis

Abrir `clinica-atendimento.html` num browser.

**Resultado esperado:** Lista de atendimentos carrega em < 2 s; estados corretos visíveis

### Teste 4.2 — Actualização em tempo real

Alterar status de um atendimento via API da clínica.

**Resultado esperado:** Dashboard reflecte a mudança no próximo polling (≤ 30 s)

---

## Critérios de Aceitação para Produção

| Critério | Mínimo |
|---------|--------|
| Taxa de entrega WhatsApp (50 consultas piloto) | ≥ 95% |
| Tempo resposta webhook WF-001 | < 2 s (p95) |
| Falsos positivos na classificação de respostas | < 5% |
| Lembretes perdidos num ciclo de 7 dias | 0 |
