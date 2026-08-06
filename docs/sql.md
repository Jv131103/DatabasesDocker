# 🗄️ Bancos de Dados SQL

## Objetivo

Este documento reúne as principais informações sobre os bancos de dados relacionais suportados pelo projeto **DatabasesDocker**.

Atualmente, o projeto contempla:

* MariaDB;
* MySQL;
* PostgreSQL;
* Oracle Database.

Este guia explica:

* as diferenças entre os bancos;
* como iniciar cada serviço;
* como funcionam portas, usuários e bancos;
* como realizar conexões;
* como acessar o terminal de cada banco;
* como criar bancos, usuários e tabelas;
* como trabalhar com persistência;
* como validar o funcionamento;
* cuidados com atualização e exclusão de dados.

---

# Bancos disponíveis

| Banco           | Imagem utilizada | Porta interna | Profile    | Container        |
| --------------- | ---------------- | ------------: | ---------- | ---------------- |
| MariaDB         | `mariadb:11`     |        `3306` | `mariadb`  | `mariadb11`      |
| MySQL           | `mysql:8.4`      |        `3306` | `mysql`    | `mysql84`        |
| PostgreSQL      | `postgres:16`    |        `5432` | `postgres` | `postgres16`     |
| Oracle Database | conforme Compose |        `1521` | `oracle`   | conforme Compose |

> [!NOTE]
> As versões e os nomes reais devem sempre ser confirmados no arquivo `compose.yaml`.

---

# O que é um banco de dados relacional?

Um banco relacional organiza informações em tabelas.

Cada tabela é formada por:

* colunas;
* linhas;
* tipos de dados;
* chaves;
* restrições;
* relacionamentos.

Exemplo:

```text
clientes
├── id
├── nome
├── email
└── criado_em
```

Um registro representa uma linha:

| id | nome   | email                                       |
| -: | ------ | ------------------------------------------- |
|  1 | Renato | [renato@email.com](mailto:renato@email.com) |

Os bancos relacionais utilizam SQL para:

* criar estruturas;
* inserir dados;
* consultar informações;
* atualizar registros;
* remover registros;
* controlar usuários;
* administrar permissões.

---

# O que é SQL?

SQL significa:

```text
Structured Query Language
```

Em português:

```text
Linguagem de Consulta Estruturada
```

Ela é utilizada para interagir com bancos relacionais.

Exemplos:

```sql
SELECT * FROM clientes;
```

```sql
INSERT INTO clientes (nome, email)
VALUES ('Renato', 'renato@email.com');
```

```sql
UPDATE clientes
SET nome = 'Renato Justino'
WHERE id = 1;
```

```sql
DELETE FROM clientes
WHERE id = 1;
```

Embora os bancos utilizem SQL, cada sistema possui particularidades.

---

# Comparação rápida

| Característica         | MariaDB          | MySQL                 | PostgreSQL                 | Oracle                        |
| ---------------------- | ---------------- | --------------------- | -------------------------- | ----------------------------- |
| Categoria              | Relacional       | Relacional            | Objeto-relacional          | Relacional empresarial        |
| Licença principal      | Open source      | Community e comercial | Open source                | Comercial e edições gratuitas |
| Compatibilidade MySQL  | Alta             | Nativa                | Não                        | Não                           |
| Recursos avançados SQL | Bons             | Bons                  | Muito amplos               | Muito amplos                  |
| Uso comum              | Web e aplicações | Web e aplicações      | Sistemas complexos e dados | Sistemas corporativos         |
| Administração local    | Simples          | Simples               | Intermediária              | Mais complexa                 |
| Consumo de recursos    | Moderado         | Moderado              | Moderado                   | Elevado                       |
| Porta interna padrão   | 3306             | 3306                  | 5432                       | 1521                          |

---

# Estrutura das configurações

Cada banco possui:

* imagem Docker;
* profile;
* porta externa;
* porta interna;
* usuário administrativo;
* usuário de aplicação;
* senha;
* banco inicial;
* volume;
* healthcheck.

Exemplo conceitual:

```yaml
services:
  mysql:
    image: mysql:8.4
    profiles:
      - mysql
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "${MYSQL_PORT:-3307}:3306"
    volumes:
      - mysql_data:/var/lib/mysql
```

---

# Configuração pelo `.env`

Exemplo:

```ini
TZ=America/Sao_Paulo

# MariaDB
MARIADB_PORT=3308
MARIADB_ROOT_PASSWORD=admin123
MARIADB_DATABASE=appdb
MARIADB_USER=app
MARIADB_PASSWORD=app123

# MySQL
MYSQL_PORT=3307
MYSQL_ROOT_PASSWORD=admin123
MYSQL_DATABASE=appdb
MYSQL_USER=app
MYSQL_PASSWORD=app123

# PostgreSQL
POSTGRES_PORT=5433
POSTGRES_PASSWORD=admin123
POSTGRES_USER=app
POSTGRES_DB=appdb

# Oracle
ORACLE_PORT=1521
ORACLE_PASSWORD=admin123
ORACLE_APP_USER=app
ORACLE_APP_PASSWORD=app123
```

> [!WARNING]
> Esses valores são apenas exemplos. Não utilize senhas simples em ambientes reais.

---

# Portas externas e internas

Cada banco escuta em uma porta interna dentro do container.

O Docker publica essa porta para a máquina.

Exemplo:

```yaml
ports:
  - "${MYSQL_PORT:-3307}:3306"
```

Com:

```ini
MYSQL_PORT=3307
```

O fluxo será:

```text
Aplicação ou DBeaver
        │
        ▼
127.0.0.1:3307
        │
        ▼
Docker
        │
        ▼
MySQL no container:3306
```

Portanto:

* `3307` é a porta externa;
* `3306` é a porta interna.

Para aplicações executadas na máquina:

```text
Host: 127.0.0.1
Porta: 3307
```

Para aplicações em outro container da mesma rede:

```text
Host: mysql
Porta: 3306
```

---

# MariaDB

## Visão geral

MariaDB é um banco relacional derivado do MySQL.

Ele possui grande compatibilidade com:

* comandos SQL do MySQL;
* drivers MySQL;
* bibliotecas;
* aplicações web;
* ferramentas administrativas.

É uma boa opção para:

* estudos;
* APIs;
* aplicações web;
* sistemas CRUD;
* projetos que já utilizam sintaxe MySQL.

---

## Iniciar MariaDB

```bash
docker compose --profile mariadb up -d
```

Verificar:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f mariadb
```

---

## Conexão

Considerando o `.env` de exemplo:

```text
Host: 127.0.0.1
Porta: 3308
Banco: appdb
Usuário: app
Senha: app123
```

---

## Acesso pelo terminal

Como root:

```bash
docker exec -it mariadb11 \
  mariadb -u root -p
```

Como usuário da aplicação:

```bash
docker exec -it mariadb11 \
  mariadb -u app -p appdb
```

---

## Comandos básicos

Listar bancos:

```sql
SHOW DATABASES;
```

Selecionar banco:

```sql
USE appdb;
```

Listar tabelas:

```sql
SHOW TABLES;
```

Ver usuário atual:

```sql
SELECT CURRENT_USER();
```

Ver versão:

```sql
SELECT VERSION();
```

---

# MySQL

## Visão geral

MySQL é um dos bancos relacionais mais utilizados em aplicações web.

É comum em projetos com:

* PHP;
* Node.js;
* Python;
* Java;
* WordPress;
* APIs;
* sistemas administrativos.

---

## Iniciar MySQL

```bash
docker compose --profile mysql up -d
```

Verificar:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f mysql
```

---

## Conexão

Considerando o `.env` de exemplo:

```text
Host: 127.0.0.1
Porta: 3307
Banco: appdb
Usuário: app
Senha: app123
```

---

## Acesso pelo terminal

Como root:

```bash
docker exec -it mysql84 \
  mysql -u root -p
```

Como usuário da aplicação:

```bash
docker exec -it mysql84 \
  mysql -u app -p appdb
```

---

## Propriedades para DBeaver

Em desenvolvimento local, caso necessário:

```text
allowPublicKeyRetrieval = true
useSSL = false
```

Essas propriedades são explicadas detalhadamente em:

```text
docs/dbeaver.md
```

---

## Comandos básicos

```sql
SHOW DATABASES;
```

```sql
USE appdb;
```

```sql
SHOW TABLES;
```

```sql
SELECT DATABASE();
```

```sql
SELECT CURRENT_USER();
```

```sql
SELECT VERSION();
```

---

# PostgreSQL

## Visão geral

PostgreSQL é um banco relacional avançado, conhecido por:

* conformidade com SQL;
* integridade dos dados;
* transações robustas;
* tipos avançados;
* JSON e JSONB;
* extensões;
* consultas complexas;
* funções;
* views;
* procedures;
* índices avançados.

É uma excelente opção para:

* APIs;
* sistemas financeiros;
* análise de dados;
* aplicações empresariais;
* geoprocessamento;
* projetos com regras complexas.

---

## Iniciar PostgreSQL

```bash
docker compose --profile postgres up -d
```

Verificar:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f postgres
```

---

## Conexão

Considerando o `.env` de exemplo:

```text
Host: 127.0.0.1
Porta: 5433
Banco: appdb
Usuário: app
Senha: admin123
```

---

## Acesso pelo terminal

```bash
docker exec -it postgres16 \
  psql -U app -d appdb
```

---

## Comandos do `psql`

Listar bancos:

```sql
\l
```

Conectar em outro banco:

```sql
\c appdb
```

Listar schemas:

```sql
\dn
```

Listar tabelas:

```sql
\dt
```

Descrever uma tabela:

```sql
\d nome_da_tabela
```

Listar usuários:

```sql
\du
```

Sair:

```sql
\q
```

---

## Consultas básicas

```sql
SELECT current_database();
```

```sql
SELECT current_user;
```

```sql
SELECT version();
```

---

# Oracle Database

## Visão geral

Oracle Database é amplamente utilizado em ambientes corporativos.

Ele possui:

* SQL;
* PL/SQL;
* schemas;
* procedures;
* packages;
* sequences;
* triggers;
* recursos avançados de segurança;
* administração empresarial.

Sua configuração é diferente dos demais bancos.

---

## Diferença principal

MySQL e PostgreSQL normalmente utilizam:

```text
Database
```

Oracle utiliza principalmente:

```text
Service Name
+
Schema
```

No Oracle, normalmente o usuário também representa seu schema.

---

## Iniciar Oracle

```bash
docker compose --profile oracle up -d
```

Acompanhar os logs:

```bash
docker compose logs -f oracle
```

O Oracle pode demorar mais do que os outros bancos para ficar disponível.

---

## Conexão

Configuração prevista:

```text
Host: 127.0.0.1
Porta: 1521
Usuário: app
Senha: app123
Service Name: FREEPDB1
```

---

## Acesso pelo terminal

O comando exato depende do nome do container e das ferramentas disponíveis na imagem.

Exemplo:

```bash
docker exec -it oracle-free \
  sqlplus app/app123@//localhost:1521/FREEPDB1
```

---

## Consultas básicas

Usuário atual:

```sql
SELECT USER
FROM dual;
```

Service Name:

```sql
SELECT SYS_CONTEXT(
    'USERENV',
    'SERVICE_NAME'
)
FROM dual;
```

Data atual:

```sql
SELECT SYSDATE
FROM dual;
```

Versão:

```sql
SELECT *
FROM v$version;
```

---

# Criando uma tabela de exemplo

A estrutura abaixo funciona com pequenas variações nos principais bancos.

```sql
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Inserir:

```sql
INSERT INTO clientes (
    id,
    nome,
    email
)
VALUES (
    1,
    'Renato',
    'renato@email.com'
);
```

Consultar:

```sql
SELECT *
FROM clientes;
```

Atualizar:

```sql
UPDATE clientes
SET nome = 'Renato Justino'
WHERE id = 1;
```

Excluir:

```sql
DELETE FROM clientes
WHERE id = 1;
```

---

# Chaves primárias

Uma chave primária identifica um registro de maneira única.

Exemplo:

```sql
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nome VARCHAR(100)
);
```

Características:

* não pode se repetir;
* normalmente não aceita `NULL`;
* identifica cada registro;
* pode ser utilizada em relacionamentos.

---

# Auto incremento

A sintaxe muda conforme o banco.

## MySQL e MariaDB

```sql
CREATE TABLE clientes (
    id INTEGER AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);
```

## PostgreSQL

Opção tradicional:

```sql
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);
```

Opção moderna:

```sql
CREATE TABLE clientes (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);
```

## Oracle

```sql
CREATE TABLE clientes (
    id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome VARCHAR2(100) NOT NULL
);
```

---

# Chaves estrangeiras

Uma chave estrangeira cria um relacionamento entre tabelas.

```sql
CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER NOT NULL,
    valor DECIMAL(10, 2),
    CONSTRAINT fk_pedido_cliente
        FOREIGN KEY (cliente_id)
        REFERENCES clientes(id)
);
```

Esse relacionamento significa:

```text
clientes
    │
    └── pedidos
```

Um cliente pode possuir vários pedidos.

---

# Transações

Transações agrupam operações.

```sql
BEGIN;
```

```sql
UPDATE contas
SET saldo = saldo - 100
WHERE id = 1;
```

```sql
UPDATE contas
SET saldo = saldo + 100
WHERE id = 2;
```

Confirmar:

```sql
COMMIT;
```

Cancelar antes da confirmação:

```sql
ROLLBACK;
```

> [!IMPORTANT]
> O comportamento e a sintaxe podem variar conforme o banco, driver e modo de auto-commit.

---

# Criação de bancos

## MySQL e MariaDB

```sql
CREATE DATABASE appdb;
```

Selecionar:

```sql
USE appdb;
```

---

## PostgreSQL

```sql
CREATE DATABASE appdb;
```

O comando precisa ser executado conectado a outro banco, como `postgres`.

---

## Oracle

No Oracle, normalmente trabalha-se com:

* pluggable database;
* service;
* schema;
* usuário.

Criar um usuário/schema:

```sql
CREATE USER dev
IDENTIFIED BY dev123;
```

Conceder permissões básicas:

```sql
GRANT CREATE SESSION TO dev;
GRANT CREATE TABLE TO dev;
GRANT CREATE SEQUENCE TO dev;
```

---

# Criação de usuários

## MySQL e MariaDB

```sql
CREATE USER
'app'@'%'
IDENTIFIED BY 'app123';
```

Conceder acesso:

```sql
GRANT ALL PRIVILEGES
ON appdb.*
TO 'app'@'%';
```

Atualizar permissões:

```sql
FLUSH PRIVILEGES;
```

Ver permissões:

```sql
SHOW GRANTS FOR 'app'@'%';
```

---

## PostgreSQL

```sql
CREATE USER dev
WITH PASSWORD 'dev123';
```

Criar banco para o usuário:

```sql
CREATE DATABASE devdb
OWNER dev;
```

Conceder conexão:

```sql
GRANT CONNECT
ON DATABASE appdb
TO dev;
```

---

## Oracle

```sql
CREATE USER dev
IDENTIFIED BY dev123;
```

Permitir conexão:

```sql
GRANT CREATE SESSION TO dev;
```

Permitir criação de tabelas:

```sql
GRANT CREATE TABLE TO dev;
```

---

# Princípio do menor privilégio

Uma aplicação não deve se conectar como root ou administrador.

O recomendado é:

```text
Administrador
    │
    ├── manutenção
    ├── criação de usuários
    ├── backup
    └── permissões

Usuário da aplicação
    │
    ├── SELECT
    ├── INSERT
    ├── UPDATE
    └── DELETE
```

Evite conceder permissões que a aplicação não utiliza.

---

# Persistência

Os dados ficam em volumes nomeados.

| Banco      | Volume          |
| ---------- | --------------- |
| MariaDB    | `mariadb_data`  |
| MySQL      | `mysql_data`    |
| PostgreSQL | `postgres_data` |
| Oracle     | `oracle_data`   |

Os dados permanecem após:

```bash
docker stop <container>
```

```bash
docker start <container>
```

```bash
docker compose down
```

```bash
docker rm <container>
```

Desde que o volume não seja removido.

---

# Quando os dados são apagados?

Os dados podem ser apagados por:

```bash
docker compose down -v
```

Ou:

```bash
docker volume rm <nome_do_volume>
```

Ou:

```bash
docker volume prune
```

> [!CAUTION]
> Nunca execute esses comandos sem confirmar se os bancos podem ser perdidos.

---

# Inicialização e volumes existentes

As imagens dos bancos normalmente utilizam as variáveis de ambiente apenas na primeira inicialização do volume.

Exemplo:

```ini
MYSQL_DATABASE=appdb
MYSQL_USER=app
MYSQL_PASSWORD=app123
```

Esses valores podem criar o banco e o usuário quando o volume está vazio.

Depois que o volume já foi inicializado, alterar essas variáveis pode não modificar:

* bancos existentes;
* usuários existentes;
* senhas existentes;
* permissões existentes.

Nesses casos, altere diretamente no banco ou recrie o volume somente quando os dados puderem ser descartados.

---

# Healthcheck

O healthcheck verifica se o banco está realmente respondendo.

Estados possíveis:

```text
starting
healthy
unhealthy
```

Verificar:

```bash
docker compose ps
```

Detalhes:

```bash
docker inspect <container>
```

Logs:

```bash
docker compose logs -f <serviço>
```

Não tente conectar antes de o banco concluir a inicialização.

---

# Iniciando mais de um banco

É possível ativar mais de um profile.

Exemplo:

```bash
docker compose \
  --profile mysql \
  --profile postgres \
  up -d
```

Confirme que as portas externas são diferentes:

```ini
MYSQL_PORT=3307
POSTGRES_PORT=5433
```

---

# Parando um banco

Parar apenas um container:

```bash
docker stop mysql84
```

Iniciar novamente:

```bash
docker start mysql84
```

Reiniciar:

```bash
docker restart mysql84
```

---

# Parando o ambiente Compose

```bash
docker compose down
```

Esse comando remove os containers e a rede do projeto, mas normalmente preserva os volumes nomeados.

---

# Logs por banco

## MariaDB

```bash
docker compose logs -f mariadb
```

## MySQL

```bash
docker compose logs -f mysql
```

## PostgreSQL

```bash
docker compose logs -f postgres
```

## Oracle

```bash
docker compose logs -f oracle
```

---

# Validação rápida

## MySQL e MariaDB

```sql
SELECT
    DATABASE() AS banco,
    CURRENT_USER() AS usuario,
    VERSION() AS versao;
```

## PostgreSQL

```sql
SELECT
    current_database() AS banco,
    current_user AS usuario,
    version() AS versao;
```

## Oracle

```sql
SELECT
    USER AS usuario,
    SYS_CONTEXT(
        'USERENV',
        'SERVICE_NAME'
    ) AS servico
FROM dual;
```

---

# Atualização de versões

Alterar:

```yaml
image: postgres:16
```

para outra versão principal pode causar incompatibilidade com o volume.

Exemplo de alteração importante:

```text
PostgreSQL 15 → PostgreSQL 16
```

Antes de atualizar:

1. identifique a versão atual;
2. faça backup;
3. teste a restauração;
4. leia as notas de atualização;
5. crie um ambiente separado;
6. valide os dados;
7. atualize a documentação.

Não utilize automaticamente:

```yaml
image: postgres:latest
```

Prefira versões explícitas.

---

# Backup

Cada banco possui uma ferramenta própria.

## MariaDB

```bash
mariadb-dump
```

## MySQL

```bash
mysqldump
```

## PostgreSQL

```bash
pg_dump
```

## Oracle

Ferramentas como:

```text
expdp
impdp
```

ou mecanismos compatíveis com a imagem utilizada.

> [!NOTE]
> Exportar CSV pelo DBeaver não substitui necessariamente um backup completo e consistente.

---

# Erros comuns

## Banco não conecta

Verifique:

```bash
docker compose ps
docker compose logs --tail 200
```

Confirme:

* container em execução;
* healthcheck;
* porta externa;
* host;
* usuário;
* senha;
* banco.

---

## `Access denied`

Normalmente relacionado a:

* usuário incorreto;
* senha incorreta;
* usuário sem permissão;
* volume inicializado com senha antiga;
* origem do usuário limitada.

Consulte:

```text
docs/troubleshooting.md
```

---

## Banco não existe

MySQL ou MariaDB:

```sql
SHOW DATABASES;
```

PostgreSQL:

```sql
\l
```

Oracle:

Confirme o Service Name e o schema.

---

## Porta ocupada

Altere no `.env`:

```ini
MYSQL_PORT=3310
```

Depois recrie o container:

```bash
docker compose \
  --profile mysql \
  up -d \
  --force-recreate
```

---

## Senha alterada no `.env`, mas não funciona

A senha antiga pode estar armazenada no banco persistido.

Altere diretamente no banco.

Não remova o volume antes de confirmar se os dados possuem backup.

---

# Qual banco escolher?

## Escolha MariaDB quando:

* deseja compatibilidade com MySQL;
* procura uma opção open source;
* possui um projeto web simples;
* deseja baixo esforço de configuração.

## Escolha MySQL quando:

* a aplicação exige MySQL;
* deseja ampla compatibilidade com hospedagens;
* utiliza ferramentas específicas do ecossistema MySQL;
* trabalha com projetos web tradicionais.

## Escolha PostgreSQL quando:

* precisa de SQL avançado;
* deseja forte integridade;
* possui consultas complexas;
* trabalha com JSONB;
* precisa de extensões;
* possui regras de negócio mais avançadas.

## Escolha Oracle quando:

* deseja estudar ambiente corporativo;
* trabalha com PL/SQL;
* a organização utiliza Oracle;
* precisa reproduzir um sistema baseado em Oracle;
* deseja aprender Service Name, schemas e ferramentas empresariais.

---

# Boas práticas

* Não utilize root na aplicação.
* Use usuários específicos por sistema.
* Mantenha portas diferentes.
* Não publique o `.env`.
* Utilize senhas fortes.
* Faça backup antes de atualizar.
* Fixe versões das imagens.
* Aguarde o healthcheck.
* Consulte os logs antes de remover containers.
* Não remova volumes para resolver erros simples.
* Documente novos bancos.
* Use transações em operações importantes.
* Utilize `WHERE` em alterações e exclusões.
* Valide a conexão antes de executar scripts.
* Separe ambientes de desenvolvimento e produção.

---

# Checklist

Antes de utilizar um banco:

* [ ] Docker está ativo.
* [ ] `.env` foi configurado.
* [ ] O profile está correto.
* [ ] A porta está disponível.
* [ ] O container foi iniciado.
* [ ] O banco está `healthy`.
* [ ] O host está correto.
* [ ] A porta externa está correta.
* [ ] O banco existe.
* [ ] O usuário existe.
* [ ] A senha está correta.
* [ ] O usuário possui permissão.
* [ ] O volume está montado.
* [ ] Há espaço em disco.
* [ ] Existe backup para dados importantes.

---

# Documentos relacionados

* [`installation.md`](./docs/installation.md) — instalação;
* [`environment.md`](./docs/environment.md) — variáveis de ambiente;
* [`docker-compose.md`](./docs/docker-compose.md) — configuração do Compose;
* [`commands.md`](./docs/commands.md) — comandos;
* [`dbeaver.md`](./docs/dbeaver.md) — conexões gráficas;
* [`troubleshooting.md`](./docs/troubleshooting.md) — solução de erros;
* [`faq.md`](./docs/faq.md) — perguntas frequentes;

---

# Resumo

Os bancos SQL deste projeto são executados de forma independente por meio de Docker Profiles.

O funcionamento geral é:

```text
.env
  │
  ▼
Docker Compose
  │
  ▼
Profile escolhido
  │
  ▼
Container do banco
  │
  ├── porta
  ├── usuário
  ├── senha
  ├── banco
  ├── healthcheck
  └── volume persistente
```

Para a máquina local, utilize:

```text
Host: 127.0.0.1
Porta: porta externa do .env
```

Para comunicação entre containers:

```text
Host: nome do serviço
Porta: porta interna do banco
```

Os volumes devem ser preservados, os logs devem ser consultados antes de qualquer remoção e alterações de versão devem ser precedidas por backup e testes.
