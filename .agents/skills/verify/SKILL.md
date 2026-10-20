---
name: verify
description: Drive the running app like a user and capture evidence that a change works. Use before handing off any change, when verifying someone else's change, or when reproducing a bug report.
---

# Verify

`./.agents/skills/verify/verify --help` is canonical. Always use this CLI. Never write a one-off driver script.

When the CLI is missing a command that a path in the ticket needs, a Builder adds the command in the same change, with a test, and drives the path. A Reviewer, who never edits, lists the path under "Not covered" and logs the gap in `docs/agents/garden.md`.

This file is the contract. Each project builds the adapter behind it, starting with the first ticket that has runtime behaviour, and documents its own commands here as it adds them.

## Command contract

Every project implements these. The adapter behind them is stack-specific (browser via Playwright/CDP, HTTP calls, a CLI process, a device).

| Command | Does | Exit code |
|---|---|---|
| `doctor` | Checks the build is fresh, the app is reachable, and fixtures exist | 0 healthy, 1 broken, 2 not configured |
| `start` / `stop` | Starts or stops an isolated instance for this checkout | 0 / non-zero |
| `drive <action> …` | Performs one user-level action | 0 / non-zero |
| `evidence <name>` | Captures an artifact (screenshot, trace, response, log) into `.verify/artifacts/` | 0 / non-zero |
| `sweep` | Walks `features/README.md` in order for a broad regression | 0 all pass |
| `sweep <file>` | Runs one feature-map file | 0 all pass |

Exit code 2 means verification is not configured. The verdict is then `UNVERIFIED`, never PASS.

A scenario that needs the running instance across several commands runs as one shell command (`verify start && ... && verify stop`) or as a feature-map block under `sweep`. Some harnesses (Codex, for one) stop a command's background processes when that command ends.

## No runtime behaviour

If the task's verification plan says the change has no runtime behaviour (for example, docs only), run the proof it names instead of this CLI, and say so. Judge whether the claim is true: a change with runtime behaviour can't opt out. If the plan names no proof, or the proof could not run, the result is `UNVERIFIED`.

## Proof bar

- Run `doctor` first. Evidence against a stale build is not evidence.
- Drive the production user path. Use inspection (eval, DB read, logs) only to check state after the user path ran, never as the main proof.
- Read the feature-map entry for each area you touch. Cover every entry point it lists and the success, cancel, error, empty, and persistence paths the change can affect.
- Check side effects, not just what's on screen: stored data, network calls, a reload round trip.
- "It opens" is not proof. Show the trigger and the stable end state.
- Mocks count only at the same boundary a real external dependency would sit behind.
- Name every path you could not reach and why.

## Feature map

`features/README.md` is the index and sweep order. Each entry follows the contract in that file.
