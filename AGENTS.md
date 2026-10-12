# AGENTS.md

Always loaded. Keep it short. Detail lives in `docs/agents/`; read it when the condition says so.

## Commands

Fill these in per project. If a command is missing, say so. Do not invent one.

| Purpose | Command | Notes |
|---|---|---|
| Install | `TODO` | |
| Check (lint + typecheck + unit tests) | `TODO` | Must be green before any handoff |
| Verify (drive the real app) | `.agents/skills/verify/verify <cmd>` | See the `verify` skill |

## Hard rules

1. **Evidence, not claims.** "Done" means `check` is green and there is verification evidence that matches the task's acceptance criteria. If `verify` is not configured, the result is `UNVERIFIED`, which never counts as PASS.
2. **Fix upward.** When you correct a mistake or work around one, log it in `docs/agents/garden.md` with the highest rung of the trust ladder that could prevent it. See `docs/agents/trust-ladder.md`.
3. **No workaround comments.** Do not leave a comment explaining why the real problem was not solved. Fix it, or log it in the garden and say so in your report.
4. **One blessed way.** Extend the existing pattern. If two patterns exist for the same thing, use the one named in `docs/agents/invariants.md`, or ask.
5. **Use the tool, don't rewrite it.** If a script or CLI exists for a mechanical job, call it. If you write the same throwaway script twice, propose it as a tool.
6. **One-way doors need a human.** Changes that touch the paths below never merge without human approval, whatever the autonomy level.

## One-way doors

<!-- Globs. Examples: migrations/**, auth/**, billing/**, infra/**, anything that deletes user data. -->
- `TODO`

## Read when

- Running or taking part in the task loop → `docs/agents/process.md` and your role file in `docs/agents/roles/`
- Correcting an agent or fixing a recurring mistake → `docs/agents/trust-ladder.md`
- Adding or changing a rule the code must never break → `docs/agents/invariants.md`
- Proving a change works → the `verify` skill (`.agents/skills/verify/SKILL.md`)
