# MedFlow — Plano de Projecto

**Versão:** 1.0  
**Data:** 2026-03-27  
**Responsável:** Robson Advincula

---

## Contexto

Clínicas médicas perdem consultas e horas de equipa em confirmações manuais por telefone. O MedFlow automatiza o ciclo completo de atendimento via WhatsApp, desde a confirmação de marcação até ao lembrete de 24h, com dashboard web para a equipa.

## Objectivos

1. Eliminar chamadas manuais para confirmação de consultas
2. Garantir que nenhum lembrete de 24h é esquecido, independentemente de quem está de turno
3. Disponibilizar ao staff um painel centralizado do estado de cada atendimento

## Âmbito

**Incluído:**
- WF-001: Webhook recebe marcação → envia confirmação WhatsApp via Evolution API
- WF-002: Actualização de status em tempo real (confirmado/cancelado/reagendado)
- WF-003: Lembrete automático 24h antes da consulta
- Dashboard HTML/CSS/JS para gestão de atendimentos
- Página de configuração de integrações

**Excluído (versão 1.0):**
- Módulo de pagamentos online
- Integração com software de agenda médica (HIS)
- App mobile para pacientes

## Fases

| Fase | Entregável | Estado |
|------|-----------|--------|
| 1 | WF-001 confirmação WhatsApp | Concluído |
| 2 | WF-002 status em tempo real | Concluído |
| 3 | WF-003 lembrete 24h | Concluído |
| 4 | Dashboard web | Concluído |
| 5 | Testes end-to-end com clínica piloto | Concluído |

## Critérios de Sucesso

- Taxa de confirmação automática ≥ 90% das consultas marcadas
- Zero lembretes perdidos
- Tempo de resposta do webhook < 2 s
- Dashboard carrega em < 1 s (sem backend — ficheiro estático)
