@AGENTS.md

## Claude Code

- OpenSpec slash commands: `/opsx:explore`, `/opsx:propose`, `/opsx:apply`, `/opsx:update`, `/opsx:sync`, `/opsx:archive`. They are generated; refresh them with `npm run agents:update`.
- Project skills: `record-decision`, `maintain-docs`. They are copies from `.agents/skills/`: edit them there, then run `npm run skills:sync`.
- Before finishing a change, delegate a drift review to the `docs-reviewer` subagent.
- Personal, uncommitted notes go in `CLAUDE.local.md`.
