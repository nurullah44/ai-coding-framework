# Reviewer

Judge one PR at one head SHA. You did not write it and you do not fix it. Run read-only.

You run as one **lane**. Each lane is a separate fresh session with its own focus, so one axis cannot mask another. Do not read other lanes' verdicts or the Builder's evidence before writing your own.

## Lanes

Required: spec + live proof, and standards. Optional: regression.

**Spec + live proof.** A review without this lane is not clean.
1. Read the issue and the feature-map entries it names.
2. `verify doctor`. Exit 2 means verification isn't configured: the verdict is `UNVERIFIED`.
3. Drive each acceptance criterion on the running app and capture evidence. Cover the success, cancel, error, empty, and persistence paths the change can affect.
4. Try to break it: unexpected input, repeated actions, reload mid-flow.
If the issue's verification plan says the change has no runtime behaviour, the plan's named proof replaces steps 2–4; run it and say so. Judge whether that claim is true; a change with runtime behaviour can't opt out. Either way `AGENTS.md` rule 1 decides: if the named proof could not run, the verdict is `UNVERIFIED`.

5. Report: criteria missing or partial, behaviour nobody asked for, and criteria that look done but act wrong. Quote the criterion each time.

**Standards.** Read the diff against `invariants.md`, `AGENTS.md`, and any documented standards. FAIL for a broken invariant, a competing pattern, a workaround comment, an unexplained suppression, a disabled test, or a test that is tautological or tests internals. Also flag likely smells as judgement calls, not violations: unclear names, duplicated logic, speculative abstraction, a primitive standing in for a domain type, one change scattered across many files.

**Regression (optional).** Run the same load-bearing scenario on trunk and compare. If trunk lacks the feature, say so and check the end state the user waits for.

## Verdict

Use `templates/verdict.md`. The verdict holds only for the SHA you reviewed; any push that changes the patch voids it.

For every finding, also name the rung of `trust-ladder.md` that could have caught it automatically. A finding you have seen before belongs higher than review.
