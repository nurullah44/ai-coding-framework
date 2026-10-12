# Process: the task graph

One task at a time per engineer. The tracker is the source of truth (default: GitHub Issues through `gh`; declare another tracker in `AGENTS.md`).

## States

```
needs-grooming ──PM──▶ ready ──Engineer──▶ needs-verification ──Verifier + Reviewer──▶ verified ──Orchestrator──▶ merged
                  │                                  ▲                     │
                  └─▶ blocked-human                  └──────── FAIL ───────┘
```

| State | Entered by | Exit condition (must be checkable) |
|---|---|---|
| `needs-grooming` | anyone files it | PM rewrites the task with `templates/task.md` |
| `ready` | PM | Engineer picks it up |
| `needs-verification` | Engineer | `check` is green, branch is pushed, engineer's own evidence is attached |
| `verified` | Verifier PASS **and** Reviewer PASS | Orchestrator merges or asks a human (see autonomy level) |
| `blocked-human` | anyone | A human answers. Use it for contradictory criteria, one-way doors, or missing access |

## Rules

- Never skip grooming. Grooming is the cheapest place to catch a misunderstanding.
- Nobody grades their own homework. The agent that wrote the change never issues its verdict and never merges it.
- The Verifier and Reviewer never fix code. They issue a verdict with `templates/verdict.md`.
- On FAIL, the verdict goes back to the Engineer as input. After 3 FAILs on one task, the task moves to `blocked-human`; the loop is not converging.
- Every verdict lists **ladder candidates**: mistakes that a higher rung could have prevented. The Orchestrator appends them to `garden.md`.
- Only the Orchestrator closes tasks and merges.

## Autonomy level

Set per project. Start at L0. Promote a level only with evidence from `garden.md` (see `trust-ladder.md`).

| Level | Merge | Human role |
|---|---|---|
| **L0** | A human approves every merge | Reviews every PR. Every review comment is also a garden entry. |
| **L1** | The Orchestrator merges after both PASS verdicts and green CI | Samples merged work daily from the commit history. Reverts and logs anything wrong. |
| **L2** | Same as L1, plus an outer loop (alerts, bug reports, chat) creates tasks automatically | Samples; tends the garden |

One-way doors in `AGENTS.md` are L0 at every level.

## Stop conditions for loops

Run a loop only with a checkable goal: "every `ready` task is `verified` or `blocked-human`", or "`verify sweep` passes". "Make it better" is not a goal.
