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

## Not vendored (referenced only)

These were intentionally excluded from vendoring:

- **github/spec-kit** — install natively as the `specify` CLI (spec-driven workflow SSOT).
- **obra/superpowers** — install natively as a plugin (plan/TDD/review discipline).
- **gotalab/cc-sdd** — optional skills-native SDD system via `npx cc-sdd`.
- **SpillwaveSolutions/sdd-skill**, **ruvnet/agentic-flow** — no upstream license; not redistributable.
- Any skill that duplicates the `mattpocock/skills` set already installed globally
  (tdd, code-review, research, diagnosing-bugs, domain-modeling, etc.).
