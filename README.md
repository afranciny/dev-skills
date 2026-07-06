# dev-skills

Um set único e curado de **agent skills de engenharia** para o Claude Code (e outros
agentes compatíveis com skills.sh). Concatena, deduplicado, o melhor de várias fontes
open-source, e **complementa** o set `mattpocock/skills` já instalado globalmente —
sem sobreposição.

Cobre o que faltava ao redor de um fluxo **spec-driven + DDD + TDD**: design
estratégico, arquitetura, padrões de backend, e entrega/manutenção.

## Skills incluídas (17)

**Design estratégico & DDD**
- `domain-driven-design` — bounded contexts, aggregates, ubiquitous language, context mapping.
- `clean-architecture` — Dependency Rule, ports/adapters (hexagonal), onion, SOLID.
- `system-design` — sistemas escaláveis: load balancing, cache, filas, particionamento.

**Craftsmanship**
- `refactoring-patterns` — transformações seguras dirigidas por smells (Fowler).
- `working-with-legacy-code` — seams, characterization tests, quebra de dependências (Feathers).
- `clean-code` — naming, funções pequenas, SRP, disciplina de comentários/erros.

**Padrões de backend**
- `api-design-principles` — design/review REST & GraphQL.
- `cqrs-implementation` — separação read/write, sistemas event-sourced.
- `event-store-design` — event stores para sistemas event-sourced.
- `microservices-patterns` — fronteiras de serviço, comunicação event-driven, resiliência.
- `saga-orchestration` — transações distribuídas / ações compensatórias.
- `postgresql-table-design` — schema PostgreSQL: tipos, índices, constraints, performance.

**Entrega & manutenção**
- `ci-cd-pipelines` — design/automação de pipelines.
- `security-hardening` — hardening de app/infra.
- `release-it` — circuit breakers, bulkheads, timeouts, chaos, observabilidade.
- `tech-debt-tracker` — identificar, priorizar e rastrear dívida técnica.
- `migration-architect` — planejar migrações grandes de código/dados (zero-downtime).

Fontes e licenças: ver [ATTRIBUTION.md](./ATTRIBUTION.md).

## Instalação

**Local (symlink, imediato)** — cada skill é linkada em `~/.claude/skills/`:

```bash
for d in ~/dev-skills/*/; do
  [ -f "$d/SKILL.md" ] && ln -sfn "$d" "$HOME/.claude/skills/$(basename "$d")"
done
```

**Via skills.sh (se publicar em GitHub, ex. `afranciny/dev-skills`)**:

```bash
npx skills add afranciny/dev-skills --global --skill '*' --agent '*' -y
```

## Companheiros nativos (não incluídos aqui de propósito)

Estes rendem mais instalados no formato nativo — instale sob demanda:

- **spec-driven development** (GitHub Spec-Kit):
  ```bash
  uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
  specify init <projeto> --integration claude
  ```
- **superpowers** (disciplina de plan/TDD/review):
  ```
  /plugin marketplace add obra/superpowers-marketplace
  /plugin install superpowers@superpowers-marketplace
  ```
- **cc-sdd** (SDD skills-native, alternativa ao spec-kit): `npx cc-sdd@latest`

## Licença

Cada skill mantém a licença do seu projeto de origem (ver `ATTRIBUTION.md` e `licenses/`).
A curadoria/organização deste repo é MIT (`LICENSE`).
