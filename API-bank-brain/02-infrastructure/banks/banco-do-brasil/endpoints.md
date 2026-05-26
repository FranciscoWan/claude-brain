---
title: Banco do Brasil — Endpoints
category: infrastructure
bank: banco-do-brasil
status: stable
updated: 2026-05-26
parent: "[[banco-do-brasil_index]]"
tags:
  - infrastructure
  - banco-do-brasil
  - endpoints
  - stable
  - bank
---

# Banco do Brasil — Endpoints

## Contexto

Endpoints da API do BB consumidos pelo ViaPIX — OAuth2 para autenticação e PIX para operações de cobrança.

## Conteúdo principal

### Variáveis de ambiente

| Variável | Descrição |
|---|---|
| `BB_OAUTH_URL` | URL base do servidor OAuth2 BB |
| `BB_PIX_URL` | URL base da API PIX BB |
| `BB_GW_APP_KEY` | App key do developer.bb.com.br (obrigatório em todas as chamadas) |

### Endpoints (Sandbox)

| Operação | Método | Rota |
|---|---|---|
| Token OAuth2 | POST | `{BB_OAUTH_URL}/oauth/token` |
| Criar COB | POST | `{BB_PIX_URL}/v2/cob` |
| Consultar COB | GET | `{BB_PIX_URL}/v2/cob/{txid}` |
| Atualizar COB | PATCH | `{BB_PIX_URL}/v2/cob/{txid}` |
| Receber webhook | POST | `/pix/bb/webhook/pix` (rota ViaPIX, não BB) |

### Autenticação OAuth2

- Fluxo: `client_credentials`
- Token cacheado por `clientId` (evita re-autenticação a cada requisição)
- mTLS obrigatório: agente HTTPS configurado com `certPem` + `certKeyPem` do cliente
- `gw-dev-app-key` enviado como query param em todas as chamadas à API PIX

```typescript
const httpsAgent = new https.Agent({
  cert: credentials.certPem,
  key: credentials.certKeyPem,
  rejectUnauthorized: process.env.NODE_ENV !== 'production',
});
```

### Formato do token

```http
POST {BB_OAUTH_URL}/oauth/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(clientId:clientSecret)

grant_type=client_credentials&scope=cob.write cob.read webhook.read
```

## Relacionados

- [[banco-do-brasil_index]] — índice do Banco do Brasil

