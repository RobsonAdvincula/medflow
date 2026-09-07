# Decisões de Arquitectura — MedFlow

**Data:** 2026-03-27

---

## ADR-001 — n8n como motor de workflows (2026-03-27)

**Contexto:** Precisávamos de orquestrar 3 fluxos de trabalho assíncronos com triggers diferentes (webhook, cron, webhook de entrada Evolution API).

**Decisão:** Usar n8n como motor de workflows em vez de código custom (Python/Node.js).

**Razões:**
- Workflows visíveis e editáveis pelo operador sem código
- Portabilidade garantida — exportar/importar JSON
- Suporte nativo a webhooks, cron e HTTP Request
- Menor superfície de manutenção vs. backend custom

**Consequências:**
- Dependência do n8n; atualizações podem quebrar nodes
- Lógica de negócio não testável com testes unitários convencionais
- Aceitável para automação de fluxo — não é lógica financeira crítica

---

## ADR-002 — Sem base de dados própria (2026-03-27)

**Contexto:** Poderíamos ter criado uma base de dados MedFlow para guardar o estado dos atendimentos.

**Decisão:** Usar a API da clínica como fonte de verdade; MedFlow não guarda estado persistente.

**Razões:**
- Evitar duplicação de dados de saúde — RGPD mais simples
- A clínica já tem o sistema de agenda; duplicar cria risco de inconsistência
- Menos infra para gerir

**Consequências:**
- MedFlow depende da disponibilidade da API da clínica
- Se a API estiver em baixo, o WF-002 não consegue actualizar status — aceitável (os status ficam desactualizados mas a mensagem chegou)

---

## ADR-003 — Dashboard HTML estático (2026-03-27)

**Contexto:** Precisávamos de um painel para a equipa da clínica visualizar atendimentos.

**Decisão:** Ficheiro HTML estático com polling REST, sem backend próprio.

**Razões:**
- Zero infra adicional
- Deployável em qualquer servidor web ou mesmo aberto como ficheiro local
- A clínica é pequena — 1-5 utilizadores simultâneos; sem necessidade de backend

**Consequências:**
- Sem autenticação de utilizadores nativa — responsabilidade da rede da clínica
- Se a API da clínica tiver CORS restritivo, o dashboard não funciona — resolver com proxy nginx

---

## ADR-004 — Evolution API para WhatsApp (2026-03-27)

**Contexto:** Alternativas avaliadas: Twilio WhatsApp API (oficial Meta), Cloud API Meta direta, Evolution API.

**Decisão:** Evolution API auto-hospedada.

**Razões:**
- Sem custo por mensagem (vs. Twilio ~0.05 USD/msg)
- Compatível com n8n via HTTP Request standard
- Controlo total da instância

**Consequências:**
- Risco de ban do número pela Meta se comportamento for detectado como não-humano — mitigar com delays e volume gradual
- Responsabilidade de manutenção da instância Evolution API

---

## Lições Aprendidas

| Lição | Data | Contexto |
|-------|------|---------|
| Testar classificação de respostas com frases reais de pacientes antes de ir a produção | 2026-03-27 | "Obrigado" foi classificado inicialmente como "confirmado" — corrigido para "ambíguo" |
| Activar WF-002 antes do WF-001 | 2026-03-27 | Se o WF-001 enviar confirmações enquanto o WF-002 ainda não está a escutar, as primeiras respostas perdem-se |
| Desactivar workflows antes de editar em produção | 2026-03-27 | Uma edição a meio de uma execução causou payload corrompido |
