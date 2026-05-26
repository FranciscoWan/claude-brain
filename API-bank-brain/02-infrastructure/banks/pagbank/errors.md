---
title: PagBank — Erros e Soluções
category: infrastructure
bank: pagbank
status: draft
updated: 2026-05-26
parent: "[[pagbank_index]]"
tags:
  - infrastructure
  - bank
  - pagbank
  - errors
  - draft
---

# PagBank — Erros e Soluções

## Contexto

Erros encontrados ou previstos na integração com o PagBank e suas soluções.

## Conteúdo principal

#### `TS2339` — Property `apiUrl` does not exist on type `ClienteCredentials`

**Contexto:** TypeScript rejeita acesso a `credentials.apiUrl` no provider PagBank.

**Causa:** O campo `apiUrl` não estava declarado na interface `ClienteCredentials`.

**Solução:** Adicionar `apiUrl?: string` à interface `ClienteCredentials` em `pix-provider.interface.ts`:

```typescript
export interface ClienteCredentials {
  // ... campos existentes ...
  apiUrl?: string;  // PagBank: URL dinâmica por cliente
}
```

**Decisão arquitetural:** `apiUrl` é opcional e específico do PagBank. Outros bancos ignoram o campo. A URL da API PIX PagBank varia por cliente (sandbox vs produção), por isso é armazenada por cliente no DB.

---

#### `401 Unauthorized` — token Bearer inválido

**Contexto:** Chamada à API PagBank retorna 401.

**Causa:** `apiKey` incorreto ou expirado.

**Solução:** Verificar o token no portal PagBank e re-cadastrar as credenciais do cliente.

## Relacionados

- [[pagbank_index]] — índice do PagBank
