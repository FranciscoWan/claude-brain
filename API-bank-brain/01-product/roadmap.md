---
title: Roadmap
category: product
status: stable
updated: 2026-05-26
parent: "[[01-product/_index]]"
tags:
  - product
  - roadmap
  - stable
---

# Roadmap

## Contexto

Registro do que foi concluído e do que ainda está pendente no ViaPIX, organizado por fase e por componente.

## Conteúdo principal

### Fases principais

| Fase | Descrição | Status |
|---|---|---|
| 1 | Módulo clientes — cadastro + criptografia RSA+AES | ✅ |
| 2 | Integração Banco do Brasil — COB + OAuth2 + mTLS | ✅ |
| 3 | Integração Itaú — COB/COBV/PIX/Devolução | ✅ |
| 4 | Integração PagBank | ❌ pendente |
| 5 | Webhooks BB — COB | ✅ |
| 6 | Webhooks COBR / REC / Itaú | ❌ pendente |

### Status por componente

| Componente | Status |
|---|---|
| BB — credenciais dinâmicas, mTLS, webhooks COB | ✅ |
| Itaú — COB/COBV/PIX/Devolução | ✅ (env vars) |
| PagBank — COB/COBR/Devolução/Webhook | ❌ pendente |
| Módulo clientes — cadastro + criptografia RSA+AES | ✅ |
| `POST /api/pix/cobrar` | ✅ |
| Frontend Angular cadastro de clientes | ✅ |
| Auth API ViaPIX — `x-api-key` global | ✅ |
| Migração Itaú para credenciais dinâmicas (DB) | ❌ pendente |
| Webhooks COBR / REC / Itaú | ❌ pendente |

### Roadmap PagBank — 10 fases

| Fase | Descrição | Status |
|---|---|---|
| 1 | Cadastro de cliente PagBank no DB | ❌ |
| 2 | Auth — Bearer token via Connect SMS | ❌ |
| 3 | Criar cobrança COB | ❌ |
| 4 | Consultar COB | ❌ |
| 5 | Criar cobrança COBR | ❌ |
| 6 | Consultar COBR | ❌ |
| 7 | Devolução PIX | ❌ |
| 8 | Webhook recebimento | ❌ |
| 9 | Testes integração sandbox | ❌ |
| 10 | Homologação produção | ❌ |

O detalhamento das 10 fases do PagBank (checkpoints e checklists) está em `02-infrastructure/banks/pagbank/payment-flows.md`.

## Relacionados

- [[01-product/_index]] — índice de produto
