---
name: explicar-openclaw
description: >-
  Explica o OpenClaw (gateway self-hosted, agente embutido, canais, sessões,
  skills e WebSocket na porta 18789) e o mapa desta máquina (~/.openclaw,
  agente main, workspace no qclawmonitor). Use quando a pessoa perguntar como
  o OpenClaw funciona, o que é o Gateway, openclaw status, canais, sessões,
  skills do OpenClaw, ou a diferença entre OpenClaw, QClaw e MemClaw.
---

# Explicar o OpenClaw

Ensine o produto. Confira o estado ao vivo antes de afirmar que o Gateway está no ar. Não cole segredo.

## Antes de responder

1. Se a pergunta for o conceito, use os fatos estáveis daqui. Se for "como está agora" ou o mapa desta máquina, rode `openclaw --version` e `openclaw status`.
2. Não abra credencial. Proibido colar token, senha, `env` ou `headers` de `~/.openclaw/openclaw.json` e de `~/.qclaw-oversea/`.
3. Doc que possa ter mudado: `openclaw docs <consulta>` ou https://docs.openclaw.ai/concepts/architecture . Protocolo e comandos: [referencia.md](referencia.md).
4. A resposta para a pessoa sai em português do Brasil, com arco de texto humano: analogia, uma ideia por bloco, fecho com o que ela leva. Sem lista de jargão no primeiro parágrafo.

## O que é

OpenClaw é um gateway que a pessoa hospeda. Um processo fica ligado e é a única ponte entre apps de chat e um agente que usa ferramentas. Sessão, roteamento e credencial de canal ficam na máquina. Licença MIT. Código: https://github.com/openclaw/openclaw . Docs: https://docs.openclaw.ai .

Analogia para a abertura: portaria de prédio. WhatsApp, Telegram, Slack, Discord, Signal, iMessage e o WebChat são entradas. A portaria (Gateway) anota quem chegou, acha a conversa e chama o morador (agente). O morador pensa, mexe em arquivo e ferramenta, e a resposta volta pela mesma entrada.

Isso ainda não é um framework de orquestração. O loop do agente é do próprio OpenClaw.

## As três peças

### Gateway

Um daemon por máquina. É o único lugar que abre a sessão de WhatsApp (Baileys) daquele host. Expõe WebSocket e HTTP no mesmo porto. Padrão: `127.0.0.1:18789`. Painel: http://127.0.0.1:18789/ .

Clientes (CLI, Control UI, app macOS) falam WebSocket. O primeiro frame tem de ser `connect`. Nodes (macOS, iOS, Android, headless) entram no mesmo soquete com `role: node` e oferecem câmera, tela e localização, depois de pareamento.

### Agente

Runtime embutido, um por agente configurado. Cada agente tem workspace, arquivos de bootstrap e sessões. No primeiro turno de uma sessão nova, o prompt recebe o que existir no workspace:

| Arquivo | Função |
|---|---|
| `AGENTS.md` | Instruções de operação |
| `SOUL.md` | Tom e limites |
| `IDENTITY.md` | Nome e jeito |
| `USER.md` | Quem é a pessoa |
| `MEMORY.md` | Memória longa, se o arquivo existir |
| `BOOTSTRAP.md` | Ritual da primeira vez; some depois |

Arquivo em branco é ignorado. Arquivo grande é cortado. A seção `## Tools` do `AGENTS.md` orienta o uso; não liga nem desliga ferramenta.

Sessão ativa fica no SQLite do agente (`~/.openclaw/agents/<id>/agent/openclaw-agent.sqlite`). JSONL em `sessions/` é legado ou arquivo de suporte.

### Skills

`SKILL.md`, nome plano no frontmatter. Ordem, da que ganha para a que perde:

1. `<workspace>/skills`
2. `<projeto>/.agents/skills`
3. `~/.agents/skills`
4. `~/.openclaw/skills`
5. as que vêm com a instalação
6. pastas em `skills.load.extraDirs`

## O caminho de uma mensagem

1. O plugin do canal recebe o texto.
2. O Gateway escolhe agente e sessão. A chave parece `agent:main:telegram:group:…` ou `agent:main:whatsapp:group:…`.
3. O runtime monta o prompt: bootstrap, skills que cabem, histórico.
4. Chama o modelo. Se o modelo pede ferramenta, executa e volta ao modelo.
5. A resposta sai pelo mesmo canal. Quem está no WebSocket vê eventos `agent` e `chat`.
6. Texto que chega no meio da resposta entra na corrida atual. O padrão é `/queue steer`. `/queue followup` e `/queue collect` esperam o próximo turno. `/queue interrupt` aborta.

`openclaw agent` dispara um turno pela CLI. `openclaw message` manda e lê mensagem de canal.

## Nesta máquina

Fatos estáveis. O que está no ar muda; confirme com `openclaw status`.

| Peça | Onde |
|---|---|
| Estado | `~/.openclaw` |
| Config | `~/.openclaw/openclaw.json` (`agents`, `skills`, `mcp`, `meta`) |
| Agente | `main` |
| Workspace | `/Users/naubergois/qclawmonitor/.stack/openclaw/state/workspace` |
| Skills extras | `~/qclawmonitor/skills` e `/Volumes/NAUBER/qclawmonitor/skills` |
| MCP no config | `qclaw-skills` (processo local) e `seedance` |
| Porto | `18789` em loopback, porque o config não redefine `gateway` |
| Painel | `openclaw dashboard` ou http://127.0.0.1:18789/ |

Esta config não tem bloco `channels`. Sessão antiga de WhatsApp ou Telegram pode continuar listada mesmo com "No channels configured".

Subir em primeiro plano: `openclaw gateway`. Serviço: `openclaw daemon` (alias do serviço do Gateway). Diagnóstico: `openclaw doctor`, `openclaw logs --follow`, `openclaw health` (só com o Gateway aceitando conexão).

Não enumere as skills `qclaw-*`. Se perguntarem o que está habilitado, mande olhar `openclaw skills` ou a lista `agents.list[].skills`, sem despejar centenas de nomes na prosa.

## O que não misturar

- **QClaw** é outro app. A config dele fica em geral em `~/.qclaw-oversea/openclaw.json`, não em `~/.openclaw/openclaw.json`. Skill com prefixo `qclaw-` é pacote carregado no agente; não é o binário `openclaw`.
- **MemClaw** guarda memória (Mongo e índice). Não é a portaria.
- **Cursor** é outro cliente de agente. Esta skill ensina a explicar o OpenClaw. Não transforma o Cursor no Gateway.
- **Hermes** é outro plano de controle da frota. Não descreva integração que `openclaw status` não mostre.

## Como contar

1. Abertura: a analogia da portaria e por que a pessoa usa (vários chats, uma máquina, um agente).
2. Bloco da portaria: um processo, um porto, clientes e nodes.
3. Bloco do morador: workspace, sessão, skills.
4. Bloco do caminho: os seis passos de uma mensagem.
5. Se for esta máquina, um bloco com a tabela e o que `openclaw status` acabou de dizer (Gateway alcançável ou não, canais, agente).
6. Fecho: um processo, uma porta, um agente. O comando para ver se a portaria está aberta é `openclaw status`. Limite honesto: isso ainda não é o QClaw nem o MemClaw.

Profundidade:

- "como funciona" → analogia, três peças, caminho, mapa local.
- protocolo, pareamento, fila → [referencia.md](referencia.md).
- "não responde" → `openclaw status`, canal configurado, `openclaw doctor`. Não invente o conserto.
