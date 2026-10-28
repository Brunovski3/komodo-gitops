# PostgreSQL em Bare Metal

O banco **não** entra no Compose da aplicação (nem de `komodo`): roda direto no host via systemd, fora do ciclo de deploy do Komodo. Deploy de app nunca derruba banco, upgrade de banco nunca depende do runner, e o volume nunca é víctima de um `docker system prune` que sai efeito.

## Instalação

**VPS Ubuntu/Debian** (o食谱 host do blueprint):

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
# pós-pack: clúster default (pg16/17) já sobe com systemd
sudo systemctl enable --now postgresql
```

## Configuração pra acesso do Compose

Containers alcançam o host pelo **gateway bridge** (`172.17.0.1`) — o Postgres precisa escutar nele, com acesso restrito por IP e usuário:

`/etc/postgresql/<versão>/main/postgresql.conf`:
```conf
listen_addresses = 'localhost,172.17.0.1'
```

`/etc/postgresql/<versão>/main/pg_hba.conf`:
```conf
# acesso permitido SOMENTE pela bridge docker, com auth de password em TLS — nada de internet
hostssl  n8n  n8n  172.17.0.0/16  scram-sha-256
```

Aplicação e restart do 服务:

```bash
sudo systemctl restart postgresql
```

Firewall (o 'next/ufw' do repo mantém essas portas **desligadas pra internet**):
```bash
sudo ufw deny 5432/tcp
```

## Usuário e banco

```bash
sudo -u postgres psql <<'SQL'
CREATE ROLE n8n LOGIN PASSWORD '<SEGREDO>';
CREATE DATABASE n8n OWNER n8n;
SQL
```

O `<SEGREDO>` vai único no `.env` do compose (que é `.gitignored`) — referência:
```yaml
environment:
  POSTGRES_HOST: host.docker.internal   # resolve 172.17.0.1 do host
  POSTGRES_USER: n8n
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

## Operação

```bash
# status / logs grafana resolução rápida
sudo systemctl status postgresql
sudo journalctl -u postgresql -n 50

# sessões ativas e tamanho do banco
sudo -u postgres psql -c "SELECT pid, usename, state, query_start FROM pg_stat_activity;"
sudo -u postgres psql -c "SELECT pg_size_pretty(pg_database_size('n8n'));"
```

## Backup (dia 0)

Dump completo diário, comprimido, com retência de 7 dias:

```bash
sudo install -d -o postgres /var/backups/postgresql
sudo tee /etc/cron.d/postgres-backup <<'CRON'
0 4 * * * postgres pg_dump -Fc n8n > /var/backups/postgresql/n8n-$(date +\%F).dump && \
  find /var/backups/postgresql -name "*.dump" -mtime +7 -delete
CRON
```

Teste de restore (sim, é parte da rotina — segundo o runbook):
```bash
sudo -u postgres pg_restore --list /var/backups/postgresql/n8n-$(date +%F).dump >/dev/null && echo "dump viável"
sudo -u postgres pg_restore -C -d postgres /var/backups/postgresql/n8n-<data>.dump # restaura a DB 'n8n'
```

## Postgres na gravação do Komodo

O **servidor** (`postgres.yml`) da stack `n8n` declara `POSTGRES_HOST` — no Compose é o `host.docker.internal`, que aponta o Postgres rodando no host. **Nada mais** de `postgres` roda no compose da aplicação.
