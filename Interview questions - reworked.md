# Developer Interview Guide — KYC / KSM Platform Team (Reworked)

---

## Table of Contents

- [What this version optimizes for](#what-this-version-optimizes-for)
- [How to use this guide](#how-to-use-this-guide)
- [Suggested interview flow](#suggested-interview-flow)

---

### Section 1 — Behavioral, Collaboration, Ownership, and Engineering Judgment
- [Level 1 — Mid](#level-1--mid)
  - [1. Ownership after a production mistake](#1-ownership-after-a-production-mistake)
  - [2. Handling disagreement in code review](#2-handling-disagreement-in-code-review)
  - [3. Clarifying vague requirements](#3-clarifying-vague-requirements)
  - [4. Teammate breaks the build](#4-teammate-breaks-the-build)
  - [5. Estimate went wrong](#5-estimate-went-wrong)
  - [6. Short-term fix vs clean solution](#6-short-term-fix-vs-clean-solution)
  - [7. Giving difficult feedback](#7-giving-difficult-feedback)
  - [8. Testing under deadline pressure](#8-testing-under-deadline-pressure)
  - [9. Explaining a technical issue to non-engineers](#9-explaining-a-technical-issue-to-non-engineers)
  - [10. Opinionated engineering prompt](#10-opinionated-engineering-prompt)
- [Level 2 — Senior](#level-2--senior)
  - [11. Managing strong disagreement between senior engineers](#11-managing-strong-disagreement-between-senior-engineers)
  - [12. Pushing back on unrealistic delivery pressure](#12-pushing-back-on-unrealistic-delivery-pressure)
  - [13. Running a useful postmortem](#13-running-a-useful-postmortem)
  - [14. Helping an underperforming engineer](#14-helping-an-underperforming-engineer)
  - [15. Changing your mind publicly](#15-changing-your-mind-publicly)
  - [16. Cross-team dependency conflict](#16-cross-team-dependency-conflict)
  - [17. Consistency vs autonomy](#17-consistency-vs-autonomy)
  - [18. Decision making with incomplete data](#18-decision-making-with-incomplete-data)
  - [19. Security/compliance vs speed conflict](#19-securitycompliance-vs-speed-conflict)
  - [20. Opinionated leadership prompt](#20-opinionated-leadership-prompt)

---

### Section 2 — Stack Knowledge, Fundamentals, and Best Practices
- [Level 1 — Mid](#level-1--mid-1)
  - [21. Spring stereotypes in practice](#21-spring-stereotypes-in-practice)
  - [22. Constructor injection](#22-constructor-injection)
  - [23. @Transactional and self-invocation](#23-transactional-and-self-invocation)
  - [24. Flyway + ddl-auto: validate](#24-flyway--ddl-auto-validate)
  - [25. Lazy loading and N+1](#25-lazy-loading-and-n1)
  - [26. Retry-safe POST requests](#26-retry-safe-post-requests)
  - [27. OAuth2 flow choice](#27-oauth2-flow-choice)
  - [28. Testing a service that calls external APIs](#28-testing-a-service-that-calls-external-apis)
  - [29. RestTemplate vs WebClient](#29-resttemplate-vs-webclient)
  - [30. Indexes and trade-offs](#30-indexes-and-trade-offs)
- [Level 2 — Senior](#level-2--senior-1)
  - [31. Separate OAuth2 registrations per downstream system](#31-separate-oauth2-registrations-per-downstream-system)
  - [32. Transaction boundaries around external calls](#32-transaction-boundaries-around-external-calls)
  - [33. Flyway in a multi-branch team](#33-flyway-in-a-multi-branch-team)
  - [34. Resilience in synchronous REST chains](#34-resilience-in-synchronous-rest-chains)
  - [35. Caching with Caffeine or local memory](#35-caching-with-caffeine-or-local-memory)
  - [36. Idempotent background jobs](#36-idempotent-background-jobs)
  - [37. Structured logging](#37-structured-logging)
  - [38. API versioning strategy](#38-api-versioning-strategy)
  - [39. Parallel batch processing](#39-parallel-batch-processing)
  - [40. When not to use JPA](#40-when-not-to-use-jpa)

---

### Section 3 — Debugging, Logic, and Practical Engineering Exercises
- [Level 1 — Mid](#level-1--mid-2)
  - [41. SQL exercise: cases with no documents](#41-sql-exercise-cases-with-no-documents)
  - [42. Java stream exercise](#42-java-stream-exercise)
  - [43. Tracing intermittent 500s](#43-tracing-intermittent-500s)
  - [44. Duplicate request logic exercise](#44-duplicate-request-logic-exercise)
  - [45. Silent scheduled-job failure](#45-silent-scheduled-job-failure)
  - [46. 401 after deploy](#46-401-after-deploy)
  - [47. Reducing cognitive complexity](#47-reducing-cognitive-complexity)
  - [48. Speeding up a sequential job](#48-speeding-up-a-sequential-job)
  - [49. Small Java correctness check](#49-small-java-correctness-check)
  - [50. Mock, spy, or real instance?](#50-mock-spy-or-real-instance)
- [Level 2 — Senior](#level-2--senior-2)
  - [51. Retry policy design exercise](#51-retry-policy-design-exercise)
  - [52. Idempotent pipeline proof](#52-idempotent-pipeline-proof)
  - [53. Parallel update contention](#53-parallel-update-contention)
  - [54. Data-fix strategy exercise](#54-data-fix-strategy-exercise)
  - [55. Diagnosing N+1 from symptoms](#55-diagnosing-n1-from-symptoms)
  - [56. Choosing sync vs async integration](#56-choosing-sync-vs-async-integration)
  - [57. Scheduling / fairness exercise](#57-scheduling--fairness-exercise)
  - [58. Full-stack duplicate-action scenario](#58-full-stack-duplicate-action-scenario)
  - [59. Stale cache logic exercise](#59-stale-cache-logic-exercise)
  - [60. Observability design exercise](#60-observability-design-exercise)

---

### Section 4 — Architecture, Trade-offs, and Full-Stack Decision Making
- [Level 1 — Mid](#level-1--mid-3)
  - [61. Adding a new external integration](#61-adding-a-new-external-integration)
  - [62. Adding a field sourced from another service](#62-adding-a-field-sourced-from-another-service)
  - [63. Async user experience design](#63-async-user-experience-design)
  - [64. Pagination and filtering](#64-pagination-and-filtering)
  - [65. Heavy report generation endpoint](#65-heavy-report-generation-endpoint)
  - [66. Rate limiting design](#66-rate-limiting-design)
  - [67. Versioning decision](#67-versioning-decision)
  - [68. REST vs event/poller](#68-rest-vs-eventpoller)
  - [69. Feature flags and conditional behavior](#69-feature-flags-and-conditional-behavior)
  - [70. Readiness, liveness, startup behavior](#70-readiness-liveness-startup-behavior)
- [Level 2 — Senior](#level-2--senior-3)
  - [71. Synchronous microservices vs event-driven boundaries](#71-synchronous-microservices-vs-event-driven-boundaries)
  - [72. End-to-end traceability design](#72-end-to-end-traceability-design)
  - [73. Availability vs consistency under partition](#73-availability-vs-consistency-under-partition)
  - [74. Downstream slowness design challenge](#74-downstream-slowness-design-challenge)
  - [75. Breaking response change rollout](#75-breaking-response-change-rollout)
  - [76. Service boundary decision](#76-service-boundary-decision)
  - [77. Security architecture across integrations](#77-security-architecture-across-integrations)
  - [78. Zero-downtime schema + code rollout](#78-zero-downtime-schema--code-rollout)
  - [79. First five minutes of a bad release](#79-first-five-minutes-of-a-bad-release)
  - [80. Opinionated architecture prompt](#80-opinionated-architecture-prompt)

---

### Section 5 — Timed Algorithmic, Logic, and Review Exercises (Add-on Pack)
- [5A. LeetCode-style exercises](#5a-leetcode-style-exercises-data-structures--algorithms)
  - [Mid (pick 2 to 3)](#mid-pick-2-to-3)
  - [Senior (pick 2 to 3)](#senior-pick-2-to-3)
- [5B. Complete-code style exercises](#5b-complete-code-style-exercises-service-level-practical-backend)
  - [Mid (pick 2 to 3)](#mid-pick-2-to-3-1)
  - [Senior (pick 2 to 3)](#senior-pick-2-to-3-1)
- [5C. PR code-review exercises](#5c-pr-code-review-exercises-anti-patterns--trade-offs)
  - [Mid (pick 2 to 3)](#mid-pick-2-to-3-2)
  - [Senior (pick 2 to 3)](#senior-pick-2-to-3-2)
- [Suggested usage with existing interview flow](#suggested-usage-with-existing-interview-flow)

---

### Appendix
- [Interviewer Notes: What strong candidates tend to do](#interviewer-notes-what-strong-candidates-tend-to-do)
- [Suggested scoring rubric](#suggested-scoring-rubric)
- [Recommended interview assembly examples](#recommended-interview-assembly-examples)

---

## What this version optimizes for

This version is intentionally designed to test more than memorized answers.

It blends:
- **knowledge-based questions** on the actual stack and common engineering best practices
- **behavioral and collaboration questions** that require concrete examples
- **thinking and logic exercises** tied to realistic development work
- **algorithmic / implementation exercises** that feel like engineering, not LeetCode trivia
- **architectural decision challenges** with explicit trade-off analysis
- **full-stack reasoning** across API, backend, data, async jobs, observability, and operations

It is also grounded in the current ecosystem we actually use or closely resemble:
- Java 17
- Spring Boot 3.x
- REST APIs
- OAuth2 with multiple client registrations
- PostgreSQL + JPA / Hibernate
- Flyway with `ddl-auto: validate`
- background jobs / pipeline processing / retries / polling
- WireMock / Mockito / AssertJ testing style
- rate limiting, logging, tracing, metrics, and Kubernetes-style operations

---

## How to use this guide

- The guide is organized into **4 sections**.
- Each section has **2 levels**: **Mid** and **Senior**.
- Each level contains **10 questions**.
- That gives **80 questions total**.
- For a real interview, do **not** ask all of them. Pick **12–18** depending on the role.
- Ask the candidate to **think out loud**.
- For behavioral questions, require **specific real examples**.
- For architecture questions, push for:
  - assumptions
  - trade-offs
  - failure modes
  - rollout strategy
  - observability
  - security

---

## Suggested interview flow

1. **Warm-up / collaboration** — 10 to 15 min
2. **Core stack / engineering fundamentals** — 15 to 20 min
3. **Practical debugging or coding / logic** — 15 to 20 min
4. **Architecture and decision making** — 15 to 25 min
5. **Optional follow-ups based on profile depth**

---


<sup>[? Back to top](#table-of-contents)</sup>

# Section 1 — Behavioral, Collaboration, Ownership, and Engineering Judgment

This section is meant to detect how the candidate behaves in a real team: ownership, conflict handling, ambiguity tolerance, communication, and maturity.

---

## Level 1 — Mid

### 1. Ownership after a production mistake
**Question:** Tell me about a time you introduced or helped introduce a bug into production. What happened next, and what did you personally do?

**Expected strong response:**
- Gives a real example, not a generic “bugs happen” answer.
- Takes ownership without turning the story into self-protection.
- Describes immediate mitigation, communication, and customer/business impact.
- Explains what changed afterward: tests, alerting, review checklist, rollout practice, or monitoring.
- Shows learning without over-dramatizing the mistake.

### 2. Handling disagreement in code review
**Question:** Describe a code review disagreement where you thought your reviewer was wrong. How did you handle it?

**Expected strong response:**
- Focuses on the decision quality, not ego.
- Explains the technical disagreement clearly.
- Uses evidence: docs, benchmarks, production constraints, conventions, or maintainability.
- Shows willingness to compromise or escalate respectfully.
- Mentions preserving team relationships and consistency.

### 3. Clarifying vague requirements
**Question:** A product owner asks for “a quick endpoint” but the behavior is vague and there are several edge cases. What do you do before writing code?

**Expected strong response:**
- Seeks examples, acceptance criteria, and non-functional expectations.
- Identifies missing cases: errors, pagination, permissions, retries, backward compatibility.
- Proposes a short written contract or sample payloads.
- Avoids false speed by coding before understanding.
- Balances progress with clarification by de-risking the unknowns first.

### 4. Teammate breaks the build
**Question:** A teammate pushes a change that breaks the build for everyone just before a release. How would you react?

**Expected strong response:**
- Prioritizes restoring a healthy main branch quickly.
- Avoids blame-first behavior.
- Suggests rollback, revert, or fast fix depending on risk.
- Communicates clearly in the team channel and updates stakeholders if needed.
- Treats the incident as both operational and cultural: fix now, improve process later.

### 5. Estimate went wrong
**Question:** Tell me about a task you underestimated. How did you recognize it, and what did you do once you knew the estimate was wrong?

**Expected strong response:**
- Mentions early detection rather than hiding the slip.
- Breaks down why the estimate failed: unknown dependencies, bad assumptions, unclear scope, legacy complexity.
- Re-scopes, re-plans, or surfaces risk early.
- Communicates impact transparently.
- Extracts a repeatable lesson: spikes, assumptions list, dependency check, incremental delivery.

### 6. Short-term fix vs clean solution
**Question:** Give me an example where you chose a pragmatic short-term solution instead of the “cleanest” design. Why was that the right call?

**Expected strong response:**
- Shows context sensitivity instead of dogmatism.
- Explains time pressure, business urgency, or dependency constraints.
- Includes a clear containment plan: TODO is not enough; mentions follow-up ticket, guardrails, tests, or deprecation plan.
- Can articulate the technical debt being created.
- Demonstrates judgment rather than preference.

### 7. Giving difficult feedback
**Question:** Tell me about a time you had to give constructive feedback to another developer.

**Expected strong response:**
- Uses a specific example and describes the relationship/context.
- Focuses on behavior/output, not personality.
- Uses direct but respectful communication.
- Includes follow-through or support, not just criticism.
- Shows that feedback is part of raising team quality.

### 8. Testing under deadline pressure
**Question:** You are close to a deadline and do not have time to test everything. How do you decide what to test?

**Expected strong response:**
- Uses risk-based thinking.
- Prioritizes critical flows, regressions, data integrity, security-sensitive paths, and failure handling.
- Differentiates between what must be automated now vs manually checked.
- Mentions observability or feature flags as release risk controls.
- Does not treat testing as optional when time is short.

### 9. Explaining a technical issue to non-engineers
**Question:** Tell me about a time you had to explain a technical problem to a non-technical stakeholder under pressure.

**Expected strong response:**
- Adapts the explanation to impact, options, and timing.
- Avoids jargon unless translated.
- Describes uncertainty honestly.
- Gives realistic recovery expectations.
- Shows calm, trust-building communication.

### 10. Opinionated engineering prompt
**Question:** What is one engineering practice you think teams often apply too mechanically, and when does it become harmful?

**Expected strong response:**
- Gives a nuanced opinion, not a hot take for its own sake.
- Uses examples such as over-abstraction, over-mocking, premature microservices, ceremony-heavy Scrum, or blanket “100% coverage” thinking.
- Explains the intended value of the practice first.
- Identifies when context makes it harmful.
- Shows pragmatism and respect for trade-offs.

---

## Level 2 — Senior

### 11. Managing strong disagreement between senior engineers
**Question:** Two senior developers on your team strongly disagree on the design of a new integration. You are the tech lead for the area. How do you drive a decision?

**Expected strong response:**
- Frames the disagreement around goals, constraints, and decision criteria.
- Makes assumptions explicit: performance, delivery date, security, operability, maintainability.
- Drives evidence gathering instead of personality-based consensus.
- Commits to a decision and documents why.
- Preserves trust even when one option is rejected.

### 12. Pushing back on unrealistic delivery pressure
**Question:** A product owner wants a high-risk change shipped by end of week, but you believe the blast radius is too large. How do you handle that?

**Expected strong response:**
- Pushes back respectfully and concretely.
- Explains risk in business terms: outage likelihood, data corruption, security, rollback difficulty.
- Offers alternatives: phased delivery, feature flag, reduced scope, dark launch, async backfill.
- Avoids binary “yes/no” behavior.
- Demonstrates both courage and partnership.

### 13. Running a useful postmortem
**Question:** Walk me through how you would run a postmortem after a serious incident affecting KYC case processing.

**Expected strong response:**
- Focuses on facts, timeline, impact, root causes, and contributing factors.
- Explicitly avoids blame culture.
- Separates remediation, prevention, and detection improvements.
- Includes owners and deadlines for follow-up actions.
- Shows that process, tooling, and communication failures can all be causes.

### 14. Helping an underperforming engineer
**Question:** Have you ever had to help a developer who was consistently struggling? What did you do?

**Expected strong response:**
- Describes observable gaps clearly: delivery, design, testing, communication, ownership.
- Starts with support and clarity, not labeling.
- Uses coaching, pairing, narrower scope, explicit expectations, and feedback loops.
- Escalates when needed, but only after trying structured support.
- Protects team outcomes while treating the person fairly.

### 15. Changing your mind publicly
**Question:** Tell me about a technical position you argued for strongly and later changed your mind on. What caused the change?

**Expected strong response:**
- Gives a credible example.
- Shows evidence-based humility.
- Explains what new data changed the decision.
- Demonstrates that changing course improved the outcome.
- Signals low ego and high judgment.

### 16. Cross-team dependency conflict
**Question:** Your team cannot deliver because another team owns a dependency and keeps slipping. How do you handle it without turning it into politics?

**Expected strong response:**
- Makes dependencies and dates visible early.
- Distinguishes between escalation and blame.
- Seeks fallback options: stub, temporary adapter, partial release, contract alignment.
- Aligns managers or product leads only when needed, with evidence.
- Keeps shared goals in the conversation.

### 17. Consistency vs autonomy
**Question:** When should teams be forced to follow the same engineering standard, and when should you allow local variation?

**Expected strong response:**
- Identifies areas where consistency matters: security, observability, release process, API conventions, incident response.
- Allows variation where context matters: internal structure, local libraries, implementation details.
- Reasons about cognitive load and operational risk.
- Avoids both anarchy and heavy-handed centralization.
- Shows platform thinking.

### 18. Decision making with incomplete data
**Question:** Tell me about a high-impact decision you had to make with incomplete information. How did you reduce risk?

**Expected strong response:**
- Explains what was unknown and why waiting had a cost.
- Uses experiments, guardrails, phased rollout, or reversible design.
- Communicates uncertainty transparently.
- Defines what signals would confirm or invalidate the choice.
- Shows comfort with ambiguity without becoming reckless.

### 19. Security/compliance vs speed conflict
**Question:** In a regulated environment, how do you respond when delivery pressure conflicts with security or compliance requirements?

**Expected strong response:**
- Treats compliance as a design constraint, not a late-stage blocker.
- Understands that some shortcuts are unacceptable.
- Looks for compliant ways to reduce scope or sequence work differently.
- Communicates risk clearly to product and management.
- Shows judgment around least privilege, auditability, data handling, and approvals.

### 20. Opinionated leadership prompt
**Question:** What is one engineering principle that is often overvalued, and one that is undervalued, in backend teams like ours?

**Expected strong response:**
- Gives a mature and defendable answer.
- Examples might include overvalued: abstraction purity, framework cleverness, broad rewrites. Undervalued: observability, rollback design, naming clarity, ownership.
- Backs opinion with experience.
- Acknowledges counterarguments.
- Demonstrates that opinions are tied to outcomes, not identity.

---


<sup>[? Back to top](#table-of-contents)</sup>

# Section 2 — Stack Knowledge, Fundamentals, and Best Practices

This section checks direct technical knowledge, but in a contextualized way tied to real backend work.

---

## Level 1 — Mid

### 21. Spring stereotypes in practice
**Question:** In a Spring Boot service, what is the practical difference between `@Component`, `@Service`, and `@Repository`? Why might a team annotate HTTP-client classes with `@Service` instead of `@Repository`?

**Expected strong response:**
- Knows all three are stereotype annotations detected by component scanning.
- Explains `@Repository` is semantically for persistence and may enable exception translation.
- Explains `@Service` communicates business/service-role intent.
- Notes that HTTP clients are not database repositories, so `@Service` may be clearer.
- Understands the difference is mostly semantic plus repository-specific behavior.

### 22. Constructor injection
**Question:** Why do most teams prefer constructor injection over field injection in Spring?

**Expected strong response:**
- Mentions immutability and required dependencies.
- Easier unit testing without reflection or Spring context.
- Makes dependencies visible.
- Supports fail-fast wiring and cleaner design.
- May mention field injection can hide large dependency graphs.

### 23. `@Transactional` and self-invocation
**Question:** What does `@Transactional` actually do in Spring, and what happens if one method in a class calls another `@Transactional` method in the same class?

**Expected strong response:**
- Explains proxy-based transaction management.
- Notes transaction boundaries are applied when calls go through the proxy.
- Self-invocation bypasses the proxy, so the annotation may not take effect as expected.
- Mentions class extraction or external proxy call as common fixes.
- Understands transaction semantics rather than just syntax.

### 24. Flyway + `ddl-auto: validate`
**Question:** Why is `ddl-auto: validate` usually safer than `update` in a multi-instance production system using Flyway?

**Expected strong response:**
- Explains schema should be versioned and explicit.
- `validate` checks compatibility without applying surprise changes at app startup.
- `update` can create uncontrolled, environment-dependent schema drift.
- Multi-instance startup races and partial changes are dangerous.
- Flyway provides auditable, ordered migrations and rollback planning.

### 25. Lazy loading and N+1
**Question:** What is the N+1 problem in JPA/Hibernate, and how would you detect and fix it?

**Expected strong response:**
- Explains repeated child fetches caused by iterating over lazily loaded associations.
- Mentions symptoms: slow endpoints, many similar SQL statements, sudden DB load.
- Detects via SQL logs, profiler, APM, Hibernate statistics, or query inspection.
- Fixes with fetch joins, entity graphs, dedicated queries, projections, or batching.
- Knows that switching everything to `EAGER` is usually a bad blanket fix.

### 26. Retry-safe POST requests
**Question:** A client sends a `POST`, times out, and retries. How do you prevent duplicate processing?

**Expected strong response:**
- Uses idempotency keys or request correlation IDs.
- Backs the API contract with persistence: unique constraints or dedup table/state.
- Returns existing result or current status for repeated requests.
- Considers partial completion and race conditions.
- Understands that API-level idempotency must be supported by storage and workflow.

### 27. OAuth2 flow choice
**Question:** What is the difference between client credentials and a delegated flow, and when would you use each in backend-to-backend systems?

**Expected strong response:**
- Client credentials: app identity, no end-user context.
- Delegated flow: acts on behalf of a user and carries user-level authorization implications.
- Chooses based on whether downstream authorization needs user context.
- Understands auditability and least privilege implications.
- Avoids saying one flow is universally better.

### 28. Testing a service that calls external APIs
**Question:** How would you test a service that calls an external REST API without hitting the real API?

**Expected strong response:**
- Distinguishes unit tests from integration tests.
- Uses mocking for unit tests at the client boundary where appropriate.
- Uses stubs such as WireMock for integration-like contract behavior.
- Verifies error handling, timeouts, malformed payloads, retries, and status mapping.
- Keeps tests fast while still covering realistic interaction behavior.

### 29. `RestTemplate` vs `WebClient`
**Question:** In an existing Spring Boot codebase, when would you keep using `RestTemplate`, and when would you reach for `WebClient`?

**Expected strong response:**
- Knows `RestTemplate` is synchronous/blocking and mature but not the strategic direction.
- Knows `WebClient` supports reactive/non-blocking patterns and richer client features.
- Avoids forcing reactive style into a non-reactive app without reason.
- Chooses based on workload, concurrency needs, and ecosystem fit.
- Shows pragmatism in existing codebases.

### 30. Indexes and trade-offs
**Question:** What is a database index, and when can adding one make the system worse instead of better?

**Expected strong response:**
- Explains faster reads through additional data structures on indexed columns.
- Mentions trade-offs: slower writes, more storage, maintenance cost, worse update/insert throughput.
- Understands low-selectivity indexes may not help much.
- Connects indexing decisions to actual query patterns.
- Avoids “add index everywhere” thinking.

---

## Level 2 — Senior

### 31. Separate OAuth2 registrations per downstream system
**Question:** Why would you use separate OAuth2 client registrations per downstream system instead of sharing one token everywhere?

**Expected strong response:**
- Emphasizes least privilege and blast-radius reduction.
- Different downstream APIs may require different scopes, grant types, or providers.
- Improves auditability and revocation control.
- Prevents hidden coupling between integrations.
- Recognizes operational complexity as a trade-off, but worth it in regulated systems.

### 32. Transaction boundaries around external calls
**Question:** A developer wants to keep a database transaction open while making an external HTTP call “so everything stays consistent.” Would you allow it?

**Expected strong response:**
- Usually says no, or “only in rare constrained cases.”
- Explains long transactions hold locks, hurt throughput, and couple DB consistency to network latency.
- Recognizes distributed transactions are hard and often not desirable.
- Suggests alternatives: state transition + outbox/event/job, saga-like flow, compensating action.
- Discusses consistency model explicitly.

### 33. Flyway in a multi-branch team
**Question:** Two developers create migrations with the same version on different branches. What problems can that cause, and what team habits prevent it?

**Expected strong response:**
- Recognizes version collision and broken migration order.
- Explains failure modes in CI, deploy, or shared environments.
- Suggests renumbering before merge, timestamp-based versioning, or team conventions.
- Mentions keeping migrations small and reviewing DB changes carefully.
- Understands migration discipline as process, not just tooling.

### 34. Resilience in synchronous REST chains
**Question:** If four internal services call each other synchronously over REST, what risks appear when one downstream becomes slow or flaky?

**Expected strong response:**
- Mentions cascading latency, thread exhaustion, timeout amplification, retry storms, and partial failures.
- Uses timeouts, circuit breakers, bulkheads, backoff, caching, fallbacks where appropriate.
- Questions whether the flow should remain synchronous at all.
- Thinks about observability and user experience.
- Identifies that resilience is architectural, not just library-based.

### 35. Caching with Caffeine or local memory
**Question:** When is local in-memory caching a good idea in a Spring service, and what are the main traps?

**Expected strong response:**
- Good for expensive reads, static/reference data, or repeated low-risk lookups.
- Calls out staleness, invalidation, multi-instance inconsistency, and warm-up behavior.
- Discusses TTL choice and failure fallback.
- Avoids caching highly dynamic or security-sensitive data blindly.
- Understands cache usefulness depends on access patterns.

### 36. Idempotent background jobs
**Question:** Our jobs often follow a trigger → poll → act pipeline. How do you make each step safe to retry?

**Expected strong response:**
- Gives each step explicit states and transition rules.
- Uses persisted correlation/request IDs and dedup rules.
- Makes side effects conditional on current state, not on “I think this hasn’t happened.”
- Handles partial success and repeated scheduler executions.
- Designs the pipeline so re-running it converges to the same correct result.

### 37. Structured logging
**Question:** What is structured logging, and why is it much more useful than plain text logs in distributed systems?

**Expected strong response:**
- Explains logs as queryable event records with fields, not free-form strings only.
- Includes request/correlation IDs, user or case IDs where appropriate, service name, outcome, latency, and error categories.
- Connects logs to incident response and cross-service tracing.
- Warns about PII/secrets in logs.
- Understands the relationship between logs, metrics, and traces.

### 38. API versioning strategy
**Question:** How would you choose between URL versioning and header-based versioning for an API used by both a frontend SPA and other backend services?

**Expected strong response:**
- Explains trade-offs, not dogma.
- URL versioning is explicit and easier to route, cache, and debug.
- Header-based versioning keeps URLs cleaner but can be less visible and harder to test/debug operationally.
- Considers consumers, tooling, observability, and rollout cost.
- Mentions compatibility strategy beyond the versioning mechanism itself.

### 39. Parallel batch processing
**Question:** A job processing 10,000 cases sequentially takes two hours. How would you speed it up safely?

**Expected strong response:**
- Breaks work into chunks and controlled concurrency.
- Considers DB contention, duplicate processing, ordering needs, API rate limits, and downstream pressure.
- Uses idempotent work units and checkpointing.
- Measures throughput before and after.
- Knows that “add threads” without backpressure is not a real plan.

### 40. When not to use JPA
**Question:** In a Spring application, when would you deliberately avoid JPA/Hibernate for a specific use case?

**Expected strong response:**
- Mentions bulk operations, reporting queries, complex joins, performance-critical read paths, or SQL-native features.
- Chooses projections, JDBC, or native SQL when object mapping becomes counterproductive.
- Still respects transaction management and maintainability.
- Avoids ideology: JPA is useful, but not universal.
- Shows tool-choice maturity.

---


<sup>[? Back to top](#table-of-contents)</sup>

# Section 3 — Debugging, Logic, and Practical Engineering Exercises

This section tests how the candidate reasons through concrete technical problems. It is intentionally more practical than trivia.

---

## Level 1 — Mid

### 41. SQL exercise: cases with no documents
**Question:** Given `cases(id, status, created_at)` and `documents(id, case_id, doc_type)`, write a query to find cases created in the last 30 days that have no documents.

**Expected strong response:**
- Uses `LEFT JOIN ... WHERE documents.case_id IS NULL` or `NOT EXISTS` correctly.
- Filters on the 30-day creation window.
- Understands duplicate-row pitfalls if using joins carelessly.
- Can explain why `NOT EXISTS` is often robust/readable.
- Example answer is logically correct even if syntax differs slightly by SQL dialect.

### 42. Java stream exercise
**Question:** Write or describe a stream pipeline that takes `List<Order>`, keeps orders with `amount > 1000`, groups them by customer name, and returns `Map<String, List<Order>>`.

**Expected strong response:**
- Uses `stream()`, `filter(...)`, and `Collectors.groupingBy(...)`.
- Produces `Map<String, List<Order>>` keyed by customer name.
- Can explain the pipeline clearly.
- Does not overcomplicate the solution.
- Recognizes when a loop would also be acceptable in production if clearer.

### 43. Tracing intermittent 500s
**Question:** A client reports intermittent 500 errors on a flow that touches four services. How do you investigate?

**Expected strong response:**
- Starts with correlation/request IDs and time window.
- Looks at logs, metrics, traces, dependency health, and recent deploys.
- Distinguishes between reproducible and intermittent failure patterns.
- Checks timeout boundaries, downstream errors, retries, and data-specific causes.
- Avoids random guessing or starting from code before narrowing scope.

### 44. Duplicate request logic exercise
**Question:** Sketch the server-side logic for handling duplicate submission retries safely when the same idempotency key arrives twice.

**Expected strong response:**
- Persists the key with request fingerprint and processing state.
- On duplicate, returns stored result or current status instead of re-running work.
- Protects against race conditions with atomic insert or unique constraint.
- Considers what happens if the first request is still in progress.
- Understands that key lifecycle and retention matter.

### 45. Silent scheduled-job failure
**Question:** A job runs every 10 minutes but suddenly stops producing expected business outcomes. Logs show nothing obvious. What do you inspect first?

**Expected strong response:**
- Checks whether the scheduler still triggers.
- Looks at success/failure counters, duration metrics, queue depth, processed-count metrics, and heartbeats.
- Verifies upstream inputs and downstream side effects, not just app logs.
- Considers swallowed exceptions, feature flags, empty data sets, and configuration drift.
- Thinks in terms of observability gaps as well as code bugs.

### 46. 401 after deploy
**Question:** Right after a release, downstream calls begin returning `401 Unauthorized`. What is your troubleshooting sequence?

**Expected strong response:**
- Checks changed configuration, credentials, scopes, token audience/provider, and registration wiring.
- Compares new release behavior against previous working version.
- Verifies whether all endpoints fail or only one integration.
- Uses logs/metrics to determine if tokens are missing, expired, malformed, or under-scoped.
- Thinks about rollback/canary containment while investigating.

### 47. Reducing cognitive complexity
**Question:** You inherit a method with deeply nested `if/else` blocks handling many business cases. How would you refactor it safely?

**Expected strong response:**
- Starts with tests around current behavior.
- Uses guard clauses, extracted methods, named predicates, or strategy objects.
- Separates validation, branching logic, and side effects.
- Improves naming and removes hidden coupling.
- Understands that readability and safe change come before clever patterns.

### 48. Speeding up a sequential job
**Question:** You need to make a long-running sequential batch faster. What questions do you ask before parallelizing it?

**Expected strong response:**
- Asks whether work units are independent.
- Checks ordering requirements, shared mutable state, DB locks, downstream rate limits, and idempotency.
- Measures where time is actually spent before redesigning.
- Considers chunking, batching, or reducing remote round-trips.
- Treats concurrency as one tool, not the first reflex.

### 49. Small Java correctness check
**Question:** What happens if you pass `null` to `Optional.of()`? And why can `==` sometimes appear to “work” for `Integer` values between `-128` and `127`?

**Expected strong response:**
- `Optional.of(null)` throws `NullPointerException`.
- `Optional.ofNullable(null)` gives an empty optional.
- Explains `==` compares references for boxed integers.
- Mentions integer caching for the small range.
- Makes clear that `.equals()` is the safe value comparison for objects.

### 50. Mock, spy, or real instance?
**Question:** In a unit test, how do you decide whether to use a mock, a spy, or a real object?

**Expected strong response:**
- Real objects for simple deterministic domain logic.
- Mocks for true external collaborators or hard-to-control dependencies.
- Spies only when partial real behavior is valuable and complexity is justified.
- Warns against over-mocking implementation details.
- Prioritizes tests that remain stable when refactoring behavior-preserving internals.

---

## Level 2 — Senior

### 51. Retry policy design exercise
**Question:** Design a retry policy for outbound HTTP calls. Which failures would you retry, which would you not retry, and how would you stop retries from making an outage worse?

**Expected strong response:**
- Retries transient failures like timeouts or some 5xx responses, not validation/business 4xx errors.
- Uses backoff and jitter.
- Caps retry count and total time budget.
- Coordinates retries with circuit breaking, rate limits, and idempotency.
- Recognizes retry storms and thundering herd failure modes.

### 52. Idempotent pipeline proof
**Question:** We have a three-step pipeline: trigger email generation, poll generation status, send the result. Explain how you would prove the pipeline is idempotent.

**Expected strong response:**
- Defines state machine and allowed transitions.
- Shows how each step behaves when repeated.
- Identifies the side effects that must happen exactly once or at least once with deduplication.
- Uses persisted correlation IDs and terminal states.
- Can reason about crashes between steps and resume behavior.

### 53. Parallel update contention
**Question:** You parallelize a case-processing job and performance gets worse. What kinds of contention or coordination issues do you suspect?

**Expected strong response:**
- DB row/index lock contention.
- Connection pool exhaustion.
- Downstream API throttling.
- Hot partitions / skewed work distribution.
- Excessive retries, thread contention, or over-serialized critical sections.

### 54. Data-fix strategy exercise
**Question:** A bug corrupted document statuses for 200 production cases over the last three weeks. Walk me through the safest recovery strategy.

**Expected strong response:**
- Contains the bug first.
- Identifies affected records precisely and validates scope.
- Chooses reversible, reviewed, auditable repair steps.
- Often prefers a controlled migration/script with backups and verification queries.
- Includes business communication, sampling, and post-fix monitoring.

### 55. Diagnosing N+1 from symptoms
**Question:** An endpoint is suddenly slow in production after a feature change. CPU is fine, but DB queries per request jumped from 5 to 300. How do you confirm and fix the likely issue?

**Expected strong response:**
- Quickly suspects N+1 or accidental repeated fetches.
- Uses SQL logs/APM/trace spans to map queries to code path.
- Reviews newly added entity traversal or serialization changes.
- Fixes with targeted fetch strategy or projection.
- Verifies improvement with query count and latency measurements.

### 56. Choosing sync vs async integration
**Question:** An external AI service can take 30 seconds to produce email content. Would you use synchronous REST, async polling, callbacks/webhooks, or a queue-based design?

**Expected strong response:**
- Usually rejects keeping the user waiting synchronously.
- Proposes async job submission with status tracking.
- Chooses between polling/webhook/queue based on provider capabilities and operational control.
- Includes retries, idempotency, timeout budgets, and user-facing status.
- Mentions observability and dead-letter/failure handling.

### 57. Scheduling / fairness exercise
**Question:** You must process outreach jobs in chunks of 10, respect a rate limit of 50 requests per second, and avoid one large client starving smaller ones. Sketch your scheduling approach.

**Expected strong response:**
- Uses queues per tenant/team/client or weighted fair scheduling.
- Applies token bucket / rate limiter globally or per downstream.
- Processes chunked units with bounded concurrency.
- Includes visibility into lag, age, and per-client fairness.
- Understands that throughput and fairness can conflict and need explicit policy.

### 58. Full-stack duplicate-action scenario
**Question:** The frontend times out waiting for a response, the backend actually finishes, and the user clicks the button again. How do you design the full flow to avoid duplicate actions and confusing UX?

**Expected strong response:**
- Uses idempotency on the backend.
- Gives the client a request ID / operation ID and a status endpoint or resumable UI flow.
- Disables repeated submission where sensible, but does not rely on UI alone.
- Returns eventual status clearly.
- Aligns client, API, persistence, and job processing behavior.

### 59. Stale cache logic exercise
**Question:** A cached reference dataset is occasionally stale, causing wrong decisions for a few minutes. How do you reason about whether this is acceptable, and what mitigations exist?

**Expected strong response:**
- Starts with business impact and correctness requirements.
- Distinguishes acceptable eventual consistency from harmful inconsistency.
- Evaluates TTL, explicit invalidation, versioning, stale-while-revalidate, or bypass on critical actions.
- Considers multi-instance behavior.
- Makes the freshness policy explicit instead of accidental.

### 60. Observability design exercise
**Question:** If you had to make a multi-service workflow debuggable in production tomorrow, what exact logs, metrics, and traces would you add first?

**Expected strong response:**
- Logs: structured events with correlation ID, business key, state change, downstream target, error category.
- Metrics: success/failure counts, latency, retries, queue depth, age of oldest item, throughput.
- Traces: spans across service boundaries and external calls.
- Makes cardinality-conscious choices.
- Includes alerts tied to user/business outcomes, not only exceptions.

---


<sup>[? Back to top](#table-of-contents)</sup>

# Section 4 — Architecture, Trade-offs, and Full-Stack Decision Making

This section is where strong candidates separate themselves: architecture, trade-offs, rollout, observability, and business-aware reasoning.

---

## Level 1 — Mid

### 61. Adding a new external integration
**Question:** You need to integrate a new third-party document verification API. Walk me through the layers and responsibilities you would create from controller to external call.

**Expected strong response:**
- Separates controller/API contract, service/orchestration, client/adapter, DTO mapping, persistence if needed.
- Avoids leaking external payloads through internal APIs.
- Includes auth, timeout, retry, error mapping, and logging.
- Mentions test strategy and config separation.
- Thinks about future change isolation.

### 62. Adding a field sourced from another service
**Question:** A product owner asks for a new field in an existing API response, but the data comes from another microservice. What questions do you ask before implementing it?

**Expected strong response:**
- Asks about freshness requirements, latency budget, failure behavior, ownership, and access control.
- Checks whether the field belongs in this API contract.
- Considers caching, pre-computation, async enrichment, or keeping the boundary separate.
- Thinks about backward compatibility and observability.
- Avoids assuming a synchronous call is always acceptable.

### 63. Async user experience design
**Question:** A user starts a process that may take 30 seconds. How would you design the API and UX so the user does not sit on a spinning loader?

**Expected strong response:**
- Returns an accepted response with operation/job ID.
- Provides polling endpoint, callback, websocket, or notification strategy as appropriate.
- Persists status and failure reason.
- Makes the operation safe to retry.
- Aligns frontend expectations with backend processing reality.

### 64. Pagination and filtering
**Question:** How would you design a KYC cases listing API that may return thousands of records?

**Expected strong response:**
- Uses pagination and explicit filter/sort parameters.
- Includes page metadata or cursor info.
- Thinks about stable ordering.
- Avoids expensive unrestricted queries by default.
- Considers frontend usability and DB efficiency together.

### 65. Heavy report generation endpoint
**Question:** A stakeholder asks for an endpoint that generates a large report and downloads it immediately. How do you decide whether it should stay synchronous or become asynchronous?

**Expected strong response:**
- Considers runtime, memory usage, user experience, infrastructure limits, and retry behavior.
- Favors async if generation is heavy or failure-prone.
- Uses sync only when the response is predictably fast and safe.
- Thinks about caching or pre-generation when relevant.
- Includes operational considerations, not just API elegance.

### 66. Rate limiting design
**Question:** When and where would you apply rate limiting in a backend platform like ours?

**Expected strong response:**
- Mentions external/public entry points first.
- May also rate-limit expensive internal operations or protect downstream dependencies.
- Differentiates per-client, per-user, and global protection.
- Includes observability and fair-use behavior.
- Understands rate limiting is both security and resilience control.

### 67. Versioning decision
**Question:** For an API used by an SPA and several backend consumers, how would you choose a versioning approach and migration plan?

**Expected strong response:**
- Chooses a versioning approach with operational reasons.
- Plans coexistence period and deprecation communication.
- Avoids unnecessary breaking changes.
- Uses contract tests or consumer coordination when relevant.
- Understands the migration plan matters as much as the version label.

### 68. REST vs event/poller
**Question:** A downstream system is slow and unreliable. When would you prefer synchronous REST, and when would you switch to an event-driven or poll-based workflow?

**Expected strong response:**
- Uses synchronous calls for fast, required, user-facing data where immediate consistency is needed.
- Uses async patterns when latency, retries, or reliability constraints make sync fragile.
- Discusses operational complexity trade-offs.
- Considers user experience and business semantics.
- Understands that decoupling usually shifts complexity rather than removing it.

### 69. Feature flags and conditional behavior
**Question:** You need to release a new integration gradually. How would you use configuration or feature flags to reduce rollout risk?

**Expected strong response:**
- Uses flags/config to enable by environment, cohort, or path.
- Keeps the off-path safe and observable.
- Avoids permanent dead branches by planning cleanup.
- Monitors both old and new paths during rollout.
- Understands flags are a release tool, not a substitute for design.

### 70. Readiness, liveness, startup behavior
**Question:** What is the difference between readiness and liveness checks, and why does it matter in containerized deployments?

**Expected strong response:**
- Liveness: is the process alive enough to restart?
- Readiness: can it serve traffic correctly right now?
- Explains why confusing them causes restart loops or bad traffic routing.
- Considers dependencies, warm-up, migrations, and background workers.
- Connects health checks to safe deployment behavior.

---

## Level 2 — Senior

### 71. Synchronous microservices vs event-driven boundaries
**Question:** In a platform with many Spring Boot services currently talking over REST, how do you decide which interactions should remain synchronous and which should be redesigned around events or jobs?

**Expected strong response:**
- Uses business semantics first: immediacy, consistency needs, user-facing latency, ownership boundaries.
- Weighs failure isolation, coupling, replayability, and observability.
- Recognizes async flows add state-management and operational complexity.
- Avoids pushing everything to events or everything to REST.
- Can articulate a boundary-by-boundary strategy.

### 72. End-to-end traceability design
**Question:** If a request spans four services and one scheduler before completing, how would you design traceability so on-call engineers can debug it quickly?

**Expected strong response:**
- Uses correlation IDs propagated across sync and async boundaries.
- Captures business identifiers carefully and safely.
- Aligns structured logs, metrics, and traces.
- Includes status/state transitions for async work.
- Makes failure localization practical, not theoretical.

### 73. Availability vs consistency under partition
**Question:** Suppose the document collection capability must stay available during a network partition. What trade-off are you accepting, and how do you contain the risk?

**Expected strong response:**
- Understands the CAP-style trade-off in practical terms.
- Accepts delayed consistency or reconciliation in some form.
- Defines what can be eventually consistent and what cannot.
- Adds repair/reconciliation, user messaging, and conflict handling.
- Avoids using CAP as a slogan instead of a system design constraint.

### 74. Downstream slowness design challenge
**Question:** One critical downstream service becomes slow but not completely down. How do you prevent it from degrading the whole platform?

**Expected strong response:**
- Timeouts that are realistic and bounded.
- Bulkheads or isolation to protect thread pools/resources.
- Circuit breaking or load shedding where appropriate.
- Fallbacks or degraded modes only when business-safe.
- Uses metrics/alerts to detect partial brownouts, not just hard failures.

### 75. Breaking response change rollout
**Question:** You must make a breaking response change that impacts an SPA and two backend consumers. What rollout plan would you choose?

**Expected strong response:**
- Usually prefers compatibility first: additive change, dual field/version, or new endpoint if possible.
- If breaking is unavoidable, coordinates consumers explicitly and stages rollout.
- Uses contract tests and release sequencing.
- Includes observability, deprecation timeline, and rollback plan.
- Understands the social coordination side of architecture.

### 76. Service boundary decision
**Question:** How do you decide whether a new capability should become a new microservice, a module in an existing service, or just a well-isolated component in the current codebase?

**Expected strong response:**
- Uses domain ownership, deployability needs, team topology, scalability profile, and operational cost.
- Recognizes microservices add coordination and platform overhead.
- Understands that modules can be the right answer for a long time.
- Avoids “microservices by default.”
- Makes boundaries based on change patterns and business cohesion.

### 77. Security architecture across integrations
**Question:** We call many downstream systems with different scopes and sometimes different grant types. What security principles should shape the integration architecture?

**Expected strong response:**
- Least privilege.
- Separate identities/registrations by integration or responsibility.
- Clear secret handling and rotation approach.
- Auditability and intentional propagation of user context only when needed.
- Avoids over-broad tokens and hidden trust chains.

### 78. Zero-downtime schema + code rollout
**Question:** How would you roll out a database schema change and application change with minimal or zero downtime across multiple instances?

**Expected strong response:**
- Uses expand-and-contract style thinking.
- Applies backward-compatible schema first when possible.
- Deploys code that can handle both old and new states during transition.
- Plans data backfill if needed.
- Verifies rollback implications before release.

### 79. First five minutes of a bad release
**Question:** You deploy a new version and within five minutes the error rate jumps from 0.1% to 15%. What happens in your first five minutes?

**Expected strong response:**
- Contains blast radius quickly: rollback, disable flag, route away, or pause rollout.
- Assigns investigation and communication in parallel.
- Uses metrics/logs/traces/recent-change analysis, not guesswork.
- Keeps stakeholders informed with facts and next checkpoint.
- Understands incident response is operational leadership, not solo debugging.

### 80. Opinionated architecture prompt
**Question:** In a regulated microservices platform like ours, what is one thing you would deliberately **not** build even if a smart engineer argued for it?

**Expected strong response:**
- Gives a thoughtful boundary such as premature event sourcing, over-generic frameworks, custom orchestration platform, speculative shared kernel, or full reactive rewrite without clear need.
- Explains the maintenance and operational costs.
- Grounds the opinion in team size, domain, compliance, and support burden.
- Acknowledges when the rejected idea *would* make sense.
- Shows restraint as an architectural skill.

---


<sup>[? Back to top](#table-of-contents)</sup>

# Interviewer Notes: What strong candidates tend to do

Strong candidates usually:
- ask clarifying questions before solving
- name assumptions explicitly
- reason in trade-offs instead of absolutes
- think about failure modes early
- include observability and rollout, not just design shape
- connect technical decisions to user impact and team cost
- give specific examples from experience
- know when to be pragmatic and when to be strict

Weak signals include:
- purely textbook answers with no context
- strong opinions with no trade-off awareness
- “I would just…” answers that ignore rollout, failure, or operations
- blaming teammates or other teams in behavioral stories
- confusing framework syntax with system design understanding
- treating testing, monitoring, or security as afterthoughts

---


<sup>[? Back to top](#table-of-contents)</sup>

# Suggested scoring rubric

## Mid
A good Mid-level candidate should:
- answer most **Mid** questions with confidence
- give at least a few strong answers in **Senior** follow-ups
- show ownership, sound coding habits, and solid debugging instincts
- understand API, database, testing, and operational basics

## Senior
A good Senior-level candidate should:
- answer most **Senior** questions with depth and trade-off awareness
- bring real examples, not only theory
- think across system boundaries: API, data, jobs, rollout, observability, security, and people
- demonstrate judgment under ambiguity and delivery pressure
- show mature collaboration and conflict-handling behavior

---


<sup>[? Back to top](#table-of-contents)</sup>

# Recommended interview assembly examples

## Example: Mid Backend Engineer
- Section 1 Mid: 4 questions
- Section 2 Mid: 4 questions
- Section 3 Mid: 4 questions
- Section 4 Mid: 3 questions
- Optional 2 Senior follow-ups

## Example: Senior Backend Engineer
- Section 1 Senior: 4 questions
- Section 2 Senior: 4 questions
- Section 3 Senior: 4 questions
- Section 4 Senior: 4 questions
- Optional live deep dive on one real production incident story

## Example: Tech Lead / Staff-oriented interview
- Section 1 Senior: 5 questions
- Section 2 Senior: 3 questions
- Section 3 Senior: 3 questions
- Section 4 Senior: 6 questions
- Strong focus on trade-offs, leadership, and incident handling

---


<sup>[? Back to top](#table-of-contents)</sup>

# Section 5 — Timed Algorithmic, Logic, and Review Exercises (Add-on Pack)

Use this pack when you want a fast signal on raw reasoning and practical engineering quality.

Time guidance:
- **10 minutes** verbal reasoning (no coding)
- **15 minutes** pseudocode or high-level design
- Avoid full IDE coding unless it is a dedicated coding round

---

## 5A. LeetCode-style exercises (data structures / algorithms)

### Mid (pick 2 to 3)

#### L-M1. Merge overlapping maintenance windows
**Prompt:**
Given a list of intervals `[(start, end)]` representing maintenance windows for dependencies, return a minimal list of merged windows.

Example:
`[(1,3), (2,4), (6,8), (7,9)] -> [(1,4), (6,9)]`

**Expected strong approach:**
- Sort intervals by start.
- Iterate once and merge if current start is <= last merged end.
- O(n log n) because of sorting.
- Handles empty input and single interval.

**Hints:**
- **Hint 1 (light):** Think about what condition means two windows overlap.
- **Hint 2 (medium):** If intervals are sorted by start, do you still need nested loops?
- **Hint 3 (strong):** Keep a `merged` list; compare each interval with `merged.getLast()` and extend the end with `max(...)`.

#### L-M2. Longest consecutive case-ID streak
**Prompt:**
Given an unsorted array of integer case IDs, return the length of the longest consecutive sequence.

Example:
`[100,4,200,1,3,2] -> 4` (sequence `1,2,3,4`)

**Expected strong approach:**
- Use a hash set for O(1) lookup.
- Only start counting from numbers that have no predecessor (`x-1` not in set).
- O(n) average time.
- Avoids sorting-based O(n log n) unless candidate explains trade-off clearly.

**Hints:**
- **Hint 1 (light):** Sorting works, but can you do better than O(n log n)?
- **Hint 2 (medium):** Fast membership checks help detect sequence boundaries.
- **Hint 3 (strong):** A number is a sequence start only when `x-1` is missing; then walk `x+1, x+2...`.

#### L-M3. First duplicate submission in event stream
**Prompt:**
Given a stream/list of submission IDs, return the first ID that appears twice (based on second occurrence index). If none, return empty.

Example:
`[A, B, C, B, A] -> B`

**Expected strong approach:**
- Use a set of seen IDs.
- Traverse once; first time `id` already in set, return it.
- O(n) time, O(n) space.
- Mentions ordered set/map variants only if requirements change.

**Hints:**
- **Hint 1 (light):** You do not need pairwise comparisons.
- **Hint 2 (medium):** You only need to know if something was seen before.
- **Hint 3 (strong):** Single pass + `HashSet`; return on first repeat.

### Senior (pick 2 to 3)

#### L-S1. Minimum log window containing required event types
**Prompt:**
Given a sequence of event types and a required set `{AUTH, CASE_READ, CASE_UPDATE}`, find the shortest contiguous window containing all required types.

**Expected strong approach:**
- Sliding window with two pointers.
- Frequency map for required types and count of how many are satisfied.
- Expand right until valid, then shrink left to minimum.
- O(n) time.

**Hints:**
- **Hint 1 (light):** Brute force checks every subarray; can this be incremental?
- **Hint 2 (medium):** Track counts per required type while moving pointers.
- **Hint 3 (strong):** Standard minimum-window-substring pattern adapted to event arrays.

#### L-S2. Top K failing endpoints with deterministic tie-break
**Prompt:**
You receive `(endpoint, statusCode)` records. Return top `k` endpoints by number of failures (`5xx`). If tied, sort lexicographically.

**Expected strong approach:**
- Count failures per endpoint using hash map.
- Use heap of size `k` or sort by `(count desc, endpoint asc)` depending on `n` and `k`.
- Explains complexity trade-off.
- Mentions stable deterministic output.

**Hints:**
- **Hint 1 (light):** Split into counting and ranking phases.
- **Hint 2 (medium):** `Map<String,Integer>` for counting, then partial order for top `k`.
- **Hint 3 (strong):** Min-heap size `k` with comparator on `(count, reverse endpoint)` then reverse output.

#### L-S3. Weighted fair scheduling simulation
**Prompt:**
Given queues for clients with weights (A:3, B:1, C:1), output the next 10 picks so heavier clients get proportionally more slots without starving others.

**Expected strong approach:**
- Uses weighted round-robin / deficit round-robin concept.
- Demonstrates fairness and no starvation.
- Can produce a deterministic pick sequence.
- Discusses how to adapt when one queue is empty.

**Hints:**
- **Hint 1 (light):** Pure FIFO across all queues may starve small clients if one is always full.
- **Hint 2 (medium):** Think in terms of “credits” or “tokens” per client.
- **Hint 3 (strong):** Add weight credits each cycle; pick while credits > 0 and queue non-empty, then decrement.

---

## 5B. Complete-code style exercises (service-level, practical backend)

These are not puzzle-only; they test if the candidate can shape robust production-like logic.

### Mid (pick 2 to 3)

#### C-M1. Idempotent `POST /cases/{id}/actions`
**Prompt:**
Design pseudocode for a handler that accepts `Idempotency-Key`, creates an action once, and returns prior result for retries.

**Expected strong approach:**
- Validate mandatory inputs.
- Attempt atomic insert of idempotency key + request fingerprint + status.
- If key exists with same fingerprint, return stored result/status.
- If key exists with different fingerprint, return conflict.
- Persist response payload or operation reference for replay.

**Hints:**
- **Hint 1 (light):** API-level idempotency must be backed by persistence.
- **Hint 2 (medium):** Define states like `IN_PROGRESS`, `DONE`, `FAILED_RETRYABLE`.
- **Hint 3 (strong):** Use unique constraint on `(idempotencyKey, endpointScope)` + transactional upsert flow.

#### C-M2. Chunked processor with rate limit
**Prompt:**
You process 1,000 items in chunks of 10. Downstream allows 50 req/s. Sketch pseudocode with bounded concurrency and retryable vs non-retryable errors.

**Expected strong approach:**
- Chunk items deterministically.
- Use fixed worker pool + global/token-bucket limiter.
- Retry transient failures with capped exponential backoff.
- No retry for validation/business errors.
- Emit metrics per outcome and chunk latency.

**Hints:**
- **Hint 1 (light):** Separate scheduling, execution, and retry policy concerns.
- **Hint 2 (medium):** Concurrency control alone is not enough; enforce QPS explicitly.
- **Hint 3 (strong):** `for each chunk -> submit task`; each task acquires rate-limit token before downstream call.

#### C-M3. Safe state transition function
**Prompt:**
Implement pseudocode for `transitionCaseStatus(caseId, expectedCurrent, target)` that avoids lost updates under concurrent workers.

**Expected strong approach:**
- Reads current state and validates allowed transition.
- Uses optimistic locking/version or conditional update (`WHERE id=? AND status=?`).
- Returns explicit outcome (`UPDATED`, `STALE`, `INVALID_TRANSITION`).
- Avoids blind overwrite.
- Includes audit log/event emit on success.

**Hints:**
- **Hint 1 (light):** Two workers can read same status and both try to update.
- **Hint 2 (medium):** The update statement itself can enforce preconditions.
- **Hint 3 (strong):** Use affected-row count from conditional update to detect concurrency conflict.

### Senior (pick 2 to 3)

#### C-S1. Trigger -> Poll -> Act orchestrator (repo-inspired)
**Prompt:**
Design pseudocode for a 3-step pipeline similar to outreach jobs: trigger external generation, poll status, apply result. Must be restart-safe and idempotent.

**Expected strong approach:**
- Explicit persisted state machine per request.
- Separate scheduler entry points for each step.
- Each step checks state guard before side effects.
- Retries are bounded and transition to terminal failure with reason.
- Supports resume after crash and avoids duplicate “act” step.

**Hints:**
- **Hint 1 (light):** Define states before writing logic.
- **Hint 2 (medium):** Persist correlation IDs and last polled status.
- **Hint 3 (strong):** Terminal states like `COMPLETED`/`FAILED_FINAL`; every job starts with “if terminal -> noop”.

#### C-S2. Outbox publisher with at-least-once semantics
**Prompt:**
You update DB rows and must publish events reliably without distributed transactions. Sketch an outbox worker and failure handling.

**Expected strong approach:**
- Transaction writes business change + outbox record together.
- Worker polls outbox, publishes, marks sent atomically (or with safe compare-and-set).
- Deduplication key for consumers (idempotent consumer).
- Retry with backoff and dead-letter policy for poison events.
- Observability: lag, retries, oldest unsent age.

**Hints:**
- **Hint 1 (light):** Separate “store intent” from “publish eventually”.
- **Hint 2 (medium):** Publisher can crash after publish before marking sent.
- **Hint 3 (strong):** Accept at-least-once delivery; enforce idempotency downstream.

#### C-S3. Adaptive retry/circuit strategy wrapper
**Prompt:**
Design a client wrapper deciding dynamically between retry, fail-fast, and open-circuit behavior based on error class and recent failure rate.

**Expected strong approach:**
- Classifies errors (`timeout`, `5xx`, `429`, `4xx business`).
- Retry only safe/transient categories.
- Circuit state machine with half-open probing.
- Jittered backoff and retry budget.
- Clear fallback behavior and metrics.

**Hints:**
- **Hint 1 (light):** Not all failures deserve retries.
- **Hint 2 (medium):** Repeated retries during outage can amplify failure.
- **Hint 3 (strong):** Combine retry policy + circuit breaker + rate limiter, each with explicit thresholds.

---

## 5C. PR code-review exercises (anti-patterns + trade-offs)

Goal: evaluate how the candidate reviews code, not just writes code.

### Mid (pick 2 to 3)

#### R-M1. PR review mini-feature: downstream enrichment endpoint
**Prompt:**
Review this snippet and list issues you would comment on in a PR:

```java
@GetMapping("/cases/{id}")
public CaseResponse getCase(@PathVariable String id) {
    CaseEntity entity = caseRepository.findById(id).orElse(null);
    String ext = restTemplate.getForObject(externalUrl + "/details/" + id, String.class);
    entity.setExternalData(ext);
    caseRepository.save(entity);
    return mapper.toResponse(entity);
}
```

**What strong reviewers should spot:**
- Possible NPE when case not found.
- GET endpoint has side effect (`save`) and mutates state.
- Missing timeout/error handling around external call.
- No resilience strategy for downstream failure.
- Potentially wrong separation of concerns (controller doing orchestration + persistence).
- Missing structured logs/metrics around external dependency.

**Opinionated trade-off discussion to allow:**
- Whether to cache external data or fetch on demand.
- Whether to degrade gracefully (partial response) vs hard fail.

**Hints:**
- **Hint 1 (light):** Start with API contract semantics and status codes.
- **Hint 2 (medium):** Check null safety and side effects.
- **Hint 3 (strong):** Examine resilience, layering, and observability concerns.

#### R-M2. PR review mini-feature: transactional notification sender
**Prompt:**
Review this snippet and provide review comments:

```java
@Transactional
public void closeCase(String caseId) {
    CaseEntity c = repo.getReferenceById(caseId);
    c.setStatus("CLOSED");
    repo.save(c);
    notificationClient.sendCloseEmail(caseId); // external HTTP call
}
```

**What strong reviewers should spot:**
- External HTTP call inside DB transaction (lock duration / coupling risk).
- Missing error classification and retry policy.
- No idempotency protection for repeated close requests.
- Hardcoded status string; enum/value-object preferred.
- `getReferenceById` may defer failure until access.

**Opinionated trade-off discussion to allow:**
- Immediate send vs async outbox/event model.
- Strong consistency of “close + notify” vs eventual consistency.

**Hints:**
- **Hint 1 (light):** Where are transaction boundaries relative to network calls?
- **Hint 2 (medium):** What happens if email call times out after DB update?
- **Hint 3 (strong):** Suggest outbox or post-commit async strategy with idempotent notification.

#### R-M3. PR review mini-feature: broad catch and silent fallback
**Prompt:**
Review this method:

```java
public String enrich(String payload) {
    try {
        return externalService.call(payload);
    } catch (Exception e) {
        return "{}";
    }
}
```

**What strong reviewers should spot:**
- Broad `catch (Exception)` hides actionable failures.
- Silent fallback with empty JSON can corrupt downstream logic.
- Missing logs/metrics/trace context.
- No distinction between retryable and non-retryable failure.
- No explicit contract documenting fallback semantics.

**Opinionated trade-off discussion to allow:**
- Degraded fallback may be acceptable for non-critical fields if clearly signaled.

**Hints:**
- **Hint 1 (light):** Think about debuggability and observability.
- **Hint 2 (medium):** Is every exception equivalent?
- **Hint 3 (strong):** Replace generic catch with typed handling + explicit fallback flag.

### Senior (pick 2 to 3)

#### R-S1. PR review scenario: aggressive retries and timeout mismatch
**Prompt:**
Review this client config proposal:

```java
timeoutMs = 30000
maxRetries = 6
retryDelayMs = 200
parallelWorkers = 40
```

Downstream SLA is 2 seconds and rate limit is 50 req/s for the whole team.

**What strong reviewers should spot:**
- Retry policy likely amplifies outage (retry storm).
- Timeout is too high relative to SLA and thread utilization.
- Worker count + retries can violate downstream rate limits.
- Missing jitter, retry budget, circuit breaker, and backpressure.
- No per-error retry classification.

**Opinionated trade-off discussion to allow:**
- Throughput vs protection of downstream dependency.
- Fail-fast user experience vs aggressive eventual success attempts.

**Hints:**
- **Hint 1 (light):** Estimate worst-case request amplification.
- **Hint 2 (medium):** Compare latency budget to timeout + retries.
- **Hint 3 (strong):** Propose capped retries, jitter, lower timeout, circuit breaker, and explicit QPS limiter.

#### R-S2. PR review scenario: premature microservice extraction
**Prompt:**
A PR introduces a brand-new microservice for a small feature with one table and two endpoints, plus cross-service REST calls for each request. Review this architecture choice.

**What strong reviewers should spot:**
- Potential over-fragmentation and operational overhead.
- Added failure modes, auth/config complexity, and deployment coupling.
- Low domain autonomy may not justify service boundary.
- Missing migration/ownership rationale.
- Cost of observability/on-call burden not addressed.

**Opinionated trade-off discussion to allow:**
- Extraction is valid when team/domain independence, scale profile, or compliance boundary truly requires it.

**Hints:**
- **Hint 1 (light):** Ask what problem the new service solves that modularization cannot.
- **Hint 2 (medium):** List operational costs introduced by a new service.
- **Hint 3 (strong):** Suggest modular monolith / in-service module first, with explicit extraction triggers.

#### R-S3. PR review scenario: cache-first compliance-sensitive reads
**Prompt:**
Review this behavior proposal: “Always serve client risk category from 10-minute local cache; refresh async.”

**What strong reviewers should spot:**
- Potential compliance impact of stale risk category.
- No policy for critical operations requiring fresh reads.
- Multi-instance inconsistency and invalidation gaps.
- Missing fallback if refresh fails repeatedly.
- No observability on staleness age and decision impact.

**Opinionated trade-off discussion to allow:**
- Cache-first can be fine for read UX; may be unacceptable for decision-grade actions.

**Hints:**
- **Hint 1 (light):** Distinguish display data from decision data.
- **Hint 2 (medium):** What stale window is acceptable for each use case?
- **Hint 3 (strong):** Suggest dual-path policy: cache for non-critical reads, force-fresh checks for critical decisions.

---

## Suggested usage with existing interview flow

- Keep Sections 1 to 4 as the main interview backbone.
- Add **one exercise block** from Section 5 depending on role:
  - Mid role: 1 LeetCode-style + 1 complete-code + 1 review exercise
  - Senior role: 1 to 2 LeetCode-style + 1 complete-code + 1 review exercise
- If time is tight, ask for reasoning only and skip pseudocode.

---

*Core guide: 80 questions across 4 sections. Add-on pack: 18 timed exercises (6 LeetCode-style, 6 complete-code, 6 PR review), each with 3 progressive hints.*

