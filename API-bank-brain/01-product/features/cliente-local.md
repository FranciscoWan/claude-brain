---
title: Feature — Cliente Local (viapix_be_cliente)
category: product
status: stable
updated: 2026-05-26
parent: "[[01-product/features/_index]]"
tags:
  - product
  - feature
  - stable
---

# Cliente Local (`viapix_be_cliente`)

## Contexto

Backend NestJS separado que simula um sistema integrador consumindo a API ViaPIX. Usado para testes e demonstração do fluxo completo de integração. Não contém lógica bancária — apenas orquestra chamadas ao ViaPIX.

## Conteúdo principal

### Identificação

| Item | Valor |
|---|---|
| Pasta | `viapix_be_cliente/` |
| Porta | `3001` |
| Framework | NestJS |
| Papel | Simula integrador que consome a API ViaPIX |

### Responsabilidade

1. Recebe requisição de cobrança
2. Monta payload com `cnpj + privateKey + dados da cobrança`
3. Chama `POST /api/pix/cobrar` no ViaPIX (porta `3006`)
4. Retorna resultado ao chamador

### Variáveis de ambiente

| Variável | Descrição |
|---|---|
| `VIAPIX_URL` | URL base do ViaPIX (ex: `http://localhost:3006`) |
| `VIAPIX_API_KEY` | Chave `x-api-key` para autenticar no ViaPIX |
| `CLIENT_CNPJ` | CNPJ do cliente cadastrado no ViaPIX |
| `CLIENT_PRIVATE_KEY` | Chave RSA privada do cliente |
| `CLIENT_BANK_CODE` | Código do banco (`001`, `ITAU`, etc.) |

## Decisões / Soluções

### Erros comuns ao consumir o ViaPIX

#### Erro 1: `401 Unauthorized` — x-api-key ausente ou incorreto

**Causa:** Header `x-api-key` não enviado ou valor incorreto.

**Solução:** Verificar se `VIAPIX_API_KEY` no `.env` do cliente é igual a `VIAPIX_API_KEY` no `.env` do ViaPIX.

```typescript
headers: {
  'x-api-key': process.env.VIAPIX_API_KEY,
  'Content-Type': 'application/json',
}
```

---

#### Erro 2: `400 Bad Request` — privateKey inválida

**Causa:** A `privateKey` foi copiada com quebras de linha perdidas ou espaços extras.

**Solução:** Garantir que `\n` literais do `.env` sejam convertidos para quebras reais:

```typescript
const privateKey = process.env.CLIENT_PRIVATE_KEY.replace(/\\n/g, '\n');
```

---

#### Erro 3: `ECONNREFUSED` — ViaPIX não está rodando

**Causa:** Servidor ViaPIX não está rodando na porta `3006`.

**Solução:**
```bash
cd viapix_be
npm run start:dev
```

---

#### Erro 4: `received: false` — pagamento não detectado no timeout

**Causas possíveis:**
1. Webhook do BB não chegou (URL inacessível publicamente)
2. `timeout` muito curto
3. `txid` da cobrança não corresponde ao da notificação

**Solução:**
1. Em desenvolvimento, usar ngrok para expor a URL do webhook
2. Aumentar `timeout` para `55` (máximo)
3. Simular o webhook manualmente (ver `webhooks.md` do BB)

## Relacionados

- [[01-product/features/_index]] — catálogo de features
