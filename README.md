# Plataforma EAD — Spring Boot + Angular

Plataforma de educação a distância (EAD) desenvolvida como projeto em grupo das disciplinas de **Programação Orientada a Objetos (POO)** e **Engenharia de Software** — Profa. Andréia D. de Leles (ESPM). Backend em **Spring Boot** (REST, JWT, DDD) e frontend em **Angular**, com gamificação, recompensas e upgrade automático de plano.

## Contexto

A aplicação modela uma plataforma de cursos online por assinatura com mecânicas de gamificação:

- alunos com assinatura básica acessam um conjunto de cursos mensalmente;
- a cada curso concluído com média acima de `7,0`, o aluno tem direito a realizar mais 3 cursos;
- o aluno mais ativo no fórum (mais tópicos e comentários) ganha 1 curso ao fim do mês;
- ao concluir 12 cursos, o plano é promovido para **Premium**, desbloqueando participação em projetos reais e moedas por curso concluído.

## Stack

### Backend

| Categoria | Tecnologia | Versão |
|---|---|---|
| Framework | Spring Boot | 3.3.5 |
| Linguagem | Java | 17 |
| ORM | Hibernate | 6.5.3 |
| Persistência | Spring Data JPA | — |
| Segurança | Spring Security + jjwt | 0.11.5 |
| API Docs | SpringDoc OpenAPI | 2.5.0 |
| Build | Maven | 3.9.9 |
| Banco de desenvolvimento | H2 (memória) | — |
| Banco de produção | PostgreSQL | 16 |

### Frontend

| Categoria | Tecnologia | Versão |
|---|---|---|
| Framework | Angular | 19.2 |
| Linguagem | TypeScript | 5.7 |
| HTTP | HttpClient + RxJS | 7.8 |
| Build | Angular CLI | 19.2.25 |
| Testes | Jasmine + Karma | — |
| Estado | Angular Signals | — |

## Arquitetura

Arquitetura baseada em DDD (Domain-Driven Design) com separação por camadas, frontend Angular desacoplado e backend REST stateless com JWT.

```text
Frontend Angular (Angular CLI)
      ↓  /api via proxy.conf.json
REST API (Spring Boot)
      ↓
Application Layer (Use Cases)
      ↓
Domain Layer (DDD)
      ↓
Infrastructure Layer (JPA / Security)
      ↓
PostgreSQL (ou H2 em dev)
```

Estrutura do backend (`src/main/java/com/exemplo/usuariosimples/`):

```text
domain/          Entidades, value objects, eventos e interfaces de repositório
application/     Casos de uso e orquestração
infrastructure/  JPA, segurança e seed
interfaces/rest/ Controllers, DTOs e contratos HTTP
```

Estrutura do frontend (`ead-platform/ead-platform/src/app/`):

```text
core/      Services, interceptors, guards, autenticação e layout
features/  auth, dashboard, learning, enrollment, instructor, projects
```

> O package `com.exemplo.usuariosimples` é o nome de pacote legado do esqueleto inicial e foi mantido na implementação.

## Como Executar

### Pré-requisitos

- Java 17+ e Maven 3.9+
- Node.js 18.19+, 20.11+ ou 22+ (Angular 19)
- Docker Desktop (para o modo com PostgreSQL)

### Opção A — Backend com H2 (dev rápido, sem Docker)

```bash
mvn spring-boot:run
```

- API em `http://localhost:8080`
- Swagger em `http://localhost:8080/swagger-ui.html`
- Console H2 em `http://localhost:8080/h2-console` (JDBC URL `jdbc:h2:mem:usuariodb`, usuário `sa`, senha vazia)
- Os dados ficam em memória e são resetados a cada reinício.

### Opção B — Backend com Docker + PostgreSQL

```bash
docker compose up -d --build
```

O compose sobe:

| Serviço | Porta | Credenciais |
|---|---|---|
| PostgreSQL 16 | 5432 | db `usuario_db` / `postgres` / `postgres` |
| pgAdmin | 5050 | `admin@admin.com` / `admin` |
| API Spring Boot | 8080 | profile `postgres` |

Para parar: `docker compose down`.

### Frontend Angular

```bash
cd ead-platform/ead-platform
npm install
npx ng serve
```

Frontend em `http://localhost:4200`. O `proxy.conf.json` encaminha `/api` para o backend em `localhost:8080` (com `pathRewrite` removendo o prefixo `/api`).

### Usuários de teste (seed automático)

| Perfil | Email | Senha |
|---|---|---|
| ALUNO | `aluno@teste.com` | `123456` |
| PROFESSOR | `prof@teste.com` | `123456` |
| ADMINISTRADOR | `admin@teste.com` | `123456` |

Para solução de problemas (porta em uso, driver PostgreSQL, erros do `@angular/build`), veja [`TUTORIAL_SETUP.md`](TUTORIAL_SETUP.md).

## Endpoints da API

| Método | Rota | Descrição |
|---|---|---|
| POST | `/auth/register` | Registrar novo usuário |
| POST | `/auth/login` | Login (retorna JWT) |
| GET | `/alunos/{id}` | Dados do aluno |
| GET | `/alunos/{id}/progresso` | Progresso do aluno |
| GET/POST | `/cursos` | Listar / criar cursos |
| GET/PUT/DELETE | `/cursos/{id}` | Consultar / atualizar / excluir curso |
| GET | `/aulas/{id}` | Consultar aula |
| GET/POST | `/matriculas` | Listar / criar matrículas |
| GET | `/matriculas/{id}` | Consultar matrícula |
| GET | `/matriculas/{id}/progresso` | Progresso da matrícula |
| POST | `/matriculas/{id}/modulos/{ordem}/concluir` | Concluir módulo na ordem |
| GET/POST | `/assinaturas` | Listar / criar assinaturas |
| GET/PUT/DELETE | `/assinaturas/{id}` | Consultar / atualizar / excluir assinatura |
| GET | `/gamificacao/moedas` | Moedas de gamificação |
| GET | `/projetos` | Listar projetos finais |
| GET | `/projetos/{id}` | Consultar projeto |
| PATCH | `/projetos/{id}/avaliar` | Avaliar projeto |

A documentação interativa completa está disponível no Swagger (`/swagger-ui.html`).

## Estrutura do Repositório

```text
projeto-spring-edu-platform/
  src/                              # Backend Spring Boot (Maven)
  ead-platform/
    ead-platform/                    # Frontend Angular
    src/main/resources/static/       # Build de referência do Angular (não versionado)
  docs/                             # Documentação acadêmica, ADR e roteiro
  Dockerfile
  docker-compose.yml
  pom.xml
  AGENTS.md
  TUTORIAL_SETUP.md
  LICENSE
  README.md
```

### Wireframes

As telas de referência (login, cadastro, realizar/enviar atividades e exclusão de curso) estão em `ead-platform/ead-platform/Wireframe - EngSoftware/`.

## Documentação

- [`docs/Documentacao_Final.pdf`](docs/Documentacao_Final.pdf) — documento final da disciplina;
- [`docs/ProjetoEmGrupoPOOESPM_API_GamificationCase_v2.pdf`](docs/ProjetoEmGrupoPOOESPM_API_GamificationCase_v2.pdf) — roteiro do projeto;
- [`docs/adr.md`](docs/adr.md) — registros de decisão de arquitetura;
- `TUTORIAL_SETUP.md` — guia de instalação e resolução de problemas.

## Funcionalidades Pendentes

| Funcionalidade | Status |
|---|---|
| RecompensaService | Pendente |
| TransacaoMoeda | Pendente |
| ProjetoFinal / Avaliação | Pendente |
| Dashboard do aluno | Parcial |

## Licença

Distribuído sob a [MIT License](LICENSE).