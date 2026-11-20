# 🧠 Jarvis in Practice — swappable brains

[![Jarvis in Practice: swappable brains](guia/assets/banner-en.jpg)](https://inematds.github.io/jarvis-na-pratica/guia/en/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## What it is

This is the INTELECTO v2 Jarvis, a free personal assistant that runs for real with three swappable “brains”: Claude Code and Codex, which you use through subscriptions you already pay for, and OpenRouter, which charges fractions of a cent per use. It’s for anyone who wants an assistant on their own computer or server without being locked into one AI company. We set up all three to have the same conversation on the same server, then measured accuracy, time, and cost. To use it, you need a Linux computer or server and at least one of the three brains.

## 📖 User guide

Complete guide (the benefits, how it works, which brain to use, and measured evidence): **https://inematds.github.io/jarvis-na-pratica/guia/en/**

<!-- VIDEO: entra o explicavideo do projeto quando publicado -->

## Start by asking your AI

Open Claude Code or Codex on the machine where Jarvis will run and paste:

```text
Read the “Jarvis na prática” project at https://github.com/inematds/jarvis-na-pratica (the README explains everything, including installation).
Then check this machine and reply in simple Portuguese:
1. Which of the three brains can I already use here (Claude Code, Codex, or OpenRouter)?
2. What needs to be installed or enabled, and does the sandbox (Docker) work on this machine?
3. Which brain do you recommend for me, and why?
Do not install anything yet. Show me the plan and wait for me to say yes.
If I approve, first run the rehearsal without AI, then run the test conversation using only the brain I choose.
```

## Measured results (20/11/2026, Linux server with 15 GB, Docker sandbox enabled, one run per brain)

The same 6-step conversation (disk → remember → recall → delete with NÃO [no] → delete with SIM [yes] → `sudo reboot`):

| brain | model | correct | total time | cost charged |
|---|---|---|---|---|
| Claude Code (subscription) | haiku | 6/6 | 38.6 s | US$ 0 (API equivalent: US$ 0.0033) |
| Codex (subscription) | gpt-6.2-sol, low effort | 6/6 | 41.3 s | US$ 0 |
| OpenRouter (API) | deepseek/deepseek-v4.1-flash | 6/6 | 8.4 s | US$ 0.000891 |

Brain swap in the middle of a conversation (`jarvis.py trocar`): one brain stores “the contract deadline for client Marta is on the 12th,” and another answers after the swap. Claude → OpenRouter: “It’s due on the 12th.” OpenRouter → Codex: “Client Marta’s contract deadline is on the 12th.” (the memory belongs to Jarvis).

The 8 `intelecto avaliar` tasks: 8/8 for all three (5.9 s, 6.5 s, and 1.6 s per task; OpenRouter charged US$ 0.002185).
Raw records are in [`resultados/`](resultados/) (the measured machine: [`maquina-vps6.txt`](resultados/maquina-vps6.txt)).

---

## Technical install

### Prerequisites

- Linux with Python 3.10+ and `git`.
- Docker, for the sandbox (`python:3.12-slim` container with no network access). Without Docker, use `--executor local`.
- At least one brain:
  - **Claude Code**, logged into your subscription (`claude` on PATH). No `ANTHROPIC_API_KEY` is used: the kit removes these variables from the process.
  - **Codex**, logged into ChatGPT (`codex login`; `codex` on PATH, recent version). No `OPENAI_API_KEY` is used.
  - **OpenRouter**: `OPENROUTER_API_KEY` in the environment or in an `.env` file specified by `OPENROUTER_ENV` (default `~/.config/inema/.env`). Pay per use.

### Install

```bash
git clone https://github.com/inematds/jarvis-na-pratica && cd jarvis-na-pratica
./instalar.sh        # creates .venv, installs INTELECTO v2 (commit d428f31 = v2.3.2), and runs the tests
```

### Use

```bash
.venv/bin/python jarvis.py medir --cerebros mock --executor local   # rehearsal without AI or Docker (see below)
.venv/bin/python jarvis.py medir                                    # same conversation with all 3 brains → resultados/
.venv/bin/python jarvis.py medir --cerebros claude                  # just one
.venv/bin/python jarvis.py avaliar --cerebro codex                  # the 8 intelecto avaliar tasks
.venv/bin/python jarvis.py conversar --cerebro openrouter           # conversation in the terminal (approval prompt in the terminal)
.venv/bin/python jarvis.py trocar --de claude --para openrouter     # swap brains midway: can the new one remember?
```

**Rehearsal without AI:** the `mock` brain is a fake brain included with INTELECTO. It doesn’t think (it only understands requests that start with `rode:`), so half the steps fail, as expected. It’s only for checking that installation, memory, the safety lock, and measurement work before spending money on a real AI.

**For everyday use:** through Telegram (section below) or the terminal: `conversar` starts the conversation; when the brain requests an action that changes something, `rodar '…'? [s/N]` appears, and it runs only if you enter `s`.

**Tested on:** only one rented Linux server (Ubuntu 24.04, 4 processors, 15 GB; [`resultados/maquina-vps6.txt`](resultados/maquina-vps6.txt)). Claude Code and Codex run on Windows, Mac, and Linux according to their official pages ([Claude Code](https://code.claude.com/docs/en/setup), [Codex](https://developers.openai.com/codex/cli)), but the kit and Docker sandbox have only been tested on Linux.

Useful variables: `CODEX_MODELO` (default `gpt-6.2-sol`), `CODEX_ESFORCO` (`low`), `OPENROUTER_MODELO` (`deepseek/deepseek-v4.1-flash`), `CLAUDE_BIN`, `CODEX_BIN`.

### Using it on Telegram

1. **Create the bot:** in Telegram, `@BotFather` → `/newbot` → copy the token.
2. **Save the token and owners in the kit’s `.env`.** This file is outside git. Paste the token directly into the machine’s terminal (for example, `nano .env`), never into a conversation with an AI or a chat:
   ```bash
   cd jarvis-na-pratica
   umask 077 && printf 'TELEGRAM_BOT_TOKEN=%s\nDONOS=\n' "<token from BotFather>" > .env   # 600
   ```
3. **Find your ID:** start it once (`.venv/bin/python jarvis.py telegram`), send `/id` to the bot, stop it with Ctrl+C, and put the number in `DONOS=` (multiple owners: `DONOS=111,222`). With `DONOS` empty, the bot only responds to `/id`.
4. **Run it as a service** (it starts again after the machine restarts; `loginctl enable-linger $USER` is required to run while you’re logged out):
   ```bash
   mkdir -p dados/area ~/.config/systemd/user
   cp systemd/jarvis-na-pratica-telegram.gerado.service ~/.config/systemd/user/jarvis-na-pratica-telegram.service   # generated by instalar.sh with this folder’s path
   systemctl --user daemon-reload && systemctl --user enable --now jarvis-na-pratica-telegram
   journalctl --user -u jarvis-na-pratica-telegram -f      # should show "Jarvis no Telegram: @seu_bot"
   ```
   Stop: `systemctl --user stop jarvis-na-pratica-telegram` · restart: `systemctl --user restart jarvis-na-pratica-telegram` · disable permanently: `systemctl --user disable --now jarvis-na-pratica-telegram`.
5. **Chat commands:** `/cerebro` (shows which brain is thinking) · `/cerebro claude|codex|openrouter` (switches brains; memory and safety lock remain) · `/sim` (yes) and `/nao` (no) (approval, 120 s) · `/id` · `/ajuda`.

Under the hood: `telegram_jarvis.py` uses INTELECTO’s own channel (`intelecto/canais/telegram.py`: owner list, `/id`, approval via `/sim` and `/nao`), with long polling (no open port) and a file lock (single instance). The token is read from `.env` without being passed into the process environment, so `claude -p` and `codex exec` never receive it. Memory and the working folder are in `dados/` (variable `JARVIS_HOME`) and persist across restarts. The service starts with the `claude` brain; OpenRouter is used only if you send `/cerebro openrouter`.

### How it works under the hood

| file | what it does |
|---|---|
| `cerebros/comum.py` | the brain rules, text conversation, and conversion of the response to INTELECTO’s format |
| `cerebros/claude.py` | `claude -p --tools "" --json-schema …` in an empty folder: Claude only decides; it doesn’t run anything |
| `cerebros/codex.py` | `codex exec -s read-only --ephemeral --output-schema …` with the prompt through stdin (closed); arguments arrive in `args_json` because OpenAI’s schema is strict |
| `cerebros/openrouter.py` | `chat/completions` call with native tools and `usage.include` (costs are charged to your account) |
| `telegram_jarvis.py` | Telegram mode: INTELECTO channel + `/cerebro` to switch AI during the conversation, token only from `.env`, single instance |
| `systemd/jarvis-na-pratica-telegram.service` | the user service that keeps the bot running |
| `jarvis.py` | assembles INTELECTO’s 8 components with the selected brain (`montar(config, provedor=…)`), without changing INTELECTO; `medir`, `avaliar`, `conversar`, `trocar` |
| `instalar.sh` | venv + fixed INTELECTO v2 + checks + tests |
| `testes/test_cerebros.py` | 8 tests without a model: fake Claude and Codex (scripts), fake OpenRouter (local server), full conversation with mock |
| `testes/test_telegram.py` | 5 tests without Telegram or a model: fake Telegram (local server) and scripted brains — unknown users ignored, owner allowed, /nao keeps, /sim deletes, /cerebro switches and remembers, token outside environment, single instance |
| `resultados/` | JSON and MD measurement results |

The brain never touches the machine. It returns `{"texto", "chamadas"}`; INTELECTO runs the commands after the safety lock: read-only commands are allowed, any other command requires the owner’s approval, and known destructive commands (`sudo`, `reboot`, `mkfs`, `rm -rf /`…) are blocked even with approval. Commands run in the network-isolated Docker sandbox, confined to the working folder, and everything is recorded in the audit log.

### Tests

```bash
.venv/bin/python -m pytest -q testes     # 13 passed, without calling any model or Telegram
```

## The process, start to finish

| # | step | command | who does it |
|---|---|---|---|
| 1 | Check the machine (Python, Docker, which brains are available) | `./instalar.sh` | ⚙️ script |
| 2 | Rehearsal without AI | `jarvis.py medir --cerebros mock --executor local` | ⚙️ script |
| 3 | Measure the same conversation with each brain | `jarvis.py medir` | ⚙️ script (calls 🤖 `claude -p`, 🤖 `codex exec`, and the OpenRouter API) |
| 4 | Run the 8 standard test tasks | `jarvis.py avaliar --cerebro <nome>` | ⚙️ script |
| 4b | Switch brains midway through the conversation | `jarvis.py trocar --de <nome> --para <nome>` | ⚙️ script |
| 5 | Review results and choose a brain | `resultados/*.md` | 🧠 you |
| 6 | Use it daily in the terminal | `jarvis.py conversar --cerebro <nome>` | 🧠 you |
| 7 | Start the Telegram bot | `.env` + `systemctl --user enable --now jarvis-na-pratica-telegram` | 🧠 you + ⚙️ service |
| 8 | Chat and switch AI from your phone | `/cerebro codex` | 🧠 you |

## Credits and license

Built on [INTELECTO v2](https://github.com/inematds/intelecto-v2) ([guide](https://inematds.github.io/intelecto-v2/guia/) (in Portuguese)). MIT.
Free on [inema.club](https://inema.club), directly on the internet.