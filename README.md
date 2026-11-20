# 🧠 Jarvis na prática — cérebro trocável

[![Jarvis na prática: cérebro trocável](guia/assets/banner.jpg)](https://inematds.github.io/jarvis-na-pratica/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## O que é

É o Jarvis do INTELECTO v2, um assistente pessoal grátis, rodando de verdade com três "cérebros" trocáveis: o Claude Code e o Codex, que você usa pela assinatura que já paga, e o OpenRouter, que cobra centavos por uso. Serve para quem quer um assistente no próprio computador ou servidor sem ficar preso a uma empresa de IA. A gente pôs os três para ter a mesma conversa, no mesmo servidor, e mediu acerto, tempo e custo. Para usar, você precisa de um computador ou servidor Linux e de pelo menos um dos três cérebros.

## 📖 Guia de uso

Guia completo (o ganho, como funciona, qual cérebro usar e a prova medida): **https://inematds.github.io/jarvis-na-pratica/guia/**

<!-- VIDEO: entra o explicavideo do projeto quando publicado -->

## Comece pedindo para a sua IA

Abra o Claude Code ou o Codex na máquina onde o Jarvis vai morar e cole:

```text
Leia o projeto "Jarvis na prática" em https://github.com/inematds/jarvis-na-pratica (o README explica tudo, inclusive a instalação).
Depois confira esta máquina e me responda em português simples:
1. Quais dos três cérebros eu já consigo usar aqui (Claude Code, Codex ou OpenRouter)?
2. O que falta instalar ou ligar, e se a caixa fechada (Docker) funciona nesta máquina.
3. Qual cérebro você recomenda para mim, e por quê.
Não instale nada ainda. Mostre o plano e espere eu dizer sim.
Se eu aprovar, rode primeiro o ensaio sem IA e depois a conversa de teste só no cérebro que eu escolher.
```

## O resultado medido (20/11/2026, servidor Linux de 15 GB, caixa Docker ligada, uma rodada por cérebro)

A mesma conversa de 6 passos (disco → lembrar → recordar → apagar com NÃO → apagar com SIM → `sudo reboot`):

| cérebro | modelo | acertos | tempo total | custo cobrado |
|---|---|---|---|---|
| Claude Code (assinatura) | haiku | 6/6 | 38,6 s | US$ 0 (equivalente na API: US$ 0,0033) |
| Codex (assinatura) | gpt-6.2-sol, esforço baixo | 6/6 | 41,3 s | US$ 0 |
| OpenRouter (API) | deepseek/deepseek-v4.1-flash | 6/6 | 8,4 s | US$ 0,000891 |

Troca de cérebro no meio da conversa (`jarvis.py trocar`): um cérebro guarda "o prazo do contrato da cliente Marta vence dia 12", outro responde depois da troca. Claude → OpenRouter: "Vence dia 12." OpenRouter → Codex: "O prazo do contrato da cliente Marta vence no dia 12." (a memória é do Jarvis).

As 8 tarefas do `intelecto avaliar`: 8/8 nos três (5,9 s, 6,5 s e 1,6 s por tarefa; o OpenRouter cobrou US$ 0,002185).
Os registros brutos estão em [`resultados/`](resultados/) (a máquina medida: [`maquina-vps6.txt`](resultados/maquina-vps6.txt)).

---

## Instalação técnica

### Pré-requisitos

- Linux com Python 3.10+ e `git`.
- Docker, para a caixa fechada (contêiner `python:3.12-slim` sem rede). Sem Docker, use `--executor local`.
- Pelo menos um cérebro:
  - **Claude Code** logado na assinatura (`claude` no PATH). Nenhuma `ANTHROPIC_API_KEY` é usada: o kit tira essas variáveis do processo.
  - **Codex** logado no ChatGPT (`codex login`; `codex` no PATH, versão recente). Nenhuma `OPENAI_API_KEY` é usada.
  - **OpenRouter**: `OPENROUTER_API_KEY` no ambiente, ou num arquivo `.env` apontado por `OPENROUTER_ENV` (padrão `~/.config/inema/.env`). Paga por uso.

### Instalar

```bash
git clone https://github.com/inematds/jarvis-na-pratica && cd jarvis-na-pratica
./instalar.sh        # cria .venv, instala o INTELECTO v2 (commit d428f31 = v2.3.2) e roda os testes
```

### Usar

```bash
.venv/bin/python jarvis.py medir --cerebros mock --executor local   # ensaio sem IA e sem Docker (ver abaixo)
.venv/bin/python jarvis.py medir                                    # a mesma conversa nos 3 cérebros → resultados/
.venv/bin/python jarvis.py medir --cerebros claude                  # só um
.venv/bin/python jarvis.py avaliar --cerebro codex                  # as 8 tarefas do intelecto avaliar
.venv/bin/python jarvis.py conversar --cerebro openrouter           # conversa no terminal (OK pedido no terminal)
.venv/bin/python jarvis.py trocar --de claude --para openrouter     # troca o cérebro no meio: o novo lembra?
```

**Ensaio sem IA:** o cérebro `mock` é um cérebro de mentira que vem com o INTELECTO. Ele não pensa (só entende pedidos que começam com `rode:`), então metade dos passos sai como falha, e isso é esperado. Serve só para conferir que a instalação, a memória, a trava e a medição funcionam antes de gastar com uma IA de verdade.

**No dia a dia:** `conversar` abre a conversa no terminal; quando o cérebro pede uma ordem que muda algo, aparece `rodar '…'? [s/N]` e só roda com `s`. Falar pelo Telegram não está ligado neste kit.

**Onde foi testado:** só num servidor Linux alugado (Ubuntu 24.04, 4 processadores, 15 GB; [`resultados/maquina-vps6.txt`](resultados/maquina-vps6.txt)). Claude Code e Codex rodam em Windows, Mac e Linux pelas páginas oficiais ([Claude Code](https://code.claude.com/docs/en/setup), [Codex](https://developers.openai.com/codex/cli)), mas o kit e a caixa Docker só foram provados no Linux.

Variáveis úteis: `CODEX_MODELO` (padrão `gpt-6.2-sol`), `CODEX_ESFORCO` (`low`), `OPENROUTER_MODELO` (`deepseek/deepseek-v4.1-flash`), `CLAUDE_BIN`, `CODEX_BIN`.

### Como funciona por dentro

| arquivo | o que faz |
|---|---|
| `cerebros/comum.py` | as regras do cérebro, a conversa em texto e a conversão da resposta para o formato do INTELECTO |
| `cerebros/claude.py` | `claude -p --tools "" --json-schema …` numa pasta vazia: o Claude só decide, não roda nada |
| `cerebros/codex.py` | `codex exec -s read-only --ephemeral --output-schema …` com o prompt pelo stdin (fechado); argumentos chegam em `args_json` porque o esquema da OpenAI é estrito |
| `cerebros/openrouter.py` | chamada `chat/completions` com ferramentas nativas e `usage.include` (o custo vem da conta) |
| `jarvis.py` | monta as 8 peças do INTELECTO com o cérebro escolhido (`montar(config, provedor=…)`), sem mudar o INTELECTO; `medir`, `avaliar`, `conversar`, `trocar` |
| `instalar.sh` | venv + INTELECTO v2 fixo + conferência + testes |
| `testes/test_cerebros.py` | 8 testes sem modelo: Claude e Codex falsos (scripts), OpenRouter falso (servidor local), conversa inteira com o mock |
| `resultados/` | os JSON e MD das medições |

O cérebro nunca toca na máquina. Ele devolve `{"texto", "chamadas"}`; quem roda é o INTELECTO, depois da trava: comando que só lê passa, qualquer outro pede o OK do dono, e os destrutivos conhecidos (`sudo`, `reboot`, `mkfs`, `rm -rf /`…) são barrados mesmo com OK. O que roda vai para a caixa Docker sem rede, presa na pasta de trabalho, e tudo entra no registro de auditoria.

### Testes

```bash
.venv/bin/python -m pytest -q testes     # 8 passed, sem chamar nenhum modelo
```

## O processo, do início ao fim

| # | passo | comando | quem faz |
|---|---|---|---|
| 1 | Conferir a máquina (Python, Docker, quais cérebros existem) | `./instalar.sh` | ⚙️ script |
| 2 | Ensaio sem IA | `jarvis.py medir --cerebros mock --executor local` | ⚙️ script |
| 3 | Medir a mesma conversa nos cérebros | `jarvis.py medir` | ⚙️ script (chama 🤖 `claude -p`, 🤖 `codex exec` e a API do OpenRouter) |
| 4 | Rodar as 8 tarefas do teste padrão | `jarvis.py avaliar --cerebro <nome>` | ⚙️ script |
| 4b | Trocar o cérebro no meio da conversa | `jarvis.py trocar --de <nome> --para <nome>` | ⚙️ script |
| 5 | Conferir os resultados e escolher o cérebro | `resultados/*.md` | 🧠 você |
| 6 | Usar no dia a dia | `jarvis.py conversar --cerebro <nome>` | 🧠 você |

## Créditos e licença

Construído sobre o [INTELECTO v2](https://github.com/inematds/intelecto-v2) ([guia](https://inematds.github.io/intelecto-v2/guia/)). MIT.
Grátis no [inema.club](https://inema.club), direto na internet.
