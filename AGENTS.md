# AGENTS.md

This repo develops the template in `template/`. Work by the template's own process (`template/docs/agents/`), using the commands and paths below.

## Commands

| Purpose | Command | Notes |
|---|---|---|
| Install | none | Needs `bash` and `python3` |
| Check | `./scripts/check` | Framework self-check. Must be green before any handoff |
| Run a role | `reference/run-role/run-role <role> <pr-or-issue>` | Fresh headless session; see `reference/README.md` |
| Verify | none | No app to drive. A docs change follows "No runtime behaviour" in `template/.agents/skills/verify/SKILL.md`; a `scripts/` change proves itself with a regression case in `scripts/check` |

## Hard rules

1. **Evidence, not claims.** "Done" means `check` is green and the proof named in the task's verification plan ran and matched the acceptance criteria. If that proof could not run, the result is `UNVERIFIED`, which never counts as PASS.
2. **Fix upward.** When you correct a mistake or work around one, log it in `docs/garden.md` with the highest rung of the trust ladder that could prevent it. See `template/docs/agents/trust-ladder.md`.
3. **No workaround comments.** Do not leave a comment explaining why the real problem was not solved. Fix it, or log it in the garden and say so in your report.
4. **One blessed way.** Extend the existing pattern. If two patterns exist for the same thing, ask.
5. **Use the tool, don't rewrite it.** If a script or CLI exists for a mechanical job, call it. If you write the same throwaway script twice, propose it as a tool.
6. **One-way doors need a human.** Changes that touch the paths below never merge without human approval, whatever the autonomy level.

## One-way doors

- `AGENTS.md`, `template/AGENTS.md`
- `template/docs/agents/process.md`, `template/docs/agents/trust-ladder.md`, `template/docs/agents/roles/**`
- `scripts/check`, `.github/workflows/**`

## Read when

- Running or taking part in the task loop → `template/docs/agents/process.md` and your role file in `template/docs/agents/roles/`
- Changing anything in `template/` → it must stay project-neutral: no filled-in commands, no garden entries, no harness or model named except as an example (#11)
