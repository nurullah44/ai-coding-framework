# How Stripe, Cloudflare and Cursor ship agent-written code: pipeline, guardrails, review, critique

Research date: 2026-10-14. Source labels used below:
- **[FH]** first-hand: the organisation's own engineers, repo, commits, advisories, or engineer interviews.
- **[FH-mkt]** first-hand company blog. Treat these as first-hand but promotional: numbers are self-reported and unaudited.
- **[2nd]** secondary reporting or commentary.
- **[snippet]** wording seen only in a search-result summary and not yet checked on the page. Treat it as unverified.
- Quote provenance: text marked "(verbatim, gh api)" was read directly from GitHub with `gh api`. Other blog quotes came through the WebFetch extractor, which is reliable but not guaranteed exact. They are tagged "(via WebFetch)" until checked against the raw page.

## 1. Stripe Minions: from task to PR, devboxes, rules files, CI, human review, numbers, limits

### Takeaway
Minions are unattended agents built on a fork of Block's goose, using pre-warmed devboxes isolated from production and unrestricted internet access. A typical run produces a CI-passing PR, but the harness stops after at most two CI rounds even if failures remain: handoff is not proof of green CI. Stripe self-reports over 1,000 merged PRs/week on 2026-02-09 and over 1,300 on 2026-02-19; these posts give no defect, revert or acceptance rate. ([Part 1, 2026-02-09](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents.md); [Part 2, 2026-02-19](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2.md))

### Cited Findings
**Primary sources:**
- [FH-mkt] Alistair Gray, "Minions: Stripe's one-shot, end-to-end coding agents" (Part 1), stripe.dev, **2026-02-09**. Full Markdown fetched and read on 2026-10-14. ([Original Markdown](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents.md))
- [FH-mkt] Alistair Gray, "Minions … Part 2", stripe.dev, **2026-02-19**. ([Original Markdown](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2.md))

**Task to PR:**
- A run "starts in a Slack message and ends in a pull request which passes CI and is ready for human review, with no interaction in between" (via WebFetch) ([Stripe Part 1, 2026-02-09](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- **Entry points.** Engineers usually tag the Slack app "directly from the thread discussing a change" (via WebFetch). Stripe also provides "CLI and web interfaces for initiating minions" (via WebFetch). Other internal tools can launch a minion too: the docs platform, the feature-flag platform, and the ticketing UI. Automated flaky-test tickets have a "start a minion" button ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- The minion creates a branch and a PR that follows Stripe's PR template. An engineer can then request review from another engineer or give the minion more instructions ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))

**Devboxes:**
- Devboxes "are pre-warmed so one can be spun up in 10 seconds" (via WebFetch). They are "isolated from production resources and the internet," which lets minions run "without human permission checks" (via WebFetch). Because each run has its own devbox, parallel runs need no git worktrees, which "wouldn't scale at Stripe" (via WebFetch) ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- "A Stripe devbox is an AWS EC2 instance that contains our source code and runs services under development" (via WebFetch) ([Stripe Part 2, 2026-02-19](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- **Pre-warming.** A pool of devboxes is kept ready, with large git repos already cloned and Bazel and type-checking caches warmed ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- **Isolation.** Devboxes run in the QA environment, so minions have no access to "real user data, Stripe's production services, or arbitrary network egress" (via WebFetch). Any mistake stays inside one devbox, so agents run with full permissions and no confirmation prompts. Devboxes are "cattle, not pets" (via WebFetch) ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Harness and blueprints:**
- **Harness.** Stripe forked Block's goose in late 2024 and adapted it for unattended runs. The fork has no interruptibility or other human-facing features ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)). The orchestration "interleave[s] agent loops and deterministic code" for git operations, linters and tests (via WebFetch) ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- **Blueprints.** "Blueprints are workflows defined in code that direct a minion run" (via WebFetch). Deterministic nodes such as "Run configured linters" and "Push changes" "don't invoke an LLM at all - they just run code" (via WebFetch). Agent nodes such as "Implement task" and "Fix CI failures" have latitude. The result "ends up looking like a state machine that intermixes deterministic code nodes and free-flowing agent nodes" (via WebFetch) ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Rules files:**
- Minions read "the same coding agent rule files that human-operated tools such as Cursor and Claude Code do" (via WebFetch). "almost all agent rules at Stripe are conditionally applied based on subdirectories" (via WebFetch). Before a run starts, MCP tools are run on likely links to pre-load context ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- Global rules are used sparingly, because otherwise "the agent's whole context window would fill with rules before the agent even starts" (via WebFetch). Most rules are "scoped to specific subdirectories or file patterns" and attached as the agent moves through the filesystem (via WebFetch). Stripe standardised on **Cursor's rule format**, still reads an older homegrown format, and syncs Cursor rules into a format Claude Code can read. The post mentions CLAUDE.md and AGENTS.md only as examples of rule formats, and gives no size limits ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Tools (MCP and Toolshed):**
- Toolshed is Stripe's central internal MCP server. Part 1 says it "hosts more than 400 MCP tools" (via WebFetch) ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents)). Part 2 says "nearly 500" tools ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- "Agents perform best when given a 'smaller box' with a tastefully curated set of tools" (via WebFetch). Each minion gets a small default subset, and users can add more. An internal security-control framework blocks destructive tool actions ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**CI and feedback loops:**
- **"Shift feedback left."** Known CI failures should be "enforced in the IDE" (via WebFetch). Pre-push hooks fix common lint issues, often in "well under a second" (via WebFetch). Minions run a subset of linters locally before pushing ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- **Part 1 version.** A local executable runs selected lints on each push in "less than five seconds" (via WebFetch). CI selects tests from "over three million" (via WebFetch). Failures that have autofixes are fixed automatically, and the rest go back to the minion. The cap is "we only have at most two rounds of CI" (via WebFetch) ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- **Part 2 version.** After the first push, CI runs and applies autofixes. Remaining failures go to an agent node for one local fix. After the second push and CI run, the branch goes to a human. The reason for stopping at two rounds is "diminishing marginal returns" (via WebFetch) ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Human review and "done":**
- Every PR is human-reviewed. The stated north star is "a pull request produced without any human code" (via WebFetch) ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- A run counts as done when the branch has been through the second push and CI run and is handed to the human operator for "manual scrutiny" (via WebFetch) ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Numbers:**
- **Part 1.** "Over a thousand pull requests merged each week" are completely minion-produced. The codebase has "hundreds of millions of lines of code" (via WebFetch) ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- **Stripe on X** [FH, snippet]: "Over a thousand pull requests merged each week at Stripe are completely minion-produced, and while they're human-reviewed, they contain no human-written code." ([Stripe on X](https://x.com/stripe/status/2021273907680997439)) (date not verified)
- **Part 2.** Over 1,300 PRs a week are "completely minion-produced, human-reviewed, but containing no human-written code" (via WebFetch) ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- **No quality metrics.** Neither post gives merge rate, defect rate, revert rate or cost in dollars. Part 1 gives only the weekly count of merged PRs ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents); [Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Limits Stripe states:**
- The codebase is Ruby with Sorbet plus many homegrown libraries "natively unfamiliar to LLMs" (via WebFetch). Runs may fail: "if they don't, then it's key to give agents feedback" (via WebFetch). A partly wrong run "is often still an excellent starting point" (via WebFetch). Interactive planning still happens in Claude and Cursor ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents))
- Part 2 describes no incidents. It names only these limits: local runs cannot cover the full test suite, extra CI rounds cost money, and tools must be curated ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))

**Later Stripe leads retained from the earlier notes (verification follows):**
- The stripe.dev listing shows later agent posts: "Stripe's Payment Method Factory: Orchestrating agents for repeated, custom integrations" (2026-09-30), "Harbor: Stripe's AI-assisted prototyping tool" (2026-09-14), "AI didn't write our SDK. It changed how we built it." (2026-08-28) and "You can't whisper at an AI agent" (2026-05-14) ([stripe.dev/blog listing](https://stripe.dev/blog)). TODO: not yet researched.

**Instruction loading and enforcement [FH-mkt], James Beswick and Peter Epsteen, 2026-05-14.**
- About a dozen integration-steering experiments found SDK/dependency-directory hints and API warnings were ignored. Explicit compatibility errors caused agents to correct outdated requests. Modular skills outperformed one monolithic skill by roughly 10% on Stripe's eval suite and used fewer tokens; the post does not disclose enough of that suite for independent replication. These are integration experiments, not Minion defect statistics. ([Steering experiments, 2026-05-14](https://stripe.dev/blog/ai-steering-experiments.md))

**Later practice verified: Payment Method Factory, 2026-09-30 [FH-mkt].**
- David Dunne, Xenofon Vourliotis and Sai Samant report three new integrations and ten migrations, reducing up-to-six-month integrations to two–six weeks. A two-engineer, one-task comparison reports 20 days with vanilla Claude Code versus four with reusable prompts; it is not a controlled benchmark. ([Original post, 2026-09-30](https://stripe.dev/blog/stripes-payment-method-factory-orchestrating-agents-for-repeated-custom-integrations.md))
- Over 100 reusable prompts start from prior implementations and human refinement. An observer reads implementation transcripts for wasted investigation, failed tools and red CI. Learning logs, decision logs and PR review comments feed later prompt improvements. ([Same source, 2026-09-30](https://stripe.dev/blog/stripes-payment-method-factory-orchestrating-agents-for-repeated-custom-integrations.md))
- Planning interviews the engineer about requirements, exports shared context, and maps each discrete step to one PR. An LLM orchestrator sometimes implemented work itself and consumed its context; coordination moved into code: “predictable coordination is better implemented in code.” ([Same source, 2026-09-30](https://stripe.dev/blog/stripes-payment-method-factory-orchestrating-agents-for-repeated-custom-integrations.md))
- Agents run provider-sandbox scenarios, check requests, responses, state changes and events, then record immutable response contracts instead of handwritten mocks. Review remains a bottleneck; UI testing still has manual steps. ([Same source, 2026-09-30](https://stripe.dev/blog/stripes-payment-method-factory-orchestrating-agents-for-repeated-custom-integrations.md))

**Spec and architecture practice: extensibility SDK [FH-mkt], Mike North, 2026-08-28.**
- A working prototype let stakeholders evaluate developer experience and boundaries before committing scope. Architecture considered abstractions across scripts, workflows, custom objects, SDK generation, runtime enforcement and tooling. ([SDK post, 2026-08-28](https://stripe.dev/blog/ai-didnt-write-our-sdk-it-changed-how-we-built-it.md))
- Delegation proceeds through bounded work, agent-drafted detailed specs, human review as GitHub issues, implementation with domain skills, and review against architectural intent. One issue specifies an ESLint rule's name, purpose, spec sections, severity, autofix, type-checker requirement, applicable files and exclusions. “Review those specs as GitHub issues before implementation begins” is the explicit stage boundary. Roughly twenty rules enforce sandbox, import and type constraints. ([Same source, 2026-08-28](https://stripe.dev/blog/ai-didnt-write-our-sdk-it-changed-how-we-built-it.md))
- Repo-local guidance records architectural invariants and anti-patterns. The author acknowledges generated code and architectural claims could appear locally reasonable while violating deeper invariants. Demos, hands-on use and targeted tests informed later refinement. ([Same source, 2026-08-28](https://stripe.dev/blog/ai-didnt-write-our-sdk-it-changed-how-we-built-it.md))

**Design/prototype stage: Harbor [FH-mkt], Cristian Rivera and Sai Samant, 2026-09-14.**
- Designers use Stripe design systems and realistic mock data to create interactive prototypes for stakeholder evaluation. The agent compiles in-browser, inspects the rendered result, clicks/types/scrolls, changes viewports and screenshots; a published snapshot is shareable. These are prototype-validation practices, not Minion production-merge gates. ([Harbor, 2026-09-14](https://stripe.dev/blog/harbor-stripes-ai-assisted-prototyping-tool.md))

### Inferences
- Stripe's quality control is mostly structural. The deterministic blueprint nodes (lint, push, CI) do not depend on the model behaving well. The devbox sandbox bounds the damage. Humans are the final gate. Instructions alone do not carry the load.
- Capping CI at two rounds sets an economic limit on iteration. **Correction:** the stop condition is a handoff after at most two CI runs; a CI-passing PR is the typical outcome, not a guaranteed acceptance condition. This distinction follows directly from the bounded failure loop. ([Part 1, 2026-02-09](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents.md); [Part 2, 2026-02-19](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2.md))
- The headline numbers measure throughput: merged PRs that contain no human-written code. They do not measure quality. The rise from 1,000 to 1,300 a week happened within 10 days of posting, so it may reflect a different counting window. That is unverified.

### Gaps
- Not found in the two posts: defect, revert or incident rates for minion PRs, how review is staffed, reviewer load, and cost per PR.
- No independent or critical evaluation of Minions was found yet. Search was limited to "Stripe Minions coding agents stripe.dev". TODO: not yet researched: HN discussion, InfoQ (2026-03) at https://infoq.com/news/2026/03/stripe-autonomous-coding-agents/, talks or podcasts by Stripe engineers, and the Sept 2026 Payment Method Factory post.
- TODO: check the via-WebFetch quotes against the raw `.md` versions of the posts.

## 2. Cloudflare workers-oauth-provider: how Kenton Varda worked with Claude, what review found, lessons, critique, later practice

### Takeaway
Kenton Varda prompted Claude with detailed, design-level instructions and recorded each prompt, and his own manual fixes, in the commit messages from 2025-02-27 onward. His own review caught a real crypto flaw (a key-wrapping construction) and several bugs Claude could not fix. Two medium-severity CVEs still shipped and were published on 2025-05-01: a missing `redirect_uri` check at the authorize endpoint and a PKCE downgrade bypass. Critics such as Neil Madden (2025-06) judged the tests "woefully inadequate" and doubted the line-by-line review claim. By 2026 the repo is maintained by others under a detailed AGENTS.md. It has CI, an MCP conformance suite, an AI review bot ("Bonk"), and a rule that only humans merge releases.

### Cited Findings
**Who and when:**
- [FH] The first commit is 3b2ae809, 2025-02-27, by Kenton Varda: "Have Claude write an OAuth provider implementation." The message contains the full prompt, starting: "We are going to write a TypeScript library that implements the provider side of the OAuth 2 protocol. The library is intended to be used on Cloudflare Workers." The prompt then gives a complete example Worker with commented config (`apiRoute`, `apiHandler`, `defaultHandler`, `authorizeEndpoint`, `tokenEndpoint`, `clientRegistrationEndpoint`) explaining what each does (verbatim, gh api) ([commit 3b2ae809](https://github.com/cloudflare/workers-oauth-provider/commit/3b2ae809))
- [FH] The early history is a series of small, one-prompt commits. Titles include "Ask Claude to implement PKCE support (for OAuth 2.1)." (2025-03-04), "Ask Claude to implement the rest of OAuth 2.1." (2025-03-04) and "Ask Claude to hash the auth code and client secret as well." (2025-02-28). Manual commits are labelled, for example "Manually fix bug propagating `encryptedProps` to access token record." (2025-03-07) (verbatim, gh api) ([commit history](https://github.com/cloudflare/workers-oauth-provider/commits/main))

**What Kenton's review caught:**
- [FH] **Key-wrapping crypto flaw** (adcbb5de, 2025-03-05). Kenton's prompt to Claude: "There's a security flaw in the way you wrap keys for tokens: You used a SHA-256 hash of the token as the key material for the wrapping. However, SHA-256 is also how we compute "token IDs". With this construction, someone would be able to unwrap the keys using only the token ID, which is stored alongside the wrapped keys, hence all keys can be trivially unwrapped." Claude's fix used PBKDF2 with 100,000 iterations, which Kenton rejected as needlessly expensive for high-entropy input. He asked for an HMAC-SHA-256 with a static key instead and supplied 32 random bytes himself. He had also judged the overall output "beautiful code with one big flaw." (verbatim, gh api) ([commit adcbb5de](https://github.com/cloudflare/workers-oauth-provider/commit/adcbb5de9c24f5b6a7dbea2e0a313a87c304d9bb))
- [FH] **Human design intervention, subsequently revised** (cb94470b, 2025-03-04). "Claude started work, but I stopped it when it implemented the new requirement that refresh tokens must be rotated every time they are used. This requirement makes it seemingly impossible to write a reliable client…" Kenton also rejected unnecessary `access_token` query-parameter rejection and an "overly verbose" README section (verbatim, gh api) ([commit cb94470b](https://github.com/cloudflare/workers-oauth-provider/commit/cb94470b))
- [FH] **Bug Claude could not fix** (bf729a54, 2025-03-04): "THIS CODE HAS A BUG. I repeatedly prompted Claude to fix the bug, but it kept getting it wrong. I will fix manually in the next commit." Next commit, 60197d5e (2025-03-05): "Claude had a bug in the previous commit. I prompted it multiple times to fix the bug but it kept doing the wrong thing. So this change is manually written by a human." (verbatim, gh api) ([commit bf729a54](https://github.com/cloudflare/workers-oauth-provider/commit/bf729a54); [commit 60197d5e](https://github.com/cloudflare/workers-oauth-provider/commit/60197d5e))
- [FH] **Claude made things worse** (9c389439, 2025-02-28). "Claude responded to this by making the code even more messed up". When it did fix the bug, "It also re-ordered some declarations, even though I told it not to" and "fixed an unrelated bug … I hadn't noticed this before, but it was indeed a bug." Kenton concluded: "Oh well, let's accept it and move on." (verbatim, gh api) ([commit 9c389439](https://github.com/cloudflare/workers-oauth-provider/commit/9c389439))
- [FH] **Over-complicated crypto fix** (d389c262, 2025-03-07). Asked to fix an AES-KW error, Claude "decided the best solution was to switch away from AES-KW and use AES-GCM instead … This made the code quite a bit more complicated and made the wrapped keys larger." Kenton changed one parameter ('jwk' to 'raw') by hand (verbatim, gh api) ([commit d389c262](https://github.com/cloudflare/workers-oauth-provider/commit/d389c262))
- [FH] **Tool friction** (0f81f7c1, 2025-02-28): "Claude seems to be having trouble applying its edits to the file, so I finished up the task from the previous commit." (verbatim, gh api) ([commit 0f81f7c1](https://github.com/cloudflare/workers-oauth-provider/commit/0f81f7c1))
- [FH] **Platform mistake** (865e7b9e, 2025-02-28). Claude used Node's `createHash`, and Kenton's prompt: "But since this is Cloudflare Workers, we should use WebCrypto instead." (verbatim, gh api) ([commit 865e7b9e](https://github.com/cloudflare/workers-oauth-provider/commit/865e7b9e))

**Kenton's stated claims and lessons:**
- [FH] The original "Written using Claude" section, now kept in HISTORY.md: "This library (including the schema documentation) was largely written with the help of Claude … Claude's output was thoroughly reviewed by Cloudflare engineers with careful attention paid to security and compliance with standards. Many improvements were made on the initial output, mostly again by prompting Claude (and reviewing the results)." (verbatim, gh api) ([HISTORY.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/HISTORY.md))
- [FH] Also from HISTORY.md: "two months ago (January 2025), I (@kentonv) would have agreed. I was an AI skeptic. … I started this project on a lark, fully expecting the AI to produce terrible code for me to laugh at. And then, uh... the code actually looked pretty good. Not perfect, but I just told the AI to fix things, and it did." And: "To emphasize, **this is not "vibe coded"**. Every line was thoroughly reviewed and cross-referenced with relevant RFCs, by security experts with previous experience with those RFCs." (verbatim, gh api) ([HISTORY.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/HISTORY.md))
- [FH, snippet] Kenton Varda on X: "The commit history has all my prompts if you're interested. When I started this I was actually intending to prove that LLMs can't code, becaues I was a huge skeptic. Ended up transforming my whole world view." ([X post](https://x.com/KentonVarda/status/1904548316697829594)) (date to be derived)
- [FH, snippet] Search summaries attribute two more statements to Kenton. One, on X, is that he reviewed every line "caerfully, and so did our security team". The other, in a Cloudflare TV interview, is "there was actually, there was a bug that, that, that I missed", which he called his fault ([Cloudflare TV](https://cloudflare.tv/this-week-in-net/ai-writes-code-kenton-varda-on-trust-review-and-why-workers-is-the-best-ai-platform/PZ5rmMg2)). TODO: verify, with date and timestamp.

**What still shipped (CVEs):**
- [FH] **GHSA-4pc9-x2fx-p7vj / CVE-2025-4143, medium, published 2025-05-01**: "The OAuth implementation failed to check that redirect_uri was among the allowed set for the client_id." The attack needs an app that auto-approves previously authorized clients, "However, many applications do implement such logic." The advisory's note reads: "It is a basic, well-known requirement … workers-oauth-provider implemented only the latter check, not the former. Unfortunately, the former is the much more important check. … The author of this library would like everyone to know that he was, in fact, well-aware of this requirement, thought about it a lot while designing the library, and then, somehow, forgot to actually make sure the check was in the code." Fixed in 0.0.5 by PR #26 (verbatim, gh api) ([GHSA-4pc9-x2fx-p7vj](https://github.com/cloudflare/workers-oauth-provider/security/advisories/GHSA-4pc9-x2fx-p7vj))
- [FH] **GHSA-qgp8-v765-qxx9 / CVE-2025-4144, medium, published 2025-05-01**, "PKCE bypass via downgrade attack": "it was found that an attacker could cause the check to be skipped. … This bug completely bypasses PKCE protection." Fixed in 0.0.5 by PR #27. The advisory credits no one (verbatim, gh api) ([GHSA-qgp8-v765-qxx9](https://github.com/cloudflare/workers-oauth-provider/security/advisories/GHSA-qgp8-v765-qxx9))

**Patch inspection and review limits, verified 2026-10-14 [FH-code].**
- Both security fixes were authored by Glen Maddern (`geelen`) and merged on **2025-04-30**, before the **2025-05-01** advisories. PR #26 adds authorize-time registered-redirect validation and a negative test that rejects an attacker redirect and checks that no grant was created. PR #27 rejects a verifier when the authorization flow had no challenge, and adds a downgrade regression test. This identifies the patch author, not necessarily the discoverer. ([PR #26, 2025-04-30](https://github.com/cloudflare/workers-oauth-provider/pull/26); [PR #27, 2025-04-30](https://github.com/cloudflare/workers-oauth-provider/pull/27))
- **Human judgement also needed correction:** after stopping strict refresh rotation on March 4, Kenton wrote in the next prompt: “In retrospect, I think I was wrong about the new refresh token one-time use requirement being backwards-incompatible.” The compromise and subsequent manual bug fix show iterative design, not an infallible human spec. ([bf729a54, 2025-03-04](https://github.com/cloudflare/workers-oauth-provider/commit/bf729a54b8c8556b0d55b2ae1c19baed3746909a))
- **Current snapshot pinned:** all current-repo observations refer to `602610ce1c11fdb412388713f66a4ca2a45c110d`, committed **2026-10-07**, the latest commit returned before the research cutoff. AGENTS.md is guidance, not proof that each run obeyed it. The actual CI YAML and conformance README were read as text; no downloaded repository code was executed. ([Snapshot](https://github.com/cloudflare/workers-oauth-provider/commit/602610ce1c11fdb412388713f66a4ca2a45c110d); [CI](https://raw.githubusercontent.com/cloudflare/workers-oauth-provider/602610ce1c11fdb412388713f66a4ca2a45c110d/.github/workflows/ci.yml); [Conformance](https://raw.githubusercontent.com/cloudflare/workers-oauth-provider/602610ce1c11fdb412388713f66a4ca2a45c110d/conformance/README.md))

**Community critique:**
- [2nd, named expert] Neil Madden, "A look at CloudFlare's AI-coded OAuth library", neilmadden.blog, **2025-06-06** (date from URL):
  - Verdict: "As a first cut of an OAuth library, it's not bad, but I wouldn't really recommend it for use yet."
  - Tests: "woefully inadequate for what I would expect of a critical auth service". He also expects "probably quite a few missing MUST checks, particularly around validating parameters."
  - Specific findings:
    - CORS handling looks "really suspicious"; he notes the commit log shows humans chose this approach.
    - Standard security headers are missing (nosniff, HSTS).
    - The deprecated implicit grant was implemented for public clients: "absolutely not needed".
    - Basic auth is implemented incorrectly, since it does not URL-decode first, plus a colon-in-secret bug.
    - Token ID generation "generates biased output". It reduces entropy, but not enough to make tokens brute-forceable, and "the LLM spat it out in the very first commit".
  - On review: he doubts the "every line was thoroughly reviewed" claim ("I don't think they did"), citing "no sign of any code review at all" for the first day's commits.
  - Wider view: "you need to have a clear idea in your head of the kind of code you're expecting the LLM to produce to be able to judge whether it did a good job." (all via WebFetch) ([Neil Madden](https://neilmadden.blog/?p=3982))
- [2nd, snippet] Ory, "Cloudflare, please don't vibe code your own crypto": "for the core, security-defining elements of your infrastructure, LLMs are not yet a replacement for expert human design, painstaking implementation, and continuous scrutiny by security professionals." ([Ory blog](https://www.ory.com/blog/cloudflare-please-dont-vibe-code-your-own-crypto)). Date and author TODO.
- [2nd] BigGo News (2025-06-02) summarised the Hacker News debate ([BigGo](https://biggo.com/news/202506021914_Cloudflare_AI_OAuth_Library_Debate)). TODO: the HN thread itself, with named commenters.

**Current practice in the same repo (as of 2026-10):**
- [FH] The AGENTS.md overview says: "This library was largely written with Claude AI assistance, with all code thoroughly reviewed by Cloudflare security engineers." (verbatim, gh api) ([AGENTS.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/AGENTS.md))
- [FH] AGENTS.md also specifies:
  - **Security invariants.** "Secrets (tokens, authorization codes) are stored as SHA-256 hashes only"; "Props are encrypted with AES-GCM, key wrapped with the token itself"; "PKCE enforced for public clients (S256 method)"; "Redirect URIs validated against XSS payloads".
  - **RFC list.** OAuth 2.1 draft-13, RFC 6750, 7009, 7591, 7636, 8414, 8693, 8707, 9207, 9728, and CIMD.
  - **Spec rule.** "always check the latest published MCP specification (not drafts)".
  - **Done rule.** "Run `npm run check` before considering work done"; "Add tests for new functionality".
  - **Ask first.** New dependencies, KV schema changes, OAuth endpoint or flow changes, new feature flags.
  - **Never.** Includes "Store unhashed tokens or secrets in KV" and "Merge, approve, or enable auto-merge on a Version Packages PR: releases are for humans only".
  - **CI on every PR.** `npm ci`, build, `npm run check` (typecheck plus tests), and a Prettier check; "All checks must pass before merge."
  - **AI review.** "Mention `/bonk` or `@ask-bonk` in PR comments to get AI-powered code review and suggestions. Bonk can analyze code, suggest fixes, and even auto-commit improvements."
  - **Semver.** Changes that invalidate tokens "must be released as a minor version bump".

  (verbatim, gh api) ([AGENTS.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/AGENTS.md))
- [FH] Since 2026-07, testing has grown. There is a black-box MCP authorization conformance suite running a real Worker in workerd ("test: add Workerd MCP authorization conformance suite (#275)", 2026-07-30). The official MCP conformance runner runs in CI ("#384", 2026-09-25). The defaults were hardened: "feat: require S256 PKCE by default (#277)" (2026-07-31) and "feat: enforce the MCP redirect URI policy by default (#340)" (2026-09-24) (verbatim, gh api) ([commit history](https://github.com/cloudflare/workers-oauth-provider/commits/main))
- [FH] Commit counts as of 2026-10-14: 334 commits. By author: "Matt" 105, Kenton Varda 88, Sunil Pai 46, github-actions[bot] 31, Glen Maddern 26, "agent-think[bot]" 4, others ≤3 each. Kenton's commits are the 2025 core. Recent work (2026-07 to 2026-10) is almost all PRs by "Matt" ([GitHub API commit list](https://github.com/cloudflare/workers-oauth-provider/commits/main))

**Later Cloudflare agent practice: vinext (only search summaries so far, unverified):**
- [snippet] In "How we rebuilt Next.js with AI in one week" (reported as **2026-02-24**), one engineer, Steve Faulkner, built vinext with AI for about $1,100 in tokens; "Almost every line of code in vinext was written by AI." [snippet] Vercel's Guillermo Rauch reported "2 critical, 2 high, 2 medium, 1 low security vulnerabilities". [snippet] Hacktron reported fetch-cache keys that ignored request headers, so different users could receive the same cached response ([Cloudflare blog (vinext)](https://blog.cloudflare.com/vinext/); [Cybernews](https://cybernews.com/security/hackers-find-critical-flaws-in-cloudflares-nextjs-alternative/); [Hacktron](https://www.hacktron.ai/blog/hacking-cloudflare-vinext); [InfoQ 2026-03](https://infoq.com/news/2026/03/cloudflare-vinext-experimental)). TODO: not yet researched against primary pages.

### Inferences
- Kenton's method works as a spec plus a series of small, single-purpose prompts. One expert holds the design in his head, and each commit is one reviewable step. The record suggests the reviewer's own expertise caught design and crypto flaws: key-wrapping, PBKDF2 cost, AES-KW, and the refresh-rotation trade-off. A "forgot to check" omission (redirect_uri) and a PKCE downgrade still got through. Line-by-line review of generated code did not replace negative tests for spec MUSTs.
- The repo later moved its trust from "an expert reviewed it" to machine-checked conformance (a black-box MCP conformance suite in CI), secure-by-default options, and explicit human-only gates for one-way doors such as npm releases.

### Gaps
- Not found yet: who discovered CVE-2025-4143/4144 (the advisories credit nobody), and whether those bugs trace to Claude-generated or human-written lines. TODO: inspect PR #26 and #27.
- TODO: not yet researched: the Hacker News thread (June 2025) with named commenters; Max Mitchell's "I read all of Cloudflare's Claude-generated commits" (found earlier as a lead, not yet located); the Cloudflare TV interview transcript; what "Bonk" is (Cloudflare's AI review bot); other Cloudflare agent-practice posts from 2026 (vinext, AI code review).

## 3. Cursor: internal practice (rules, review, Bugbot), long-running agent experiments, and what the browser achieved

### Takeaway
Cursor's internal gate for agent-written code is review automation. Bugbot runs "on all internal code". Since 2026-03 a security-review agent acts as a blocking CI gate, with 3,000+ internal PRs a week reviewed, as reported by Cursor. The long-running experiments (2026-01 and 2026-02) were research runs. Hundreds to thousands of planner and worker agents produced over 1M lines of Rust for the FastRender browser in about a week. Cursor deliberately accepted a "small and constant" error rate rather than a green build per commit. Critics showed that recent commits did not compile and CI mostly failed. A human fixed the build before outsiders could run it, and the "from scratch" claim was contested.

### Cited Findings
**Long-running experiment ("Scaling long-running autonomous coding"):**
- [FH-mkt] Wilson Lin, cursor.com, **2026-01-14**.
- Scale: "hundreds" of concurrent agents. The browser run lasted "close to a week" and produced over 1M lines across about 1,000 files.
- Other projects:
  - Solid-to-React migration in Cursor's own codebase: +266K/-193K over 3 weeks, which "still needs careful review".
  - Java LSP: 7.4K commits, 550K lines of code.
  - Windows 7 emulator: 14.6K commits, 1.2M lines of code.
  - Excel clone: 12K commits, 1.6M lines of code.
- Video-rendering code that is reportedly "25x faster" was merged.
- Total: "trillions of tokens" (via WebFetch) ([Cursor, Scaling agents](https://cursor.com/blog/scaling-agents))
- **Coordination failures.** Shared-file locks failed: "Agents would hold locks for too long, or forget to release them entirely" and "Twenty agents would slow down to the effective throughput of two or three". Under optimistic concurrency without hierarchy, agents grew risk-averse. A separate "integrator" role "created more bottlenecks than it solved". The system still needs "periodic fresh starts to combat drift and tunnel vision" (via WebFetch) ([Cursor, Scaling agents](https://cursor.com/blog/scaling-agents))
- **Design that worked.** Planners explore the codebase and create tasks, possibly through sub-planners. Workers "just grind on their assigned task until it's done, then push their changes". A judge decides at the end of each cycle whether to continue (via WebFetch) ([Cursor, Scaling agents](https://cursor.com/blog/scaling-agents))
- **Models.** GPT-5.2 is "much better at extended autonomous work". Opus 4.5 "tends to stop earlier and take shortcuts when convenient". GPT-5.2 is a better planner than GPT-5.1-Codex. Cursor now picks a model per role. "The harness and models matter, but the prompts matter more." (via WebFetch) ([Cursor, Scaling agents](https://cursor.com/blog/scaling-agents))
- **Quality evidence.** The post gives no test pass rates or compile status for the browser, and makes no claim that it renders pages correctly. Per the extractor, the judge agent is the only quality gate described ([Cursor, Scaling agents](https://cursor.com/blog/scaling-agents))

**Follow-up ("Towards self-driving codebases"):**
- [FH-mkt] Wilson Lin, cursor.com, **2026-02-05**.
- **Structure.** A root planner "owns the entire scope of the user's instructions" and "does no coding itself". Sub-planners split the work recursively. Workers "are solely responsible for driving them to completion" and hand off "notes, concerns, deviations, findings". The judge was removed because agents were "reasonably good at following instructions to completion".
- **Throughput.** "~1,000 commits per hour across 10M tool calls over a period of one week" (via WebFetch) ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))
- **Correctness policy.** Requiring full correctness before each commit caused serialisation, so they accepted: "The error rate remains small and constant, perhaps rarely completely clean but steady and manageable." They propose a "green" branch where an agent "regularly takes snapshots and does a quick fixup pass before release" (via WebFetch) ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))
- **Spec lessons:**
  - "Agents have immense engineering skill but will follow instructions to the end, good or bad."
  - A flawed initial spec produced an architecture "unfit to evolve into a full browser".
  - Agents pulled in unwanted dependencies until a dependency policy was stated.
  - "Constraints are more effective than instructions"; a concrete range ("Generate 20-100 tasks") beat "generate many tasks".
  - The browser "was not intended to be used externally and we expected the code to have imperfections". A restructured run was "in a heavily broken state" but "converged toward working code in a few days".

  (via WebFetch) ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))

**Wilson Lin interview (Simon Willison, "FastRender", 2026-01-23):**
- [FH interview + 2nd commentary] About 2,000 concurrent agents at peak, about 300 per machine. Models: Opus 4.5, GPT-5.1 and GPT-5.2. The project took about 3 weeks, with the longest uninterrupted run about a week. On intervention: "The only thing you can do is stop it" and "We don't alter the trajectory while executing."
- Verification: "you can get a lot of verification just from compilation" (Rust). Screenshot diffs were compared "against a golden sample". Specs (csswg-drafts, tc39-ecma262, whatwg-dom, whatwg-html) were included as git submodules.
- Acknowledged trade-offs: "It's a stable rate of errors." "We never intended for it to be a production software or usable."
- Dependencies: Skia, HarfBuzz and Taffy (vendored). QuickJS was pulled in to unblock JavaScript work.
- Simon's assessment: "the extreme edge of what a single engineer can achieve in early 2026" (via WebFetch) ([Simon Willison, 2026-01-23](https://simonwillison.net/2026/Jan/23/fastrender/))

**Critique:**
- [2nd, named HN users] HN thread on emsh's "Cursor's latest 'browser experiment' implied success without evidence" (mid-January 2026). The submitter, embedding-shape, reported that across the last 100 commits "every single of them failed in some way".
- paulus_magnus2: "CI success was not part of PR accept criteria at any point."
- Wilson Lin (wilsonzlin): "The experimental harness can occasionally leave the repo in an incomplete state but does converge". On the time spent: "those 2 hours were fixing compile errors and bugs, not compile time". He also said the vendored JS parser was "a copy of my personal JS parser project vendored to make it easier to commit to".
- Critics of "from scratch": nindalf ("just calling out to code written by humans") and acdha ("'From scratch' is inarguably wrong").
- Defence, M4v3R: Servo crates are used only for tokenisation and selector matching, while JS, DOM and layout were new.
- After fixes, Snuggly73: "The latest commit now builds _and_ runs", but "I couldn't make it render the apple page that was on the Cursor promo". thegeomaster found the Excel clone UI "buggy to the point of being unusable" (via WebFetch) ([HN 46646777](https://news.ycombinator.com/item?id=46646777)); article: [emsh.cat](https://emsh.cat/en/cursor-implied-success-without-evidence/) (not yet read)
- [2nd, snippet] Other reporting:
  - The Register (2026-01-22), "Cursor shows AI agents capable of shoddy code at scale", and an opinion piece (2026-01-26), "Cursor is better at marketing than coding". Coverage quotes a critic saying "An 88 percent job failure rate is very high" about FastRender's GitHub Actions.
  - Reports cite 34 compile errors ("could not compile 'fastrender' (lib) due to 34 previous errors").
  - Fortune (2026-01-23) said these are not production-ready systems.

  Sources: [The Register 2026-01-22](https://www.theregister.com/2026/01/22/cursor_ai_wrote_a_browser/); [The Register 2026-01-26](https://theregister.com/2026/01/26/cursor_opinion); [Fortune 2026-01-23](https://fortune.com/2026/01/23/cursor-built-web-browser-with-swarm-ai-agents-powered-openai). TODO: verify.

**Internal review practice:**
- [FH-mkt] Jon Kaplan, "Building a better Bugbot", cursor.com, **2026-01-15**:
  - Why: "As coding agents became more capable, we found ourselves spending more time on review." "We also run Bugbot on all internal code at Cursor."
  - Metric: "resolution rate" means "It uses AI to determine, at PR merge time, which bugs were actually resolved by the author in the final code". Spot-checks with PR authors found "nearly all" classified correctly. The rate went from 52% to over 70% across 40 major experiments, and bugs flagged per run from 0.4 to 0.7.
  - Scale: "Bugbot reviews more than two million PRs per month."
  - Architecture: originally 8 parallel passes with randomised diff order and majority voting, then an agentic design from fall 2025 ("largest gains"). There are also "Bugbot rules" for codebase-specific checks, for example unsafe migrations. Offline evaluation uses BugBench, a set of real diffs with human-annotated bugs (via WebFetch) ([Cursor, Building Bugbot](https://cursor.com/blog/building-bugbot))
- [FH-mkt] Travis McPeak, "Securing our codebase with autonomous agents", cursor.com, **2026-03-16**:
  - Velocity: "Over the last nine months, our PR velocity has increased 5x." A graphic states: "Security agents are reviewing 3,000+ internal PRs each week, catching 200+ vulnerabilities."
  - Agents:
    - Agentic Security Review: progressed from a private Slack channel to PR comments to a blocking CI gate.
    - Vuln Hunter: segments the codebase, and the team triages its findings.
    - Anybump: dependency patching with reachability analysis, opening PRs only after tests pass, with canary deploy as "a final safety gate".
    - Invariant Sentinel: runs daily to check for drift from security and compliance properties.
  - Deduplication: a Gemini Flash 2.5 classifier.
  - Why not Bugbot: a general-purpose tool "can't be prompt-tuned to our specific threat model".
  - Not given: false-positive rates and concrete examples of caught vulnerabilities (via WebFetch) ([Cursor, Security agents](https://cursor.com/blog/security-agents))
- [FH-mkt] Jon Kaplan, **2026-02-26**, Bugbot Autofix: resolution rate “has increased from 52% to 76%”; over 35% of Autofix changes are merged into the base PR. Fixers run in separate VMs. These are different denominators: resolved review findings versus accepted generated fixes. They are not a defect-escape rate or recall measurement. ([Bugbot Autofix](https://cursor.com/blog/bugbot-autofix))

**Internal workflow in first-hand speech: John Gilhuly, Arize Observe 2026 [FH-talk], uploaded 2026-06-19 (upload date from yt-dlp metadata).**
- At **04:48–05:09**, Gilhuly describes agent planning → human review of the plan → implementation and demo → shipping with continuous retrospectives. At **03:25–03:48**, he describes a pinned-chat feature accompanied by a video of the agent testing it. ([Talk, 2026-06-19, 04:48](https://www.youtube.com/watch?v=Lsp5YZ9Jj3Q&t=288s))
- At **12:36–13:19**, he estimates 30–40% of PRs “within our ecosystem” merge without human review; low-blast-radius changes receive automated approval, others route to an appropriate person. This is his estimate, with no denominator or escaped-defect rate supplied. It qualifies the earlier notes' blanket implication that Cursor always uses human review. ([Talk, 2026-06-19, 12:36](https://www.youtube.com/watch?v=Lsp5YZ9Jj3Q&t=756s))
- At **08:14–09:16**, he reports about 150 monorepo skills and asks: “Can we reduce the number of skills?” A “how” skill locates implementation knowledge; a “why” skill searches Slack, Notion and git history for rationale. At **15:09–15:26**, review reactions, replies and comments feed learned review rules. ([Talk, 2026-06-19, 08:14](https://www.youtube.com/watch?v=Lsp5YZ9Jj3Q&t=494s); [2026-06-19, 15:09](https://www.youtube.com/watch?v=Lsp5YZ9Jj3Q&t=909s))
- **[2nd] Sara Verdi's Arize write-up, published 2026-07-20, updated 2026-10-07**, is a secondary synthesis of that talk. Its risk-routing pseudocode is explicitly a conceptual sketch, not Cursor's disclosed production code. Its adoption checklist is editorial advice and should not be passed off as Cursor policy. ([Arize article](https://arize.com/blog/inside-cursors-agent-factory-how-it-verifies-ai-written-code/))

**H2 internal environment practice [FH-mkt], Mathew Hogan and Arvind Saripalli, 2026-07-30.**
- Cloud-agent share of merged monorepo PRs rose from roughly one in ten in December to over half in July; this measures authorship, not correctness. Ubuntu VMs reproduce local tooling, with egress restrictions, scoped/proxied git access, secret scanning and tool-output redaction. ([Environment post, 2026-07-30](https://cursor.com/blog/cloud-agent-environment))
- Skills alone only marginally helped convoluted commands. Cursor built `anydev`, a service/tool CLI with help menus and a supervisor that restarts long-running builds, removing process babysitting from the model. Working services plus computer-use and screen recording let agents demonstrate end-to-end changes. Cloud Doctor reads traces, detects unhealthy environments and fixes misleading skills or workflows. ([Same source, 2026-07-30](https://cursor.com/blog/cloud-agent-environment))

**Later swarm and cheaper-worker evidence [FH-mkt], Wilson Lin, 2026-07-20.**
- The browser was a proof of concept that “fell far short of polished software.” The new experiment implements an 835-page SQLite manual with source, tests, binary and internet withheld; hidden `sqllogictest` checks progress, followed by human inspection for shortcuts. Four-hour scores were 73–85% for new runs; every new configuration eventually passed the suite. This is a test-suite result, not SQLite production equivalence. ([July update, 2026-07-20](https://cursor.com/blog/agent-swarm-model-economics))
- Mixes include Opus 4.8 or Fable 5 planners with Composer 2.5 workers. Shared design docs with compile-checked references address conflicting designs; an independent agent resolves merges; bloated files trigger commit blocking and decomposition. Different reviewer lenses/models are stacked because “No single lens catches everything”. This revises January's claim that workers handled conflicts adequately and the integrator was unnecessary. ([Same source, 2026-07-20](https://cursor.com/blog/agent-swarm-model-economics))
- The account supports cheaper workers for specified subproblems; it does **not** test GPT Luna or Haiku, disclose a small-model ticket template, or establish parameter-size equivalence. ([Same source, 2026-07-20](https://cursor.com/blog/agent-swarm-model-economics))

**Published team practice, Lee Robinson [FH-mkt], 2026-01-09.**
- Plan Mode researches relevant code, asks requirements questions, creates a file-referenced Markdown plan and waits for approval. Plans can be saved in `.cursor/plans/`. Rules should identify commands and canonical examples, with lint replacing style-guide prose. ([Best practices, 2026-01-09](https://cursor.com/blog/agent-best-practices))
- The prescribed TDD sequence is expected input/output tests → verify they fail → review/commit tests → implementation without modifying those tests → review/commit implementation. Architecture diagrams can expose data-flow issues before review. These are published recommendations; they do not establish that every internal PR follows them. ([Same source, 2026-01-09](https://cursor.com/blog/agent-best-practices))
- **[FH-doc, publication/update date not supplied; retrieved 2026-10-14]** Cursor says its own `agents.md` teaches running/debugging monorepo microservices, and triggered skills teach service-specific testing. A custom CLI addresses forgotten arguments and distracting build logs. Actual internal files and task templates are not public in this source. ([Cloud-agent best practices](https://cursor.com/docs/cloud-agent/best-practices))

### Inferences
- Cursor makes two different bets. For production code, it adds automated reviewers (Bugbot, then a security gate) in front of human review. For research swarms, it traded per-commit correctness for throughput and pushed correctness to a later "green branch" fix-up. Critics attacked the gap between that policy and the launch messaging, not the policy itself.
- "Done" for a worker is a handoff to its planner. The posts give no machine-checked acceptance criteria for the browser beyond compilation and screenshot comparison.

### Gaps
- TODO: not yet researched:
  - Cursor's "Best practices for coding with agents" post (rules, plans, tests).
  - Arize's "Inside Cursor's agent factory" and the talk it is based on.
  - How Cursor engineers write tasks or rules internally.
  - Any 2026 H2 Cursor posts on internal practice.
  - The emsh article itself.
  - The Bugbot Autofix post date.
  - Verbatim checks of the WebFetch quotes.

## 4. Through the lens: how tasks/specs are written and checked, how work is split, what "done" means, whether a small model could follow, how reviewers/QA catch misses

### Takeaway
The three organisations sit at different points:
- **Stripe** puts the process into code. Blueprints are deterministic nodes around narrow agent nodes, with scoped rule files, curated tools, two CI rounds and mandatory human review.
- **Cloudflare (Kenton)** put the process into an expert's head. A detailed design prompt, then one prompt per commit and line-by-line expert review. The project later moved to AGENTS.md, CI and a conformance suite.
- **Cursor's swarm experiments** put the process into a planner hierarchy and prompts, and accepted a steady error rate.

None of the sources tests whether a small model could follow their instructions.

### Cited Findings
**How tasks and specs are written:**
- Stripe: tasks often come straight from a Slack thread or a ticket button, for example flaky-test tickets. Context comes from scoped rule files and MCP tools that pre-fetch linked pages, not from a written spec ([Stripe Part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents); [Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- Cloudflare: the first prompt was a full API sketch with annotated example code (commit 3b2ae809). Later prompts state design constraints and reasons, for example the refresh-token "compromise" (bf729a54) and "encrypted such that you need a valid token to decrypt" (adcbb5de) (verbatim, gh api) ([commit 3b2ae809](https://github.com/cloudflare/workers-oauth-provider/commit/3b2ae809); [commit bf729a54](https://github.com/cloudflare/workers-oauth-provider/commit/bf729a54); [commit adcbb5de](https://github.com/cloudflare/workers-oauth-provider/commit/adcbb5de9c24f5b6a7dbea2e0a313a87c304d9bb))
- Cursor: "Agents … will follow instructions to the end, good or bad". A vague spec led agents into obscure features, a flawed spec produced an architecture unfit to grow, "Constraints are more effective than instructions", and concrete ranges worked (via WebFetch) ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))

**How work is split:**
- Stripe: one minion per task, each in its own devbox. Within a run, the blueprint state machine splits deterministic steps from agent steps ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- Cloudflare: small prompt-bearing and manual commits; 55 commits dated 2025-02-27 through 2025-03-07 inclusive (recounted from the four API pages, not 45) ([commit history](https://github.com/cloudflare/workers-oauth-provider/commits/main))
- Cursor: recursive planners, sub-planners and workers. Workers run on their own repo copy and hand off to their planner ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))

**What "done" means:**
- Stripe: a CI-passing PR after at most two CI rounds, handed to a human ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- Cloudflare (2026 AGENTS.md): "Run `npm run check` before considering work done" plus new tests. Merge requires CI green. Releases are human-only (verbatim, gh api) ([AGENTS.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/AGENTS.md))
- Cursor swarm: a worker hands off. There was no per-commit green requirement; an error rate was accepted, with a proposed green-branch fix-up ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))

**Models and capability:**
- Cursor found real differences between models for long-horizon work. Opus 4.5 "tends to stop earlier and take shortcuts". GPT-5.2 plans better than GPT-5.1-Codex. Cursor chooses a model per role (via WebFetch) ([Cursor, Scaling agents](https://cursor.com/blog/scaling-agents))
- Cursor uses a small model (Gemini Flash 2.5) only for a narrow classification job: deduplicating security findings ([Cursor, Security agents](https://cursor.com/blog/security-agents))

**How misses are caught:**
- Stripe: lint, CI autofix, then a human. Cloudflare: expert review caught crypto and design flaws, but basic MUST checks still shipped as CVEs; later a conformance suite was added. Cursor: Bugbot plus a blocking security gate in production work, versus compile and screenshot checks only in the swarm, where outsiders found the failures, sources as cited in sections 1–3.

### Inferences
- **Small models.** Stripe's design pushes critical steps (lint, push, CI, test selection) out of the model and into code, which should lower the capability needed in the agent nodes. Kenton's prompts assume a strong model and a strong reviewer: Claude still failed several fixes he had to write by hand. Cursor's evidence suggests even frontier models differ sharply on long-horizon autonomy. This is an inference; no source tests small models.
- A pattern shared by all three: deterministic, machine-checkable gates (lint, CI, conformance tests, blocking security CI) were added over time because review by humans or a single model missed basic, checkable requirements.

### Gaps
- No source directly evaluates whether a small or cheap model could follow these organisations' instructions or rule files. Searches so far did not target this; TODO.
- Not found: Stripe's own rule-file contents and examples of minion task prompts.

## 5. Architecture, system design and layers (app, security, network), with examples

### Takeaway
The material is strongest on the execution-environment and security layer. Stripe runs devboxes in QA with no production access and no arbitrary egress. Cloudflare has OAuth crypto design, CVEs and AGENTS.md security invariants. Cursor has a blocking security CI gate, dependency-reachability patching and daily invariant checks. Little covers application architecture quality, except Cursor's admission that a flawed spec produced an architecture "unfit to evolve".

### Cited Findings
- **Network and sandbox (Stripe).** QA-environment devboxes with no "real user data, Stripe's production services, or arbitrary network egress". The blast radius is limited to one devbox, so agents skip permission prompts. A security-control framework blocks destructive MCP tool actions (via WebFetch) ([Stripe Part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2))
- **Application security design (Cloudflare):**
  - Tokens are stored only as SHA-256 hashes, and props are encrypted with a per-grant key wrapped per token.
  - Review caught the wrapping key being derivable from the stored token ID.
  - HMAC-SHA-256 with a static key was chosen over PBKDF2 for performance on high-entropy input (verbatim, gh api) ([commit adcbb5de](https://github.com/cloudflare/workers-oauth-provider/commit/adcbb5de9c24f5b6a7dbea2e0a313a87c304d9bb); [AGENTS.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/AGENTS.md))
- **Protocol-layer misses (Cloudflare).** The redirect_uri was not checked at the authorize endpoint (CVE-2025-4143), and PKCE could be downgraded (CVE-2025-4144) (verbatim, gh api) ([GHSA-4pc9-x2fx-p7vj](https://github.com/cloudflare/workers-oauth-provider/security/advisories/GHSA-4pc9-x2fx-p7vj); [GHSA-qgp8-v765-qxx9](https://github.com/cloudflare/workers-oauth-provider/security/advisories/GHSA-qgp8-v765-qxx9))
- **HTTP-layer critique (Cloudflare).** Permissive CORS, missing nosniff and HSTS headers, and incorrect Basic-auth decoding (via WebFetch) ([Neil Madden 2025-06-06](https://neilmadden.blog/?p=3982))
- **Architecture hygiene (Cloudflare, 2026).**
  - "Audit-oriented architecture": consent and confused-deputy protections in `src/oauth-consent.ts`; experimental Enterprise-Managed Authorization isolated in `src/ema/` "so its JWT and trust-boundary code can be reviewed independently".
  - A PImpl pattern so private methods are not exposed over Workers RPC.
  - Zero runtime dependencies, with new dependencies "Ask first" (verbatim, gh api) ([AGENTS.md](https://github.com/cloudflare/workers-oauth-provider/blob/main/AGENTS.md))
- **Security pipeline (Cursor).** A blocking security CI gate, dependency patching with reachability analysis and a canary deploy as the final gate, and a daily invariant-drift agent (via WebFetch) ([Cursor, Security agents](https://cursor.com/blog/security-agents))
- **App architecture (Cursor swarm).** A flawed initial spec produced an architecture "unfit to evolve into a full browser". Dependencies drifted until a policy was stated (via WebFetch) ([Cursor, Self-driving codebases](https://cursor.com/blog/self-driving-codebases))
- **Cache and data-isolation bug (Cloudflare vinext)** [snippet]. Fetch-cache keys ignored request headers, so different users could get the same cached response ([Hacktron](https://www.hacktron.ai/blog/hacking-cloudflare-vinext)). TODO: verify.

### Inferences
- Across all three, isolation and blast-radius limits (sandbox, QA environment, canary) carry more of the safety load than review alone. The security bugs that escaped (redirect_uri, PKCE downgrade, cache-key isolation) are cross-request or cross-party properties. Per-diff review tends to miss these, while conformance or adversarial tests can catch them.

### Gaps
- TODO: network-layer detail for Cursor's cloud agents (sandbox egress) and Cloudflare's agent sandboxes. Not yet researched.
