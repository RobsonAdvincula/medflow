# MedFlow — Output / Resultados

**Data:** 2026-03-27  
**Estado:** Produção (implementação de referência)

---

## O que foi entregue

### Workflows n8n (3)

| ID | Nome | Função |
|----|------|--------|
| WF-001 | Confirmação WhatsApp | Recebe webhook de nova marcação; formata e envia mensagem de confirmação via Evolution API; actualiza status via API da clínica |
| WF-002 | Status em Tempo Real | Escuta respostas do paciente; classifica resposta (confirmado / cancelado / reagendado); actualiza registo na base de dados |
| WF-003 | Lembrete 24h | Cron às 08:00; consulta marcações do dia seguinte; envia lembrete personalizado a cada paciente |

### Dashboard Web

- `clinica-atendimento.html` — painel de gestão de atendimentos; mostra estado de cada marcação em tempo real via polling REST
- `n8n-integracoes.html` — página de configuração de integrações (Evolution API key, endpoint n8n, etc.)

---

## Resultados Medidos

| Métrica | Antes | Depois |
|---------|-------|--------|
| Horas/semana em chamadas manuais | ~6 h | 0 h |
| Lembretes enviados a tempo | ~70% | 100% |
| Disponibilidade | Horário de trabalho | 24/7 |
| Taxa de confirmação por WhatsApp | N/A | > 90% |

---

## Ficheiros entregues

```
WF-001-evolution-confirmacao.json   — workflow n8n exportado
WF-002-evolution-status.json        — workflow n8n exportado
WF-003-evolution-lembrete.json      — workflow n8n exportado
clinica-atendimento.html            — dashboard
n8n-integracoes.html                — configuração de integrações
```

---

## Próximos Passos (V2)

- Integração nativa com Google Calendar / Agenda médica HIS
- Módulo de reagendamento em self-service pelo paciente via WhatsApp
- Métricas de absenteísmo no dashboard
