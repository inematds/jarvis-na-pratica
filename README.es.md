# 🧠 Jarvis en la práctica — cerebro intercambiable

[![Jarvis en la práctica: cerebro intercambiable](guia/assets/banner-es.jpg)](https://inematds.github.io/jarvis-na-pratica/guia/es/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## Qué es

Es el Jarvis de INTELECTO v2, un asistente personal gratuito que funciona de verdad con tres «cerebros» intercambiables: Claude Code y Codex, que usas con la suscripción que ya pagas, y OpenRouter, que cobra unos centavos por uso. Sirve para quienes quieren un asistente en su propia computadora o servidor sin depender de una empresa de IA. Pusimos a los tres a mantener la misma conversación en el mismo servidor y medimos la precisión, el tiempo y el costo. Para usarlo, necesitas una computadora o un servidor Linux y al menos uno de los tres cerebros.

## 📖 Guía de uso

Guía completa (el beneficio, cómo funciona, qué cerebro usar y la prueba medida): **https://inematds.github.io/jarvis-na-pratica/guia/es/**

<!-- VIDEO: entra o explicavideo do projeto quando publicado -->

## Empieza pidiéndoselo a tu IA

Abre Claude Code o Codex en la máquina donde va a funcionar Jarvis y pega esto:

```text
Lee el proyecto «Jarvis na prática» en https://github.com/inematds/jarvis-na-pratica (el README explica todo, incluida la instalación).
Después revisa esta máquina y respóndeme en español sencillo:
1. ¿Cuáles de los tres cerebros puedo usar aquí (Claude Code, Codex u OpenRouter)?
2. ¿Qué falta instalar o activar, y si la caja cerrada (Docker) funciona en esta máquina?
3. ¿Qué cerebro me recomiendas y por qué?
Todavía no instales nada. Muéstrame el plan y espera a que te diga que sí.
Si lo apruebo, ejecuta primero el ensayo sin IA y después la conversación de prueba solo con el cerebro que elija.
```

## El resultado medido (20/11/2026, servidor Linux de 15 GB, caja Docker activada, una ronda por cerebro)

La misma conversación de 6 pasos (disco → recordar → evocar → borrar con NO → borrar con SÍ → `sudo reboot`):

| cerebro | modelo | aciertos | tiempo total | costo cobrado |
|---|---|---|---|---|
| Claude Code (suscripción) | haiku | 6/6 | 38,6 s | US$ 0 (equivalente en la API: US$ 0,0033) |
| Codex (suscripción) | gpt-6.2-sol, esfuerzo bajo | 6/6 | 41,3 s | US$ 0 |
| OpenRouter (API) | deepseek/deepseek-v4.1-flash | 6/6 | 8,4 s | US$ 0,000891 |

Las 8 tareas de `intelecto avaliar`: 8/8 en los tres (5,9 s, 6,5 s y 1,6 s por tarea; OpenRouter cobró US$ 0,002185).
Los registros en bruto están en [`resultados/`](resultados/) ([`maquina-vps6.txt`](resultados/maquina-vps6.txt)).

---

## Instalación técnica

### Requisitos previos

- Linux con Python 3.10+ y `git`.
- Docker para la caja cerrada (contenedor `python:3.12-slim` sin red). Sin Docker, usa `--executor local`.
- Al menos un cerebro:
  - **Claude Code** con la sesión iniciada en la suscripción (`claude` en el PATH). No se usa ninguna `ANTHROPIC_API_KEY`: el kit elimina esas variables del proceso.
  - **Codex** con la sesión iniciada en ChatGPT (`codex login`; `codex` en el PATH, versión reciente). No se usa ninguna `OPENAI_API_KEY`.
  - **OpenRouter**: `OPENROUTER_API_KEY` en el entorno o en un archivo `.env` indicado por `OPENROUTER_ENV` (por defecto `~/.config/inema/.env`). Se paga por uso.

### Instalar

```bash
git clone https://github.com/inematds/jarvis-na-pratica && cd jarvis-na-pratica
./instalar.sh        # crea .venv, instala INTELECTO v2 (commit d428f31 = v2.3.2) y ejecuta las pruebas
```

### Usar

```bash
.venv/bin/python jarvis.py medir --cerebros mock --executor local   # ensayo sin IA ni Docker
.venv/bin/python jarvis.py medir                                    # la misma conversación con los 3 cerebros → resultados/
.venv/bin/python jarvis.py medir --cerebros claude                  # solo uno
.venv/bin/python jarvis.py evaluar --cerebro codex                  # las 8 tareas de intelecto avaliar
.venv/bin/python jarvis.py conversar --cerebro openrouter           # conversación en la terminal (OK solicitado en la terminal)
```

Variables útiles: `CODEX_MODELO` (por defecto `gpt-6.2-sol`), `CODEX_ESFORCO` (`low`), `OPENROUTER_MODELO` (`deepseek/deepseek-v4.1-flash`), `CLAUDE_BIN`, `CODEX_BIN`.

### Cómo funciona por dentro

| archivo | qué hace |
|---|---|
| `cerebros/comum.py` | las reglas del cerebro, la conversación en texto y la conversión de la respuesta al formato de INTELECTO |
| `cerebros/claude.py` | `claude -p --tools "" --json-schema …` en una carpeta vacía: Claude solo decide, no ejecuta nada |
| `cerebros/codex.py` | `codex exec -s read-only --ephemeral --output-schema …` con el prompt por stdin (cerrado); los argumentos llegan en `args_json` porque el esquema de OpenAI es estricto |
| `cerebros/openrouter.py` | llamada `chat/completions` con herramientas nativas y `usage.include` (el costo aparece en la cuenta) |
| `jarvis.py` | arma las 8 piezas de INTELECTO con el cerebro elegido (`montar(config, provedor=…)`), sin modificar INTELECTO; `medir`, `avaliar`, `conversar` |
| `instalar.sh` | venv + INTELECTO v2 fijo + comprobación + pruebas |
| `testes/test_cerebros.py` | 8 pruebas sin modelo: Claude y Codex falsos (scripts), OpenRouter falso (servidor local), conversación completa con el mock |
| `resultados/` | los archivos JSON y MD de las mediciones |

El cerebro nunca toca la máquina. Devuelve `{"texto", "chamadas"}`; quien ejecuta es INTELECTO, después del bloqueo: los comandos que solo leen pasan, cualquier otro pide la autorización del dueño y los comandos destructivos conocidos (`sudo`, `reboot`, `mkfs`, `rm -rf /`…) se bloquean incluso con autorización. Lo que se ejecuta va a la caja Docker sin red, restringida a la carpeta de trabajo, y todo queda en el registro de auditoría.

### Pruebas

```bash
.venv/bin/python -m pytest -q testes     # 8 passed, sin llamar a ningún modelo
```

## El proceso, de principio a fin

| # | paso | comando | quién lo hace |
|---|---|---|---|
| 1 | Revisar la máquina (Python, Docker, qué cerebros están disponibles) | `./instalar.sh` | ⚙️ script |
| 2 | Ensayo sin IA | `jarvis.py medir --cerebros mock --executor local` | ⚙️ script |
| 3 | Medir la misma conversación con los cerebros | `jarvis.py medir` | ⚙️ script (llama a 🤖 `claude -p`, 🤖 `codex exec` y a la API de OpenRouter) |
| 4 | Ejecutar las 8 tareas de la prueba estándar | `jarvis.py avaliar --cerebro <nome>` | ⚙️ script |
| 5 | Revisar los resultados y elegir el cerebro | `resultados/*.md` | 🧠 tú |
| 6 | Usarlo a diario | `jarvis.py conversar --cerebro <nome>` | 🧠 tú |

## Créditos y licencia

Construido sobre [INTELECTO v2](https://github.com/inematds/intelecto-v2) ([guía](https://inematds.github.io/intelecto-v2/guia/) (en portugués)). MIT.  
Gratis en [inema.club](https://inema.club), directamente en internet.