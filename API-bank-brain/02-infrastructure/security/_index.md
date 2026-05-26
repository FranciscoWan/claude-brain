---
title: Segurança — Índice
category: security
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/_index]]"
tags:
  - infrastructure
  - security
  - stable
---

# Segurança

## Contexto

Documentação das camadas de segurança do ViaPIX: autenticação da API, criptografia de credenciais e mTLS por banco.

## Conteúdo principal

### Camadas de segurança

| Camada | Mecanismo | Status |
|---|---|---|
| API ViaPIX | `x-api-key` global (`APP_GUARD`) | ✅ |
| Credenciais cliente | RSA-2048 + AES-256-CBC | ✅ |
| OAuth2 BB | `client_credentials` + mTLS obrigatório | ✅ |
| OAuth2 Itaú | `client_credentials` via APIGateway STS | ✅ |
| mTLS BB | Certificado PEM por cliente (`certPem` + `certKeyPem`) | ✅ |
| mTLS Itaú | Não exige | — |
| mTLS PagBank | Não exige | — |

### Arquivos

- [[authentication]] — detalhes de autenticação, fluxo de descriptografia e controle de acesso

## Relacionados

- [[02-infrastructure/_index]] — índice de infraestrutura
- [[00-index]] — índice geral do vault
