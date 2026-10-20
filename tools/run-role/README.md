# run-role

Starts one role as a fresh, headless session in Claude Code or Codex.

```
tools/run-role/run-role reviewer <pr> --lane spec|standards|regression [--harness claude|codex]
tools/run-role/run-role builder <issue> [--harness ...]
tools/run-role/run-role builder <issue> --pr <pr> [--harness ...]
tools/run-role/run-role planner|coordinator [<issue>] [--harness ...]
```

- **One checkout.** A run starts in the project checkout, the one `run-role` is started from. It makes no worktree and no clone. A run refuses a checkout with uncommitted changes, then switches the checkout:
  - a Reviewer: detached at the PR head, unless it is already there. The lanes of one PR may run at the same time.
  - a Builder: a new `issue-<n>` branch from an up-to-date `main`. A checkout with no local `main` gets one at the remote's. A branch that already exists refuses the run.
  - a fix round (`builder <issue> --pr <pr>`): the PR's branch at its head, with the latest verdict of each lane in its prompt. A local branch at any other commit refuses the run.
  - a planner or coordinator: an up-to-date `main`.

  A Reviewer whose lane changes the checkout leaves the change there for inspection, and the next run refuses until the checkout is clean.
- **Pointers, not context.** The prompt is the role file plus the target: for a Reviewer, the PR head SHA, the linked issue and the diff. It refuses a PR that links no issue.
- **Blind review.** A Reviewer is told not to read PR comments, other verdicts or the Builder's evidence. It may use `gh` itself.
- **Models from the table.** Without `--harness`, `--model` or `--effort`, a role starts on its row of the Models table in `docs/agents/process.md`. The launcher reads the row from the commit the run starts from, so the table is the only place the defaults are written. The flags override the row. With `--harness` for another harness, the row's model and effort no longer apply, and that harness's defaults in `DEFAULTS` at the top of the script are used. A role whose row is missing refuses to start unless `--model` and `--effort` are given. The test checks the defaults against the table, and changes a row to check that the launcher follows it.
- **Evidence.** Each run writes a unique `<run>` = `.verify/runs/<time>-<role>[-pr<n>|-issue<n>[-fix-pr<n>]][-lane]-<harness>-<id>/`, with `<run>/prompt.md`, `<run>/output.md`, `<run>/stderr.log` and `<run>/meta.json`. `meta.json` holds the harness, model, effort, SHA, exit, `peak_context_tokens`, `quota_five_hour_start`, `quota_five_hour_end`, `thinking_tokens`, the PR of a fix round, and for a Reviewer whether the checkout stayed clean. For Claude Code, `output.md` is the final answer only, and the raw stream is kept as `<run>/stream.jsonl`. Verify artifacts that the run wrote under `.verify/artifacts/` are copied to `<run>/verify/`.
- **Peak context.** For Claude Code, `peak_context_tokens` is the largest `input_tokens + cache_read_input_tokens + cache_creation_input_tokens` of any one `assistant` event: the most context the model held at once. It is `null` when there was no assistant event, and for Codex, where it is not measured.
- **Quota and thinking.** For Claude Code, `quota_five_hour_start` and `quota_five_hour_end` are the first and last `five_hour` utilization in the stream's `rate_limit_event` lines, and `thinking_tokens` is the sum of `estimated_tokens_delta` over the `thinking_tokens` system events. Each is `null` when the stream has none, and for Codex.

### Permissions

| Run | Claude Code `--tools` | Codex |
|---|---|---|
| Reviewer, standards lane | Read, Grep, Glob | `--sandbox read-only` |
| Reviewer, spec or regression lane | + Bash | `--sandbox workspace-write`, with network access |
| Every other role | + Bash, Edit, Write | `--sandbox workspace-write` |

For Claude Code, `--tools` sets which tools exist; `--allowedTools` only auto-approves them, so it is not a restriction on its own. `--strict-mcp-config` drops MCP servers from user config, which could otherwise write to external services.

The spec and regression lanes must run commands to drive the app or run `check`, which a read-only sandbox blocks (Codex can't even write `/tmp`). So they may run commands but not edit tools, and afterwards `run-role` checks that the checkout is unchanged and still at the reviewed SHA. If it isn't, the run exits 3 and the change stays in the checkout for inspection.

`run-role` exits with the harness's exit code, 124 on timeout, or 127 if the harness is not on `PATH`. A Reviewer whose checkout changed exits 3 whatever the harness exited with. `--timeout` is in seconds. The default is 3600 for a Builder and 1200 for every other role.

### Tests

`python3 tools/run-role/test` runs offline. Add it to your project's `check`. It drives the CLI against a local bare repo standing in for GitHub, with stub `gh`, `claude` and `codex` executables. It covers the checkout each role starts from, the refusals for uncommitted changes, a local `main` ahead of the remote, and a fix round's branch not at the PR head, the permissions per lane, the Codex network access, refusing a PR with no issue, exit 3 on an edited checkout (also when the harness failed), a failing harness on a clean checkout keeping its exit, the kept evidence, timeout, a Builder's branch, the fix round and its verdicts, parallel lanes of one PR (which must run at the same time in the checkout), the Models table (the defaults, and that the launcher follows the table), the refusal for a role whose row is missing, the Builder's timeout, the Claude Code stream, the peak context, and the quota and thinking tokens.

`--dry-run` prints the harness, model, effort, timeout, the exact command and the prompt. It changes no checkout or branch, creates no run dir, and starts nothing. Like a real run, it is refused for a checkout with uncommitted changes, and for a Builder whose branch already exists, except on a fix round, whose branch exists by design.
