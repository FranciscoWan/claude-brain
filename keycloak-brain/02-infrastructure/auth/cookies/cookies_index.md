---
title: Cookies — Visão Geral
category: infrastructure
service: cookies
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/auth/_index]]"
tags:
  - infrastructure
  - service
  - stable
---

# Cookies — Visão Geral

## Contexto

O `CookieService` (`src/modules/auth/services/cookie.service.ts`) centraliza toda a leitura e escrita de cookies de autenticação. Nenhum outro serviço acessa cookies diretamente.

**Data de inclusão no projeto:** desconhecida
**Status atual:** stable

## Conteúdo principal

### Dependências

- [[02-infrastructure/auth/cookies/cookie-management|Cookie Management]] — cookies e seus parâmetros detalhados

### Cookies gerenciados

| Cookie | Finalidade | MaxAge |
|---|---|---|
| `oauth_state` | Armazena o `state` PKCE temporariamente | 5 min |
| `oauth_code_verifier` | Armazena o `codeVerifier` PKCE temporariamente | 5 min |
| `access_token` | Token de acesso do usuário | 15 min |
| `refresh_token` | Token de renovação de sessão | 30 dias |

### Opções base (todos os cookies)

```ts
{
  httpOnly: true,
  secure: !env.development,   // true em produção
  sameSite: 'lax',
  path: '/',
  domain: env.oauth.cookieDomain,
}
```

## Relacionados

- [[02-infrastructure/auth/_index]] — Índice de auth services
