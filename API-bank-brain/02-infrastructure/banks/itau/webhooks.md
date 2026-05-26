---
title: Itaú — Webhooks
category: infrastructure
bank: itau
status: draft
updated: 2026-05-26
parent: "[[itau_index]]"
tags:
  - infrastructure
  - bank
  - itau
  - webhooks
  - draft
---

# Itaú — Webhooks

## Contexto

Recebimento de notificações PIX do Itaú via webhook. Pendente de implementação.

## Conteúdo principal

### Status

| Evento | Status |
|---|---|
| Webhook COB | ❌ pendente |
| Webhook COBV | ❌ pendente |

### Planejamento

O padrão de implementação deve seguir o mesmo modelo do BB:

1. Criar rota `POST /pix/itau/webhook/pix` marcada `@Public()`
2. Criar `itau.webhook.ts` para normalizar o payload do Itaú
3. Salvar notificação em `pix_notifications` com `banco = 'itau'`
4. Reutilizar o long polling existente em `GET /pix/notifications/{txid}/poll`

## Relacionados

- [[itau_index]] — índice do Itaú
