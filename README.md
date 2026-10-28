# Agente professor de OpenClaw

O OpenClaw é a portaria. Um processo fica ligado, recebe WhatsApp, Telegram e os outros chats, acha a conversa e chama o agente. A resposta volta pelo mesmo canal. Esta pasta ensina um agente a explicar isso para uma pessoa, sem misturar com o QClaw nem com o MemClaw.

## Instalar

Copie `explicar-openclaw` para as skills do OpenClaw:

```
~/.openclaw/skills/explicar-openclaw
```

O nome que o agente vê está no frontmatter: `explicar-openclaw`. Ele entra quando alguém pergunta como o OpenClaw funciona, o que é o Gateway, a porta 18789, ou a diferença para o QClaw e o MemClaw.

## O que tem na pasta

`SKILL.md` é o roteiro: a portaria, o agente, o caminho de uma mensagem. `referencia.md` guarda protocolo, pareamento e comandos. O agente só abre a referência se a pergunta passar do fluxo.

A doc viva continua em https://docs.openclaw.ai. Se o Gateway está no ar, quem responde é o `openclaw status`, não este texto.
