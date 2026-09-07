# Plano Estratégico — MedFlow

**Data:** 2026-03-27  
**Responsável:** Robson Advincula  
**Estado:** Implementação concluída

---

## 1. Problema de Negócio

Clínicas médicas de pequeno e médio porte perdem consultas por falta de confirmação atempada e esquecem lembretes de 24h quando a equipa está reduzida. O processo manual — ligar para cada paciente, registar resposta, reenviar se necessário — consome 4 a 8 horas semanais de staff.

## 2. Visão

Sistema automático que gere o ciclo completo de confirmação de consulta sem intervenção humana para o caso standard, libertando a equipa para atendimento presencial de qualidade.

## 3. Objectivos de Negócio

| # | Objectivo | Indicador | Meta |
|---|----------|-----------|------|
| 1 | Eliminar chamadas manuais de confirmação | Horas/semana gastas em chamadas | < 1 h |
| 2 | Aumentar taxa de confirmação | % consultas confirmadas 24h antes | ≥ 90% |
| 3 | Garantir lembretes sem falha | % lembretes enviados a tempo | 100% |
| 4 | Disponibilidade do sistema | Uptime | ≥ 99% |

## 4. Âmbito da Versão 1.0

**Incluído:**
- Webhook de recepção de nova marcação
- Envio de confirmação WhatsApp (Evolution API)
- Captura de resposta do paciente
- Actualização de status em tempo real
- Lembrete automático 24h antes (cron 08:00)
- Dashboard web para a equipa

**Excluído explicitamente:**
- Módulo de pagamentos — a clínica usa terminal físico; sem requisito para V1
- Integração com HIS (Health Information System) — APIs não foram disponibilizadas; fica para V2
- App mobile — custo-benefício negativo para a dimensão actual da clínica

## 5. Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|--------------|---------|-----------|
| Número WhatsApp bloqueado pela Meta | Médio | Alto | Usar conta Evolution API com aquecimento gradual; manter número de backup |
| n8n offline | Baixo | Alto | n8n em serviço gerido (Railway/cloud); alertas por email em caso de falha |
| Resposta do paciente não reconhecida | Médio | Médio | WF-002 trata ambiguidades; fallback para estado "pendente" sem bloquear fluxo |
| Dados de pacientes expostos | Baixo | Muito alto | Dados mínimos nos workflows; nome + telefone + data/hora apenas; ver `03-seguranca-e-privacidade/` |

## 6. Cronograma

| Semana | Actividade |
|--------|-----------|
| 1 | Setup n8n + Evolution API; WF-001 confirmação |
| 2 | WF-002 status em tempo real; testes de resposta |
| 3 | WF-003 lembrete 24h; dashboard |
| 4 | Testes end-to-end com clínica piloto; ajustes |

## 7. Dependências Externas

- **Evolution API** — gateway WhatsApp; requer conta com número activo
- **n8n** — motor de workflows; auto-hospedado ou cloud
- **API da clínica** — endpoint para actualizar status de atendimento (fornecido pelo cliente)

## 8. Critérios de Saída de Produção

- WF-001, WF-002, WF-003 testados com dados reais
- Dashboard visualiza estados correctamente
- Taxa de entrega WhatsApp ≥ 95% em teste piloto de 50 consultas
- Nenhum dado pessoal armazenado em logs em texto claro
