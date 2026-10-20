# Invariants

Rules the code must never break, each with the check that enforces it (principle 2). The Planner writes them during design; they grow as the garden moves mistakes up the ladder.

Each invariant names its enforcement. `none` is allowed but always has a matching garden row, because an invariant without enforcement is only a wish.

| # | Invariant | Enforced by | Rung |
|---|---|---|---|
| 1 | <!-- e.g. UI code never imports from server/ --> | <!-- e.g. the import-boundary lint in `check` --> | 2 |

## Blessed patterns

One way to do each recurring thing. Agents copy the nearest example, so each row names one. Agents extend these, never alternatives. A pattern that gets broken twice becomes an invariant with a check.

| Concern | The one way | Example file |
|---|---|---|
| | | |

## Module boundaries

Dependencies point one way. Write the allowed direction here, then enforce it with an import-boundary check as soon as the stack exists. A boundary rule covers the whole repo by default and names its exceptions; a list of places to check leaves the next place unchecked.

```
<!-- e.g. features/* → shared/* → core/*   (never the reverse; features never import each other) -->
```
