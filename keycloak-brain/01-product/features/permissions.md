---
title: Feature — Permissões por Role
category: product
status: stable
updated: 2026-05-28
parent: "[[01-product/features/_index]]"
tags:
  - product
  - feature
  - stable
---

# Feature — Permissões por Role

## Contexto

O sistema de permissões é derivado diretamente do JWT do Keycloak, sem consulta adicional ao servidor. As permissões são calculadas a partir dos campos `resource_access` e `realm_access` do token decodificado.

## Conteúdo principal

### Estrutura de permissões (`GET /permissions`)

```ts
{
  user: { sub, email, name, preferred_username },
  permissions: {
    viaauth: {
      canCreateUsers: boolean,  // roles.includes('admin')
      canManageUsers: boolean,  // roles.includes('admin')
      canViewUsers: boolean,    // roles.includes('admin')
    },
    viaatendimento: {
      isAtendente: boolean,     // resource_access.viaatendimento.roles.includes('atendente')
      isGestor: boolean,        // resource_access.viaatendimento.roles.includes('gestor')
    },
    realm: {
      roles: string[],          // realm_access.roles
      isSuper: boolean,         // roles.includes('super')
      isAdmin: boolean,         // roles.includes('admin')
    }
  }
}
```

### Mapeamento de roles → permissões viaauth

| Role no Keycloak (`resource_access.viaauth.roles`) | Permissão derivada |
|---|---|
| `admin` | `canCreateUsers`, `canManageUsers`, `canViewUsers` |

> [!note]
> A rota `GET /permissions` usa o decorator `@SessionToken()` que extrai o token do cookie ou header Authorization.

### Endpoint de dados do usuário (`GET /me`)

Retorna `{ user: User, isAuthenticated: true }` com todos os `resource_access` e `realm_access` brutos do token.

## Relacionados

- [[01-product/features/_index]] — Índice de features
