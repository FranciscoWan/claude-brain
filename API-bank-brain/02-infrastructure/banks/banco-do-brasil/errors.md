---
title: Banco do Brasil — Erros e Soluções
category: infrastructure
bank: banco-do-brasil
status: stable
updated: 2026-05-26
parent: "[[banco-do-brasil_index]]"
tags:
  - infrastructure
  - banco-do-brasil
  - errors
  - stable
  - bank
---

# Banco do Brasil — Erros e Soluções

## Contexto

Erros encontrados na integração com o BB (autenticação, API PIX) e nos webhooks/long polling. Consolidado de duas fontes originais.

## Conteúdo principal

### Erros da integração BB

#### `UNABLE_TO_VERIFY_LEAF_SIGNATURE`

**Contexto:** Chamada OAuth2 ou API PIX BB falha com erro TLS.

**Causa:** O certificado do BB sandbox não é reconhecido pela cadeia de certificados do Node.js.

**Solução:** Usar `rejectUnauthorized: false` apenas em desenvolvimento/sandbox:

```typescript
const httpsAgent = new https.Agent({
  cert: credentials.certPem,
  key: credentials.certKeyPem,
  rejectUnauthorized: process.env.NODE_ENV !== 'production',
});
```

Em produção, importar a cadeia de certificados do BB.

---

#### `401 Unauthorized` no token OAuth2

**Contexto:** POST para `{BB_OAUTH_URL}/oauth/token` retorna 401.

**Causa:** `clientId` ou `clientSecret` incorretos, ou certificado mTLS não corresponde ao cadastrado no portal BB.

**Solução:** Verificar credenciais no portal developer.bb.com.br. Confirmar que o `certPem` cadastrado no ViaPIX é o mesmo assinado pelo BB para esse `clientId`.

---

#### `403 Forbidden` — escopo insuficiente

**Contexto:** Operação PIX retorna 403.

**Causa:** A aplicação no portal BB não tem o escopo necessário (ex: `cob.write`).

**Solução:** developer.bb.com.br → sua aplicação → Escopos → adicionar escopos necessários.

---

#### `gw-dev-app-key` ausente

**Contexto:** API BB retorna erro de autenticação mesmo com token válido.

**Causa:** O query param `gw-dev-app-key` não foi enviado na requisição.

**Solução:** Garantir que `BB_GW_APP_KEY` está no `.env` e que o adapter BB inclui o param em todas as chamadas à API PIX.

---

### Erros dos webhooks BB

#### Migration falhou — tabela `pix_notifications` não criada

**Contexto:** Aplicação sobe, mas webhook não consegue salvar notificação.

**Causa:** Migration da tabela `pix_notifications` não foi executada.

**Solução:**
```bash
npm run typeorm migration:run
```
Verificar se `PixNotification` está listada em `entities` no `TypeOrmModule`.

---

#### Validação de DTO falhou no webhook

**Contexto:** `POST /pix/bb/webhook/pix` retorna `400 Bad Request`.

**Causa:** O payload do BB tem campo `pix` como array, mas o DTO esperava objeto.

**Solução:**
```typescript
export class WebhookPixDto {
  @IsArray()
  pix: PixNotificationItemDto[];
}
```

---

#### TypeORM export ausente no módulo

**Contexto:** `WebhookService` não consegue injetar o repositório de `PixNotification`.

**Causa:** `TypeOrmModule.forFeature([PixNotification])` não estava exportado no módulo.

**Solução:**
```typescript
imports: [TypeOrmModule.forFeature([PixNotification])],
exports: [TypeOrmModule],
```

---

#### Long polling retorna imediatamente sem aguardar

**Contexto:** `GET /pix/notifications/{txid}/poll` retorna `{ received: false }` sem esperar.

**Causa:** O loop de polling não aguardava entre verificações — faltava o `sleep` de 2 segundos.

**Solução:**
```typescript
const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));
while (elapsed < timeout) {
  const notification = await this.repo.findOne({ where: { txid } });
  if (notification) return { received: true, ...notification };
  await sleep(2000);
  elapsed += 2;
}
```

## Relacionados

- [[banco-do-brasil_index]] — índice do Banco do Brasil
