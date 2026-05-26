---
title: Produto — Índice
category: product
status: stable
updated: 2026-05-26
parent: "[[00-index]]"
tags:
  - product
  - stable
---

# Produto

Documentação de produto do ViaPIX: roadmap de desenvolvimento, features implementadas e decisões arquiteturais registradas como ADRs.

## Contexto

Esta seção cobre o que o sistema faz do ponto de vista do produto — o que está pronto, o que está pendente e as decisões que moldaram a arquitetura.

## Conteúdo

### Planejamento

- [[roadmap]] — fases concluídas e pendentes por componente

### Features

- [[01-product/features/_index]] — catálogo de features implementadas
  - [[endpoint-cobrar]] — `POST /api/pix/cobrar`: criar cobrança e aguardar pagamento
  - [[cliente-local]] — `viapix_be_cliente`: sistema integrador de referência

### Decisões arquiteturais

- [[adr-001-modelo-credenciais]] — por que as credenciais pertencem ao cliente, não ao ViaPIX

## Relacionados

- [[00-index]] — índice geral do vault
