# Software project foundations v0: options

## Big picture

The goal is to choose a foundation that reduces the need for rewrites while growing within the 0–5,000-user range and checks agents' work against evidence. The sources provide no capacity or no-rewrite guarantee for this range.

The backbone chosen by the project owner is **spec → ticket → build → review → ship**: write what needs to be done, divide it into verifiable tasks, build, review, and release. Matt Pocock and Alexey Grigorev support this flow; their detailed sequences differ. Matt develops architectural decisions before the spec; Alexey has technology and architecture chosen after the product spec. [Matt][M-flow], [Alexey][A]

Lauren/pstack clarifies usage examples, data, and component boundaries before implementation. The order for enforcing rules is: **architecture → types** (the allowed shapes of data) **→ lint/CI** (automated code rules and checks that run on every change) **→ behavior tests → documentation for matters that require judgment**. The mechanisms must be set up and run in the project. Work is not considered finished until the action, result, and side effect are demonstrated in the running application. [Design][P-arch], [enforcement][P-correct], [evidence][P-verify]

```text
Need → product scope ↔ architecture / system design → implementation-ready spec
                       data, security, network, capacity, operations decisions
spec → ticket → build → review + evidence in the running application → ship
         acceptance/test criteria                                     ↓
         ←←←←←←←←← monitoring, maintenance, and learning ←←←←←←←←←←←←←←
```

## Project stages

The backbone is fixed; the options below determine how much detail we record about the design and decisions. They can be used together. System design means deciding on the application, database, external services, and the data flow between them. Security, capacity, and release decisions are addressed here; they are not left solely to the final review. [Ubl][design-doc], [OWASP][threat]

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Short design note | Before coding, you bring together goals, exclusions, the solution, and its alternatives in a short document. | [Malte Ubl / Google][design-doc] | When the solution is uncertain or an important choice needs discussion; it may be unnecessary for an obvious small change. |
| Decision record + simple diagram | You record the reasons for important decisions in a short record called an ADR and show the system's components using a diagramming approach called C4. | [Michael Nygard][adr], [C4][c4] | To find the reasoning behind decisions later and give agents clear boundaries; not every decision is made at the start. |
| Lightweight planning from Shape Up | You bound the problem and the time to spend in advance, then determine the detailed tasks as you work. | [Ryan Singer / Basecamp][shape-size], [defining the work][shape-handover] | To reduce meetings in a small team; the original method does not divide all work into tickets up front. |

**What the sources recommend for v0:** Ubl recommends adjusting the amount of documentation to the uncertainty; Basecamp recommends reducing meetings and process steps in a small team. Nygard's practice of recording important decisions with their reasoning can be added to this lightweight approach. [Ubl][design-doc], [Basecamp][shape-size], [Nygard][adr]

**How the agent enforces this:** pstack's plan check verifies that evidence steps have been written down; separate evidence from the live application demonstrates that these steps actually work. [Plan][P-plan], [evidence][P-verify]

## Architectural style

This choice determines how many separate applications we deploy and their internal boundaries. “Single application” and “modular application” are not mutually exclusive; a module is a component that brings together the knowledge and behavior for a particular task.

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Single application: monolith | You keep the product's functionality in a single application that is developed and deployed together. | [DHH][dhh-monolith], [Martin Fowler][monolith-first] | For a new, simple product; to start without the overhead of separate deployment processes. |
| Single application with clear boundaries: modular monolith | Within a single application, you establish boundaries by business domain and explicit access points. | [Shopify][shopify] | When components becoming entangled makes changes difficult; without spending weeks designing boundaries for an unfamiliar product. |
| Independent services: microservices | You divide the product into applications that are developed and deployed separately. | [Stefan Tilkov][tilkov] | When the business domain is well understood and independent teams working in parallel are a real need; this adds network and operational overhead. |

**What the sources recommend for v0:** DHH recommends a monolith for most web products; Shopify recommends limited architectural investment at the start. Boundaries can grow stronger as needs become clearer. Tilkov's different recommendation depends on the conditions of independent development and a known business domain. [DHH][dhh-monolith], [Shopify][shopify], [Tilkov][tilkov]

**How the agent enforces this:** pstack first establishes the boundary through a usage example and types, then calls for rules that stop the build to prevent improper access to another module's internals. [Architecture][P-arch], [enforcement][P-correct]

## Technology selection

This decision is less about the number of languages and tools than about which tools the team can understand and operate. A new tool's cost is not just setup; debugging, monitoring, and maintenance add to it. [McKinley][boring]

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| A small set of mature, familiar tools | You build the product with existing tools whose behavior and failures you understand. | [Dan McKinley][boring] | When technological novelty is not the product's core value and existing tools meet the need. |
| A justified new or specialized tool | You show why existing tools fall short and add a new tool for a specific need. | [Dan McKinley][boring] | When the benefit offsets the learning and operational overhead; popularity alone is not sufficient justification. |

**What the sources recommend for v0:** McKinley prioritizes mature tools and few additions; being old does not by itself mean being good. A specific language, framework, product list, or tool count: **Not in sources**. [McKinley][boring]

**How the agent enforces this:** pstack's principle of a single supported path and identical local/CI checks preserve the chosen way of using the tools; they do not make the technology choice themselves. [Enforcement][P-correct]

## Data and scaling readiness

The first two options can be used together; the third is added for a specific problem. User count alone is not a measure of capacity: concurrent work, data volume, and the operations performed matter. The sources provide no measured capacity for these products. [AWS][db-scale], [pstack measurement][P-benchmark]

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Relational database, improving the existing setup first | You keep data in related tables and improve queries and database server capacity as load grows. | [AWS][db-scale], [shared database example: Shopify][shopify] | When the relational model fits the product and a single database has sufficient resources; increasing capacity may require downtime. |
| Keeping persistent state outside the application | You do not tie persistent data to the application's temporary memory or local files. | [Adam Wiggins / Twelve-Factor][processes] | For restarts and scaling with additional application instances when needed; there is no need to run many servers from the start. |
| Background jobs or caching when needed | You separate long-running work from the user request or reduce repeated work by keeping a copy of frequently read data. | [Heroku][jobs], [Amazon Builders' Library][cache] | When there are long-running jobs or a measured read load; queues and caches solve different problems and have maintenance overhead. |

**What the sources recommend for v0:** AWS supports using a managed database where possible and increasing existing capacity; Twelve-Factor supports keeping persistent state outside the application. A requirement to add a second database, queue, or cache at a particular user count: **Not in sources**. [AWS][aws-startup], [scaling up][db-scale], [Twelve-Factor][processes]

**How the agent enforces this:** pstack designs data ownership and access patterns first and requires reruns to be safe; it calls for measuring speed claims with realistic data and concurrent load. [Ownership][P-red], [data][P-domain], [safe repetition][P-idempotent], [measurement][P-benchmark]

## Security foundation

OWASP Top 10 introduces risks; ASVS is a list of verifiable security requirements. We choose the level based on the sensitivity of the data held and the potential harm, rather than user count. [OWASP][top10], [ASVS][asvs]

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Starting with ASVS Level 1 | You implement and verify the basic security controls that apply to the application. | [OWASP ASVS][asvs] | In the example of an early-stage startup collecting limited sensitive data; being a small product alone is not sufficient justification. |
| Targeting ASVS Level 2 | You add broader security requirements on top of the basic controls. | [OWASP ASVS][asvs] | As OWASP's recommended target for most applications; when the data and risk require stronger assurance. |

With either choice, the questions asked during design are: “What can go wrong? What are we going to do about it? Did we do a good job?” Authentication establishes who a person is; authorization determines which data and operations they can access. Using a maintained login library and connecting to an external identity provider are separate choices; OWASP recommends the login standard called OIDC for the second approach. [Threat assessment][threat], [libraries][security-library], [login][auth]

Passwords and access keys must not end up in code or logs; access to them, their rotation, and their revocation must be addressed. Twelve-Factor recommends environment variables, while OWASP notes the risk of their leaking in packaged applications; there is therefore no shared recommendation for a single storage method. Vulnerabilities in the software dependencies used must also be monitored. [Configuration][config], [secrets][secrets], [dependencies][supply-chain]

**What the sources recommend for v0:** OWASP recommends targeting Level 2 for most applications; Level 1 may be considered as a starting point with limited sensitive data. A definitive level selection without information about the product's data and risks: **Not in sources**. [ASVS][asvs]

**How the agent enforces this:** pstack validates external data at entry; independent review looks for authentication/authorization gaps, dangerous inputs, and key leaks, but does not automatically verify all of ASVS. [Boundaries][P-boundary], [review][P-rubric]

## Network and deployment

This choice determines where we run the application and who takes responsibility for server maintenance. Using a managed service does not mean all security and recovery settings are ready.

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Managed platform and database | You handle tasks such as deployment, external access, and certificates through services offered by the provider. | [Render example][render-web], [Kamal's guidance][kamal] | When knowledge of Linux and Docker (a tool for packaging an application with its dependencies) is limited; check what the plan includes. |
| Single virtual server + deployment tool | You take responsibility for maintaining a rented server and deploy the application with a tool such as Kamal. | [Kamal][kamal] | When you have server operations knowledge and want portability; responsibility for updates, the database, and backups remains. |
| Kubernetes | You use a common system to manage the placement and operation of packaged applications across multiple servers. | [Kubernetes documentation][kubernetes] | When distributed workloads and automatic recovery are needed; when the cluster's security and maintenance costs can be covered. |

HTTPS, meaning an encrypted web connection, database access limited to the machines that need it, and least privilege are basic concerns. For example, external database access on Render may be enabled by default; the limitation on full certificate verification for internal connections also differs from OWASP's recommendation. The provider's specific settings must be examined. [OWASP networking][db-security], [encrypted connection][tls], [Render settings][render-db]

**What the sources recommend for v0:** Kamal's own recommendation is to choose a fully managed service if Linux/Docker is difficult. A single provider for everyone in this user range or a numerical threshold that requires Kubernetes: **Not in sources**. [Kamal][kamal], [Kubernetes][kubernetes-prod]

**How the agent enforces this:** pstack requires evidence from the running application and calls for human oversight of deployment; a ready-made mechanism that sets up and checks general network rules: **Not in sources**. [Evidence][P-verify], [deployment boundary][P-mode]

## Operations/observability

The second option can be added to the first; backups are addressed separately with either option. The goal is to notice when users are experiencing a problem and to be able to recover lost data within the accepted time.

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Basic logs, error reporting, and metrics | You record events and monitor latency, traffic, errors, and resource saturation. | [Twelve-Factor][logs], [Google SRE][monitoring] | For initial visibility; user errors that never reach the server need separate observation. |
| Service objective and error budget | You define a success target and an accepted allowance for failure for an important user task. | [Google SRE][slo] | To make decisions between reliability and new features; with low traffic, a single error can change the rate substantially. |

There are two approaches to backups: a copy of the data at a particular moment, or returning to a selected point in time using transaction logs. The second allows more granular recovery and requires more setup or a suitable managed service plan. You need to choose acceptable data loss and recovery time, restore the data to a separate location, and check its contents; a “backup completed” notification is not evidence. [PostgreSQL dump][dump], [point-in-time recovery][pitr], [AWS recovery evidence][restore]

Who will make decisions during an outage and how actions will be recorded are also established; Google allows roles to be combined for small incidents. [Incident management][incident]

**What the sources recommend for v0:** Google recommends starting service objectives with inexpensive metrics; AWS recommends testing backups by actually restoring them according to recovery needs. A universal target percentage or backup schedule: **Not in sources**. [Google][slo], [AWS][restore]

**How the agent enforces this:** pstack provides evidence of live user flows; a ready-made mechanism that operates production monitoring, alerts, and backups: **Not in sources**. [Scope of live evidence][P-verify]

## Testing strategy

The two approaches distribute the emphasis on tests differently. A unit test checks a small behavior, an integration test checks components working together, and an end-to-end test checks the user's task from start to finish.

| Option | What it means | Who recommends it | When it fits |
|---|---|---|---|
| Test pyramid | You use many small tests, behavior tests below the user interface, and fewer end-to-end tests. | [Mike Cohn][cohn], [Martin Fowler][pyramid] | When tests that check the entire flow through the screen are slow, fragile, or expensive; when small tests provide quick feedback. |
| Testing Trophy: emphasis on integration | Alongside automated code checks, you emphasize tests that exercise components in a way close to how they are used. | [Kent C. Dodds][trophy] | Particularly for JavaScript/UI work and the backend monoliths within its scope; the recommendation does not cover microservices. |

Both approaches test business rules and database/external service connections; selected end-to-end scenarios are added for core user tasks. Separate security and restore evidence is also required. [Practical testing approach][practical-tests], [ASVS][asvs], [recovery][restore]

**What the sources recommend for v0:** A single winner, fixed test ratio, or coverage percentage for this user range: **Not in sources**. The common ground is testing observable behavior and verifying important user tasks in the running system. [Fowler][pyramid], [Dodds][trophy], [pstack][P-testing]

**How the agent enforces this:** pstack requires the test to fail when behavior breaks, the live recipe to be run, and independent user-flow evidence covering the current change before merging. [Testing][P-testing], [live recipe][P-verify], [merging][P-ship]

## What is not in the sources

The gaps here concern concrete answers for this product, rather than the general method; the basis is [note A](../research_notes/v0_foundations/A_classic_engineering.md) and [note B](../research_notes/v0_foundations/B_lauren_pstack_matt_design.md).

- **Product load:** Concurrent users, data volume, the heaviest tasks, and accepted response times: **Not in sources**; the owner must choose the targets.
- **Specific technology and budget:** Language, framework, database product, provider, and monthly budget: **Not in sources**; these must be chosen based on the team's knowledge.
- **Product-specific security:** Roles, separation of customer data, the scope of sensitive data, login method, and the corresponding definitive ASVS level: **Not in sources**.
- **Recovery and service objectives:** Accepted data loss, downtime, backup frequency, retention period, and who is responsible for alerts: **Not in sources**.
- **Human approval and deployment:** Who approves architectural decisions in this project, deployment authority, and the recipe for rolling back a failed deployment: **Not in sources**; pstack's stance on deployment does not provide this recipe.

## Recommended v0 package

**This table is a recommendation derived from the sources; it is presented for the project owner's approval or revision.** It is not a single mandatory package shared by all the sources. The security and deployment choices apply under the stated conditions.

| Area | Recommended option | Why |
|---|---|---|
| Project stages | Fixed backbone + short design note + records of important decisions | Reduces uncertainty before coding and limits the documentation burden on a small team. [Ubl][design-doc], [Nygard][adr] |
| Architectural style | Single application; module boundaries as explicit as needed | Limits component entanglement while keeping deployment overhead low. [DHH][dhh-monolith], [Shopify][shopify] |
| Technology selection | A small set of mature tools the team knows | Reduces the learning and operational costs of new tools. [McKinley][boring] |
| Data and scaling readiness | A single relational database if suitable; persistent state outside the application | Keeps open the option of increasing existing capacity and adding application instances when needed. [AWS][db-scale], [Twelve-Factor][processes] |
| Security foundation | Target Level 2; consider starting at Level 1 with limited sensitive data | Follows OWASP's direction for most applications; ties the starting level to actual risk. [ASVS][asvs] |
| Network and deployment | Managed platform and database if operations knowledge is limited | Reduces server maintenance overhead; network and recovery settings are still checked. [Kamal][kamal], [OWASP][db-security] |
| Operations/observability | Basic metrics + simple service objective + tested restore | Makes it possible to see problems and demonstrate that recovery actually works. [Google][slo], [AWS][restore] |
| Testing strategy | Behavior and integration tests + selected live user tasks | Tests business rules and user outcomes together; the distribution of tests is chosen according to the product. [Vocke][practical-tests], [pstack][P-verify] |

[M-flow]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/ask-matt/SKILL.md
[A]: https://aishippingblog.com/p/ai-native-development-specifications
[P-arch]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/SKILL.md
[P-correct]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/correct/SKILL.md
[P-verify]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/create-verification-skill/SKILL.md
[P-plan]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/multi-phase-plan.md
[P-domain]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-model-the-domain/SKILL.md
[P-red]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/references/design-red-flags.md
[P-idempotent]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-make-operations-idempotent/SKILL.md
[P-benchmark]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/benchmark-checklist/SKILL.md
[P-boundary]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-boundary-discipline/SKILL.md
[P-rubric]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/interrogate/references/rubric.md
[P-mode]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/SKILL.md
[P-testing]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-test-behavior-not-implementation/SKILL.md
[P-ship]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/shipping.md
[design-doc]: https://www.industrialempathy.com/posts/design-docs-at-google/
[adr]: https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions
[c4]: https://c4model.com/diagrams
[shape-size]: https://basecamp.com/shapeup/4.1-appendix-02
[shape-handover]: https://basecamp.com/shapeup/3.1-chapter-10
[dhh-monolith]: https://signalvnoise.com/svn3/the-majestic-monolith-can-become-the-citadel/
[monolith-first]: https://martinfowler.com/bliki/MonolithFirst.html
[shopify]: https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity
[tilkov]: https://martinfowler.com/articles/dont-start-monolith.html
[boring]: https://mcfunley.com/choose-boring-technology
[db-scale]: https://aws.amazon.com/blogs/database/scale-your-relational-database-for-saas-part-1-common-scaling-patterns/
[aws-startup]: https://aws.amazon.com/blogs/startups/how-to-scale-to-10k-users-with-aws/
[processes]: https://12factor.net/processes
[jobs]: https://devcenter.heroku.com/articles/background-jobs-queueing
[cache]: https://aws.amazon.com/builders-library/caching-challenges-and-strategies/
[top10]: https://owasp.org/projects/Top-Ten
[asvs]: https://github.com/OWASP/ASVS/blob/5cf9b032440be53ce345ab3c130fda46ba1ce7a2/5.0/en/0x03-What-is-the-ASVS.md
[threat]: https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
[security-library]: https://top10proactive.owasp.org/archive/2018/c2-leverage-security-frameworks-libraries/
[auth]: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
[config]: https://12factor.net/config
[secrets]: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
[supply-chain]: https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/
[render-web]: https://render.com/docs/web-services
[kamal]: https://kamal-deploy.org/
[kubernetes]: https://kubernetes.io/docs/concepts/overview/
[kubernetes-prod]: https://kubernetes.io/docs/setup/production-environment/
[db-security]: https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html
[tls]: https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
[render-db]: https://render.com/docs/postgresql-creating-connecting
[logs]: https://12factor.net/logs
[monitoring]: https://sre.google/sre-book/monitoring-distributed-systems/
[slo]: https://sre.google/workbook/implementing-slos/
[dump]: https://www.postgresql.org/docs/18/backup-dump.html
[pitr]: https://www.postgresql.org/docs/18/continuous-archiving.html
[restore]: https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_backing_up_data_periodic_recovery_testing_data.html
[incident]: https://sre.google/workbook/incident-response/
[cohn]: https://www.mountaingoatsoftware.com/agile/the-forgotten-layer-of-the-test-automation-pyramid
[pyramid]: https://martinfowler.com/bliki/TestPyramid.html
[trophy]: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
[practical-tests]: https://martinfowler.com/articles/practical-test-pyramid.html
