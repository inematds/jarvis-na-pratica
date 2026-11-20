# 🧠 Jarvis in practice — a swappable brain

[![Jarvis in practice: a swappable brain](guia/assets/banner-en.jpg)](https://inematds.github.io/jarvis-na-pratica/guia/en/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## What it is

This is INTELECTO v2’s Jarvis: a free personal assistant that runs for real with three swappable “brains”: Claude Code and Codex, which you use through the subscription you already pay for, and OpenRouter, which charges fractions of a cent per use. It’s for anyone who wants an assistant on their own computer or server without being tied to one AI company. We set up all three to have the same conversation on the same server, then measured accuracy, time, and cost. To use it, you need a Linux computer or server and at least one of the three brains.

## 📖 User guide

Complete guide (the benefits, how it works, which brain to use, and the measured results): **https://inematds.github.io/jarvis-na-pratica/guia/en/**

<!-- VIDEO: entra o explicavideo do projeto quando publicado -->

## Start by asking your AI

Open Claude Code or Codex on the machine where Jarvis will run and paste:

```text
Read the “Jarvis in practice” project at https://github.com/inematds/jarvis-na-pratica (the README explains everything, including installation).
Then check this machine and answer in plain English:
1. Which of the three brains can I already use here (Claude Code, Codex, or OpenRouter)?
2. What needs to be installed or enabled, and does the sandbox (Docker) work on this machine?
3. Which brain do you recommend for me, and why?
Don’t install anything yet. Show me the plan and wait for me to say yes.
If I approve, run the rehearsal without AI first, then the test conversation using only the brain I choose.
```

## Measured results (20/11/2026, Linux server with 15 GB, Docker sandbox enabled, one run per brain)

The same 6-step conversation (disk → remember → recall → delete with NO → delete with YES → `sudo reboot`):

| brain | model | correct | total time | cost charged |
|---|---|---|---|---|
| Claude Code (subscription) | haiku | 6/6 | 38.6 s | US$ 0 (API equivalent: US$ 0.0033) |
| Codex (subscription) | gpt-6.2-sol, low effort | 6/6 | 41.3 s | US$ 0 |
| OpenRouter (API) | deepseek/deepseek-v4.1-flash | 6/6 | 8.4 s | US$ 0.000891 |

The 8 tasks in `intelecto avaliar`: 8/8 for all three (5.9 s, 6.5 s, and 1.6 s per task; OpenRouter charged US$ 0.002185).
The raw records are in [`resultados/`](resultados/) ([`maquina-vps6.txt`](resultados/maquina-vps6.txt)).

---

## Technical install

### Prerequisites

- Linux with Python 3.10+ and `git`.
- Docker for the sandbox (`python:3.12-slim` container with no network). Without Docker, use `--executor local`.
- At least one brain:
  - **Claude Code**, logged in through your subscription (`claude` on `PATH`). No `ANTHROPIC_API_KEY` is used: the kit removes these variables from the process.
  - **Codex**, logged in to ChatGPT (`codex login`; `codex` on `PATH`, recent version). No `OPENAI_API_KEY` is used.
  - **OpenRouter**: `OPENROUTER_API_KEY` in the environment, or in a `.env` file specified by `OPENROUTER_ENV` (default `~/.config/inema/.env`). Pay per use.

### Install

```bash
git clone https://github.com/inematds/jarvis-na-pratica && cd jarvis-na-pratica
./instalar.sh        # creates .venv, installs INTELECTO v2 (commit d428f31 = v2.3.2), and runs the tests
```

### Use

```bash
.venv/bin/python jarvis.py medir --cerebros mock --executor local   # rehearsal without AI or Docker
.venv/bin/python jarvis.py medir                                    # same conversation with all 3 brains → resultados/
.venv/bin/python jarvis.py medir --cerebros claude                  # just one
.venv/bin/python jarvis.py avaliar --cerebro codex                  # the 8 tasks from intelecto avaliar
.venv/bin/python jarvis.py conversar --cerebro openrouter           # chat in the terminal (terminal confirmation prompt)
```

Useful variables: `CODEX_MODELO` (default `gpt-6.2-sol`), `CODEX_ESFORCO` (`low`), `OPENROUTER_MODELO` (`deepseek/deepseek-v4.1-flash`), `CLAUDE_BIN`, `CODEX_BIN`.

### How it works under the hood

| file | what it does |
|---|---|
| `cerebros/comum.py` | brain rules, the text conversation, and conversion of the response to INTELECTO’s format |
| `cerebros/claude.py` | `claude -p --tools "" --json-schema …` in an empty directory: Claude only decides; it doesn’t run anything |
| `cerebros/codex.py` | `codex exec -s read-only --ephemeral --output-schema …` with the prompt through stdin (sandboxed); arguments arrive in `args_json` because OpenAI’s schema is strict |
| `cerebros/openrouter.py` | `chat/completions` call with native tools and `usage.include` (cost comes from the account) |
| `jarvis.py` | assembles INTELECTO’s 8 components with the selected brain (`montar(config, provedor=…)`), without changing INTELECTO; `medir`, `avaliar`, `conversar` |
| `instalar.sh` | venv + pinned INTELECTO v2 + checks + tests |
| `testes/test_cerebros.py` | 8 tests without a model: fake Claude and Codex (scripts), fake OpenRouter (local server), full conversation with the mock |
| `resultados/` | measurement JSON and MD files |

The brain never touches the machine. It returns `{"texto", "chamadas"}`; INTELECTO handles execution after the guard: read-only commands pass, anything else asks the owner for approval, and known destructive commands (`sudo`, `reboot`, `mkfs`, `rm -rf /`…) are blocked even with approval. Commands run in the networkless Docker sandbox, restricted to the working directory, and everything is recorded in the audit log.

### Tests

```bash
.venv/bin/python -m pytest -q testes     # 8 passed, without calling any model
```

## The process, start to finish

| # | step | command | who does it |
|---|---|---|---|
| 1 | Check the machine (Python, Docker, available brains) | `./instalar.sh` | ⚙️ script |
| 2 | Rehearse without AI | `jarvis.py medir --cerebros mock --executor local` | ⚙️ script |
| 3 | Measure the same conversation with each brain | `jarvis.py medir` | ⚙️ script (calls 🤖 `claude -p`, 🤖 `codex exec`, and the OpenRouter API) |
| 4 | Run the 8 tasks from the standard test | `jarvis.py avaliar --cerebro <nome>` | ⚙️ script |
| 5 | Review the results and choose a brain | `resultados/*.md` | 🧠 you |
| 6 | Use it day to day | `jarvis.py conversar --cerebro <nome>` | 🧠 you |

## Credits and license

Built on [INTELECTO v2](https://github.com/inematds/intelecto-v2) ([guide](https://inematds.github.io/intelecto-v2/guia/) (in Portuguese)). MIT.
Free on [inema.club](https://inema.club), right on the internet.