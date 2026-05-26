---
title: Migrations
category: database
status: stable
updated: 2026-05-26
parent: "[[02-infrastructure/database/_index]]"
tags:
  - infrastructure
  - database
  - migrations
  - stable
---

# Migrations

## Contexto

Gerenciamento de migrations do banco de dados PostgreSQL via TypeORM CLI.

## Conteúdo principal

### Localização

```
src/migrations/
```

### Comandos

**Executar migrations pendentes:**
```bash
npm run typeorm migration:run
```

**Gerar nova migration a partir das entidades:**
```bash
npm run typeorm migration:generate -- -n NomeDaMigration
```

**Reverter última migration:**
```bash
npm run typeorm migration:revert
```

### Entidades gerenciadas

As entidades devem estar listadas em `TypeOrmModule.forFeature([...])` no módulo correspondente para que o TypeORM as reconheça:

| Entidade | Módulo |
|---|---|
| `CadastroCliente` | `ClientesModule` |
| `PixNotification` | `PixModule` |

## Decisões / Soluções

Se `PixNotification` não estiver listada em `entities` no `TypeOrmModule`, o webhook não consegue salvar notificações e a migration pode não ser detectada. Garantir:

```typescript
// PixModule
imports: [TypeOrmModule.forFeature([PixNotification])],
exports: [TypeORM],
```

## Relacionados

- [[02-infrastructure/database/_index]] — índice do banco de dados
