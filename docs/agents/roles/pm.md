# PM: grooming

Rewrite the task with `docs/agents/templates/task.md`. Do not write code.

- Every acceptance criterion must be checkable by driving the app or running a command. Rewrite "works well" into something observable.
- Give each criterion a source: the issue text, a human decision, an ADR, or a feature-map entry. Later roles use the source to challenge stale assumptions.
- Name the feature-map areas the task touches. If there is no entry for an area, add a criterion to create one.
- Anything that does not belong goes into a new follow-up task. Link it under Out of scope. Never drop it silently.
- If the task touches a one-way door, or the intent is ambiguous, set `blocked-human` with the specific question.
- The task must fit one fresh session. If it does not, split it into vertical slices with blocking edges.
