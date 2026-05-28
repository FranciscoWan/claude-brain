---
title: Feature — Refresh de Token
category: product
status: stable
updated: 2026-05-28
parent: "[[01-product/features/_index]]"
tags:
  - product
  - feature
  - stable
---

# Feature — Refresh de Token

## Contexto

O módulo suporta dois modos de renovação de token: automático (transparente, via middleware) e manual (via endpoint explícito). Ambos usam o `refresh_token` armazenado em cookie httpOnly.

## Conteúdo principal

### Refresh automático (JwtMiddleware)

O `JwtMiddleware` intercepta todas as requisições e:

| Situação | Comportamento |
|---|---|
| `access_token` ausente + `refresh_token` presente | Renova automaticamente antes de continuar |
| `access_token` expira em ≤ 120 segundos | Renova proativamente durante a requisição |
| Refresh falha | Limpa ambos os cookies, continua sem token (rotas públicas passam, rotas protegidas falham no guard) |

O token renovado é injetado em `req.cookies['access_token']` para que guards downstream usem o token atualizado na mesma requisição.

### Refresh manual (endpoint)

`POST /refresh` — aceita o `refresh_token` do cookie e devolve novos tokens, atualizando os cookies. É uma rota `@Public()` (não requer autenticação prévia).

Erros tratados:
- Refresh token não encontrado → `UnauthorizedException`
- Token `not active` (sessão expirada no Keycloak) → `UnauthorizedException('Sessão expirada...')`
- Outros erros → `UnauthorizedException('Falha ao renovar token...')`

## Relacionados

- [[01-product/features/_index]] — Índice de features
