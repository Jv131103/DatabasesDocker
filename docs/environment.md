# 🌱 Variáveis de Ambiente (.env)

## Objetivo

Este documento explica detalhadamente como funciona o arquivo `.env` utilizado pelo projeto.

Ao final desta leitura você entenderá:

- O que são variáveis de ambiente.
- Por que utilizamos um arquivo `.env`.
- Como o Docker Compose interpreta essas variáveis.
- Como personalizar portas, usuários e senhas.
- Como adicionar novas variáveis.
- Boas práticas de segurança.
- Problemas comuns envolvendo arquivos `.env`.

---

# O que são Variáveis de Ambiente?

Variáveis de ambiente (Environment Variables) são pares de **chave = valor** utilizados para configurar aplicações sem alterar o código-fonte.

Exemplo:

```ini
MYSQL_PORT=3307
```

Nesse exemplo:

- chave → `MYSQL_PORT`
- valor → `3307`

Ao iniciar o Docker Compose, esse valor será lido automaticamente e utilizado durante a criação do container.

---

# Por que utilizar um arquivo .env?

Sem um arquivo `.env`, todas as configurações precisariam ficar escritas diretamente no `compose.yaml`.

Exemplo:

```yaml
ports:
  - "3307:3306"

environment:
  MYSQL_ROOT_PASSWORD: admin123
```

Isso apresenta alguns problemas:

- senhas ficam expostas;
- alteração exige modificar o Compose;
- difícil reutilização;
- pouca flexibilidade.

Utilizando `.env`, o Compose torna-se genérico.

```yaml
ports:
  - "${MYSQL_PORT}:3306"

environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

Agora basta alterar o `.env`.

---

# Como o Docker Compose utiliza essas variáveis?

Sempre que encontrar:

```yaml
${NOME_DA_VARIAVEL}
```

o Docker procura essa variável dentro do arquivo `.env`.

Exemplo.

Compose:

```yaml
ports:

- "${MYSQL_PORT}:3306"
```

.env

```ini
MYSQL_PORT=3307
```

Resultado:

```yaml
ports:

- "3307:3306"
```

Essa substituição acontece antes do container ser criado.

---

# Estrutura do arquivo

Exemplo:

```ini
TZ=America/Sao_Paulo

MYSQL_PORT=3307

MYSQL_ROOT_PASSWORD=admin123

MYSQL_DATABASE=appdb

MYSQL_USER=app

MYSQL_PASSWORD=app123
```

Cada linha representa uma configuração.

---

# Sintaxe

Sempre utilize:

```ini
NOME=VALOR
```

Exemplo:

```ini
POSTGRES_PORT=5432
```

Não utilize:

```ini
POSTGRES_PORT = 5432
```

Espaços podem causar comportamentos inesperados.

---

# Comentários

Comentários começam com:

```ini
#
```

Exemplo:

```ini
# Configuração do PostgreSQL
POSTGRES_PORT=5432
```

Eles são ignorados pelo Docker.

---

# Organização recomendada

Agrupe por banco.

```ini
# MariaDB

...

# MySQL

...

# PostgreSQL

...

# Oracle

...
```

Isso facilita futuras manutenções.

---

# Explicação de cada variável

## TZ

```ini
TZ=America/Sao_Paulo
```

Define o fuso horário utilizado pelo container.

Impactos:

- registros de log;
- timestamps;
- backups;
- datas automáticas.

Caso seja omitido, normalmente será utilizado UTC.

---

# MYSQL_PORT

```ini
MYSQL_PORT=3307
```

Define a porta exposta para o computador hospedeiro.

Internamente o MySQL continuará utilizando:

```
3306
```

Fluxo:

```
Computador

3307

↓

Docker

↓

3306

↓

MySQL
```

---

# MYSQL_ROOT_PASSWORD

Senha do usuário administrador.

Esse usuário possui acesso total ao banco.

Utilize apenas para administração.

Evite utilizar aplicações conectando com root.

---

# MYSQL_DATABASE

Banco criado automaticamente durante a primeira inicialização.

Caso o volume já exista, alterações posteriores não recriarão o banco automaticamente.

---

# MYSQL_USER

Usuário comum criado automaticamente.

Ideal para aplicações.

---

# MYSQL_PASSWORD

Senha do usuário comum.

Utilize uma senha forte em ambientes reais.

---

# Variáveis do PostgreSQL

As mesmas regras se aplicam.

```
POSTGRES_USER
POSTGRES_PASSWORD
POSTGRES_DB
POSTGRES_PORT
```

---

# Variáveis do MariaDB

```
MARIADB_PORT

MARIADB_DATABASE

MARIADB_USER

MARIADB_PASSWORD

MARIADB_ROOT_PASSWORD
```

Funcionam da mesma forma.

---

# Variáveis do Oracle

```
ORACLE_PORT

ORACLE_PASSWORD

ORACLE_APP_USER

ORACLE_APP_PASSWORD
```

---

# Como alterar portas

Exemplo.

De:

```ini
MYSQL_PORT=3307
```

Para:

```ini
MYSQL_PORT=3310
```

Depois execute:

```bash
docker compose down

docker compose up -d
```

A nova porta será utilizada.

---

# Como alterar usuários

Altere:

```ini
MYSQL_USER=renato
```

Da mesma forma para senha.

Lembre-se:

Se o banco já foi inicializado anteriormente, algumas imagens não recriam automaticamente usuários.

Nesses casos será necessário remover o volume ou criar o usuário manualmente.

---

# Ordem de leitura

Durante a inicialização acontece:

1.

Docker Compose lê `.env`

↓

2.

Substitui variáveis

↓

3.

Cria container

↓

4.

Executa imagem

↓

5.

Imagem configura banco

↓

6.

Container inicia

---

# Segurança

Nunca envie seu `.env` para o GitHub.

O correto é subir apenas:

```
.env.example
```

O `.env.example` deve conter apenas valores de exemplo.

Exemplo.

```ini
MYSQL_PASSWORD=sua_senha
```

Nunca:

```ini
MYSQL_PASSWORD=minhasenhapessoal123
```

---

# Versionamento

Arquivos recomendados:

```
.env

.gitignore

.env.example
```

No `.gitignore`

```
.env
```

Assim suas senhas nunca serão publicadas.

---

# Erros comuns

## Esquecer de copiar o .env

Erro.

```
variable is not set
```

Solução.

```bash
cp .env.example .env
```

---

## Alterar o .env com container ligado

O Docker não atualiza automaticamente.

É necessário recriar o container.

```bash
docker compose down

docker compose up -d
```

---

## Porta ocupada

```
Bind for 3307 failed
```

Troque:

```ini
MYSQL_PORT=3308
```

---

## Senha alterada

Alterar:

```ini
MYSQL_PASSWORD
```

não muda automaticamente usuários existentes.

Isso ocorre porque os dados ficam armazenados no volume.

---

# Boas práticas

✔ Nunca usar root na aplicação.

✔ Nunca subir `.env` para o GitHub.

✔ Utilizar senhas fortes.

✔ Organizar variáveis por banco.

✔ Manter comentários.

✔ Documentar novas variáveis.

✔ Utilizar nomes consistentes.

✔ Nunca remover variáveis utilizadas pelo Compose.

✔ Evitar reutilizar portas entre bancos.

---

# Fluxo resumido

```
.env

↓

Docker Compose

↓

Substituição das variáveis

↓

Container

↓

Banco de Dados
```

---

# Próximo documento

Após compreender o funcionamento do `.env`, recomenda-se seguir para:

- `docker-compose.md` → funcionamento detalhado do Docker Compose.
- `commands.md` → principais comandos para gerenciamento dos containers.
