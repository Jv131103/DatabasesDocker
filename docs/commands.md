# 🛠️ Comandos Docker e Docker Compose

## Objetivo

Este documento reúne todos os comandos utilizados durante o gerenciamento do ambiente Docker deste projeto.

Além de apresentar cada comando, este guia explica:

- quando utilizá-lo;
- o que acontece internamente;
- quais são os riscos;
- boas práticas;
- diferenças entre comandos semelhantes.

Este documento serve como material de consulta para administração do ambiente.

---

# Como interpretar os comandos

Sempre que encontrar um comando neste documento:

```bash
docker compose up -d
```

Significa:

- `docker` → CLI do Docker;
- `compose` → gerenciamento do Docker Compose;
- `up` → cria e inicia os containers;
- `-d` → executa em segundo plano (Detached Mode).

---

# Índice

- Containers
- Docker Compose
- Logs
- Imagens
- Volumes
- Redes
- Inspeção
- Banco de Dados
- Limpeza
- Desenvolvimento
- Backup
- Atualizações
- Diagnóstico

---

# Containers

## Listar containers em execução

```bash
docker ps
```

Mostra apenas os containers ativos.

Exemplo:

```text
CONTAINER ID   IMAGE      STATUS      PORTS
mysql84        mysql:8.4  Up 2 min
```

Quando utilizar:

- verificar se um banco está ligado;
- confirmar o nome do container;
- verificar portas expostas.

---

## Listar todos os containers

```bash
docker ps -a
```

Mostra inclusive containers parados.

Muito útil para investigar falhas.

---

## Iniciar um container

```bash
docker start mysql84
```

Utilize quando o container já existe.

Não recria o ambiente.

---

## Parar um container

```bash
docker stop mysql84
```

Encerra o container de forma segura.

Os dados permanecem preservados.

---

## Reiniciar um container

```bash
docker restart mysql84
```

Equivale a:

```text
stop

↓

start
```

Muito utilizado após alterações de configuração.

---

## Pausar um container

```bash
docker pause mysql84
```

Suspende a execução sem desligar.

Pouco utilizado em bancos de dados.

---

## Retomar container pausado

```bash
docker unpause mysql84
```

---

## Remover um container

```bash
docker rm mysql84
```

Remove apenas o container.

O volume continua existindo.

Os dados permanecem salvos.

---

## Forçar remoção

```bash
docker rm -f mysql84
```

Remove mesmo que esteja em execução.

Utilize apenas quando necessário.

---

# Docker Compose

## Criar e iniciar ambiente

```bash
docker compose up
```

Cria:

- containers;
- redes;
- volumes.

Executa em primeiro plano.

---

## Executar em segundo plano

```bash
docker compose up -d
```

É o comando mais utilizado.

---

## Subir apenas MySQL

```bash
docker compose --profile mysql up -d
```

Utiliza Docker Profiles.

Somente o serviço MySQL será iniciado.

---

## Subir PostgreSQL

```bash
docker compose --profile postgres up -d
```

---

## Subir MariaDB

```bash
docker compose --profile mariadb up -d
```

---

## Derrubar ambiente

```bash
docker compose down
```

Remove:

- containers;
- rede.

Os volumes permanecem.

---

## Derrubar removendo volumes

```bash
docker compose down -v
```

⚠ Atenção.

Este comando apaga todos os bancos de dados.

---

## Recriar containers

```bash
docker compose up --force-recreate
```

Muito útil após alterar:

- compose.yaml
- .env

---

## Atualizar imagens

```bash
docker compose pull
```

Baixa versões mais recentes das imagens.

---

## Reconstruir imagens

```bash
docker compose build
```

Utilizado quando existe Dockerfile.

---

## Reiniciar ambiente

```bash
docker compose restart
```

---

# Logs

## Ver logs

```bash
docker compose logs
```

---

## Logs em tempo real

```bash
docker compose logs -f
```

Muito útil para acompanhar a inicialização.

---

## Logs de um container

```bash
docker logs mysql84
```

---

## Logs em tempo real

```bash
docker logs -f mysql84
```

---

## Últimas linhas

```bash
docker logs --tail 100 mysql84
```

---

# Acessando Containers

## Shell Linux

```bash
docker exec -it mysql84 bash
```

Caso a imagem utilize Alpine:

```bash
docker exec -it mysql84 sh
```

---

## Entrar no MySQL

```bash
docker exec -it mysql84 mysql -u root -p
```

---

## MariaDB

```bash
docker exec -it mariadb11 mariadb -u root -p
```

---

## PostgreSQL

```bash
docker exec -it postgres16 psql -U app -d appdb
```

---

## Oracle

```bash
docker exec -it oracle sqlplus app/app123
```

---

# Volumes

## Listar

```bash
docker volume ls
```

---

## Inspecionar

```bash
docker volume inspect mysql_data
```

---

## Remover

```bash
docker volume rm mysql_data
```

⚠ Remove permanentemente os dados.

---

## Remover volumes não utilizados

```bash
docker volume prune
```

---

# Redes

## Listar

```bash
docker network ls
```

---

## Inspecionar

```bash
docker network inspect bridge
```

---

## Remover

```bash
docker network rm nome_rede
```

---

# Imagens

## Listar imagens

```bash
docker images
```

---

## Remover imagem

```bash
docker rmi mysql:8.4
```

---

## Remover imagens não utilizadas

```bash
docker image prune
```

---

## Baixar imagem

```bash
docker pull mysql:8.4
```

---

# Sistema

## Informações do Docker

```bash
docker info
```

---

## Versão

```bash
docker version
```

---

## Espaço utilizado

```bash
docker system df
```

---

## Limpeza geral

```bash
docker system prune
```

Remove:

- containers parados;
- imagens sem uso;
- cache.

---

## Limpeza completa

```bash
docker system prune -a
```

Muito mais agressivo.

---

# Backup

## Exportar container

```bash
docker export mysql84 > mysql.tar
```

---

## Importar

```bash
docker import mysql.tar
```

---

## Backup do banco

Cada banco possui ferramentas próprias.

Exemplo MySQL:

```bash
mysqldump
```

PostgreSQL:

```bash
pg_dump
```

Oracle:

```bash
expdp
```

MongoDB:

```bash
mongodump
```

---

# Diagnóstico

## Inspecionar container

```bash
docker inspect mysql84
```

Mostra:

- IP;
- volumes;
- portas;
- variáveis;
- status;
- rede.

---

## Consumo de recursos

```bash
docker stats
```

Mostra:

- CPU;
- memória;
- rede;
- disco.

---

## Eventos do Docker

```bash
docker events
```

Útil para depuração.

---

# Boas práticas

- Utilize `docker compose` para iniciar e parar ambientes completos.
- Utilize `docker start` e `docker stop` apenas para containers já existentes.
- Nunca utilize `docker rm -f` sem necessidade.
- Nunca execute `docker compose down -v` sem confirmar que os dados podem ser descartados.
- Consulte os logs antes de remover containers.
- Prefira `docker compose logs -f` durante a inicialização dos bancos.
- Utilize `docker inspect` para investigar problemas de configuração.
- Faça backup dos dados antes de remover volumes.

---

# Erros comuns

## Container não encontrado

```text
No such container
```

Verifique:

```bash
docker ps -a
```

---

## Porta ocupada

```text
Bind for 3306 failed
```

Altere a porta no `.env` ou pare o serviço que está utilizando a porta.

---

## Container reiniciando continuamente

```text
Restarting (1)
```

Verifique:

```bash
docker logs <container>
```

---

## Volume inexistente

```text
No such volume
```

Liste os volumes:

```bash
docker volume ls
```

---

## Permissão negada

```text
permission denied
```

No Linux, verifique se o usuário possui permissão para utilizar o Docker.

---

# Fluxo recomendado

Durante o desenvolvimento, o fluxo mais comum será:

```text
Editar .env
        │
        ▼
docker compose down
        │
        ▼
docker compose up -d
        │
        ▼
docker compose logs -f
        │
        ▼
docker ps
        │
        ▼
Conectar pelo DBeaver
```

---

# Resumo

Os comandos apresentados neste documento cobrem praticamente todo o ciclo de vida de um ambiente Docker:

- criação;
- inicialização;
- parada;
- inspeção;
- monitoramento;
- atualização;
- limpeza;
- backup;
- diagnóstico.

Dominar esses comandos é essencial para administrar ambientes Docker de forma segura, eficiente e reproduzível.
