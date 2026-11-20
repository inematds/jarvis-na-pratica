# 🧠 Jarvis in practice — a swappable brain

[![Jarvis in practice: a swappable brain](guia/assets/banner-en.jpg)](https://inematds.github.io/jarvis-na-pratica/guia/en/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## What it is

It is INTELECTO v2’s Jarvis, a free personal assistant that really runs with three swappable “brains”: Claude Code and Codex, which you use through the subscriptions you already pay for, and OpenRouter, which charges fractions of a cent per use. It is for anyone who wants an assistant on their own computer or server without being tied to one AI company. We had all three handle the same conversation on the same server and measured accuracy, time, and cost. To use it, you need a Linux computer or server and at least one of the three brains.

## 📖 User guide

Complete guide (the benefits, how it works, which brain to use, and measured evidence): **https://inematds.github.io/jarvis-na-pratica/guia/en/**

<!-- VIDEO: entra o explicavideo do projeto quando publicado -->

## Start by asking your AI

Open Claude Code or Codex on the machine where Jarvis will run and paste:

```text
Read the “Jarvis in practice” project at https://github.com/inematds/jarvis-na-pratica (the README explains everything, including installation).
Then check this machine and answer me in simple English:
1. Which of the three brains can I already use here (Claude Code, Codex, or OpenRouter)?
2. What needs to be installed or enabled, and does the sandbox (Docker) work on this machine?
3. Which brain do you recommend for me, and why?
Do not install anything yet. Show me the plan and wait for me to say yes.
If I approve, first run the test without AI, then run the test conversation using only the brain I choose.
```

## Measured results (20/11/2026, Linux server with 15 GB, Docker sandbox enabled, one run per brain)

The same 6-step conversation (disk → remember → recall → delete with NO → delete with YES → `sudo reboot`):

| brain | model | correct | total time | cost charged |
|---|---|---|---|---|
| Claude Code (subscription) | haiku | 6/6 | 38.6 s | US$ 0 (API equivalent: US$ 0.0033) |
| Codex (subscription) | gpt-6.2-sol, low effort | 6/6 | 41.3 s | US$ 0 |
| OpenRouter (API) | deepseek/deepseek-v4.1-flash | 6/6 | 8.4 s | US$ 0.000891 |

Switch brains mid-conversation (`jarvis.py trocar`): one brain stores “the deadline for client Marta’s contract is the 12th,” and another responds after the switch. Claude → OpenRouter: “It’s due on the 12th.” OpenRouter → Codex: “The deadline for client Marta’s contract is on the 12th.” (the memory belongs to Jarvis).

All 8 tasks from `intelecto avaliar`: 8/8 for all three (5.9 s, 6.5 s, and 1.6 s per task; OpenRouter charged US$ 0.002185).
The raw records are in [`resultados/`](resultados/) (the measured machine: [`maquina-vps6.txt`](resultados/maquina-vps6.txt)).

---

## Technical install

### Prerequisites

- Linux with Python 3.10+ and `git`.
- Docker, for the sandbox (`python:3.12-slim` container with no network access). Without Docker, use `--executor local`.
- At least one brain:
  - **Claude Code** logged in through your subscription (`claude` on PATH). No `ANTHROPIC_API_KEY` is used: the toolkit removes these variables from the process.
  - **Codex** logged in to ChatGPT (`codex login`; `codex` on PATH, recent version). No `OPENAI_API_KEY` is used.
  - **OpenRouter**: `OPENROUTER_API_KEY` in the environment, or in a `.env` file specified by `OPENROUTER_ENV` (default `~/.config/inema/.env`). Pay per use.

### Install

```bash
git clone https://github.com/inematds/jarvis-na-pratica && cd jarvis-na-pratica
./instalar.sh        # creates .venv, installs INTELECTO v2 (commit d428f31 = v2.3.2), and runs the tests
```

### Use

```bash
.venv/bin/python jarvis.py medir --cerebros mock --executor local   # test without AI or Docker (see below)
.venv/bin/python jarvis.py medir                                    # same conversation with all 3 brains → resultados/
.venv/bin/python jarvis.py medir --cerebros claude                  # one only
.venv/bin/python jarvis.py avaliar --cerebro codex                  # the 8 tasks from intelecto avaliar
.venv/bin/python jarvis.py conversar --cerebro openrouter           # chat in the terminal (terminal prompt approval)
.venv/bin/python jarvis.py trocar --de claude --para openrouter     # switch brains mid-conversation: does the new one remember?
```

**Test without AI:** the `mock` brain is a fake brain included with INTELECTO. It does not think (it only understands requests that start with `rode:`), so half the steps fail, as expected. It is only for checking that installation, memory, safeguards, and measurement work before spending money on a real AI.

**For everyday use:** `conversar` starts a conversation in the terminal; when the brain requests an order that changes something, `rodar '…'? [s/N]` appears (Portuguese for "run?"; answer `s` or `y`), and it runs only if you enter `y`. Telegram chat is not enabled in this toolkit.

**Where it was tested:** only on a rented Linux server (Ubuntu 24.04, 4 processors, 15 GB; [`resultados/maquina-vps6.txt`](resultados/maquina-vps6.txt)). Claude Code and Codex run on Windows, Mac, and Linux according to their official pages ([Claude Code](https://code.claude.com/docs/en/setup), [Codex](https://developers.openai.com/codex/cli)), but the toolkit and Docker sandbox have only been proven on Linux.

Useful variables: `CODEX_MODELO` (default `gpt-6.2-sol`), `CODEX_ESFORCO` (`low`), `OPENROUTER_MODELO` (`deepseek/deepseek-v4.1-flash`), `CLAUDE_BIN`, `CODEX_BIN`.

### How it works inside

| file | what it does |
|---|---|
| `cerebros/comum.py` | brain rules, text conversation, and conversion of the response to INTELECTO format |
| `cerebros/claude.py` | `claude -p --tools "" --json-schema …` in an empty folder: Claude only decides; it runs nothing |
| `cerebros/codex.py` | `codex exec -s read-only --ephemeral --output-schema …` with the prompt sent through stdin (closed); arguments arrive in `args_json` because OpenAI’s schema is strict |
| `cerebros/openrouter.py` | `chat/completions` call with native tools and `usage.include` (cost is taken from the account) |
| `jarvis.py` | assembles INTELECTO’s 8 components with the selected brain (`montar(config, provedor=…)`), without changing INTELECTO; `medir`, `avaliar`, `conversar`, `trocar` |
| `instalar.sh` | venv + pinned INTELECTO v2 + verification + tests |
| `testes/test_cerebros.py` | 8 tests without a model: fake Claude and Codex (scripts), fake OpenRouter (local server), full conversation with mock |
| `resultados/` | JSON and MD files with the measurements |

The brain never touches the machine. It returns `{"texto", "chamadas"}`; INTELECTO runs them after the safeguard: commands that only read are allowed, anything else asks the owner for approval, and known destructive commands (`sudo`, `reboot`, `mkfs`, `rm -rf /`…) are blocked even with approval. Commands run in the offline Docker sandbox, restricted to the working folder, and everything is recorded in the audit log.

### Tests

```bash
.venv/bin/python -m pytest -q testes     # 8 passed, without calling any model
```

## The process, start to finish

| # | step | command | who does it |
|---|---|---|---|
| 1 | Check the machine (Python, Docker, available brains) | `./instalar.sh` | ⚙️ script |
| 2 | Test without AI | `jarvis.py medir --cerebros mock --executor local` | ⚙️ script |
| 3 | Measure the same conversation with each brain | `jarvis.py medir` | ⚙️ script (calls 🤖 `claude -p`, 🤖 `codex exec`, and the OpenRouter API) |
| 4 | Run the 8 standard test tasks | `jarvis.py avaliar --cerebro <nome>` | ⚙️ script |
| 4b | Switch brains mid-conversation | `jarvis.py trocar --de <nome> --para <nome>` | ⚙️ script |
| 5 | Review the results and choose a brain | `resultados/*.md` | 🧠 you |
| 6 | Use it day to day | `jarvis.py conversar --cerebro <nome>` | 🧠 you |

## Credits and license

Built on [INTELECTO v2](https://github.com/inematds/intelecto-v2) ([guide](https://inematds.github.io/intelecto-v2/guia/) (in Portuguese)). MIT.
Free on [inema.club](https://inema.club), directly on the internet.