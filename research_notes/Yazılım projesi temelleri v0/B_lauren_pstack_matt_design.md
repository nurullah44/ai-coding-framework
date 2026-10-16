# B — Architecture and system design in the delivery loop

Retrieved: **2026-10-16**. Builds on the existing Lauren written/code and talk notes, Matt/pstack notes, and the report's Lauren/template sections. This note adds placement, product boundaries, enforcement and growth findings. No video claims are used.

**Snapshots:** pstack uses latest `cursor/plugins` main **f713f14203cd6420cec13cd7c55c32d598dbab68**, 2026-10-16; Matt uses latest main **49dd158d1076134a641b33efb035946536778336**, 2026-10-09. Links pin the whole-repo snapshots, not necessarily each file's last change. [pstack commit][P-head], [Matt commit][M-head]. Alexey's article is dated 2026-07-22. [Article][A].

Coverage: all pstack guide chapters; architect and all its references; router, relevant playbooks/principles, review, verification, TypeScript and webhook skills. Matt's requested skills plus flow, domain modeling, diagnosis, retro and wizard. Other repository Markdown was scanned for security/network/performance terms. Repository material was read as source, not executed. “Not in sources” refers to this material, not everything these authors have ever said. [pstack tree][P-tree], [Matt tree][M-tree].

## 1. Where architecture happens

| Source | Placement and first-hand quotation | What the artifact contains |
|---|---|---|
| Matt | Decisions develop **before the spec**, through grilling and optional runnable prototypes. The spec then captures them under Implementation Decisions, including “Architectural decisions”, “Schema changes” and “API contracts”. Multi-session work proceeds to tickets; small work can go directly to implementation. [Flow][M-flow], [spec][M-spec]. | Agreed test seams, affected modules/interfaces, schemas, contracts, tests and exclusions. Existing glossary/ADRs constrain the work. This is synthesis of discussed decisions, not a new architecture interview. [Spec][M-spec]. |
| Lauren/pstack | **Understand → separate design/sketch → implementation**; optional written plan **after design**. Guide: “You plan through code” and “the written plan comes last.” Architect: “The caller's usage is written first and the type sketch derived from it.” [Guide 04][P-design], [architect][P-arch]. | Caller examples → types/signatures → module map, with rationale, invariants, validation placement, alternatives, accepted tradeoffs and risks. Revisit the design when repeated implementation friction shows it is wrong. [Architect][P-arch], [rationale][P-rationale]. |
| Alexey's article | **Product spec → separate stack/architecture choice → backlog → grooming per issue.** “During the brainstorming session, we didn’t choose the tech stack.” The coding agent proposes options without coding; the human selects. [Article, “Choose the stack and architecture”][A]. | Detailed system-design procedure: **Not in sources** in the article. [Article][A]. |

Matt's architecture survey is also a **maintenance entry point**: find observed friction, select a deepening opportunity, then grill through its design. It is not mandatory before every feature. [Survey][M-survey]. pstack's Feature playbook invokes architect before delegating implementation; the router triggers it for code crossing a function boundary. [Feature][P-feature], [router][P-mode].

## 2. Concrete architecture and data/state rules

| Concern | Matt | Lauren/pstack |
|---|---|---|
| Deep modules | Small interface, substantial behavior; callers and tests cross the same seam. “Interface” includes ordering, invariants, errors, configuration and performance. A module can be a function, package or tier-spanning slice. [Design vocabulary][M-design]. | Compare structurally distinct designs by interface depth. A deep module concentrates capability; a long call chain scatters it. Reject pass-through layers and interfaces that expose internal coordination. [Architect][P-arch], [red flags][P-red]. |
| Usage and types first | Alternative designs include types/methods, usage, hidden implementation, dependencies and tradeoffs. This is not an explicit caller-examples-first rule. [Design it twice][M-twice]. | Settle caller usage, types and module shape before bodies. Derive the sketch from usage; describe data structures and trace dominant access patterns before logic. [Runner][P-runner], [rationale][P-rationale]. |
| One supported way | Entry-point-only imports in the beta TypeScript setup; several public entry points are permitted. [TS setup][M-boundaries]. | One supported way per task. Remove alternate paths agents would copy; make outside imports of internals fail the build. [Red flags][P-red], [correct][P-correct]. |
| Ownership and source of truth | Domain glossary names concepts; ADR examples record ownership, integration and hard-to-reverse technology choices. An ADR is offered only for consequential, surprising tradeoffs. [Domain modeling][M-domain], [ADR format][M-adr]. | One owner per state; derive duplicated lists or fail the build when they disagree. Organize around owned knowledge, not load/validate/save phases. [Red flags][P-red]. |
| State/data modeling | Prototype reducers, explicit state machines or pure transforms; display state and exercise awkward/illegal transitions with the user. Do not depend on production persistence by default. [Logic prototype][M-logic], [prototype][M-prototype]. | State machines instead of booleans; typed models, tables/registries, reducers and collections suited to access patterns. Keep boring local code when an abstraction adds no value. [Model the domain][P-domain]. |
| Boundaries and types | Accept dependencies rather than construct them internally. For owned remote systems, inject a port with production/test adapters; real variation must justify a seam. [Design][M-design], [deepening][M-deep]. | Parse external input into domain types at CLI/config/API/network/database boundaries. Brand semantic IDs; derive types from authoritative schemas; use discriminated unions and exhaustive checks. Keep transport/storage/framework representations private. [Boundary][P-boundary], [types][P-types], [TypeScript][P-ts]. |
| Concurrent state | The cited module guidance supplies adapters/test seams; a general concurrent-state ownership procedure is **Not in sources** there. [Deepening][M-deep]. | Remove unnecessary shared writes first. Otherwise enforce serialization structurally through ownership, locks or compare-and-swap; conventions are insufficient. [Shared state][P-state]. |
| Foundation and reader load | Prefer actual variation and observed friction over speculative seams/refactors. [Design][M-design], [survey][M-survey]. | Put shared types, CI/lint and test infrastructure before features when later phases benefit. Remove dead code first. Reduce both layers to trace and mutable state to remember. [Foundational thinking][P-foundation], [reader load][P-reader]. |
| Retries and partial failure | A general recovery/idempotency architecture is **Not in sources** in the requested design/TDD skills. [Design][M-design], [TDD][M-tdd]. | Reconcile from current state; repeat safely after crashes; make partial artifacts recognizable and recoverable. Ask what happens after a second execution or a crash at each point. [Idempotency][P-idempotent]. |

These are design prescriptions, not evidence that every application using the skills satisfies them. The files supply instructions and selected setup/checking tools. [Matt TS setup][M-boundaries], [pstack correct][P-correct].

## 3. How instructions become enforcement and proof

pstack's repair order is **architecture → types → lint/CI → behavior tests → docs for judgment**. Give each state one owner, remove wrong imports/old paths, then enforce remaining constraints. The lint error should name the supported replacement. Run the same check locally and in CI, prove it rejects a real past mistake, and keep a rule/enforcer table. [Correct][P-correct].

Matt also prefers deterministic checks for mechanical violations and reserves prose standards for judgment. `retro` checks whether existing gates are missing, unwired or broken, then presents improvements. His **beta, plugin-excluded** `setup-ts-deep-modules` concretely installs dependency-cruiser, checks entry-point imports/test imports/cycles, wires an umbrella check, and requires **pass → illegal import fails → pass**. Package-layer dependency rules are explicitly left to the consuming repo. [Retro][M-retro], [TS setup][M-boundaries], [beta status][M-beta].

| Mechanism | What it proves; limit |
|---|---|
| pstack plan checker | Checks Markdown plan structure, required verification blocks, lanes, screenshot paths and pass predicates. It does **not** run those scenarios or prove architectural correctness. [Plan][P-plan], [checker code][P-checker]. |
| Behavior tests | Matt tests at agreed public seams with independent expected values. pstack requires tests to fail when behavior breaks and survive internal refactors. [Matt TDD][M-tdd], [pstack testing][P-testing]. |
| Live verification | pstack generates Launch/Doctor/Drive/Evidence/Cleanup plus a feature map. It must execute its generated recipe once; proof captures the real action, resulting state and side effects. [Generator][P-verify]. |
| Independent acceptance | Alexey: “Only the acceptance criteria and the running code count.” His graph is **PM → engineer → QA**, looping failures back to engineer; a separate reviewer is **Not in sources**. [Article][A]. |
| Merge gate | pstack Shipping requires an independent per-PR user-path verdict, checks it still covers the patch, and lands only a contiguous verified stack. Green CI alone is insufficient. [Shipping][P-ship]. |

## 4. Product security, auth, secrets and network

| Topic | Matt | Lauren/pstack | Alexey |
|---|---|---|---|
| Input/trust boundaries | Ports/adapters isolate owned network services and third parties for testing. This is not an authorization model. [Deepening][M-deep]. | Validate raw external data once; keep domain logic pure. “Trust internal code” concerns typed internal data; the principle supplies no identity/permission design. [Boundary][P-boundary], [types][P-types]. | The linked product spec limits pre-reveal visibility and defines member/facilitator actions. [Original plan][A-plan]. |
| Auth and product security review | ADRs may record auth-provider choices; UI prototypes retain existing auth. A required dedicated product security review is **Not in sources** in the reviewed flow/review skills. [ADR][M-adr], [UI prototype][M-ui], [review][M-review]. | Review explicitly checks endpoint authentication/authorization gaps, SQL/shell/eval/HTML injection paths, secret leaks and security-sensitive races. Findings must trace the actual input path. High-risk security/privacy/data findings are escalated, not adopted as general dismissal rules. [Rubric][P-rubric], [bot triage][P-triage]. | Product spec requires contributor anonymity and facilitator approval before AI-extracted outcomes are saved. Login/session/authorization implementation: **Not in sources** there. [Original plan][A-plan]. |
| Secrets | Redact debugging artifacts and keep credentials in environment variables. Wizard uses hidden input, writes `.env`/CI secrets, and copies only CI-needed values to CI. This is handling guidance, not a rotation/encryption policy. [Diagnosis][M-diagnose], [wizard][M-wizard]. | Bot UI keeps the webhook sender key on the server, outside browser/chat/logs; webhook content is untrusted data. [Bot UI][P-bot]. | Secrets procedure: **Not in sources** in the article. A later project QA file explicitly fails checked-in secrets. [Article][A], [later QA][A-qa]. |
| Network | Owned remote dependencies can use HTTP/gRPC/queue production adapters and in-memory test adapters. Network topology, TLS policy and firewall rules: **Not in sources** in this guidance. [Deepening][M-deep]. | Specific bot recipe: browser → local server → webhook; eight-second timeout, one attempt. Tailscale hosting uses `0.0.0.0` and HTTP. This is a bot dashboard recipe, not a general internet deployment design. [Bot UI][P-bot]. | Network design: **Not in sources** in the article or linked product spec. [Article][A], [plan][A-plan]. |

A complete threat model, tenant-isolation design, authentication lifecycle, security acceptance suite or general network blueprint for small products is **Not in sources** in this set. The concrete controls above should not be presented as that blueprint. [Matt flow/review][M-flow], [pstack rubric][P-rubric], [Alexey spec][A-plan].

**Later repository supplement, distinct from the article:** Retroloop main at **3443c0e04d002bfea2e37ef56c2f5bd6b3ad66cf** (2026-09-19) tells PMs to settle remaining technical choices in issue Constraints with reasons. Its QA adds browser clicking, migration checks, lint and secrets checks. This is project-specific per-ticket refinement, beyond the article's tutorial. [Commit][A-head], [PM][A-pm], [QA][A-qa].

## 5. Performance, scale and growth

| Source | Concrete guidance | Not supplied |
|---|---|---|
| Matt | Performance belongs to an interface. Diagnose regressions with a baseline timing harness, profiler or query plan, then bisect. Architecture improvement prioritizes observed change hotspots and YAGNI. [Design][M-design], [diagnosis][M-diagnose], [survey][M-survey]. | Capacity targets, load-test method and growth topology: **Not in sources** in these skills. |
| Lauren/pstack | Choose structures for dominant access patterns before logic. Measure a baseline, trace the cause, avoid work/repetition before attempting concurrency. Hillclimb names realistic data/history/state/concurrency dimensions, proves harness sensitivity, freezes it, and keeps/reverts one change at a time. [Runner][P-runner], [perf][P-perf], [hillclimb][P-hill]. | A prescribed deployment architecture, user-count sizing rule or guaranteed growth without rewrite: **Not in sources**. |
| Lauren/pstack measurement | Production-like configuration; count errors and completed work; alternate repeated runs; report median/range; check physical limits and end-to-end impact. Correctness remains a gate. [Benchmark checklist][P-benchmark]. | Benchmark results for a 0–5,000-user product: **Not in sources**. |
| Alexey | MVP scope; more complex agent graphs cost extra time/tokens. [Article][A]. | Product performance/scaling design: **Not in sources**. Agent orchestration cost is not product capacity. |

**For the user's 0–5,000-user range:** none of these sources establishes a capacity guarantee or a no-rewrite promise. pstack supplies access-pattern/state and measurement disciplines; Matt supplies interface/locality and incremental design disciplines. A default monolith/microservices choice, day-one distributed setup or numerical sizing recommendation is **Not in sources**. [pstack domain][P-domain], [hillclimb][P-hill], [Matt design][M-design], [survey][M-survey].

## 6. Mapping to the delivery loop

Research mapping of the cited instructions. Gaps are scoped to those documents.

| Stage | What Matt / Alexey give | What Lauren/pstack add | Gap |
|---|---|---|---|
| Need/discovery | Matt: grill decisions; retain glossary/ADRs; prototype uncertainty. Alexey: brainstorm product scope. [Flow][M-flow], [article][A]. | Goal, falsifiable done check, desired evidence and real constraints; understand existing mechanics/history. `sharpen` checks the level of specification and keeps user choices open. [Guide 02][P-prompt], [03][P-understand], [sharpen][P-sharpen]. | A mandatory product-risk/abuse discovery checklist: **Not in sources**. |
| Design | Matt: deep modules, seams/adapters, alternatives, domain decisions. Alexey: stack choice after spec. [Design][M-design], [alternatives][M-twice], [article][A]. | Usage → types/data → boundaries; screen red flags; test uncertain choices with runnable prototypes. [Architect][P-arch], [runner][P-runner], [guide 04][P-design]. | Complete system/deployment/security design template: **Not in sources**. |
| Spec | Matt: user stories, implementation/contracts/schema choices, agreed testing and exclusions. Alexey's linked spec: product permissions/state. [Spec][M-spec], [original plan][A-plan]. | Usage/rationale/type sketch as contract; written prose plan is optional and later. [Rationale][P-rationale], [guide 04][P-design]. | Shared mandatory artifact or automatic proof of user intent: **Not in sources**. |
| Tickets | Matt: verifiable vertical slices, blockers and approved granularity. Alexey: PM grooms checkable issues. [Tickets][M-tickets], [PM][A-pm], [template][A-template]. | Per-PR files/build/observable result/unit/live/perf proof; shared-state and decomposition checkpoint. [Plan][P-plan], [Feature][P-feature]. | Different scope contracts: Matt avoids durable file paths; pstack names files. [Tickets][M-tickets], [plan][P-plan]. |
| Build | Matt: one red/green slice at agreed seams, regular typecheck/tests. Alexey: engineer implements fixed criteria. [Implement][M-implement], [TDD][M-tdd], [engineer][A-engineer]. | Named data shape before logic; bounded delegate scope; boundary parsing and structural constraints. [Feature][P-feature], [boundary][P-boundary]. | The consuming repo still has to implement its actual commands, checks and environment. |
| Review | Matt: separate Standards and Spec axes. Alexey's article defines QA, without a separate reviewer. [Review][M-review], [article][A]. | Independent model-family review, explicit security lens, executable blast-radius proof. [Interrogate][P-review], [rubric][P-rubric], [blast radius][P-blast]. | A guarantee that review finds every defect: **Not in sources**. |
| QA | Alexey: criterion-by-criterion runtime verdict. Matt's implement skill has no equivalent mandatory live QA gate. [QA][A-qa], [implement][M-implement]. | Reusable launch/doctor/drive recipe and feature map; action/state/side-effect evidence. [Generator][P-verify]. | App-specific fixtures, credentials and reachable scenarios must exist; generated verification must itself run. [Generator][P-verify]. |
| Ship | Matt: reviewed integration branch / PR ready for review. Alexey: close after QA. [Implement spec][M-implement-spec], [article][A]. | Independent current-patch proof before merge; stop at verification gaps. Deploys remain on the router's human-pause list. [Shipping][P-ship], [router][P-mode]. | Merge is not a production release recipe; common deploy/rollback/operations contract: **Not in sources**. |
| Maintain | Matt: retro, observed architecture friction and ADRs. Alexey: update working documents. [Retro][M-retro], [survey][M-survey], [article][A]. | Prevent repeat mistakes structurally; audit source plus every mapped feature live; report product regressions rather than rewriting the map to hide them. [Correct][P-correct], [maintain verify][P-maintain]. | A complete production security/capacity/incident program: **Not in sources**. |

## 7. Differences that must remain visible

| Topic | Difference / qualification |
|---|---|
| Human design gate | Matt waits for shared understanding, agreed test seams and ticket approval. pstack architect continues by default; checkpoint is opt-in. [Grilling][M-grill], [TDD][M-tdd], [tickets][M-tickets], [architect][P-arch]. |
| Migration strategy | Matt's wide-refactor exception uses expand → migrate → contract, allowing an integration branch if batches cannot stay green. pstack favors same-wave deletion for internal APIs **when no external users need compatibility**; planned reversible breakage may occur at declared verification boundaries. These are different instructions with stated scope. [Tickets][M-tickets], [migration][P-migrate], [outcome][P-outcome]. |
| Review before implementation | pstack guide discourages adversarial review of abstract plans and points it at a diff after prototypes. Architect Phase C nevertheless permits interrogating the synthesized sketch. Both statements exist; a universal ban would overstate the sources. [Guide 04][P-design], [architect][P-arch]. |
| Shared recommendations versus tested outcomes | Instructions for deep modules, static checks, review and live proof are present. A controlled demonstration that this combined loop guarantees trustworthy software or rewrite-free growth is **Not in sources**. [Matt flow][M-flow], [pstack guide][P-design], [article][A]. |

[P-ship]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/shipping.md
[P-foundation]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-foundational-thinking/SKILL.md
[P-reader]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-minimize-reader-load/SKILL.md
[P-head]: https://github.com/cursor/plugins/commit/f713f14203cd6420cec13cd7c55c32d598dbab68
[P-tree]: https://github.com/cursor/plugins/tree/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack
[P-arch]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/SKILL.md
[P-red]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/references/design-red-flags.md
[P-rationale]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/references/rationale-template.md
[P-runner]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/references/runner-prompt.md
[P-design]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/docs/guide/04-design.md
[P-prompt]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/docs/guide/02-poteto-mode.md
[P-understand]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/docs/guide/03-understand.md
[P-sharpen]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/sharpen/SKILL.md
[P-mode]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/SKILL.md
[P-feature]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/feature.md
[P-plan]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/multi-phase-plan.md
[P-checker]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/scripts/check-plan.mjs
[P-domain]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-model-the-domain/SKILL.md
[P-boundary]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-boundary-discipline/SKILL.md
[P-types]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-type-system-discipline/SKILL.md
[P-ts]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/typescript-best-practices/SKILL.md
[P-state]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-separate-before-serializing-shared-state/SKILL.md
[P-idempotent]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-make-operations-idempotent/SKILL.md
[P-testing]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-test-behavior-not-implementation/SKILL.md
[P-correct]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/correct/SKILL.md
[P-verify]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/create-verification-skill/SKILL.md
[P-maintain]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/maintain-verification-skill/SKILL.md
[P-review]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/interrogate/SKILL.md
[P-rubric]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/interrogate/references/rubric.md
[P-triage]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/references/bugbot-triage.md
[P-blast]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/blast-radius/SKILL.md
[P-bot]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/make-bot-ui/SKILL.md
[P-perf]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/perf-issue.md
[P-hill]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/hillclimb.md
[P-benchmark]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/benchmark-checklist/SKILL.md
[P-migrate]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md
[P-outcome]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-outcome-oriented-execution/SKILL.md
[M-head]: https://github.com/mattpocock/skills/commit/49dd158d1076134a641b33efb035946536778336
[M-tree]: https://github.com/mattpocock/skills/tree/49dd158d1076134a641b33efb035946536778336
[M-flow]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/ask-matt/SKILL.md
[M-grill]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/productivity/grilling/SKILL.md
[M-spec]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/to-spec/SKILL.md
[M-tickets]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/to-tickets/SKILL.md
[M-prototype]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/prototype/SKILL.md
[M-logic]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/prototype/LOGIC.md
[M-ui]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/prototype/UI.md
[M-design]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/codebase-design/SKILL.md
[M-deep]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/codebase-design/DEEPENING.md
[M-twice]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/codebase-design/DESIGN-IT-TWICE.md
[M-domain]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/domain-modeling/SKILL.md
[M-adr]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/domain-modeling/ADR-FORMAT.md
[M-survey]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/improve-codebase-architecture/SKILL.md
[M-boundaries]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/in-progress/setup-ts-deep-modules/SKILL.md
[M-beta]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/in-progress/README.md
[M-retro]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/retro/SKILL.md
[M-review]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/code-review/SKILL.md
[M-tdd]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/tdd/SKILL.md
[M-implement]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/implement/SKILL.md
[M-implement-spec]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/implement-spec/SKILL.md
[M-diagnose]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/diagnosing-bugs/SKILL.md
[M-wizard]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/wizard/SKILL.md
[A]: https://aishippingblog.com/p/ai-native-development-specifications
[A-plan]: https://github.com/alexeygrigorev/retroloop/blob/711c33cc5b6ba44c66d994f3bfc9bd5a6915172e/_docs/plan.md
[A-head]: https://github.com/alexeygrigorev/retroloop/commit/3443c0e04d002bfea2e37ef56c2f5bd6b3ad66cf
[A-pm]: https://github.com/alexeygrigorev/retroloop/blob/3443c0e04d002bfea2e37ef56c2f5bd6b3ad66cf/_docs/team/pm.md
[A-engineer]: https://github.com/alexeygrigorev/retroloop/blob/3443c0e04d002bfea2e37ef56c2f5bd6b3ad66cf/_docs/team/software-engineer.md
[A-qa]: https://github.com/alexeygrigorev/retroloop/blob/3443c0e04d002bfea2e37ef56c2f5bd6b3ad66cf/_docs/team/qa-engineer.md
[A-template]: https://github.com/alexeygrigorev/retroloop/blob/3443c0e04d002bfea2e37ef56c2f5bd6b3ad66cf/_docs/task-template.md
