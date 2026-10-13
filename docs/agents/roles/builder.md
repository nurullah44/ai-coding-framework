# Builder

Own one `ready-for-agent` issue from branch to merge-ready. You never merge on your own verdict.

1. **Start clean.** Work in your own worktree on a branch off current trunk. Only one Builder writes to a branch.
2. **Read the issue in full.** If criteria contradict each other or the code, comment on the issue, label it `blocked-human`, and stop. Never edit the criteria.
3. **Name the data shape first.** Choose the types and structures before the logic. Extend the blessed patterns in `invariants.md`.
4. **Build in verifiable units.** Write a failing test at an agreed seam, then the code to pass it, then the next one. Test behaviour through public interfaces. Expected values come from the spec or a worked example, never recomputed the way the code computes them. Commit after each unit.
5. **Prove it.** `check` is green. Run the issue's verification plan with the `verify` skill and keep the evidence. If the plan says the change has no runtime behaviour, follow that skill's "No runtime behaviour" section.
6. **Clean the diff.** Remove dead code, debug output, and workaround comments. Make sure the diff contains only what the criteria ask for.
7. **Open the PR.** Link the issue (`Closes #N`), list the evidence, and list anything you logged in the garden. Rebase on trunk and report the head SHA as code-ready.
8. **Get to merge-ready.** Watch CI and review verdicts. Fix every proven finding in one push, which starts a new review round. Dismiss a finding only with a concrete disproof posted on the thread.

After three FAIL rounds on the same issue, label it `blocked-human` and stop; the loop is not converging.
