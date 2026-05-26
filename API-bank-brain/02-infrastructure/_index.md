---
title: Infraestrutura — Índice
category: infrastructure
status: stable
updated: 2026-05-26
parent: "[[00-index]]"
tags:
  - infrastructure
  - stable
---

# Infraestrutura

## Contexto

Documentação técnica do ViaPIX: arquitetura de camadas, stack, variáveis de ambiente, integrações bancárias, banco de dados e segurança.

## Conteúdo principal

### Arquitetura — fluxo de requisição

```
Request (x-api-key + cnpj + privateKey + payload)
  → ApiKeyGuard         valida header x-api-key contra VIAPIX_API_KEY
  → Controller          valida entrada via DTOs
  → PixService          chama ClientesService
  → ClientesService     busca cadastro_cliente, descriptografa encrypted_blob + encrypted_certify
  → ProviderRegistry    resolve bankCode → PixProvider
  → Provider            obtém token OAuth2 (cache por clientId), chama API do banco
  → Adapter             traduz DTO interno ↔ payload do banco; normaliza erros
  → API do banco
```

**Autenticação por tipo de endpoint:**

| Tipo | Como enviar |
|---|---|
| POST / PUT / PATCH | `cnpj` + `privateKey` no **body** |
| GET | Headers `x-cnpj` + `x-private-key` |

**`x-api-key`:**

| Contexto | Responsável |
|---|---|
| Produção / integração real | O **cliente** envia o header em toda requisição |
| Testes manuais (arquivos `.http`) | O **desenvolvedor** inclui manualmente com o valor de `VIAPIX_API_KEY` do `.env` |

Único endpoint sem `x-api-key`: `POST /pix/{bank}/webhook/pix` — marcado `@Public()`, pois é chamado diretamente pelo banco.

### Estrutura de pastas

```
src/
├── common/
│   ├── decorators/public.decorator.ts   @Public() — isenta rota do ApiKeyGuard
│   └── guards/api-key.guard.ts          valida x-api-key; APP_GUARD global
├── modules/
│   ├── pix/
│   │   ├── controllers/    pix.controller.ts + webhook.controller.ts
│   │   ├── services/       pix.service.ts + provider-registry.service.ts + webhook.service.ts
│   │   ├── interfaces/     pix-provider.interface.ts
│   │   ├── providers/
│   │   │   ├── banco-do-brasil/  bb.provider / bb.adapter / bb.auth / bb.types / bb.webhook
│   │   │   └── itau/             itau.provider / itau.adapter / itau.auth / itau.types
│   │   └── dto/
│   └── clientes/
│       ├── controllers/    clientes.controller.ts
│       ├── services/       clientes.service.ts + crypto.service.ts
│       ├── entities/       cadastro-cliente.entity.ts
│       └── dto/
├── config/                 env.config.ts (validação Joi)
└── main.ts
```

### Camadas obrigatórias

| Camada | Responsabilidade | Localização |
|---|---|---|
| DTO | Entrada/saída, validação | `pix/dto/` |
| Interface | Contrato dos providers | `pix/interfaces/pix-provider.interface.ts` |
| Provider | Integração com banco | `pix/providers/[banco]/[banco].provider.ts` |
| Adapter | Tradução de payloads | `pix/providers/[banco]/[banco].adapter.ts` |
| Registry | Mapa `bankCode → provider` | `pix/services/provider-registry.service.ts` |
| ClientesService | Descriptografia de credenciais | `clientes/services/clientes.service.ts` |

### Convenções de DTO

- POST/PUT/PATCH: `IntersectionType(PixAuthDto, <OperacaoDto>)` — mescla `cnpj + privateKey` com payload
- Response DTOs: **classes** (não interfaces) com `@ApiProperty` para o Swagger
- `UpdateXDto` herda de `PartialType(CreateXDto)` via `@nestjs/swagger`

### Contrato `PixProvider`

```typescript
export interface PixProvider {
  createCob(credentials: ClienteCredentials, dto: CreateCobDto): Promise<CobResponseDto>;
  getCob(credentials: ClienteCredentials, txid: string): Promise<CobResponseDto>;
  updateCob(credentials: ClienteCredentials, txid: string, dto: UpdateCobDto): Promise<CobResponseDto>;
  createCobv?(credentials: ClienteCredentials, dto: CreateCobvDto): Promise<CobvResponseDto>;
  createPix?(credentials: ClienteCredentials, dto: CreatePixDto): Promise<PixResponseDto>;
  createDevolucao?(credentials: ClienteCredentials, e2eId: string, id: string, dto: CreateDevolucaoDto): Promise<DevolucaoResponseDto>;
}
```

Métodos com `?` são opcionais — implementados apenas pelos bancos que suportam a operação.

### Contrato `ClienteCredentials`

```typescript
export interface ClienteCredentials {
  clientId: string;
  clientSecret: string;
  chavePix: string;
  apiKey?: string;       // PagBank: token bearer
  refreshToken?: string; // reservado para futuro
  certPem?: string;      // BB: certificado mTLS
  certKeyPem?: string;   // BB: chave privada mTLS
  apiUrl?: string;       // PagBank: URL dinâmica por cliente
}
```

Localização: `src/modules/pix/interfaces/pix-provider.interface.ts`

### Operações PIX expostas

| Operação | Rota | Bancos suportados |
|---|---|---|
| Criar cobrança COB | `POST /pix/{bank}/cob` | BB, Itaú |
| Consultar COB | `GET /pix/{bank}/cob/{txid}` | BB, Itaú |
| Atualizar COB | `PATCH /pix/{bank}/cob/{txid}` | BB, Itaú |
| Criar cobrança COBV | `POST /pix/{bank}/cobv` | Itaú |
| Criar PIX | `POST /pix/{bank}/pix` | Itaú |
| Devolver PIX | `POST /pix/{bank}/pix/{e2eId}/devolucao/{id}` | Itaú |
| Cobrar (atalho) | `POST /api/pix/cobrar` | BB, Itaú |
| Webhook receber | `POST /pix/{bank}/webhook/pix` | BB |
| Poll notificações | `GET /pix/notifications/{txid}/poll` | BB |

Parâmetro `{bank}`: `bb` → Banco do Brasil · `itau` → Itaú · `pagbank` → PagBank (pendente)

### Stack técnica

| Item | Valor |
|---|---|
| Framework | NestJS |
| HTTP Adapter | Fastify |
| Porta | `3006` |
| Linguagem | TypeScript |
| Node | LTS |
| Frontend | Angular 21 + Tailwind CSS v4 (pasta `viapix_fe/`) |
| SGBD | PostgreSQL |
| ORM | TypeORM |

**Pacotes principais:**

| Pacote | Uso |
|---|---|
| `@nestjs/platform-fastify` | Adapter HTTP |
| `@nestjs/typeorm` + `typeorm` + `pg` | ORM + driver |
| `@nestjs/swagger` | OpenAPI |
| `class-validator` + `class-transformer` | Validação de DTOs |
| `joi` | Validação de env vars |
| `axios` | Chamadas HTTP para APIs dos bancos |
| `node-forge` | Criptografia RSA + AES |

### Variáveis de ambiente

Validação feita com Joi em `src/config/env.config.ts`. Variáveis ausentes ou inválidas causam falha na inicialização.

**Gerais:**

| Variável | Descrição | Obrigatória |
|---|---|---|
| `NODE_ENV` | Ambiente (`development`, `production`) | ✅ |
| `PORT` | Porta da aplicação (padrão `3006`) | — |
| `VIAPIX_API_KEY` | Chave de autenticação da API ViaPIX | ✅ |

**Banco de dados:**

| Variável | Descrição | Obrigatória |
|---|---|---|
| `DB_HOST` | Host PostgreSQL | ✅ |
| `DB_PORT` | Porta PostgreSQL (padrão `5432`) | — |
| `DB_USER` | Usuário PostgreSQL | ✅ |
| `DB_PASS` | Senha PostgreSQL | ✅ |
| `DB_NAME` | Nome do banco | ✅ |

**Banco do Brasil:**

| Variável | Descrição | Obrigatória |
|---|---|---|
| `BB_PIX_URL` | URL base da API PIX BB | ✅ |
| `BB_OAUTH_URL` | URL do OAuth2 BB | ✅ |
| `BB_GW_APP_KEY` | App key do developer.bb.com.br | ✅ |

**Itaú** (temporário — migração para DB pendente):

| Variável | Descrição | Obrigatória |
|---|---|---|
| `ITAU_CLIENT_ID` | Client ID Itaú | ✅ |
| `ITAU_CLIENT_SECRET` | Client Secret Itaú | ✅ |
| `ITAU_CHAVE_PIX` | Chave PIX Itaú | ✅ |
| `ITAU_PIX_URL` | URL base da API PIX Itaú | ✅ |
| `ITAU_OAUTH_URL` | URL do OAuth2 Itaú (STS APIGateway) | ✅ |

**PagBank:**

| Variável | Descrição | Obrigatória |
|---|---|---|
| `PAGBANK_OWNER_TOKEN` | Token global do dono da conta PagBank | ✅ |

## Seções

- [[02-infrastructure/banks/_index]] — integrações bancárias (BB, Itaú, PagBank)
- [[02-infrastructure/database/_index]] — schema e migrations
- [[02-infrastructure/security/_index]] — autenticação e criptografia
- [[02-infrastructure/external-apis/_index]] — APIs externas

## Relacionados

- [[00-index]] — índice geral do vault
