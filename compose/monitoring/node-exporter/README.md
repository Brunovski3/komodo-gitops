# node-exporter

Métrica de host (CPU, memória, disco, rede) para o Prometheus do OBS. Um container por host,
em `network_mode: host`, escutando em `${NODE_BIND_IP}:9100` com TLS e basic auth.

## Responsabilidades

| Item | Responsável | Local |
|---|---|---|
| `certs/node.crt` e `node.key` | Ansible | `/data/services/node-exporter/certs/` |
| `web-config.yml` | Ansible | `/data/services/node-exporter/` |
| Firewall `9100/tcp` para o OBS | Ansible | host |
| `compose.yaml` | Komodo | este repositório |

Certificado e hash de senha não ficam no Git. São gerados pelo playbook `node-exporter.yaml`
do `infra-ansible`.

## Aplicação

1. Rodar o playbook no OBS:

```bash
ansible-playbook node-exporter.yaml --limit <host>
```

2. Criar a stack no Komodo com `run_directory` `node-exporter/` e `NODE_BIND_IP` no ambiente:

| Stack | Servidor | NODE_BIND_IP |
|---|---|---|
| `node-exporter-core` | CORE | `<IP_CORE>` |
| `node-exporter-obs` | OBS | `<IP_OBS>` |
| `node-exporter-glpi-new` | GLPI-NEW | `<IP_HOST_3>` |
| `node-exporter-apps` | APPS | `<IP_HOST_4>` |
| `node-exporter-lbexterno` | LBEXTERNO | `<IP_LB_EXT>` |

3. Validar:

```bash
ansible-playbook check-node-exporter.yaml --limit <host>
```

O playbook precisa rodar antes do deploy. Caso contrário o Docker cria `web-config.yml` como
diretório e o container não sobe. Nesse caso, remover o diretório, rodar o playbook e fazer
o deploy de novo.

## Coleta

O Prometheus coleta em `https` com `insecure_skip_verify` e basic auth. A senha fica em
`/data/services/prometheus/node_exporter_password`. Ver `prometheus/prometheus.yml`.
