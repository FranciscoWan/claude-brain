---
title: Keycloak — Token Exchange
category: infrastructure
service: keycloak
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/auth/keycloak/keycloak_index]]"
tags:
  - infrastructure
  - service
  - keycloak
  - stable
---

# Keycloak — Token Exchange

## Contexto

Implementa o RFC 8693 (OAuth 2.0 Token Exchange) para troca de tokens entre serviços (service-to-service). Permite que um token do usuário seja trocado por um token com audience de outro serviço interno.

## Conteúdo principal

### `KeycloakClient.exchangeToken(subjectToken, audience)`

Parâmetros enviados ao endpoint `/token`:

| Parâmetro | Valor |
|---|---|
| `grant_type` | `urn:ietf:params:oauth:grant-type:token-exchange` |
| `client_id` | `env.oauth.clientId` |
| `subject_token` | token de acesso do usuário |
| `subject_token_type` | `urn:ietf:params:oauth:token-type:access_token` |
| `requested_token_type` | `urn:ietf:params:oauth:token-type:access_token` |
| `audience` | identificador do serviço alvo |

Retorna o `access_token` já trocado (string).

### Logs de diagnóstico

A operação loga o token **antes** e **depois** da troca, incluindo: `sub`, `aud`, `resource_access`, `realm_access`. Isso facilita debugging de problemas de permissão no Keycloak.

### Operações de refresh e revogação

| Método | Endpoint | Grant Type |
|---|---|---|
| `refreshToken(refreshToken)` | `/token` | `refresh_token` |
| `revokeToken(refreshToken)` | `/logout` | — (POST direto) |
| `exchangeCodeForToken(code, verifier)` | `/token` | `authorization_code` |

### Inspeção de token (sem chamada ao servidor)

| Método | Descrição |
|---|---|
| `decodeToken(token)` | Decodifica payload JWT via base64 — **sem verificação de assinatura** |
| `isTokenValid(token)` | Verifica se `exp > now` |
| `hasAudience(token, aud)` | Verifica se `aud` contém o valor requerido (suporta string ou array) |
| `hasResourceAccess(token, clientId, roles?)` | Verifica `resource_access[clientId].roles` |

> [!warning]
> `decodeToken()` não verifica a assinatura do JWT. A validação de autenticidade é responsabilidade do `KeycloakAuthGuard`.

## Relacionados

- [[02-infrastructure/auth/keycloak/keycloak_index]] — Índice do serviço Keycloak
