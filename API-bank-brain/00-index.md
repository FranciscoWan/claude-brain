---
title: ViaPIX — Índice Geral
category: product
status: stable
updated: 2026-05-26
parent: ""
tags:
  - product
  - infrastructure
  - stable
---

# ViaPIX

API intermediária que expõe endpoints PIX padronizados e roteia operações para o banco correto de forma transparente. O cliente não precisa conhecer as particularidades de cada banco.

## Contexto

O ViaPIX recebe requisições com `cnpj + privateKey + payload`, descriptografa as credenciais bancárias armazenadas no banco de dados e delega a operação ao provider do banco correto. A resposta é normalizada antes de retornar ao cliente.

```
Cliente (cnpj + privateKey + payload)
  → ViaPIX descriptografa credenciais (RSA+AES)
  → roteia para o banco correto (BB, Itaú, PagBank)
  → retorna resultado padronizado
```

**Repositório:**
```
viapix_be/    ← backend NestJS (porta 3006)
viapix_fe/    ← frontend Angular 21 + Tailwind v4
brain/        ← este vault
```

## Status atual

| Componente | Status |
|---|---|
| BB — credenciais dinâmicas, mTLS, webhooks COB | ✅ |
| Itaú — COB/COBV/PIX/Devolução | ✅ (env vars) |
| PagBank — COB/COBR/Devolução/Webhook | ❌ pendente |
| Módulo clientes — cadastro + criptografia RSA+AES | ✅ |
| `POST /api/pix/cobrar` | ✅ |
| Frontend Angular cadastro de clientes | ✅ |
| Auth API ViaPIX — `x-api-key` global | ✅ |
| Migração Itaú para credenciais dinâmicas (DB) | ❌ pendente |
| Webhooks COBR / REC / Itaú | ❌ pendente |

## Seções do vault

- [[01-product/_index]] — roadmap, features e decisões arquiteturais
- [[02-infrastructure/_index]] — arquitetura, bancos, banco de dados, segurança

## Relacionados

- [[01-product/_index]] — produto, roadmap e decisões
- [[02-infrastructure/_index]] — infraestrutura e integrações bancárias
