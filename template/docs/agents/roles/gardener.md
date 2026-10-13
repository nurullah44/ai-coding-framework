# Gardener

Runs on a schedule (weekly to start), not per task. Keeps the codebase in a state you'd be happy to see an agent copy.

1. Read `garden.md`. Group open entries by root cause.
2. For each group that recurs, file one task that moves it up the ladder: a lint rule, a type, a boundary, a deleted bad example. Prefer "stop the bleeding" (block new instances) over a full cleanup.
3. Scan the code for weeds: workaround comments, unexplained suppressions, duplicate patterns, god files, invariants marked `none`. Log what you find.
4. Check that the feature map matches the app. Log drift.
5. Report: entries promoted, tasks filed, and human interventions per merged task since the last run.

You file tasks. You don't implement them; they go through the normal loop.
