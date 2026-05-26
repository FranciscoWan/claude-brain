---
title: Banco do Brasil — Fluxos de Pagamento
category: infrastructure
bank: banco-do-brasil
status: stable
updated: 2026-05-26
parent: "[[banco-do-brasil_index]]"
tags:
  - infrastructure
  - bank
  - banco-do-brasil
  - payment-flows
  - stable
---

# Banco do Brasil — Fluxos de Pagamento

## Contexto

Fluxos de cobrança PIX suportados pelo Banco do Brasil no ViaPIX.

## Conteúdo principal

### Fluxo COB (cobrança imediata)

```
Cliente → POST /pix/bb/cob (ViaPIX)
  → ApiKeyGuard valida x-api-key
  → PixService chama ClientesService.findAndDecryptCredentials(cnpj, privateKey)
  → ClientesService descriptografa encrypted_blob → clientId, clientSecret, chavePix
  → ClientesService descriptografa encrypted_certify → certPem, certKeyPem
  → ProviderRegistry resolve bankCode=001 → BbProvider
  → BbAuthService obtém token OAuth2 (cache por clientId)
  → BbAdapter monta payload COB com txid gerado
  → POST {BB_PIX_URL}/v2/cob?gw-dev-app-key={BB_GW_APP_KEY}
  → Retorna CobResponseDto normalizado
```

### Payload de criação COB

```json
{
  "cnpj": "00.000.000/0001-00",
  "privateKey": "-----BEGIN RSA PRIVATE KEY-----...",
  "calendario": { "expiracao": 3600 },
  "valor": { "original": "10.00" },
  "chave": "sua-chave-pix",
  "solicitacaoPagador": "Pagamento teste"
}
```

### Geração de txid

O `txid` é gerado internamente com `crypto.randomBytes(16).toString('hex')` (32 chars) — compatível com BB e Itaú (exigência: 26–35 caracteres alfanuméricos).

### Fluxo COB + aguardar pagamento

Para aguardar o pagamento em uma única chamada, usar `POST /api/pix/cobrar` — ver [[endpoint-cobrar]] em features.

### Operações suportadas

| Operação | Rota ViaPIX | Status |
|---|---|---|
| Criar COB | `POST /pix/bb/cob` | ✅ |
| Consultar COB | `GET /pix/bb/cob/{txid}` | ✅ |
| Atualizar COB | `PATCH /pix/bb/cob/{txid}` | ✅ |

## Relacionados

- [[banco-do-brasil_index]] — índice do Banco do Brasil
