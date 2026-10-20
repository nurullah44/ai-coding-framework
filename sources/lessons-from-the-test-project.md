# Lessons from the test project

The evidence behind this version of the framework. The test project was `url-shortener-lab`: a self-hosted URL shortener after [Shlink](https://shlink.io), built by agents from an empty repo to a deployed app with backups, between 2026-10-17 and 2026-10-20.

Numbers come from the 31 PRs' verdict comments and the per-ticket notes. The classification of findings is ours.

## What was built

Six roadmap steps: a skeleton (check, CI, verify), short links and redirects, a dashboard, link management with API keys, background visit locations, and a deploy with a backup and restore drill. 31 PRs merged. The app runs on a server, reachable over Tailscale only.

## How the loop performed

| Measure | Result |
|---|---|
| Passed both review lanes in round 1 | 7 of 31 (23%); the target was 80% |
| Merged by round 2 | 20 of 31 (65%) |
| Merged by round 3 | 30 of 31 (97%) |
| Average rounds per PR | 2.2 |
| Blocking findings in total | 71 |
| `UNVERIFIED` work merged | 0 |
| Defects that got past review | 1 (a Node version difference between tests and the server) |

## Where the 71 blocking findings came from

| Cause | Count | What fixes it in this version |
|---|---|---|
| Real behaviour bugs | 26 (37%) | Nothing should: catching these is review's job |
| A rule written only in docs was broken | 15 (21%) | Principle 2: turn a rule broken twice into a check |
| Edge cases the ticket never scoped | 13 (18%) | "Failure modes in scope" in the task template; out-of-scope edges are notes |
| Proof incomplete (a path left undriven) | 7 (10%) | The Builder extends `verify` instead of dropping a path |
| Design details (a font size, a weight) | 6 (8%) | Already notes, not failures |
| The ticket itself was wrong or contradictory | 4 (6%) | The Planner's check before publishing |

## What worked

- **Checks hold, written rules don't.** Once "tests import a module only through its public entry" and "the database driver is imported only in two places" became checks, neither was broken again. Two rules that stayed in the docs only (parse outside input with the shared schema; send errors through one helper) were broken five times between them. One agent even hid a forbidden import inside a string to get past a check; the standards Reviewer caught it.
- **Independent review catches real bugs.** 26 of the blocking findings were real: an auth bypass through a malformed path, a race that discarded a user's edit, dates compared as text.
- **Running heavy checks once.** Having the Builder run the full check once at handoff, instead of around every commit, saved 10 to 15 minutes per ticket.
- **One lane after a small fix.** Both lanes run in parallel, so this saves Reviewer quota more than time. In the PRs that ran both lanes after a fix, a lane that had passed found something new five times; three were real bugs that had been there since round 1. The real problem is a Reviewer that does not report everything in round 1.
- **A Coordinator running alone.** The last roadmap step (deploy, backups, restore drill) was finished overnight with no human.
- **Module boundaries in a monorepo.** No boundary break got through; the server and the dashboard shared one set of schemas.

## What did not work

- **Vague tickets.** "Ignoring case" without a definition sent Reviewers through the Unicode tables over three rounds. A deploy ticket that did not say which failures mattered found a new edge in every round.
- **Open-ended review.** A Reviewer told to "try to break it", with every defect a person might hit counted as blocking, found something new every round.
- **A cheap Builder model.** Haiku's mistakes were caught, but each cost a round, and its context ran to two or three times the estimate. The default Builder is now Opus at medium effort. Model choice alone did not raise the first-round pass rate; ticket quality matters more.
- **The garden without a rhythm.** 232 rows in three days, about 115 still open. The Gardener role was scheduled weekly and never ran in a three-day project. Every review note went into the garden, and the same root cause got a new row each time, so nothing counted repeats. Now: one row per root cause with a count, and the Coordinator reads it at the end of each roadmap step.
- **Instructions for mechanical jobs.** A wait loop that matched its own command line cost two hours unattended; agents left containers and files behind three times in one night. These need tools, not more text.

## Open gaps carried into this version

These are known, not fixed yet:

- `run-role` has no `wait` command, so a Coordinator writes its own wait loop.
- Nothing cleans up what a Builder or Reviewer leaves running.
- A fix round is not told its round number.
- Nothing stops a Builder that goes far past its context budget.
- The success test of this framework (a newcomer, with only this repo, is led through a project) has not been run yet.
