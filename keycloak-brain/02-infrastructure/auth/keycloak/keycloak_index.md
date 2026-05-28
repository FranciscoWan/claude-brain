---
title: Keycloak — Visão Geral
category: infrastructure
service: keycloak
status: stable
updated: 2026-05-28
parent: "[[02-infrastructure/auth/_index]]"
tags:
  - infrastructure
  - service
  - keycloak
  - stable
---

# Keycloak — Visão Geral

## Contexto

O Keycloak é o Authorization Server central da aplicação. Esta integração implementa OAuth2/OIDC com PKCE, refresh de tokens, revogação e token exchange (RFC 8693) para acesso service-to-service.

**Data de inclusão no projeto:** desconhecida (não há histórico git disponível)
**Status atual:** stable — em uso em produção

## Conteúdo principal

### Serviços implementados

| Arquivo | Classe | Responsabilidade |
|---|---|---|
| `keycloak-client.service.ts` | `KeycloakClient` | Todas as chamadas HTTP ao Keycloak |
| `pkce.service.ts` | `PkceService` | Orquestração do fluxo PKCE |
| `refresh-token.service.ts` | `RefreshTokenService` | Renovação de tokens com tratamento de erro |

### Dependências do serviço

- [[02-infrastructure/auth/keycloak/endpoints|Endpoints]] — rotas HTTP expostas pelo módulo
- [[02-infrastructure/auth/keycloak/pkce-flow|PKCE Flow]] — fluxo de autenticação PKCE completo
- [[02-infrastructure/auth/keycloak/token-exchange|Token Exchange]] — RFC 8693 service-to-service
- [[02-infrastructure/auth/keycloak/errors|Errors]] — erros mapeados do Keycloak

### Configuração

Toda a configuração lida via `EnvService` (campo `env.oauth` do tipo `OAuthConfig`):

| Campo | Uso |
|---|---|
| `baseUrl` | Base para `/token`, `/auth`, `/logout` |
| `clientId` | Identificador do cliente no Keycloak |
| `clientSecret` | Opcional — adicionado quando presente |
| `redirectUri` | URI de callback pós-autenticação |
| `cookieDomain` | Domínio dos cookies de autenticação |
| `frontendUrl` | URL de redirecionamento pós-callback |

## Relacionados

- [[02-infrastructure/auth/_index]] — Índice de auth services
