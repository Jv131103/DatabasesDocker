
---

## 📄 Extra: `INSTALLDBEAVER.md`

```markdown
# Guia de Instalação do DBeaver

[DBeaver](https://dbeaver.io/) é um cliente gráfico para gerenciar bancos de dados.

---

## Instalação

### Windows
1. Acesse [downloads](https://dbeaver.io/download/).
2. Baixe o instalador `.exe`.
3. Execute e siga o assistente.

### macOS
1. Baixe o `.dmg` no [site oficial](https://dbeaver.io/download/).
2. Arraste o app para a pasta **Applications**.

### Linux
1. Baixe o `.deb` ou `.rpm` conforme sua distro.
2. Instale via terminal:
```bash
sudo dpkg -i dbeaver-ce_latest_amd64.deb
```

ou

```bash
sudo rpm -ivh dbeaver-ce-latest-stable.x86_64.rpm
```

# Conexão com os bancos do projeto

. Host: 127.0.0.1

. Porta: definida no .env

. Usuário/Senha: conforme configuração do .env
