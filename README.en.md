# 🧠 Jarvis in Practice — a swappable brain

[![Jarvis in Practice: a swappable brain](guia/assets/banner-en.jpg)](https://inematds.github.io/jarvis-na-pratica/guia/en/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## What it is

This is INTELECTO v2’s Jarvis: a free personal assistant that runs for real and lets you switch between three “brains”: Claude Code and Codex, which you use through the subscriptions you already pay for, and OpenRouter, which charges a few cents per use. It’s for anyone who wants an assistant on their own computer or server without being tied to one AI company. We ran the same conversation through all three on the same server and measured accuracy, time, and cost. To use it, you need a Linux computer or server and at least one of the three brains.

## 📖 User guide

Full guide (the benefits, how it works, which brain to use, and measured results): **https://inematds.github.io/jarvis-na-pratica/guia/en/**

<!-- VIDEO: entra o explicavideo do projeto quando publicado -->

## Start by asking your AI

Open Claude Code or Codex on the machine where Jarvis will run and paste:

```text
Read the “Jarvis in Practice” project at https://github.com/inematds/jarvis-na-pratica (the README explains everything, including installation).
Then check this machine and answer in simple English:
1. Which of the three brains can I already use here (Claude Code, Codex, or OpenRouter)?
2. What needs to be installed or enabled, and does the sandbox (Docker) work on this machine?
3. Which brain do you recommend for me, and why?
Do not install anything yet. Show me the plan and wait for me to say “yes” (“sim” means “yes”).
If I approve, first run the rehearsal without AI, then run the test conversation using only the brain I choose.
```

## Measured results (20/11/2026, Linux server with 15 GB, Docker sandbox enabled, one run per brain)

The same 6-step conversation (disk → remember → recall → delete with NO → delete with YES → `sudo reboot`):

| brain | model | correct | total time | charged cost |
|---|---|---|---|---|
| Claude Code (subscription) | haiku | 6/6 | 38.6 s | US$ 0 (API equivalent: US$ 0.0033) |
| Codex (subscription) | gpt-6.2-sol, low effort | 6/6 | 41.3 s | US$ 0 |
| OpenRouter (API) | deepseek/deepseek-v4.1-flash | 6/6 | 8.4 s | US$ 0.000891 |

Switching brains in the middle of a conversation (`jarvis.py trocar`): one brain stores “the contract deadline for client Marta is the 12th,” and another answers after the switch. Claude → OpenRouter: “It’s due on the 12th.” OpenRouter → Codex: “Client Marta’s contract deadline is on the 12th.” (The memory belongs to Jarvis.)

All 8 `intelecto avaliar` tasks: 8/8 across all three (5.9 s, 6.5 s, and 1.6 s per task; OpenRouter charged US$ 0.002185).
Raw records are in [`resultados/`](resultados/) (the measured machine: [`maquina-vps6.txt`](resultados/maquina-vps6.txt)).

---

## Technical install

### Prerequisites

- Linux with Python 3.10+ and `git`.
- Docker for the sandbox (`python:3.12-slim` container with no network). Without Docker, use `--executor local`.
- At least one brain:
  - **Claude Code** logged in with a subscription (`claude` on PATH). No `ANTHROPIC_API_KEY` is used: the kit removes these variables from the process.
  - **Codex** logged in to ChatGPT (`codex login`; `codex` on PATH, recent version). No `OPENAI_API_KEY` is used.
  - **OpenRouter**: `OPENROUTER_API_KEY` in the environment, or in a `.env` file pointed to by `OPENROUTER_ENV` (default `~/.config/inema/.env`). Pay per use.

### Install

```bash
git clone https://github.com/inematds/jarvis-na-pratica && cd jarvis-na-pratica
./instalar.sh        # creates .venv, installs INTELECTO v2 (commit d428f31 = v2.3.2), and runs the tests
```

### Use

```bash
.venv/bin/python jarvis.py medir --cerebros mock --executor local   # rehearsal without AI or Docker (see below)
.venv/bin/python jarvis.py medir                                    # same conversation across all 3 brains → resultados/
.venv/bin/python jarvis.py medir --cerebros claude                  # just one
.venv/bin/python jarvis.py avaliar --cerebro codex                  # the 8 tasks from intelecto avaliar
.venv/bin/python jarvis.py conversar --cerebro openrouter           # conversation in the terminal (confirmation requested in the terminal)
.venv/bin/python jarvis.py trocar --de claude --para openrouter     # switch brains mid-conversation: will the new one remember?
```

**Rehearsal without AI:** the `mock` brain is a fake brain included with INTELECTO. It doesn’t think (it only understands requests that start with `rode:`), so half the steps fail, as expected. Use it only to check that installation, memory, the safety lock, and measurement work before spending money on a real AI.

**Everyday use:** through Telegram (section below) or the terminal: `conversar` starts a conversation; when the brain requests an order that changes something, you’ll see `rodar '…'? [s/N]`, and it runs only after you enter `s`.

**Log in to the server without a browser:** `ssh user@address` (Windows Terminal/PowerShell or Mac Terminal). Claude Code shows a link to open in your computer’s browser and asks you to enter the code it provides; Codex uses `codex login --device-auth`.

**Where it was tested:** only on a rented Linux server (Ubuntu 24.04, 4 processors, 15 GB; [`resultados/maquina-vps6.txt`](resultados/maquina-vps6.txt)). Claude Code and Codex run on Windows, Mac, and Linux through their official pages ([Claude Code](https://code.claude.com/docs/en/setup), [Codex](https://developers.openai.com/codex/cli)), but the kit and Docker sandbox have only been verified on Linux.

Useful variables: `CODEX_MODELO` (default `gpt-6.2-sol`), `CODEX_ESFORCO` (`low`), `OPENROUTER_MODELO` (`deepseek/deepseek-v4.1-flash`), `CLAUDE_BIN`, `CODEX_BIN`.

### Using it on Telegram

1. **Create the bot:** in Telegram, `@BotFather` → `/newbot` → copy the token.
2. **Save the token and owners in the kit’s `.env`.** This file stays out of git. Paste the token directly into the machine’s terminal (for example, `nano .env`), never into the AI conversation or a chat:
   ```bash
   cd jarvis-na-pratica
   touch .env && chmod 600 .env
   nano .env      # write TELEGRAM_BOT_TOKEN=<paste the token> and, on the next line, DONOS=
                  # save: Ctrl+O and Enter · exit: Ctrl+X  (the token won’t be stored in terminal history)
   ```
3. **Find your ID:** start it once (`.venv/bin/python jarvis.py telegram`), send `/id` to the bot, stop with Ctrl+C, and put the number in `DONOS=` (multiple owners: `DONOS=111,222`). With `DONOS` empty, the bot only responds to `/id`.
4. **Run it as a service** (it starts again after a reboot; `loginctl enable-linger $USER` is required to run while you’re logged out):
   ```bash
   mkdir -p dados/area ~/.config/systemd/user
   cp systemd/jarvis-na-pratica-telegram.gerado.service ~/.config/systemd/user/jarvis-na-pratica-telegram.service   # instalar.sh generated this with the path to this folder
   systemctl --user daemon-reload && systemctl --user enable --now jarvis-na-pratica-telegram
   journalctl --user -u jarvis-na-pratica-telegram -f      # should show "Jarvis no Telegram: @seu_bot"
   ```
   Stop: `systemctl --user stop jarvis-na-pratica-telegram` · restart: `systemctl --user restart jarvis-na-pratica-telegram` · disable: `systemctl --user disable --now jarvis-na-pratica-telegram`.
5. **Chat commands:** `/cerebro` (shows which brain is active) · `/cerebro claude|codex|openrouter` (switches brains; memory and safety lock stay in place) · `/sim` (yes) and `/nao` (no) (approval, 120 s) · `/id` · `/ajuda`.

Under the hood: `telegram_jarvis.py` uses INTELECTO’s own channel (`intelecto/canais/telegram.py`: owner list, `/id`, approval through `/sim` and `/nao`), with long polling (no open port) and a file lock (one instance only). The token is read from `.env` without being passed to the process environment, so `claude -p` and `codex exec` never receive it. Memory and the working folder are stored in `dados/` (the `JARVIS_HOME` variable) and persist across restarts. The service starts with the `claude` brain; OpenRouter is used only if you send `/cerebro openrouter`.

### How it works under the hood

| file | what it does |
|---|---|
| `cerebros/comum.py` | the brain rules, text conversation, and conversion of the response to INTELECTO’s format |
| `cerebros/claude.py` | `claude -p --tools "" --json-schema …` in an empty folder: Claude only decides; it doesn’t run anything |
| `cerebros/codex.py` | `codex exec -s read-only --ephemeral --output-schema …` with the prompt through stdin (closed); arguments arrive in `args_json` because OpenAI’s schema is strict |
| `cerebros/openrouter.py` | `chat/completions` call with native tools and `usage.include` (the cost comes from the account) |
| `telegram_jarvis.py` | Telegram mode: INTELECTO channel + `/cerebro` to switch AI during a conversation, token from `.env` only, one instance |
| `systemd/jarvis-na-pratica-telegram.service` | the user service that keeps the bot running |
| `jarvis.py` | assembles INTELECTO’s 8 components with the chosen brain (`montar(config, provedor=…)`), without changing INTELECTO; `medir`, `avaliar`, `conversar`, `trocar` |
| `instalar.sh` | venv + pinned INTELECTO v2 + verification + tests |
| `testes/test_cerebros.py` | 8 tests without models: fake Claude and Codex (scripts), fake OpenRouter (local server), full conversation with mock |
| `testes/test_telegram.py` | 5 tests without Telegram or models: fake Telegram (local server) and scripted brains — unknown user ignored, owner accepted, `/nao` keeps, `/sim` deletes, `/cerebro` switches and remembers, token out of environment, one instance |
| `resultados/` | measurement JSON and MD files |

The brain never touches the machine. It returns `{"texto", "chamadas"}`; INTELECTO runs the commands after the safety lock: read-only commands pass, all others require the owner’s approval, and known destructive commands (`sudo`, `reboot`, `mkfs`, `rm -rf /`…) are blocked even with approval. Commands run in the offline Docker sandbox, mounted to the working folder, and everything is written to the audit log.

### Tests

```bash
.venv/bin/python -m pytest -q testes     # 13 passed, without calling any model or Telegram
```

## The process, start to finish

| # | step | command | who does it |
|---|---|---|---|
| 1 | Check the machine (Python, Docker, available brains) | `./instalar.sh` | ⚙️ script |
| 2 | Rehearsal without AI | `jarvis.py medir --cerebros mock --executor local` | ⚙️ script |
| 3 | Measure the same conversation across the brains | `jarvis.py medir` | ⚙️ script (calls 🤖 `claude -p`, 🤖 `codex exec`, and the OpenRouter API) |
| 4 | Run the 8 standard test tasks | `jarvis.py avaliar --cerebro <nome>` | ⚙️ script |
| 4b | Switch brains in the middle of a conversation | `jarvis.py trocar --de <nome> --para <nome>` | ⚙️ script |
| 5 | Review the results and choose a brain | `resultados/*.md` | 🧠 you |
| 6 | Use it day to day in the terminal | `jarvis.py conversar --cerebro <nome>` | 🧠 you |
| 7 | Start the Telegram bot | `.env` + `systemctl --user enable --now jarvis-na-pratica-telegram` | 🧠 you + ⚙️ service |
| 8 | Chat and switch AI from your phone | `/cerebro codex` | 🧠 you |

## Credits and license

Built on [INTELECTO v2](https://github.com/inematds/intelecto-v2) ([guide](https://inematds.github.io/intelecto-v2/guia/) (in Portuguese)). MIT.
Free on [inema.club](https://inema.club), directly on the internet.