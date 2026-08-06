# 🐳 Docker Compose

## Objetivo

Este documento explica detalhadamente como funciona o arquivo `compose.yaml` utilizado pelo projeto.

Ao final desta leitura você será capaz de:

- Entender o que é Docker Compose.
- Compreender a estrutura de um arquivo `compose.yaml`.
- Entender o papel de cada seção.
- Criar novos serviços.
- Adicionar novos bancos de dados.
- Configurar redes, volumes e variáveis de ambiente.
- Utilizar Docker Profiles.
- Interpretar Healthchecks.
- Realizar manutenção do ambiente.

---

# O que é Docker Compose?

Docker Compose é uma ferramenta oficial do Docker utilizada para definir e gerenciar aplicações compostas por múltiplos containers.

Ao invés de iniciar cada container manualmente utilizando vários comandos `docker run`, todas as configurações ficam centralizadas em um único arquivo chamado `compose.yaml`.

Exemplo:

```bash
docker compose up -d
```

Com apenas um comando, o Docker Compose:

- cria os containers;
- cria as redes;
- cria os volumes;
- aplica as variáveis de ambiente;
- inicia todos os serviços.

---

# Por que utilizar Docker Compose?

Imagine um projeto contendo:

- MySQL
- Redis
- MongoDB
- API
- Nginx

Sem Compose seriam necessários diversos comandos `docker run`, cada um contendo dezenas de parâmetros.

Com Docker Compose tudo fica documentado em um único arquivo.

Isso torna o ambiente:

- reproduzível;
- organizado;
- portátil;
- fácil de compartilhar;
- fácil de manter.

---

# Como o Docker Compose funciona?

O fluxo de inicialização acontece da seguinte forma:

```text
Usuário
      │
      ▼
docker compose up
      │
      ▼
Leitura do compose.yaml
      │
      ▼
Leitura do .env
      │
      ▼
Substituição das variáveis
      │
      ▼
Criação da rede
      │
      ▼
Criação dos volumes
      │
      ▼
Download das imagens (caso necessário)
      │
      ▼
Criação dos containers
      │
      ▼
Inicialização dos serviços
      │
      ▼
Execução do Healthcheck
      │
      ▼
Containers disponíveis
```

---

# Estrutura do compose.yaml

Um arquivo Compose normalmente possui esta estrutura:

```yaml
services:

volumes:

networks:
```

Cada seção possui uma responsabilidade específica.

---

# services

É a parte mais importante do Compose.

Cada item representa um container.

Exemplo:

```yaml
services:

  mysql:

  postgres:

  redis:
```

Neste projeto cada banco é um serviço independente.

---

# image

Define qual imagem será utilizada.

Exemplo:

```yaml
image: mysql:8.4
```

Significa:

- utilizar a imagem oficial do MySQL;
- versão 8.4;
- baixar automaticamente caso ainda não exista.

---

# container_name

```yaml
container_name: mysql84
```

Define o nome do container.

Sem essa configuração o Docker gera nomes aleatórios.

Exemplo:

```
adoring_einstein
```

Com `container_name`, fica mais fácil executar comandos como:

```bash
docker exec -it mysql84 bash
```

---

# profiles

Um dos recursos mais importantes deste projeto.

Exemplo:

```yaml
profiles:
  - mysql
```

Os Profiles permitem escolher quais serviços serão iniciados.

Exemplo:

```bash
docker compose --profile mysql up -d
```

Somente o MySQL será iniciado.

Isso evita consumir memória desnecessariamente.

---

# restart

```yaml
restart: unless-stopped
```

Define a política de reinicialização.

Neste projeto utilizamos:

```
unless-stopped
```

Significa:

- reinicia automaticamente após falhas;
- reinicia após reinicialização do computador;
- não reinicia caso tenha sido parado manualmente.

---

# environment

Utilizado para enviar variáveis para dentro do container.

Exemplo:

```yaml
environment:

  MYSQL_DATABASE: ${MYSQL_DATABASE}
```

O Docker Compose substitui automaticamente pelo valor existente no `.env`.

---

# ports

Responsável por mapear portas.

Exemplo:

```yaml
ports:

- "3307:3306"
```

Significado:

```
Computador

3307

↓

Container

3306

↓

MySQL
```

O lado esquerdo representa a máquina hospedeira.

O lado direito representa o container.

---

# volumes

Volumes são responsáveis pela persistência dos dados.

Exemplo:

```yaml
volumes:

- mysql_data:/var/lib/mysql
```

Fluxo:

```
Aplicação

↓

MySQL

↓

Volume Docker

↓

Disco
```

Mesmo removendo o container, o volume continua existindo.

---

# healthcheck

Responsável por verificar se o serviço realmente está funcionando.

Exemplo:

```yaml
healthcheck:

  test:

  interval:

  timeout:

  retries:
```

Enquanto o Healthcheck não passar, o container poderá aparecer como:

```
starting
```

Após sucesso:

```
healthy
```

---

# networks

Embora o projeto utilize a rede padrão do Docker Compose, é possível criar redes personalizadas.

Exemplo:

```yaml
networks:

  backend:
```

Todos os containers conectados nessa rede poderão se comunicar utilizando seus nomes.

---

# Volumes nomeados

Neste projeto utilizamos:

```
mysql_data

postgres_data

mariadb_data

oracle_data

mongo_data

redis_data

cassandra_data
```

Cada volume armazena os dados permanentemente.

---

# Ordem de inicialização

Sempre que um serviço é iniciado, ocorre:

1. Leitura do compose.
2. Leitura do `.env`.
3. Download da imagem (caso necessário).
4. Criação da rede.
5. Criação do volume.
6. Criação do container.
7. Inicialização.
8. Healthcheck.
9. Container disponível.

---

# Adicionando um novo banco

Exemplo:

```yaml
services:

  exemplo:

    image:

    container_name:

    profiles:

    environment:

    ports:

    volumes:

    restart:

    healthcheck:
```

Mantendo este padrão qualquer novo banco poderá ser incorporado ao projeto.

---

# Comandos principais

Subir ambiente

```bash
docker compose up -d
```

Subir apenas MySQL

```bash
docker compose --profile mysql up -d
```

Parar

```bash
docker compose down
```

Ver logs

```bash
docker compose logs
```

Ver logs em tempo real

```bash
docker compose logs -f
```

Recriar containers

```bash
docker compose up --force-recreate
```

Atualizar imagens

```bash
docker compose pull
```

Remover containers

```bash
docker compose down
```

Remover containers e volumes

```bash
docker compose down -v
```

---

# Boas práticas

- Nunca altere diretamente as imagens oficiais.
- Nunca coloque senhas no `compose.yaml`.
- Utilize sempre o arquivo `.env`.
- Utilize volumes para persistência.
- Utilize Healthcheck.
- Utilize Profiles para reduzir consumo de recursos.
- Nomeie corretamente os containers.
- Documente novos serviços adicionados.

---

# Erros comuns

## Porta já utilizada

```
Bind for 3306 failed
```

Solução:

Altere a porta no `.env`.

---

## Variável inexistente

```
Variable is not set
```

Solução:

Verifique o arquivo `.env`.

---

## Container reiniciando continuamente

```
Restarting (1)
```

Possíveis causas:

- senha inválida;
- volume corrompido;
- configuração incorreta;
- imagem incompatível.

Verifique:

```bash
docker compose logs
```

---

## Healthcheck falhando

```
unhealthy
```

Verifique:

- credenciais;
- inicialização do banco;
- logs do container.

---

# Resumo

O Docker Compose é responsável por orquestrar todo o ambiente do projeto.

Neste projeto ele gerencia:

- imagens Docker;
- containers;
- redes;
- volumes;
- variáveis de ambiente;
- Healthchecks;
- Profiles;
- persistência dos dados.

Sem ele seria necessário iniciar manualmente cada banco utilizando diversos comandos `docker run`, tornando o ambiente mais complexo, difícil de manter e pouco reproduzível.

---

# Próximo documento

Após compreender o Docker Compose, recomenda-se a leitura de:

- `commands.md`
- `architecture.md`
- `troubleshooting.md`
