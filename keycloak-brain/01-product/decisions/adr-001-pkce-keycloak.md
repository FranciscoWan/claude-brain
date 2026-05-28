---
title: ADR-001 — PKCE com Keycloak e Cookies httpOnly
category: product
status: stable
updated: 2026-05-28
parent: "[[01-product/_index]]"
tags:
  - product
  - decision
  - stable
---

# ADR-001 — PKCE com Keycloak e Cookies httpOnly

## Contexto

O projeto precisava de um mecanismo de autenticação seguro para uma arquitetura com frontend separado (SPA). As alternativas avaliadas foram: PKCE com cookies httpOnly, PKCE com tokens no localStorage, e autenticação via session server-side.

## Conteúdo principal

### Decisão

Usar OAuth2 Authorization Code Flow com **PKCE** (S256), tokens armazenados em **cookies httpOnly** (não acessíveis via JavaScript), e refresh automático transparente no middleware.

### Justificativas

| Critério | Escolha | Motivo |
|---|---|---|
| Proteção contra XSS | Cookies httpOnly | Tokens inacessíveis ao JavaScript |
| Proteção CSRF | PKCE + state com campo `csrf` aleatório | State validado no callback |
| Clientes públicos | PKCE sem client_secret obrigatório | Suporte a `clientSecret` opcional via `appendClientSecret()` |
| Refresh transparente | JwtMiddleware com threshold de 120s | UX sem interrupção de sessão |
| Propagação de contexto | CLS (Context Local Storage) | Dados do usuário disponíveis em serviços downstream sem passar por parâmetro |

### Consequências

- Cookies exigem `sameSite: lax` e `domain` configurado via `EnvService`
- Em produção, `secure: true` é forçado automaticamente (`!env.development`)
- O `state` carrega `returnParams` (query params do frontend) para restaurar a navegação pós-login
- A decodificação do JWT é feita **sem verificação de assinatura** (base64 decode manual) — a validação de autenticidade fica por conta do `KeycloakAuthGuard` downstream

## Decisões / Soluções

A ausência de verificação de assinatura no `decodeToken()` é intencional para evitar dependência de chave pública em todos os serviços — o guard centraliza essa responsabilidade.

## Relacionados

- [[01-product/_index]] — Índice de produto
