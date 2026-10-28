# Referência OpenClaw

Leia só quando a pergunta passar do fluxo (protocolo, pareamento, fila, comando). Fonte viva: https://docs.openclaw.ai/concepts/architecture e `openclaw docs <consulta>`.

## Protocolo

Transporte: WebSocket, frame de texto JSON. O primeiro frame é `connect`. Qualquer outra coisa fecha a conexão.

Depois do aperto de mão:

- Pedido: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Evento: `{type:"event", event, payload, seq?, stateVersion?}`

`hello-ok` traz snapshot (`presence`, `health`, `stateVersion`, `uptimeMs`) e limites (`maxPayload`, `maxBufferedBytes`, `tickIntervalMs`). A lista `features.methods` é descoberta conservadora, não o catálogo inteiro de rotas.

Um turno de agente é em dois tempos: ack imediato `{runId, status:"accepted"}`, eventos `agent` enquanto gera, resposta final `{runId, status, summary}`.

Métodos com efeito (`send`, `agent`) exigem chave de idempotência. O servidor deduplica por um tempo curto.

Eventos não são regravados. Buraco na sequência: o cliente pede o snapshot de novo.

HTTP no mesmo porto: `/v1/models`, `/v1/embeddings`, `/v1/chat/completions`, `/v1/responses`, `/tools/invoke`, rotas de plugin e a Control UI. Widgets em `/__openclaw__/canvas/`.

## Auth e pareamento

- Segredo compartilhado: `gateway.auth.token` ou `gateway.auth.password` (env `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`). No fio, o campo pode ser `connect.params.auth.token` ou `password`; o modo da config escolhe qual segredo vale.
- Bind padrão: `loopback`. Em container detectado o efetivo vira `auto` (`0.0.0.0`), salvo Tailscale serve/funnel, que força loopback.
- `gateway.auth.mode: "none"` só em entrada privada. Fora disso, não.
- Todo cliente manda identidade de dispositivo e assina o nonce `connect.challenge`.
- Loopback local pode auto-aprovar. Tailnet e LAN, mesmo na mesma máquina, pedem aprovação explícita. O Gateway devolve um device token para os connects seguintes.
- Node inclui `role: "node"` mais caps, commands e permissions.

Acesso remoto preferido: Tailscale ou VPN. Alternativa: `ssh -N -L 18789:127.0.0.1:18789 user@host`. O mesmo handshake e o mesmo token valem dentro do túnel.

## Invariantes

- Um Gateway por host controla uma sessão Baileys.
- O aperto de mão é obrigatório.
- O scheduler do Gateway é um só. Cron e manutenção registram nele; trabalho durável reconstrói o prazo no boot, em vez de gravar um segundo scheduler.

## Comandos úteis

| Comando | Para quê |
|---|---|
| `openclaw status` | Gateway, canal, modelo, sessões recentes |
| `openclaw status --deep` | Sonda a mais |
| `openclaw health` | Saúde, com o Gateway aceitando conexão |
| `openclaw logs --follow` | Log ao vivo |
| `openclaw doctor` | Config, plugin, canal |
| `openclaw gateway` | Sobe em primeiro plano |
| `openclaw dashboard` | Abre a Control UI com o token atual |
| `openclaw docs <consulta>` | Busca a doc viva |
| `openclaw skills` | Lista skills carregadas |
| `openclaw sessions` | Sessões guardadas |
| `openclaw channels` | Canais |
| `openclaw agent` | Um turno via Gateway |
| `openclaw message` | Enviar ou ler mensagem de canal |
| `openclaw devices` | Pareamento |
| `openclaw nodes` | Nodes |

Perfil isolado: `openclaw --profile <nome>` usa `~/.openclaw-<nome>`. `--dev` usa `~/.openclaw-dev` e porto `19001`.

Não imprima a saída de `openclaw dashboard` se ela embutir o token na URL. Diga que o comando abre o painel.
