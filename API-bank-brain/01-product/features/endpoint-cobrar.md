---
title: Feature — Endpoint /api/pix/cobrar
category: product
status: stable
updated: 2026-05-26
parent: "[[01-product/features/_index]]"
tags:
  - product
  - feature
  - stable
  - endpoints
---

# Endpoint `/api/pix/cobrar`

## Contexto

Atalho que combina a criação de uma cobrança COB com a espera pelo pagamento via long polling em uma única chamada HTTP. Elimina a necessidade do cliente orquestrar dois endpoints separados.

## Conteúdo principal

### Rota

**POST** `/api/pix/cobrar`

Requer header `x-api-key`.

### Payload

```json
{
  "cnpj": "00.000.000/0001-00",
  "privateKey": "-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEA...\n-----END RSA PRIVATE KEY-----",
  "bankCode": "001",
  "valor": "10.00",
  "chave": "sua-chave-pix@email.com",
  "solicitacaoPagador": "Pagamento de teste",
  "expiracao": 3600,
  "timeout": 30
}
```

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `cnpj` | string | ✅ | CNPJ do cliente |
| `privateKey` | string (PEM) | ✅ | Chave RSA privada do cliente |
| `bankCode` | string | ✅ | Código do banco (`001`, `ITAU`, etc.) |
| `valor` | string | ✅ | Valor em reais (ex: `"10.00"`) |
| `chave` | string | ✅ | Chave PIX de destino |
| `solicitacaoPagador` | string | — | Texto exibido ao pagador |
| `expiracao` | number | — | Segundos até expirar (padrão: `3600`) |
| `timeout` | number | — | Segundos de poll (padrão: `30`, máximo: `55`) |

### Observações sobre o payload

- A `privateKey` deve ser enviada com `\n` reais (não literais). Ao ler do `.env`: `process.env.CLIENT_PRIVATE_KEY.replace(/\\n/g, '\n')`
- O `valor` é string decimal com duas casas: `"10.00"`, nunca `10`
- O `timeout` define por quanto tempo o ViaPIX aguarda o pagamento antes de retornar `{ received: false }`

### Resposta — pagamento recebido

```json
{
  "received": true,
  "txid": "abc123",
  "endToEndId": "E00000000...",
  "valor": "10.00",
  "horario": "2024-01-01T12:00:00.000Z",
  "qrCode": "00020101..."
}
```

### Resposta — timeout sem pagamento

```json
{
  "received": false,
  "txid": "abc123",
  "qrCode": "00020101..."
}
```

### Fluxo interno

```
POST /api/pix/cobrar
  → cria COB no banco (POST /pix/{bank}/cob)
  → inicia long polling (GET /pix/notifications/{txid}/poll)
  → retorna resultado com qrCode + status de pagamento
```

### Exemplo de chamada (via viapix_be_cliente)

**Headers:**
```
Content-Type: application/json
x-api-key: {VIAPIX_API_KEY}
```

**Variáveis de ambiente necessárias no cliente:**

| Variável | Descrição |
|---|---|
| `VIAPIX_URL` | URL base do ViaPIX (ex: `http://localhost:3006`) |
| `VIAPIX_API_KEY` | Chave `x-api-key` para autenticar no ViaPIX |
| `CLIENT_CNPJ` | CNPJ do cliente cadastrado no ViaPIX |
| `CLIENT_PRIVATE_KEY` | Chave RSA privada do cliente |
| `CLIENT_BANK_CODE` | Código do banco (`001`, `ITAU`, etc.) |

## Relacionados

- [[01-product/features/_index]] — catálogo de features
