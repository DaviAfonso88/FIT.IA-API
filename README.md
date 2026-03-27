# Treinos API

API RESTful para gestão de planos de treino personalizados com assistente de IA. Desenvolvida para o projeto Bootcamp Treinos do FSC.

[![Node.js](https://img.shields.io/badge/Node.js-24.x-green?style=flat&logo=node.js)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-blue?style=flat&logo=typescript)](https://www.typescriptlang.org/)
[![Fastify](https://img.shields.io/badge/Fastify-5.7-purple?style=flat)](https://fastify.dev/)
[![Prisma](https://img.shields.io/badge/Prisma-7.4-2D3748?style=flat&logo=prisma)](https://www.prisma.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat&logo=postgresql)](https://www.postgresql.org/)

---

## Sumário

1. [Sobre o Projeto](#sobre-o-projeto)
2. [Tech Stack](#tech-stack)
3. [Arquitetura](#arquitetura)
4. [Estrutura do Projeto](#estrutura-do-projeto)
5. [Fluxo de Requisição](#fluxo-de-requisição)
6. [Banco de Dados](#banco-de-dados)
7. [Autenticação](#autenticação)
8. [Recursos Principais](#recursos-principais)
9. [Endpoints](#endpoints)
10. [Variáveis de Ambiente](#variáveis-de-ambiente)
11. [Primeiros Passos](#primeiros-passos)
12. [Docker](#docker)
13. [Convenções de Código](#convenções-de-código)
14. [Tratamento de Erros](#tratamento-de-erros)

---

## Sobre o Projeto

### O que é esta API?

A **Treinos API** é uma backend API completa para gerenciamento de planos de treino personalizados. Ela resolve um problema comum: a dificuldade de criar planos de treino personalizados sem um personal trainer.

**O problema:** Tradicionalmente, usuários dependiam de:
- Personal trainers presenciais (caro e com horários limitados)
- Planos genéricos da internet (não consideram individualidade)
- Apps com planos pré-definidos (pouca flexibilidade)

**A solução:** Esta API utiliza **Inteligência Artificial** (Google Gemini) para criar planos 100% personalizados baseados em:

- Dados antropométricos (peso, altura, idade, % gordura corporal)
- Objetivos do usuário (hipertrofia, emagrecimento, força)
- Disponibilidade de tempo (dias por semana)
- Restrições físicas e lesões
- Equipamentos disponíveis

### Funcionalidades Principais

- **Autenticação via Google OAuth** - Login social sem necessidade de criar senhas
- **Chat com IA Personal Trainer** - Conversa natural para criar e ajustar planos
- **Sistema de Planos Semanais** - Planos de 7 dias com exercícios, séries e repetições
- **Tracking de Sessões** - Registrar início e fim de treinos com tempo
- **Estatísticas** - Streak, consistência e evolução do usuário
- **Dados Antropométricos** - Peso, altura, idade e % gordura corporal

### Para quem é?

- **Usuários finais** que querem treinos personalizados sem pagar um personal trainer
- **Desenvolvedores** que querem integrar um sistema de treinos em seus apps
- **Estudantes** do Bootcamp Treinos FSC que querem aprender a construir APIs modernas

---

## Tech Stack

### Por que estas tecnologias?

| Tecnologia | Versão | Por que usar? |
| ---------- | ------ | ------------- |
| **Node.js** | 24.x | Runtime moderno com performance excelente para I/O bound |
| **pnpm** | 10.30.0 | Gerenciador rápido com espaço em disco reduzido |
| **Fastify** | 5.7.4 | 3x mais rápido que Express, com typing nativo |
| **TypeScript** | 5.9.3 | Type safety em tempo de desenvolvimento |
| **PostgreSQL** | 16 | Banco relacional robusto para dados estruturados |
| **Prisma** | 7.4.0 | ORM com migrations, type safety e DX excellent |
| **Better-Auth** | 1.4.18 | Autenticação completa sem reinventar a roda |
| **Zod** | 4.3.6 | Validação de dados com inferência de tipos |
| **AI SDK** | 6.0.100 | Abstrai provedores de IA (Google, OpenAI) |
| **Google Gemini** | 3.0.34 | IA multimodal, rápida e econômica |

---

## Arquitetura

### Padrão Arquitetural: Camadas

Este projeto segue uma **arquitetura em camadas** (também conhecida como Layered Architecture). Mas por que não usar outro padrão?

**Alternativas consideradas:**

| Padrão | Por que rejeitamos |
| ------- | ----------------- |
| MVC | Foco em views, mas temos API REST |
| Clean Architecture | Ótimo, mas pode ser overkill para API simples |
| Hexagonal | Excelente para plugins, não necessário aqui |

**Por que Camadas?**

- **Simplicidade**: Separação clara e intuitiva (Route → Use Case → DB)
- **Testabilidade**: Cada camada pode ser testada isoladamente
- **Manutenibilidade**: Mudanças em uma camada não afetam outras
- **Padrão Fastify**: Integra-se naturalmente com o modelo de plugins

### Visão Geral da Arquitetura

```mermaid
graph TB
    subgraph Client["🖥️ Cliente"]
        Web["Frontend Web<br/>Next.js :3000"]
        Mobile["App Mobile<br/>(futuro)"]
    end

    subgraph API["⚡ Treinos API (:8081)"]
        Routes["📍 Routes<br/>src/routes/"]
        Middleware["🔒 Middleware<br/>Validação, Auth"]
        UseCases["⚙️ Use Cases<br/>src/usecases/"]
        Schemas["📝 Schemas Zod<br/>src/schemas/"]
        Errors["🚨 Errors<br/>src/errors/"]
    end

    subgraph Lib["📚 Lib"]
        Auth["🔐 Better-Auth<br/>src/lib/auth.ts"]
        DB["🗄️ Prisma Client<br/>src/lib/db.ts"]
        Env["⚙️ Environment<br/>src/lib/env.ts"]
    end

    subgraph DB["🗄️ Banco de Dados"]
        PG["🐘 PostgreSQL :5432"]
    end

    subgraph AI["🤖 Serviços Externos"]
        Gemini["✨ Google Gemini AI"]
        Google["🔵 Google OAuth"]
    end

    Web --> Routes
    Mobile --> Routes
    Routes --> Middleware
    Middleware --> Schemas
    Schemas --> UseCases
    UseCases --> Errors
    UseCases --> Auth
    UseCases --> DB
    DB --> PG
    Auth --> Google
    UseCases --> Gemini
```

### Diagrama de Componentes

```mermaid
classDiagram
    class Routes {
        +home.ts
        +me.ts
        +stats.ts
        +workout-plan.ts
        +ai.ts
    }
    
    class UseCases {
        +GetHomeData
        +GetUserTrainData
        +UpsertUserTrainData
        +GetStats
        +ListWorkoutPlans
        +GetWorkoutPlan
        +CreateWorkoutPlan
        +StartWorkoutSession
        +UpdateWorkoutSession
    }
    
    class Lib {
        +auth.ts
        +db.ts
        +env.ts
    }
    
    Routes --> UseCases
    UseCases --> Lib
    UseCases --> Errors
```

---

## Estrutura do Projeto

```
treinos-api/
├── prisma/
│   ├── schema.prisma              # Schema do banco de dados
│   └── migrations/                # Migrations do Prisma
│
├── src/
│   ├── index.ts                  # 🎯 Entry point da aplicação
│   │
│   ├── errors/                    # Classes de erro customizadas
│   │   └── index.ts
│   │       ├── NotFoundError
│   │       ├── WorkoutPlanNotActiveError
│   │       └── SessionAlreadyStartedError
│   │
│   ├── generated/                # Tipos gerados automaticamente
│   │   └── prisma/              # Tipos Prisma (prisma generate)
│   │
│   ├── lib/                      # Configurações globais
│   │   ├── auth.ts              # Better-Auth configuration
│   │   ├── db.ts                # Prisma client instance
│   │   └── env.ts               # Variáveis de ambiente tipadas
│   │
│   ├── routes/                   # 🎯 Handlers de rotas Fastify
│   │   ├── home.ts              # GET /home
│   │   ├── me.ts                # GET/PATCH /me
│   │   ├── stats.ts             # GET /stats
│   │   ├── workout-plan.ts      # CRUD /workout-plans
│   │   └── ai.ts                # POST /ai (chat)
│   │
│   ├── schemas/                  # Schemas Zod compartilhados
│   │   └── index.ts
│   │
│   └── usecases/                 # 🎯 Lógica de negócio
│       ├── GetHomeData.ts
│       ├── GetUserTrainData.ts
│       ├── UpsertUserTrainData.ts
│       ├── GetStats.ts
│       ├── ListWorkoutPlans.ts
│       ├── GetWorkoutPlan.ts
│       ├── GetWorkoutDay.ts
│       ├── CreateWorkoutPlan.ts
│       ├── StartWorkoutSession.ts
│       └── UpdateWorkoutSession.ts
│
├── docker-compose.yml            # PostgreSQL + pgAdmin
├── Dockerfile                    # Container da API
├── package.json
├── tsconfig.json
├── pnpm-lock.yaml
└── README.md
```

### Responsabilidades por Diretório

| Diretório | O que faz | Quem usa |
| ---------- | --------- | -------- |
| `src/routes/` | Recebe requisições HTTP, retorna respostas | Fastify |
| `src/usecases/` | Contém lógica de negócio | Routes |
| `src/schemas/` | Define formato de dados (input/output) | Routes, Use Cases |
| `src/errors/` | Define erros customizados | Use Cases |
| `src/lib/` | Configurações compartilhadas | Todas as camadas |
| `src/generated/` | Tipos gerados (não editar manualmente) | TypeScript |

---

## Fluxo de Requisição

### Exemplo: Criar Plano de Treino via Chat IA

Vamos rastrear uma requisição do início ao fim:

```mermaid
sequenceDiagram
    participant User as Usuário
    participant Front as Frontend
    participant API as API Fastify
    participant Route as /ai route
    participant UseCase as CreateWorkoutPlan
    participant Prisma as Prisma
    participant DB as PostgreSQL

    User->>Front: "Crie um plano de treino"
    Front->>API: POST /ai { messages: [...] }
    API->>Route: Direciona para route handler
    Route->>Route: Valida sessão (auth.api.getSession)
    Route->>UseCase: Chama tool createWorkoutPlan
    UseCase->>Prisma: $transaction
    Prisma->>DB: UPDATE existing plans (isActive=false)
    Prisma->>DB: INSERT workoutPlan
    Prisma->>DB: INSERT workoutDays
    Prisma->>DB: INSERT workoutExercises
    DB-->>Prisma: Dados criados
    Prisma-->>UseCase: WorkoutPlan completo
    UseCase-->>Route: Retorna plano criado
    Route-->>API: Stream response
    API-->>Front: Streaming AI response
    Front-->>User: "Plano criado com sucesso!"
```

### Passo a Passo no Código

#### 1. Entry Point (`src/index.ts:84-90`)

As rotas são registradas como plugins Fastify:

```typescript
// src/index.ts:84-90
// RESTful Routes
await app.register(homeRoutes, { prefix: "/home" });
await app.register(meRoutes, { prefix: "/me" });
await app.register(statsRoutes, { prefix: "/stats" });
await app.register(workoutPlanRoutes, { prefix: "/workout-plans" });
await app.register(aiRoutes, { prefix: "/ai" });
```

#### 2. Route Handler (`src/routes/ai.ts:78-92`)

A rota valida a sessão e direciona para o use case:

```typescript
// src/routes/ai.ts:78-92
app.withTypeProvider<ZodTypeProvider>().route({
  method: "POST",
  url: "/",
  handler: async (request, reply) => {
    // 1. Extrai sessão do cookie
    const session = await auth.api.getSession({
      headers: fromNodeHeaders(request.headers),
    });

    // 2. Verifica autenticação
    if (!session) {
      return reply.status(401).send({ error: "Unauthorized" });
    }

    const userId = session.user.id;
    // 3. Processa com IA...
  },
});
```

#### 3. Use Case (`src/usecases/CreateWorkoutPlan.ts`)

A lógica de negócio é encapsulada em classes:

```typescript
// src/usecases/CreateWorkoutPlan.ts (simplificado)
export class CreateWorkoutPlanUseCase {
  async execute(dto: CreateWorkoutPlanDTO) {
    return this.prisma.$transaction(async (tx) => {
      // 1. Desativa planos ativos existentes
      await tx.workoutPlan.updateMany({
        where: { userId: dto.userId, isActive: true },
        data: { isActive: false },
      });

      // 2. Cria novo plano
      return tx.workoutPlan.create({
        data: {
          name: dto.name,
          userId: dto.userId,
          isActive: true,
          workoutDays: {
            create: dto.workoutDays,
          },
        },
      });
    });
  }
}
```

---

## Banco de Dados

### Schema do Banco

O schema do Prisma (`prisma/schema.prisma`) define a estrutura do banco:

```mermaid
erDiagram
    USER ||--o{ WORKOUT_PLAN : "has"
    USER ||--o{ SESSION : "has"
    USER ||--o{ ACCOUNT : "has"
    WORKOUT_PLAN ||--o{ WORKOUT_DAY : "contains"
    WORKOUT_DAY ||--o{ WORKOUT_EXERCISE : "has"
    WORKOUT_DAY ||--o{ WORKOUT_SESSION : "has"
```

A relação entre entidades:

| Entidade | Relacionamento | Entidade |
|----------|---------------|----------|
| User | 1:N | WorkoutPlan |
| User | 1:N | Session |
| User | 1:N | Account |
| WorkoutPlan | 1:N | WorkoutDay |
| WorkoutDay | 1:N | WorkoutExercise |
| WorkoutDay | 1:N | WorkoutSession |

### Relacionamentos

| Relacionamento | Descrição |
| ---------------| --------- |
| User → WorkoutPlan | Um usuário pode ter múltiplos planos |
| User → Session | Um usuário pode ter múltiplas sessões ativas |
| User → Account | Um usuário pode ter múltiplas contas OAuth |
| WorkoutPlan → WorkoutDay | Um plano tem exatamente 7 dias |
| WorkoutDay → WorkoutExercise | Um dia tem 0-N exercícios |
| WorkoutDay → WorkoutSession | Um dia pode ter múltiplas sessões |

### Decisões de Design

**Por que PostgreSQL e não MongoDB?**

- **Dados estruturados**: Planos de treino têm estrutura fixa (dias → exercícios)
- **Relacionamentos**: Forte relação entre entidades (User → Plan → Day → Exercise)
- **Transações**: Precisamos de atomicidade ao criar planos (Prisma $transaction)

**Por que Prisma e não Drizzle?**

- Prisma tem DX excellent com migrations e generated types
- Interface fluente intuitiva
- Suporte nativo a PostgreSQL com good PostgreSQL features

---

## Autenticação

### Como a Autenticação Funciona

A autenticação é handled pelo **Better-Auth**, uma biblioteca completa que abstrai toda a complexidade de OAuth e gerenciamento de sessões.

#### Fluxo OAuth com Google

```mermaid
flowchart LR
    A[Usuário] -->|1| B[Frontend]
    B -->|2| C[GET /api/auth/oauth/google]
    C -->|3| D[Redirect Google OAuth]
    D -->|4| E[Usuário autoriza]
    E -->|5| F[Callback /api/auth/oauth/google/callback]
    F -->|6| G[Cria Session + Account no DB]
    G -->|7| H[Set-Cookie: better-auth.session_token]
    H -->|8| I[Redirect /dashboard]
```

#### Configuração do Better-Auth

```typescript
// src/lib/auth.ts
export const auth = betterAuth({
  // URL base da API (usada para gerar links)
  baseURL: env.API_BASE_URL,
  
  // Origins confiáveis (frontend permitido)
  trustedOrigins: [env.WEB_APP_BASE_URL],
  
  // Provedores sociais
  socialProviders: {
    google: {
      prompt: "select_account",  // Força usuário escolher conta
      clientId: env.GOOGLE_CLIENT_ID,
      clientSecret: env.GOOGLE_CLIENT_SECRET,
    },
  },
  
  // Adapter para PostgreSQL
  database: prismaAdapter(prisma, {
    provider: "postgresql",
  }),
  
  // Gera spec OpenAPI para rotas de auth
  plugins: [openAPI()],
  
  // Cross-domain cookies para produção
  advanced: {
    crossSubDomainCookies: {
      enabled: true,
      domain: env.NODE_ENV === "production" ? ".davidev.net.br" : undefined,
    },
  },
});
```

#### Protegendo Rotas

Todas as rotas que precisam de autenticação verificam a sessão:

```typescript
// src/routes/ai.ts:86-92
const session = await auth.api.getSession({
  headers: fromNodeHeaders(request.headers),
});

if (!session) {
  return reply.status(401).send({ error: "Unauthorized" });
}

// A partir daqui, temos acesso ao usuário:
// session.user.id
// session.user.email
// session.user.name
// session.user.image
```

### Sessão via Cookies

```
Set-Cookie: better-auth.session_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...;
            HttpOnly;
            SameSite=Lax;
            Path=/;
            Secure;
            Expires=Mon, 15 Jan 2024 12:00:00 GMT
```

| Atributo | Valor | Por que? |
|----------|-------|----------|
| `HttpOnly` | true | Previne XSS (JS não acessa cookie) |
| `SameSite` | Lax | Permite navegação normal |
| `Path` | / | Disponível em toda API |
| `Secure` | true (prod) | Apenas HTTPS em produção |

### Rotas de Autenticação

Todas as rotas são expostas em `/api/auth/*`:

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/auth/sign-in` | Iniciar login |
| POST | `/api/auth/sign-up` | Criar conta |
| POST | `/api/auth/sign-out` | Logout |
| GET | `/api/auth/get-session` | Obter sessão atual |
| GET | `/api/auth/oauth/google` | Login Google |
| GET | `/api/auth/oauth/google/callback` | Callback Google |

### Segurança

- **Tokens seguros**: UUIDs aleatórios como tokens de sessão
- **Expiration**: Sessões expiram automaticamente
- **HttpOnly cookies**: Previne XSS attacks
- **SameSite=Lax**: Previne CSRF em navegação normal
- **Cross-domain**: Suporte a subdomínios em produção

---

## Recursos Principais

### 1. Chat com IA Personal Trainer

O coração do sistema. O chat utiliza **Google Gemini** com tools para:

```mermaid
flowchart TB
    A[Usuário] -->|Mensagem| B[Gemini]
    B -->|tool: getUserTrainData| C[Dados do usuário]
    C -->|retorna| B
    B -->|tool: updateUserTrainData| D[Salva dados]
    D -->|retorna| B
    B -->|tool: createWorkoutPlan| E[Cria plano]
    E -->|retorna| B
    B -->|Resposta| A
```

**Tools disponíveis para a IA:**

| Tool | Função |
|------|--------|
| `getUserTrainData` | Busca peso, altura, idade, % gordura |
| `updateUserTrainData` | Salva dados antropométricos |
| `getWorkoutPlans` | Lista planos do usuário |
| `createWorkoutPlan` | Cria novo plano de 7 dias |

**Prompt do Sistema (`src/routes/ai.ts:21-75`):**

```typescript
const SYSTEM_PROMPT = `Você é um personal trainer virtual...

## Regras de Interação

1. SEMPRE chame getUserTrainData antes de qualquer interação
2. Se não tem dados, pergunte peso, altura, idade, % gordura
3. Se já tem dados, cumprimente pelo nome

## Criação de Plano
- Plano DEVE ter exatamente 7 dias (MONDAY a SUNDAY)
- isRest: true para dias de descanso
- Escolha split baseado em dias disponíveis...`;
```

### 2. Sistema de Planos Semanais

- **7 dias exatos**: Monday a Sunday
- **Dias de descanso**: Suportados com `isRest: true`
- **Exercícios estruturados**: ordem, nome, séries, reps, descanso
- **Imagens de capa**: URLs para cada dia

### 3. Tracking de Sessões

```mermaid
stateDiagram-v2
    [*] --> Inativo
    Inativo --> Ativo: POST /workout-sessions/start
    Ativo --> Completo: PATCH /workout-sessions/:id
    Completo --> [*]
    
    Ativo --> Expirado: Timeout 2h
    Expirado --> [*]
```

- Iniciar sessão: `POST /workout-sessions/start`
- Finalizar sessão: `PATCH /workout-sessions/:id`
- Tempo é calculado automaticamente

### 4. Estatísticas

- **Streak**: Dias consecutivos treinando
- **Consistência**: % dias treinados na semana
- **Taxa de conclusão**: Iniciado vs completado

---

## Endpoints

### Autenticação

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| \* | `/api/auth/*` | Todas as rotas do Better-Auth |

### Rotas da API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/home` | Dashboard: plano ativo, dia atual |
| GET | `/me` | Dados do usuário logado |
| GET | `/me/train-data` | Dados antropométricos |
| PATCH | `/me/train-data` | Atualiza dados antropométricos |
| GET | `/stats` | Estatísticas (streak, consistência) |
| GET | `/workout-plans` | Lista todos os planos |
| POST | `/workout-plans` | Cria novo plano (admin only?) |
| GET | `/workout-plans/:id` | Detalhes de um plano |
| GET | `/workout-days/:id` | Detalhes de um dia |
| POST | `/workout-sessions/start` | Inicia sessão |
| PATCH | `/workout-sessions/:id` | Finaliza sessão |
| POST | `/ai` | Chat com IA (streaming) |

### Documentação

| Endpoint | Descrição |
|----------|-----------|
| `/docs` | UI interativa Scalar (:8081) |
| `/swagger.json` | Spec OpenAPI |

> A API roda na porta **8081** em desenvolvimento.

---

## Variáveis de Ambiente

### Arquivo `.env`

```env
# ====================
# SERVIDOR
# ====================
PORT=8081
NODE_ENV=development

# ====================
# BANCO DE DADOS
# ====================
DATABASE_URL="postgresql://postgres:password@localhost:5432/treinos-api"

# ====================
# AUTENTICAÇÃO
# ====================
BETTER_AUTH_SECRET="generate-with: openssl rand -base64 32"
API_BASE_URL="http://localhost:8081"
WEB_APP_BASE_URL="http://localhost:3000"

# ====================
# GOOGLE OAUTH
# ====================
GOOGLE_CLIENT_ID="seu-google-client-id"
GOOGLE_CLIENT_SECRET="seu-google-client-secret"

# ====================
# GOOGLE AI (GEMINI)
# ====================
GOOGLE_GENERATIVE_AI_API_KEY="sua-chave-api-gemini"

# ====================
# OPENAI (OPCIONAL)
# ====================
# OPENAI_API_KEY="sk-..."
```

### Descrição das Variáveis

| Variável | Tipo | Obrigatório | Descrição |
|----------|------|-------------|-----------|
| `PORT` | number | Sim | Porta do servidor |
| `NODE_ENV` | string | Sim | Ambiente (development/production/test) |
| `DATABASE_URL` | string | Sim | String de conexão PostgreSQL |
| `BETTER_AUTH_SECRET` | string | Sim | Chave para criptografar sessões |
| `API_BASE_URL` | string | Sim | URL base da API |
| `WEB_APP_BASE_URL` | string | Sim | URL do frontend |
| `GOOGLE_CLIENT_ID` | string | Sim | OAuth Client ID do Google Cloud |
| `GOOGLE_CLIENT_SECRET` | string | Sim | OAuth Client Secret |
| `GOOGLE_GENERATIVE_AI_API_KEY` | string | Sim | API Key do Google AI Studio |
| `OPENAI_API_KEY` | string | Não | API Key OpenAI (alternativa) |

---

## Primeiros Passos

### Pré-requisitos

- **Node.js 24.x** (obrigatório - verificado via `engine-strict`)
- **pnpm 10.30.0** (obrigatório - verificado via `packageManager`)
- **Docker** (para PostgreSQL)

### Instalação

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/treinos-api.git
cd treinos-api

# 2. Instale dependências
pnpm install

# 3. Copie o arquivo de exemplo
cp .env.example .env
```

### Configuração do Banco

```bash
# 4. Iniciar PostgreSQL via Docker
docker-compose up -d

# 5. Criar migrations
pnpm exec prisma migrate dev

# 6. Gerar tipos do Prisma
pnpm exec prisma generate
```

### Executar

```bash
# Desenvolvimento (hot-reload na porta 8081)
pnpm dev
```

A API estará disponível em:
- **API**: http://localhost:8081
- **Docs**: http://localhost:8081/docs

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

---

## Docker

### Docker Compose

O `docker-compose.yml` inicia apenas o PostgreSQL:

```bash
docker-compose up -d
```

### Dockerfile

Para build de produção:

```bash
# Build da imagem
docker build -t treinos-api .

# Executar container
docker run -p 8081:8081 --env-file .env treinos-api
```

---

## Convenções de Código

### Padrões de Design

| Padrão | Onde Usar | Por quê |
|--------|-----------|---------|
| **Use Case** | `src/usecases/` | Encapsula lógica de negócio |
| **DTO** | Parâmetros de Use Cases | Define formato de entrada |
| **Schema Zod** | `src/schemas/` | Validação e tipagem de API |
| **Errors Custom** | `src/errors/` | Erros específicos do domínio |

### Estrutura de um Use Case

```typescript
// src/usecases/NomeDoUseCase.ts
export class NomeDoUseCase {
  constructor(private prisma: PrismaClient) {}

  async execute(dto: NomeDoUseCaseDTO): Promise<ReturnType> {
    // 1. Validação de regras de negócio
    
    // 2. Transação (se necessário)
    return this.prisma.$transaction(async (tx) => {
      // 3. Operações no banco
    });
  }
}
```

### Zod Schemas

```typescript
// ✅ Correto (usa z.interface)
export const createWorkoutPlanSchema = z.interface({
  name: z.string().min(1),
  workoutDays: z.array(workoutDaySchema),
});

// ❌ Evitar (z.object é para objetos anônimos)
export const badSchema = z.object({
  name: z.string(),
});
```

### Imports

O projeto usa `simple-import-sort` para ordenação automática:

```typescript
// 1. Built-in do Node
import path from "path";
import fs from "fs";

// 2. Pacotes externos
import { betterAuth } from "better-auth";
import { z } from "zod";

// 3. Módulos internos (src/)
import { auth } from "./lib/auth";
import { CreateWorkoutPlan } from "./usecases/CreateWorkoutPlan";

// 4. Tipos gerados
import { WeekDay } from "../generated/prisma/enums";
```

---

## Tratamento de Erros

### Classes de Erro Customizadas

O projeto define erros específicos do domínio em `src/errors/index.ts`:

```typescript
// src/errors/index.ts
export class NotFoundError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "NotFoundError";
  }
}

export class WorkoutPlanNotActiveError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "WorkoutPlanNotActiveError";
  }
}

export class SessionAlreadyStartedError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "SessionAlreadyStartedError";
  }
}
```

### Como Usar

```typescript
// Em um Use Case
const workoutPlan = await this.prisma.workoutPlan.findUnique({
  where: { id },
});

if (!workoutPlan) {
  throw new NotFoundError(`Plano de treino ${id} não encontrado`);
}

if (!workoutPlan.isActive) {
  throw new WorkoutPlanNotActiveError("Este plano não está ativo");
}
```

### Validação com Zod

As rotas usam Zod para validar requests automaticamente:

```typescript
// src/routes/workout-plan.ts
app.withTypeProvider<ZodTypeProvider>().route({
  method: "POST",
  url: "/",
  schema: {
    body: createWorkoutPlanSchema,  // Valida input
    response: {
      201: workoutPlanResponseSchema,  // Valida output
    },
  },
  handler: async (request, reply) => {
    // request.body já está validado
    const data = request.body;
  },
});
```

---

## Como Contribuir

### Setup de Desenvolvimento

```bash
# Fork no GitHub
# Clone seu fork
git clone https://github.com/seu-usuario/treinos-api.git

# Crie uma branch
git checkout -b feature/nova-funcionalidade

# Instale dependências
pnpm install

# Execute os testes (quando houver)
pnpm test

# Commit com Conventional Commits
git commit -m "feat: adiciona nova funcionalidade"

# Push e abra PR
git push origin feature/nova-funcionalidade
```

### Convenções

- **Commits**: [Conventional Commits](https://www.conventionalcommits.org/)
- **Branches**: `feature/`, `bugfix/`, `hotfix/`
- **Code Style**: ESLint + Prettier
- **Schemas Zod**: Usar `z.interface()` (não `z.object()`)

---

Desenvolvido com Fastify, TypeScript e PostgreSQL.
