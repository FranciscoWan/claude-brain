---
title: Authentication — JwtMiddleware e AuthorizationService
category: infrastructure
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/security/_index]]"
tags:
  - infrastructure
  - security
  - authentication
  - stable
---

# Authentication — JwtMiddleware e AuthorizationService

## Contexto

Dois componentes compõem a camada de autenticação por requisição: o `JwtMiddleware` (executa em toda requisição) e o `AuthorizationService` (chamado explicitamente pelos endpoints `/me` e `/permissions`).

## Conteúdo principal

### JwtMiddleware (`jwt.middleware.ts`)

Executa em todas as requisições via `NestMiddleware`. Responsabilidades:

1. **Extrair token** — lê `access_token` do cookie ou do header `Authorization: Bearer <token>`
2. **Auto-refresh proativo** — se `exp - now ≤ 120s` e há `refresh_token`, renova antes de continuar
3. **Recuperação de sessão** — se `access_token` ausente mas `refresh_token` presente, tenta renovar
4. **Popular CLS** — injeta `userId` (`sub`), `userName` (`name` ou `preferred_username`), `userRoles` (`realm_access.roles`) no `ClsService<AppClsStore>`

**Tabela de decisão do middleware:**

| Estado dos cookies | Ação |
|---|---|
| Só `access_token`, válido, expira em > 120s | Popula CLS, chama `next()` |
| Só `access_token`, expira em ≤ 120s, tem `refresh_token` | Renova, atualiza cookie na request, popula CLS |
| Sem `access_token`, tem `refresh_token` | Tenta renovar; sucesso → atualiza cookie; falha → limpa cookies, `next()` |
| Nem `access_token` nem `refresh_token` | Chama `next()` sem CLS |
| Refresh falha com `access_token` presente | Limpa ambos os cookies, `next()` sem CLS |

> [!important]
> O middleware **não lança exceção**. Toda falha de autenticação é tratada pelos guards downstream (ex: `KeycloakAuthGuard`). Rotas com `@Public()` passam sem validação.

### CLS Store populado

```ts
cls.set('userId', decoded.sub)
cls.set('userName', decoded.name || decoded.preferred_username)
cls.set('userRoles', decoded.realm_access?.roles || [])
```

Disponível em qualquer serviço via injeção de `ClsService<AppClsStore>`.

### AuthorizationService (`authorization.service.ts`)

| Método | Entrada | Retorno |
|---|---|---|
| `getUserInfo(req, accessToken)` | token do cookie | `{ user: User, isAuthenticated: true }` |
| `getPermissions(req, token)` | token da sessão | `PermissionsResponse` |

Ambos decodificam o token via `KeycloakClient.decodeToken()` (sem verificação de assinatura) e usam `UserResponsePresenter` para formatar a resposta.

### Interfaces de dados

**`DecodedToken`** (payload JWT do Keycloak):
```ts
{
  sub: string,
  email?: string,
  name?: string,
  preferred_username?: string,
  realm_access?: { roles: string[] },
  resource_access?: { [client: string]: { roles: string[] } },
  aud?: string | string[],
  exp: number,
  iat: number,
  iss?: string,
  session_state?: string,
  typ?: string,
  azp?: string,
  scope?: string
}
```

**`User`** (resposta do `/me`):
```ts
{
  sub: string,
  email?: string,
  name?: string,
  preferred_username?: string,
  resource_access: Record<string, { roles: string[] }>,
  realm_access: { roles: string[] }
}
```

## Relacionados

- [[02-infrastructure/security/_index]] — Índice de segurança
