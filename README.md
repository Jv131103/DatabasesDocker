# 🐳 DatabasesDocker

Ambiente Docker para criação rápida de bancos de dados relacionais e NoSQL.

O objetivo deste projeto é permitir que qualquer desenvolvedor tenha um ambiente completo de banco de dados em poucos minutos, utilizando apenas Docker Compose e arquivos de configuração.

---

# Recursos

- Docker Compose
- Docker Profiles
- Persistência com Volumes
- Configuração por `.env`
- Bancos independentes
- Fácil integração com DBeaver
- Ambiente totalmente reproduzível

---

# Bancos suportados

## SQL

- MariaDB
- MySQL
- PostgreSQL
- Oracle Database

## NoSQL

- MongoDB
- Redis
- Cassandra

---

# Pré-requisitos

- Docker Desktop ou Docker Engine
- Docker Compose v2

---

# Instalação rápida

Clone o projeto

```bash
git clone <repositorio>
```

Copie o arquivo de ambiente

```bash
cp .env.example .env
```

Suba um banco

```bash
docker compose --profile mysql up -d
```

---

# Documentação

Toda documentação detalhada encontra-se na pasta **docs**.

| Documento | Descrição |
|------------|-----------|
| [`docs/installation.md`](./docs/installation.md) | Instalação completa |
| [`docs/environment.md`](./docs/environment.md) | Configuração do .env |
| [`docs/docker-compose.md`](./docs/docker-compose.md) | Estrutura do Compose |
| [`docs/commands.md`](./docs/commands.md) | Comandos Docker |
| [`docs/dbeaver.md`](./docs/dbeaver.md) | Configuração do DBeaver |
| [`docs/troubleshooting.md`](./docs/troubleshooting.md) | Solução de erros |
| [`docs/faq.md`](./docs/faq.md) | Perguntas frequentes |
| [`docs/sql.md`](./docs/sql.md) | Bancos SQL, conexões, usuários, permissões e exemplos |

---

# Estrutura

```
DatabasesDocker
│
├── compose.yaml
├── compose-nosql.yaml
├── .env.example
└── docs/
```

---

# Objetivos do projeto

- Padronizar ambientes de banco de dados
- Evitar instalação local
- Facilitar testes
- Facilitar aprendizado
- Facilitar integração entre equipes
- Manter persistência dos dados

---

# Próximas evoluções

- pgAdmin
- phpMyAdmin
- Mongo Express
- Redis Insight
- Backup automático
- Restore automático
- Scripts de seed
- CI/CD

---
