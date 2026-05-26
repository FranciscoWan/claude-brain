---
title: PagBank — Webhooks
category: infrastructure
bank: pagbank
status: draft
updated: 2026-05-26
parent: "[[pagbank_index]]"
tags:
  - infrastructure
  - bank
  - pagbank
  - webhooks
  - draft
---

# PagBank — Webhooks

## Contexto

Recebimento de notificações de pagamento do PagBank. Pendente de implementação (Fase 8 do roadmap PagBank).

## Conteúdo principal

### Status

| Evento | Status |
|---|---|
| Webhook pagamento | ❌ pendente |

### Planejamento (Fase 8)

**Checkpoint:** PagBank consegue entregar notificação para `POST /pix/pagbank/webhook/pix`.

- [ ] Criar `pagbank.webhook.ts`
- [ ] Registrar rota no `WebhookController` marcada `@Public()`
- [ ] Normalizar payload PagBank → formato interno ViaPIX
- [ ] Salvar em `pix_notifications` com `banco = 'pagbank'`

### Payload esperado do PagBank

```json
{
  "id": "ORDR-ABC123",
  "reference_id": "txid-abc123",
  "type": "CHARGE_UPDATED",
  "charges": [
    {
      "id": "CHAR-ABC",
      "status": "PAID",
      "amount": { "value": 1000 },
      "payment_method": {
        "type": "PIX",
        "pix": { "end_to_end_id": "E..." }
      }
    }
  ]
}
```

O campo `reference_id` mapeia para `txid` interno. O `end_to_end_id` mapeia para `endToEndId`.

### Configuração no cadastro da Order

A URL do webhook deve ser enviada ao criar a Order:
```json
"notification_urls": ["https://seu-dominio.com/pix/pagbank/webhook/pix"]
```

## Relacionados

- [[pagbank_index]] — índice do PagBank
