---
title: Banco do Brasil — Credenciais e mTLS
category: infrastructure
bank: banco-do-brasil
status: stable
updated: 2026-05-26
parent: "[[banco-do-brasil_index]]"
tags:
  - infrastructure
  - banco-do-brasil
  - credentials
  - stable
  - bank
---

# Banco do Brasil — Credenciais e mTLS

## Contexto

O BB exige OAuth2 com mTLS obrigatório. Cada cliente deve gerar um certificado TLS mútuo, submetê-lo ao portal BB e cadastrar no ViaPIX junto com `clientId`, `clientSecret` e `chavePix`.

## Conteúdo principal

### Credenciais necessárias

| Campo | Origem |
|---|---|
| `clientId` | Portal developer.bb.com.br |
| `clientSecret` | Portal developer.bb.com.br |
| `chavePix` | Conta BB do cliente |
| `certPem` | Gerado via OpenSSL + assinado pelo BB |
| `certKeyPem` | Gerado via OpenSSL |

### Passos para obter credenciais

1. Criar aplicação no portal developer.bb.com.br
2. Obter `clientId` e `clientSecret`
3. Gerar certificado mTLS (ver seção abaixo)
4. Submeter CSR no portal e baixar `client.crt`
5. Obter `chavePix` na conta BB

### Geração do certificado mTLS

#### 1. Gerar chave privada

```bash
openssl genrsa -out client.key 2048
```

#### 2. Criar arquivo de extensões `bb-ext.conf`

```ini
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
CN = seu-cnpj-ou-nome

[v3_req]
keyUsage = digitalSignature
extendedKeyUsage = clientAuth
```

#### 3. Gerar CSR

```bash
openssl req -new -key client.key -out client.csr -config bb-ext.conf
```

#### 4. Submeter CSR no portal developer.bb.com.br

- Acesse o portal → sua aplicação → Certificados
- Faça upload do `client.csr`
- Baixe o certificado assinado `client.crt`

#### 5. Verificar certificado

```bash
openssl x509 -in client.crt -text -noout
```

### Cadastro no ViaPIX

`POST /clientes` com o payload:

```json
{
  "cnpj": "00.000.000/0001-00",
  "bankCode": "001",
  "clientId": "...",
  "clientSecret": "...",
  "chavePix": "...",
  "certPem": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
  "certKeyPem": "-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----"
}
```

O ViaPIX armazena `certPem` + `certKeyPem` criptografados em `encrypted_certify` no banco de dados.

### Renovação do certificado

- Validade: 1 ano
- Repetir o processo de geração acima e re-cadastrar as credenciais do cliente

### Escopos OAuth2 necessários

| Escopo | Uso |
|---|---|
| `cob.write` | Criar/atualizar cobranças |
| `cob.read` | Consultar cobranças |
| `webhook.read` | Consultar webhooks |

Configure os escopos em developer.bb.com.br → sua aplicação → Escopos.

## Decisões / Soluções

- O BB sandbox aceita certificados auto-assinados para testes
- Em produção, o CSR deve ser submetido e assinado pelo BB
- `encrypted_certify` usa o mesmo padrão RSA+AES de `encrypted_blob`

## Relacionados

- [[banco-do-brasil_index]] — índice do Banco do Brasil

