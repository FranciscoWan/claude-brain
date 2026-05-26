---
title: Itaú — Credenciais
category: infrastructure
bank: itau
status: stable
updated: 2026-05-26
parent: "[[itau_index]]"
tags:
  - infrastructure
  - itau
  - credentials
  - stable
  - bank
---

# Itaú — Credenciais

## Contexto

O Itaú usa OAuth2 sem mTLS. As credenciais estão temporariamente em variáveis de ambiente — a migração para credenciais dinâmicas por cliente no banco de dados está pendente.

## Conteúdo principal

### Credenciais necessárias

| Campo | Origem |
|---|---|
| `clientId` | Portal developer.itau.com.br |
| `clientSecret` | Portal developer.itau.com.br |
| `chavePix` | Conta Itaú do cliente |

### Situação atual — env vars

As credenciais do Itaú são lidas das variáveis de ambiente:

| Variável | Descrição |
|---|---|
| `ITAU_CLIENT_ID` | Client ID Itaú |
| `ITAU_CLIENT_SECRET` | Client Secret Itaú |
| `ITAU_CHAVE_PIX` | Chave PIX Itaú |
| `ITAU_PIX_URL` | URL base da API PIX Itaú |
| `ITAU_OAUTH_URL` | URL do OAuth2 Itaú (STS APIGateway) |

### Passos para obter credenciais

1. Criar aplicação no portal developer.itau.com.br
2. Obter `clientId` e `clientSecret`
3. Obter `chavePix` na conta Itaú

### Cadastro futuro no ViaPIX (após migração)

```json
{
  "cnpj": "00.000.000/0001-00",
  "bankCode": "ITAU",
  "clientId": "...",
  "clientSecret": "...",
  "chavePix": "..."
}
```

## Decisões / Soluções

A migração para credenciais dinâmicas (DB) segue o mesmo padrão do BB — armazenar `clientId`, `clientSecret` e `chavePix` em `encrypted_blob` por cliente. Enquanto não é feita, o Itaú funciona com credenciais de um único cliente via env vars.

O `ProviderRegistry` registra o Itaú em dois aliases para garantir compatibilidade:
```typescript
this.providers.set('ITAU', itauProvider);
this.providers.set('341', itauProvider);
```

## Relacionados

- [[itau_index]] — índice do Itaú
