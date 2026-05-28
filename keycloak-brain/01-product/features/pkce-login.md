---
title: Feature — Login PKCE
category: product
status: stable
updated: 2026-05-28
parent: "[[01-product/features/_index]]"
tags:
  - product
  - feature
  - stable
---

# Feature — Login PKCE

## Contexto

O módulo de login usa o fluxo OAuth2 Authorization Code com PKCE (Proof Key for Code Exchange). O usuário é redirecionado ao Keycloak, autentica, e retorna com um authorization code que é trocado por tokens armazenados em cookies httpOnly.

## Conteúdo principal

### Fluxo do usuário

1. Usuário acessa `GET /login` (com query params opcionais, ex: `?returnTo=/dashboard`)
2. Backend gera `state` (CSRF + returnParams codificado em base64url) e `codeVerifier` (32 bytes aleatórios)
3. Backend redireciona para Keycloak com `code_challenge` (SHA-256 do verifier)
4. Usuário autentica no Keycloak
5. Keycloak redireciona para `GET /callback?code=...&state=...`
6. Backend valida o `state`, troca o `code` + `codeVerifier` por tokens
7. `access_token` (maxAge: 15 min) e `refresh_token` (maxAge: 30 dias) são setados em cookies httpOnly
8. Usuário é redirecionado para o frontend com os `returnParams` na query string

### Proteção CSRF

O campo `csrf` dentro do `state` é gerado com `crypto.randomBytes(16)`. O backend rejeita qualquer callback onde `state` recebido ≠ `state` salvo em cookie, lançando `UnauthorizedException('State inválido - possível ataque CSRF')`.

### Endpoints envolvidos

- `GET /login` — inicia o fluxo
- `GET /callback` — recebe o retorno do Keycloak

## Relacionados

- [[01-product/features/_index]] — Índice de features
