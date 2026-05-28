---
title: Cookies — Gerenciamento
category: infrastructure
service: cookies
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/auth/cookies/cookies_index]]"
tags:
  - infrastructure
  - service
  - stable
---

# Cookies — Gerenciamento

## Contexto

Métodos públicos do `CookieService` e seus comportamentos. Usa `@fastify/cookie` para serialização.

## Conteúdo principal

### Métodos de escrita

| Método | Cookie(s) setado(s) | MaxAge |
|---|---|---|
| `setPKCECookies(res, state, codeVerifier)` | `oauth_state`, `oauth_code_verifier` | 5 min (300s) |
| `setAccessToken(res, token)` | `access_token` | 15 min (900s) |
| `setRefreshToken(res, token)` | `refresh_token` | 30 dias (2592000s) |

### Métodos de leitura

| Método | Retorno |
|---|---|
| `getPKCECookies(req)` | `{ savedState, savedCodeVerifier }` (ambos `string \| undefined`) |
| `getAccessToken(req)` | `string \| undefined` — lê `req.cookies['access_token']` |
| `getRefreshToken(req)` | `string \| undefined` — lê `req.cookies['refresh_token']` |

### Métodos de limpeza

| Método | Cookie(s) removido(s) | Quando usar |
|---|---|---|
| `clearPKCECookies(res)` | `oauth_state`, `oauth_code_verifier` | Após callback bem-sucedido |
| `clearAuthCookies(res)` | `access_token`, `refresh_token` | Logout ou refresh falho |

### Opções de limpeza

As opções de limpeza (`getClearOptions`) **não** incluem `httpOnly` ou `sameSite` — apenas `path` e `domain`. Isso é obrigatório para que `res.clearCookie()` encontre e remova o cookie correto.

## Decisões / Soluções

O `secure` flag é derivado de `!env.development` para permitir cookies em HTTP durante desenvolvimento local, sem necessidade de configuração adicional.

## Relacionados

- [[02-infrastructure/auth/cookies/cookies_index]] — Índice do serviço de cookies
