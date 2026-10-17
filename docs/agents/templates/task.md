# <Title>

## Goal

<One or two sentences, from the user's point of view.>

## Acceptance criteria

<!-- Each criterion is observable. Source = issue, human decision, ADR, or feature-map entry. -->
- [ ] <criterion> (source: <link or quote>)

## Verification plan

- Feature-map areas: <`.agents/skills/verify/features/<area>.md`, …>
- Paths to drive: <success / cancel / error / empty / persistence, as relevant>
- Not reachable by the harness: <path and reason, or "none">
- No runtime behaviour? <Only when the change cannot be driven at all (e.g. docs). Name the proof that replaces live driving, or write "none" and accept `UNVERIFIED`.>

## Out of scope

- <item> → <follow-up task link>

## Constraints

<Files, libraries, prior decisions, one-way doors touched (if any, this task is L0).>

## Budget

- Estimated peak context: <tokens, at most 90k; see `process.md`>
- Builder: Haiku | Opus (only with the reason splitting would be wasteful)
- Measured peak context: <filled in by the Coordinator after the build>

## Blocked by

<Task links, or "none".>
