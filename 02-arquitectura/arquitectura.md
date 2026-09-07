# Arquitectura — MedFlow

**Data:** 2026-03-27

---

## Visão Geral

O MedFlow é um sistema de automação de fluxo de trabalho baseado em n8n, sem base de dados própria. O estado dos atendimentos vive na API da clínica. Os workflows são desencadeados por webhooks ou por cron e comunicam com o paciente exclusivamente via WhatsApp (Evolution API).

---

## Diagrama de Componentes

```mermaid
graph TB
    subgraph Clínica
        A[Sistema de Agenda\nda Clínica] 
        B[Dashboard Web\nclinica-atendimento.html]
        C[Staff da Clínica]
    end

    subgraph MedFlow
        D[n8n · WF-001\nConfirmação]
        E[n8n · WF-002\nStatus Tempo Real]
        F[n8n · WF-003\nLembrete 24h\ncron 08:00]
    end

    subgraph Externo
        G[Evolution API\nGateway WhatsApp]
        H[WhatsApp\nPaciente]
        I[API da Clínica\nREST endpoint]
    end

    A -->|POST webhook\nnova marcação| D
    D -->|sendText| G
    G -->|mensagem| H
    H -->|resposta| G
    G -->|webhook entrada| E
    E -->|PATCH status| I
    F -->|GET marcações amanhã| I
    F -->|sendText lembretes| G
    I -->|GET atendimentos| B
    C -->|visualiza| B
```

---

## Fluxo de Dados — WF-001 (Confirmação)

```mermaid
sequenceDiagram
    participant A as Agenda Clínica
    participant N as n8n WF-001
    participant E as Evolution API
    participant P as Paciente (WhatsApp)
    participant API as API Clínica

    A->>N: POST /webhook/medflow-confirmar\n{paciente, telefone, data, horario, medico, atendimento_id}
    N->>N: Formatar mensagem de confirmação
    N->>E: POST /message/sendText/medflow\n{number, textMessage}
    E->>P: Mensagem WhatsApp
    N->>API: PATCH /atendimentos/{id}\n{status: "confirmacao_enviada"}
    N-->>A: 200 OK
```

---

## Fluxo de Dados — WF-002 (Status em Tempo Real)

```mermaid
sequenceDiagram
    participant P as Paciente (WhatsApp)
    participant E as Evolution API
    participant N as n8n WF-002
    participant API as API Clínica

    P->>E: Resposta do paciente
    E->>N: POST webhook Evolution\n{message, from, body}
    N->>N: Classificar resposta\n(confirmado/cancelado/reagendado/desconhecido)
    alt confirmado
        N->>API: PATCH /atendimentos/{id}\n{status: "confirmado"}
    else cancelado
        N->>API: PATCH /atendimentos/{id}\n{status: "cancelado"}
    else reagendado
        N->>API: PATCH /atendimentos/{id}\n{status: "pendente_reagendamento"}
    end
```

---

## Fluxo de Dados — WF-003 (Lembrete 24h)

```mermaid
sequenceDiagram
    participant CRON as Cron 08:00
    participant N as n8n WF-003
    participant API as API Clínica
    participant E as Evolution API
    participant P as Pacientes

    CRON->>N: Trigger diário
    N->>API: GET /atendimentos?data={amanha}&status=confirmado
    API-->>N: Lista de marcações
    loop Para cada marcação
        N->>N: Formatar lembrete personalizado
        N->>E: POST /message/sendText/medflow
        E->>P: Mensagem lembrete
    end
```

---

## Decisões de Design

| Decisão | Razão |
|---------|-------|
| Sem base de dados própria | O estado dos atendimentos já existe na API da clínica; duplicar seria fonte de inconsistências |
| Dashboard HTML estático | Sem necessidade de backend próprio; o dashboard consome a API da clínica directamente |
| Evolution API para WhatsApp | API estável, auto-hospedável, compatível com n8n via HTTP Request |
| n8n como motor | Workflows visíveis e editáveis pelo operador sem código; portabilidade garantida por JSON export |

---

## Variáveis de Ambiente

| Variável | Descrição |
|----------|-----------|
| `EVOLUTION_API_URL` | Base URL da instância Evolution API |
| `EVOLUTION_API_KEY` | API key da instância |
| `EVOLUTION_INSTANCE` | Nome da instância WhatsApp (ex: `medflow`) |
| `MEDFLOW_API_URL` | Base URL da API da clínica |
| `MEDFLOW_API_TOKEN` | Token de autenticação da API da clínica |
