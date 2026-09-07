# Observabilidade — MedFlow

**Data:** 2026-03-27

---

## Estratégia

O MedFlow assenta em n8n como motor de workflows. A observabilidade nativa do n8n (execuções, logs, erros) cobre a maioria das necessidades operacionais. Não existe backend próprio que requeira APM adicional.

---

## O que monitorizar

| O quê | Onde ver | Alarme |
|-------|---------|--------|
| Execuções falhadas (WF-001/002/003) | n8n → Executions → Error | Sim — email para operador |
| Taxa de entrega WhatsApp | Evolution API → Logs de mensagens | Verificação diária |
| Webhook sem resposta (WF-001) | n8n timeout + resposta da clínica | Sim — alerta se timeout > 5 s |
| Cron WF-003 não correu às 08:00 | n8n → Executions → WF-003 | Sim — alerta se ausente às 08:15 |
| Uptime n8n | Health check `/healthz` | Sim — UptimeRobot ou equivalente |

---

## Configuração de Alertas n8n

### Email em caso de erro

No n8n, activar **Error Workflow** global:
1. Criar workflow `medflow-error-handler` com trigger `Error Trigger`
2. Adicionar node `Send Email` ou `HTTP Request → Slack/webhook`
3. Associar em **Settings → Error Workflow**

### Payload de alerta mínimo

```json
{
  "workflow": "{{ $workflow.name }}",
  "executionId": "{{ $execution.id }}",
  "error": "{{ $json.message }}",
  "timestamp": "{{ $now.toISO() }}"
}
```

---

## Logs

### n8n

```env
N8N_LOG_LEVEL=warn
N8N_LOG_OUTPUT=console
EXECUTIONS_DATA_SAVE_ON_ERROR=all
EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
EXECUTIONS_DATA_MAX_AGE=7
```

Manter execuções de erro por 7 dias para diagnóstico; não guardar execuções bem-sucedidas (dados pessoais minimizados).

### Evolution API

- Logs disponíveis no painel da instância Evolution API
- Verificar mensagens não entregues (status `PENDING` > 5 min)

---

## Dashboard de Estado (sem infra adicional)

O dashboard `clinica-atendimento.html` faz polling à API da clínica a cada 30 s. Serve também como indicador de saúde operacional — se o painel mostrar atendimentos a acumular no estado `confirmacao_enviada` sem avançar, o WF-002 pode estar em falha.

---

## Runbook de Diagnóstico Rápido

| Sintoma | Verificar |
|---------|-----------|
| Confirmações não estão a ser enviadas | n8n Executions → WF-001 → ver erro; verificar API key Evolution API |
| Status não actualiza após resposta do paciente | n8n Executions → WF-002; verificar webhook Evolution configurado |
| Lembretes não enviados hoje | n8n → WF-003 → execução das 08:00; verificar se cron está activo |
| WhatsApp entregue mas status não muda | API da clínica em baixo; verificar endpoint MEDFLOW_API_URL |
