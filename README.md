# MedFlow — Sistema de Atendimento para Clínicas

> Automação completa do ciclo de atendimento para clínicas médicas — confirmações, lembretes e status de consultas via WhatsApp, com dashboard web.

![n8n](https://img.shields.io/badge/n8n-workflow-orange)
![Evolution API](https://img.shields.io/badge/Evolution_API-WhatsApp-green)
![HTML](https://img.shields.io/badge/Dashboard-HTML/CSS/JS-blue)

---

## O Problema

Clínicas perdem consultas por falta de confirmação atempada. O processo manual de contactar pacientes, confirmar presenças e enviar lembretes é demorado e propenso a erros.

## A Solução

O MedFlow automatiza o ciclo completo:

1. **Confirmação** — envia pedido de confirmação da consulta via WhatsApp
2. **Status em tempo real** — actualiza o estado da consulta automaticamente
3. **Lembrete 24h** — envia lembrete automático no dia anterior

Tudo integrado num dashboard web para a equipa da clínica.

---

## Arquitectura

```
Agenda da Clínica
       │
       ▼
   n8n Trigger
       │
   ┌───┴────────────────────┐
   │                        │
   ▼                        ▼
WF-001                   WF-002
Confirmação              Status
WhatsApp                 Tempo Real
   │                        │
   └───────────┬────────────┘
               │
               ▼
            WF-003
           Lembrete
            24h
               │
               ▼
        Evolution API
               │
               ▼
           WhatsApp
          (Paciente)
```

---

## Stack

| Componente | Ferramenta |
|------------|-----------|
| Orquestração | n8n |
| WhatsApp | Evolution API |
| Dashboard | HTML + CSS + JavaScript |
| Integrações | Webhooks REST |

---

## Workflows

| Workflow | Função |
|----------|--------|
| WF-001 | Confirmação de consulta via WhatsApp |
| WF-002 | Actualização de status em tempo real |
| WF-003 | Lembrete automático 24h antes |

---

## Funcionalidades

- Confirmação automática de consultas
- Lembretes 24h antes da consulta
- Dashboard de gestão de atendimentos
- Página de integrações configuráveis
- Redução de faltas sem intervenção manual

---

*Built by [Robson Advincula](https://linkedin.com/in/robsonadvincula) — AI & Automation Consultant*
