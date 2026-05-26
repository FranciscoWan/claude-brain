---
title: PagBank — Credenciais
category: infrastructure
bank: pagbank
status: draft
updated: 2026-05-26
parent: "[[pagbank_index]]"
tags:
  - infrastructure
  - bank
  - pagbank
  - credentials
  - draft
---

# PagBank — Credenciais

## Contexto

O PagBank usa Bearer token para autenticação. A URL da API PIX é dinâmica por cliente (sandbox vs produção), então deve ser armazenada por cliente no banco de dados junto com as demais credenciais.

## Conteúdo principal

### Credenciais necessárias

| Campo | Origem |
|---|---|
| `apiKey` | Token Bearer do cliente PagBank (via Connect SMS) |
| `chavePix` | Chave PIX do cliente |
| `apiUrl` | URL da API PagBank do cliente (dinâmica: sandbox ou produção) |

### Passos para obter credenciais

1. Criar conta PagBank em developer.pagseguro.uol.com.br
2. Obter token Bearer via fluxo Connect SMS
3. Obter `chavePix`
4. Anotar a URL da API PIX da conta (sandbox ou produção)

### Cadastro no ViaPIX (planejado)

```json
{
  "cnpj": "00.000.000/0001-00",
  "bankCode": "290",
  "apiKey": "Bearer-token-pagbank",
  "chavePix": "...",
  "apiUrl": "https://sandbox.api.pagseguro.com"
}
```

### Variáveis de ambiente

| Variável | Descrição |
|---|---|
| `PAGBANK_OWNER_TOKEN` | Token global do dono da conta PagBank |

### Adaptações necessárias no código

O campo `apiUrl` é opcional e específico do PagBank. Já está declarado na interface `ClienteCredentials`:

```typescript
export interface ClienteCredentials {
  // ... campos existentes ...
  apiKey?: string;   // PagBank: token bearer
  apiUrl?: string;   // PagBank: URL dinâmica por cliente
}
```

Para o cadastro funcionar, será necessário:
- Atualizar `CreateClienteDto` para aceitar `apiKey` e `apiUrl`
- Atualizar `CryptoService` para incluir `apiKey` e `apiUrl` no `encrypted_blob`

## Relacionados

- [[pagbank_index]] — índice do PagBank
