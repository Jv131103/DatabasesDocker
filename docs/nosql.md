# 🍃 Bancos de Dados NoSQL

## Objetivo

Este documento reúne as principais informações sobre os bancos NoSQL suportados pelo projeto **DatabasesDocker**.

Atualmente, o ambiente contempla:

* MongoDB;
* Redis;
* Cassandra.

Este guia explica:

* o que são bancos NoSQL;
* quando utilizar cada banco;
* as diferenças entre MongoDB, Redis e Cassandra;
* como iniciar e parar os serviços;
* como acessar os bancos pelo terminal;
* como testar o funcionamento;
* como configurar portas, usuários e senhas;
* como funciona a persistência;
* como conectar aplicações;
* quais cuidados devem ser tomados;
* como diagnosticar problemas comuns.

---

# Bancos disponíveis

| Banco     | Categoria                     | Porta interna | Profile ou serviço   | Container      |
| --------- | ----------------------------- | ------------: | -------------------- | -------------- |
| MongoDB   | Banco documental              |       `27017` | `mongo` ou `mongodb` | `mongo_db`     |
| Redis     | Chave-valor em memória        |        `6379` | `redis`              | `redis_db`     |
| Cassandra | Banco distribuído por colunas |        `9042` | `cassandra`          | `cassandra_db` |

> [!NOTE]
> Os nomes exatos dos services, profiles, containers, imagens e volumes devem ser confirmados no arquivo Compose real do projeto.

---

# O que é NoSQL?

NoSQL significa, de forma geral:

```text
Not Only SQL
```

Em português:

```text
Não apenas SQL
```

Esse termo representa bancos que não seguem obrigatoriamente o modelo tradicional de tabelas relacionais.

Bancos NoSQL podem organizar dados como:

* documentos;
* pares de chave e valor;
* famílias de colunas;
* grafos;
* séries temporais;
* estruturas em memória.

O NoSQL não substitui automaticamente um banco SQL.

A escolha depende do problema que precisa ser resolvido.

---

# SQL e NoSQL

## Banco relacional

Um banco SQL normalmente organiza os dados em tabelas:

```text
clientes
├── id
├── nome
├── email
└── criado_em
```

Os relacionamentos são definidos por:

* chaves primárias;
* chaves estrangeiras;
* constraints;
* joins.

## Banco NoSQL

Um banco NoSQL pode organizar os mesmos dados de outra maneira.

Exemplo documental:

```json
{
  "id": 1,
  "nome": "Renato",
  "email": "renato@email.com",
  "enderecos": [
    {
      "cidade": "São Paulo",
      "estado": "SP"
    }
  ]
}
```

Nesse caso, informações relacionadas podem ser armazenadas dentro do mesmo documento.

---

# Comparação geral

| Característica          | MongoDB                | Redis                  | Cassandra                     |
| ----------------------- | ---------------------- | ---------------------- | ----------------------------- |
| Modelo                  | Documentos             | Chave-valor            | Colunas distribuídas          |
| Armazenamento principal | Disco e memória        | Principalmente memória | Disco distribuído             |
| Consulta                | MongoDB Query Language | Comandos Redis         | CQL                           |
| Estrutura               | Flexível               | Chaves e estruturas    | Tabelas orientadas à consulta |
| Uso comum               | APIs e documentos      | Cache e filas          | Grandes volumes distribuídos  |
| Escalabilidade          | Horizontal             | Replicação e cluster   | Horizontal nativa             |
| Velocidade              | Alta                   | Muito alta             | Alta em escala                |
| Persistência            | Sim                    | Opcional/configurável  | Sim                           |
| Porta padrão            | `27017`                | `6379`                 | `9042`                        |

---

# Quando utilizar cada banco?

## Escolha MongoDB quando:

* os dados possuem estrutura flexível;
* os registros podem ter campos diferentes;
* a aplicação trabalha naturalmente com JSON;
* deseja armazenar documentos;
* precisa evoluir o schema com flexibilidade;
* trabalha com catálogos, conteúdos, eventos ou perfis;
* quer integração simples com aplicações Node.js ou Python.

## Escolha Redis quando:

* precisa de cache;
* precisa armazenar sessões;
* precisa de respostas muito rápidas;
* deseja implementar filas simples;
* precisa contar acessos;
* precisa controlar expiração;
* deseja armazenar dados temporários;
* precisa de pub/sub;
* deseja diminuir consultas repetidas ao banco principal.

## Escolha Cassandra quando:

* precisa armazenar grandes volumes;
* deseja distribuir dados entre nós;
* precisa de alta disponibilidade;
* possui muitas gravações;
* trabalha com séries temporais ou eventos;
* deseja tolerância a falhas;
* possui consultas previsíveis e planejadas previamente.

---

# Arquivo Compose NoSQL

O ambiente NoSQL pode estar definido:

* no mesmo `compose.yaml`;
* em um arquivo separado;
* em `docker-compose.yaml`;
* em `compose-nosql.yaml`.

Exemplo de execução com arquivo separado:

```bash
docker compose -f compose-nosql.yaml up -d
```

Exemplo utilizando um serviço específico:

```bash
docker compose -f compose-nosql.yaml up -d mongo
```

Exemplo utilizando profiles:

```bash
docker compose \
  -f compose-nosql.yaml \
  --profile mongo \
  up -d
```

Use o comando correspondente à estrutura real do projeto.

---

# Validando o arquivo Compose

Antes de iniciar:

```bash
docker compose -f compose-nosql.yaml config
```

Caso o projeto utilize o arquivo principal:

```bash
docker compose config
```

Esse comando permite identificar:

* erro de YAML;
* variáveis ausentes;
* portas resolvidas;
* volumes;
* redes;
* serviços;
* profiles.

---

# Configuração pelo `.env`

Exemplo conceitual:

```ini
TZ=America/Sao_Paulo

# MongoDB
MONGO_PORT=27017
MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=admin123
MONGO_INITDB_DATABASE=appdb

# Redis
REDIS_PORT=6379
REDIS_PASSWORD=redis123

# Cassandra
CASSANDRA_PORT=9042
CASSANDRA_CLUSTER_NAME=DatabasesDocker
CASSANDRA_DC=datacenter1
CASSANDRA_RACK=rack1
```

> [!WARNING]
> Os nomes das variáveis dependem do Compose real. Uma variável no `.env` só possui efeito se ela for utilizada no arquivo Compose ou no comando de inicialização do serviço.

---

# Portas externas e internas

Considere:

```yaml
ports:
  - "${MONGO_PORT:-27017}:27017"
```

Com:

```ini
MONGO_PORT=27018
```

O fluxo será:

```text
Aplicação
   │
   ▼
127.0.0.1:27018
   │
   ▼
Docker
   │
   ▼
MongoDB no container:27017
```

Portanto:

* `27018` é a porta externa;
* `27017` é a porta interna.

Para uma aplicação executada na máquina:

```text
Host: 127.0.0.1
Porta: 27018
```

Para uma aplicação em outro container:

```text
Host: mongo
Porta: 27017
```

---

# MongoDB

## Visão geral

MongoDB é um banco orientado a documentos.

Os dados são armazenados em documentos semelhantes a JSON.

Internamente, o MongoDB utiliza BSON, que permite tipos adicionais.

Exemplo:

```javascript
{
  nome: "Renato",
  idade: 22,
  ativo: true,
  cursos: ["Python", "Docker"],
  endereco: {
    cidade: "São Paulo",
    estado: "SP"
  }
}
```

---

## Principais conceitos

| MongoDB           | Comparação aproximada com SQL |
| ----------------- | ----------------------------- |
| Database          | Database                      |
| Collection        | Tabela                        |
| Document          | Linha                         |
| Field             | Coluna                        |
| `_id`             | Chave primária                |
| Embedded document | Estrutura relacionada interna |
| Reference         | Referência a outro documento  |

A comparação é apenas conceitual. MongoDB e bancos relacionais funcionam de maneiras diferentes.

---

## Iniciar o MongoDB

Caso use serviço diretamente:

```bash
docker compose -f compose-nosql.yaml up -d mongo
```

Caso use profile:

```bash
docker compose \
  -f compose-nosql.yaml \
  --profile mongo \
  up -d
```

Ou, conforme o projeto:

```bash
docker compose --profile mongodb up -d
```

---

## Verificar status

```bash
docker compose -f compose-nosql.yaml ps
```

Ou:

```bash
docker ps
```

---

## Acompanhar logs

```bash
docker compose \
  -f compose-nosql.yaml \
  logs -f mongo
```

Ou:

```bash
docker logs -f mongo_db
```

---

## Acessar pelo terminal

```bash
docker exec -it mongo_db \
  mongosh \
  -u admin \
  -p admin123 \
  --authenticationDatabase admin
```

A senha deve ser a configurada no `.env`.

---

## String de conexão

Exemplo local:

```text
mongodb://admin:admin123@127.0.0.1:27017/?authSource=admin
```

Com banco definido:

```text
mongodb://admin:admin123@127.0.0.1:27017/appdb?authSource=admin
```

### Componentes

```text
mongodb://
```

Protocolo utilizado.

```text
admin:admin123
```

Usuário e senha.

```text
127.0.0.1:27017
```

Host e porta.

```text
/appdb
```

Banco padrão.

```text
authSource=admin
```

Banco em que o usuário foi criado e será autenticado.

---

## Ver bancos

Dentro do `mongosh`:

```javascript
show dbs
```

---

## Selecionar ou criar um banco

```javascript
use appdb
```

O banco poderá aparecer em `show dbs` somente depois que possuir dados.

---

## Criar uma collection

```javascript
db.createCollection("clientes")
```

Listar collections:

```javascript
show collections
```

---

## Inserir documento

```javascript
db.clientes.insertOne({
  nome: "Renato",
  email: "renato@email.com",
  ativo: true
})
```

---

## Inserir vários documentos

```javascript
db.clientes.insertMany([
  {
    nome: "Ana",
    email: "ana@email.com",
    ativo: true
  },
  {
    nome: "João",
    email: "joao@email.com",
    ativo: false
  }
])
```

---

## Consultar documentos

```javascript
db.clientes.find()
```

Formatado:

```javascript
db.clientes.find().pretty()
```

Filtrar:

```javascript
db.clientes.find({
  ativo: true
})
```

Consultar um:

```javascript
db.clientes.findOne({
  email: "renato@email.com"
})
```

---

## Atualizar documento

```javascript
db.clientes.updateOne(
  {
    email: "renato@email.com"
  },
  {
    $set: {
      nome: "Renato Justino"
    }
  }
)
```

---

## Remover documento

```javascript
db.clientes.deleteOne({
  email: "renato@email.com"
})
```

---

## Criar índice

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

Listar índices:

```javascript
db.clientes.getIndexes()
```

---

## Ver usuário autenticado

```javascript
db.runCommand({
  connectionStatus: 1
})
```

---

## Teste rápido do MongoDB

```javascript
use appdb

db.teste.insertOne({
  nome: "Teste MongoDB",
  criadoEm: new Date()
})

db.teste.find()
```

---

## Persistência do MongoDB

O diretório de dados normalmente é:

```text
/data/db
```

Exemplo de volume:

```yaml
volumes:
  - mongo_data:/data/db
```

O volume deve ser declarado:

```yaml
volumes:
  mongo_data:
```

---

## Problemas comuns do MongoDB

### `Authentication failed`

Confira:

* usuário;
* senha;
* `authSource`;
* porta;
* banco de autenticação;
* credenciais usadas na primeira criação do volume.

Exemplo:

```text
authSource=admin
```

### Banco não aparece em `show dbs`

O banco pode estar vazio.

Insira um documento:

```javascript
db.teste.insertOne({
  teste: true
})
```

### Senha do `.env` foi alterada, mas não funciona

O usuário pode ter sido criado com a senha antiga no volume existente.

Altere diretamente no MongoDB ou recrie o volume somente se os dados puderem ser apagados.

---

# Redis

## Visão geral

Redis é um banco de dados baseado em chave e valor.

Ele trabalha principalmente em memória e é conhecido por sua alta velocidade.

Exemplo:

```text
nome → Renato
```

Uma chave identifica um valor.

Redis também suporta estruturas como:

* strings;
* hashes;
* lists;
* sets;
* sorted sets;
* streams;
* bitmaps;
* dados geoespaciais.

---

## Usos comuns

* cache;
* sessões;
* filas;
* rankings;
* contadores;
* rate limiting;
* pub/sub;
* armazenamento temporário;
* tokens com expiração;
* processamento de eventos.

---

## Iniciar Redis

```bash
docker compose -f compose-nosql.yaml up -d redis
```

Ou, com profile:

```bash
docker compose \
  -f compose-nosql.yaml \
  --profile redis \
  up -d
```

---

## Verificar status

```bash
docker compose -f compose-nosql.yaml ps
```

---

## Ver logs

```bash
docker compose \
  -f compose-nosql.yaml \
  logs -f redis
```

Ou:

```bash
docker logs -f redis_db
```

---

## Acessar o Redis CLI

Sem senha:

```bash
docker exec -it redis_db redis-cli
```

Com senha:

```bash
docker exec -it redis_db \
  redis-cli \
  -a 'redis123'
```

Outra opção:

```text
AUTH redis123
```

---

## Testar conexão

```text
PING
```

Resposta esperada:

```text
PONG
```

---

## Criar chave

```text
SET nome Renato
```

Consultar:

```text
GET nome
```

---

## Expiração

Criar com expiração de 60 segundos:

```text
SET token abc123 EX 60
```

Consultar tempo restante:

```text
TTL token
```

---

## Remover chave

```text
DEL nome
```

---

## Verificar existência

```text
EXISTS nome
```

---

## Trabalhar com números

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

Diminuir:

```text
DECR acessos
```

---

## Hashes

Criar:

```text
HSET usuario:1 nome Renato email renato@email.com
```

Consultar campo:

```text
HGET usuario:1 nome
```

Consultar todos:

```text
HGETALL usuario:1
```

---

## Lists

Adicionar ao início:

```text
LPUSH fila tarefa1
```

Adicionar ao final:

```text
RPUSH fila tarefa2
```

Listar:

```text
LRANGE fila 0 -1
```

Remover do início:

```text
LPOP fila
```

---

## Sets

Adicionar:

```text
SADD linguagens Python Java Docker
```

Consultar:

```text
SMEMBERS linguagens
```

Verificar membro:

```text
SISMEMBER linguagens Python
```

---

## Sorted sets

Adicionar pontuações:

```text
ZADD ranking 100 Renato
ZADD ranking 80 Ana
```

Consultar:

```text
ZRANGE ranking 0 -1 WITHSCORES
```

Em ordem decrescente:

```text
ZREVRANGE ranking 0 -1 WITHSCORES
```

---

## Seleção de database lógico

Redis normalmente disponibiliza databases numerados.

Selecionar:

```text
SELECT 1
```

Voltar ao padrão:

```text
SELECT 0
```

> [!NOTE]
> Esses databases são separações lógicas dentro da mesma instância, não equivalem aos databases tradicionais de MySQL ou PostgreSQL.

---

## Ver informações do servidor

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

## Teste rápido do Redis

```text
SET nome Renato
GET nome

SET contador 0
INCR contador
GET contador
```

---

## Persistência no Redis

Redis pode operar com mecanismos como:

* RDB;
* AOF;
* combinação de RDB e AOF;
* sem persistência, dependendo da configuração.

### RDB

Cria snapshots periódicos.

### AOF

Registra operações de escrita.

### Volume

O diretório comum de dados é:

```text
/data
```

Exemplo:

```yaml
volumes:
  - redis_data:/data
```

> [!IMPORTANT]
> Criar um volume não garante sozinho que o Redis esteja configurado para persistir todas as operações. A política de persistência precisa estar habilitada adequadamente no serviço.

---

## Redis com senha

Uma variável como:

```ini
REDIS_PASSWORD=redis123
```

não terá efeito automaticamente apenas por existir no `.env`.

O serviço precisa utilizar a variável no comando.

Exemplo conceitual:

```yaml
command:
  - redis-server
  - --requirepass
  - ${REDIS_PASSWORD}
```

A forma real deve ser validada no Compose.

---

## Cuidados com comandos Redis

Evite em ambientes com dados importantes:

```text
FLUSHDB
```

Apaga o database lógico atual.

```text
FLUSHALL
```

Apaga todos os databases lógicos da instância.

> [!CAUTION]
> Esses comandos podem apagar os dados imediatamente.

---

## Problemas comuns do Redis

### `NOAUTH Authentication required`

Autentique:

```text
AUTH redis123
```

Ou:

```bash
docker exec -it redis_db \
  redis-cli \
  -a 'redis123'
```

### Redis não exige senha mesmo com variável no `.env`

Verifique se `REDIS_PASSWORD` está sendo usada pelo comando do container.

### Dados desapareceram após reiniciar

Confirme:

* volume montado;
* RDB ou AOF;
* diretório correto;
* configuração de persistência;
* logs do Redis.

---

# Cassandra

## Visão geral

Apache Cassandra é um banco distribuído orientado a famílias de colunas.

Ele foi desenvolvido para:

* alta disponibilidade;
* escalabilidade horizontal;
* grande volume de dados;
* múltiplos nós;
* tolerância a falhas;
* altas taxas de escrita.

O Cassandra é diferente de um banco relacional tradicional.

Mesmo utilizando uma linguagem parecida com SQL, o modelo precisa ser desenhado de acordo com as consultas.

---

# Principais conceitos

| Cassandra         | Significado                             |
| ----------------- | --------------------------------------- |
| Cluster           | Conjunto de nós                         |
| Node              | Instância do Cassandra                  |
| Datacenter        | Agrupamento lógico ou físico de nós     |
| Keyspace          | Espaço semelhante a um database         |
| Table             | Estrutura de dados                      |
| Partition key     | Define onde os dados ficam distribuídos |
| Clustering column | Define organização dentro da partição   |
| CQL               | Linguagem de consulta do Cassandra      |

---

# CQL

CQL significa:

```text
Cassandra Query Language
```

A sintaxe lembra SQL, mas não possui o mesmo comportamento.

Exemplo:

```sql
SELECT *
FROM clientes;
```

Apesar da aparência semelhante, Cassandra não deve ser modelado como um banco relacional.

---

## Iniciar Cassandra

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d cassandra
```

Ou, com profile:

```bash
docker compose \
  -f compose-nosql.yaml \
  --profile cassandra \
  up -d
```

---

## Verificar status

```bash
docker compose -f compose-nosql.yaml ps
```

---

## Acompanhar logs

```bash
docker compose \
  -f compose-nosql.yaml \
  logs -f cassandra
```

Ou:

```bash
docker logs -f cassandra_db
```

Cassandra pode levar mais tempo para inicializar do que MongoDB e Redis.

---

## Recursos necessários

Cassandra normalmente consome mais memória.

Antes de iniciar, verifique:

```bash
docker stats
```

Evite subir Cassandra, Oracle e vários outros bancos pesados ao mesmo tempo em máquinas com pouca memória.

---

## Acessar o CQL Shell

```bash
docker exec -it cassandra_db cqlsh
```

Com host e porta:

```bash
docker exec -it cassandra_db \
  cqlsh localhost 9042
```

---

## Ver keyspaces

```sql
DESCRIBE KEYSPACES;
```

---

## Criar keyspace

Para ambiente local com um único nó:

```sql
CREATE KEYSPACE appdb
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

> [!WARNING]
> `SimpleStrategy` é adequada apenas para exemplos e ambientes simples. Ambientes distribuídos reais devem utilizar uma estratégia apropriada ao datacenter.

---

## Selecionar keyspace

```sql
USE appdb;
```

---

## Criar tabela

```sql
CREATE TABLE clientes_por_id (
  id UUID PRIMARY KEY,
  nome TEXT,
  email TEXT,
  criado_em TIMESTAMP
);
```

---

## Inserir dados

```sql
INSERT INTO clientes_por_id (
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

## Consultar

```sql
SELECT *
FROM clientes_por_id;
```

---

## Criar tabela orientada à consulta

Em Cassandra, é comum criar uma tabela para uma consulta específica.

Exemplo:

```sql
CREATE TABLE clientes_por_email (
  email TEXT PRIMARY KEY,
  id UUID,
  nome TEXT,
  criado_em TIMESTAMP
);
```

Consultar:

```sql
SELECT *
FROM clientes_por_email
WHERE email = 'renato@email.com';
```

---

## Atualizar

```sql
UPDATE clientes_por_email
SET nome = 'Renato Justino'
WHERE email = 'renato@email.com';
```

---

## Excluir

```sql
DELETE FROM clientes_por_email
WHERE email = 'renato@email.com';
```

---

## Partition key

A partition key determina:

* onde o dado será armazenado;
* como os dados serão distribuídos;
* quais consultas serão eficientes;
* tamanho das partições;
* capacidade de escalar.

Uma escolha ruim pode causar:

* partições muito grandes;
* concentração de dados;
* consultas lentas;
* desequilíbrio entre nós.

---

## Clustering columns

Clustering columns organizam registros dentro de uma partição.

Exemplo:

```sql
CREATE TABLE eventos_por_usuario (
  usuario_id UUID,
  criado_em TIMESTAMP,
  evento TEXT,
  PRIMARY KEY (
    usuario_id,
    criado_em
  )
) WITH CLUSTERING ORDER BY (
  criado_em DESC
);
```

Nesse exemplo:

* `usuario_id` é a partition key;
* `criado_em` é clustering column.

---

## Diferenças importantes para SQL

Cassandra não foi criado para:

* joins tradicionais;
* foreign keys;
* consultas arbitrárias;
* normalização relacional;
* transações complexas entre muitas tabelas.

A modelagem deve começar pelas consultas necessárias.

---

## Persistência do Cassandra

O Cassandra armazena dados em diretórios próprios da imagem.

Exemplo conceitual:

```yaml
volumes:
  - cassandra_data:/var/lib/cassandra
```

Confirme sempre o diretório correto da imagem utilizada.

---

## Teste rápido do Cassandra

```sql
DESCRIBE KEYSPACES;

CREATE KEYSPACE teste
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};

USE teste;

CREATE TABLE mensagens (
  id UUID PRIMARY KEY,
  texto TEXT
);

INSERT INTO mensagens (
  id,
  texto
)
VALUES (
  uuid(),
  'Cassandra funcionando'
);

SELECT *
FROM mensagens;
```

---

## Problemas comuns do Cassandra

### `Connection refused`

O serviço pode ainda estar inicializando.

Verifique:

```bash
docker logs -f cassandra_db
```

### `NoHostAvailable`

Possíveis causas:

* Cassandra ainda não está pronto;
* porta incorreta;
* pouca memória;
* container reiniciando;
* configuração de cluster inválida.

### Container encerrado por falta de memória

Verifique:

```bash
docker inspect cassandra_db
docker stats
```

Aumente os recursos do Docker ou reduza os serviços simultâneos.

---

# Profiles e serviços

Dependendo da configuração, o projeto pode usar:

```yaml
profiles:
  - mongo
```

ou serviços sem profile:

```yaml
services:
  mongo:
```

## Com profiles

```bash
docker compose \
  --profile mongo \
  up -d
```

## Sem profiles

```bash
docker compose up -d mongo
```

## Mais de um banco

```bash
docker compose \
  --profile mongo \
  --profile redis \
  up -d
```

Ou:

```bash
docker compose up -d mongo redis
```

---

# Iniciando todos os NoSQL

Exemplo sem profiles:

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d \
  mongo \
  redis \
  cassandra
```

Com profiles:

```bash
docker compose \
  -f compose-nosql.yaml \
  --profile mongo \
  --profile redis \
  --profile cassandra \
  up -d
```

> [!NOTE]
> Cassandra possui consumo maior. Verifique memória disponível antes de iniciar tudo.

---

# Parando os serviços

## Parar sem remover

MongoDB:

```bash
docker compose \
  -f compose-nosql.yaml \
  stop mongo
```

Redis:

```bash
docker compose \
  -f compose-nosql.yaml \
  stop redis
```

Cassandra:

```bash
docker compose \
  -f compose-nosql.yaml \
  stop cassandra
```

---

# Iniciando novamente

```bash
docker compose \
  -f compose-nosql.yaml \
  start mongo
```

---

# Removendo um container

```bash
docker compose \
  -f compose-nosql.yaml \
  rm -f mongo
```

Os dados devem permanecer se o volume nomeado não for removido.

---

# Derrubando o ambiente

```bash
docker compose \
  -f compose-nosql.yaml \
  down
```

Esse comando normalmente remove:

* containers;
* rede do projeto.

Os volumes nomeados permanecem.

---

# Apagando dados

```bash
docker compose \
  -f compose-nosql.yaml \
  down -v
```

> [!CAUTION]
> Esse comando remove os volumes associados ao projeto e pode apagar permanentemente MongoDB, Redis e Cassandra.

---

# Volumes

Exemplo de volumes:

```yaml
volumes:
  mongo_data:
  redis_data:
  cassandra_data:
```

Montagens conceituais:

```yaml
services:
  mongo:
    volumes:
      - mongo_data:/data/db

  redis:
    volumes:
      - redis_data:/data

  cassandra:
    volumes:
      - cassandra_data:/var/lib/cassandra
```

---

# Verificando volumes

```bash
docker volume ls
```

Inspecionar:

```bash
docker volume inspect <nome_do_volume>
```

Identificar volumes montados em um container:

```bash
docker inspect <container>
```

Procure a seção:

```text
Mounts
```

---

# Inicialização com volume existente

Assim como ocorre com bancos SQL, as variáveis de criação inicial normalmente atuam apenas quando o diretório de dados está vazio.

Alterar:

```ini
MONGO_INITDB_ROOT_PASSWORD=nova_senha
```

pode não mudar a senha do usuário existente em um volume já inicializado.

O mesmo princípio pode afetar:

* usuários;
* senhas;
* bancos;
* keyspaces;
* configurações persistidas;
* scripts de inicialização.

---

# Redes

O Compose cria uma rede padrão para os serviços.

Dentro dessa rede, aplicações podem utilizar o nome do serviço.

Exemplos:

```text
mongo:27017
redis:6379
cassandra:9042
```

Uma aplicação em outro container não deve utilizar:

```text
127.0.0.1
```

para acessar esses serviços.

Dentro do container, `127.0.0.1` aponta para o próprio container.

---

# Exemplos de conexão por aplicação

## MongoDB

Aplicação na máquina:

```text
mongodb://admin:senha@127.0.0.1:27017/appdb?authSource=admin
```

Aplicação em outro container:

```text
mongodb://admin:senha@mongo:27017/appdb?authSource=admin
```

---

## Redis

Aplicação na máquina:

```text
redis://127.0.0.1:6379
```

Com senha:

```text
redis://:senha@127.0.0.1:6379
```

Aplicação em outro container:

```text
redis://redis:6379
```

---

## Cassandra

Aplicação na máquina:

```text
Host: 127.0.0.1
Porta: 9042
```

Aplicação em outro container:

```text
Host: cassandra
Porta: 9042
```

---

# Healthchecks

Um healthcheck verifica se o serviço está respondendo.

## MongoDB

Exemplo conceitual:

```yaml
healthcheck:
  test:
    [
      "CMD",
      "mongosh",
      "--eval",
      "db.adminCommand('ping')"
    ]
  interval: 10s
  timeout: 5s
  retries: 10
  start_period: 20s
```

## Redis

```yaml
healthcheck:
  test:
    [
      "CMD",
      "redis-cli",
      "ping"
    ]
  interval: 10s
  timeout: 5s
  retries: 10
```

Com senha, o comando precisa considerar a autenticação.

## Cassandra

O healthcheck deve aguardar o tempo maior de inicialização.

Exemplo conceitual:

```yaml
healthcheck:
  test:
    [
      "CMD-SHELL",
      "cqlsh -e 'DESCRIBE KEYSPACES'"
    ]
  interval: 20s
  timeout: 10s
  retries: 15
  start_period: 60s
```

> [!NOTE]
> Os exemplos precisam ser adaptados à imagem, às credenciais e aos comandos disponíveis dentro de cada container.

---

# Verificando saúde

```bash
docker compose -f compose-nosql.yaml ps
```

Detalhes:

```bash
docker inspect <container>
```

Exemplo:

```bash
docker inspect \
  --format='{{json .State.Health}}' \
  mongo_db
```

---

# Logs

## MongoDB

```bash
docker logs -f mongo_db
```

## Redis

```bash
docker logs -f redis_db
```

## Cassandra

```bash
docker logs -f cassandra_db
```

Últimas linhas:

```bash
docker logs --tail 200 <container>
```

---

# Interfaces gráficas

## MongoDB

Possíveis ferramentas:

* MongoDB Compass;
* Mongo Express;
* DBeaver em edição compatível;
* outras interfaces MongoDB.

## Redis

Possíveis ferramentas:

* Redis Insight;
* redis-cli;
* DBeaver em edição compatível;
* outras interfaces Redis.

## Cassandra

Possíveis ferramentas:

* cqlsh;
* ferramentas do ecossistema Cassandra;
* DBeaver em edição compatível;
* clientes específicos.

Consulte:

```text
dbeaver.md
```

---

# Segurança

## Nunca publique senhas reais

O `.env` deve estar no `.gitignore`.

Suba apenas:

```text
.env.example
```

---

## Não exponha os bancos na internet

Evite publicar portas para todas as interfaces quando não for necessário.

Exemplo mais restrito:

```yaml
ports:
  - "127.0.0.1:${MONGO_PORT:-27017}:27017"
```

Isso limita o acesso à máquina local.

---

## Use autenticação

MongoDB e Redis não devem ficar expostos sem autenticação em redes não confiáveis.

Cassandra também deve possuir configuração de segurança adequada quando utilizado fora do ambiente local.

---

## Troque senhas de exemplo

Não utilize em ambientes reais:

```text
admin123
redis123
app123
```

---

## Princípio do menor privilégio

Evite utilizar usuários administradores nas aplicações.

Crie usuários com acesso apenas ao necessário.

---

## Criptografia

Em conexões remotas ou de produção, avalie:

* TLS;
* certificados;
* redes privadas;
* VPN;
* autenticação forte;
* secrets;
* firewall.

---

# Backup

Cada banco possui mecanismos próprios.

## MongoDB

Ferramenta:

```text
mongodump
```

Restauração:

```text
mongorestore
```

Exemplo conceitual:

```bash
docker exec mongo_db \
  mongodump \
  --username admin \
  --password admin123 \
  --authenticationDatabase admin \
  --out /backup
```

---

## Redis

O backup depende da configuração de persistência.

Possibilidades:

* copiar snapshot RDB;
* copiar AOF;
* utilizar mecanismos gerenciados;
* executar comandos de persistência;
* parar com segurança antes de copiar arquivos, conforme o procedimento adotado.

---

## Cassandra

Possibilidades:

* snapshots;
* ferramentas nativas;
* cópia controlada;
* estratégias específicas para cluster;
* backup por nó;
* ferramentas externas.

Um backup Cassandra precisa considerar:

* quantidade de nós;
* replicação;
* schema;
* consistência;
* restauração do cluster.

---

# Atualização de versões

Evite utilizar:

```yaml
image: mongo:latest
```

```yaml
image: redis:latest
```

```yaml
image: cassandra:latest
```

Prefira versões explícitas:

```yaml
image: mongo:8
```

```yaml
image: redis:8
```

```yaml
image: cassandra:5
```

Os números acima são apenas exemplos. Use as versões definidas e testadas no projeto.

Antes de atualizar:

1. confirme a versão atual;
2. leia as notas da versão;
3. faça backup;
4. teste a restauração;
5. crie um ambiente paralelo;
6. valide compatibilidade;
7. atualize a documentação;
8. mantenha o volume anterior até confirmar o novo ambiente.

---

# Erros comuns

## Porta já está ocupada

Sintoma:

```text
port is already allocated
```

Altere no `.env`:

```ini
MONGO_PORT=27018
```

Depois recrie o container:

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d \
  --force-recreate \
  mongo
```

---

## Container não inicia

Verifique:

```bash
docker ps -a
docker logs --tail 200 <container>
```

Possíveis causas:

* variável ausente;
* imagem incorreta;
* pouca memória;
* volume incompatível;
* porta ocupada;
* comando inválido;
* permissão.

---

## Aplicação não conecta

Confira:

* host;
* porta;
* nome do serviço;
* autenticação;
* rede;
* container saudável;
* string de conexão;
* porta externa ou interna.

---

## Alteração do `.env` não funciona

Recrie o container:

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d \
  --force-recreate
```

Credenciais persistidas podem precisar ser alteradas dentro do banco.

---

## Dados desapareceram

Verifique:

```bash
docker volume ls
docker inspect <container>
```

Possíveis causas:

* volume removido;
* projeto Compose com outro nome;
* serviço usando outro volume;
* persistência Redis não configurada;
* container iniciado sem montagem;
* uso de `down -v`.

---

## Cassandra muito lento

Possíveis causas:

* pouca memória;
* poucos recursos no Docker;
* primeira inicialização;
* outros bancos pesados ativos;
* disco lento;
* healthcheck prematuro.

Verifique:

```bash
docker stats
docker logs -f cassandra_db
```

---

# Diagnóstico rápido

Execute:

```bash
docker compose \
  -f compose-nosql.yaml \
  config
```

```bash
docker compose \
  -f compose-nosql.yaml \
  ps
```

```bash
docker ps -a
```

```bash
docker compose \
  -f compose-nosql.yaml \
  logs --tail 200
```

Depois inspecione o serviço específico:

```bash
docker inspect <container>
```

---

# Comandos rápidos

## Subir MongoDB

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d mongo
```

## Subir Redis

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d redis
```

## Subir Cassandra

```bash
docker compose \
  -f compose-nosql.yaml \
  up -d cassandra
```

## Ver status

```bash
docker compose \
  -f compose-nosql.yaml \
  ps
```

## Ver logs

```bash
docker compose \
  -f compose-nosql.yaml \
  logs -f
```

## Parar MongoDB

```bash
docker compose \
  -f compose-nosql.yaml \
  stop mongo
```

## Derrubar tudo preservando volumes

```bash
docker compose \
  -f compose-nosql.yaml \
  down
```

## Derrubar e apagar os dados

```bash
docker compose \
  -f compose-nosql.yaml \
  down -v
```

> [!CAUTION]
> O último comando apaga os volumes do ambiente.

---

# Boas práticas

* Nunca publique o `.env`.
* Não utilize senhas de exemplo em ambientes reais.
* Utilize volumes nomeados.
* Confirme a persistência do Redis.
* Aguarde Cassandra concluir a inicialização.
* Utilize `authSource` corretamente no MongoDB.
* Não use `localhost` entre containers.
* Fixe versões das imagens.
* Faça backup antes de atualizar.
* Consulte logs antes de apagar containers.
* Não remova volumes para resolver erros simples.
* Utilize portas diferentes.
* Monitore o consumo de memória.
* Crie modelos de dados de acordo com o banco escolhido.
* Não trate MongoDB ou Cassandra como um banco SQL.
* Evite armazenar dados permanentes somente no cache Redis.
* Teste restauração dos backups.

---

# Checklist do MongoDB

* [ ] Container está em execução.
* [ ] Porta está correta.
* [ ] Usuário e senha estão corretos.
* [ ] `authSource=admin` foi configurado.
* [ ] O volume está montado em `/data/db`.
* [ ] A string de conexão está correta.
* [ ] O banco possui ao menos um documento.
* [ ] O usuário foi criado no banco esperado.
* [ ] O healthcheck está aprovado.

---

# Checklist do Redis

* [ ] Container está em execução.
* [ ] A porta está correta.
* [ ] `PING` retorna `PONG`.
* [ ] A senha é exigida, caso configurada.
* [ ] A variável de senha está sendo usada no comando.
* [ ] O volume está montado.
* [ ] A persistência foi configurada.
* [ ] A aplicação não depende do Redis como única fonte permanente sem planejamento.
* [ ] Nenhum `FLUSHALL` foi executado acidentalmente.

---

# Checklist do Cassandra

* [ ] Container está em execução.
* [ ] Cassandra concluiu a inicialização.
* [ ] A máquina possui memória suficiente.
* [ ] A porta `9042` está acessível.
* [ ] `cqlsh` conecta.
* [ ] O keyspace existe.
* [ ] A estratégia de replicação está adequada.
* [ ] As tabelas foram modeladas de acordo com as consultas.
* [ ] A partition key foi planejada.
* [ ] O volume está montado.
* [ ] O healthcheck possui tempo suficiente.

---

# Documentos relacionados

* [`installation.md`](./installation.md) — instalação;
* [`environment.md`](./environment.md) — variáveis;
* [`docker-compose.md`](./docker-compose.md) — estrutura do Compose;
* [`commands.md`](./commands.md) — comandos;
* [`dbeaver.md`](./dbeaver.md) — clientes gráficos;
* [`sql.md`](./sql.md) — bancos relacionais;
* [`troubleshooting.md`](./troubleshooting.md) — solução de erros;
* [`faq.md`](./faq.md) — perguntas frequentes;

---

# Resumo

Os bancos NoSQL do projeto possuem finalidades diferentes:

```text
MongoDB
   └── documentos flexíveis

Redis
   └── cache, estruturas rápidas e dados temporários

Cassandra
   └── grande volume, distribuição e alta disponibilidade
```

O funcionamento geral é:

```text
.env
   │
   ▼
Docker Compose
   │
   ▼
Serviço ou profile
   │
   ▼
Container NoSQL
   │
   ├── porta
   ├── autenticação
   ├── healthcheck
   ├── rede
   └── volume
```

Para aplicações na máquina local:

```text
Host: 127.0.0.1
Porta: porta externa do .env
```

Para aplicações executadas em containers:

```text
Host: nome do serviço
Porta: porta interna
```

MongoDB deve ser utilizado como banco documental, Redis como armazenamento rápido e Cassandra como banco distribuído orientado às consultas planejadas.
