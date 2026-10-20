# <Title>

## Goal

<One or two sentences, from the user's point of view.>

## Acceptance criteria

<!-- Each criterion is observable, names its source (issue, human decision, ADR, feature-map entry), and says how it is proven. Every command, path or field below was run once against the code or the app, not written from memory. -->
- [ ] <criterion> (source: <link or quote>)
  - Pass when: <the observable result, for example "`verify drive http GET /links` answers 200 and the new link is first">

## Failure modes in scope

<!-- The error and edge cases this ticket must handle. A Reviewer reports anything not listed as a note, not a failure. Define every vague word here ("ignoring case" means ...). -->
- <case>: <expected behaviour>
- Not in scope: <case>, <case>

## Verification plan

- Feature-map areas: <`.agents/skills/verify/features/<area>.md`, ...>
- `verify` needs extending for: <path and what to add, or "nothing">
- No runtime behaviour? <Only when the change cannot be driven at all, such as docs. Name the proof that replaces live driving, or write "none" and accept `UNVERIFIED`.>

## Out of scope

- <item>: <follow-up issue link>

## Constraints

<Files, libraries, ADRs and invariants that apply; one-way doors touched (if any, this ticket is L0).>

## Budget

- Estimated peak context: <tokens; see "Ticket size" in `process.md`>
- Measured peak context: <filled in by the Coordinator after the build>

## Blocked by

<Issue links, or "none".>
