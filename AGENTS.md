# AGENTS.md

Guidance for AI coding agents and humans working in this repository. This file is canonical: `CLAUDE.md` imports it, and other tools read it directly.
Keep it short and link rather than copy. Every line should prevent a real mistake.

## Project overview

<!-- Replace this paragraph: what the product is, who it is for, the one or two things that make it hard. -->
A long-lived project run with spec-driven development. Behavior is specified in OpenSpec and decisions are recorded as ADRs.
**Status:** no implementation yet. The implementation language is **not decided** ([ADR-0005](docs/decisions/0005-choose-primary-implementation-language.md), Java is the leading candidate).

## Sources of truth

Read the relevant ones before proposing or changing anything:

| Question | Where |
| --- | --- |
| What must the system do? | `openspec/specs/<capability>/spec.md` (`npx openspec list --specs`) |
| What is being changed right now? | `openspec/changes/<change-id>/` (`npx openspec list`) |
| What was decided, and what is still open? | [Decision log](docs/decisions/README.md). Accepted ADRs are binding; `proposed` ones are open questions |
| What guides trade-offs? | [Principles](docs/principles.md) |
| How is it built now? | [Architecture](docs/architecture/README.md) |
| What does this term mean? | [Glossary](docs/glossary.md) |
| How do we work? | [Development workflow](docs/process/development-workflow.md), [documentation lifecycle](docs/process/documentation-lifecycle.md) |

## Commands

Documentation and spec tooling (Node.js 20.19 or later):

- `npm ci`: install tooling
- `npm run check`: validate specs, lint Markdown, check decisions, links and review dates. **Run it before you say a task is done.**
- `npm run docs:lint:fix`: auto-fix Markdown style
- `npm run adr:new -- "Title"`: create the next ADR (status `proposed`) and update the decision log
- `npm run docs:index`: regenerate the decision log after editing an ADR's title, status or date
- `npm run skills:sync`: copy skills from `.agents/skills/` to `.claude/skills/` after editing them
- `npx openspec list [--specs]`, `npx openspec show <id>`, `npx openspec validate <id> --strict`, `npx openspec archive <id> -y`
- The OpenSpec skills call `openspec` directly. If it is not on `PATH`, run the same command with `npx openspec`.

Application build and test: **none yet**. Replace this line with the exact commands once ADR-0005 is accepted.

## Workflow

1. **Behavior change:** create an OpenSpec change (`/opsx:propose` or `npx openspec new change <verb-first-id>`). Never edit `openspec/specs/` directly.
2. **Costly-to-reverse decision:** `npm run adr:new -- "Title"` and link it from the change's `design.md`. See the `record-decision` skill.
3. **Implement:** `/opsx:apply`, and tick `tasks.md` as you go. If the spec turns out to be wrong, fix the delta spec first.
4. **Document:** update affected docs in the same change and bump `last-reviewed`. See the `maintain-docs` skill.
5. **Finish:** `npm run check`, then `/opsx:archive` so the specs reflect the new behavior.

Bug fixes that restore specified behavior and pure refactors need no spec change. Full details: [development workflow](docs/process/development-workflow.md).

## Boundaries

### Always

- Check the decision log before proposing a design; follow accepted ADRs and name the ones you rely on.
- Use glossary terms verbatim; add new domain terms to the glossary in the same change.
- Use relative Markdown links (`[text](../path.md)`); never `[[wikilinks]]`.
- State assumptions explicitly in the proposal or design instead of silently picking one.

### Ask first

- Adding dependencies, build files, source trees or CI jobs. The stack is undecided (ADR-0005).
- Anything that contradicts an accepted ADR. Propose a superseding ADR instead of working around it.
- Changing `AGENTS.md` boundaries, `.claude/settings.json`, `openspec/config.yaml` or CI workflows.
- Deleting or renaming specs, ADRs or docs.

### Never

- Edit `openspec/specs/**` by hand, or rewrite archived changes (`openspec/changes/archive/**`).
- Rewrite the decision of an accepted ADR. Supersede it with a new one.
- Hand-edit generated content: the decision log block, `openspec-*` skills, `.claude/commands/opsx/`, or `.claude/skills/` copies of skills that live in `.agents/skills/`.
- Commit secrets, credentials or personal data.

## Repository map

```text
AGENTS.md, CLAUDE.md     agent guidance (this file is canonical)
openspec/                specs (current behavior) and changes (proposals + archive); see openspec/AGENTS.md
docs/                    decisions, architecture, guides, reference, process, research, templates; see docs/AGENTS.md
.agents/skills/          agent skills (canonical; Codex, Gemini CLI, Cursor, Copilot)
.claude/                 Claude Code: skills (synced copies), OpenSpec commands, subagents, settings
.github/                 PR/issue templates, CI
scripts/docs.mjs         docs tooling behind the npm scripts
```

## Maintaining this guidance

- An agent made the same mistake twice? Add one line here, in the nearest nested `AGENTS.md`, or in a skill. Do not add general advice.
- Nested guides: [`docs/AGENTS.md`](docs/AGENTS.md) and [`openspec/AGENTS.md`](openspec/AGENTS.md). The closest one to the edited file wins.
- Keep this file under about 150 lines. Move detail into `docs/` or a skill and link it.
- How the agent setup works: [working with agents](docs/process/working-with-agents.md).
