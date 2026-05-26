---
title: Itaú — Fluxos de Pagamento
category: infrastructure
bank: itau
status: stable
updated: 2026-05-26
parent: "[[itau_index]]"
tags:
  - infrastructure
  - itau
  - payment-flows
  - stable
  - bank
---

# Itaú — Fluxos de Pagamento

## Contexto

Fluxos de pagamento PIX suportados pelo Itaú no ViaPIX: COB, COBV, PIX e Devolução.

## Conteúdo principal

### Operações suportadas

| Operação | Rota ViaPIX | Status |
|---|---|---|
| Criar COB | `POST /pix/itau/cob` | ✅ |
| Consultar COB | `GET /pix/itau/cob/{txid}` | ✅ |
| Atualizar COB | `PATCH /pix/itau/cob/{txid}` | ✅ |
| Criar COBV | `POST /pix/itau/cobv` | ✅ |
| Criar PIX | `POST /pix/itau/pix` | ✅ |
| Devolução PIX | `POST /pix/itau/pix/{e2eId}/devolucao/{id}` | ✅ |

### Fluxo COB

```
Cliente → POST /pix/itau/cob (ViaPIX)
  → ApiKeyGuard valida x-api-key
  → PixService chama ClientesService (busca credenciais via env vars atualmente)
  → ProviderRegistry resolve bankCode=ITAU → ItauProvider
  → ItauAuthService obtém token OAuth2 (cache por clientId)
  → ItauAdapter monta payload COB com txid gerado
  → POST {ITAU_PIX_URL}/v2/cob
  → Retorna CobResponseDto normalizado
```

### Geração de txid

`crypto.randomBytes(16).toString('hex')` — 32 chars, dentro da faixa exigida pelo Itaú (26–35 caracteres alfanuméricos).

### Payload COB

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

O adapter Itaú mapeia este DTO interno para o formato esperado pela API do Itaú.

## Relacionados

- [[itau_index]] — índice do Itaú
