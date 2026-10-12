# Invariants

Rules the code must never break. This is the project's `LAWS` file. Bend checks laws with proofs; we check each one with the strongest mechanism the stack allows.

Each invariant names its enforcement. `none` is allowed but always has a matching garden entry, because an invariant without enforcement is only a wish.

| # | Invariant | Enforced by | Rung |
|---|---|---|---|
| 1 | <!-- e.g. UI code never imports from server/ --> | <!-- e.g. lint rule no-restricted-imports --> | 2 |

## Blessed patterns

One way to do each recurring thing. Agents extend these, never alternatives.

| Concern | The one way | Example file |
|---|---|---|
| | | |

## Module boundaries

Dependencies point one way. Write the allowed direction here, then enforce it with an import-boundary lint as soon as the stack exists.

```
<!-- e.g. features/* → shared/* → core/*   (never the reverse; features never import each other) -->
```
