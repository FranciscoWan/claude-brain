---
title: Itaú — Erros e Soluções
category: infrastructure
bank: itau
status: stable
updated: 2026-05-26
parent: "[[itau_index]]"
tags:
  - infrastructure
  - bank
  - itau
  - errors
  - stable
---

# Itaú — Erros e Soluções

## Contexto

Erros encontrados na integração do ViaPIX com o Itaú e suas soluções.

## Conteúdo principal

#### `401 Unauthorized` no token OAuth2

**Contexto:** POST para `{ITAU_OAUTH_URL}/oauth/token` retorna 401.

**Causa:** `clientId` ou `clientSecret` incorretos, ou env vars não carregadas.

**Solução:** Verificar `ITAU_CLIENT_ID` e `ITAU_CLIENT_SECRET` no `.env`. Confirmar no portal developer.itau.com.br.

---

#### `400 Bad Request` — campo obrigatório ausente

**Contexto:** POST `/pix/itau/cob` retorna 400.

**Causa:** Payload não incluiu campo obrigatório exigido pelo Itaú (ex: `calendario.expiracao`).

**Solução:** Verificar no adapter Itaú se todos os campos obrigatórios estão sendo mapeados corretamente.

---

#### `txid` inválido

**Contexto:** Itaú rejeita o `txid` gerado.

**Causa:** O Itaú exige `txid` com 26 a 35 caracteres alfanuméricos.

**Solução:** Gerar com `crypto.randomBytes(16).toString('hex')` (32 chars) — compatível com BB e Itaú.

---

#### Token expirado em requisições longas

**Contexto:** Operação falha com 401 depois de algum tempo.

**Causa:** Cache de token expirou.

**Solução:** O `itau.auth.ts` deve verificar `expires_in` antes de usar o token cacheado e renovar se necessário.

---

#### `bankCode` não resolvido

**Contexto:** `ProviderRegistry` lança exceção para bankCode `ITAU`.

**Causa:** O registry estava mapeando apenas `341` (código numérico), não o alias `ITAU`.

**Solução:** Registrar ambos no `ProviderRegistry`:
```typescript
this.providers.set('ITAU', itauProvider);
this.providers.set('341', itauProvider);
```

## Relacionados

- [[itau_index]] — índice do Itaú
