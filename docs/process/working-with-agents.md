---
last-reviewed: 2026-09-30
owner: ""
---

# Working with AI agents

This page explains how agent guidance is organized in this repository, how to work with agents day to day, and how to keep the guidance useful.
The reasoning is in [ADR-0003](../decisions/0003-use-agents-md-as-canonical-agent-guidance.md).

## What agents read

```mermaid
flowchart TD
    agents[AGENTS.md<br/>canonical, always loaded] --> nested[docs/AGENTS.md<br/>openspec/AGENTS.md<br/>loaded when working there]
    claude[CLAUDE.md<br/>@AGENTS.md + Claude notes] -.imports.-> agents
    gemini[.gemini/settings.json<br/>context.fileName] -.points to.-> agents
    agents --> truth[Sources of truth<br/>specs, decisions, glossary,<br/>architecture, principles]
    skills[Skills<br/>.agents/skills → .claude/skills<br/>loaded on demand] --> truth
    config[openspec/config.yaml<br/>context + rules injected<br/>into OpenSpec workflows] --> truth
```

| Layer | File | Loaded | Put here |
| --- | --- | --- | --- |
| Root guide | `AGENTS.md` | Every session | Overview, sources of truth, exact commands, workflow, boundaries, repo map |
| Nested guides | `docs/AGENTS.md`, `openspec/AGENTS.md` | When the agent works in that folder | Rules that only apply there |
| Tool adapters | `CLAUDE.md` (+ one-line `CLAUDE.md` next to each nested `AGENTS.md`), `.gemini/settings.json` | By that tool | Imports or pointers only, plus genuinely tool-specific notes |
| Skills | `.agents/skills/<name>/SKILL.md` (copied to `.claude/skills/`) | When the task matches the skill's `description` | Step-by-step procedures: recording a decision, maintaining docs |
| OpenSpec integration | `openspec/config.yaml`, generated `openspec-*` skills and `/opsx:*` commands | During OpenSpec workflows | Project context and per-artifact rules for proposals, specs, designs, tasks |
| Subagents | `.claude/agents/*.md` | When delegated to | Focused reviewers, e.g. `docs-reviewer` |
| Settings | `.claude/settings.json` | By Claude Code | Pre-approved safe commands |

## Day-to-day use

| You want to... | Ask the agent / run |
| --- | --- |
| Think through an idea without writing code | `/opsx:explore` |
| Turn an idea into a proposal with specs, design and tasks | `/opsx:propose "<what and why>"` |
| Record a decision | "Record a decision about X". Triggers the `record-decision` skill |
| Implement an agreed change | `/opsx:apply <change-id>` |
| Adjust a change after feedback | `/opsx:update <change-id>` |
| Check docs are in sync before a PR | "Review docs drift", `docs-reviewer` subagent, `maintain-docs` skill |
| Finish a change | `npm run check`, then `/opsx:archive <change-id>` |

Command names differ per tool: `/opsx:propose` in Claude Code, `/opsx-propose` in Cursor and Copilot, `$openspec-propose` in Codex.
OpenSpec's optional extra workflows (`/opsx:verify`, `/opsx:onboard`, `/opsx:ff` and others) can be enabled on your machine with `npx openspec config profile`, then `npm run agents:update`.

Good habits:

- **Start each task in a fresh session**, pointing at the change id. Long sessions accumulate stale context.
- **Plan before code** for anything non-trivial: proposal first, implementation after review.
- **Give the agent a way to verify its work.** The agent should run `npm run check` and the project tests, not only you.
- **Review agent output like a colleague's PR.** Agents may propose ADRs but never accept them on their own.

## Adding another AI tool

1. Check whether the tool reads `AGENTS.md` natively. Most do. If so, nothing else is needed for guidance.
2. If it doesn't, add the thinnest possible adapter that imports or points to `AGENTS.md`, and list it in the "What agents read" table above.
3. Generate OpenSpec commands for it: `npx openspec init --tools <tool-id>` (see `npx openspec init --help`), then add the generated folders to the `ignores` in `.markdownlint-cli2.jsonc` if needed.
4. If the tool does not read `.agents/skills/`, add its skill folder to `SKILLS_COPIES` in `scripts/docs.mjs`.

## Keeping guidance useful

Research on agent context files is sobering. Long, generic or LLM-generated instruction files often *lower* agent success rates and raise cost. Short, human-written, specific guidance works best.

- **Every line must prevent a real mistake.** Ask: "Would removing this line make the agent do something wrong?" If not, delete it.
- **Grow from failures.** When an agent makes the same mistake twice, add one precise line to the nearest `AGENTS.md` or skill, in the same PR as the fix.
- **Commands, not advice.** Exact commands with flags beat "make sure tests pass".
- **Link, don't copy.** Point to specs, ADRs and docs instead of summarizing them.
- **Budget.** Root `AGENTS.md` under about 150 lines; a skill's `SKILL.md` under 500 lines; the whole chain of `AGENTS.md` files under 32 KiB (the Codex default limit).
- **Checks over prose.** If a rule must always hold, add it to `npm run check` (`scripts/docs.mjs`) or CI and shorten the prose.
- **Review quarterly.** This page and the `AGENTS.md` files are living documents. Prune rules that no longer fire.

## Updating generated agent files

The OpenSpec CLI version is pinned in `package.json`. To upgrade:

1. `npm install -D @fission-ai/openspec@<version>`
2. `npm run agents:update` (runs `openspec update` to regenerate `openspec-*` skills and `/opsx` commands)
3. Review the diff, run `npm run check`, and commit. Never hand-edit the generated files.
