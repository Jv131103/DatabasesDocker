# 📦 Instalação do Ambiente

## Objetivo

Este documento descreve todo o processo de preparação do ambiente de desenvolvimento para utilização do projeto **DatabasesDocker**.

Ao final deste guia você terá:

- Docker instalado
- Docker Compose funcionando
- Projeto configurado
- Arquivo `.env` personalizado
- Banco de dados iniciado
- Persistência dos dados configurada
- Ambiente pronto para conexão via DBeaver ou qualquer cliente SQL.

---

# Visão Geral

O projeto foi desenvolvido para eliminar a necessidade de instalar bancos de dados diretamente no sistema operacional.

Em vez disso, todos os bancos são executados dentro de containers Docker, garantindo:

- isolamento;
- facilidade de manutenção;
- portabilidade;
- ambientes reproduzíveis;
- facilidade para trocar versões de banco.

Todo o gerenciamento é realizado através do Docker Compose.

---

# Pré-requisitos

Antes de iniciar, certifique-se de possuir os seguintes softwares instalados.

## Docker

Responsável por executar os containers.

Download:

https://www.docker.com/products/docker-desktop/

Após instalar, confirme:

```bash
docker --version
```

Saída esperada:

```text
Docker version 28.x.x
```

---

## Docker Compose

O Compose é utilizado para subir todos os serviços definidos no arquivo `compose.yaml`.

Verifique:

```bash
docker compose version
```

Saída esperada:

```text
Docker Compose version v2.x.x
```

Caso o comando não exista, atualize sua instalação do Docker.

---

## Git (Opcional)

Caso deseje clonar o projeto diretamente do GitHub.

Verifique:

```bash
git --version
```

---

# Clonando o Projeto

Caso utilize Git:

```bash
git clone <URL_DO_REPOSITORIO>
```

Entre na pasta:

```bash
cd DatabasesDocker
```

Caso tenha baixado um arquivo ZIP, basta extraí-lo normalmente.

---

# Estrutura Inicial

Após abrir a pasta do projeto, você deverá possuir algo semelhante a:

```text
DatabasesDocker
│
├── compose.yaml
├── compose-nosql.yaml
├── .env.example
├── README.md
└── docs/
```

---

# Configuração do Arquivo .env

O projeto utiliza variáveis de ambiente para evitar que senhas e portas fiquem fixas dentro do Docker Compose.

Primeiramente copie:

Linux/macOS

```bash
cp .env.example .env
```

Windows PowerShell

```powershell
copy .env.example .env
```

Agora abra o arquivo `.env`.

Exemplo:

```ini
MYSQL_PORT=3307

MYSQL_ROOT_PASSWORD=admin123

MYSQL_DATABASE=appdb

MYSQL_USER=app

MYSQL_PASSWORD=app123
```

Cada banco possui suas próprias configurações.

Você pode alterar:

- portas
- usuário
- senha
- nome do banco
- timezone

Sempre que alterar o `.env`, recomenda-se recriar o container.

---

# Escolhendo o Banco

O projeto utiliza Docker Profiles.

Isso significa que apenas o banco solicitado será iniciado.

Por exemplo:

MariaDB

```bash
docker compose --profile mariadb up -d
```

MySQL

```bash
docker compose --profile mysql up -d
```

PostgreSQL

```bash
docker compose --profile postgres up -d
```

Oracle

```bash
docker compose --profile oracle up -d
```

MongoDB

```bash
docker compose --profile mongodb up -d
```

Redis

```bash
docker compose --profile redis up -d
```

Cassandra

```bash
docker compose --profile cassandra up -d
```

---

# O que acontece durante a inicialização?

Ao executar:

```bash
docker compose --profile mysql up -d
```

o Docker realiza automaticamente:

1. Download da imagem (caso ainda não exista).

2. Criação do container.

3. Criação do volume persistente.

4. Configuração da rede.

5. Aplicação das variáveis do `.env`.

6. Inicialização do banco.

7. Execução do Healthcheck.

Somente após o Healthcheck o banco estará pronto para conexões.

---

# Verificando se tudo está funcionando

Liste os containers:

```bash
docker ps
```

Exemplo:

```text
mysql84    Up (healthy)
```

Caso ainda apareça:

```text
starting
```

Aguarde alguns segundos.

Também é possível acompanhar:

```bash
docker compose logs -f
```

---

# Primeiro acesso

Após o container estar saudável ("healthy"), o banco estará pronto para receber conexões.

Neste momento você poderá utilizar:

- DBeaver
- DataGrip
- VSCode
- aplicações Java
- aplicações Python
- aplicações Node.js
- qualquer cliente SQL compatível

As credenciais utilizadas são exatamente as definidas no arquivo `.env`.

---

# Persistência dos Dados

Os dados **não ficam dentro do container**.

Eles ficam armazenados em Volumes Docker.

Isso significa que:

- remover o container NÃO apaga o banco;

- atualizar a imagem NÃO apaga o banco;

- reiniciar o computador NÃO apaga o banco.

Os dados somente serão removidos quando os volumes forem excluídos manualmente.

---

# Atualizando o Ambiente

Caso novas versões do projeto sejam disponibilizadas:

```bash
git pull
```

Depois:

```bash
docker compose pull
```

Em seguida:

```bash
docker compose up -d
```

Assim todas as imagens serão atualizadas.

---

# Desinstalação

Para remover apenas os containers:

```bash
docker compose down
```

Os dados permanecerão salvos.

---

Para remover containers e volumes:

```bash
docker compose down -v
```

⚠️ Esta operação apagará permanentemente todos os bancos de dados.

---

# Próximo passo

Após concluir a instalação, siga para:

- `environment.md` → configuração detalhada do `.env`;
- `docker-compose.md` → arquitetura do Docker Compose;
- `dbeaver.md` → conexão utilizando interface gráfica.

---

# Resumo

Ao finalizar este processo você terá:

- Docker configurado;
- Docker Compose funcional;
- Projeto instalado;
- Arquivo `.env` personalizado;
- Banco iniciado;
- Persistência habilitada;
- Ambiente pronto para desenvolvimento.
