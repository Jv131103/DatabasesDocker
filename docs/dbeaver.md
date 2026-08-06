# Guia de Instalação do DBeaver

[DBeaver](https://dbeaver.io/) é um cliente gráfico para gerenciar bancos de dados (PostgreSQL, MySQL/MariaDB, SQLite, Oracle, etc.).

--- 

## Instalação

### Windows
1. Acesse a página de [downloads](https://dbeaver.io/download/).
2. Baixe o instalador `.exe` (Community Edition).
3. Execute e siga o assistente.

### macOS
1. Baixe o `.dmg` no [site oficial](https://dbeaver.io/download/).
2. Arraste o aplicativo para a pasta **Applications**.

### Linux (Debian/Ubuntu)
Baixe o `.deb` no site e, no terminal:

```bash
sudo dpkg -i dbeaver-ce_latest_amd64.deb
# Se aparecerem dependências pendentes:
sudo apt -f install
```

### Linux (RHEL/Fedora)
Baixe o `.rpm` e instale:

```bash
sudo rpm -ivh dbeaver-ce-latest-stable.x86_64.rpm
```

> Dica: você também pode instalar via **Flatpak** ou **Snap**, conforme sua distro.

---

## Conexão com os bancos do projeto

- **Host**: `127.0.0.1`  
- **Porta**: conforme definido no arquivo `.env`  
- **Usuário/Senha**: conforme o `.env`  
- **Banco**: `appdb` (ou o que você configurou)

### Exemplo (MariaDB/MySQL)
1. **New Connection** → escolha **MariaDB** ou **MySQL**.  
2. **Host**: `127.0.0.1`; **Port**: `3308` (MariaDB) ou `3307` (MySQL); **Database**: `appdb`.  
3. **User**: `app`; **Password**: `app123` (ajuste conforme `.env`).  
4. Clique em **Test Connection** e salve.

### Exemplo (PostgreSQL)
1. **New Connection** → **PostgreSQL**.  
2. **Host**: `127.0.0.1`; **Port**: `5433`; **Database**: `appdb`.  
3. **User**: `app`; **Password**: `admin123`.  
4. Teste e salve.

---

## Dicas rápidas

- Prefira `127.0.0.1` em vez de `localhost` para evitar issues de resolução.
- As portas e credenciais vêm do arquivo `.env` do projeto.
- Se o banco ainda está **subindo**, aguarde o **healthcheck** ficar OK; veja logs:
  ```bash
  docker compose logs -f
  ```
- Erro de autenticação (MySQL/MariaDB)? Dentro do container execute:
  ```sql
  CREATE USER 'app'@'%' IDENTIFIED BY 'app123';
  GRANT ALL PRIVILEGES ON appdb.* TO 'app'@'%';
  FLUSH PRIVILEGES;
  ```
- Usuário extra no PostgreSQL:
  ```sql
  CREATE USER dev WITH PASSWORD 'dev123';
  CREATE DATABASE devdb OWNER dev;
  ```

---

## Referências

- Site oficial: <https://dbeaver.io/>
- Documentação: <https://dbeaver.io/docs/>