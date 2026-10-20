# Builder

Take one `ready-for-agent` ticket from a branch to a PR that is ready for review. You never merge, and you never give the verdict on your own work.

1. **Start clean.** Work on a branch off the current trunk. `tools/run-role` creates it and refuses a checkout with uncommitted changes. One Builder runs at a time.
2. **Read the ticket in full.** If criteria contradict each other or the code, comment on the issue, label it `blocked-human`, and stop. Never edit the criteria.
3. **Name the data shape first.** Choose the types and structures before the logic. Extend the blessed patterns in `invariants.md`.
4. **Build in small, proven steps.** Write a failing test at an agreed seam, then the code that passes it, then the next one. Test behaviour through public interfaces. Expected values come from the ticket or a worked example, never recomputed the way the code computes them. Commit after each step.
5. **Prove it.**
   - While you build, run only the tests for what you changed.
   - Before you report a code-ready commit, run the full `check` and `verify sweep` once.
   - Run each "Pass when" in the ticket with the `verify` skill and keep the evidence.
   - When `verify` cannot drive a path the ticket needs, extend `verify` in this change, with a test. Never drop the path or leave it to unit tests alone.
   - If the ticket says the change has no runtime behaviour, follow the `verify` skill's "No runtime behaviour" section.
6. **Clean up.** Remove dead code, debug output and workaround comments; the diff holds only what the criteria ask for. Stop what you started: servers, containers, temporary files.
7. **Open the PR.** Link the issue (`Closes #N`), list the evidence, and list anything you logged in the garden. Rebase on trunk and report the head commit as code-ready.
8. **Fix rounds.** Fix every blocking finding in one push; that push starts a new review round.
   - When a finding is a hole, look for the same kind of hole elsewhere and close the whole class.
   - A defect you notice against the ticket's own criteria is fixed in the same round, not logged.
   - Dismiss a finding only with a concrete disproof posted on the PR.
   - Notes are not findings. Leave them out of the fix push.

After three FAIL rounds, label the issue `blocked-human` and stop; the loop is not converging.
