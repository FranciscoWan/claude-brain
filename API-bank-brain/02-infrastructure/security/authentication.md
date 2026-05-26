---
title: Autenticação e Criptografia
category: security
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/security/_index]]"
tags:
  - infrastructure
  - security
  - authentication
  - stable
---

# Autenticação e Criptografia

## Contexto

Mecanismos de autenticação do ViaPIX: controle de acesso via `x-api-key`, criptografia de credenciais bancárias com RSA+AES, e mTLS para o Banco do Brasil.

## Conteúdo principal

### Autenticação da API ViaPIX

- Header `x-api-key` validado em toda requisição pelo `ApiKeyGuard` (registrado como `APP_GUARD` global)
- Valor configurado em `VIAPIX_API_KEY` no `.env`
- Único endpoint isento: `POST /pix/{bank}/webhook/pix` — marcado com `@Public()`

```typescript
// Isentar rota do ApiKeyGuard
@Public()
@Post('webhook/pix')
receiveWebhook(@Body() dto: WebhookPixDto) { ... }
```

### Criptografia de credenciais — RSA-2048 + AES-256-CBC

#### Cadastro do cliente

```
POST /clientes com credenciais em texto plano
  → CryptoService gera par RSA-2048 (publicKey, privateKey)
  → gera chave AES-256 aleatória
  → cifra credenciais com AES → encrypted_blob
  → cifra chave AES com publicKey RSA → armazenado junto em encrypted_blob
  → salva publicKey no DB (public_key_pem)
  → retorna { id, privateKey } — privateKey exibida apenas uma vez
```

#### Descriptografia por requisição

```
cnpj + privateKey (RSA-2048)
  → ClientesService.findAndDecryptCredentials()
  → busca cadastro_cliente por cnpj
  → decriptografa encrypted_blob (AES-256-CBC) → clientId, clientSecret, chavePix...
  → decriptografa encrypted_certify (AES-256-CBC) → certPem, certKeyPem (BB only)
  → monta ClienteCredentials → Provider
```

#### Propriedades do modelo

| Propriedade | Detalhe |
|---|---|
| A `privateKey` nunca é armazenada | Exibida apenas uma vez no cadastro |
| `encrypted_blob` | Credenciais serializadas em JSON e cifradas com AES; chave AES cifrada com RSA pública |
| `encrypted_certify` | Mesmo padrão — armazena `certPem` + `certKeyPem` do BB |
| Perda da privateKey | Cliente precisa re-cadastrar as credenciais |

### mTLS — Banco do Brasil

- O BB exige que cada chamada OAuth2 e PIX seja feita com o certificado mTLS do cliente
- `certPem` + `certKeyPem` armazenados em `encrypted_certify` no DB
- Agente HTTPS configurado por requisição com os certificados descriptografados

```typescript
const httpsAgent = new https.Agent({
  cert: credentials.certPem,
  key: credentials.certKeyPem,
  rejectUnauthorized: process.env.NODE_ENV !== 'production',
});
```

- Em sandbox: `rejectUnauthorized: false` aceita certificados auto-assinados
- Em produção: certificado deve ser assinado pelo BB (ver `credentials.md` do BB)

### Controle de acesso por tipo de endpoint

| Tipo de endpoint | Credenciais do cliente |
|---|---|
| POST / PUT / PATCH | `cnpj` + `privateKey` no **body** |
| GET | Headers `x-cnpj` + `x-private-key` |
| `POST /pix/{bank}/webhook/pix` | Nenhuma — `@Public()` |

## Relacionados

- [[02-infrastructure/security/_index]] — índice de segurança
