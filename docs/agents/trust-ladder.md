# Trust ladder

Every time a human corrects an agent, or an agent works around something, ask: **what is the highest rung that makes this mistake impossible, or at least caught automatically?** Fix it there. Fixing only this one PR does not count.

| Rung | Mechanism | Strength |
|---|---|---|
| 1. Codebase | Types, data structures, module boundaries, a single blessed pattern, deleting the bad example | Mistake can't be written. Agents copy what they see, so the codebase is their memory. |
| 2. Static analysis | Lint rules, import-boundary rules, compiler flags, CI gates | Mistake is caught on every run |
| 3. Guidance | `AGENTS.md`, skills, role files, review bots | Usually followed, sometimes skipped |
| 4. Human review | A person notices it | Does not scale. Treat each catch as a gap in rungs 1 to 3. |

Verification (the `verify` skill) runs alongside the ladder. It proves the behaviour is **correct**. It does not prove quality or performance; those need their own checks.

## Gardening

Anti-patterns spread: an agent copies a workaround once, and within weeks it is the de facto pattern.

- **Stop the bleeding first.** Add a lint rule or CI tripwire against the pattern now. Cleaning up existing instances can wait.
- **Buffer before fixing.** Log weeds in `garden.md` instead of fixing them one at a time. Review the buffer in batches; patterns show up only when entries are seen together.
- **Mine transcripts.** Periodically have an agent read past session transcripts for places a human intervened or repeated themselves. Each one is a garden entry.

## Promotion to the next autonomy level

Promote only when all of these hold for the work since the last level change:

- Verification is configured and covers the feature-map areas the work touched.
- The garden has no open entries at rung 4 that recur. A recurring entry means a missing rule.
- Human interventions per merged task are trending down. Count them from `garden.md`.
