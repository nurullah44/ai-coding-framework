# Process

Work lives in GitHub Issues and PRs. Labels carry the state.

## Roles

| Role | Does | Never |
|---|---|---|
| [Planner](roles/planner.md) | Turns intent or a report into `ready-for-agent` issues | Writes code |
| [Builder](roles/builder.md) | Takes one issue to a merge-ready PR | Merges, or edits acceptance criteria |
| [Reviewer](roles/reviewer.md) | Judges one PR at one SHA, one lane per session | Fixes code, or reads other verdicts first |
| [Coordinator](roles/coordinator.md) | Dispatches, collects verdicts, merges | Writes code |
| [Gardener](roles/gardener.md) | Scheduled: moves recurring problems up the ladder | Implements its own findings |

Without a Coordinator, the human plays that part.

## Models

Always pass the model and effort explicitly; never rely on a harness default.

| Role | Harness | Model | Effort |
|---|---|---|---|
| Planner | Claude Code | `claude-opus-5-5` | xhigh |
| Builder | Claude Code | `claude-haiku-5-5` | max |
| Builder, for a ticket too big to split | Claude Code | `claude-opus-5-5` | medium |
| Reviewer, every lane | Codex | `gpt-6.1-sol` | xhigh |

## Ticket budget

The budget is the builder's peak context (what the model holds at once), not the total tokens spent.

- Plan every ticket to fit in 90k. An empty Haiku session in Claude Code already starts at about 18.5k (measured 2026-10-17).
- If a ticket will surely need more than 100k and splitting it would be wasteful, the Planner marks it for the Opus builder and says why.
- The ticket states its estimate. After the build, the Coordinator writes the measured peak next to it, so estimates can be checked against reality.
- Nothing stops a builder at the limit yet. Until a guard exists, runs up to about 120k are tolerated and recorded.

## Flow

```
intent ─Planner (with human)─┐
                             ├─▶ issue: ready-for-agent ─Builder─▶ PR (code-ready SHA) ─Reviewer lanes─▶ verdicts
report ─Planner (triage)─────┘                                          ▲                                 │
                                                                        └──── FAIL: one fix push ─────────┤
                                                                                                          ▼
                                                                                     all PASS at this SHA ─▶ merge
```

## Labels

| Label | Meaning |
|---|---|
| `needs-triage` | A report nobody has looked at yet |
| `needs-info` | Waiting on the reporter |
| `ready-for-agent` | Groomed; a Builder can take it |
| `blocked-human` | Needs a human: contradictory criteria, one-way door, design call, or three FAIL rounds |

## Rules

- **Nobody grades their own homework.** The Builder never issues the verdict on its own PR and never merges on it.
- **A verdict belongs to a SHA.** A push that changes the patch voids every verdict and starts a new round.
- **Live proof is the floor.** No merge without PASS from both required lanes: spec + live proof, and standards. `UNVERIFIED` is not PASS.
- **One writer per branch.** Each Builder works in its own worktree.
- **Three FAIL rounds** on one issue → `blocked-human`. The loop is not converging.
- **Every verdict lists ladder candidates.** They go into the garden.

## Autonomy level

Set per project. Start at L0. Promote only with the evidence `trust-ladder.md` asks for.

| Level | Merge | Human role |
|---|---|---|
| **L0** | A human merges after the verdicts | Reads every PR. Every review comment is also a garden entry. |
| **L1** | The Coordinator merges when the merge conditions in `roles/coordinator.md` hold | Samples merged work from the commit history. Reverts and logs anything wrong. |
| **L2** | L1, plus reports and alerts open `needs-triage` issues automatically | Samples; tends the garden |

One-way doors listed in `AGENTS.md` stay L0 at every level.

## Loops

Run a loop only with a checkable stop condition, such as "no `ready-for-agent` issue is open". A duration or "make it better" is not a stop condition.
