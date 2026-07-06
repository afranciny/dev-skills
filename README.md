# dev-skills

Set único e curado de **agent skills de engenharia** para o Claude Code, mais o guia
de instalação do stack completo de produtividade em código: as skills do
`mattpocock/skills`, este set (`dev-skills`), o plugin `superpowers` e o CLI
`spec-kit`.

Este README ensina a **reproduzir o ambiente do zero** em qualquer máquina.

- [O que é o stack](#o-que-é-o-stack)
- [Pré-requisitos](#pré-requisitos)
- [Instalação passo a passo](#instalação-passo-a-passo)
- [Configurar a mensagem inicial (hook)](#configurar-a-mensagem-inicial-hook)
- [Setup por projeto](#setup-por-projeto)
- [Verificação](#verificação)
- [Skills incluídas neste repo](#skills-incluídas-neste-repo)
- [Atualizar e remover](#atualizar-e-remover)

---

## O que é o stack

| Camada | O que é | Como instala | Formato |
|--------|---------|--------------|---------|
| **mattpocock/skills** | 38 skills de processo (spec, review, TDD, triagem, domínio, escrita) | `npx skills` | skills globais |
| **dev-skills** (este repo) | 17 skills net-new (DDD, arquitetura, backend, entrega) | `npx skills` ou symlink | skills globais |
| **superpowers** | Disciplina brainstorm → plan → TDD → review | plugin marketplace | plugin |
| **spec-kit** | Fluxo Spec → Plan → Tasks → Implement por projeto | `uv tool install` | CLI (`specify`) |

As duas primeiras camadas são **skills** (invocáveis com `/nome`, muitas disparam
sozinhas). O superpowers é um **plugin** (carrega a cada sessão). O spec-kit é um
**CLI** que você roda por projeto.

Guia visual de qual skill usar em cada fase:
**https://claude.ai/code/artifact/793aa989-fe42-48af-9a24-396df46c988d**

---

## Pré-requisitos

- **Claude Code** instalado (`claude --version`).
- **Node.js 18+** (para o `npx skills`). `node -v`.
- **git** e, para o spec-kit, o **uv** (instalado no passo 4).
- Opcional: **`gh` CLI** autenticado, se for usar tracker no GitHub Issues.

---

## Instalação passo a passo

### 1. Skills do mattpocock (base, 38 skills)

Instala globalmente para o Claude Code, de forma não-interativa:

```bash
npx skills@latest add mattpocock/skills --global --agent claude-code --skill '*' -y
```

O instalador detecta o agente `claude-code` sozinho. As skills vão para
`~/.claude/skills/` (via symlink de `~/.agents/skills/`).

### 2. dev-skills (este repo, 17 skills net-new)

```bash
npx skills@latest add afranciny/dev-skills --global --agent claude-code --skill '*' -y
```

Alternativa sem `npx` (clonar e linkar manualmente):

```bash
git clone https://github.com/afranciny/dev-skills.git ~/dev-skills
for d in ~/dev-skills/*/; do
  [ -f "$d/SKILL.md" ] && ln -sfn "$d" "$HOME/.claude/skills/$(basename "$d")"
done
```

### 3. superpowers (plugin)

```bash
claude plugin marketplace add obra/superpowers-marketplace
claude plugin install superpowers@superpowers-marketplace --scope user
```

Dentro do Claude Code os mesmos passos são `/plugin marketplace add obra/superpowers-marketplace`
e `/plugin install superpowers@superpowers-marketplace`.

### 4. spec-kit (CLI `specify`)

Precisa do `uv` (gerenciador Python da Astral):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh          # instala uv
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

Isso deixa o executável `specify` em `~/.local/bin/` (garanta que está no `PATH`).

---

## Configurar a mensagem inicial (hook)

Um hook `SessionStart` no `~/.claude/settings.json` imprime, a cada sessão, um
lembrete de usar as skills com o link do guia. Adicione o bloco `hooks` (mesclando
com o que já existir no arquivo):

```jsonc
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"systemMessage\":\"Você tem 55 skills + superpowers + spec-kit. Guia: https://claude.ai/code/artifact/793aa989-fe42-48af-9a24-396df46c988d — acione /ask-matt se estiver em dúvida.\",\"hookSpecificOutput\":{\"hookEventName\":\"SessionStart\",\"additionalContext\":\"O usuário mantém um set curado de skills de engenharia. Prefira acionar a skill adequada à fase do trabalho.\"}}'"
          }
        ]
      }
    ]
  }
}
```

- `systemMessage` é o texto que **você** vê ao abrir a sessão.
- `additionalContext` é injetado no **modelo**, para ele lembrar de acionar as skills.

Valide depois de editar: `jq empty ~/.claude/settings.json` (sem saída = JSON ok).
O hook passa a valer numa nova sessão — se não aparecer, abra `/hooks` uma vez para
recarregar a config.

---

## Setup por projeto

As skills do mattpocock assumem uma config por repositório (issue tracker, labels de
triagem, layout de docs de domínio). Rode **uma vez dentro do projeto**:

```
/setup-matt-pocock-skills
```

Ele pergunta o issue tracker (GitHub, GitLab, Linear ou arquivos locais), as labels de
triagem e onde salvar docs, e grava `docs/agents/*.md` + um bloco `## Agent skills`
no `CLAUDE.md`.

Para começar uma feature nova com spec-driven development:

```bash
specify init <projeto> --integration claude
# ou, para instalar como skills em vez de slash commands:
specify init <projeto> --integration claude --integration-options="--skills"
```

---

## Verificação

```bash
ls ~/.claude/skills | wc -l                 # deve listar 55 (38 + 17)
claude plugin list | grep superpowers       # plugin ativo
specify version                             # CLI do spec-kit
jq empty ~/.claude/settings.json && echo ok # settings íntegro
```

Numa nova sessão do Claude Code, as skills aparecem via `/` (ex.: `/domain-driven-design`,
`/code-review`, `/cqrs-implementation`) e a mensagem inicial do hook é exibida.

---

## Skills incluídas neste repo

**Design estratégico & DDD** — `domain-driven-design`, `clean-architecture`, `system-design`
**Craftsmanship** — `refactoring-patterns`, `working-with-legacy-code`, `clean-code`
**Padrões de backend** — `api-design-principles`, `cqrs-implementation`, `event-store-design`, `microservices-patterns`, `saga-orchestration`, `postgresql-table-design`
**Entrega & manutenção** — `ci-cd-pipelines`, `security-hardening`, `release-it`, `tech-debt-tracker`, `migration-architect`

Fontes e licenças de cada skill: ver [ATTRIBUTION.md](./ATTRIBUTION.md). Elas
**complementam** as 38 do mattpocock, sem duplicar (por isso não incluem tdd,
code-review, research, domain-modeling, etc., que já vêm de lá).

---

## Atualizar e remover

**Atualizar as skills** (mattpocock e dev-skills):

```bash
npx skills@latest update --global -y
```

**Atualizar o superpowers**: `claude plugin marketplace update superpowers-marketplace`
**Atualizar o spec-kit**: `uv tool upgrade specify-cli`

**Remover**:

```bash
npx skills@latest remove --global --skill '*' -y      # skills
claude plugin uninstall superpowers@superpowers-marketplace
uv tool uninstall specify-cli
```

---

## Licença

A curadoria/organização deste repo é MIT ([LICENSE](./LICENSE)). Cada skill vendorizada
mantém a licença do projeto de origem — ver [ATTRIBUTION.md](./ATTRIBUTION.md) e
[`licenses/`](./licenses/).
