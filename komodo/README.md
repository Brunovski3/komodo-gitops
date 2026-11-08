# komodo/

Recursos declarados em TOML, no formato de Resource Sync do Komodo. A intenção é o GitOps declarativo: o Komodo (Core + Periphery) polla este repo e faz o diff entre o declarado em Git e o rodando no host — nenhuma mudança manual em `compose.yaml` sobrevive ao próximo sync.

## Arquivos

| Arquivo | O que declara |
| --- | --- |
| `resources/lab.toml` | Arquivo único: o servidor (via Periphery) + 9 stacks (n8n, komodo e as 7 de observabilidade). **Managed Mode exige arquivo único** — é nele que o Komodo escreve de volta |
| `resource-sync.toml` | O sync em si, com **`managed = true`** (bidirecional) |

## Primeiro import

1. Ajuste os `TODO` em `resources/lab.toml`: nome do servidor como registrado no Komodo e endereço do Periphery (`http://<IP_DA_VPS>:8120`).
2. Confira se os `server =` das stacks batem com o nome do servidor no Komodo, e se `branch = "main"` reflete a branch de deploy.
3. Configure no Core um git account com **PAT de write** no repo (privado: sem write não há pull nem push do managed mode).
4. Importe o `resource-sync.toml` na UI (ou crie um ResourceSync apontando pra este repo).
5. O primeiro sync vai acusar diff entre o declarado e o existente. Para stacks que já rodam fora do Komodo, o caminho é **New** (criar o recurso sem deploy) — não aplicar deploy em lote.
6. Com os recursos casando, troque `deploy = false` para `true` stack por stack, conforme entra em produção.

## Commits gerenciados

Com o Managed Mode ativo no sync, mudança feita na UI do Komodo volta como commit no Git — a sincronização bidirecional. O Git permanece a fonte da verdade: mudanças fora do fluxo (não vindas de commit nem da UI) aparecem como divergência no diff do sync.

## Referências

- [Sync Resources — guia oficial](https://github.com/moghtech/komodo/blob/main/docsite/docs/automate/sync-resources.md)
- Formatos dos [Server](https://docs.rs/komodo_client/latest/komodo_client/entities/server/struct.ServerConfig.html), [Stack](https://docs.rs/komodo_client/latest/komodo_client/entities/stack/struct.StackConfig.html) e [ResourceSync](https://docs.rs/komodo_client/latest/komodo_client/entities/sync/type.ResourceSync.html)
