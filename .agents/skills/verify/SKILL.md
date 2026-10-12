---
name: verify
description: Drive the running app like a user and capture evidence that a change works. Use before handing off any change, when verifying someone else's change, or when reproducing a bug report.
---

# Verify

`./.agents/skills/verify/verify --help` is canonical. Always use this CLI. Never write a one-off driver script; if the CLI is missing a command you need, say so and log it in `docs/agents/garden.md`.

## Command contract

Every project implements these. The adapter behind them is stack-specific (browser via Playwright/CDP, HTTP calls, a CLI process, a device).

| Command | Does | Exit code |
|---|---|---|
| `doctor` | Checks the build is fresh, the app is reachable, and fixtures exist | 0 healthy, 1 broken, 2 not configured |
| `start` / `stop` | Starts or stops an isolated instance for this checkout | 0 / non-zero |
| `drive <action> …` | Performs one user-level action | 0 / non-zero |
| `evidence <name>` | Captures an artifact (screenshot, trace, response, log) into `.verify/artifacts/` | 0 / non-zero |
| `sweep` | Walks `features/README.md` in order for a broad regression | 0 all pass |

Exit code 2 means verification is not configured. The verdict is then `UNVERIFIED`, never PASS.

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
