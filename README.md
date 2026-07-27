# Copy Logic Peer Review

A Claude Code skill that simulates the AWAI-style "Peer Review" meeting for a sales-letter or email lead — a structured group session where a Peer Group rates the headline and lead numerically (1.0–4.0), proposes concrete rewrite suggestions, and rates each other's suggestions — using parallel `Agent` tool calls as stand-ins for a live human panel.

## What it does

- Runs the headline and lead through separate rate → improve → re-rate rounds.
- Dispatches a Peer Group of 5 simulated copywriters (fixed personas + model assignments) in parallel via the `Agent` tool.
- Verifies the headline's promise is actually kept by the body content before evaluating anything.
- Applies strict editing rules: unanimous positive ratings force a suggestion in, unanimous negative blocks it, mixed ratings get flagged back to the user.

## Install

Via [skills.sh](https://www.skills.sh) (recommended):

```
npx skills add luandatt/copy-logic-peer-review
```

This picks up `SKILL.md` from the repo root and installs it for Claude Code automatically. Pass `-a claude-code` if you want to target Claude Code specifically (omit it to install for all supported agents), and run `npx skills list` afterward to confirm it's installed.

Manual install:

Copy `SKILL.md` into your Claude Code skills directory as `~/.claude/skills/copy-peer-review/SKILL.md`.

## Usage

Trigger it by asking to "run a peer review" / "test the headline with a peer group" on a sales letter, ad, or newsletter/email lead.

---

# Copy Logic Peer Review (Português)

Uma skill do Claude Code que simula a reunião de "Peer Review" no estilo AWAI para o headline ou lead de uma carta de vendas ou e-mail — uma sessão em grupo estruturada onde um Peer Group avalia o headline e o lead numericamente (1.0–4.0), propõe sugestões concretas de reescrita e avalia as sugestões uns dos outros — usando chamadas paralelas da tool `Agent` como substitutas de um painel humano ao vivo.

## O que ela faz

- Roda o headline e o lead em rodadas separadas de avaliar → melhorar → reavaliar.
- Dispara um Peer Group de 5 copywriters simulados (personas fixas + modelos atribuídos) em paralelo via a tool `Agent`.
- Verifica se a promessa do headline é realmente cumprida pelo conteúdo do corpo do texto antes de avaliar qualquer coisa.
- Aplica regras estritas de edição: notas unânimes positivas forçam a aplicação de uma sugestão, unânimes negativas bloqueiam, notas mistas voltam pro usuário decidir.

## Instalação

Via [skills.sh](https://www.skills.sh) (recomendado):

```
npx skills add luandatt/copy-logic-peer-review
```

Isso pega o `SKILL.md` da raiz do repositório e instala automaticamente para o Claude Code. Passe `-a claude-code` se quiser direcionar especificamente pro Claude Code (omita pra instalar em todos os agentes suportados), e rode `npx skills list` depois pra confirmar a instalação.

Instalação manual:

Copie o `SKILL.md` para o diretório de skills do seu Claude Code, como `~/.claude/skills/copy-peer-review/SKILL.md`.

## Uso

Acione a skill pedindo pra "rodar um peer review" / "testar a headline com um grupo de pares" numa carta de vendas, anúncio, ou lead de newsletter/e-mail.
