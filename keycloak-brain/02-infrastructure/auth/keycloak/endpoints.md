---
title: Keycloak — Endpoints do Módulo Auth
category: infrastructure
service: keycloak
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/auth/keycloak/keycloak_index]]"
tags:
  - infrastructure
  - service
  - keycloak
  - endpoints
  - stable
---

# Keycloak — Endpoints do Módulo Auth

## Contexto

Rotas HTTP expostas pelo `AuthController` (`src/modules/auth/auth.controller.ts`). O controller está montado no prefixo configurado pelo módulo (sem prefixo próprio — usa o prefixo global da API, ex: `/api`).

## Conteúdo principal

### Rotas públicas (`@Public()`)

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/` | Home — HTML com links de login/logout |
| `GET` | `/login` | Inicia fluxo PKCE. Aceita query params (`returnTo`, etc.) que são preservados no `state` e devolvidos pós-login |
| `GET` | `/callback` | Recebe `code` + `state` do Keycloak. Valida state, troca código por tokens, seta cookies, redireciona para `frontendUrl` |
| `POST` | `/refresh` | Renova `access_token` usando `refresh_token` do cookie. Atualiza cookies |
| `POST` | `/logout` | Revoga `refresh_token` no Keycloak, limpa cookies de autenticação |

### Rotas autenticadas

| Método | Rota | Decorator | Descrição |
|---|---|---|---|
| `GET` | `/me` | — (guard padrão) | Retorna `{ user: User, isAuthenticated: true }` do access token |
| `GET` | `/permissions` | `@SessionToken()` | Retorna permissões calculadas a partir do token de sessão |

### Comportamento do callback

- Sucesso: redireciona `301` para `frontendUrl` com `returnParams` na query string
- Tokens ausentes: redireciona `301` para `frontendUrl?error=tokens_nao_obtidos`
- Erro genérico (catch): redireciona `301` para `frontendUrl` sem params

### Decorator `@SessionToken()`

Extrai o token do cookie `access_token` ou do header `Authorization: Bearer <token>`. Usado em `/permissions`.

## Relacionados

- [[02-infrastructure/auth/keycloak/keycloak_index]] — Índice do serviço Keycloak
