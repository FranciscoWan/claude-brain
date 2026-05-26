---
title: Schema do Banco de Dados
category: database
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/database/_index]]"
tags:
  - infrastructure
  - database
  - schema
  - stable
---

# Schema do Banco de Dados

## Contexto

Schema PostgreSQL do ViaPIX — duas tabelas principais: `cadastro_cliente` (clientes e credenciais criptografadas) e `pix_notifications` (notificações de pagamento recebidas via webhook).

## Conteúdo principal

### Tabela `cadastro_cliente`

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | UUID | Identificador único do cliente |
| `cnpj` | VARCHAR | CNPJ do cliente (único) |
| `bank_code` | VARCHAR | Código do banco (`001`, `ITAU`, `290`) |
| `public_key_pem` | TEXT | Chave pública RSA-2048 do cliente |
| `encrypted_blob` | TEXT | Credenciais bancárias criptografadas (RSA+AES) |
| `encrypted_certify` | TEXT | Certificado mTLS criptografado (BB only) |
| `created_at` | TIMESTAMP | Data de cadastro |
| `updated_at` | TIMESTAMP | Última atualização |

**Conteúdo de `encrypted_blob`** (após descriptografia):
```json
{
  "clientId": "...",
  "clientSecret": "...",
  "chavePix": "...",
  "apiKey": "...",
  "apiUrl": "..."
}
```

**Conteúdo de `encrypted_certify`** (após descriptografia, BB only):
```json
{
  "certPem": "-----BEGIN CERTIFICATE-----\n...",
  "certKeyPem": "-----BEGIN RSA PRIVATE KEY-----\n..."
}
```

### Tabela `pix_notifications`

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | UUID | Identificador único |
| `txid` | VARCHAR | ID da transação PIX |
| `end_to_end_id` | VARCHAR | ID end-to-end do pagamento |
| `valor` | VARCHAR | Valor pago |
| `horario` | TIMESTAMP | Horário do pagamento |
| `banco` | VARCHAR | Banco que gerou a notificação |
| `payload` | JSONB | Payload completo do webhook |
| `created_at` | TIMESTAMP | Data de recebimento |

### Entidades TypeORM

| Entidade | Arquivo |
|---|---|
| `CadastroCliente` | `clientes/entities/cadastro-cliente.entity.ts` |
| `PixNotification` | `pix/entities/pix-notification.entity.ts` |

## Relacionados

- [[02-infrastructure/database/_index]] — índice do banco de dados
