# komodo/

Recursos declarados em TOML, no formato de Resource Sync do Komodo. A intenção é a governança que o CV descreve: o Komodo (Core + Periphery) polla este repo e faz o diff entre o declarado em Git e o rodando no host — nenhuma mudança manual em `compose.yaml` sobrevive a uma semana.

## Arquivos

| Arquivo | O que declara |
| --- | --- |
| `resources/server.toml` | O host de produção via Periphery |
| `resources/stacks.toml` | As 4 stacks: n8n, GLPI, Grafana e o próprio Komodo |
| `resource-sync.toml` | O sync que carrega os dois arquivos acima |

## Primeiro import

1. Ajuste os `TODO` em `resources/server.toml` (nome e endereço do Periphery).
2. Confira se os `server =` das stacks batem com o nome do seu servidor no Komodo.
3. Importe `resource-sync.toml` na UI (ou aponte um ResourceSync novo pra este repo).
4. O primeiro sync vai acusar diff entre "declarado" e "existe na UI". Para stacks que já rodam fora do Komodo, o caminho é **New** (criar o recurso sem deploy) — não aplicar deploy de enfião.
5. Com os recursos casando, troque `deploy = false` pra `true` no `n8n` e vá liberando as outras conforme entra em produção.

## Commits gerenciados

Com o Managed Mode ativo no sync, mudança feita na UI do Komodo volta como commit no Git — a "sincronização bidirecional".

## Referências

- [Sync Resources — guia oficial](https://github.com/moghtech/komodo/blob/main/docsite/docs/automate/sync-resources.md)
- Formatos dos [Server](https://docs.rs/komodo_client/latest/komodo_client/entities/server/struct.ServerConfig.html), [Stack](https://docs.rs/komodo_client/latest/komodo_client/entities/stack/struct.StackConfig.html) e [ResourceSync](https://docs.rs/komodo_client/latest/komodo_client/entities/sync/type.ResourceSync.html)
