---
title: PagBank — Índice
category: infrastructure
bank: pagbank
status: draft
updated: 2026-05-26
parent: "[[02-infrastructure/banks/_index]]"
tags:
  - infrastructure
  - bank
  - pagbank
  - draft
---

# PagBank

## Contexto

Integração planejada do ViaPIX com o PagBank. Usa Bearer token via Connect SMS. O PagBank tem modelo de cobrança próprio (Order) — diferente do COB padrão do Banco Central. Status: totalmente pendente.

## Conteúdo principal

### Identificação

| Campo | Valor |
|---|---|
| `bankCode` | `290` |
| Slug na rota | `pagbank` |
| Auth | Bearer token via Connect SMS |
| mTLS | Não exige |
| Inclusão no projeto | Fase 4 (pendente) |
| Status | ❌ não iniciado |

### Localização no código (planejado)

```
src/modules/pix/providers/pagbank/
  pagbank.provider.ts
  pagbank.adapter.ts
  pagbank.auth.ts
  pagbank.types.ts
  pagbank.webhook.ts
```

### Dependências

| Arquivo | Conteúdo |
|---|---|
| [[credentials]] | apiKey bearer, chavePix, apiUrl (dinâmica por cliente) |
| [[endpoints]] | Endpoints sandbox — Order, COBR, devolução, webhook |
| [[webhooks]] | Notificações de pagamento (planejado) |
| [[payment-flows]] | 10 fases de integração com checkpoints |
| [[errors]] | Erros previstos |

### Status por operação

| Operação | Status |
|---|---|
| Criar COB (Order) | ❌ |
| Consultar COB | ❌ |
| Criar COBR | ❌ |
| Consultar COBR | ❌ |
| Devolução PIX | ❌ |
| Webhook | ❌ |

## Relacionados

- [[02-infrastructure/banks/_index]] — índice de bancos
- [[00-index]] — índice geral do vault
