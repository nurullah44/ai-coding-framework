# Garden

Append-only buffer of corrections, workarounds, and weeds. The Gardener reviews it in batches and turns repeated entries into rung 1 or 2 fixes.

Status values: `open`, `promoted` (a fix landed; link it), `wontfix` (reason given).

| Date | Source | What happened | Target rung | Proposed fix | Status |
|---|---|---|---|---|---|
| | <!-- task id, review, transcript, human --> | | <!-- 1-4 --> | | open |
| 2026-10-13 | PR #9 review, round 1 | Docs pointed at paths that don't exist (`features/<area>.md` in the task template; the feature map lives under `.agents/skills/verify/features/`). Fixed by hand in PR #9. | 2 | A link and path check in the framework self-check (#5) | promoted: `scripts/check`, PR #13 |
| 2026-10-13 | Human + test | Claude Code silently skips `AGENTS.md` when a `CLAUDE.md` without `@AGENTS.md` exists (tested on 2.1.296: AGENTS.md alone loads; with a plain CLAUDE.md it doesn't). `/init` creates exactly that file. | 2 | Self-check (#5) fails if `CLAUDE.md` exists without `@AGENTS.md` | promoted: `scripts/check`, PR #13 |
| 2026-10-13 | PR #9–#14, harness runs | `claude -p --allowedTools X "prompt"`: the variadic flag swallows the prompt. | 1 | `run-role` sends Claude's prompt on stdin; tested | promoted: `reference/run-role`, PR #14 |
| 2026-10-13 | PR #13 review | `codex exec --sandbox read-only` can't write `/tmp`, so a spec lane couldn't run `check` and came back UNVERIFIED. | 1 | Spec and regression lanes run commands in a disposable worktree, and the run fails if it changed | promoted: `reference/run-role`, PR #14 |
| 2026-10-13 | PR #14 review | Claude's `--allowedTools` only auto-approves; a "read-only" session still had MCP tools that write to external services. | 1 | `run-role` sets `--tools` and `--strict-mcp-config`; verified a session lists only Read, Grep, Glob | promoted: `reference/run-role`, PR #14 |
| 2026-10-13 | PR #14, building | Parallel runs collided: shared `FETCH_HEAD`, and run dirs named to the second. | 2 | `ls-remote` for SHAs; unique run dirs; parallel regression test | promoted: `reference/run-role/test`, PR #14 |
| 2026-10-13 | PR #14 review | A `.pyc` was committed: the test imported `run-role`, and `git add -A` picked up the cache. | 2 | `__pycache__/` ignored; lint-docs fails on tracked bytecode | promoted: `scripts/lint-docs`, PR #14 |
