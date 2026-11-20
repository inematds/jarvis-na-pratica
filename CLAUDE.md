# CLAUDE.md — jarvis-na-pratica

Jarvis do INTELECTO v2 com três cérebros trocáveis (Claude Code e Codex pela assinatura, OpenRouter por API).

- Conta e autor: `inematds <inematds@gmail.com>` (repo inematds/jarvis-na-pratica).
- Não alterar o repo intelecto-v2: o kit usa `montar(config, provedor=…)` e `avaliacao.avaliar(config, provedor)` por fora.
- Claude e Codex só pela assinatura (o código tira `ANTHROPIC_*`/`OPENAI_*` do processo). OpenRouter foi autorizado pelo Nei só para esta demonstração (20/11/2026).
- Número no guia = número em `resultados/`. Mudou a medição, refaça a tabela do guia nas 3 línguas e nos 3 READMEs.
- Onde roda a demonstração: VPS6 (`~/kits/jarvis-na-pratica`, usuário nmaldaner). É CLI, nada fica no ar.
- Testes: `python3 -m pytest -q testes` (sem modelo).

## Self-learning

Quando eu for corrigido, ou perceber um erro meu: antes de continuar, acrescentar a lição como uma linha no TOPO da seção Lessons abaixo.

## Lessons

- Codex `--output-schema` é estrito (additionalProperties false, tudo required): argumentos de ferramenta vão como texto JSON (`args_json`). (20/11/2026)
- OpenRouter só devolve o custo com `"usage": {"include": true}` no pedido. (20/11/2026)
