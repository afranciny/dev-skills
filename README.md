# dev-skills

> Set curado de agent skills de engenharia para o Claude Code + referência de setup do stack completo.

`skills: 17` · `stack: 4 componentes` · `scope: global` · `license: MIT`

Referência técnica para instalar e operar o stack de skills: `mattpocock/skills` (base),
`dev-skills` (este repo), `superpowers` (plugin) e `spec-kit` (CLI). Formato de referência
— cada operação traz assinatura, parâmetros, retorno e exemplo.

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
- [Pré-requisitos](#pré-requisitos)
- [Operações de instalação](#operações-de-instalação)
  - [`install mattpocock`](#install-mattpocock)
  - [`install dev-skills`](#install-dev-skills)
  - [`install superpowers`](#install-superpowers)
  - [`install spec-kit`](#install-spec-kit)
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
| `dev-skills` | skills (17) | `~/.claude/skills/*` | `/nome` · auto | `npx skills` |
| `superpowers` | plugin | `~/.claude/plugins/*` | auto por sessão | plugin marketplace |
| `spec-kit` | CLI | `~/.local/bin/specify` | `specify <cmd>` | `uv tool` |

**Guia de uso** (qual skill em cada fase): `https://claude.ai/code/artifact/793aa989-fe42-48af-9a24-396df46c988d`

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

## Operações de instalação

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

Este repo — 17 skills net-new (DDD, arquitetura, backend, entrega) que complementam a base.

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

**Retorno** — 17 diretórios de skill disponíveis em `~/.claude/skills/`.

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
echo '{"systemMessage":"Você tem 55 skills + superpowers + spec-kit. Guia: https://claude.ai/code/artifact/793aa989-fe42-48af-9a24-396df46c988d — acione /ask-matt se estiver em dúvida.","hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":"O usuário mantém um set curado de skills de engenharia. Prefira acionar a skill adequada à fase do trabalho."}}'
```

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
| `ls ~/.claude/skills \| wc -l` | `55` (38 + 17) |
| `claude plugin list \| grep superpowers` | `superpowers@superpowers-marketplace` |
| `specify version` | versão do spec-kit |
| `jq empty ~/.claude/settings.json && echo ok` | `ok` |

Em nova sessão: skills disponíveis via `/` (ex.: `/domain-driven-design`, `/cqrs-implementation`)
e a mensagem do hook exibida.

---

## Resources: catálogo de skills

Skills deste repo (17). Nomes = comando de invocação (`/nome`).

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

Fontes e licenças por skill: [ATTRIBUTION.md](./ATTRIBUTION.md). Complementam a base do
mattpocock sem duplicar (por isso não trazem `tdd`, `code-review`, `research`,
`domain-modeling`, etc., que já vêm de lá).

---

## Manutenção

| Operação | Comando |
|----------|---------|
| Atualizar skills | `npx skills@latest update --global -y` |
| Atualizar superpowers | `claude plugin marketplace update superpowers-marketplace` |
| Atualizar spec-kit | `uv tool upgrade specify-cli` |
| Remover skills | `npx skills@latest remove --global --skill '*' -y` |
| Remover superpowers | `claude plugin uninstall superpowers@superpowers-marketplace` |
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

## Licença

Curadoria/organização: MIT ([LICENSE](./LICENSE)). Cada skill vendorizada mantém a licença
de origem — ver [ATTRIBUTION.md](./ATTRIBUTION.md) e [`licenses/`](./licenses/).
