---
title: Banco do Brasil — Índice
category: infrastructure
bank: banco-do-brasil
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/banks/_index]]"
tags:
  - infrastructure
  - bank
  - banco-do-brasil
  - stable
---

# Banco do Brasil

## Contexto

Integração do ViaPIX com o Banco do Brasil. Usa OAuth2 `client_credentials` com mTLS obrigatório. Credenciais são dinâmicas — armazenadas por cliente no banco de dados.

## Conteúdo principal

### Identificação

| Campo | Valor |
|---|---|
| `bankCode` | `001` |
| Slug na rota | `bb` |
| Auth | OAuth2 `client_credentials` + mTLS obrigatório |
| mTLS | Certificado PEM por cliente (`certPem` + `certKeyPem`) |
| Inclusão no projeto | Fase 2 |
| Status | ✅ implementado |

### Localização no código

```
src/modules/pix/providers/banco-do-brasil/
  bb.provider.ts
  bb.adapter.ts
  bb.auth.ts
  bb.types.ts
  bb.webhook.ts
```

### Dependências

| Arquivo | Conteúdo |
|---|---|
| [[credentials]] | clientId, clientSecret, chavePix, certificado mTLS (geração + cadastro) |
| [[endpoints]] | OAuth2 token + rotas COB (criar, consultar, atualizar) |
| [[webhooks]] | Recebimento de notificações PIX + long polling |
| [[payment-flows]] | Fluxo de cobrança COB |
| [[errors]] | Erros da integração e dos webhooks |

### Status por operação

| Operação | Status |
|---|---|
| OAuth2 + mTLS | ✅ |
| Criar COB | ✅ |
| Consultar COB | ✅ |
| Atualizar COB | ✅ |
| Webhook COB | ✅ |
| Webhook COBR | ❌ pendente |
| Webhook REC | ❌ pendente |

## Relacionados

- [[02-infrastructure/banks/_index]] — índice de bancos
- [[00-index]] — índice geral do vault
