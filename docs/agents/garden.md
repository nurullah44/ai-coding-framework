# Garden

Append-only buffer of corrections, workarounds, and weeds. The Gardener reviews it in batches and turns repeated entries into rung 1 or 2 fixes.

Status values: `open`, `promoted` (a fix landed; link it), `wontfix` (reason given).

| Date | Source | What happened | Target rung | Proposed fix | Status |
|---|---|---|---|---|---|
| 2026-10-12 | framework, first session | The agent built the whole framework before any decision was settled with the human, skipping its own planning step. | 3 | Start every piece of work from an issue planned with the human | open |
| 2026-10-13 | nurullah44/ai-coding-framework#9 review | Docs pointed at paths that don't exist. | 2 | A check that every path named in the docs exists | open: the framework's path check was removed in nurullah44/ai-coding-framework#16 |
| 2026-10-13 | nurullah44/ai-coding-framework#9 review | Review rounds never converged: each standards lane re-read every doc and failed the PR for problems that were already there. | 3 | Reviewers judge only the diff; older problems go to the garden | promoted: `docs/agents/roles/reviewer.md` |
| 2026-10-13 | test | Claude Code skips `AGENTS.md` when a `CLAUDE.md` without `@AGENTS.md` exists, and `/init` creates exactly that file. | 3 | Say so in the README | promoted: framework README |
| 2026-10-13 | nurullah44/ai-coding-framework#9 review | Headless `codex exec` and `pi -p` hung waiting on stdin. | 1 | Bring the review launcher from nurullah44/ai-coding-framework#14 the next time we run reviews | open |
| 2026-10-13 | nurullah44/ai-coding-framework#9–#14 | `claude -p --allowedTools X "prompt"`: the variadic flag swallows the prompt. | 1 | Bring the review launcher from nurullah44/ai-coding-framework#14 the next time we run reviews | open |
| 2026-10-13 | nurullah44/ai-coding-framework#9–#14 | Review lanes ran on harness defaults; Codex fell back to low effort. | 1 | Bring the review launcher from nurullah44/ai-coding-framework#14 the next time we run reviews | open |
| 2026-10-13 | nurullah44/ai-coding-framework#13 review | `codex exec --sandbox read-only` can't write `/tmp`, so a spec lane couldn't run checks and came back UNVERIFIED. | 1 | Bring the review launcher from nurullah44/ai-coding-framework#14 the next time we run reviews | open |
| 2026-10-13 | nurullah44/ai-coding-framework#14 review | Claude's `--allowedTools` only auto-approves; a "read-only" session still had MCP tools that write to external services. | 1 | Bring the review launcher from nurullah44/ai-coding-framework#14 the next time we run reviews | open |
| 2026-10-13 | nurullah44/ai-coding-framework#14 | Parallel review runs collided: a shared `FETCH_HEAD`, and run dirs named to the second. | 1 | Bring the review launcher from nurullah44/ai-coding-framework#14 the next time we run reviews | open |
| 2026-10-13 | nurullah44/ai-coding-framework#14 review | A `.pyc` got committed: `git add -A` picked up the cache. | 1 | Ignore `__pycache__/` in `.gitignore` once there is Python here | open |
| 2026-10-14 | human | Too much at once: branches, PRs and issues piled up, and the framework grew parts nobody had used. | 1 | Squash-only merges that delete the branch; AGENTS.md rules 7 and 8 | promoted: repo settings, `AGENTS.md` |
