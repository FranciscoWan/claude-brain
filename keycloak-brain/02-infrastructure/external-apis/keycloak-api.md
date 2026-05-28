---
title: Keycloak API — Servidor Externo
category: infrastructure
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/external-apis/_index]]"
tags:
  - infrastructure
  - external-api
  - keycloak
  - stable
---

# Keycloak API — Servidor Externo

## Contexto

Endpoints do servidor Keycloak que o `KeycloakClient` consome. Todos derivam de `env.oauth.baseUrl` (ex: `https://keycloak.example.com/realms/<realm>/protocol/openid-connect`).

## Conteúdo principal

### Endpoints consumidos

| Endpoint | Método | Uso | Grant Types |
|---|---|---|---|
| `{baseUrl}/auth` | `GET` (redirect) | Inicia fluxo PKCE | — |
| `{baseUrl}/token` | `POST` | Troca de código, refresh, token exchange | `authorization_code`, `refresh_token`, `urn:ietf:params:oauth:grant-type:token-exchange` |
| `{baseUrl}/logout` | `POST` | Revogação de refresh token | — |

### Formato das requisições ao `/token`

Todas as requisições são `application/x-www-form-urlencoded`. O `client_secret` é adicionado automaticamente quando `env.oauth.clientSecret` está definido.

### Resposta padrão (`TokenExchangeResponse`)

```ts
{
  access_token: string,
  expires_in: number,
  refresh_token?: string,
  refresh_expires_in?: number,
  token_type: string,
  scope?: string
}
```

### Erros retornados pelo Keycloak

O corpo de erro segue o padrão OAuth2:

```json
{
  "error": "invalid_grant",
  "error_description": "Token is not active"
}
```

Campos mapeados em `KeycloakErrorBody`: `error`, `error_description`. O `error_description` tem prioridade na mensagem de erro ao usuário.

### Configuração de ambiente

| Variável (`env.oauth.*`) | Descrição |
|---|---|
| `baseUrl` | Base URL do realm no Keycloak |
| `clientId` | Client ID registrado no Keycloak |
| `clientSecret` | Opcional — para clientes confidenciais |
| `redirectUri` | Deve estar registrado como Valid Redirect URI no Keycloak |

## Relacionados

- [[02-infrastructure/external-apis/_index]] — Índice de external APIs
