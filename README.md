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
- [Importante](#importante)

---

<a id="pré-requisitos"></a>
## ✅ Pré-requisitos
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose v2](https://docs.docker.com/compose/install/)
- Opcional: [DBeaver](./INSTALLDBEAVER.md) (ou outro cliente SQL)

---

<a id="estrutura-do-projeto"></a>
## 📂 Estrutura do projeto
```bash
db-stack/
┣ 📄 compose.yaml
┣ 📄 .env
┗ 📄 .env-example
```

- **compose.yaml** → define os serviços (MariaDB, MySQL, PostgreSQL)  
- **.env** → define variáveis de ambiente (portas, usuário, senha, etc.)

---

<a id="configuração-do-ambiente-env"></a>
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

⚠️ **Importante:** altere as senhas antes de usar em produção.

---

<a id="como-subir-um-banco"></a>
## 🚀 Como subir um banco

### ▶️ MariaDB
```bash
docker compose --profile mariadb up -d
```

Conectar no DBeaver:  
- Host: 127.0.0.1  
- Port: 3308  
- DB: appdb  
- User: app  
- Pass: app123  

### ▶️ MySQL 8.4
```bash
docker compose --profile mysql up -d
```

Conectar no DBeaver:  
- Host: 127.0.0.1  
- Port: 3307  
- DB: appdb  
- User: app  
- Pass: app123  

### ▶️ PostgreSQL 16
```bash
docker compose --profile postgres up -d
```

Conectar no DBeaver:  
- Host: 127.0.0.1  
- Port: 5433  
- DB: appdb  
- User: app  
- Pass: admin123  

---

<a id="persistência-dos-dados"></a>
## 💾 Persistência dos dados

Os dados ficam em volumes nomeados:

- MariaDB → `mariadb_data`  
- MySQL → `mysql_data`  
- PostgreSQL → `postgres_data`  

Eles **persistem mesmo após `docker compose down`**.  
Somente serão apagados se você rodar:
```bash
docker compose down -v
# ou
docker volume rm <nome_do_volume>
```

---

<a id="comandos-úteis"></a>
## 🛠️ Comandos úteis

- Ver status:
```bash
docker container ls
docker container ls -a
```

- Logs (até passar no healthcheck):
```bash
docker compose logs -f
```

- Acessar o shell do banco:
```bash
# MariaDB
docker exec -it mariadb11 mariadb -u root -p

# MySQL
docker exec -it mysql84 mysql -u root -p

# PostgreSQL
docker exec -it postgres16 psql -U app -d appdb
```

- Parar o banco atual:
```bash
docker compose --profile mariadb down
# (ou mysql / postgres)
```

- Parar todos os containers:
```bash
docker stop $(docker ps -aq)
```

- Remover todos os containers:
```bash
docker rm $(docker ps -aq)
```

- Remover volumes (⚠️ apaga bancos):
```bash
docker volume rm $(docker volume ls -q)
```

- (Opcional) Remover imagens também:
```bash
docker rmi $(docker images -q)
```

---

<a id="conexão-com-dbeaver-ou-similar"></a>
## 🖥️ Conexão com DBeaver (ou similar)

- Prefira `127.0.0.1` em vez de `localhost`.  
- Configure a conexão usando as variáveis do `.env`.

**MySQL/MariaDB – Driver properties:**
```txt
allowPublicKeyRetrieval = true
useSSL = false
```

- Guia de instalação do DBeaver: [INSTALLDBEAVER.md](./INSTALLDBEAVER.md)

---

<a id="dicas-rápidas"></a>
## ⚡ Dicas rápidas

- Se der erro de autenticação no MariaDB/MySQL, crie o usuário dentro do container:
```sql
CREATE USER 'app'@'%' IDENTIFIED BY 'app123';
GRANT ALL PRIVILEGES ON appdb.* TO 'app'@'%';
FLUSH PRIVILEGES;
```

- PostgreSQL: criar usuários/DBs adicionais:
```sql
CREATE USER dev WITH PASSWORD 'dev123';
CREATE DATABASE devdb OWNER dev;
```

---

<a id="importante"></a>
## 🚨 Importante

- Sempre use:
```bash
docker stop <container_name>
docker start <container_name>
```

- Se remover o container mas mantiver o volume:
```bash
docker rm <name_container>
```
➡️ Os dados permanecem no volume que gerou.  

- Só perderá tudo se rodar:
```bash
docker compose down -v
```
