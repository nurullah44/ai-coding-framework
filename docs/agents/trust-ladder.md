# Trust ladder

Every time a human corrects an agent, or an agent works around something, ask: **what is the highest rung that makes this mistake impossible, or at least caught automatically?** Fix it there. Fixing only this one PR does not count.

| Rung | Mechanism | Strength |
|---|---|---|
| 1. Codebase | Types, data structures, module boundaries, one blessed pattern, deleting the bad example | The mistake can't be written. Agents copy what they see, so the codebase is their memory. |
| 2. Checks | Lint rules, import-boundary rules, compiler flags, tests, CI gates | The mistake is caught on every run |
| 3. Guidance | `AGENTS.md`, skills, role files | Usually followed, sometimes skipped |
| 4. Review | A Reviewer or a person notices it | Catches it once. A review that catches the same mistake on every PR is a gap in rungs 1 to 3. |

Verification (the `verify` skill) runs alongside the ladder. It proves the behaviour is correct. It does not prove quality or performance; those need their own checks.

## When to climb

- **Seen once:** log it in `garden.md`.
- **Seen twice:** move it up now. A new check must fail on the real past mistake before it counts.
- A rule written only in `AGENTS.md` or a role file (rung 3) that gets broken twice is a candidate for rung 2.

## Gardening

Bad patterns spread: an agent copies a workaround once, and within weeks it is the usual way.

- **Stop the bleeding first.** Add a check against the pattern now. Cleaning up the existing cases can wait.
- **Collect before fixing.** Patterns show up only when entries are seen together. The Coordinator reads the garden at the end of each roadmap step (`roles/coordinator.md`).

## Moving to the next autonomy level

Move up only when all of these hold for the work since the last change:

- Verification is set up and covers the feature-map areas the work touched.
- The garden has no open rung 4 entry seen twice or more.
- Human interventions per merged ticket are going down.
