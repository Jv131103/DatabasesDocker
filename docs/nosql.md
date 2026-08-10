# 🍃 Bancos de Dados NoSQL

## Objetivo

Este documento reúne todas as informações necessárias para utilizar os bancos de dados **NoSQL** disponibilizados pelo projeto **DatabasesDocker**.

Atualmente o ambiente fornece suporte para:

- MongoDB;
- Redis;
- Cassandra.

Ao final desta documentação você será capaz de:

- entender o que é NoSQL;
- conhecer as diferenças entre MongoDB, Redis e Cassandra;
- saber quando utilizar cada banco;
- iniciar apenas o banco desejado utilizando Docker Compose Profiles;
- conectar aplicações aos bancos;
- utilizar o terminal de cada banco;
- compreender como funciona a persistência de dados;
- validar a configuração do ambiente;
- diagnosticar problemas comuns.

---

# Bancos disponíveis

| Banco | Categoria | Porta Interna | Docker Compose Profile | Container |
|--------|-----------|--------------:|------------------------|-----------|
| MongoDB | Banco documental | `27017` | `mongo` | `mongo_db` |
| Redis | Chave-valor em memória | `6379` | `redis` | `redis_db` |
| Cassandra | Banco distribuído por colunas | `9042` | `cassandra` | `cassandra_db` |

---

# Estrutura do projeto

Todos os bancos do projeto são definidos em um único arquivo:

```text
docker-compose.yaml
```

Cada banco possui seu próprio **Docker Compose Profile**, permitindo iniciar apenas os serviços necessários.

Profiles disponíveis:

```text
mariadb
mysql
postgres
oracle
mongo
redis
cassandra
```

Isso significa que você pode iniciar somente o banco que deseja utilizar, economizando memória e processamento.

Exemplos:

```bash
docker compose --profile mongo up -d
```

```bash
docker compose --profile redis up -d
```

```bash
docker compose --profile cassandra up -d
```

Também é possível iniciar vários bancos simultaneamente:

```bash
docker compose \
  --profile mongo \
  --profile redis \
  up -d
```

---

# O que é NoSQL?

NoSQL significa:

```text
Not Only SQL
```

Em português:

```text
Não Apenas SQL
```

O termo representa bancos de dados que não seguem obrigatoriamente o modelo relacional tradicional baseado em tabelas.

Ao contrário dos bancos SQL, onde praticamente tudo é organizado em tabelas relacionadas entre si, bancos NoSQL podem armazenar informações em diversos formatos.

Os principais modelos existentes são:

- documentos;
- chave e valor;
- famílias de colunas;
- grafos;
- séries temporais;
- estruturas em memória.

Cada modelo foi criado para resolver problemas específicos.

Por esse motivo, NoSQL não substitui SQL.

Na prática, ambos costumam trabalhar juntos dentro da mesma aplicação.

---

# Quando utilizar NoSQL?

Cada banco foi criado para resolver um tipo específico de problema.

Não existe um banco "melhor".

Existe o banco mais adequado para determinada necessidade.

## MongoDB

Ideal para:

- APIs REST;
- aplicações Web;
- microsserviços;
- documentos JSON;
- catálogos;
- CMS;
- perfis de usuários;
- aplicações com schema flexível.

---

## Redis

Ideal para:

- cache;
- sessões;
- filas;
- autenticação;
- rate limiting;
- rankings;
- armazenamento temporário;
- Pub/Sub.

---

## Cassandra

Ideal para:

- Big Data;
- IoT;
- Telemetria;
- Eventos;
- Logs;
- Sistemas distribuídos;
- Alta disponibilidade;
- Grande volume de escrita.

---

# Comparação Geral

| Característica | MongoDB | Redis | Cassandra |
|----------------|----------|--------|-----------|
| Modelo | Documentos | Chave-Valor | Colunas |
| Persistência | Sim | Configurável | Sim |
| Velocidade | Alta | Muito Alta | Alta |
| Escalabilidade | Horizontal | Horizontal | Horizontal |
| Estrutura | Flexível | Estruturas em memória | Orientado à consulta |
| Melhor uso | Documentos | Cache | Grandes volumes |

---

# Docker Compose

Todo o ambiente utiliza um único arquivo:

```text
docker-compose.yaml
```

Antes de iniciar qualquer banco, recomenda-se validar o Compose.

```bash
docker compose config
```

Esse comando verifica:

- sintaxe YAML;
- variáveis do `.env`;
- profiles;
- portas;
- redes;
- volumes;
- containers.

Caso exista algum erro de configuração, ele será apresentado antes da criação dos containers.

---

# Docker Compose Profiles

Todos os bancos deste projeto utilizam Docker Compose Profiles.

Os Profiles disponíveis são:

| Banco | Profile |
|--------|---------|
| MariaDB | mariadb |
| MySQL | mysql |
| PostgreSQL | postgres |
| Oracle | oracle |
| MongoDB | mongo |
| Redis | redis |
| Cassandra | cassandra |

Exemplo iniciando somente MongoDB:

```bash
docker compose --profile mongo up -d
```

Somente Redis:

```bash
docker compose --profile redis up -d
```

Somente Cassandra:

```bash
docker compose --profile cassandra up -d
```

MongoDB e Redis:

```bash
docker compose \
  --profile mongo \
  --profile redis \
  up -d
```

Todos os bancos:

```bash
docker compose --profile "*" up -d
```

---

# Configuração pelo .env

As portas, usuários e senhas são configuradas através do arquivo `.env`.

Exemplo:

```ini
TZ=America/Sao_Paulo

# MongoDB
MONGO_ROOT_USER=admin
MONGO_ROOT_PASSWORD=admin123
MONGO_DB=appdb
MONGO_PORT=27017

# Redis
REDIS_PORT=6379

# Cassandra
CASSANDRA_CLUSTER_NAME=DatabasesDocker
CASSANDRA_PORT=9042
```

As variáveis somente terão efeito quando utilizadas pelo `docker-compose.yaml`.

---

# Portas

Exemplo do MongoDB:

```yaml
ports:
  - "${MONGO_PORT:-27017}:27017"
```

A porta da esquerda representa a porta da máquina.

A porta da direita representa a porta utilizada dentro do container.

Fluxo:

```text
Aplicação

      │

      ▼

127.0.0.1:27017

      │

      ▼

Docker

      │

      ▼

MongoDB:27017
```

Quando outra aplicação estiver rodando dentro do Docker, utilize:

```text
mongo:27017
```

Nunca:

```text
127.0.0.1
```

pois dentro de um container `localhost` representa o próprio container.

---

# MongoDB

## O que é?

MongoDB é um banco de dados orientado a documentos.

Ele armazena informações utilizando documentos BSON (Binary JSON).

Na prática, trabalha quase sempre com estruturas muito parecidas com JSON.

Exemplo:

```javascript
{
    nome: "Joao",
    idade: 22,
    ativo: true,

    endereco: {
        cidade: "São Paulo",
        estado: "SP"
    },

    cursos: [
        "Python",
        "Docker",
        "SQL"
    ]
}
```

Ao contrário dos bancos relacionais, documentos diferentes podem possuir estruturas diferentes.

Isso torna o MongoDB extremamente flexível.

---

## Principais conceitos

| MongoDB | Equivalente aproximado no SQL |
|----------|------------------------------|
| Database | Database |
| Collection | Tabela |
| Document | Linha |
| Field | Coluna |
| _id | Primary Key |
| Embedded Document | Objeto relacionado |
| Reference | Referência externa |

> **Importante:** a comparação acima é apenas conceitual.

MongoDB não funciona como um banco relacional.

# Iniciando o MongoDB

Para iniciar somente o MongoDB:

```bash
docker compose --profile mongo up -d
```

Verifique se o container foi iniciado:

```bash
docker compose ps
```

Saída esperada:

```text
NAME         STATUS
mongo_db     Up (healthy)
```

Também é possível verificar utilizando o Docker:

```bash
docker ps
```

---

# Logs do MongoDB

Para acompanhar os logs em tempo real:

```bash
docker compose logs -f mongo
```

Ou:

```bash
docker logs -f mongo_db
```

As últimas linhas do log:

```bash
docker logs --tail 100 mongo_db
```

---

# Acessando o terminal

Entre no Mongo Shell:

```bash
docker exec -it mongo_db mongosh
```

Caso exista autenticação:

```bash
docker exec -it mongo_db \
    mongosh \
    --username admin \
    --password admin123 \
    --authenticationDatabase admin
```

> Utilize sempre as credenciais configuradas no arquivo `.env`.

---

# String de conexão

Aplicações executadas na máquina:

```text
mongodb://admin:admin123@127.0.0.1:27017/?authSource=admin
```

Especificando um banco:

```text
mongodb://admin:admin123@127.0.0.1:27017/appdb?authSource=admin
```

Aplicações executadas em outro container:

```text
mongodb://admin:admin123@mongo:27017/appdb?authSource=admin
```

---

# Estrutura da URI

```text
mongodb://
```

Protocolo.

```text
admin:senha
```

Usuário e senha.

```text
mongo
```

Nome do serviço Docker.

```text
27017
```

Porta interna do container.

```text
appdb
```

Banco padrão.

```text
authSource=admin
```

Banco responsável pela autenticação.

---

# Databases

Listar bancos:

```javascript
show dbs
```

Selecionar um banco:

```javascript
use appdb
```

O MongoDB cria o banco automaticamente quando o primeiro documento é salvo.

---

# Collections

Criar:

```javascript
db.createCollection("clientes")
```

Listar:

```javascript
show collections
```

Remover:

```javascript
db.clientes.drop()
```

---

# Inserindo documentos

```javascript
db.clientes.insertOne({

    nome: "Joao",

    email: "joao@email.com",

    idade: 22,

    ativo: true

})
```

Inserindo vários:

```javascript
db.clientes.insertMany([

    {

        nome: "Ana",

        idade: 20

    },

    {

        nome: "Carlos",

        idade: 35

    }

])
```

---

# Consultas

Todos os documentos:

```javascript
db.clientes.find()
```

Formatado:

```javascript
db.clientes.find().pretty()
```

Somente um:

```javascript
db.clientes.findOne()
```

Filtrar:

```javascript
db.clientes.find({

    ativo: true

})
```

Filtrar por idade:

```javascript
db.clientes.find({

    idade: {

        $gte: 18

    }

})
```

---

# Ordenação

Ascendente:

```javascript
db.clientes.find().sort({

    nome: 1

})
```

Descendente:

```javascript
db.clientes.find().sort({

    idade: -1

})
```

---

# Limite

Primeiros cinco:

```javascript
db.clientes.find().limit(5)
```

---

# Atualização

```javascript
db.clientes.updateOne(

    {

        email: "joao@email.com"

    },

    {

        $set: {

            nome: "Joao Vitor Justino"

        }

    }

)
```

Atualizar vários:

```javascript
db.clientes.updateMany(

    {},

    {

        $set: {

            ativo: true

        }

    }

)
```

---

# Removendo documentos

Um documento:

```javascript
db.clientes.deleteOne({

    email: "joao@email.com"

})
```

Vários:

```javascript
db.clientes.deleteMany({

    ativo: false

})
```

Todos:

```javascript
db.clientes.deleteMany({})
```

---

# Índices

Criar índice:

```javascript
db.clientes.createIndex({

    email: 1

})
```

Índice único:

```javascript
db.clientes.createIndex(

    {

        email: 1

    },

    {

        unique: true

    }

)
```

Consultar:

```javascript
db.clientes.getIndexes()
```

Remover:

```javascript
db.clientes.dropIndex("email_1")
```

---

# Operadores comuns

Maior que:

```javascript
$gt
```

Maior ou igual:

```javascript
$gte
```

Menor:

```javascript
$lt
```

Menor ou igual:

```javascript
$lte
```

Diferente:

```javascript
$ne
```

Dentro de uma lista:

```javascript
$in
```

AND:

```javascript
$and
```

OR:

```javascript
$or
```

---

# Exemplo completo

```javascript
db.clientes.find({

    idade: {

        $gte: 18

    },

    ativo: true

}).sort({

    nome: 1

})
```

---

# Persistência

Os dados ficam armazenados em:

```text
/data/db
```

No Compose:

```yaml
volumes:

    - mongo_data:/data/db
```

O volume nomeado garante que os dados permaneçam após reiniciar o container.

---

# Testando a persistência

Criar um documento:

```javascript
db.teste.insertOne({

    mensagem: "Persistência funcionando"

})
```

Pare o container:

```bash
docker compose stop mongo
```

Inicie novamente:

```bash
docker compose --profile mongo up -d
```

Verifique:

```javascript
db.teste.find()
```

O documento deverá continuar existindo.

---

# Verificando autenticação

Consultar usuário autenticado:

```javascript
db.runCommand({

    connectionStatus: 1

})
```

---

# Teste rápido

```javascript
use appdb

db.teste.insertOne({

    nome: "MongoDB",

    criadoEm: new Date()

})

db.teste.find()
```

---

# Problemas comuns

## Authentication Failed

Verifique:

- usuário;
- senha;
- authSource;
- porta;
- banco utilizado;
- credenciais existentes no volume.

---

## Banco não aparece

Um banco vazio normalmente não aparece em:

```javascript
show dbs
```

Insira um documento primeiro.

---

## Senha alterada no .env

As variáveis

```ini
MONGO_ROOT_USER
```

e

```ini
MONGO_ROOT_PASSWORD
```

são utilizadas apenas durante a primeira inicialização do volume.

Alterar o `.env` posteriormente não modifica usuários já existentes.

---

## Porta ocupada

Sintoma:

```text
Bind for 0.0.0.0:27017 failed
```

Altere:

```ini
MONGO_PORT=27018
```

Depois recrie o container:

```bash
docker compose \
    --profile mongo \
    up -d \
    --force-recreate
```

---

## Container reiniciando

Verifique:

```bash
docker compose logs mongo
```

e

```bash
docker inspect mongo_db
```

---

## Healthcheck

Consultar:

```bash
docker compose ps
```

ou

```bash
docker inspect \
    --format='{{json .State.Health}}' \
    mongo_db
```

Status esperado:

```text
healthy
```

---

# Boas práticas

- Utilize índices para consultas frequentes.
- Evite documentos extremamente grandes.
- Utilize Collections bem definidas.
- Faça backup regularmente.
- Não exponha o MongoDB diretamente na Internet.
- Utilize autenticação.
- Utilize volumes nomeados.
- Evite utilizar a conta administrativa na aplicação.
- Mantenha versões fixas da imagem Docker.
- Monitore o crescimento das Collections.
- Utilize o MongoDB Compass para inspeção visual dos dados.

---

# Redis

## O que é?

Redis é um banco de dados **NoSQL** baseado em **chave e valor (Key-Value)**.

Diferente da maioria dos bancos tradicionais, ele mantém os dados principalmente **em memória (RAM)**, tornando as operações extremamente rápidas.

Em vez de tabelas ou documentos, tudo é armazenado utilizando uma chave que aponta para um valor.

Exemplo:

```text
nome -> Joao

idade -> 22

cidade -> São Paulo
```

O Redis também suporta estruturas mais avançadas como:

- Strings;
- Hashes;
- Lists;
- Sets;
- Sorted Sets;
- Streams;
- Bitmaps;
- HyperLogLog;
- Dados Geoespaciais.

Por isso ele é muito utilizado como cache e armazenamento temporário de dados.

---

# Quando utilizar Redis?

Redis é indicado quando é necessário:

- Cache de consultas;
- Sessões de usuários;
- Tokens JWT;
- Filas simples;
- Pub/Sub;
- Rate Limiting;
- Contadores;
- Rankings;
- Dados temporários;
- Armazenamento extremamente rápido.

---

# Iniciando o Redis

Subir somente Redis:

```bash
docker compose --profile redis up -d
```

Verificar:

```bash
docker compose ps
```

Saída esperada:

```text
NAME

redis_db

STATUS

Up (healthy)
```

---

# Logs

Acompanhar os logs:

```bash
docker compose logs -f redis
```

Ou:

```bash
docker logs -f redis_db
```

Últimas linhas:

```bash
docker logs --tail 100 redis_db
```

---

# Acessando o Redis CLI

Sem senha:

```bash
docker exec -it redis_db redis-cli
```

Caso exista senha:

```bash
docker exec -it redis_db \
    redis-cli \
    -a "redis123"
```

---

# Testando a conexão

```text
PING
```

Resposta esperada:

```text
PONG
```

---

# Strings

Criar:

```text
SET nome Joao
```

Consultar:

```text
GET nome
```

Alterar:

```text
SET nome "Joao Vitor Justino"
```

Apagar:

```text
DEL nome
```

Verificar existência:

```text
EXISTS nome
```

---

# Expiração

Criar chave válida por 60 segundos:

```text
SET token abc123 EX 60
```

Consultar tempo restante:

```text
TTL token
```

Remover expiração:

```text
PERSIST token
```

---

# Contadores

Criar:

```text
SET acessos 0
```

Incrementar:

```text
INCR acessos
```

Incrementar por valor:

```text
INCRBY acessos 10
```

Decrementar:

```text
DECR acessos
```

---

# Hashes

Criar:

```text
HSET usuario:1 \
    nome Joao \
    email joao@email.com \
    idade 22
```

Consultar campo:

```text
HGET usuario:1 nome
```

Consultar todos:

```text
HGETALL usuario:1
```

Listar somente as chaves:

```text
HKEYS usuario:1
```

---

# Lists

Adicionar no início:

```text
LPUSH fila tarefa1
```

Adicionar no final:

```text
RPUSH fila tarefa2
```

Consultar:

```text
LRANGE fila 0 -1
```

Remover do início:

```text
LPOP fila
```

Remover do final:

```text
RPOP fila
```

---

# Sets

Adicionar:

```text
SADD linguagens Python Docker Redis
```

Consultar:

```text
SMEMBERS linguagens
```

Verificar existência:

```text
SISMEMBER linguagens Python
```

Remover:

```text
SREM linguagens Docker
```

---

# Sorted Sets

Criar ranking:

```text
ZADD ranking 100 Joao
```

Adicionar outro:

```text
ZADD ranking 80 Ana
```

Consultar crescente:

```text
ZRANGE ranking 0 -1 WITHSCORES
```

Consultar decrescente:

```text
ZREVRANGE ranking 0 -1 WITHSCORES
```

---

# Databases Lógicos

Redis possui databases numerados.

Selecionar:

```text
SELECT 1
```

Voltar:

```text
SELECT 0
```

Consultar database atual:

```text
CLIENT INFO
```

---

# Informações do servidor

Informações gerais:

```text
INFO
```

Memória:

```text
INFO memory
```

Persistência:

```text
INFO persistence
```

Clientes:

```text
CLIENT LIST
```

---

# Teste rápido

```text
SET nome Joao

GET nome

SET contador 0

INCR contador

GET contador
```

---

# Persistência

Redis pode utilizar:

- RDB;
- AOF;
- RDB + AOF;
- somente memória.

No projeto, os dados ficam em:

```text
/data
```

Volume:

```yaml
volumes:

    - redis_data:/data
```

---

# Verificando persistência

Criar:

```text
SET teste funcionando
```

Pare o container:

```bash
docker compose stop redis
```

Inicie novamente:

```bash
docker compose --profile redis up -d
```

Consulte:

```text
GET teste
```

Se a persistência estiver configurada corretamente, o valor continuará existindo.

---

# Autenticação

Caso exista senha:

```text
AUTH redis123
```

Ou:

```bash
docker exec -it redis_db \
    redis-cli \
    -a "redis123"
```

---

# Problemas comuns

## NOAUTH Authentication Required

Autentique:

```text
AUTH senha
```

---

## Porta ocupada

Erro:

```text
port is already allocated
```

Alterar:

```ini
REDIS_PORT=6380
```

Depois:

```bash
docker compose \
    --profile redis \
    up -d \
    --force-recreate
```

---

## Redis não pede senha

Verifique se o Compose utiliza a variável:

```ini
REDIS_PASSWORD
```

Ela não possui efeito automaticamente apenas por existir no `.env`.

---

## Dados desapareceram

Verifique:

- volume montado;
- persistência configurada;
- utilização de `down -v`;
- configuração RDB/AOF.

---

## Container reiniciando

Consultar:

```bash
docker compose logs redis
```

ou

```bash
docker inspect redis_db
```

---

# Healthcheck

Consultar:

```bash
docker compose ps
```

ou

```bash
docker inspect \
    --format='{{json .State.Health}}' \
    redis_db
```

Status esperado:

```text
healthy
```

---

# Cuidados

Evite executar:

```text
FLUSHDB
```

Apaga todo o database atual.

Também evite:

```text
FLUSHALL
```

Apaga todos os databases da instância.

---

# Persistência x Cache

Nem todo dado deve ficar apenas no Redis.

Utilize Redis para:

- cache;
- sessões;
- tokens;
- filas;
- dados temporários.

Dados permanentes devem permanecer em um banco persistente como PostgreSQL, MySQL ou MongoDB.

---

# Boas práticas

- Utilize expiração sempre que possível.
- Não utilize Redis como banco principal sem planejamento.
- Configure autenticação.
- Utilize volumes.
- Monitore consumo de memória.
- Utilize chaves padronizadas.
- Evite valores gigantes.
- Faça backup quando utilizar persistência.
- Utilize nomes de chaves consistentes.
- Monitore comandos lentos (`SLOWLOG`).

---

# Cassandra

## O que é?

Apache Cassandra é um banco de dados **NoSQL distribuído**, orientado ao modelo de **famílias de colunas (Wide Column Store)**.

Foi desenvolvido para aplicações que precisam de:

- alta disponibilidade;
- escalabilidade horizontal;
- tolerância a falhas;
- replicação entre nós;
- milhares ou milhões de gravações por segundo;
- grandes volumes de dados.

Ao contrário de bancos relacionais, Cassandra foi projetado para que o banco nunca tenha um único ponto de falha.

---

# Quando utilizar Cassandra?

Cassandra é indicado para:

- IoT;
- Telemetria;
- Logs;
- Eventos;
- Big Data;
- Sistemas Financeiros;
- Monitoramento;
- Redes Sociais;
- Histórico de Eventos;
- Sistemas distribuídos.

Quando o principal requisito for:

- disponibilidade;
- escrita em larga escala;
- replicação;
- distribuição de dados.

---

# Como Cassandra funciona?

Em Cassandra não existe um único servidor central.

Os dados são distribuídos entre vários nós.

Exemplo:

```text
                Cluster

      ┌──────────┬──────────┬──────────┐

      ▼          ▼          ▼

    Node 1     Node 2     Node 3

      │          │          │

      └──── Dados Replicados ─────┘
```

Mesmo que um servidor fique indisponível, os demais continuam respondendo.

---

# Principais conceitos

| Cassandra | Equivalente aproximado |
|------------|-----------------------|
| Cluster | Conjunto de servidores |
| Node | Servidor |
| Datacenter | Agrupamento de nós |
| Keyspace | Database |
| Table | Tabela |
| Row | Registro |
| Partition Key | Chave de distribuição |
| Clustering Column | Ordenação interna |
| CQL | Cassandra Query Language |

---

# CQL

CQL significa:

```text
Cassandra Query Language
```

Sua sintaxe lembra SQL.

Exemplo:

```sql
SELECT *

FROM clientes;
```

Entretanto, Cassandra não possui o mesmo funcionamento de um banco relacional.

---

# Iniciando Cassandra

```bash
docker compose --profile cassandra up -d
```

Verificar:

```bash
docker compose ps
```

Saída esperada:

```text
NAME

cassandra_db

STATUS

Up (healthy)
```

---

# Logs

```bash
docker compose logs -f cassandra
```

Ou:

```bash
docker logs -f cassandra_db
```

Últimas linhas:

```bash
docker logs --tail 100 cassandra_db
```

---

# Tempo de inicialização

Cassandra normalmente leva mais tempo para iniciar do que MongoDB e Redis.

Dependendo da máquina, pode levar alguns minutos até que o Healthcheck seja aprovado.

---

# Recursos necessários

Antes de iniciar Cassandra, recomenda-se verificar os recursos disponíveis:

```bash
docker stats
```

Em máquinas com pouca memória, evite iniciar simultaneamente:

- Oracle;
- Cassandra;
- PostgreSQL;
- outros bancos pesados.

---

# Acessando o terminal

```bash
docker exec -it cassandra_db cqlsh
```

Especificando host:

```bash
docker exec -it cassandra_db \
    cqlsh localhost 9042
```

---

# Keyspaces

Listar:

```sql
DESCRIBE KEYSPACES;
```

Criar:

```sql
CREATE KEYSPACE appdb

WITH replication = {

'class':'SimpleStrategy',

'replication_factor':1

};
```

Selecionar:

```sql
USE appdb;
```

---

# Criando tabelas

```sql
CREATE TABLE clientes (

id UUID PRIMARY KEY,

nome TEXT,

email TEXT,

criado_em TIMESTAMP

);
```

---

# Inserindo registros

```sql
INSERT INTO clientes (

id,

nome,

email,

criado_em

)

VALUES (

uuid(),

'Renato',

'renato@email.com',

toTimestamp(now())

);
```

---

# Consultando

```sql
SELECT *

FROM clientes;
```

Consultar um cliente:

```sql
SELECT *

FROM clientes

WHERE id = uuid();
```

---

# Atualizando

```sql
UPDATE clientes

SET nome='Renato Justino'

WHERE id=uuid();
```

---

# Removendo

```sql
DELETE

FROM clientes

WHERE id=uuid();
```

---

# Partition Key

A Partition Key determina onde os dados serão armazenados.

Ela influencia diretamente:

- desempenho;
- distribuição;
- escalabilidade;
- balanceamento dos nós.

Escolher uma Partition Key inadequada pode gerar:

- Hotspots;
- baixa performance;
- nós sobrecarregados.

---

# Clustering Columns

As Clustering Columns organizam os dados dentro da mesma partição.

Exemplo:

```sql
PRIMARY KEY (

usuario_id,

data_evento

)
```

Onde:

```text
usuario_id
```

é a Partition Key.

Enquanto:

```text
data_evento
```

é a Clustering Column.

---

# Modelagem

Em Cassandra não se modela pensando nas tabelas.

Modela-se pensando nas consultas.

Primeiro define-se:

```text
Quais consultas existirão?
```

Depois:

```text
Como armazenar os dados?
```

Esse conceito é diferente do modelo relacional.

---

# Exemplo

Tabela para consultar clientes por e-mail:

```sql
CREATE TABLE clientes_por_email (

email TEXT PRIMARY KEY,

nome TEXT,

telefone TEXT

);
```

Consulta:

```sql
SELECT *

FROM clientes_por_email

WHERE email='renato@email.com';
```

---

# Persistência

Os dados ficam armazenados em:

```text
/var/lib/cassandra
```

Volume:

```yaml
volumes:

    - cassandra_data:/var/lib/cassandra
```

---

# Testando persistência

Criar tabela:

```sql
CREATE TABLE teste (

id UUID PRIMARY KEY,

mensagem TEXT

);
```

Inserir:

```sql
INSERT INTO teste (

id,

mensagem

)

VALUES (

uuid(),

'Persistência OK'

);
```

Parar:

```bash
docker compose stop cassandra
```

Iniciar:

```bash
docker compose --profile cassandra up -d
```

Consultar:

```sql
SELECT *

FROM teste;
```

---

# Problemas comuns

## Connection Refused

Verifique:

```bash
docker compose logs cassandra
```

O serviço pode ainda estar iniciando.

---

## NoHostAvailable

Possíveis causas:

- pouca memória;
- container reiniciando;
- porta incorreta;
- Healthcheck ainda não aprovado.

---

## Cassandra muito lento

Verifique:

```bash
docker stats
```

e

```bash
docker compose logs cassandra
```

---

## Container reiniciando

Consultar:

```bash
docker inspect cassandra_db
```

---

## Porta ocupada

Erro:

```text
port is already allocated
```

Altere:

```ini
CASSANDRA_PORT=9043
```

Depois:

```bash
docker compose \
    --profile cassandra \
    up -d \
    --force-recreate
```

---

# Healthcheck

Consultar:

```bash
docker compose ps
```

Detalhes:

```bash
docker inspect \
    --format='{{json .State.Health}}' \
    cassandra_db
```

Status esperado:

```text
healthy
```

---

# Boas práticas

- Modele pensando nas consultas.
- Planeje corretamente a Partition Key.
- Evite tabelas genéricas.
- Não utilize JOINs.
- Não espere comportamento igual ao SQL.
- Utilize replicação adequada.
- Faça backup regularmente.
- Utilize volumes nomeados.
- Aguarde completamente a inicialização antes de conectar aplicações.
- Monitore consumo de memória.
- Utilize versões fixas da imagem Docker.

---

# Volumes

Todos os bancos deste projeto utilizam **Docker Volumes Nomeados** para garantir a persistência dos dados.

Volumes configurados:

| Banco | Volume |
|--------|---------|
| MongoDB | `mongo_data` |
| Redis | `redis_data` |
| Cassandra | `cassandra_data` |

Consultar volumes:

```bash
docker volume ls
```

Inspecionar um volume:

```bash
docker volume inspect mongo_data
```

Remover um volume:

```bash
docker volume rm mongo_data
```

> **Atenção:** somente remova um volume quando realmente desejar apagar todos os dados armazenados.

---

# Redes Docker

Todos os containers são conectados automaticamente à rede criada pelo Docker Compose.

Consultar redes:

```bash
docker network ls
```

Inspecionar a rede do projeto:

```bash
docker network inspect dev-databases_default
```

Dentro da rede Docker os serviços se comunicam pelo nome do serviço.

Exemplos:

MongoDB

```text
mongo:27017
```

Redis

```text
redis:6379
```

Cassandra

```text
cassandra:9042
```

Não utilize:

```text
localhost
```

quando outra aplicação estiver executando dentro de um container Docker.

---

# Conectando aplicações

## MongoDB

```text
mongodb://admin:senha@mongo:27017/appdb?authSource=admin
```

---

## Redis

```text
redis://redis:6379
```

---

## Cassandra

```text
Host: cassandra

Porta: 9042
```

---

# Healthchecks

Todos os bancos possuem Healthcheck configurado no `docker-compose.yaml`.

Consultar:

```bash
docker compose ps
```

Exemplo:

```text
NAME             STATUS

mongo_db         Up (healthy)

redis_db         Up (healthy)

cassandra_db     Up (healthy)
```

Consultar detalhes:

MongoDB

```bash
docker inspect \
    --format='{{json .State.Health}}' \
    mongo_db
```

Redis

```bash
docker inspect \
    --format='{{json .State.Health}}' \
    redis_db
```

Cassandra

```bash
docker inspect \
    --format='{{json .State.Health}}' \
    cassandra_db
```

---

# Backup

## MongoDB

Exportar:

```bash
docker exec mongo_db \
mongodump \
--out /tmp/backup
```

Importar:

```bash
docker exec mongo_db \
mongorestore \
/tmp/backup
```

---

## Redis

Salvar snapshot:

```bash
docker exec redis_db \
redis-cli SAVE
```

---

## Cassandra

Criar snapshot:

```bash
docker exec cassandra_db \
nodetool snapshot
```

---

# Atualizando imagens

Verificar imagens:

```bash
docker images
```

Baixar versões mais recentes:

MongoDB

```bash
docker pull mongo:7
```

Redis

```bash
docker pull redis:7
```

Cassandra

```bash
docker pull cassandra:4
```

Depois recriar:

```bash
docker compose \
    --profile mongo \
    up -d \
    --force-recreate
```

---

# Reiniciando bancos

MongoDB

```bash
docker compose restart mongo
```

Redis

```bash
docker compose restart redis
```

Cassandra

```bash
docker compose restart cassandra
```

---

# Parando bancos

MongoDB

```bash
docker compose stop mongo
```

Redis

```bash
docker compose stop redis
```

Cassandra

```bash
docker compose stop cassandra
```

---

# Iniciando novamente

MongoDB

```bash
docker compose start mongo
```

Redis

```bash
docker compose start redis
```

Cassandra

```bash
docker compose start cassandra
```

---

# Removendo containers

MongoDB

```bash
docker compose rm -f mongo
```

Redis

```bash
docker compose rm -f redis
```

Cassandra

```bash
docker compose rm -f cassandra
```

---

# Derrubando o ambiente

Todos os containers ativos:

```bash
docker compose down
```

Volumes permanecem preservados.

---

# Removendo também os volumes

```bash
docker compose down -v
```

⚠️ Esse comando remove todos os volumes do projeto e apaga permanentemente os dados armazenados.

---

# Checklist antes de utilizar

- Docker instalado.
- Docker Compose funcionando.
- Arquivo `.env` configurado.
- `docker-compose.yaml` válido.
- Profile correto informado.
- Container iniciado.
- Healthcheck aprovado.
- Porta disponível.
- Volume criado.
- Aplicação utilizando a porta correta.

---

# Fluxo de utilização

```text
Docker Compose

        │

        ▼

Docker Compose Profile

        │

        ▼

Container

        │

        ▼

Healthcheck

        │

        ▼

Aplicação

        │

        ▼

Persistência (Volume)
```

---

# Comandos rápidos

## MongoDB

```bash
docker compose --profile mongo up -d
docker compose stop mongo
docker compose start mongo
docker compose restart mongo
docker compose logs -f mongo
docker exec -it mongo_db mongosh
```

---

## Redis

```bash
docker compose --profile redis up -d
docker compose stop redis
docker compose start redis
docker compose restart redis
docker compose logs -f redis
docker exec -it redis_db redis-cli
```

---

## Cassandra

```bash
docker compose --profile cassandra up -d
docker compose stop cassandra
docker compose start cassandra
docker compose restart cassandra
docker compose logs -f cassandra
docker exec -it cassandra_db cqlsh
```

---

# Resumo

Este documento apresentou os três bancos NoSQL disponíveis no projeto:

- **MongoDB**, um banco orientado a documentos, ideal para aplicações com estrutura flexível e dados em formato JSON/BSON.
- **Redis**, um banco chave-valor extremamente rápido, amplamente utilizado para cache, sessões, filas e armazenamento temporário.
- **Cassandra**, um banco distribuído orientado a colunas, indicado para sistemas com grande volume de dados, alta disponibilidade e escalabilidade horizontal.

Todos os bancos são gerenciados por um único `docker-compose.yaml`, utilizando **Docker Compose Profiles** para permitir a inicialização individual de cada serviço.

Com esta documentação, você possui uma referência completa para:

- iniciar e parar os bancos;
- conectar aplicações;
- acessar os terminais de cada banco;
- executar operações básicas;
- verificar logs e Healthchecks;
- realizar backup;
- solucionar problemas comuns;
- compreender as características de cada tecnologia.

---

# Documentos relacionados

* [`installation.md`](./installation.md) — instalação;
* [`environment.md`](./environment.md) — variáveis;
* [`docker-compose.md`](./docker-compose.md) — estrutura do Compose;
* [`commands.md`](./commands.md) — comandos;
* [`dbeaver.md`](./dbeaver.md) — clientes gráficos;
* [`nosql.md`](./nosql.md) — bancos não relacionais;
* [`troubleshooting.md`](./troubleshooting.md) — solução de erros;
* [`faq.md`](./faq.md) — perguntas frequentes;

