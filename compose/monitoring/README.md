# Observabilidade (Docker Compose)

Stack de monitoramento do host e das aplicações: métricas (Prometheus), logs (Loki + Alloy),
dashboards (Grafana), alertas (Alertmanager → webhook Cliq) e probes (Blackbox + Node Exporter).

## Componentes

| Pasta | Serviço | Papel |
|---|---|---|
| `prometheus/` | Prometheus | Coleta métricas dos exporters e das APIs (Komodo, Traefik), recording rules e alertas em `alerts.yml` |
| `grafana/` | Grafana | Dashboards (`dashboards/*.json`), auth por LDAP/OAuth configurável em `.env` |
| `loki/` | Loki | Armazena logs recebidos do Alloy; regras de alerta por log em `rules/` (ex.: força bruta no sshd) |
| `alloy/` | Grafana Alloy | Coleta logs dos containers e do journald e envia pro Loki |
| `alertmanager/` | Alertmanager | Agrupa e roteia alertas; receiver `cliq` posta num webhook (bot de referência em `cliq-bot.deluge`) |
| `node-exporter/` | Node Exporter | Métricas de sistema de cada host (rede, CPU, disco) |
| `blackbox/` | Blackbox Exporter | Probes HTTP dos endpoints do Traefik (`blackbox.yml`) |

## Antes de subir (placeholders)

Os arquivos vieram de um ambiente de referência e foram higienizados. Substitua antes de aplicar:

- `<IP_CORE>`, `<IP_LB_EXT>`, `<IP_OBS>`, `<IP_HOST_3>`, `<IP_HOST_4>`, `<IP_DNS>` — IPs dos hosts nos targets do `prometheus/prometheus.yml`, `blackbox/compose.yaml` e nos READMEs de `loki/` e `node-exporter/`
- `.env` de cada stack (copie de `.env.example`): senhas do Grafana, LDAP/OAuth se usar, URL do webhook do Alertmanager
- `<SEU_CANAL_CLIQ>` em `alertmanager/cliq-bot.deluge` — canal de destino do bot
- Endpoints em `prometheus/prometheus.yml` (bloco de HTTP probes) — apontem para os serviços reais deste host
- Sufixo `.example.local` — troque pelos domínios reais do Traefik
- Paths de host em `/data/services/...` (certs do Blackbox, config e dados do Alloy, secrets do Alertmanager) — ajuste para os diretórios reais ou crie com `install -d`

## Ordem de subida

1. `node-exporter` e `blackbox` (exporters não dependem de nada)
2. `loki` e depois `alloy` (o Alloy precisa do destino de logs no ar)
3. `prometheus` (consome exporters)
4. `alertmanager` (recebe alertas do Prometheus)
5. `grafana` (dashboards apontam pros dois)

## Backup do Postgres

O alerta de backup (`BackupFailed`) depende do `pg_dump` reportando status via metrics —
provisionamento do banco em [`docs/postgres-bare-metal.md`](../../docs/postgres-bare-metal.md).
