# 🩺 Solução de Problemas — Docker e Bancos de Dados

## Objetivo

Este documento reúne procedimentos de diagnóstico e soluções para os principais problemas que podem ocorrer ao utilizar o projeto **DatabasesDocker**.

Ele deve ser consultado quando:

* um container não iniciar;
* um banco não aceitar conexões;
* o DBeaver apresentar erro;
* uma porta estiver ocupada;
* uma senha não funcionar;
* uma alteração no `.env` não for aplicada;
* um volume apresentar comportamento inesperado;
* um serviço ficar com status `unhealthy`;
* o Docker Compose apresentar erro;
* houver risco de perda de dados.

> [!IMPORTANT]
> Antes de remover containers, imagens ou volumes, consulte os logs e confirme se os dados possuem backup.

---

# Sumário

* [Diagnóstico rápido](#diagnóstico-rápido)
* [Ordem recomendada de investigação](#ordem-recomendada-de-investigação)
* [Comandos seguros de diagnóstico](#comandos-seguros-de-diagnóstico)
* [Entendendo os estados dos containers](#entendendo-os-estados-dos-containers)
* [Problemas com Docker e Docker Compose](#problemas-com-docker-e-docker-compose)
* [Problemas com portas e conexões](#problemas-com-portas-e-conexões)
* [Problemas com variáveis de ambiente](#problemas-com-variáveis-de-ambiente)
* [Problemas com autenticação](#problemas-com-autenticação)
* [Problemas com volumes e persistência](#problemas-com-volumes-e-persistência)
* [Problemas específicos do MySQL e MariaDB](#problemas-específicos-do-mysql-e-mariadb)
* [Problemas específicos do PostgreSQL](#problemas-específicos-do-postgresql)
* [Problemas específicos do Oracle](#problemas-específicos-do-oracle)
* [Problemas específicos de bancos NoSQL](#problemas-específicos-de-bancos-nosql)
* [Problemas no DBeaver](#problemas-no-dbeaver)
* [Procedimento de recuperação](#procedimento-de-recuperação)
* [Coleta de informações para suporte](#coleta-de-informações-para-suporte)
* [Tabela de risco dos comandos](#tabela-de-risco-dos-comandos)
* [Checklist final](#checklist-final)

---

# Diagnóstico rápido

Quando algo não funcionar, execute primeiro:

```bash
docker compose config
docker compose ps
docker ps -a
docker compose logs --tail 200
```

Esses comandos respondem quatro perguntas importantes:

1. O arquivo Compose é válido?
2. As variáveis do `.env` foram substituídas?
3. O container foi criado?
4. Qual erro aparece nos logs?

Depois verifique:

```bash
docker inspect <nome_do_container>
```

Exemplo:

```bash
docker inspect mysql84
```

Para acompanhar os logs em tempo real:

```bash
docker compose logs -f
```

Ou apenas de um serviço:

```bash
docker compose logs -f mysql
```

Para sair do acompanhamento sem parar o container:

```text
Ctrl + C
```

---

# Ordem recomendada de investigação

Siga esta ordem para evitar alterações desnecessárias:

```text
1. Confirmar que o Docker está funcionando
             ↓
2. Validar o compose.yaml
             ↓
3. Validar as variáveis do .env
             ↓
4. Verificar o status do container
             ↓
5. Consultar os logs
             ↓
6. Confirmar portas e redes
             ↓
7. Testar a conexão dentro do container
             ↓
8. Testar a conexão pela máquina
             ↓
9. Verificar usuário, senha e banco
             ↓
10. Recriar somente o container
             ↓
11. Restaurar ou recriar o volume apenas em último caso
```

> [!WARNING]
> Não comece o diagnóstico executando `docker compose down -v`. Esse comando pode apagar permanentemente os bancos armazenados nos volumes.

---

# Comandos seguros de diagnóstico

Os comandos abaixo não devem apagar dados:

```bash
docker version
docker info
docker compose version
docker compose config
docker compose ps
docker ps
docker ps -a
docker compose logs
docker compose logs --tail 200
docker inspect <container>
docker stats
docker volume ls
docker network ls
```

Comandos que alteram o estado, mas normalmente preservam os dados:

```bash
docker stop <container>
docker start <container>
docker restart <container>
docker compose restart
docker compose down
docker compose up -d
```

Comandos destrutivos:

```bash
docker compose down -v
docker volume rm <volume>
docker volume prune
docker system prune --volumes
```

---

# Entendendo os estados dos containers

## `created`

O container foi criado, mas ainda não foi iniciado.

Verifique:

```bash
docker ps -a
```

Inicie:

```bash
docker start <container>
```

Ou pelo Compose:

```bash
docker compose --profile mysql up -d
```

---

## `running` ou `Up`

O processo principal do container está funcionando.

Isso não garante, sozinho, que o banco já esteja pronto para conexões.

---

## `starting`

O container está em execução, mas o healthcheck ainda não foi aprovado.

Acompanhe:

```bash
docker compose logs -f
```

---

## `healthy`

O container está funcionando e o healthcheck foi aprovado.

---

## `unhealthy`

O processo está ativo, mas o teste de saúde está falhando.

Verifique:

```bash
docker inspect <container>
```

Para visualizar apenas o healthcheck:

```bash
docker inspect \
  --format='{{json .State.Health}}' \
  <container>
```

---

## `exited`

O processo principal terminou.

Consulte:

```bash
docker logs <container>
```

---

## `restarting`

O container está iniciando, falhando e reiniciando continuamente.

Consulte imediatamente:

```bash
docker logs --tail 200 <container>
```

---

# Problemas com Docker e Docker Compose

## 1. Docker não está em execução

### Sintomas

```text
Cannot connect to the Docker daemon
```

```text
Is the docker daemon running?
```

No Windows também pode aparecer:

```text
error during connect
```

### Causa provável

O Docker Engine ou Docker Desktop não está iniciado.

### Verificação

```bash
docker info
```

### Solução no Windows e macOS

1. Abra o Docker Desktop.
2. Aguarde aparecer que o Docker Engine está ativo.
3. Execute:

```bash
docker info
```

### Solução no Linux

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Verifique:

```bash
sudo systemctl status docker
```

---

## 2. Permissão negada ao acessar o Docker

### Sintoma

```text
permission denied while trying to connect to the Docker daemon socket
```

### Causa provável

O usuário atual não possui acesso ao socket do Docker.

### Solução temporária

Execute o comando com `sudo`:

```bash
sudo docker ps
```

### Solução permanente no Linux

```bash
sudo usermod -aG docker "$USER"
```

Depois encerre a sessão e entre novamente.

Confirme:

```bash
groups
docker ps
```

> [!NOTE]
> Participar do grupo `docker` concede permissões elevadas no sistema. Faça isso apenas em uma máquina confiável.

---

## 3. Comando `docker compose` não encontrado

### Sintomas

```text
docker: 'compose' is not a docker command
```

```text
docker compose: command not found
```

### Causa provável

O plugin Docker Compose v2 não está instalado ou o Docker está desatualizado.

### Verificação

```bash
docker compose version
```

### Solução

Atualize o Docker Desktop ou instale o plugin Compose correspondente ao seu sistema.

Após a instalação:

```bash
docker compose version
```

### Observação

O comando moderno é:

```bash
docker compose
```

O comando antigo utiliza hífen:

```bash
docker-compose
```

Este projeto deve priorizar o Compose v2.

---

## 4. Arquivo Compose não encontrado

### Sintoma

```text
no configuration file provided: not found
```

### Causa provável

O comando foi executado fora da pasta que contém o arquivo Compose.

### Verificação

Linux/macOS:

```bash
pwd
ls -la
```

Windows PowerShell:

```powershell
Get-Location
Get-ChildItem
```

### Solução

Entre na pasta correta:

```bash
cd DatabasesDocker
```

Depois execute:

```bash
docker compose config
```

Caso o arquivo possua outro nome:

```bash
docker compose -f docker-compose.yaml config
```

---

## 5. Erro de sintaxe ou indentação no YAML

### Sintomas

```text
yaml: line X: did not find expected key
```

```text
mapping values are not allowed in this context
```

```text
services must be a mapping
```

### Causa provável

* indentação incorreta;
* uso de tabulação;
* dois-pontos ausentes;
* lista escrita incorretamente;
* bloco posicionado no nível errado.

### Verificação

```bash
docker compose config
```

### Solução

Utilize espaços, preferencialmente dois por nível:

```yaml
services:
  mysql:
    image: mysql:8.4
    ports:
      - "${MYSQL_PORT:-3307}:3306"
```

Não utilize:

```yaml
services:
	mysql:
```

O YAML não deve ser indentado com tabulação.

---

## 6. Profile não foi ativado

### Sintomas

```text
no service selected
```

Ou nenhum banco é iniciado.

### Causa provável

O serviço possui `profiles`, mas o profile não foi informado.

### Verificação

```bash
docker compose config --profiles
```

### Solução

Use o profile correto:

```bash
docker compose --profile mysql up -d
```

Outros exemplos:

```bash
docker compose --profile mariadb up -d
docker compose --profile postgres up -d
docker compose --profile oracle up -d
```

### Verificação final

```bash
docker compose ps
```

---

## 7. Nome do serviço incorreto

### Sintoma

```text
no such service: mysql84
```

### Causa provável

Foi utilizado o nome do container no lugar do nome do serviço.

Exemplo:

```yaml
services:
  mysql:
    container_name: mysql84
```

Nesse caso:

* serviço: `mysql`;
* container: `mysql84`.

### Solução

Comandos Compose usam o nome do serviço:

```bash
docker compose logs mysql
docker compose restart mysql
```

Comandos Docker usam o nome do container:

```bash
docker logs mysql84
docker restart mysql84
```

---

## 8. Conflito de nome de container

### Sintoma

```text
Conflict. The container name "/mysql84" is already in use
```

### Causa provável

Já existe outro container com o mesmo `container_name`.

### Verificação

```bash
docker ps -a --filter name=mysql84
```

### Solução segura

Se o container antigo pertence ao mesmo projeto:

```bash
docker stop mysql84
docker rm mysql84
docker compose --profile mysql up -d
```

O volume nomeado continuará preservado, desde que não seja removido.

### Alternativa

Altere o `container_name` no Compose.

---

## 9. Imagem não encontrada ou tag inexistente

### Sintomas

```text
manifest unknown
```

```text
pull access denied
```

```text
repository does not exist
```

### Causas prováveis

* nome incorreto da imagem;
* tag inexistente;
* imagem privada;
* autenticação necessária.

### Verificação

Confira o campo:

```yaml
image: mysql:8.4
```

Tente baixar diretamente:

```bash
docker pull mysql:8.4
```

### Soluções

Corrija o nome ou a tag.

Para repositório privado:

```bash
docker login
```

Depois:

```bash
docker compose pull
```

---

## 10. Falha ao baixar imagem

### Sintomas

```text
TLS handshake timeout
```

```text
context deadline exceeded
```

```text
connection reset by peer
```

### Causas prováveis

* internet instável;
* proxy;
* VPN;
* DNS;
* firewall;
* indisponibilidade temporária do registry.

### Diagnóstico

```bash
docker pull mysql:8.4
```

### Soluções

1. Teste a conexão com a internet.
2. Desative temporariamente VPN ou proxy.
3. Reinicie o Docker.
4. Tente novamente:

```bash
docker compose pull
```

5. Em ambiente corporativo, configure o proxy no Docker Desktop ou daemon.

---

## 11. Falta de espaço em disco

### Sintomas

```text
no space left on device
```

```text
failed to register layer
```

### Verificação

```bash
docker system df
```

Linux:

```bash
df -h
```

### Solução segura

Primeiro identifique o que está ocupando espaço:

```bash
docker system df -v
```

Remova containers parados:

```bash
docker container prune
```

Remova imagens não utilizadas:

```bash
docker image prune
```

### Solução agressiva

```bash
docker system prune -a
```

> [!WARNING]
> Revise os itens antes de confirmar. Não utilize `--volumes` sem saber exatamente quais dados serão removidos.

---

## 12. Container encerra imediatamente

### Sintoma

O container aparece como:

```text
Exited (1)
```

### Verificação

```bash
docker ps -a
docker logs --tail 200 <container>
```

### Causas prováveis

* variável obrigatória ausente;
* senha inválida;
* volume incompatível;
* permissão no diretório;
* arquitetura incompatível;
* comando de inicialização inválido.

### Solução

Corrija o erro indicado no log e recrie:

```bash
docker compose up -d --force-recreate
```

---

## 13. Container reiniciando continuamente

### Sintoma

```text
Restarting (1)
```

### Diagnóstico

```bash
docker inspect <container>
docker logs --tail 200 <container>
```

Consulte o código de saída:

```bash
docker inspect \
  --format='{{.State.ExitCode}}' \
  <container>
```

### Soluções possíveis

* corrigir variável no `.env`;
* corrigir permissão do volume;
* usar imagem compatível;
* restaurar backup;
* remover somente o container e recriá-lo.

```bash
docker stop <container>
docker rm <container>
docker compose --profile <profile> up -d
```

---

## 14. Container com status `unhealthy`

### Sintoma

```text
Up (...) (unhealthy)
```

### Diagnóstico

```bash
docker inspect \
  --format='{{json .State.Health}}' \
  <container>
```

Consulte também:

```bash
docker logs --tail 200 <container>
```

### Causas prováveis

* comando do healthcheck incorreto;
* usuário ou senha incorretos;
* banco ainda inicializando;
* tempo de inicialização maior que o configurado;
* nome do banco incorreto;
* binário do teste não existe na imagem.

### Solução

Revise:

```yaml
healthcheck:
  test: [...]
  interval: 10s
  timeout: 5s
  retries: 10
  start_period: 30s
```

Bancos mais pesados podem precisar de um `start_period` maior.

---

# Problemas com portas e conexões

## 15. Porta já está em uso

### Sintomas

```text
Bind for 0.0.0.0:3307 failed: port is already allocated
```

```text
address already in use
```

### Verificação pelo Docker

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

### Verificação no Windows

```powershell
netstat -ano | findstr :3307
```

Depois identifique o processo:

```powershell
tasklist /FI "PID eq <PID>"
```

### Verificação no Linux

```bash
sudo ss -ltnp | grep 3307
```

### Verificação no macOS

```bash
lsof -i :3307
```

### Solução 1: alterar a porta

No `.env`:

```ini
MYSQL_PORT=3310
```

Recrie:

```bash
docker compose --profile mysql up -d --force-recreate
```

### Solução 2: parar o serviço conflitante

```bash
docker stop <container_conflitante>
```

---

## 16. Conexão recusada

### Sintomas

```text
Connection refused
```

```text
ECONNREFUSED
```

```text
Could not connect to server
```

### Causas prováveis

* container desligado;
* banco ainda inicializando;
* porta errada;
* serviço não está escutando;
* conexão usando a porta interna em vez da externa.

### Diagnóstico

```bash
docker compose ps
docker compose logs --tail 200
docker port <container>
```

### Solução

Confirme os dados do `.env`.

Exemplo:

```ini
MYSQL_PORT=3307
```

No DBeaver, use:

```text
Host: 127.0.0.1
Porta: 3307
```

Não use a porta interna `3306` caso ela tenha sido mapeada para `3307`.

---

## 17. Timeout de conexão

### Sintoma

```text
Connection timed out
```

### Diferença para `Connection refused`

* `refused`: o endereço respondeu, mas não há serviço aceitando conexão;
* `timed out`: não houve resposta dentro do tempo esperado.

### Causas prováveis

* firewall;
* VPN;
* IP incorreto;
* porta não publicada;
* serviço travado;
* máquina remota inacessível.

### Diagnóstico

```bash
docker compose ps
docker inspect <container>
```

Teste a porta.

Linux/macOS:

```bash
nc -vz 127.0.0.1 3307
```

Windows PowerShell:

```powershell
Test-NetConnection 127.0.0.1 -Port 3307
```

---

## 18. `localhost` não conecta, mas `127.0.0.1` funciona

### Causa provável

`localhost` pode ser resolvido como IPv6 (`::1`) enquanto o serviço está acessível em IPv4.

### Solução

Utilize:

```text
127.0.0.1
```

No DBeaver:

```text
Host: 127.0.0.1
```

---

## 19. Aplicação em outro container não conecta ao banco

### Erro comum

Utilizar:

```text
localhost
```

dentro do container da aplicação.

### Explicação

Dentro de um container, `localhost` aponta para o próprio container, não para o banco e nem para a máquina hospedeira.

### Solução

Use o nome do serviço Compose.

Exemplo:

```yaml
services:
  mysql:
  api:
```

Na API:

```text
Host: mysql
Porta: 3306
```

Entre containers, utilize a porta interna do serviço.

Não utilize:

```text
Host: 127.0.0.1
Porta: 3307
```

---

## 20. Containers não conseguem se comunicar

### Causas prováveis

* estão em redes diferentes;
* foram iniciados por projetos Compose diferentes;
* nome do serviço incorreto;
* container não está conectado à rede esperada.

### Verificação

```bash
docker network ls
docker network inspect <nome_da_rede>
```

### Solução

Defina uma rede comum:

```yaml
services:
  mysql:
    networks:
      - databases

  api:
    networks:
      - databases

networks:
  databases:
    driver: bridge
```

Recrie:

```bash
docker compose up -d --force-recreate
```

---

## 21. Porta publicada não aparece

### Verificação

```bash
docker compose ps
docker port <container>
```

### Causas prováveis

* serviço criado antes da alteração;
* erro na seção `ports`;
* profile incorreto;
* container antigo;
* Compose usando outro arquivo.

### Solução

Valide a configuração resolvida:

```bash
docker compose config
```

Depois recrie:

```bash
docker compose up -d --force-recreate
```

---

# Problemas com variáveis de ambiente

## 22. Variável não definida

### Sintomas

```text
The "MYSQL_PASSWORD" variable is not set
```

```text
variable is not set
```

### Causas prováveis

* arquivo `.env` não existe;
* nome incorreto;
* arquivo está em outra pasta;
* variável foi removida;
* comando executado em outro diretório.

### Verificação

```bash
docker compose config
```

### Solução

Crie o `.env`:

Linux/macOS:

```bash
cp .env.example .env
```

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

Depois edite os valores.

---

## 23. Alterações no `.env` não foram aplicadas

### Causa provável

O container existente ainda está usando a configuração anterior.

### Verificação

```bash
docker inspect <container>
```

### Solução

Recrie o container:

```bash
docker compose --profile mysql up -d --force-recreate
```

Ou:

```bash
docker compose down
docker compose --profile mysql up -d
```

> [!IMPORTANT]
> Alterar senhas no `.env` não necessariamente altera usuários que já foram criados dentro de um banco persistido.

---

## 24. Arquivo `.env` possui nome incorreto

### Exemplos problemáticos

```text
.env.txt
env
.env.example
```

### Verificação no Windows

```powershell
Get-ChildItem -Force
```

### Verificação no Linux/macOS

```bash
ls -la
```

### Solução

O arquivo usado automaticamente pelo Compose deve se chamar:

```text
.env
```

Também é possível indicar outro arquivo:

```bash
docker compose --env-file .env.dev up -d
```

---

## 25. Espaços, aspas ou caracteres especiais causam erro

### Exemplo recomendado

```ini
MYSQL_USER=app
MYSQL_PASSWORD=uma_senha_forte
```

### Possíveis problemas

```ini
MYSQL_PORT = 3307
MYSQL_PASSWORD=senha#123
```

Dependendo do formato, espaços e caracteres especiais podem ser interpretados de forma inesperada.

### Diagnóstico

```bash
docker compose config
```

### Solução

* evite espaços ao redor de `=`;
* revise a necessidade de aspas;
* use senhas compatíveis com o parser;
* confirme o valor resolvido pelo Compose;
* não inclua comentários depois do valor sensível.

---

## 26. Compose está usando outro arquivo ou projeto

### Sintomas

* nome dos containers diferente;
* variáveis inesperadas;
* portas que não correspondem ao `.env`;
* volumes com prefixos inesperados.

### Verificação

```bash
docker compose config
docker compose ls
```

### Solução

Informe explicitamente o arquivo:

```bash
docker compose -f compose.yaml config
```

E, se necessário, o arquivo de ambiente:

```bash
docker compose \
  -f compose.yaml \
  --env-file .env \
  --profile mysql \
  up -d
```

---

## 27. Valor padrão da variável não foi entendido

Exemplo no Compose:

```yaml
ports:
  - "${MYSQL_PORT:-3307}:3306"
```

Significado:

* usa `MYSQL_PORT` quando definida;
* usa `3307` quando ausente ou vazia.

### Diagnóstico

```bash
docker compose config
```

O resultado deve mostrar o valor final:

```yaml
ports:
  - target: 3306
    published: "3307"
```

---

# Problemas com autenticação

## 28. Usuário ou senha incorretos

### Sintomas no MySQL/MariaDB

```text
Access denied for user
```

### Sintoma no PostgreSQL

```text
password authentication failed for user
```

### Causas prováveis

* senha digitada incorretamente;
* usuário incorreto;
* banco já inicializado com credenciais antigas;
* aplicação usando outro `.env`;
* caracteres especiais interpretados incorretamente.

### Diagnóstico

Verifique a configuração:

```bash
docker compose config
```

Teste dentro do container.

MySQL:

```bash
docker exec -it mysql84 mysql -u root -p
```

MariaDB:

```bash
docker exec -it mariadb11 mariadb -u root -p
```

PostgreSQL:

```bash
docker exec -it postgres16 psql -U app -d appdb
```

### Solução

Caso consiga entrar como administrador, redefina a senha do usuário.

Não remova o volume antes de tentar corrigir as credenciais diretamente no banco.

---

## 29. Banco ou usuário não foi criado automaticamente

### Causa principal

As variáveis de inicialização normalmente são utilizadas apenas quando o diretório de dados está vazio.

Se o volume já possui um banco, mudar:

```ini
MYSQL_USER=
MYSQL_PASSWORD=
MYSQL_DATABASE=
```

ou:

```ini
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=
```

não recriará automaticamente a estrutura existente.

### Solução segura

Crie o usuário ou banco manualmente.

### Solução destrutiva para ambiente descartável

```bash
docker compose down -v
docker compose --profile mysql up -d
```

> [!CAUTION]
> Isso apaga os dados persistidos. Utilize somente quando o banco puder ser recriado.

---

## 30. Usuário existe, mas não tem permissão no banco

### MySQL/MariaDB

Entre como administrador:

```bash
docker exec -it mysql84 mysql -u root -p
```

Execute:

```sql
CREATE USER IF NOT EXISTS 'app'@'%' IDENTIFIED BY 'app123';

GRANT ALL PRIVILEGES
ON appdb.*
TO 'app'@'%';

FLUSH PRIVILEGES;
```

Confira:

```sql
SHOW GRANTS FOR 'app'@'%';
```

### PostgreSQL

Entre com usuário administrativo:

```bash
docker exec -it postgres16 \
  psql -U app -d appdb
```

Exemplo:

```sql
GRANT CONNECT ON DATABASE appdb TO dev;
GRANT USAGE ON SCHEMA public TO dev;
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO dev;
```

---

## 31. Usuário permitido apenas para `localhost`

### Sintoma

O usuário funciona dentro do container, mas não no DBeaver.

### Causa provável

No MySQL/MariaDB, a conta pode estar cadastrada apenas como:

```text
'app'@'localhost'
```

### Verificação

```sql
SELECT User, Host
FROM mysql.user;
```

### Solução

```sql
CREATE USER IF NOT EXISTS
'app'@'%'
IDENTIFIED BY 'app123';

GRANT ALL PRIVILEGES
ON appdb.*
TO 'app'@'%';

FLUSH PRIVILEGES;
```

---

# Problemas com volumes e persistência

## 32. Dados desapareceram após recriar o ambiente

### Causas possíveis

* o volume foi removido;
* o nome do projeto Compose mudou;
* o volume foi recriado com outro nome;
* foi usado `docker compose down -v`;
* o container está montando outro volume;
* o banco foi iniciado sem o volume esperado.

### Diagnóstico

```bash
docker volume ls
docker inspect <container>
```

Procure a seção:

```text
Mounts
```

### Verificação do Compose

```bash
docker compose config
```

### Solução

Confirme qual volume contém os dados e monte-o novamente.

Não apague volumes desconhecidos.

---

## 33. `docker compose down` não apagou os dados

### Explicação

Esse é o comportamento esperado quando são usados volumes nomeados.

O comando:

```bash
docker compose down
```

remove containers e redes do projeto, mas preserva os volumes nomeados.

### Para visualizar

```bash
docker volume ls
```

### Para apagar deliberadamente

```bash
docker compose down -v
```

> [!CAUTION]
> Isso apaga os dados armazenados pelos volumes associados ao projeto.

---

## 34. Volume está sendo usado

### Sintoma

```text
volume is in use
```

### Causa provável

Um container ainda está conectado ao volume.

### Verificação

```bash
docker ps -a --filter volume=<nome_do_volume>
```

### Solução

Pare e remova o container correspondente:

```bash
docker stop <container>
docker rm <container>
```

Depois:

```bash
docker volume rm <nome_do_volume>
```

Faça isso apenas quando os dados puderem ser apagados.

---

## 35. Permissão negada no diretório de dados

### Sintomas

```text
Permission denied
```

```text
could not change permissions
```

```text
cannot create directory
```

### Causas prováveis

* bind mount com proprietário incorreto;
* pasta protegida;
* restrições do sistema operacional;
* SELinux;
* diferença de usuário entre host e container.

### Diagnóstico

```bash
docker inspect <container>
docker logs <container>
```

### Recomendação

Para bancos locais, prefira volumes nomeados:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Em vez de bind mounts, quando não houver necessidade de acessar diretamente os arquivos do banco.

> [!WARNING]
> Não aplique `chmod -R 777` como solução padrão. Isso cria riscos de segurança e pode não corrigir o proprietário necessário.

---

## 36. Volume incompatível após atualização da versão do banco

### Sintomas

* container não inicia após trocar a imagem;
* mensagens sobre versão incompatível;
* diretório criado por outra versão;
* falha de upgrade.

### Causa provável

Os arquivos persistidos foram criados por outra versão principal do banco.

### Exemplo de alteração de risco

```yaml
image: postgres:15
```

para:

```yaml
image: postgres:16
```

### Solução correta

1. Faça backup com a versão antiga.
2. Pare o ambiente.
3. Crie um volume novo.
4. Inicie a nova versão.
5. Restaure o backup.
6. Valide os dados.
7. Só depois remova o volume antigo.

> [!CAUTION]
> Não presuma que trocar a tag da imagem realiza automaticamente uma migração segura entre versões principais.

---

# Problemas específicos do MySQL e MariaDB

## 37. `Public Key Retrieval is not allowed`

### Onde ocorre

Normalmente no DBeaver ou em aplicações Java conectando ao MySQL.

### Solução no DBeaver

Abra:

```text
Edit Connection
→ Driver properties
```

Configure:

```text
allowPublicKeyRetrieval = true
useSSL = false
```

Depois teste novamente.

### Observação de segurança

Desabilitar SSL é aceitável em desenvolvimento local isolado. Em ambientes remotos ou de produção, configure TLS corretamente.

---

## 38. Erro de SSL no MySQL ou MariaDB

### Sintomas

```text
SSL connection error
```

```text
The server does not support SSL
```

### Solução para ambiente local

No DBeaver, revise as propriedades:

```text
useSSL = false
```

Também desative a exigência de SSL na tela específica do driver, quando aplicável.

### Em produção

Não desative SSL indiscriminadamente. Configure certificados e conexão segura.

---

## 39. `Access denied for user 'root'`

### Causas possíveis

* senha root incorreta;
* volume foi inicializado com senha antiga;
* variável errada para a imagem utilizada;
* tentativa de acesso por origem não permitida.

### Diagnóstico

```bash
docker logs mysql84
docker compose config
```

### Teste interno

```bash
docker exec -it mysql84 mysql -u root -p
```

### Solução

Use a senha definida na primeira inicialização do volume.

Caso o ambiente seja descartável e a senha seja desconhecida:

```bash
docker compose down -v
docker compose --profile mysql up -d
```

> [!CAUTION]
> Esse procedimento apaga todos os dados do volume.

---

## 40. `Unknown database`

### Sintoma

```text
Unknown database 'appdb'
```

### Causas prováveis

* banco não foi criado;
* nome incorreto;
* volume existente foi inicializado antes de `MYSQL_DATABASE`;
* conexão aponta para outro container.

### Verificação

```bash
docker exec -it mysql84 mysql -u root -p
```

Depois:

```sql
SHOW DATABASES;
```

### Solução

```sql
CREATE DATABASE appdb;
```

Conceda permissão:

```sql
GRANT ALL PRIVILEGES
ON appdb.*
TO 'app'@'%';

FLUSH PRIVILEGES;
```

---

## 41. MySQL ainda não está pronto para conexões

### Sintomas

```text
Can't connect to MySQL server
```

```text
MySQL is initializing
```

### Diagnóstico

```bash
docker compose logs -f mysql
```

Verifique:

```bash
docker compose ps
```

### Solução

Aguarde o status `healthy`.

Para inicializações lentas, adicione:

```yaml
healthcheck:
  start_period: 30s
```

---

## 42. Comando do healthcheck do MySQL está incorreto

### Problema comum

Usar uma senha fixa ou usuário incorreto no teste.

### Exemplo mais seguro

```yaml
healthcheck:
  test:
    [
      "CMD-SHELL",
      "mysqladmin ping -h localhost -u root -p\"$${MYSQL_ROOT_PASSWORD}\""
    ]
  interval: 10s
  timeout: 5s
  retries: 10
  start_period: 30s
```

### Importante sobre `$`

Dentro de um comando executado no container, pode ser necessário utilizar:

```text
$$
```

para evitar que o Compose tente substituir a variável antes da execução.

---

# Problemas específicos do PostgreSQL

## 43. `password authentication failed for user`

### Sintoma

```text
FATAL: password authentication failed for user "app"
```

### Causas prováveis

* senha incorreta;
* usuário não existe;
* volume foi inicializado com senha antiga;
* conexão aponta para outro PostgreSQL.

### Teste interno

```bash
docker exec -it postgres16 \
  psql -U app -d appdb
```

### Verificação dos usuários

Dentro do `psql`:

```sql
\du
```

### Redefinir senha

```sql
ALTER USER app WITH PASSWORD 'nova_senha';
```

---

## 44. Banco PostgreSQL não existe

### Sintoma

```text
FATAL: database "appdb" does not exist
```

### Verificação

```bash
docker exec -it postgres16 \
  psql -U app -d postgres
```

Liste os bancos:

```sql
\l
```

### Solução

```sql
CREATE DATABASE appdb OWNER app;
```

---

## 45. PostgreSQL não aceita conexão externa

### Causas possíveis

* porta incorreta;
* container não está saudável;
* porta não publicada;
* configuração de autenticação;
* conexão apontando para `localhost` dentro de outro container.

### Diagnóstico

```bash
docker compose ps
docker port postgres16
docker logs postgres16
```

### Conexão pela máquina

```text
Host: 127.0.0.1
Porta: valor de POSTGRES_PORT
Database: valor de POSTGRES_DB
User: valor de POSTGRES_USER
```

### Conexão por outro container

```text
Host: postgres
Porta: 5432
```

---

## 46. `database files are incompatible with server`

### Sintoma

```text
The data directory was initialized by PostgreSQL version X
```

### Causa

O volume foi criado por outra versão principal do PostgreSQL.

### Solução

Realize migração por backup e restauração:

```bash
pg_dump
pg_restore
```

Não remova o volume antigo antes de validar o novo ambiente.

---

## 47. `pg_isready` aponta usuário ou banco incorreto

### Sintoma

O PostgreSQL funciona, mas aparece como `unhealthy`.

### Solução

Revise o healthcheck:

```yaml
healthcheck:
  test:
    [
      "CMD-SHELL",
      "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"
    ]
  interval: 10s
  timeout: 5s
  retries: 10
  start_period: 20s
```

Use `$$` quando a variável deve ser interpretada dentro do container.

---

# Problemas específicos do Oracle

## 48. Oracle demora muito para ficar disponível

### Sintomas

* container permanece em `starting`;
* conexão recusada nos primeiros minutos;
* alto consumo de memória;
* logs ainda mostram inicialização.

### Explicação

O Oracle normalmente exige mais recursos e pode levar mais tempo para concluir a preparação inicial.

### Diagnóstico

```bash
docker compose logs -f oracle
docker stats
```

### Soluções

* aguarde a mensagem de disponibilidade;
* aumente o `start_period`;
* aumente a memória destinada ao Docker;
* confirme espaço em disco;
* evite iniciar muitos bancos simultaneamente.

---

## 49. Erro de Service Name no Oracle

### Sintomas

```text
ORA-12514
```

```text
listener does not currently know of service requested
```

### Causa provável

Foi utilizado um SID ou Service Name incorreto.

### Configuração esperada no projeto

```text
Host: 127.0.0.1
Porta: 1521
Service Name: FREEPDB1
```

### No DBeaver

Selecione a conexão por:

```text
Service Name
```

e não por SID, caso a imagem esteja configurada dessa forma.

---

## 50. Listener do Oracle não está disponível

### Sintoma

```text
ORA-12541: TNS:no listener
```

### Causas prováveis

* Oracle ainda inicializando;
* container parado;
* porta incorreta;
* listener falhou;
* pouca memória.

### Diagnóstico

```bash
docker compose ps
docker logs <container_oracle>
docker port <container_oracle>
```

Dentro do container, quando disponível:

```bash
lsnrctl status
```

---

# Problemas específicos de bancos NoSQL

## 51. MongoDB rejeita autenticação

### Sintoma

```text
Authentication failed
```

### Causas prováveis

* usuário criado no banco `admin`;
* `authSource` ausente;
* senha incorreta;
* volume inicializado com credenciais antigas.

### Exemplo de conexão

```text
mongodb://admin:admin123@127.0.0.1:27017/?authSource=admin
```

### Teste interno

```bash
docker exec -it mongo_db \
  mongosh \
  -u admin \
  -p admin123 \
  --authenticationDatabase admin
```

---

## 52. Redis responde sem autenticação ou exige senha inesperada

### Diagnóstico

```bash
docker exec -it redis_db redis-cli
```

Teste:

```text
PING
```

Resposta esperada:

```text
PONG
```

Se houver senha configurada:

```bash
docker exec -it redis_db \
  redis-cli \
  -a '<senha>'
```

### Observação

A variável no `.env` só terá efeito se o comando do serviço Redis utilizar essa variável para iniciar o servidor com autenticação.

---

## 53. Cassandra ainda não aceita conexão

### Sintomas

```text
Connection refused
```

```text
NoHostAvailable
```

### Causa provável

O Cassandra ainda está formando e inicializando o cluster local.

### Diagnóstico

```bash
docker logs -f cassandra_db
```

Verifique recursos:

```bash
docker stats cassandra_db
```

### Soluções

* aguarde a conclusão da inicialização;
* aumente memória disponível;
* não inicie muitos bancos pesados simultaneamente;
* configure healthcheck e `start_period` adequados.

---

# Problemas no DBeaver

## 54. Driver não foi baixado

### Sintoma

O DBeaver informa que faltam arquivos do driver.

### Solução

Na criação da conexão:

1. Selecione o banco correto.
2. Clique em **Download** quando solicitado.
3. Aguarde o download.
4. Execute **Test Connection**.

### Em rede corporativa

Verifique proxy e firewall nas configurações do DBeaver.

---

## 55. DBeaver conecta no banco errado

### Sintomas

* tabelas inesperadas;
* senha aparentemente não corresponde;
* banco está vazio;
* versão diferente da esperada.

### Causas prováveis

* porta de outro banco;
* conexão antiga salva;
* container diferente usando a porta;
* host remoto configurado.

### Diagnóstico

Confira:

```text
Host
Port
Database
User
```

Compare com:

```bash
docker compose ps
docker port <container>
```

---

## 56. DBeaver não salva ou usa a senha antiga

### Solução

1. Abra **Edit Connection**.
2. Digite novamente a senha.
3. Marque ou desmarque **Save password locally**, conforme desejado.
4. Clique em **Invalidate/Reconnect**.
5. Teste novamente.

Se necessário, exclua a conexão e crie outra com os valores atuais do `.env`.

---

## 57. Erro de timezone ou horário incorreto

### Sintomas

* registros com horas diferentes;
* logs em UTC;
* timestamps deslocados;
* aplicação e banco com fusos diferentes.

### Verificação

No `.env`:

```ini
TZ=America/Sao_Paulo
```

Confirme no container:

```bash
docker exec -it <container> date
```

### Solução

Corrija `TZ` e recrie o container:

```bash
docker compose up -d --force-recreate
```

Também revise o timezone:

* do sistema operacional;
* da aplicação;
* do driver;
* da sessão do banco.

---

# Procedimento de recuperação

## Nível 1 — Reiniciar o serviço

Use quando o container está travado, mas a configuração não mudou:

```bash
docker compose restart <serviço>
```

Exemplo:

```bash
docker compose restart mysql
```

---

## Nível 2 — Parar e iniciar o container

```bash
docker stop mysql84
docker start mysql84
```

---

## Nível 3 — Recriar somente o container

Use quando o Compose ou `.env` mudou:

```bash
docker compose \
  --profile mysql \
  up -d \
  --force-recreate
```

O volume nomeado deve ser preservado.

---

## Nível 4 — Remover somente o container

```bash
docker stop mysql84
docker rm mysql84
docker compose --profile mysql up -d
```

Isso não deve remover o volume nomeado.

Confirme antes:

```bash
docker volume ls
```

---

## Nível 5 — Restaurar backup em volume novo

Use quando:

* o volume está corrompido;
* houve mudança de versão principal;
* o banco não inicia;
* precisa validar uma recuperação segura.

Fluxo recomendado:

```text
Preservar volume antigo
        ↓
Criar volume novo
        ↓
Subir banco limpo
        ↓
Restaurar backup
        ↓
Validar tabelas e registros
        ↓
Atualizar aplicações
        ↓
Remover volume antigo somente depois
```

---

## Nível 6 — Reinicialização destrutiva

Somente para ambiente descartável:

```bash
docker compose down -v
docker compose --profile mysql up -d
```

> [!CAUTION]
> Esse processo apaga os dados persistidos do projeto.

---

# Coleta de informações para suporte

Quando não conseguir resolver um problema, reúna as informações abaixo.

## Versões

```bash
docker version
docker compose version
```

## Configuração resolvida

```bash
docker compose config
```

> [!WARNING]
> Revise a saída antes de compartilhá-la. Ela pode conter senhas e outras informações sensíveis.

## Status

```bash
docker compose ps
docker ps -a
```

## Logs

```bash
docker compose logs --tail 300
```

## Inspeção

```bash
docker inspect <container>
```

## Volumes

```bash
docker volume ls
```

## Redes

```bash
docker network ls
docker network inspect <rede>
```

## Recursos

```bash
docker stats --no-stream
docker system df
```

## Informações adicionais

Registre:

* sistema operacional;
* versão do Docker;
* banco utilizado;
* imagem e tag;
* comando executado;
* erro completo;
* alteração realizada antes do problema;
* conteúdo relevante do Compose;
* nomes das variáveis, sem expor senhas;
* comportamento esperado;
* comportamento observado.

---

# Modelo para abertura de issue

````md
## Descrição

Descreva claramente o problema.

## Ambiente

- Sistema operacional:
- Docker:
- Docker Compose:
- Banco:
- Imagem:
- Profile:

## Comando executado

```bash
docker compose --profile mysql up -d
````

## Resultado esperado

Descreva o que deveria acontecer.

## Resultado observado

Descreva o que aconteceu.

## Logs

```text
Cole somente os trechos relevantes.
Remova senhas e informações sensíveis.
```

## Tentativas realizadas

* [ ] Validei com `docker compose config`
* [ ] Consultei `docker compose ps`
* [ ] Consultei os logs
* [ ] Verifiquei a porta
* [ ] Testei as credenciais
* [ ] Confirmei o volume

````

---

# Tabela de risco dos comandos

| Comando | Risco | Efeito sobre os dados |
|---|---:|---|
| `docker ps` | Baixo | Nenhum |
| `docker compose ps` | Baixo | Nenhum |
| `docker logs` | Baixo | Nenhum |
| `docker inspect` | Baixo | Nenhum |
| `docker compose config` | Baixo | Nenhum |
| `docker restart` | Baixo | Reinicia o processo |
| `docker stop` | Baixo | Para o container |
| `docker start` | Baixo | Inicia o container |
| `docker compose down` | Médio | Remove containers e rede |
| `docker rm` | Médio | Remove o container |
| `docker compose pull` | Médio | Baixa imagens atualizadas |
| `docker rmi` | Médio | Remove uma imagem local |
| `docker volume rm` | Alto | Apaga dados do volume |
| `docker volume prune` | Alto | Apaga volumes não utilizados |
| `docker compose down -v` | Muito alto | Remove os volumes do projeto |
| `docker system prune --volumes` | Crítico | Pode apagar diversos recursos e dados |

---

# Checklist final

Antes de afirmar que o banco está com defeito, confirme:

- [ ] O Docker está em execução.
- [ ] `docker info` funciona.
- [ ] `docker compose version` funciona.
- [ ] Estou na pasta correta.
- [ ] O arquivo Compose está presente.
- [ ] `docker compose config` não apresenta erro.
- [ ] O arquivo `.env` existe.
- [ ] As variáveis possuem os nomes corretos.
- [ ] O profile correto foi utilizado.
- [ ] O container aparece em `docker ps -a`.
- [ ] Consultei os logs.
- [ ] O container está `healthy`.
- [ ] A porta externa está correta.
- [ ] A porta não está ocupada.
- [ ] Estou usando `127.0.0.1` na conexão local.
- [ ] Dentro de outro container, estou usando o nome do serviço.
- [ ] O usuário existe.
- [ ] O usuário possui permissão.
- [ ] O banco existe.
- [ ] A senha corresponde à inicialização do volume.
- [ ] O DBeaver está usando o driver correto.
- [ ] Revisei SSL e `allowPublicKeyRetrieval`, quando aplicável.
- [ ] O volume correto está montado.
- [ ] Há espaço em disco.
- [ ] Há memória suficiente.
- [ ] Fiz backup antes de qualquer procedimento destrutivo.

---

# Resumo do fluxo mais seguro

```bash
# 1. Validar a configuração
docker compose config

# 2. Verificar os containers
docker compose ps
docker ps -a

# 3. Consultar os logs
docker compose logs --tail 200

# 4. Verificar o container específico
docker inspect <container>

# 5. Confirmar porta e volumes
docker port <container>
docker volume ls

# 6. Recriar somente o container, se necessário
docker compose --profile <profile> up -d --force-recreate
````

A remoção de volumes deve ser sempre a última alternativa.

---

# Documentos relacionados

* [`installation.md`](./docs/installation.md) — instalação do ambiente;
* [`environment.md`](./docs/environment.md) — variáveis do `.env`;
* [`docker-compose.md`](./docs/docker-compose.md) — estrutura do Compose;
* [`commands.md`](./docs/commands.md) — referência de comandos;
* [`dbeaver.md`](./docs/dbeaver.md) — instalação e configuração do DBeaver;
* [`sql.md`](./docs/sql.md) — bancos relacionais;
* [`nosql.md`](./docs/nosql.md) — bancos não relacionais;
* [`faq.md`](./docs/faq.md) — perguntas frequentes.
