# DatabasesDocker

Projeto para subir bancos de dados em **containers Docker** com **persistência de dados**, podendo escolher entre **MariaDB**, **MySQL** ou **PostgreSQL**.  
A configuração usa **Docker Compose com profiles**, o que permite ligar apenas o banco que você precisar no momento.

---

## 📑 Índice
- [Pré-requisitos](#pré-requisitos)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Configuração do ambiente (.env)](#configuração-do-ambiente-env)
- [Como subir um banco](#como-subir-um-banco)
- [Persistência dos dados](#persistência-dos-dados)
- [Comandos úteis](#comandos-úteis)
- [Conexão com DBeaver (ou similar)](#conexão-com-dbeaver-ou-similar)
- [Dicas rápidas](#dicas-rápidas)
- [Próximos passos](#próximos-passos)

---

## ✅ Pré-requisitos
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose v2](https://docs.docker.com/compose/install/)
- Opcional: [DBeaver](./INSTALLDBEAVER.md) (ou outro cliente SQL)

---

## 📂 Estrutura do projeto
``` bash
db-stack/
┣ 📄 compose.yaml
┗ 📄 .env
┗ 📄 .env-example
```


- **compose.yaml** → define os serviços (MariaDB, MySQL, PostgreSQL)  
- **.env** → define as variáveis de ambiente (portas, usuário, senha, etc.)

---

## ⚙️ Configuração do ambiente (.env)

Exemplo de `.env`:

```ini
# Fuso horário
TZ=America/Sao_Paulo

# ----- MariaDB -----
MARIADB_PORT=3308
MARIADB_ROOT_PASSWORD=admin123
MARIADB_DATABASE=appdb
MARIADB_USER=app
MARIADB_PASSWORD=app123

# ----- MySQL -----
MYSQL_PORT=3307
MYSQL_ROOT_PASSWORD=admin123
MYSQL_DATABASE=appdb
MYSQL_USER=app
MYSQL_PASSWORD=app123

# ----- PostgreSQL -----
POSTGRES_PORT=5433
POSTGRES_PASSWORD=admin123
POSTGRES_USER=app
POSTGRES_DB=appdb
```

`⚠️ Importante: altere as senhas antes de usar em produção.`

## 🚀 Como subir um banco

### ▶️ MariaDB
``` bash
docker compose --profile mariadb up -d
```

`Conexão no DBeaver`:
Host: 127.0.0.1 • Port: 3308 • DB: appdb • User: app • Pass: app123

### ▶️ MySQL 8.4
```bash
docker compose --profile mysql up -d
```

`Conexão`:
Host: 127.0.0.1 • Port: 3307 • DB: appdb • User: app • Pass: app123

### ▶️ PostgreSQL 16
```bash
docker compose --profile postgres up -d
```

`Conexão`:
Host: 127.0.0.1 • Port: 5433 • DB: appdb • User: app • Pass: admin123

# 💾 Persistência dos dados

Os dados são mantidos em volumes nomeados:

    - `MariaDB` → mariadb_data
    
    - `MySQL` → mysql_data
    
    - `PostgreSQL` → postgres_data

Eles persistem mesmo após docker compose down.

Serão apagados apenas se rodar:
``` bash
docker compose down -v
# ou
docker volume rm <nome_do_volume>
```

# 🛠️ Comandos úteis

1. Ver containers ativos:
```bash
docker container ls
```

2. Ver todos os containers (inclusive parados):
```bash
docker container ls -a
```

3. Ver logs (útil até passar no healthcheck):
```bash
docker compose logs -f
```

4. Entrar no shell do banco:
```bash
# MariaDB/MySQL
docker exec -it mariadb11 mariadb -u root -p
docker exec -it mysql84 mysql -u root -p

# PostgreSQL
docker exec -it postgres16 psql -U app -d appdb
```

5. Parar o banco atual:
```bash
docker compose --profile mariadb down
# (ou mysql / postgres)
```

# 🖥️ Conexão com DBeaver (ou similar)

- Prefira sempre 127.0.0.1 em vez de localhost.

- Portas e credenciais estão no .env.

- [Guia de instalação do DBeaver](./INSTALLDBEAVER.md)

# Dicas rápidas:
. Se der erro de autenticação no MariaDB/MySQL, crie o usuário dentro do container:
```sql
CREATE USER 'app'@'%' IDENTIFIED BY 'app123';
GRANT ALL PRIVILEGES ON appdb.* TO 'app'@'%';
FLUSH PRIVILEGES;
```

. Para PostgreSQL, crie usuários adicionais com:
```sql
CREATE USER dev WITH PASSWORD 'dev123';
CREATE DATABASE devdb OWNER dev;
```
