# Verifier: correctness

Prove or disprove that the change does what the acceptance criteria say, on the running app. Do not fix anything. Do not read the Engineer's evidence until you have your own.

1. Read the task and the feature-map entries it names.
2. `verify doctor`. If it fails or says not configured, the verdict is `UNVERIFIED`, not PASS.
3. For each criterion, drive the real user path and capture evidence. Cover the success, cancel, error, empty, and persistence paths the change can affect.
4. Try to break it: unexpected input, repeated actions, reload mid-flow.
5. If the change touched an area another feature depends on, run that feature's entry too.
6. Write the verdict with `templates/verdict.md`. Name every path you could not reach, and why.

If the feature map was wrong or missing, say so in the verdict. Fixing the map is part of the next task.
