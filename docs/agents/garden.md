# Garden

Append-only buffer of corrections, workarounds, and weeds. The Gardener reviews it in batches and turns repeated entries into rung 1 or 2 fixes.

Status values: `open`, `promoted` (a fix landed; link it), `wontfix` (reason given).

| Date | Source | What happened | Target rung | Proposed fix | Status |
|---|---|---|---|---|---|
| | <!-- task id, review, transcript, human --> | | <!-- 1-4 --> | | open |
| 2026-10-13 | PR #9 review, round 1 | Docs pointed at paths that don't exist (`features/<area>.md` in the task template; the feature map lives under `.agents/skills/verify/features/`). Fixed by hand in PR #9. | 2 | A link and path check in the framework self-check (#5) | open |
| 2026-10-13 | Human + test | Claude Code silently skips `AGENTS.md` when a `CLAUDE.md` without `@AGENTS.md` exists (tested on 2.1.296: AGENTS.md alone loads; with a plain CLAUDE.md it doesn't). `/init` creates exactly that file. | 2 | Self-check (#5) fails if `CLAUDE.md` exists without `@AGENTS.md` | open |
