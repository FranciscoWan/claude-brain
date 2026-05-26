---
title: Banco do Brasil — Webhooks
category: infrastructure
bank: banco-do-brasil
status: stable
updated: 2026-05-26
parent: "[[banco-do-brasil_index]]"
tags:
  - infrastructure
  - bank
  - banco-do-brasil
  - webhooks
  - stable
---

# Banco do Brasil — Webhooks

## Contexto

Mecanismo de recebimento de notificações de pagamento PIX enviadas pelo BB ao ViaPIX, com long polling para que o cliente aguarde a confirmação.

## Conteúdo principal

### Fluxo completo

```
BB detecta pagamento PIX
  → POST /pix/bb/webhook/pix (ViaPIX)
  → WebhookService salva em pix_notifications
  → Cliente fazendo poll em GET /pix/notifications/{txid}/poll recebe resposta
```

### Endpoint de recebimento

**POST** `/pix/bb/webhook/pix`

- Marcado `@Public()` — não exige `x-api-key`
- Chamado diretamente pelo BB ao detectar pagamento
- Payload normalizado e salvo em `pix_notifications`

**Payload recebido do BB:**
```json
{
  "pix": [
    {
      "endToEndId": "E00000000...",
      "txid": "abc123",
      "valor": "10.00",
      "horario": "2024-01-01T12:00:00.000Z",
      "infoPagador": "Pagamento teste"
    }
  ]
}
```

### Long polling

**GET** `/pix/notifications/{txid}/poll`

**Headers:** `x-cnpj` + `x-private-key`

**Query params:**

| Param | Padrão | Máximo |
|---|---|---|
| `timeout` | `30` (segundos) | `55` |

**Comportamento:**
1. Verifica `pix_notifications` no DB a cada 2 segundos
2. Retorna assim que encontrar notificação para o `txid`
3. Retorna `{ received: false }` ao atingir o timeout

**Response — pagamento recebido:**
```json
{
  "received": true,
  "txid": "abc123",
  "endToEndId": "E00000000...",
  "valor": "10.00",
  "horario": "2024-01-01T12:00:00.000Z"
}
```

### Teste manual

```http
### Simular webhook do BB
POST http://localhost:3006/pix/bb/webhook/pix
Content-Type: application/json

{
  "pix": [
    {
      "endToEndId": "E00000000202401011200000000001",
      "txid": "seu-txid-aqui",
      "valor": "10.00",
      "horario": "2024-01-01T12:00:00.000Z"
    }
  ]
}
```

### Status de implementação

| Evento | Status |
|---|---|
| COB (cobrança imediata) | ✅ implementado |
| COBR (cobrança com vencimento) | ❌ pendente |
| REC (recebimento avulso) | ❌ pendente |

## Relacionados

- [[banco-do-brasil_index]] — índice do Banco do Brasil

