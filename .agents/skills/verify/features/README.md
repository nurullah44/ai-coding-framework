# Feature map

Behaviour-level inventory of the app. Agents use it to decide what to drive and what counts as evidence. Humans use it as the regression checklist. Fix drift in the same PR as the change that caused it.

## Baseline preconditions

<!-- Project-wide: signed-in state, seed data, flags, selector conventions. -->

## Sweep order

Walk top to bottom for a broad regression.

<!-- - [area](area.md): one-line summary -->

## Entry contract

One file per feature area, short enough to run without reading source. Every file has exactly these four H2s:

1. `Sub-features`: what exists.
2. `How to get to it (user POV)`: entry points a user takes.
3. `Driving it with verify`: the commands, selectors, and expected end states.
4. `Gotchas`: what usually lies, flakes, or needs a manual step.

Split a file when one section needs its own preconditions.
