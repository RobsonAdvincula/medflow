# Snapshot de Código — MedFlow

**Data:** 2026-03-27

---

## Ficheiros de Código no Repositório

| Ficheiro | Tipo | Descrição |
|---------|------|-----------|
| `WF-001-evolution-confirmacao.json` | n8n workflow export | Recebe webhook de marcação; envia confirmação WhatsApp; actualiza status |
| `WF-002-evolution-status.json` | n8n workflow export | Escuta respostas WhatsApp; classifica e actualiza status do atendimento |
| `WF-003-evolution-lembrete.json` | n8n workflow export | Cron 08:00; envia lembretes do dia seguinte |
| `clinica-atendimento.html` | HTML/CSS/JS | Dashboard de gestão de atendimentos; polling REST a cada 30 s |
| `n8n-integracoes.html` | HTML/CSS/JS | Página de configuração de integrações (endpoints, API keys de forma visual) |

---

## Estrutura dos Workflows n8n

### WF-001 — Nós

1. **Webhook · Receber Agendamento** — `POST /webhook/medflow-confirmar`; expõe endpoint público
2. **Set · Formatar Dados** — normaliza telefone, formata mensagem WhatsApp com dados do paciente
3. **Evolution API · Enviar Confirmação** — `POST /message/sendText/{instancia}` via HTTP Request
4. **API Clínica · Actualizar Status** — `PATCH /atendimentos/{id}` com `{status: "confirmacao_enviada"}`
5. **Respond to Webhook** — retorna `200 OK` à agenda da clínica

### WF-002 — Nós

1. **Webhook · Receber Resposta Evolution** — escuta eventos `messages.upsert` da Evolution API
2. **Set · Extrair Mensagem** — extrai corpo da mensagem e número do remetente
3. **Switch · Classificar Resposta** — regex para confirmar / cancelar / reagendar / ambíguo
4. **API Clínica · Actualizar Status** — PATCH com status correspondente

### WF-003 — Nós

1. **Cron · 08:00 Diário** — trigger diário
2. **API Clínica · Buscar Marcações Amanhã** — `GET /atendimentos?data={amanha}`
3. **SplitInBatches** — itera sobre cada marcação
4. **Set · Formatar Lembrete** — mensagem personalizada por paciente
5. **Evolution API · Enviar Lembrete** — `POST /message/sendText/{instancia}`

---

## Padrão de Mensagem WhatsApp — WF-001

```
Olá, *{nome}*! 👋

Sua consulta foi confirmada com sucesso! ✅

📅 *Data:* {data}
🕐 *Horário:* {horario}
👨‍⚕️ *Médico(a):* {medico}
🏥 *Especialidade:* {especialidade}

Caso precise reagendar ou cancelar, responda esta mensagem.

_MedFlow · Sistema de Atendimento_
```

---

## Nota sobre Credenciais no Código

Os ficheiros JSON dos workflows exportados do n8n **não contêm credenciais** — as referências são por ID de credential do n8n. Ao importar noutro n8n, é necessário recriar as credentials e associá-las.

O campo `apikey` visível no JSON de exemplo do README é apenas um placeholder de demonstração. Em produção, toda a autenticação está em n8n Credentials, nunca em texto nos nodes.
