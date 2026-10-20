# Principles

Six principles. Every rule in this framework serves one of them. When a rule and a principle disagree, the principle wins and the rule gets fixed.

Each principle names where it comes from and what the first test project (a URL shortener built by agents over three days, 31 PRs) showed about it. The data is in [`sources/lessons-from-the-test-project.md`](../../sources/lessons-from-the-test-project.md).

## 1. Proof, not claims

Work is done when it has been shown working on the running app, with evidence someone else can check. Passing tests are not proof of behaviour. A proof that could not run gives `UNVERIFIED`, and `UNVERIFIED` is never a pass.

- **From:** Lauren Tan ("A compile, unit test, code review, or plausible diff is not after evidence"), Alexey Grigorev ("Only the acceptance criteria and the running code count").
- **Showed:** no `UNVERIFIED` work merged. In 31 PRs one defect got past review: the server ran a different Node version than the tests, and the first real backup failed. The proof run on the server caught it.

## 2. Enforce, don't instruct

A rule that matters becomes something that fails: a type, a module boundary, a lint rule, a test. Written guidance is for judgment calls only, because nothing fails when an agent skips it.

- **From:** Lauren Tan's ladder ("Write docs or agent rules last, only for judgment calls. Nothing fails when an agent skips them.").
- **Showed:** rules backed by a check were never broken again. Two rules that stayed in the docs only were broken five times between them.

## 3. The ticket defines done

Before building starts, the ticket says what done looks like: each criterion, how it is proven (its "Pass when"), and which failure cases are in scope. The Builder builds to that and the Reviewer judges against that, not against their own idea of done.

- **From:** Matt Pocock's agent brief ("concrete, testable acceptance criteria", "Explicit scope boundaries"), Lauren Tan's plans ("Every single task in the plan is structured around proof and verification").
- **Showed:** about a quarter of the blocking findings came from tickets that were wrong, vague, or silent on failure cases, and these were the findings that kept rounds going. Switching to a stronger Builder model did not raise the first-round pass rate.

## 4. Nobody grades their own work

The one who builds never gives the verdict and never merges. Each Reviewer is a fresh session that has not seen the Builder's reasoning. A verdict holds for one commit only.

- **From:** Lauren Tan (independent verifiers, a verdict per commit), Matt Pocock (separate Standards and Spec reviewers), the role-based loop (independent QA).
- **Showed:** independent reviewers found 71 blocking problems; 26 of them were real bugs.

## 5. Fix it upward

When the same mistake shows up twice, move it up the [trust ladder](trust-ladder.md) so it can't happen a third time: from review to a written rule, from a rule to a check, from a check to a structure that makes the mistake impossible.

- **From:** Lauren Tan ("add a skill or a check when the same failure shows up twice").
- **Showed:** where we did this, the mistake stopped. Where the log of mistakes waited for a weekly review that never came, it piled up.

## 6. Earn trust before speed

Run one agent at a time, watch what it does, and turn its repeated failures into tools and checks. Give it more autonomy, or run agents in parallel, only after that.

- **From:** Lauren Tan ("Earn the trust before the loop. A loop you don't trust just produces unchecked work faster").
- **Showed:** one agent at a time with a Coordinator finished the last roadmap step overnight with no human in the loop.
