# Autenticação — MedFlow

**Data:** 2026-03-27

---

## Componentes e Mecanismos de Autenticação

### 1. Evolution API

- **Mecanismo:** API key por header `apikey`
- **Scope:** Por instância WhatsApp (ex: instância `medflow`)
- **Armazenamento:** n8n Credentials (tipo HTTP Header Auth) — nunca em texto no node
- **Rotação:** Trimestral ou imediatamente após suspeita de comprometimento

### 2. n8n Webhooks (entrada)

- **Mecanismo:** Header secreto `X-MedFlow-Secret` validado no primeiro node do WF-001
- **Geração do segredo:** `openssl rand -hex 32`
- **Configuração:** A agenda da clínica envia o header em cada POST; n8n valida e rejeita sem o header correcto

```javascript
// Node de validação no início do WF-001
if ($request.headers['x-medflow-secret'] !== $vars.WEBHOOK_SECRET) {
  return $response.json({ error: 'Unauthorized' }, { statusCode: 401 });
}
```

### 3. API da Clínica (saída)

- **Mecanismo:** Bearer token no header `Authorization`
- **Armazenamento:** n8n Credentials (tipo Bearer Token)
- **Validade:** Definida pela clínica; verificar expiração

### 4. Dashboard Web

- O dashboard HTML é um ficheiro estático sem autenticação própria
- **Recomendação de produção:** Servir atrás de nginx com Basic Auth ou acesso apenas via VPN da clínica
- A autenticação de utilizadores do staff não está implementada no dashboard — é responsabilidade da rede da clínica

---

## Autenticação de Utilizadores (Staff)

O MedFlow V1 não implementa autenticação de utilizadores no dashboard. O acesso é controlado pela rede:

| Cenário | Controlo |
|---------|---------|
| Dashboard em rede interna da clínica | Firewall/router da clínica limita o acesso |
| Dashboard em servidor externo | Basic Auth nginx; credenciais únicas por clínica |
| Dashboard em produção multi-clínica | Fora de âmbito V1 — requererá SSO (ex: Zitadel) |

---

## Matriz de Credenciais

| Credencial | Dono | Armazenamento | Rotação |
|-----------|------|--------------|---------|
| Evolution API key | Operador MedFlow | n8n Credentials | Trimestral |
| Webhook secret | Operador MedFlow | n8n Variable | Trimestral |
| Bearer token API clínica | Clínica | n8n Credentials | Por política da clínica |
| Basic Auth dashboard | Operador / clínica | nginx .htpasswd | Anual |
