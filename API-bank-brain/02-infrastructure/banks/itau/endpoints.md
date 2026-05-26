---
title: Itaú — Endpoints
category: infrastructure
bank: itau
status: stable
updated: 2026-05-26
parent: "[[itau_index]]"
tags:
  - infrastructure
  - itau
  - endpoints
  - stable
  - bank
---

# Itaú — Endpoints

## Contexto

Endpoints da API do Itaú consumidos pelo ViaPIX — OAuth2 via APIGateway STS e rotas PIX para COB, COBV, PIX e Devolução.

## Conteúdo principal

### Variáveis de ambiente

| Variável | Descrição |
|---|---|
| `ITAU_OAUTH_URL` | URL do OAuth2 Itaú (STS APIGateway) |
| `ITAU_PIX_URL` | URL base da API PIX Itaú |

### Endpoints (Sandbox)

| Operação | Método | Rota |
|---|---|---|
| Token OAuth2 | POST | `{ITAU_OAUTH_URL}/oauth/token` |
| Criar COB | POST | `{ITAU_PIX_URL}/v2/cob` |
| Consultar COB | GET | `{ITAU_PIX_URL}/v2/cob/{txid}` |
| Atualizar COB | PATCH | `{ITAU_PIX_URL}/v2/cob/{txid}` |
| Criar COBV | POST | `{ITAU_PIX_URL}/v2/cobv` |
| Criar PIX | POST | `{ITAU_PIX_URL}/v2/pix` |
| Devolução | POST | `{ITAU_PIX_URL}/v2/pix/{e2eId}/devolucao/{id}` |

### Autenticação OAuth2

- Fluxo: `client_credentials`
- Endpoint STS do APIGateway Itaú (não o padrão do Banco Central)
- Token cacheado por `clientId`
- Sem mTLS

### Formato do token

```http
POST {ITAU_OAUTH_URL}/oauth/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(clientId:clientSecret)

grant_type=client_credentials
```

## Relacionados

- [[itau_index]] — índice do Itaú
