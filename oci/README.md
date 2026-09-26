# Cluster OCI — K3s HA + Zero-Trust (diretório `oci/`)

Cluster Kubernetes multi-node (1 control-plane + 2 workers) rodando K3s na Oracle Cloud. Acesso sem porta aberta: a tailnet do Tailscale carrega a entrada, o Traefik termina o ingress e o TLS, os workloads entram via Helm (Keycloak é o IAM, n8n é a automação).

## Estrutura

| Caminho | O que vai ter |
|---|---|
| `cluster/` | `config.yaml` do k3s, installer e roles dos nó |
| `networking/traefik/` | Ingress e TLS |
| `networking/tailscale/` | Operator do Tailscale e o ProxyGroup que publica os serviços em HTTPS privado |
| `apps/` | Helm releases dos workloads (Keycloak, n8n...) |

## Como os manifests entram aqui

Os manifests vivos existem no servidor. O script `scripts/pull-from-server.sh` faz o rsync dos autodeploy manifests (`/var/lib/rancher/k3s/server/manifests/` — HelmCharts do Traefik, Tailscale Operator, n8n) e do `config.yaml`, e aponta qualquer segredo encontrado na varredura. Fluxo:

```bash
./scripts/pull-from-server.sh <host-oci> .
# revisão em local-export/ (fora do Git)
# transporte dos manifests limpos para os diretórios, com commit
```

Nunca copia token do cluster, chaves TLS nem credenciais. Segredo de produção segue via `secretKeyRef`/gerenciador de segredos, nunca no Git.

## Estado

Estrutura e coletor prontos. Os manifests entram em commits de `cluster/`, `networking/` e `apps/` conforme revisão do `local-export/`.
