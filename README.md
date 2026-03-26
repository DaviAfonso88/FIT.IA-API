# API FIT.IA

API RESTful para gestão de planos de treino personalizados com assistente de IA. Desenvolvida para o projeto Bootcamp Treinos do FSC.

[![Node.js](https://img.shields.io/badge/Node.js-24.x-green?style=flat&logo=node.js)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-blue?style=flat&logo=typescript)](https://www.typescriptlang.org/)
[![Fastify](https://img.shields.io/badge/Fastify-5.7-purple?style=flat)](https://fastify.dev/)
[![Prisma](https://img.shields.io/badge/Prisma-7.4-2D3748?style=flat&logo=prisma)](https://www.prisma.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat&logo=postgresql)](https://www.postgresql.org/)

## Sumário

- [Visão Geral](#visão-geral)
- [Tech Stack](#tech-stack)
- [Arquitetura](#arquitetura)
- [Modelos de Dados](#modelos-de-dados)
- [Autenticação](#autenticação)
- [Endpoints](#endpoints)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Primeiros Passos](#primeiros-passos)
- [Docker](#docker)
- [Recursos Principais](#recursos-principais)

## Visão Geral

A **Treinos API** é uma backend API completa para gerenciamento de planos de treino personalizados. O projeto inclui:

- **Autenticação** via Better-Auth com login social (Google)
- **Criação Inteligente de Planos** através de chat com IA (Google Gemini)
- **Gestão de Treinos** com dias de treino e exercícios estruturados
- **Tracking de Sessões** para acompanhamento do progresso
- **Estatísticas** de evolução e consistência
- **Documentação Interativa** via Scalar UI

## Tech Stack

### Runtime & Package Manager

| Tecnologia | Versão  | Descrição                                                 |
| ---------- | ------- | --------------------------------------------------------- |
| Node.js    | 24.x    | Runtime JavaScript (obrigatório via `engine-strict`)      |
| pnpm       | 10.30.0 | Gerenciador de pacotes (obrigatório via `packageManager`) |

### Framework & Linguagem

| Tecnologia | Versão | Descrição                                    |
| ---------- | ------ | -------------------------------------------- |
| Fastify    | 5.7.4  | Framework web Node.js de alta performance    |
| TypeScript | 5.9.3  | Linguagem com tipagem estática (strict mode) |

### Banco de Dados

| Tecnologia         | Versão | Descrição                        |
| ------------------ | ------ | -------------------------------- |
| PostgreSQL         | 16     | Banco de dados relacional        |
| Prisma             | 7.4.0  | ORM com migrations e type safety |
| @prisma/adapter-pg | 7.4.0  | Adaptador Prisma para PostgreSQL |

### Autenticação & Sessão

| Tecnologia                  | Versão | Descrição                         |
| --------------------------- | ------ | --------------------------------- |
| Better-Auth                 | 1.4.18 | Autenticação completa com sessões |
| better-auth/adapters/prisma | -      | Adaptador Prisma para Better-Auth |

### Validação & Tipagem

| Tecnologia                | Versão | Descrição                    |
| ------------------------- | ------ | ---------------------------- |
| Zod                       | 4.3.6  | Validação de dados e schemas |
| fastify-type-provider-zod | 6.1.0  | Integração Zod com Fastify   |

### AI & Machine Learning

| Tecnologia     | Versão  | Descrição                  |
| -------------- | ------- | -------------------------- |
| ai             | 6.0.100 | SDK unificado para IA      |
| @ai-sdk/google | 3.0.34  | Provedor Google Gemini     |
| @ai-sdk/openai | 3.0.33  | Provedor OpenAI (opcional) |

### Documentação da API

| Tecnologia                    | Versão  | Descrição                |
| ----------------------------- | ------- | ------------------------ |
| @fastify/swagger              | 9.7.0   | Geração de specs OpenAPI |
| @scalar/fastify-api-reference | 1.44.20 | UI interativa Scalar     |

### Logging & Utilities

| Tecnologia  | Versão  | Descrição                               |
| ----------- | ------- | --------------------------------------- |
| pino-pretty | 13.1.3  | Formatação de logs para desenvolvimento |
| dayjs       | 1.11.19 | Manipulação de datas                    |
| dotenv      | 17.3.1  | Carregamento de variáveis de ambiente   |

### Linting & Formatting

| Tecnologia                       | Versão | Descrição                  |
| -------------------------------- | ------ | -------------------------- |
| ESLint                           | 9.39.2 | Análise estática de código |
| typescript-eslint                | 8.56.0 | Integração ESLint para TS  |
| prettier                         | 3.8.1  | Formatação de código       |
| eslint-config-prettier           | 10.1.8 | Conflitos ESLint/Prettier  |
| eslint-plugin-simple-import-sort | 12.1.1 | Ordenação de imports       |
| @eslint/js                       | 10.0.1 | Configuração base ESLint   |
| globals                          | 17.3.0 | Definições de globals      |

### Desenvolvimento

| Tecnologia  | Versão | Descrição                          |
| ----------- | ------ | ---------------------------------- |
| tsx         | 4.21.0 | Execução TypeScript com hot-reload |
| @types/node | 24     | Tipos Node.js                      |

## Arquitetura

O projeto segue uma **arquitetura em camadas** com separação clara de responsabilidades:

```
Routes → Use Cases → Prisma
```

### Estrutura de Diretórios

```
src/
├── errors/          # Classes de erro customizadas
├── generated/      # Tipos gerados pelo Prisma
├── lib/            # Configurações (auth, db, env)
├── routes/         # Handlers de rotas Fastify
├── schemas/        # Schemas Zod compartilhados
├── usecases/       # Lógica de negócio
└── index.ts        # Ponto de entrada
```

### Camadas

#### 1. Routes (`src/routes/`)

- Handlers de rotas Fastify
- Registram schemas Zod para validação request/response
- Extraem sessão de autenticação
- Definem status HTTP

#### 2. Use Cases (`src/usecases/`)

- Classes de lógica de negócio
- Recebem DTOs (Data Transfer Objects)
- Usam transações Prisma para atomicidade
- Uma classe por caso de uso

#### 3. Prisma

- Acesso ao banco de dados
- Tipos gerados automaticamente
- Migrations para versionamento do schema

### Padrão de Requisição

```
1. Cliente → Route (valida schema Zod)
2. Route → Use Case (executa lógica)
3. Use Case → Prisma (acesso a dados)
4. Prisma → Banco PostgreSQL
5. Retorno inverso até o cliente
```

## Modelos de Dados

O schema do Prisma define os seguintes modelos:

### Modelos de Usuário

```prisma
User {
  id                  String
  name                String
  email               String
  emailVerified       Boolean
  image               String?
  weightInGrams      Int?
  heightInCentimeters Int?
  age                 Int?
  bodyFatPercentage   Int?
}
```

### Modelos de Treino

```prisma
WorkoutPlan {
  id          String
  name        String
  userId      String
  isActive    Boolean
  workoutDays WorkoutDay[]
}

WorkoutDay {
  id                         String
  name                       String
  weekDay                    WeekDay (MONDAY-SUNDAY)
  isRest                     Boolean
  estimatedDurationInSeconds Int
  coverImageUrl              String?
  exercises                  WorkoutExercise[]
  sessions                   WorkoutSession[]
}

WorkoutExercise {
  id                String
  name              String
  order             Int
  sets              Int
  reps              Int
  restTimeInSeconds Int
}

WorkoutSession {
  id           String
  startedAt    DateTime
  completedAt  DateTime?
}
```

### Modelos de Autenticação (Better-Auth)

```prisma
Session {
  id        String
  expiresAt DateTime
  token     String
  userId    String
}

Account {
  id                    String
  accountId             String
  providerId            String
  userId                String
  accessToken           String?
  refreshToken         String?
}

Verification {
  id         String
  identifier String
  value      String
  expiresAt  DateTime
}
```

## Autenticação

A autenticação é handled pelo **Better-Auth**, uma biblioteca completa de autenticação para aplicações modernas. O sistema utiliza **cookies HTTP** para manter a sessão do usuário de forma segura.

### Como Funciona

#### 1. Login Social (Google OAuth)

O projeto suporta autenticação via Google OAuth. Quando o usuário faz login:

1. Redirecionamento para Google para autorização
2. Google retorna authorization code
3. Backend troca code por tokens de acesso
4. Usuário e conta são criados/atualizados no banco
5. Sessão é criada e cookie é definido

#### 2. Sessão via Cookies

A autenticação utiliza **cookies HTTP** (não localStorage/token no header):

```
Set-Cookie: better-auth.session_token=token_aqui;
            HttpOnly;
            SameSite=Lax;
            Path=/;
            Expires=...
```

**Características do cookie:**

| Atributo   | Valor           | Descrição                                  |
| ---------- | --------------- | ------------------------------------------ |
| `HttpOnly` | true            | Previne XSS - JS não pode acessar o cookie |
| `SameSite` | Lax             | Permite cookies em navegação normal        |
| `Path`     | /               | Disponível em toda a aplicação             |
| `Secure`   | true (produção) | Apenas HTTPS em produção                   |

#### 3. Cross-Domain Cookies

Em produção, o sistema configura **cross-domain cookies** para compartilhar sessão entre diferentes subdomínios:

```typescript
// src/lib/auth.ts
advanced: {
  crossSubDomainCookies: {
    enabled: true,
    domain: env.NODE_ENV === "production" ? ".davidev.net.br" : undefined,
  },
}
```

Isso permite que `app.davidev.net.br` e `api.davidev.net.br` compartilhem a mesma sessão.

### Rotas de Autenticação

Todas as rotas do Better-Auth são expostas em `/api/auth/*`:

| Método | Endpoint                          | Descrição           |
| ------ | --------------------------------- | ------------------- |
| POST   | `/api/auth/sign-in`               | Iniciar login       |
| POST   | `/api/auth/sign-up`               | Criar conta         |
| POST   | `/api/auth/sign-out`              | Logout              |
| GET    | `/api/auth/get-session`           | Obter sessão atual  |
| POST   | `/api/auth/verify-email`          | Verificar email     |
| POST   | `/api/auth/forgot-password`       | Esqueci minha senha |
| POST   | `/api/auth/reset-password`        | Resetar senha       |
| GET    | `/api/auth/oauth/google`          | Login Google        |
| GET    | `/api/auth/oauth/google/callback` | Callback Google     |

### Protegendo Rotas

As rotas da API verificam a sessão em cada requisição:

```typescript
// Exemplo em src/routes/ai.ts
const session = await auth.api.getSession({
  headers: fromNodeHeaders(request.headers),
});

if (!session) {
  return reply.status(401).send({ error: "Unauthorized" });
}
```

O cookie de sessão é automaticamente enviado pelo navegador em todas as requisições, permitindo que o backend identifique o usuário.

### Fluxo Completo

```
1. Usuário acessa frontend (localhost:3000)
2. Frontend redireciona para /api/auth/oauth/google
3. Backend redireciona para Google
4. Usuário autoriza
5. Google redireciona para /api/auth/oauth/google/callback
6. Backend cria sessão e define cookie
7. Frontend recebe cookie (SameDomain=Lax)
8. Requisições futuras incluem o cookie
9. Backend valida sessão em cada rota protegida
```

### Configuração de Origins

O Better-Auth exige configuração de origins confiáveis:

```typescript
// src/lib/auth.ts
export const auth = betterAuth({
  baseURL: env.API_BASE_URL, // URL da API
  trustedOrigins: [env.WEB_APP_BASE_URL], // Frontend permitido
  // ...
});
```

### Modelo de Dados de Autenticação

#### Session

Armazena sessões ativas dos usuários:

| Campo     | Tipo     | Descrição                 |
| --------- | -------- | ------------------------- |
| id        | String   | UUID da sessão            |
| token     | String   | Token único para a sessão |
| expiresAt | DateTime | Data de expiração         |
| userId    | String   | ID do usuário             |
| ipAddress | String?  | IP do cliente             |
| userAgent | String?  | User agent do navegador   |

#### Account

Armazena contas de provedores OAuth:

| Campo        | Tipo    | Descrição               |
| ------------ | ------- | ----------------------- |
| id           | String  | UUID da conta           |
| accountId    | String  | ID no provedor (Google) |
| providerId   | String  | ID do provedor (google) |
| userId       | String  | ID do usuário local     |
| accessToken  | String? | Token de acesso OAuth   |
| refreshToken | String? | Token de refresh OAuth  |

#### Verification

Armazena códigos de verificação (email, password reset):

| Campo      | Tipo     | Descrição             |
| ---------- | -------- | --------------------- |
| id         | String   | UUID                  |
| identifier | String   | Identificador (email) |
| value      | String   | Código de verificação |
| expiresAt  | DateTime | Data de expiração     |

### Segurança

- **Senhas hasheadas**: Better-Auth usa bcrypt
- **Tokens seguros**: UUIDs aleatórios como tokens
- **Expiration**: Sessões expiram automaticamente
- **HttpOnly cookies**: Previne XSS
- **SameSite**: Previne CSRF
- **Secure cookies**: Apenas HTTPS em produção

## Endpoints

### Autenticação

| Método | Endpoint      | Descrição                     |
| ------ | ------------- | ----------------------------- |
| \*     | `/api/auth/*` | Todas as rotas do Better-Auth |

### Rotas da API

| Método    | Endpoint                  | Descrição                                        |
| --------- | ------------------------- | ------------------------------------------------ |
| GET       | `/home`                   | Dados para tela inicial (plano ativo, dia atual) |
| GET       | `/me`                     | Dados do usuário logado                          |
| GET/PATCH | `/me/train-data`          | Dados antropométricos do usuário                 |
| GET       | `/stats`                  | Estatísticas de treino (streak, consistência)    |
| GET/POST  | `/workout-plans`          | Listar/criar planos de treino                    |
| GET       | `/workout-plans/:id`      | Detalhes de um plano                             |
| GET       | `/workout-days/:id`       | Detalhes de um dia de treino                     |
| POST      | `/workout-sessions/start` | Iniciar sessão de treino                         |
| PATCH     | `/workout-sessions/:id`   | Finalizar sessão de treino                       |
| POST      | `/ai`                     | Chat com IA (Gemini)                             |

### Documentação

| Endpoint        | Descrição            |
| --------------- | -------------------- |
| `/docs`         | UI interativa Scalar |
| `/swagger.json` | Spec OpenAPI         |

## Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
# Servidor
PORT=8080
NODE_ENV=development

# Banco de Dados
DATABASE_URL="postgresql://user:password@localhost:5432/treinos-api"

# Autenticação
BETTER_AUTH_SECRET="sua-chave-secreta-aqui"
API_BASE_URL="http://localhost:8080"
WEB_APP_BASE_URL="http://localhost:3000"

# Google OAuth
GOOGLE_CLIENT_ID="seu-google-client-id"
GOOGLE_CLIENT_SECRET="seu-google-client-secret"

# Google AI (Gemini)
GOOGLE_GENERATIVE_AI_API_KEY="sua-chave-api-gemini"

# OpenAI (opcional)
# OPENAI_API_KEY="sua-chave-openai"
```

## Primeiros Passos

### Pré-requisitos

- Node.js 24.x
- pnpm 10.30.0
- Docker (para PostgreSQL)

### Instalação

```bash
# Instalar dependências
pnpm install
```

### Banco de Dados

```bash
# Iniciar PostgreSQL via Docker
docker-compose up -d

# Criar migrations
pnpm exec prisma migrate dev

# Gerar tipos do Prisma
pnpm exec prisma generate
```

### Executar

```bash
# Desenvolvimento (hot-reload na porta 8080)
pnpm dev
```

### Build

```bash
# Compilar TypeScript
pnpm run build
```

### Linting

```bash
# Verificar código
pnpm exec eslint .

# Formatar código
pnpm exec prettier --write .
```

## Docker

### Dockerfile

O projeto inclui um Dockerfile multi-stage otimizado:

```bash
# Build da imagem
docker build -t treinos-api .

# Executar container
docker run -p 8080:8080 --env-file .env treinos-api
```

### Docker Compose

O `docker-compose.yml` inicia o PostgreSQL:

```bash
docker-compose up -d
```

## Recursos Principais

### 1. Chat com IA Personal Trainer

A API integra o Google Gemini para criar planos de treino personalizados via chat. O assistente:

- Solicita dados antropométricos (peso, altura, idade, % gordura)
- Pergunta sobre objetivo, disponibilidade e restrições
- Cria planos de 7 dias com divisões adequadas (Full Body, PPL, Upper/Lower, etc.)
- Fornece imagens de capa para cada dia de treino

### 2. Sistema de Planos Semanais

- Planos com exatamente 7 dias (Monday a Sunday)
- Suporte a dias de descanso
- Exercícios com séries, repetições e tempo de descanso
- Imagens de capa por foco muscular

### 3. Tracking de Sessões

- Iniciar/finalizar sessões de treino
- Registrar tempo de treino
- Histórico de sessões por dia

### 4. Estatísticas

- Sequência de treinos (streak)
- Consistencia por dia
- Taxa de conclusão
- Tempo total de treino

### 5. Autenticação Segura

- Sessões baseadas em cookies
- Login social Google
- Suporte a cross-domain cookies (produção)

---

Desenvolvido com Fastify, TypeScript e PostgreSQL.
