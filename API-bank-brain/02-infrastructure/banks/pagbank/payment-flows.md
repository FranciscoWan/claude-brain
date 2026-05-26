---
title: PagBank — Fluxos de Pagamento e Roadmap de Integração
category: infrastructure
bank: pagbank
status: draft
updated: 2026-05-26
parent: "[[pagbank_index]]"
tags:
  - infrastructure
  - bank
  - pagbank
  - payment-flows
  - draft
---

# PagBank — Fluxos de Pagamento e Roadmap de Integração

## Contexto

O PagBank usa o modelo **Order** (não o COB padrão do Banco Central). Esta página documenta o modelo de dados e as 10 fases de integração com checkpoints e checklists.

## Conteúdo principal

### Modelo Order

```json
{
  "reference_id": "txid-gerado",
  "customer": { "name": "...", "email": "...", "tax_id": "..." },
  "items": [{ "name": "Cobrança PIX", "quantity": 1, "unit_amount": 1000 }],
  "qr_codes": [{ "amount": { "value": 1000 }, "expiration_date": "..." }]
}
```

O adapter PagBank deve mapear o DTO interno ViaPIX → Order PagBank e normalizar os status de volta.

---

### Fase 1 — Cadastro de cliente PagBank no DB

**Checkpoint:** `POST /clientes` aceita `apiKey` + `apiUrl` + `chavePix` para PagBank e armazena criptografado.

- [ ] Atualizar `CreateClienteDto` para aceitar `apiKey` e `apiUrl`
- [ ] Atualizar `CryptoService` para incluir `apiKey` e `apiUrl` no `encrypted_blob`
- [ ] Testar cadastro e consulta

### Fase 2 — Auth — Bearer token

**Checkpoint:** `PagBankAuthService` retorna token Bearer válido usando `apiKey` do cliente.

- [ ] Criar `pagbank.auth.ts`
- [ ] Implementar cache de token por `clientId`

### Fase 3 — Criar cobrança COB

**Checkpoint:** `POST /pix/pagbank/cob` cria Order no PagBank e retorna `txid` + QR code.

- [ ] Criar `pagbank.provider.ts` com `createCob`
- [ ] Criar `pagbank.adapter.ts` — mapear DTO interno → Order PagBank
- [ ] Registrar no `ProviderRegistry` com `bankCode='290'`

### Fase 4 — Consultar COB

**Checkpoint:** `GET /pix/pagbank/cob/{txid}` retorna status da Order.

- [ ] Implementar `getCob` no provider
- [ ] Mapear status PagBank → status ViaPIX

### Fase 5 — Criar cobrança COBR

**Checkpoint:** `POST /pix/pagbank/cobr` cria cobrança com vencimento.

- [ ] Implementar `createCobv` no provider (mapeado como COBR)

### Fase 6 — Consultar COBR

**Checkpoint:** `GET /pix/pagbank/cobr/{txid}` retorna status.

- [ ] Implementar `getCobv` no provider

### Fase 7 — Devolução PIX

**Checkpoint:** `POST /pix/pagbank/pix/{e2eId}/devolucao/{id}` devolve valor.

- [ ] Implementar `createDevolucao` no provider

### Fase 8 — Webhook

**Checkpoint:** PagBank consegue entregar notificação para `POST /pix/pagbank/webhook/pix`.

- [ ] Criar `pagbank.webhook.ts`
- [ ] Registrar rota no `WebhookController`
- [ ] Salvar notificação em `pix_notifications`

### Fase 9 — Testes integração sandbox

**Checkpoint:** Fluxo completo funciona no sandbox PagBank.

- [ ] Criar arquivos `.http` de teste
- [ ] Validar todos os cenários

### Fase 10 — Homologação produção

**Checkpoint:** Integração aprovada em ambiente de produção PagBank.

- [ ] Configurar URLs de produção
- [ ] Validar certificados e tokens

## Relacionados

- [[pagbank_index]] — índice do PagBank
