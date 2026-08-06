# ❓ Perguntas Frequentes (FAQ)

## Objetivo

Este documento reúne as dúvidas mais comuns sobre o projeto **DatabasesDocker**.

Sempre consulte este documento antes de procurar um problema no `troubleshooting.md`.

A diferença entre os dois é simples:

- **FAQ** → dúvidas e comportamentos esperados.
- **Troubleshooting** → erros e problemas.

---

# Índice

- Sobre o projeto
- Docker
- Docker Compose
- Perfis (Profiles)
- Containers
- Volumes
- Redes
- Variáveis de ambiente
- Bancos de Dados
- DBeaver
- Segurança
- Desenvolvimento

---

# Sobre o Projeto

## O que é este projeto?

É um ambiente Docker que permite subir rapidamente bancos de dados SQL e NoSQL utilizando Docker Compose.

Todo o ambiente foi desenvolvido para facilitar estudos, desenvolvimento local e testes.

---

## Quais bancos são suportados?

Atualmente:

### SQL

- MariaDB
- MySQL
- PostgreSQL
- Oracle Database

### NoSQL

- MongoDB
- Redis
- Cassandra

---

## Posso utilizar este projeto em produção?

Não foi desenvolvido com esse objetivo.

O foco principal é:

- desenvolvimento;
- estudos;
- homologação;
- testes.

Para produção recomenda-se adicionar:

- backup automatizado;
- monitoramento;
- alta disponibilidade;
- gerenciamento de segredos;
- certificados SSL/TLS;
- políticas de segurança.

---

# Docker

## Preciso instalar o banco na minha máquina?

Não.

Todo banco será executado dentro de um container Docker.

Seu sistema operacional permanece limpo.

---

## O Docker precisa estar aberto?

Sim.

Antes de utilizar qualquer comando:

```bash
docker compose up
```

o Docker Engine deve estar em execução.

---

## Preciso saber Docker para usar este projeto?

Não.

Entretanto, conhecer conceitos básicos facilitará bastante a manutenção.

A documentação explica todos os conceitos utilizados.

---

# Docker Compose

## O que é Docker Compose?

É uma ferramenta responsável por criar e gerenciar vários containers utilizando apenas um arquivo YAML.

---

## Por que usar Docker Compose?

Porque ele:

- automatiza a criação dos containers;
- evita comandos enormes;
- facilita manutenção;
- torna o ambiente reproduzível.

---

## Posso alterar o compose.yaml?

Sim.

Apenas tome cuidado para manter:

- indentação;
- profiles;
- volumes;
- variáveis do `.env`.

---

# Profiles

## O que são Profiles?

Profiles permitem iniciar apenas determinados serviços.

Exemplo:

```bash
docker compose --profile mysql up -d
```

---

## Posso subir mais de um banco?

Sim.

Exemplo:

```bash
docker compose \
--profile mysql \
--profile redis \
up -d
```

Desde que utilizem portas diferentes.

---

## Posso subir todos os bancos?

Sim.

Basta ativar todos os profiles.

Lembre-se que isso aumentará significativamente o consumo de memória.

---

# Containers

## O que acontece quando paro um container?

Nada é apagado.

O banco apenas deixa de executar.

---

## O que acontece quando inicio novamente?

O banco continuará exatamente do ponto onde estava.

---

## Posso remover um container?

Sim.

Desde que o volume seja mantido.

---

## O que acontece ao remover um container?

Os dados permanecem armazenados no volume.

---

# Volumes

## O que é um Volume?

É o local onde os dados do banco ficam armazenados.

---

## Onde ficam os bancos?

Dentro dos volumes Docker.

Não ficam dentro do container.

---

## Posso remover um container sem perder dados?

Sim.

---

## Quando os dados são apagados?

Somente quando o volume é removido.

Exemplo:

```bash
docker compose down -v
```

ou

```bash
docker volume rm
```

---

## Posso fazer backup dos volumes?

Sim.

Inclusive é recomendado antes de qualquer atualização importante.

---

# Redes

## Preciso criar uma rede manualmente?

Não.

O Docker Compose cria automaticamente uma rede para o projeto.

---

## Os containers conseguem conversar entre si?

Sim.

Utilizando o nome do serviço.

Exemplo:

```
mysql
```

```
postgres
```

```
redis
```

---

## Posso utilizar localhost entre containers?

Não.

Dentro de outro container utilize o nome do serviço.

---

# Variáveis de Ambiente

## Para que serve o .env?

Centralizar configurações como:

- portas;
- usuários;
- senhas;
- timezone;
- banco padrão.

---

## Posso alterar a porta?

Sim.

Basta alterar:

```ini
MYSQL_PORT=3310
```

Depois recrie o container.

---

## Posso alterar a senha?

Sim.

Porém, bancos já inicializados podem manter as credenciais antigas.

---

## Preciso subir o .env para o GitHub?

Nunca.

Somente:

```
.env.example
```

---

# Bancos de Dados

## Posso utilizar vários bancos ao mesmo tempo?

Sim.

Desde que utilizem portas diferentes.

---

## Posso trocar a versão do banco?

Sim.

Exemplo:

```
mysql:8.4
```

↓

```
mysql:9
```

Porém, faça backup antes.

---

## Posso adicionar outro banco?

Sim.

Basta criar um novo serviço no Compose seguindo o padrão do projeto.

---

# DBeaver

## Posso utilizar outro cliente SQL?

Sim.

Exemplos:

- DataGrip
- HeidiSQL
- Beekeeper Studio
- VSCode SQLTools
- Azure Data Studio
- TablePlus

---

## Qual Host devo utilizar?

```
127.0.0.1
```

---

## Qual porta utilizar?

A porta configurada no `.env`.

---

## Por que localhost não funciona?

Em alguns sistemas ele resolve IPv6.

Por isso recomenda-se utilizar:

```
127.0.0.1
```

---

# Segurança

## Posso utilizar senha simples?

Em ambiente local sim.

Em produção nunca.

---

## Posso conectar utilizando root?

Tecnicamente sim.

Mas recomenda-se criar usuários específicos para aplicações.

---

## Posso deixar SSL desabilitado?

Somente em ambiente local.

Nunca em produção.

---

# Desenvolvimento

## Como atualizar as imagens?

```bash
docker compose pull
```

---

## Como atualizar os containers?

```bash
docker compose up -d
```

---

## Como recriar tudo?

```bash
docker compose down

docker compose up -d
```

---

## Como apagar completamente todos os bancos?

```bash
docker compose down -v
```

⚠ Essa operação remove permanentemente todos os dados.

---

## Como saber se o banco está funcionando?

```bash
docker compose ps
```

ou

```bash
docker compose logs -f
```

---

## Como descobrir o que aconteceu?

Sempre siga esta ordem:

1.

```bash
docker compose config
```

↓

2.

```bash
docker compose ps
```

↓

3.

```bash
docker compose logs
```

↓

4.

Consultar:

```
troubleshooting.md
```

---

# Boas práticas

- Utilize sempre o `.env`.
- Nunca publique senhas reais.
- Faça backup antes de atualizar bancos.
- Utilize volumes nomeados.
- Não utilize root em aplicações.
- Leia os logs antes de remover containers.
- Evite alterar o Compose sem documentar.

---

# Documentação relacionada

- `README.md`
- `installation.md`
- `environment.md`
- `docker-compose.md`
- `commands.md`
- `troubleshooting.md`
- `dbeaver.md`
- `roadmap.md`

---

# Ainda ficou com dúvida?

Caso a resposta não esteja neste documento:

1. Consulte o `troubleshooting.md`;
2. Consulte os logs do Docker;
3. Abra uma Issue contendo:
   - Sistema Operacional;
   - Versão do Docker;
   - Banco utilizado;
   - Comando executado;
   - Erro completo;
   - Logs relevantes.