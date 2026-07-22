# dev-skills

> Set curado de agent skills de engenharia para o Claude Code + referência de setup do stack completo.

`skills: 19` · `stack: 6 componentes` · `scope: global` · `license: MIT`

Referência técnica para instalar e operar o stack de skills: `mattpocock/skills` (base),
`dev-skills` (este repo), `superpowers` (plugin), `spec-kit` (CLI), `claude-mem` (plugin de
memória) e `skills` (CLI de descoberta, `npx skills`). Formato de referência — cada operação
traz assinatura, parâmetros, retorno e exemplo.

---

## Por que este stack

O gargalo de codar com um agente não é mais a velocidade de gerar código — é o loop de
review-e-retrabalho quando o output diverge da intenção. Um estudo de 20.574 sessões reais
mostra que a falha dominante é *misalignment*: o agente produz código plausível e confiante
que resolve o problema errado ([arXiv 2605.29442](https://arxiv.org/pdf/2605.29442)). Este
stack existe para atacar isso: transforma trabalho ad-hoc ("vibe coding") em um fluxo com
enquadramento, spec, design explícito, TDD e verificação — cada etapa com um artefato que a
próxima consome.

### O que é usado

Quatro camadas, cada uma com um papel:

- **spec-kit** ([github/spec-kit](https://github.com/github/spec-kit)) — coloca a
  **especificação versionada como fonte de verdade**, não o código. Fluxo Spec → Plan →
  Tasks → Implement, cada fase gerando um artefato Markdown que alimenta a próxima
  ([IBM: o que é SDD](https://www.ibm.com/think/topics/spec-driven-development)).
- **superpowers** ([obra/superpowers](https://github.com/obra/superpowers)) — impõe a
  **disciplina clarify → design → plan → code → verify**: planejar antes de tocar em
  arquivo, escrever o teste antes da implementação, e revisar em duas etapas antes de dar
  por concluído.
- **mattpocock/skills** + **dev-skills** — o **vocabulário e os procedimentos** de
  engenharia (DDD, arquitetura, review, refactor, migração) empacotados como *agent skills*.
  Skills usam **progressive disclosure**: só os metadados ficam no contexto; o corpo carrega
  quando é relevante — o que torna o número de skills praticamente ilimitado sem inchar o
  context window ([Anthropic: Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).

### Problemas que resolve

| Problema | Sem o stack | Com o stack |
|----------|-------------|-------------|
| **Intent drift** | "adiciona login" é subespecificado; o agente chuta defaults que raramente batem com o que o time queria | a spec captura a intenção antes de gerar código ([SDD e drift](https://arxiv.org/html/2602.00180v1)) |
| **Context explosion** | o agente raciocina sobre o repo inteiro e a qualidade cai conforme o contexto enche | tarefas fatiadas + progressive disclosure mantêm o foco no que importa |
| **Spec-code drift silencioso** | o código evolui, a doc não; a divergência só aparece quando é cara de consertar | a spec é o contrato versionado, revisto a cada fase ([arXiv 2606.27045](https://arxiv.org/abs/2606.27045)) |
| **Testes que passam por construção** | sem testes, o agente assume que está tudo ok enquanto quebrou coisas | TDD obriga o teste a falhar antes da implementação existir |
| **Ambiguidade de domínio** | sinônimos e traduções erradas entre negócio e código | ubiquitous language alinha termo de negócio ↔ código, reduzindo abstrações enganosas ([DDD SLR, arXiv 2310.01905](https://arxiv.org/pdf/2310.01905)) |
| **Erosão arquitetural** | features novas ignoram as convenções existentes e a base degrada | design explícito (DDD/clean architecture) antes de implementar |

### Melhorias esperadas

Pragmaticamente, o ganho não é "menos código" — é **menos retrabalho**:

- **Menos ciclos de review-e-rework**, porque a intenção é fixada antes da geração e
  verificada em checkpoints, pegando drift cedo, antes de compor ([zeroshot: SDD com agentes](https://zeroshot.ghost.io/spec-driven-development-with-ai-coding-agents/)).
- **Output mais consistente com a arquitetura** e menos regressões silenciosas, porque
  existe um teste e um contrato para gatear cada mudança.
- **Contexto mais barato e mais focado** — progressive disclosure reduz tokens/custo e
  mantém a atenção do modelo só no passo atual, o que também reduz alucinação.
- **Menos ambiguidade** entre o que foi pedido e o que foi construído, via spec + linguagem
  ubíqua.

Trade-off honesto: o fluxo adiciona **overhead de cerimônia** (escrever spec, plano, testes).
Compensa em trabalho não-trivial e multi-sessão; para um one-liner, é exagero — use `/ask-matt`
para decidir a profundidade certa.

---

## Índice

- [Por que este stack](#por-que-este-stack) — o que é usado, problemas que resolve, ganhos
- [Componentes](#componentes) — os 4 módulos do stack
- [Guia de uso](#guia-de-uso) — qual skill em cada fase do trabalho
- [Setup via prompt](#setup-via-prompt-copiar-e-colar) — cole numa sessão em andamento
- [Pré-requisitos](#pré-requisitos)
- [Operações de instalação](#operações-de-instalação)
  - [`install mattpocock`](#install-mattpocock)
  - [`install dev-skills`](#install-dev-skills)
  - [`install superpowers`](#install-superpowers)
  - [`install spec-kit`](#install-spec-kit)
  - [`install claude-mem`](#install-claude-mem)
  - [`install find-skills`](#install-find-skills)
- [Configuração: hook SessionStart](#configuração-hook-sessionstart)
- [Operações por projeto](#operações-por-projeto)
- [Health checks](#health-checks)
- [Resources: catálogo de skills](#resources-catálogo-de-skills)
- [Manutenção](#manutenção)
- [Troubleshooting](#troubleshooting)
- [Referências](#referências) — papers, relatórios e casos verificados

---

## Componentes

| Componente | Tipo | Artefato instalado | Invocação | Fonte |
|------------|------|--------------------|-----------|-------|
| `mattpocock/skills` | skills (38) | `~/.claude/skills/*` | `/nome` · auto | `npx skills` |
| `dev-skills` | skills (19) | `~/.claude/skills/*` | `/nome` · auto | `npx skills` |
| `superpowers` | plugin | `~/.claude/plugins/*` | auto por sessão | plugin marketplace |
| `claude-mem` | plugin | `~/.claude/plugins/*` + worker `~/.claude-mem/` | auto por sessão · MCP `mcp-search` | plugin marketplace |
| `spec-kit` | CLI | `~/.local/bin/specify` | `specify <cmd>` | `uv tool` |
| `skills` (find-skills) | CLI | efêmero (`npx`) | `npx skills <cmd>` | `npx skills` |

**Guia de uso** (qual skill em cada fase): ver [Guia de uso](#guia-de-uso) abaixo.

---

## Guia de uso

Qual skill acionar em cada fase do trabalho. Invoque com `/nome`; muitas disparam sozinhas
pelo contexto. Origem: `matt` = mattpocock · `dev` = dev-skills · `sup` = superpowers ·
`spec` = spec-kit · `cli` = `npx skills` (find-skills).

### 1. Descoberta & enquadramento

Antes de escrever qualquer coisa: entenda o problema e pressione o plano.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/ask-matt` | matt | Não sabe qual skill/fluxo usar — roteador da situação |
| `brainstorming` | sup | Antes de qualquer trabalho criativo; explora intenção e requisitos (auto) |
| `/grilling` · `/grill-me` | matt | Testar um plano/design antes de construir |
| `/grill-with-docs` | matt | Mesma entrevista, gerando ADRs + glossário |
| `/research` | matt | Investigar contra fontes primárias e capturar em Markdown |
| `/wayfinder` | matt | Trabalho enorme, multi-sessão, como tickets de investigação |

### 2. Especificação

Transformar a ideia validada em spec, plano e tarefas.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `specify init` → `/speckit.specify` · `.plan` · `.tasks` | spec | Novo projeto/feature: Spec → Plan → Tasks → Implement |
| `writing-plans` | sup | Virar uma spec em plano executável antes de tocar código (auto) |
| `/to-prd` | matt | Consolidar a conversa num PRD e publicar no tracker |
| `/to-issues` | matt | Quebrar plano/PRD em issues "grabbable" |

### 3. Modelagem de domínio & arquitetura

Onde vive a lógica, quais as fronteiras, qual a linguagem do negócio.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/domain-driven-design` | dev | Bounded contexts, aggregates, ubiquitous language, context mapping |
| `/domain-modeling` · `/ubiquitous-language` | matt | Construir/afinar o modelo de domínio deste repo; glossário |
| `/clean-architecture` | dev | Decidir camada; Dependency Rule, hexagonal, onion, SOLID |
| `/codebase-design` | matt | Desenhar deep modules e seams; testabilidade |
| `/system-design` | dev | Sistema distribuído escalável: LB, cache, filas, particionamento |
| `/design-an-interface` | matt | Explorar múltiplos formatos de uma interface |
| `/improve-codebase-architecture` | matt | Auditar a base atual → relatório → grill |
| `/impeccable` | dev | **UI/frontend**: shape/audit/critique/polish de interface — design system, a11y, anti-genérico |

### 4. Padrões de backend

Design de APIs, dados e comunicação entre serviços.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/api-design-principles` | dev | Desenhar/revisar API REST ou GraphQL |
| `/cqrs-implementation` · `/event-store-design` | dev | Separar leitura/escrita; sistemas event-sourced |
| `/microservices-patterns` · `/saga-orchestration` | dev | Fronteiras de serviço, event-driven, transações distribuídas |
| `/postgresql-table-design` | dev | Modelar/revisar schema PostgreSQL |

### 5. Implementação

Executar o plano com disciplina — teste primeiro, verifique depois.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/implement` | matt | Implementar a partir de um PRD ou conjunto de issues |
| `test-driven-development` · `/tdd` | sup · matt | Qualquer feature/bugfix: red-green-refactor |
| `executing-plans` · `subagent-driven-development` | sup | Executar plano com checkpoints; despachar tarefas em subagentes |

### 6. Debug, revisão & qualidade

Quando algo quebra, ou antes de dar por concluído.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `systematic-debugging` · `/diagnosing-bugs` | sup · matt | Bug difícil, teste falhando, regressão — diagnóstico antes de corrigir |
| `/code-review` | matt | Revisar mudanças (eixos padrões do repo + fidelidade à spec) |
| `requesting-code-review` · `receiving-code-review` | sup | Ao concluir/antes do merge; lidar com feedback com rigor |
| `/refactoring-patterns` · `/clean-code` | dev | Melhorar código existente (smells; SRP, naming) |
| `/working-with-legacy-code` · `/tech-debt-tracker` | dev | Código sem teste ou dívida acumulada |
| `verification-before-completion` | sup | Antes de afirmar "pronto/passando": evidência antes da afirmação |
| `/qa` | matt | Sessão de QA conversacional que arquiva issues |

### 7. Entrega & manutenção

Pipeline, segurança, produção, migrações e higiene de git.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/ci-cd-pipelines` · `/security-hardening` | dev | Automatizar entrega; endurecer segurança de app/infra |
| `/release-it` | dev | Estabilidade em produção: circuit breakers, timeouts, chaos, observabilidade |
| `/migration-architect` | dev | Migração de risco (DB, cutover) com rollback explícito |
| `/setup-pre-commit` · `/resolving-merge-conflicts` · `/git-guardrails-claude-code` | matt | Pre-commit; resolver merge/rebase; bloquear git perigoso |
| `using-git-worktrees` · `finishing-a-development-branch` | sup | Workspace isolado; opções de merge/PR/cleanup ao terminar |

### 8. Colaboração, handoff & meta

Passar bastão, ensinar, triar e criar novas skills.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/handoff` · `/claude-handoff` | matt | Compactar conversa num doc de handoff |
| `/triage` · `/request-refactor-plan` | matt | Máquina de estados de triagem; plano de refactor como issue |
| `/teach` | matt | Ensinar uma nova skill/conceito no workspace |
| `writing-skills` · `/writing-great-skills` | sup · matt | Criar/editar skills |
| `task-observer` | dev | Observa a sessão → propõe melhorias/novas skills (revisar antes de aplicar) |
| `npx skills find` · `add` | cli | Descobrir e instalar skills do ecossistema aberto |

### 9. Escrita (não-código)

Para textos, artigos e notas — separado do fluxo de engenharia.

| Skill | Origem | Quando usar |
|-------|--------|-------------|
| `/writing-fragments` → `/writing-shape` → `/writing-beats` | matt | Do material bruto ao artigo |
| `/edit-article` · `/obsidian-vault` | matt | Reestruturar rascunho; notas no Obsidian |

### Fluxo padrão de uma feature

1. **Enquadre** — `brainstorming` / `/grill-me` para afiar a ideia.
2. **Especifique** — `specify init` → spec / plan / tasks.
3. **Desenhe** — `/domain-driven-design` + `/clean-architecture` conforme a fronteira.
4. **Implemente** — `test-driven-development`, uma tarefa por vez.
5. **Revise** — `/code-review` + `verification-before-completion` antes do "pronto".
6. **Entregue** — `finishing-a-development-branch`, e `/migration-architect` se houver risco.

---

## Pré-requisitos

| Dependência | Versão | Checagem | Necessária para |
|-------------|--------|----------|-----------------|
| Claude Code | — | `claude --version` | tudo |
| Node.js | `>=18` | `node -v` | `npx skills` |
| git | — | `git --version` | clone / dev-skills |
| uv | — | `uv --version` | spec-kit (passo 4) |
| gh CLI | — | `gh auth status` | tracker GitHub (opcional) |

---

## Setup via prompt (copiar e colar)

A forma mais rápida: cole o prompt abaixo **numa sessão do Claude Code já em andamento**.
Ele instala tudo seguindo este README.

```text
Instale e configure na minha máquina o stack de skills de engenharia do repo
https://github.com/afranciny/dev-skills, seguindo o README dele. Faça:

1. Instale as skills do mattpocock/skills e do afranciny/dev-skills globalmente para o
   Claude Code (npx skills add ... --global --agent claude-code --skill '*' -y).
2. Instale o plugin superpowers (marketplace obra/superpowers-marketplace) e o CLI do
   spec-kit (specify, via uv tool install).
3. Rode o health check do README e me mostre o resultado.
4. Me diga exatamente o que exige reabrir a sessão para passar a valer.

Não altere nada fora disso.
```

> **Importante:** skills e plugin só entram em vigor numa **sessão nova**. Depois que o
> Claude terminar de instalar, feche e rode `claude` de novo na mesma pasta do projeto —
> `/clear` não recarrega o catálogo de skills.

Já com a sessão reaberta, configure o projeto (setup guiado — tracker, labels, docs):

```text
/setup-matt-pocock-skills
```

E, para o fluxo spec-driven num repo existente: `specify init --here`.

---

## Operações de instalação

Passo a passo manual (o que o prompt acima automatiza).

### `install mattpocock`

Base do stack — 38 skills de processo (spec, review, TDD, triagem, domínio, escrita).

```bash
npx skills@latest add mattpocock/skills --global --agent claude-code --skill '*' -y
```

**Parâmetros**

| Flag | Tipo | Default | Descrição |
|------|------|---------|-----------|
| `--global` | bool | `false` | Instala em `~/.claude/skills` (user-level) |
| `--agent` | string | auto-detect | Agente alvo; `claude-code` é detectado sozinho |
| `--skill` | glob | — | `'*'` = todas as skills do repo |
| `-y` | bool | `false` | Pula prompts interativos |

**Retorno** — symlinks em `~/.claude/skills/` apontando para `~/.agents/skills/`.

---

### `install dev-skills`

Este repo — 19 skills net-new (DDD, arquitetura, backend, entrega, UI e meta) que complementam a base.

```bash
npx skills@latest add afranciny/dev-skills --global --agent claude-code --skill '*' -y
```

**Alternativa** — clone + symlink manual (sem `npx`):

```bash
git clone https://github.com/afranciny/dev-skills.git ~/dev-skills
for d in ~/dev-skills/*/; do
  [ -f "$d/SKILL.md" ] && ln -sfn "$d" "$HOME/.claude/skills/$(basename "$d")"
done
```

**Retorno** — 19 diretórios de skill disponíveis em `~/.claude/skills/`.

---

### `install superpowers`

Plugin que impõe disciplina brainstorm → plan → TDD → review.

```bash
claude plugin marketplace add obra/superpowers-marketplace
claude plugin install superpowers@superpowers-marketplace --scope user
```

**Parâmetros**

| Flag | Valores | Default | Descrição |
|------|---------|---------|-----------|
| `--scope` | `user` \| `project` \| `local` | `user` | Escopo de instalação do plugin |

**Equivalente in-app** — `/plugin marketplace add …` + `/plugin install …`.

**Retorno** — plugin ativo; carrega em toda nova sessão.

---

### `install spec-kit`

CLI `specify` — fluxo Spec → Plan → Tasks → Implement por projeto. Requer `uv`.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

**Retorno** — executável `~/.local/bin/specify` (garanta `~/.local/bin` no `PATH`).

---

### `install claude-mem`

Plugin de memória persistente entre sessões (worker local + SQLite + MCP `mcp-search`).

```bash
claude plugin marketplace add thedotmack/claude-mem
claude plugin install claude-mem@thedotmack --scope user
```

**Config** — `~/.claude-mem/settings.json` (flat key/value). Chaves úteis:
`CLAUDE_MEM_PROVIDER` (default `claude`, usa a assinatura), `CLAUDE_MEM_EXCLUDED_PROJECTS`
(padrões separados por vírgula — projetos a **não** capturar), telemetria off por
`telemetry.json`.

> **Privacidade/isolamento** — o worker captura observações de *todas* as tools num SQLite
> **global**. Em ambientes multi-cliente/confidenciais, exclua o projeto via
> `CLAUDE_MEM_EXCLUDED_PROJECTS` e mantenha Cloud Sync desligado. **Retorno** — plugin ativo;
> worker sobe a cada sessão.

---

### `install find-skills`

CLI para descobrir/instalar skills do ecossistema aberto. Não persiste — roda via `npx`.

```bash
npx skills find <termo>        # busca skills por palavra-chave
npx skills add <owner/repo> --global --agent claude-code   # instala
npx skills list                # lista instaladas
```

**Retorno** — nenhum artefato instalado do próprio CLI; ele instala/gerencia *outras* skills.

---

## Configuração: hook SessionStart

Imprime, a cada sessão, um lembrete de usar as skills com o link do guia. Adicionar em
`~/.claude/settings.json`, **mesclando** com o objeto existente.

**Schema**

```jsonc
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '<json>'"
          }
        ]
      }
    ]
  }
}
```

**Payload emitido pelo comando** (`<json>`)

| Campo | Destino | Descrição |
|-------|---------|-----------|
| `systemMessage` | usuário | Texto exibido ao abrir a sessão |
| `hookSpecificOutput.hookEventName` | runtime | Deve ser `"SessionStart"` |
| `hookSpecificOutput.additionalContext` | modelo | Contexto injetado para o modelo lembrar de acionar skills |

**Comando completo** (o que gravar em `command`):

```bash
echo '{"systemMessage":"Stack de skills ativo: mattpocock + dev-skills + superpowers + claude-mem + spec-kit + npx skills. Guia: https://afranciny.github.io/dev-skills/#guia-de-uso — acione /ask-matt se estiver em dúvida.","hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":"O usuário mantém um set curado de skills de engenharia (mattpocock + ~/dev-skills) + plugins superpowers/claude-mem + CLIs spec-kit/npx skills. Prefira acionar a skill adequada à fase do trabalho (descoberta, spec, design/DDD, UI via /impeccable, implementação, review, entrega, meta via task-observer)."}}'
```

> Sem contagem fixa no texto — a lista de skills cresce; nomear os componentes evita drift.

**Validação** — `jq empty ~/.claude/settings.json` (sem saída = ok). Efetivo na próxima
sessão; se não disparar, abra `/hooks` uma vez para recarregar a config.

---

## Operações por projeto

### `setup-matt-pocock-skills`

Config por repositório exigida pelas skills do mattpocock. Rodar **uma vez** dentro do projeto.

```
/setup-matt-pocock-skills
```

**Prompts** — issue tracker (`github` \| `gitlab` \| `linear` \| `local`), labels de
triagem, local dos docs. **Retorno** — `docs/agents/*.md` + bloco `## Agent skills` no `CLAUDE.md`.

### `specify init`

Inicia o fluxo spec-driven num projeto.

```bash
specify init <projeto> --integration claude
specify init <projeto> --integration claude --integration-options="--skills"   # como skills
```

---

## Health checks

| Comando | Esperado |
|---------|----------|
| `ls ~/.claude/skills \| wc -l` | `57` (38 + 19) — mais quaisquer skills locais do operador |
| `claude plugin list \| grep superpowers` | `superpowers@superpowers-marketplace` |
| `claude plugin list \| grep claude-mem` | `claude-mem@thedotmack` |
| `specify version` | versão do spec-kit |
| `npx skills list` | lista as skills instaladas |
| `jq empty ~/.claude/settings.json && echo ok` | `ok` |

Em nova sessão: skills disponíveis via `/` (ex.: `/domain-driven-design`, `/cqrs-implementation`)
e a mensagem do hook exibida.

---

## Resources: catálogo de skills

Skills deste repo (19). Nomes = comando de invocação (`/nome`).

| Skill | Grupo | Trigger |
|-------|-------|---------|
| `domain-driven-design` | design | Bounded contexts, aggregates, ubiquitous language, context mapping |
| `clean-architecture` | design | Dependency Rule, ports/adapters, onion, SOLID |
| `system-design` | design | Sistemas escaláveis: LB, cache, filas, particionamento |
| `refactoring-patterns` | craft | Transformações seguras dirigidas por smells (Fowler) |
| `working-with-legacy-code` | craft | Seams, characterization tests, quebra de dependências (Feathers) |
| `clean-code` | craft | Naming, funções pequenas, SRP, disciplina de comentários |
| `api-design-principles` | backend | Design/review REST & GraphQL |
| `cqrs-implementation` | backend | Separação read/write, event-sourced |
| `event-store-design` | backend | Event stores para sistemas event-sourced |
| `microservices-patterns` | backend | Fronteiras de serviço, event-driven, resiliência |
| `saga-orchestration` | backend | Transações distribuídas / compensação |
| `postgresql-table-design` | backend | Schema PostgreSQL: tipos, índices, constraints, performance |
| `ci-cd-pipelines` | entrega | Design/automação de pipelines |
| `security-hardening` | entrega | Hardening de app/infra |
| `release-it` | entrega | Circuit breakers, bulkheads, timeouts, chaos, observabilidade |
| `tech-debt-tracker` | entrega | Identificar, priorizar e rastrear dívida técnica |
| `migration-architect` | entrega | Migração de risco com rollback explícito (zero-downtime) |
| `impeccable` | frontend | UI/frontend: shape/audit/critique/polish; design system, a11y, anti-genérico |
| `task-observer` | meta | Observa sessões → propõe melhorias/novas skills (revisar antes de aplicar) |

Fontes e licenças por skill: [ATTRIBUTION.md](./ATTRIBUTION.md). Complementam a base do
mattpocock sem duplicar (por isso não trazem `tdd`, `code-review`, `research`,
`domain-modeling`, etc., que já vêm de lá).

---

## Manutenção

| Operação | Comando |
|----------|---------|
| Atualizar skills | `npx skills@latest update --global -y` |
| Atualizar superpowers | `claude plugin marketplace update superpowers-marketplace` |
| Atualizar claude-mem | `claude plugin marketplace update thedotmack` |
| Atualizar spec-kit | `uv tool upgrade specify-cli` |
| Remover skills | `npx skills@latest remove --global --skill '*' -y` |
| Remover superpowers | `claude plugin uninstall superpowers@superpowers-marketplace` |
| Remover claude-mem | `claude plugin uninstall claude-mem@thedotmack` |
| Remover spec-kit | `uv tool uninstall specify-cli` |

---

## Troubleshooting

| Sintoma | Causa provável | Ação |
|---------|----------------|------|
| Skills não aparecem via `/` | Sessão anterior ao install | Abrir nova sessão |
| Hook não dispara | Watcher não recarregou a config | Abrir `/hooks` uma vez, ou reiniciar |
| `specify: command not found` | `~/.local/bin` fora do `PATH` | Adicionar ao `PATH` no shell rc |
| `settings.json` ignorado | JSON malformado | `jq empty ~/.claude/settings.json` e corrigir |
| Skill duplicada | Mesmo nome em duas fontes | Remover uma; este repo já é deduplicado vs mattpocock |

---

## Referências

Todas as fontes abaixo foram **verificadas** (título, autores e data conferidos na origem).
Nenhuma afirmação da seção [Por que este stack](#por-que-este-stack) é inventada — cada
ponto rastreia até um destes itens.

### Papers acadêmicos (peer-reviewed / arXiv)

- Tang, N. et al. (2026). **How Coding Agents Fail Their Users: A Large-Scale Analysis of
  Developer-Agent Misalignment in 20,574 Real-World Sessions.** arXiv:2605.29442 —
  https://arxiv.org/abs/2605.29442
  *(evidência do modo de falha dominante: misalignment; >91% das resoluções exigem correção humana.)*
- Grabowski, H. (2026). **The Spec Growth Engine: Spec-Anchored, Code-Coupled, Drift-Enforced
  Architecture for AI-Assisted Software Development.** arXiv:2606.27045 —
  https://arxiv.org/abs/2606.27045
  *(context explosion e spec-code drift; detecção automática de divergência.)*
- Piskala, D. B. (2026). **Spec-Driven Development: From Code to Contract in the Age of AI
  Coding Assistants.** arXiv:2602.00180 — https://arxiv.org/abs/2602.00180
  *(spec como artefato primário; guia para praticantes.)*
- Taghavi, P.; Bhavani, S. (2026). **Spec Kit Agents: Context-Grounded Agentic Workflows.**
  arXiv:2604.05278 — https://arxiv.org/abs/2604.05278
  *(grounding por contexto de repositório melhora qualidade mantendo compatibilidade de testes.)*
- Özkan, O.; Babur, Ö.; van den Brand, M. (2023–2025). **Domain-Driven Design in Software
  Development: A Systematic Literature Review on Implementation, Challenges, and
  Effectiveness.** arXiv:2310.01905 (também em *Journal of Systems and Software*, 2025) —
  https://arxiv.org/abs/2310.01905
  *(revisão sistemática de 36 estudos; DDD melhora decomposição e coesão.)*
- **Domain-Driven Design for Microservices: An Evidence-Based Investigation.** *IEEE
  Transactions on Software Engineering* (2024) —
  https://dl.acm.org/doi/abs/10.1109/TSE.2024.3385835
  *(métodos DDD produzem serviços com maior coesão vs. dataflow-driven.)*

### Estudos de indústria e relatórios

- Anthropic Engineering. **Equipping agents for the real world with Agent Skills.** —
  https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
  *(fundamento do progressive disclosure: contexto sob demanda, skills ~ilimitadas.)*
- Forsgren, N.; Humble, J.; Kim, G. **Accelerate / DORA — State of DevOps.** —
  https://dora.dev/capabilities/
  *(evidência empírica: automação de testes e CD elevam qualidade e throughput.)*
- GitHub. **spec-kit — Toolkit for Spec-Driven Development.** —
  https://github.com/github/spec-kit · guia: https://github.github.com/spec-kit/
- IBM. **What is Spec-Driven Development?** —
  https://www.ibm.com/think/topics/spec-driven-development

### Casos de negócio para estudo

- **GitHub Spec Kit** na prática (blog oficial GitHub): *Spec-driven development with AI —
  get started with a new open source toolkit.* —
  https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/
- **Microsoft for Developers**: *Diving Into Spec-Driven Development With GitHub Spec Kit.* —
  https://developer.microsoft.com/blog/spec-driven-development-spec-kit
- **DDD Sample (Cargo Tracking System)** — a aplicação de referência do livro de Evans, usada
  em vários estudos comparativos de decomposição. Ver a SLR (Özkan et al.) para os 36 casos
  revisados.

### Livros fundadores (base intelectual das skills deste repo)

- Evans, E. (2003). *Domain-Driven Design.* — base de `domain-driven-design`.
- Fowler, M. (2018). *Refactoring, 2ª ed.* — base de `refactoring-patterns`.
- Feathers, M. (2004). *Working Effectively with Legacy Code.* — base de `working-with-legacy-code`.
- Martin, R. C. (2017). *Clean Architecture.* / (2008) *Clean Code.* — base de `clean-architecture`, `clean-code`.
- Nygard, M. (2018). *Release It!, 2ª ed.* — base de `release-it`.
- Kleppmann, M. (2017). *Designing Data-Intensive Applications.* — base de `system-design`.
- Beck, K. (2002). *Test-Driven Development by Example.* — base do fluxo TDD.

---

## Sobre o André

André Franciny é Gerente de Receitas no Templo. Trabalha com IA e automação para tirar a
operação de receita do trabalho manual: roteamento de leads, higiene de pipeline, debriefs de
reunião, follow-up e dashboards executivos. Este stack de skills faz parte do método —
transformar processo ad-hoc em um fluxo com especificação, design explícito e verificação,
cada etapa com um artefato que a próxima consome.

- LinkedIn: [andre-franciny-br](https://www.linkedin.com/in/andre-franciny-br/)
- Instagram: [andre.franciny](https://www.instagram.com/andre.franciny)

---

## Licença

Curadoria/organização: MIT ([LICENSE](./LICENSE)). Cada skill vendorizada mantém a licença
de origem — ver [ATTRIBUTION.md](./ATTRIBUTION.md) e [`licenses/`](./licenses/).
