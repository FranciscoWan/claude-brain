---
title: Bancos — Índice
category: infrastructure
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/_index]]"
tags:
  - infrastructure
  - bank
  - stable
---

# Bancos

## Contexto

Hub central de todas as integrações bancárias do ViaPIX. Cada banco tem seu próprio provider, adapter, auth e documentação.

## Conteúdo principal

### Bancos integrados

| Banco | bankCode | Slug na rota | Status |
|---|---|---|---|
| Banco do Brasil | `001` | `bb` | ✅ OAuth2 + mTLS + Webhooks COB |
| Itaú | `ITAU` → `341` | `itau` | ✅ COB/COBV/PIX/Devolução |
| PagBank | `290` | `pagbank` | ❌ pendente |

### Arquivos por banco

**Banco do Brasil:**
- [[banco-do-brasil_index]] — visão geral, status e índice de dependências
- `credentials.md` — clientId, clientSecret, chavePix, certificado mTLS
- `endpoints.md` — OAuth2 + rotas PIX
- `webhooks.md` — recebimento de notificações + long polling
- `payment-flows.md` — fluxo de cobrança COB
- `errors.md` — erros da integração e dos webhooks

**Itaú:**
- [[itau_index]] — visão geral, status e índice de dependências
- `credentials.md` — clientId, clientSecret, chavePix
- `endpoints.md` — OAuth2 + rotas COB/COBV/PIX/Devolução
- `webhooks.md` — pendente
- `payment-flows.md` — fluxos de pagamento
- `errors.md` — erros da integração

**PagBank:**
- [[pagbank_index]] — visão geral, status e índice de dependências
- `credentials.md` — apiKey bearer, chavePix, apiUrl
- `endpoints.md` — endpoints sandbox
- `webhooks.md` — planejado
- `payment-flows.md` — 10 fases de integração
- `errors.md` — erros previstos

## Relacionados

- [[02-infrastructure/_index]] — índice de infraestrutura
- [[00-index]] — índice geral do vault
