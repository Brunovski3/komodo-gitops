# loki + alloy

Log centralizado dos hosts do parque interno.

- **Loki** roda no OBS (`<IP_OBS>`), em modo monolítico
- **Alloy** roda em cada host e envia para `<IP_OBS>:3100`

O Loki não tem autenticação. O acesso à `3100` é restrito por firewall aos IPs dos hosts.

## Configuração

| Item | Valor |
|---|---|
| Retenção | 30 dias, via compactor |
| Memória | `mem_limit: 1g`, sem swap |
| Labels | `host`, `job`, `container`, `stream`, `level`, `unit` |

Valores por requisição (`request_id`, IP, usuário) não viram label. Ficam no texto e são
filtrados na consulta:

```logql
{host="glpi-new", container="fpahub_api"} |= "request_id=abc123"
```

## Aplicação

1. Rodar o playbook no OBS:

```bash
ansible-playbook loki-alloy.yaml
```

2. Stack `loki` no Komodo, servidor OBS, com `OBS_BIND_IP=<IP_OBS>`
3. Datasource no Grafana: `http://loki:3100`
4. Stack `alloy`, uma por servidor:

| Stack | Servidor | `HOST_LABEL` |
|---|---|---|
| `alloy-obs` | OBS | `obs` |
| `alloy-core` | CORE | `core` |
| `alloy-glpi-new` | GLPI-NEW | `glpi-new` |
| `alloy-apps` | APPS | `apps` |
| `alloy-lbexterno` | LBEXTERNO | `lbexterno` |

`LOKI_URL` é `http://<IP_OBS>:3100/loki/api/v1/push` em todas.

O playbook precisa rodar antes do deploy. O `config.alloy` vem do Ansible, não do clone.
Depois de alterar o `config.alloy`, reiniciar o container do Alloy.

## Validação

```bash
curl -s http://<IP_OBS>:3100/ready
curl -s http://<IP_OBS>:3100/loki/api/v1/label/host/values
curl -s http://<IP_OBS>:3100/loki/api/v1/rules
```

## Alertas

Regras em `rules/alertas-log.yaml`, enviadas ao mesmo Alertmanager das métricas.

| Alerta | Condição |
|---|---|
| `ContainerOOMKilled` | log de OOM em container |
| `PostgresFatal` | mais de 5 linhas `FATAL` em 5 min |
| `TraefikErro5xx` | mais de 20 respostas 5xx em 5 min |
| `SSHFalhaAutenticacao` | mais de 20 falhas de senha em 5 min |

Com `auth_enabled: false` o tenant é `fake`, por isso as regras são montadas em
`/loki/rules/fake/`.
