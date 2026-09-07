# Guia de Demo e Onboarding — MedFlow

**Data:** 2026-03-27

---

## Público-alvo da Demo

- Gestor/director da clínica médica
- Recepcionista / coordenadora de agendamentos
- Responsável de TI da clínica (se existir)

---

## Sequência de Demo (15 minutos)

### Parte 1 — O problema (2 min)

Mostrar o fluxo manual actual:
- Ligar para o paciente → esperar → registar resposta → agendar lembrete manual
- Perguntar: "Quantas chamadas faz por semana só para confirmações?"

### Parte 2 — O fluxo automático ao vivo (8 min)

**Setup prévio:** Ter o n8n de demo activo com os 3 workflows; número WhatsApp de demo com um número real do interlocutor.

1. **Simular nova marcação:** Enviar o curl de teste com o número do interlocutor como destinatário
2. O interlocutor recebe a mensagem no próprio WhatsApp em segundos
3. O interlocutor responde "Sim" — mostrar que o status actualiza no dashboard em tempo real
4. Mostrar o dashboard `clinica-atendimento.html` ao vivo
5. Explicar o WF-003: "Às 08:00 de amanhã, todos os pacientes com consulta no dia seguinte recebem um lembrete sem mais nenhuma acção"

### Parte 3 — Dashboard (3 min)

- Mostrar a lista de atendimentos com estados
- Mostrar a página de integrações — "a equipa pode ver e ajustar as configurações aqui"

### Parte 4 — Próximos passos (2 min)

- Quantas consultas tem por semana?
- Qual é o software de agenda actual? (para avaliar a integração do webhook)
- Definir data de arranque piloto

---

## Onboarding de Nova Clínica

### Informação necessária da clínica

| Campo | Descrição |
|-------|-----------|
| Número WhatsApp | Número a usar para envio de mensagens (deve estar activo e aquecido) |
| Endpoint de agenda | URL da API ou sistema de agenda para webhook |
| Formato de dados | Schema do payload de marcação (nome, telefone, data, hora, médico) |
| Responsável técnico | Contacto para coordenar integração do webhook |

### Checklist de arranque

- [ ] Número WhatsApp configurado na instância Evolution API
- [ ] Webhooks de marcação a apontar para o endpoint n8n da clínica
- [ ] Credenciais da API da clínica configuradas no n8n
- [ ] Teste piloto com 10 marcações reais validado
- [ ] Staff formado no uso do dashboard
- [ ] Contacto de suporte comunicado

---

## Formação de Staff (30 min)

1. **O que o sistema faz** — confirmação automática + lembretes (sem intervenção para o ciclo standard)
2. **O que o staff deve fazer** — verificar o dashboard diariamente; escalar casos com status `pendente_reagendamento`
3. **O que fazer se algo falha** — verificar n8n → se workflow com erro, contactar suporte Wise Pirates
4. **O que NÃO fazer** — não enviar confirmações manuais enquanto o sistema está activo (duplicação confunde o paciente)
