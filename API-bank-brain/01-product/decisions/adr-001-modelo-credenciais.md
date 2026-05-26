---
title: "ADR-001: Credenciais bancárias pertencem ao cliente, não ao ViaPIX"
category: product
status: stable
updated: 2026-05-26
parent: "[[01-product/_index]]"
tags:
  - product
  - decision
  - stable
---

# ADR-001: Credenciais bancárias pertencem ao cliente, não ao ViaPIX

## Contexto

O ViaPIX precisa autenticar-se nas APIs dos bancos (BB, Itaú, PagBank) para executar operações PIX em nome dos clientes. A questão é: de quem são as credenciais (clientId, clientSecret, certificado mTLS)?

## Conteúdo principal

### Decisão

> As credenciais bancárias pertencem ao **cliente** (integrador), não ao ViaPIX.

O ViaPIX não possui conta bancária própria. Cada cliente cadastrado traz suas próprias credenciais — obtidas diretamente no portal do banco (ex: developer.bb.com.br) — e as armazena criptografadas no ViaPIX.

### Fluxo de cadastro

```
Cliente obtém credenciais no portal do banco
  → POST /clientes com credenciais em texto plano
  → ViaPIX criptografa com RSA+AES e armazena no DB
  → Retorna { id, privateKey } — privateKey nunca mais exibida
```

### Fluxo por requisição

```
cnpj + privateKey (na requisição)
  → ClientesService descriptografa credenciais do DB
  → Provider usa credenciais para autenticar no banco
  → Executa operação PIX
```

## Decisões / Soluções

### Implicações da decisão

| Implicação | Detalhe |
|---|---|
| Sem credenciais em repouso | O ViaPIX conhece as credenciais apenas em memória, durante a requisição |
| Par de chaves RSA por cliente | Cada cliente tem seu próprio par RSA-2048 |
| Segredo único do cliente | A `privateKey` é o único segredo que o cliente precisa guardar |
| Perda da privateKey | Se o cliente perder a `privateKey`, precisa re-cadastrar as credenciais |

### Por que não centralizar as credenciais no ViaPIX?

- Responsabilidade legal: cada cliente é o titular da conta bancária
- Isolamento: um vazamento de credenciais de um cliente não afeta os demais
- Compliance: o banco valida as credenciais contra o CNPJ do titular

## Relacionados

- [[01-product/_index]] — índice de produto
