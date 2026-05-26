---
title: PagBank — Endpoints Sandbox
category: infrastructure
bank: pagbank
status: draft
updated: 2026-05-26
parent: "[[pagbank_index]]"
tags:
  - infrastructure
  - bank
  - pagbank
  - endpoints
  - draft
---

# PagBank — Endpoints Sandbox

## Contexto

Endpoints do sandbox PagBank com exemplos completos de request e response.

## Conteúdo principal

### Base URL Sandbox

`https://sandbox.api.pagseguro.com`

### Autenticação

Header em todas as requisições:
```
Authorization: Bearer {apiKey}
```

---

### 1. Criar Order (COB)

**POST** `/orders`

**Request:**
```json
{
  "reference_id": "txid-abc123",
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com",
    "tax_id": "12345678909"
  },
  "items": [
    {
      "name": "Cobrança PIX",
      "quantity": 1,
      "unit_amount": 1000
    }
  ],
  "qr_codes": [
    {
      "amount": { "value": 1000 },
      "expiration_date": "2024-12-31T23:59:59-03:00"
    }
  ],
  "notification_urls": ["https://seu-dominio.com/pix/pagbank/webhook/pix"]
}
```

**Response:**
```json
{
  "id": "ORDER-ABC123",
  "reference_id": "txid-abc123",
  "status": "WAITING",
  "qr_codes": [
    {
      "id": "QRCO-ABC",
      "amount": { "value": 1000 },
      "expiration_date": "2024-12-31T23:59:59-03:00",
      "text": "00020101021226...",
      "links": [
        { "rel": "QRCODE.PNG", "href": "https://..." }
      ]
    }
  ]
}
```

---

### 2. Consultar Order

**GET** `/orders/{id}`

Response: mesmo formato da criação, com `status` atualizado.

---

### 3. Criar Cobrança com Vencimento (COBR)

Similar ao COB, mas com `due_date` no objeto `qr_codes`.

---

### 4. Listar Pagamentos da Order

**GET** `/orders/{id}/charges`

---

### 5. Devolver Pagamento

**POST** `/charges/{chargeId}/refunds`

**Request:**
```json
{
  "amount": { "value": 1000 }
}
```

---

### 6. Consultar Devolução

**GET** `/charges/{chargeId}/refunds/{refundId}`

---

### 7. Webhook — Notificação de Pagamento

**POST** para URL configurada em `notification_urls`.

**Payload:**
```json
{
  "id": "ORDR-ABC123",
  "reference_id": "txid-abc123",
  "type": "CHARGE_UPDATED",
  "charges": [
    {
      "id": "CHAR-ABC",
      "status": "PAID",
      "amount": { "value": 1000, "summary": { "total": 1000, "paid": 1000 } },
      "payment_method": {
        "type": "PIX",
        "pix": { "end_to_end_id": "E..." }
      }
    }
  ]
}
```

---

### 8. Consultar Chave PIX

**GET** `/pix/keys/{chave}`

---

### 9. Listar Recebimentos PIX

**GET** `/pix`

---

### 10. Consultar Recebimento PIX

**GET** `/pix/{e2eId}`

## Decisões / Soluções

O PagBank usa o modelo **Order** em vez do COB padrão do Banco Central. O adapter PagBank deve mapear o DTO interno ViaPIX → Order PagBank e normalizar o status `WAITING`/`PAID` → status interno ViaPIX.

## Relacionados

- [[pagbank_index]] — índice do PagBank
