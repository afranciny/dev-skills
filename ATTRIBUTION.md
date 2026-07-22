# Attribution

This repository vendors (copies) agent skills from upstream open-source projects.
Each skill below retains its original upstream license; the full upstream license
texts are preserved under [`licenses/`](./licenses/).

No modifications were made to the vendored `SKILL.md` content unless noted.

| Skill | Upstream repo | License | License file |
|-------|---------------|---------|--------------|
| domain-driven-design | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| clean-architecture | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| system-design | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| refactoring-patterns | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| working-with-legacy-code | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| clean-code | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| release-it | [wondelai/skills](https://github.com/wondelai/skills) | MIT © 2025 Wondel.ai sp. z o.o. | licenses/wondelai-skills.LICENSE |
| api-design-principles | [wshobson/agents](https://github.com/wshobson/agents) | MIT © 2024 Seth Hobson | licenses/wshobson-agents.LICENSE |
| cqrs-implementation | [wshobson/agents](https://github.com/wshobson/agents) | MIT © 2024 Seth Hobson | licenses/wshobson-agents.LICENSE |
| event-store-design | [wshobson/agents](https://github.com/wshobson/agents) | MIT © 2024 Seth Hobson | licenses/wshobson-agents.LICENSE |
| microservices-patterns | [wshobson/agents](https://github.com/wshobson/agents) | MIT © 2024 Seth Hobson | licenses/wshobson-agents.LICENSE |
| saga-orchestration | [wshobson/agents](https://github.com/wshobson/agents) | MIT © 2024 Seth Hobson | licenses/wshobson-agents.LICENSE |
| postgresql-table-design | [wshobson/agents](https://github.com/wshobson/agents) | MIT © 2024 Seth Hobson | licenses/wshobson-agents.LICENSE |
| ci-cd-pipelines | [rohitg00/awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) | Apache-2.0 | licenses/rohitg00-toolkit.LICENSE |
| security-hardening | [rohitg00/awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) | Apache-2.0 | licenses/rohitg00-toolkit.LICENSE |
| tech-debt-tracker | [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) | MIT © 2025 Alireza Rezvani | licenses/alirezarezvani-claude-skills.LICENSE |
| migration-architect | [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) | MIT © 2025 Alireza Rezvani | licenses/alirezarezvani-claude-skills.LICENSE |
| impeccable | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Apache-2.0 © Paul Bakaus | licenses/pbakaus-impeccable.LICENSE (+ `.NOTICE`) |
| task-observer | [rebelytics/one-skill-to-rule-them-all](https://github.com/rebelytics/one-skill-to-rule-them-all) | CC-BY-4.0 © Eoghan Henn / rebelytics | licenses/rebelytics-task-observer.LICENSE |

> **impeccable** inclui o `NOTICE` de origem (Apache-2.0 §4d), que credita conteúdo derivado de
> [ehmo/platform-design-skills](https://github.com/ehmo/platform-design-skills) (MIT) nos arquivos
> de referência de plataforma (iOS/Android). Ver `licenses/pbakaus-impeccable.NOTICE`.
> **task-observer** é CC-BY-4.0 — a atribuição a Eoghan Henn / rebelytics acima cumpre a licença.

## Not vendored (referenced only)

These were intentionally excluded from vendoring:

- **github/spec-kit** — install natively as the `specify` CLI (spec-driven workflow SSOT).
- **obra/superpowers** — install natively as a plugin (plan/TDD/review discipline).
- **thedotmack/claude-mem** — install natively as a plugin (cross-session memory worker + `mcp-search`).
- **vercel-labs/skills** — the `skills` CLI (`npx skills`), used to discover/install skills; nothing to vendor.
- **gotalab/cc-sdd** — optional skills-native SDD system via `npx cc-sdd`.
- **SpillwaveSolutions/sdd-skill**, **ruvnet/agentic-flow** — no upstream license; not redistributable.
- Any skill that duplicates the `mattpocock/skills` set already installed globally
  (tdd, code-review, research, diagnosing-bugs, domain-modeling, etc.).
