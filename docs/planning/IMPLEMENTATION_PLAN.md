# FederatedHackathon implementation plan

Version: 1.0 | Date: 2026-09-30 | Status: proposed implementation baseline

## 1. Outcome and authority

Build a complete BYOC platform where 500 humans and their independently operated agents can coordinate toward one continuously functional game. Participants select their own models, harnesses, hardware, and working methods. The platform controls the cost of crossing into shared coordination, verification, attention, and canonical state.

The first production deployment must include authentication, useful-work discovery, leases, candidate admission, bounded trusted validation, sequenced integration, promotion, event delivery, dependency unblocking, and moderation. A simulated runner or simulated Git promotion can support development but cannot satisfy this deployment gate.

Preserve these invariants throughout implementation:

1. Edge claims and peer evidence never confer canonical authority.
2. Leases communicate activity; they never establish exclusive ownership of files or tasks.
3. Every shared mutation is authenticated, authorized, attributable, and governed.
4. Every promotion identifies the exact validated tree, parent, constitution version, and evidence.
5. Required evidence gates cannot be weakened to clear a backlog.
6. Owner and organization budgets apply to all subordinate credentials and agents.
7. Canonical repository changes and their decisions are recoverably auditable.
8. Moderation and emergency controls remain available during overload.
9. Local private work is outside central compute quotas.
10. Scale claims require measured evidence from the actual workload.

## 2. Source interpretation

The v1.0 specification supplies the architecture and scope. The separate review supplies proposed corrections. This plan adopts its validator budgeting, integration queue, owner-budget inheritance, candidate adjudication, guarded replication incentives, class-aware leases, and explicit oracle requirements as implementation requirements.

Where the review proposes defaults, treat them as configurable policies to test, not proven optimal settings. Earliest qualifying L3 is the initial work-item selection policy; it is not automatic canonical promotion. Current-parent integration, risk policy, and constitution compliance still apply.

The source documents remain unchanged. This plan is an engineering translation, not a claim that the system already exists.

## 3. Initial technical decisions

| Area | Proposed baseline | Reason and decision gate |
| --- | --- | --- |
| Control plane | Python 3.12, FastAPI, modular monolith | Small operational surface and explicit domain modules. P00 confirms runtime/dependency compatibility. |
| Authority | PostgreSQL with versioned migrations | Transactions, unique constraints, row locks, and concurrent worker safety are needed from the first real loop. SQLite may be an edge cache, not a second authoritative backend. |
| Source history | Git; GitHub-compatible remote adapter | Local bare repositories support deterministic integration tests; remote promotion needs recovery/reconciliation. |
| Durable events | PostgreSQL event log and transactional outbox | State and event creation commit together; deliveries are at least once, with sequence-aware client deduplication. |
| Objects | S3-compatible interface and development filesystem adapter | Immutable checksummed evidence and builds, bounded upload and retention policies. |
| Client | Vanilla HTML/CSS/JS initially, REST and resumable WebSocket | Accessible operator UI without introducing a second large application framework. |
| Agent surface | CLI and MCP adapter over the same application services | One authorization and policy path; no bypass via alternate transports. |
| Authentication | OAuth identity mapping plus event invitations and scoped credentials | OAuth proves account control, not unique humanity. |
| Trusted execution | External isolated ephemeral runner with enforceable limits | Candidate code runs outside the control plane. No Docker requirement; choose a VM or equivalent isolation implementation in P04. |
| Presence/cache | Optional, introduced only after measurement | Never authoritative; Redis is a later option, not an initial prerequisite. |

These are proposed choices. Record binding decisions in ADRs during P00. Do not install dependencies, provision external services, or execute participant code merely to implement these documents.

## 4. Module boundaries

| Module | Owns | Must not own |
| --- | --- | --- |
| Identity | Owners, organizations, memberships, credentials, nodes, invitations | Candidate acceptance or canonical Git writes |
| Projects | Constitution versions, milestones, oracle definitions | Arbitrary execution commands from contributors |
| Work | Work items, dependency graph, cells, leases, challenges, contracts | Exclusive file ownership |
| Governor | Rate envelopes, hierarchical budgets, credit reservations, surface controls | Private compute restrictions or public reputation score |
| Candidates | Immutable source identity, artifacts, lineage, lifecycle, supersession | Self-issued trusted evidence |
| Validation | Jobs, assignments, evidence verification, measured usage | Final repository authority |
| Integration | Queue parents, integration revisions, evidence gates, decisions, promotion recovery | Unvalidated or arbitrary target-tree promotion |
| Events | Ordered durable changes, resumable transport, derived projections | Separate authoritative copies of domain state |
| Collaboration | Scoped messaging, blocks, discovery, activity indexes | Unbounded broadcast or unfenced global mutation |
| Operations | Moderation, circuit breakers, metrics, recovery, operator views | Hidden policy changes without audit |

Start in one codebase with separate domain/application/infrastructure concerns where they clarify authority. Do not create generic repository frameworks, plugin registries, agent execution engines, or microservices before a demonstrated need. Keep adapters concrete and interfaces narrow. CPU-heavy indexing and all validation are separate jobs, never blocking API request handling.

## 5. Data and transaction model

Authoritative entities include Owner/User, Organization, Membership, Credential, Agent, Node, Project, ConstitutionVersion, Milestone, WorkItem, Dependency, WorkCell, Lease, LeaseChallenge, InterfaceContract, Candidate, CandidateArtifact, Evidence, ValidationJob, CreditAccount, CreditReservation, CreditLedgerEntry, IntegrationRevision/Batch, AdjudicationDecision, PromotionIntent, Promotion, Rollback, Message, Flag, ModerationAction, GovernanceState, RatePolicy, ProjectEvent, OutboxEntry, and IdempotencyRecord.

Each table needs project/owner scope where applicable, explicit constraints, creation and modification provenance, and concurrency semantics. Candidate source commit/base and queued integration parent/tree are separate immutable identities. Idempotency keys are scoped to actor/project/operation and bind the request digest; reuse with another payload is an error.

Shared mutations write state, event, and outbox in one database transaction. Per-project durable sequence numbers are monotonically ordered by commit visibility; choose and test a transactional allocation mechanism rather than assuming an ordinary sequence prevents visibility inversions. Presence updates use a separate ephemeral channel and cannot consume the authoritative mutation log at presence frequency.

Git and PostgreSQL cannot share one atomic transaction. Promotion therefore uses an intent plus reconciliation protocol: persist fenced intent; compare-and-swap the protected remote ref from expected parent to validated commit; finalize record/outbox; recover crashes by observing whether the remote equals the parent, intended commit, or an unexpected value. Unexpected movement freezes promotion for review. One active authority is enforced with fencing and remote permissions, not process convention.

Recovery must cover a remote-success/database-finalize failure, lost network acknowledgment, worker lease expiry, duplicated messages, partial object uploads, and projection lag. See P05 and P07.

## 6. Validator economics and scheduling

Keep attention envelopes and runner resource credits separate. Define a versioned credit formula including CPU time, GPU class/time if used, memory class, and execution duration. A reservation bounds a job's maximum charge; reserve at admission, cap execution within that envelope, and settle from runner-measured usage. If insufficient credits remain, expose VALIDATION_BUDGET_CONSTRAINED with reason and next eligibility information. Do not start work on an unfunded promise.

Use owner, organization, and event ceilings together. Admission and reservation must be atomic under concurrency. Child credentials do not create fresh pools. Invitations grant starter budgets; useful verified work may raise access under capped policy. Invite linkage and abuse review remain private; IP addresses are signals, never automatic personhood proofs.

Scheduling combines risk, unblock value, queue age, cost, and reserved urgent/promotion capacity. Hard caps apply to jobs, retries, output, and total event spend. Reserve a defined platform-rerun pool so contributors are not charged for platform failure while the event still has a hard spending ceiling. Repeated contributor failures consume their configured budgets. Queue age must prevent starvation within documented eligibility rules.

## 7. Integration and adjudication

Keep L0 private, L1 published, L2 submitted, L3 verified, and L4 canonical distinct. Represent withdrawal, rejection, supersession, stale evidence, and queue failure explicitly. A candidate may pass verification and still lose selection or require fresh integration evidence.

Before a task accepts competing work, publish its selection policy, oracle, threshold, expected cost, and unresolved human judgment. Use server-assigned qualification sequence and candidate ID to resolve ordering deterministically. Comparative tasks use preregistered metrics; incomplete oracles use named maintainers and bounded review queues/deadlines. Deadlines escalate unresolved decisions, not auto-approve them.

The ordered integration queue constructs a revision on its current parent, validates that exact result, and promotes it with CAS. Source candidates remain immutable through rebase/merge. Initially serialize actual integration; parallel speculative descendants and compatible batching are later throughput improvements with explicit dependency invalidation and batch splitting.

Evidence cache keys include the actual tree or declared input digest, suite, environment/toolchain, constitution/policy version, and relevant dependencies. R0-R1 reuse is allowed only where policy proves inputs unchanged. R2-R5 integration checks target the actual result tree. Risk classification must be centrally computed from touched paths and change types; authors cannot lower their own risk class. P00 defines each risk class and required checks.

Rollback is an audited forward repository action restoring a known-good tree while preserving history. It fences promotion, invalidates descendant speculation and affected evidence, updates builds/work states as necessary, and resumes the queue against the restored parent.

## 8. Delivery phases and dependencies

Each phase contains implementation slices and a mandatory review/consolidation/documentation gate in TODO.md. Completion means evidence passes and records are current, not merely that endpoints exist.

| Phase | Deliverable | Depends on | Exit proof |
| --- | --- | --- | --- |
| P00 | Decisions, threat model, protocol/state contracts, traceability | Source documents | No unresolved authority or budget ambiguity in the first loop |
| P01 | Database, event/outbox core, project identity and credentials | P00 | Isolation, concurrency, replay, and revocation tests |
| P02 | Work graph, cells, leases, discovery, owner governor, moderation | P01 | Coordinated work remains bounded under contention and abuse |
| P03 | Candidate lifecycle, immutable evidence, admission and reservations | P02 | Cheap rejection, idempotency, and atomic budget enforcement |
| P04 | Real isolated runner and evidence pipeline | P03 | Hostile-code isolation and capped measured execution |
| P05 | Ordered queue, adjudication, promotion and rollback | P04 | Exact-tree promotion and crash recovery against real Git |
| P06 | First complete deployment: UI, CLI, streams, operator loop | P05 | Two contributors complete a real governed contribution loop |
| P07 | Security and operational reliability hardening | P06 | Failure drills, backup restore, and threat-model findings closed |
| P08 | Full collaboration: messaging, interfaces, presence, indexing, MCP | P06; P07 before public release | Cross-harness coordination uses the same governance path |
| P09 | Independent replication, rewards, speculation and batching | P07-P08 | No collusive credit minting; throughput gain without weaker gates |
| P10 | Measured capacity and 500-person event qualification | P07-P09 | Sustained target load within budgets and latency policy |
| P11 | Release, event operations, post-event consolidation | P10 | Reproducible release, practiced runbooks, public measured limits |
| P12 | Future scaling and additional domains | P11 and evidence of need | New bottlenecks/oracles measured before new claims |

Critical path: P00 -> P01 -> P02 -> P03 -> P04 -> P05 -> P06 -> P07 -> P10 -> P11. P08 and P09 complete the target feature set before the full event. Smaller invite-only rehearsals may occur after P07 using explicitly narrower capability and capacity promises.

Do not give calendar estimates before P00 and the real runner pilot establish effort and cost. At each phase exit, estimate the next phase using observed delivery rate, review effort, and unresolved dependencies. Identify a named implementer, reviewer, and operator per active slice; no unowned work enters the committed delivery window.

## 9. Repeated engineering checkpoints

Every PR: review authority, concurrency, error behavior, scope, meaningful tests, documentation, and newly introduced debt. Every three merged feature PRs OR weekly while actively developing, whichever comes first: stop feature merges for a consolidation checkpoint. Review the combined main branch, remove redundant abstractions and dead code, resolve migration/API drift, reconcile TODO and docs, and complete required cleanup before resuming.

Every phase: separate code/invariant review, consolidation/deslopping, and documentation/reproducibility gates. Every release: security, recovery, operator, and capacity gates. A failed checkpoint creates bounded corrective tasks; authority, security, budget, and data-loss defects block the next dependent phase.

The detailed rubric and evidence forms are in ENGINEERING_CHECKPOINTS.md. Reserve one consolidation slot per three feature PRs as a planning default; do not pad this with cosmetic churn or tests that simply mirror implementation. A no-change checkpoint is valid only with recorded evidence explaining why cleanup is unnecessary.

## 10. Verification and capacity policy

Use domain unit tests for state transitions and policy; real PostgreSQL tests for concurrency, locking, idempotency, and event ordering; real Git tests for revision identity, CAS, rollback, and crash reconciliation; runner adversarial tests for isolation/caps; API/CLI/WebSocket/MCP parity tests; and browser checks for real operator and contributor workflows. Stubbed tests are useful locally but cannot prove external guarantees.

The P10 baseline is 500 humans, 5,000 active agents, 2,000 advertised nodes, 1,000 simultaneous cells, 100 submissions/minute, and 50,000 presence/event updates/minute. Specify the split between ephemeral and durable updates. Distinguish offered, admitted, verified, and promoted rates. The spec's submission target is not a promise to execute 100 expensive validations per minute.

At P00 define benchmark latency/error targets and hardware profile; tune them with P04/P06 pilot data and freeze the event acceptance profile before P10. Include at least a 60-minute sustained target run, burst traffic, a deliberately constrained validator budget, and a recovery/soak run of at least four hours. Measure queue slope, age, repeated runner cost, invalidations per promotion, database lock pressure, reconnect behavior, and urgent-capacity protection. A bounded validator queue may defer/reject overload; its behavior must be documented, visible, and bounded.

Mandatory first-event signals: canonical playable/build health; API p50/p95/p99; budget estimated/reserved/actual; owner/event utilization; queue length/age; check cache hits; promotion failures; recovery time; event lag; moderation availability; operator review backlog; and storage/output consumption. Publish workload and configuration with results, not an unsupported scale badge.

## 11. Security and operations

Never run arbitrary participant commands inside the API, integrator, or privileged Git writer. Runner jobs receive narrowly scoped artifact channels and no production/control-plane credentials. Authentication, signature/nonce replay protection, environment binding, filesystem/network restrictions, process/CPU/memory/time/output limits, cache isolation, and dependency policies must be verified at the actual boundary.

Protect canonical refs from direct participant writes; validate remote permission configuration. Credential revocation, user/organization freezes, admission pause, messaging freeze, lease revocation, and promotion freeze require no deployment. Governance applies independently by surface; an event mute need not block code contribution. Shadow handling, if enabled, must explicitly mark non-authoritative state in internal audit and never manufacture canonical-looking success.

Define backup/restore, retention/deletion, secret rotation, audit access, orphan artifacts, operational incident ownership, and moderation appeals. Keep private abuse linkage out of public feeds and artifacts. If a required provider is unavailable, fail closed at authority/evidence gates and preserve readable status where possible.

## 12. Scope and readiness

M1 foundation: P00-P03, no canonical execution claim. M2 verified integration: P04-P05. M3 first complete governed deployment: P06-P07. M4 full event readiness: P08-P11. M5 measured expansion: P12.

Deferred until evidence justifies them: generic distributed compute marketplace, central model hosting, permissionless signup, public reputation scoring, microservice decomposition, multi-region authority, and 50,000-human claims. These deferrals do not remove any first-event moderation or recovery requirement.

Release blockers include unexplained authority gaps, unfenced promotion, unbounded validation, budget multiplication, missing moderation, unaudited winner decisions, fake runner integration, untested recovery, and unresolved critical/high security findings. Medium findings require a named owner and dated resolution accepted by the relevant gate owner.

## 13. Source coverage

| Source sections / review topic | Primary phases |
| --- | --- |
| Spec 1-4, 34, 36: boundary and goals | P00 and all phase reviews |
| Spec 5-6: identity and constitution | P01, P07 |
| Spec 7-8, 17: work, leases, discovery | P02, P08 |
| Spec 9-13, 28: repository, lifecycle, evidence, integration, rollback, supersession | P03-P05, P09 |
| Spec 14-16: governor, abuse, moderation | P02-P03, P06-P07, P10 |
| Spec 18-21: interfaces, messaging, protocol, activity index | P06, P08 |
| Spec 22-25: game checks, distributed validation, security, provenance | P03-P04, P07, P09 |
| Spec 26-27: observability and director | P06, P08, P10 |
| Spec 29-30: persistence, ordering, recovery | P01, P05, P07 |
| Spec 31-33: scale, full loop, stack | P00, P06, P10-P11 |
| Spec 35: generalization | P12 |
| Review 01: validator economics | P03-P04, P10 |
| Review 02: queue and exact-tree promotion | P05, P09-P10 |
| Review 03: identity multiplication | P01-P03, P07 |
| Review 04: selection and replication incentives | P02, P05, P09 |
| Review 05: leases, oracles, delivery order | P00, P02, P06, P10-P12 |
