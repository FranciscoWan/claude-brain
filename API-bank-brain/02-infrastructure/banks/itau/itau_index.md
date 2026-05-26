---
title: Itaú — Índice
category: infrastructure
bank: itau
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/banks/_index]]"
tags:
  - infrastructure
  - bank
  - itau
  - stable
---

# Itaú

## Contexto

Integração do ViaPIX com o Itaú. Usa OAuth2 `client_credentials` via APIGateway STS. Não exige mTLS. Credenciais atualmente em env vars — migração para banco de dados pendente.

## Conteúdo principal

### Identificação

| Campo | Valor |
|---|---|
| `bankCode` | `ITAU` (normalizado para `341` internamente) |
| Slug na rota | `itau` |
| Auth | OAuth2 `client_credentials` via APIGateway STS |
| mTLS | Não exige |
| Inclusão no projeto | Fase 3 |
| Status | ✅ implementado (env vars) |

### Localização no código

```
src/modules/pix/providers/itau/
  itau.provider.ts
  itau.adapter.ts
  itau.auth.ts
  itau.types.ts
```

### Dependências

| Arquivo | Conteúdo |
|---|---|
| [[credentials]] | clientId, clientSecret, chavePix (env vars + migração pendente) |
| [[endpoints]] | OAuth2 token + rotas COB/COBV/PIX/Devolução |
| [[webhooks]] | Pendente |
| [[payment-flows]] | Fluxos COB, COBV, PIX e Devolução |
| [[errors]] | Erros da integração |

### Status por operação

| Operação | Status |
|---|---|
| OAuth2 | ✅ |
| Criar COB | ✅ |
| Consultar COB | ✅ |
| Atualizar COB | ✅ |
| Criar COBV | ✅ |
| Criar PIX | ✅ |
| Devolução PIX | ✅ |
| Webhook | ❌ pendente |
| Credenciais dinâmicas (DB) | ❌ pendente |

## Relacionados

- [[02-infrastructure/banks/_index]] — índice de bancos
- [[00-index]] — índice geral do vault
