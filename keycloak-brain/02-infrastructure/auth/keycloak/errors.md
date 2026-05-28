---
title: Keycloak — Erros
category: infrastructure
service: keycloak
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/auth/keycloak/keycloak_index]]"
tags:
  - infrastructure
  - service
  - keycloak
  - errors
  - stable
---

# Keycloak — Erros

## Contexto

Mapeamento de todos os erros tratados no módulo de autenticação, organizados por origem.

## Conteúdo principal

### Erros do `KeycloakClient`

| Operação | Condição | Mensagem lançada |
|---|---|---|
| `exchangeCodeForToken` | Resposta não-OK do Keycloak | `'Falha ao obter tokens'` |
| `refreshToken` | Resposta não-OK | `'Falha ao renovar tokens'` |
| `revokeToken` | Resposta não-OK | `'Falha ao revogar token: <detalhe>'` |
| `exchangeToken` | HTTP 400 + `error: invalid_token` | Error com `status: 401`, msg: `'Token expirado ou inválido para o serviço <audience>'` |
| `exchangeToken` | HTTP 403 ou 400 genérico | `'Você não tem permissão para acessar o serviço <audience>...'` |
| `exchangeToken` | Outros erros | `'Falha ao realizar token exchange: <detalhe>'` |
| `decodeToken` | Token malformado (sem `.`) | `'Falha ao decodificar token JWT'` |

O corpo do erro do Keycloak (`error_description`, `error`) é sempre preferido sobre `statusText` quando disponível.

### Erros do `PkceService`

| Condição | Tipo | Mensagem |
|---|---|---|
| `state` recebido ≠ `state` salvo | `UnauthorizedException` | `'State inválido - possível ataque CSRF'` |
| `codeVerifier` ausente no cookie | `UnauthorizedException` | `'Code verifier não encontrado'` |

### Erros do `RefreshTokenService`

| Condição | Tipo | Mensagem |
|---|---|---|
| `refreshToken` ausente na entrada | `UnauthorizedException` | `'Refresh token não encontrado'` |
| Erro contém `'not active'` | `UnauthorizedException` | `'Sessão expirada. Por favor, faça login novamente.'` |
| Outros erros de refresh | `UnauthorizedException` | `'Falha ao renovar token. Por favor, faça login novamente.'` |

### Erros do `AuthorizationService`

| Condição | Tipo | Mensagem |
|---|---|---|
| `accessToken` ausente em `getUserInfo` | `UnauthorizedException` | `'Access token não encontrado'` |
| `token` ausente em `getPermissions` | `UnauthorizedException` | `'Token de autenticação não encontrado'` |

### Comportamento do `JwtMiddleware` em erro

O middleware captura exceções via `try/catch` e **não** lança — apenas loga o erro e chama `next()`. A proteção real acontece nos guards downstream.

## Relacionados

- [[02-infrastructure/auth/keycloak/keycloak_index]] — Índice do serviço Keycloak
