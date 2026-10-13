# Reference

How we run the framework in this repo. It is one example, not a requirement. `template/` never depends on anything here; `scripts/check` fails if it points here.

Bring your own: any tool that meets the template's contracts (roles, labels, verdicts, the `verify` command contract) works.

## run-role

Starts one role as a fresh headless session in Claude Code, Codex or Pi.

```
reference/run-role/run-role reviewer <pr> --lane spec|standards|regression [--harness claude|codex|pi]
reference/run-role/run-role builder <issue> [--harness ...]
reference/run-role/run-role planner|coordinator|gardener [<issue>] [--harness ...]
```

- **Fresh and isolated.** Each run gets its own git worktree: detached at the PR head for a Reviewer, a new `issue-<n>` branch off `main` for a Builder.
- **Pointers, not context.** The prompt is the role file plus the target: for a Reviewer, the PR head SHA, the linked issue and the diff. It refuses a PR that links no issue.
- **Blind review.** A Reviewer is told not to read PR comments, other verdicts or the Builder's evidence, and runs with an empty `gh` config so it can't fetch them.
- **Pinned models.** Model and effort are always passed explicitly; defaults are at the top of the script. Override with `--model` and `--effort`.
- **Evidence.** Each run writes `<run>` = `.verify/runs/<time>-<role>[-lane]-<harness>/`, with `<run>/prompt.md`, `<run>/output.md`, `<run>/stderr.log` and `<run>/meta.json` (harness, model, effort, SHA, exit, whether the checkout stayed clean).

### Permissions

| Run | Claude Code | Codex | Pi |
|---|---|---|---|
| Reviewer, standards lane | Read, Grep, Glob | `--sandbox read-only` | read, grep, find, ls |
| Reviewer, spec or regression lane | + Bash; Edit and Write denied | `--sandbox workspace-write` | + bash |
| Every other role | + Edit, Write | `--sandbox workspace-write` | + edit, write |

The spec and regression lanes must run commands to drive the app or run `check`, which a read-only sandbox blocks (Codex can't even write `/tmp`). So they may run commands but not edit tools, and afterwards `run-role` checks that the worktree is unchanged and still at the reviewed SHA. If it isn't, the run exits 3 and the worktree is kept for inspection.

`run-role` exits with the harness's exit code, 3 for a modified checkout, or 124 on timeout (`--timeout`, default 1200 s).

### Tests

`reference/run-role/test` runs offline, as part of `scripts/check`. `--dry-run` prints the exact command and prompt without starting a session.
