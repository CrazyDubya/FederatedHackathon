# Phased implementation TODO

All tasks are initially unchecked. IDs are stable. Checking a task requires linked evidence, not an assertion of completion. See [plan](IMPLEMENTATION_PLAN.md) and [checkpoint rules](ENGINEERING_CHECKPOINTS.md).

For each active task record: implementer, reviewer, dependencies, PR, acceptance evidence, debt impact, documentation changes, and status. Status is planned / ready / active / blocked / review / complete. Store this in the task's issue or a linked phase record when implementation starts. Checkboxes summarize only complete status.

Split each phase into the suggested PR slices below. Slice order follows dependencies; combine tiny tasks only when they form one coherent outcome. Do not bundle unrelated changes to avoid review. Every three merged feature PRs, or weekly during active development, run checkpoint RC-n before further feature merges. Number checkpoint records monotonically. Phase exit order is C -> D -> R: consolidate first, finish documentation, then review the final implementation and documentation together. Preliminary review may inform cleanup, but it is not phase approval. Later changes require renewed review of the affected scope.

## P00 - Establish the executable design

Dependencies: supplied spec and recommendations. Suggested slices: decisions; state/protocol contracts; acceptance/threat model.

- [ ] P00-01 Create ADRs for runtime, PostgreSQL authority, modular boundaries, Git remote, runner isolation, object storage, OAuth, and no required Docker workflow; document alternatives and reversal triggers. [ADR deliverable](../adr/README.md) authored by Codex; status: review pending. Acceptance review: topic coverage, alternatives, verification gates, and reversal triggers; no runtime compatibility is claimed.
- [ ] P00-02 Define owners, organizations, memberships, invitation sponsorship, role matrix, credential scopes, and cross-project authorization rules.
- [ ] P00-03 Define L0-L4 transitions, work-item/cell/lease states, evidence classes, risk R0-R5, and required checks per class.
- [ ] P00-04 Specify REST contracts, error codes, idempotency semantics, pagination, versioning, event schema, WebSocket resumption, and MCP parity requirements.
- [ ] P00-05 Specify credit units/formula, job caps, hierarchical ceilings, refund/rerun rules, urgent capacity, queue bounds, and budget-constrained responses.
- [ ] P00-06 Specify oracle metadata and adjudication policies, qualification ordering, comparative metrics, maintainer escalation, and tie handling.
- [ ] P00-07 Define threat model/trust boundaries covering hostile code, credential compromise, identity multiplication, collusion, supply chain, split brain, and output attacks.
- [ ] P00-08 Define first-loop acceptance scenarios, load profiles, provisional latency/error/recovery objectives, benchmark hardware, and human-review capacity.
- [ ] P00-09 Create source-to-task traceability and a prioritized risk register; assign owners to the highest-risk experiments.
- [ ] P00-10 Author the seed-game brief, starter-repository layout, initial versioned constitution, first concrete work items, and reproducible oracle inputs/commands/thresholds. Define a bounded browser puzzle seed: move through one room, interact with a switch, open a door, reach an exit, and save/reload progress. Specify startup/performance measurement environment, forbidden changes, required checks, and human-only aesthetic criteria. Pin the seed commit and oracle version when built; P04-08 implements this project rather than an unrelated fixture; P05/P06 depend on its runnable baseline.
- [ ] P00-11 Run a development-only walking skeleton immediately after P00-03/P00-04/P00-10 and before P01-P03 hardening: one test contributor, temporary state adapters, fake runner with forced pass/fail, and a real local bare Git remote. Exercise discover -> lease -> submit -> reserve -> fake verify -> select -> local CAS -> event -> unblock, plus one rejection/retry. Record contract/state mistakes and corrective tasks; discard or explicitly replace shortcuts. This experiment satisfies no phase, deployment, security, authority, or capacity acceptance gate.
- [ ] P00-12 Initialize docs/engineering/DEBT.md, docs/engineering/checkpoints/, and docs/engineering/CADENCE.md with debt-entry schema, checkpoint template, feature PR counter, last passed checkpoint, last merge date, and next weekly due date; templates/zero state are not completed review records.
- [ ] P00-13 Verify the source manifest and committed text against the supplied PDF hashes; audit §13 coverage links and distinguish source requirements, adopted recommendations, and new engineering choices. Record extraction limitations and source-change review procedure.
- [ ] P00-C Consolidate duplicated requirements and invented layers; remove abstractions without concrete first-loop use; log justified deferrals.
- [ ] P00-D Publish architecture, domain glossary, protocol/state diagrams, ADRs, and acceptance profile; verify links and terminology.
- [ ] P00-R Review design against every constitutional invariant; resolve ambiguities at authority, money/resource, and recovery boundaries.

Exit evidence: one unambiguous first-loop design; every critical assumption has a test; proposed defaults are distinguished from requirements.

## P01 - Persistence, identity, projects, and durable events

Depends: P00. Slices: bootstrap/migrations; identity/project authorization; durable transaction/event core.

- [ ] P01-01 Bootstrap Python package, supported dependency pins, developer venv setup, formatting/static checks, test commands, and CI without external production credentials.
- [ ] P01-02 Add PostgreSQL migration framework; define project-scoped IDs, timestamps, constraints, and transaction helpers; verify fresh install and forward upgrade.
- [ ] P01-03 Implement owners, organizations, membership roles, invitations, OAuth account linkage, scoped credential issuance/hash storage, expiry, and revocation.
- [ ] P01-04 Implement subordinate agent/node registration and inherited ownership; prevent clients from rewriting their budget owner or role.
- [ ] P01-05 Implement projects, immutable constitution versions, approval-controlled constitution change, milestones, and effective policy snapshots.
- [ ] P01-06 Implement request authentication and project authorization at application services; negative tests cover cross-project and revoked credentials.
- [ ] P01-07 Implement payload-bound idempotency records and transactional state/event/outbox writes.
- [ ] P01-08 Implement per-project event ordering, outbox dispatch claims, duplicate delivery handling, and projection checkpoints.
- [ ] P01-09 Prove concurrent transactions cannot expose a later resumable sequence before an earlier committed event; define retention/resume-gap behavior.
- [ ] P01-10 Add audit-safe logging, configuration validation, health/readiness endpoints, and bounded request sizes.
- [ ] P01-C Remove redundant model/DTO conversions and generic wrappers; consolidate shared mutation/authorization paths; review dependency footprint.
- [ ] P01-D Document schema ownership, migrations, transaction semantics, credential lifecycle, configuration, and exact development commands.
- [ ] P01-R Review authorization placement, unique constraints, lost updates, event visibility, and secret handling using real PostgreSQL tests.

Exit evidence: fresh DB bootstrap; invitation-to-authenticated-project read; revocation and project isolation; transactional event replay under concurrency.

## P02 - Work coordination and owner-level governance

Depends: P01. Slices: work graph/oracles; cells/leases/discovery; governor/moderation. Run RC checkpoint after these slices.

- [ ] P02-01 Implement WorkItem states, acceptance/oracle/selection metadata, priority, task class, effort band, and checkpoint plan.
- [ ] P02-02 Implement dependencies, blocked/unblocked derivation, cycle detection and explicit approved cycle groups; avoid endless traversal.
- [ ] P02-03 Implement WorkCells with immutable base identity, expected paths, owner and attached agents; allow competing cells.
- [ ] P02-04 Implement transactional lease claims, class-aware duration, progress renewal, expiry, release, and restart-safe expiry processing.
- [ ] P02-05 Implement lease challenges with evidence thresholds, cooldowns, bounded renewals, moderator revocation, and audit.
- [ ] P02-06 Implement useful-work recommendations using skills, dependency value, contention, time band, and explainable reasons.
- [ ] P02-07 Implement rate envelopes and active-lease limits charged to credentials' verified owner and organization plus event ceilings.
- [ ] P02-08 Implement per-surface governance states, flag/report intake, mute, quarantine, suspension, ban, restoration, and appeal records.
- [ ] P02-09 Implement credential/user/org freezes, task creation pause, admission pause, and promotion freeze; controls work without deployment.
- [ ] P02-10 Test account/agent multiplication, concurrent lease acquisition, squat-and-renew patterns, high-volume useful contributors, and shared-household review.
- [ ] P02-C Consolidate policy enforcement, expiry machinery, and graph state updates; remove state duplicated across independent handlers.
- [ ] P02-D Document lease classes, renewals/challenges, owner ceilings, enforcement explanations, appeals, and operator emergency actions.
- [ ] P02-R Review race conditions and moderation privilege boundaries; prove competing private work remains possible despite lease contention.

Exit evidence: competing contributors discover/claim work; abusive shared actions are bounded; moderators restore participation; durable work remains readable.

## P03 - Candidate publication, evidence, and budgeted admission

Depends: P02. Slices: candidate/artifact model; cheap triage; atomic reservations.

- [ ] P03-01 Implement immutable candidate source commit/base, work cell, owner/agent lineage, touched paths, artifact references, and lifecycle history.
- [ ] P03-02 Implement publication versus submission, withdrawal, supersession, rejection, and idempotent retries without duplicate queue entries.
- [ ] P03-03 Verify source/base existence, diff bounds, forbidden paths, risk class, credential signatures/nonces, and constitution compatibility.
- [ ] P03-04 Reject malformed, empty, oversized, invalid, and already-superseded candidates before runner scheduling; flag likely duplicates without automatically suppressing useful competition.
- [ ] P03-05 Implement bounded artifact upload, checksum/content identity, content-type policy, completion verification, expiry, and orphan cleanup.
- [ ] P03-06 Define evidence bundle binding: candidate, base/tree, suite, environment/toolchain, nonce, result, logs, timestamps, and signer trust class.
- [ ] P03-07 Implement owner/org/event credit accounts, immutable ledger, reserve/settle/release state machine, and versioned estimates.
- [ ] P03-08 Make admission/reservation atomic; rejected/withdrawn/expired work releases unused reservation once; crashes cannot mint or lose credits.
- [ ] P03-09 Add VALIDATION_BUDGET_CONSTRAINED status with reason/eligibility; bound both participant and event queues and define retention/retry policy.
- [ ] P03-10 Add priority/age/cost/risk scheduling policy and reserved urgent/promotion capacity; expose estimates, actuals, queue age, and spend.
- [ ] P03-11 Test concurrent double-spend, new-agent budget resets, org membership budget changes, duplicate settlement, cache accounting, and platform versus contributor reruns.
- [ ] P03-C Consolidate lifecycle/policy/accounting implementations; remove duplicated status flags and unofficial state machines.
- [ ] P03-D Document candidate/evidence schemas, credit formula, queue eligibility, failure/refund examples, and bounded storage lifecycle.
- [ ] P03-R Review candidate immutability, artifact trust, accounting conservation, and all admission bypass paths.

Exit evidence: malformed work costs no expensive execution; credit ceilings hold under concurrency; candidate publication cannot claim trusted verification.

## P04 - Real isolated validation

Depends: P03. Slices: runner pilot; job/evidence protocol; hostile workload and accounting proof.

- [ ] P04-01 Choose and demonstrate an isolated VM or equivalent runner; measure cold start and a representative game build/check mix.
- [ ] P04-02 Implement authenticated job claim with fenced lease, nonce, allowlisted environment/suite, bounded artifacts, and cancellation.
- [ ] P04-03 Enforce CPU, memory, wall-time, process, disk, network, and output caps at the runner boundary; isolate dependency caches.
- [ ] P04-04 Keep candidate code away from DB, OAuth, control-plane and Git-writer secrets; narrowly scope artifact upload credentials.
- [ ] P04-05 Implement job states, heartbeat/expiry, crash handling, retry limits, dead-letter review, and actual measured settlement.
- [ ] P04-06 Verify runner attestation binding before granting trusted evidence; late/duplicate/replayed results cannot advance stale jobs.
- [ ] P04-07 Implement evidence cache keys and invalidation; untrusted self-reports remain visibly distinct from trusted results.
- [ ] P04-08 Build the runnable seed game and oracle suite specified in P00-10, pin its initial Git commit, and supply a reproducible web-game validation fixture with boot, movement, interaction, objective reachability, save/load, fatal-error, and performance checks.
- [ ] P04-09 Test fork/process bombs, excessive output, filesystem escape attempts, network exfiltration, disk exhaustion, malicious dependencies, and timeouts.
- [ ] P04-10 Measure estimates versus actual cost; confirm reservation and platform-rerun pools bound total spend even through worker failure.
- [ ] P04-C Consolidate runner/protocol error states, remove command-string shortcuts, trim unused execution backends, and simplify environment setup.
- [ ] P04-D Publish runner provisioning, isolation threat assumptions, job protocol, caps, reproducibility, check costs, and remaining platform limitations.
- [ ] P04-R Review actual isolation configuration and results; a fake runner or timeout-only subprocess fails this gate.

Exit evidence: real hostile-code isolation; reproducible game checks; measured, capped spending; no control-plane secrets in job execution.

## P05 - Adjudication and canonical integration

Depends: P04, including the runnable seed baseline from P00-10/P04-08. Slices: selection/ordered revisions; fenced Git promotion; crash/rollback recovery.

- [ ] P05-01 Implement preregistered work-item selection policy, qualifying L3 sequence, candidate-ID tie break, maintainer decisions/deadlines, and superseded history.
- [ ] P05-02 Create ordered queue entries and immutable integration revisions bound to source candidates, current parent, result tree, constitution, and policy version.
- [ ] P05-03 Construct integration revisions in bounded disposable Git workspaces; handle conflicts without rewriting published source identity.
- [ ] P05-04 Validate the exact resulting tree and enforce centrally computed risk/evidence gates; prevent all L1/L2/self-report promotion bypasses.
- [ ] P05-05 Enforce single active canonical authority via fencing, protected remote permissions, and compare-and-swap on expected parent.
- [ ] P05-06 Implement persisted promotion intents, remote ref update, DB finalization, and reconciliation for missing acknowledgment/partial completion.
- [ ] P05-07 Detect unexpected external HEAD movement and freeze promotion; record actor, decision, evidence, tree, parent, resulting commit, and build.
- [ ] P05-08 Implement audited rollback to known-good tree through a new commit; invalidate affected evidence/descendants and restart queue safely.
- [ ] P05-09 Publish canonical events and unblock dependencies transactionally after reconciled promotion; repair projections without replaying Git writes.
- [ ] P05-10 Test two authorities, parallel qualifying candidates, stale parents, constitution changes, conflicts, failing checks, all crash boundaries, and duplicate retries against real Git.
- [ ] P05-C Consolidate promotion/rollback/reconciliation transitions; remove direct-write shortcuts and duplicate winner-selection rules.
- [ ] P05-D Document adjudication, queue semantics, protected ref setup, promotion recovery table, rollback procedure, and evidence reuse rules.
- [ ] P05-R Review every path capable of touching canonical refs; compare recorded evidence/tree identities with actual Git objects.

Exit evidence: every promoted tree has exact bound evidence; one auditable winner; crash recovery neither duplicates nor loses successful promotion.

## P06 - First complete contributor and operator deployment

Depends: P05 and the pinned seed constitution/oracles. Slices: CLI/REST loop; live UI/stream; rehearsal and release gate.

- [ ] P06-01 Provide CLI for authentication, status, discovery, claims, renewals, candidate publish/submit/status, and event watch.
- [ ] P06-02 Provide resumable authorized WebSocket events with backpressure, pagination/catch-up, heartbeat, deduplication guidance, and retention-gap errors.
- [ ] P06-03 Build contributor UI for project health, work graph, contention, leases, candidate evidence, queue/budget explanations, and playable canonical artifact.
- [ ] P06-04 Build operator UI for moderation, appeals, leases, reservations, queued jobs, human decisions, freezes, and recovery status.
- [ ] P06-05 Ensure keyboard access, usable error states, loading/reconnect behavior, explicit stale data, and safe display of participant-controlled text.
- [ ] P06-06 Deploy a reproducible invite-only environment with real PostgreSQL, Git, object storage, isolated runner, scoped credentials, and monitored services.
- [ ] P06-07 Run two-human/multiple-agent loop: invitation -> discovery -> lease -> local work -> submission -> admission -> validation -> selection -> promotion -> stream -> dependency unblocking.
- [ ] P06-08 During the loop inject budget constraint, rejected candidate, mute, quarantine, lease revoke, ban, runner failure, stream reconnect, and rollback; verify readable outcomes.
- [ ] P06-09 Record service objectives from pilot measurements; narrow unsupported promises and identify event readiness gaps.
- [ ] P06-C Consolidate API/CLI/UI assumptions and status names; remove demo bypasses, dead endpoints, placeholder buttons, and repeated client logic.
- [ ] P06-D Publish quickstart, contributor walkthrough, operator guide, API examples, environment manifest, and known limits; a fresh operator follows them.
- [ ] P06-R Review end-to-end evidence in real deployment, browser, CLI, and repository; screenshots alone do not prove promotion safety.

Exit evidence: first complete deployment including moderation and restart recovery; no mocks at authority or isolation boundaries.

## P07 - Reliability and security qualification

Depends: P06. Slices: threat closure; recovery drills; operational hardening.

- [ ] P07-01 Revisit threat model with actual implementation; review auth/OAuth, authorization, signatures, artifacts, web injection, dependency risk, and log privacy.
- [ ] P07-02 Test credential rotation/revocation, compromised owner containment, shared organization freezes, role changes, and runner credentials expiring mid-job.
- [ ] P07-03 Restore database/object backup and reconcile Git refs; define tested RPO/RTO and audit what cannot be reconstructed.
- [ ] P07-04 Inject DB restart, dispatcher death, runner death, object outage, remote Git timeout, integrator crash, and network partitions.
- [ ] P07-05 Verify bounded retries/queues, read availability, event replay, job fencing, and emergency control availability under each failure.
- [ ] P07-06 Implement retention/deletion, artifact/log redaction, audit access controls, abandoned reservation cleanup, and alert routing.
- [ ] P07-07 Review repository protection/service permissions and practice admission/promotion freeze plus rollback without deployment.
- [ ] P07-08 Close critical/high findings; assign dated medium findings and capacity risks; repeat checks only for changed or unresolved behavior.
- [ ] P07-C Consolidate retry/backoff/cleanup policies and configuration; remove incident patches with conflicting semantics.
- [ ] P07-D Publish backup/restore, incident, degraded-state, credential rotation, privacy/retention, and appeal runbooks verified by rehearsal.
- [ ] P07-R Review all failure-drill evidence, not just nominal tests; release remains blocked on authority/budget/security/data-loss defects.

Exit evidence: practiced recovery, closed release blockers, and a bounded degraded mode that preserves authority and moderation.

## P08 - Complete collaboration and agent interfaces

Depends: P06; P07 for public use. Slices: contracts/dependencies; messaging/presence; indexing/director; MCP parity. Apply recurring checkpoint after first three slices.

- [ ] P08-01 Implement interface publication, accepted-for-work versus canonical authority, version compatibility, deprecation, and supersession.
- [ ] P08-02 Implement NEED/PROVIDES discovery and dependency coordination with explicit governance of graph/contract mutations.
- [ ] P08-03 Implement direct/cell/task/module/team/event/system messaging, membership checks, notification budgets, local blocks, and elevated event broadcasts.
- [ ] P08-04 Implement node capabilities/presence with TTL, coalescing, explicit stale status, and no automatic trust in advertised capacity.
- [ ] P08-05 Build derived path/module/symbol/test activity index from Git and events; bound parsing and reconstruct views from authoritative data.
- [ ] P08-06 Improve advisory discovery using contention, milestones, neglected areas, and downstream unblock value; expose explanations.
- [ ] P08-07 Implement Event Director milestone views and suggestions without granting it canonical authority.
- [ ] P08-08 Add MCP-compatible tools backed by the same application services, credential scopes, idempotency and budgets as REST/CLI.
- [ ] P08-09 Run cross-harness contract tests with two different agent clients; replay interrupted sessions and verify neither transport bypasses controls.
- [ ] P08-10 Test broadcast storms, local blocking, poisoned metadata, index rebuild, presence loss, and permission-filtered search.
- [ ] P08-C Remove duplicate state in indexes/UI, consolidate transport adapters, and retire unused presence/discovery scaffolding.
- [ ] P08-D Publish tool catalog, contracts, scope rules, stream consumption, cross-harness recipes, and index rebuild procedure.
- [ ] P08-R Review transport policy parity, private metadata exposure, notification amplification, and index trust boundaries.

Exit evidence: collaborating agents observe consistent state and governance across supported transports; director/index never becomes an alternative authority.

## P09 - Guarded distributed validation and throughput improvements

Depends: P07-P08. Slices: independent assignments; rewards/spot checks; speculative queue and batch recovery.

- [ ] P09-01 Implement opt-in node enrollment, assigned replication jobs, random/independent assignment, acceptance rules, and trusted spot checks.
- [ ] P09-02 Bind peer attestations to candidate/base/tree, environment, suite, nonce, signer, result, and owner; reject self/same-owner independence claims.
- [ ] P09-03 Implement capped reward ledger for assigned accepted reproductions; prohibit recursive reward minting, duplicate payment, and same-owner reward loops.
- [ ] P09-04 Model colluding owners, false attestations, unreliable nodes, and reward exhaustion; preserve risk-class trusted-runner requirements.
- [ ] P09-05 Implement look-ahead revisions against explicit queue parents, descendant invalidation, cancellation/refund semantics, and safe requeue.
- [ ] P09-06 Add compatible batches with exact-tree validation, failure splitting, offender isolation, and auditable candidate-to-promotion lineage.
- [ ] P09-07 Implement narrowly justified R0-R1 input-based evidence reuse; R2-R5 retain actual-target checks.
- [ ] P09-08 Compare serial baseline with speculation/batching: throughput, invalidations/promotion, extra runner cost, tail latency, and rollback recovery.
- [ ] P09-C Consolidate serial/speculative queue code, prune uneconomic heuristics, and remove reward machinery that lacks measured useful work.
- [ ] P09-D Document peer trust/rewards/caps, speculation invalidation, batch splitting, cache policies, and benchmark tradeoffs.
- [ ] P09-R Review independence/reward conservation and speculative dependency graphs; reject optimization that trades away canonical evidence.

Exit evidence: incentives cannot mint unlimited access; promotion guarantees survive batching/rollback; improvements have measured value.

## P10 - Capacity, abuse, and target-event rehearsal

Depends: P07-P09. Slices: harness/baseline; overload/abuse; full rehearsal and capacity gate.

- [ ] P10-01 Classify the §31 numeric workload as required target-event qualification targets, distinct from provisional policy/latency/soak choices; freeze representative workload, environments, budget, offered/admitted rates, latency/error objectives, and durable/ephemeral update split.
- [ ] P10-02 Build workload drivers for 500 owners, 5,000 agents, 2,000 nodes, 1,000 cells, 100 submissions/minute, and 50,000 updates/minute.
- [ ] P10-03 Run at least 60 minutes sustained target traffic with actual expensive/cheap check mix and four-hour recovery/soak profile.
- [ ] P10-04 Run budget-constrained mode, bursts, competing candidates, queue saturation, account multiplication, graph churn, spam, and conflict bombing.
- [ ] P10-05 Measure queue growth/age, central spend, estimate error, urgent capacity, lock pressure, event lag, reconnects, human backlog, and invalidations/promotion.
- [ ] P10-06 Verify degradation order: reduce presence/coalesce, slow graph writes, defer admission/validation, preserve canonical integrity/moderation/repository reads.
- [ ] P10-07 Test budget caps with costly/time-out candidates; prove failed work and platform reruns cannot exceed configured total capacity.
- [ ] P10-08 Practice event freeze/rollback/restoration while target load continues; measure recovery and reconcile all promotion and ledger records.
- [ ] P10-09 Tune only measured bottlenecks; document whether extra caches/workers/batches improve outcomes and rerun affected profiles.
- [ ] P10-10 Rehearse operators and moderators with a human-review budget; choose smaller event admission if measured capacity misses targets.
- [ ] P10-C Consolidate performance patches/configuration and remove obsolete workarounds; check that cache/partition changes preserve authority semantics.
- [ ] P10-D Publish reproducible benchmark report, sizing/cost model, backpressure policy, validated limits, and event go/no-go record.
- [ ] P10-R Review raw results against frozen objectives; distinguish advertised nodes from trusted runner capacity and offered submissions from completed validations.

Exit evidence: measured target workload fits budget and service policy; operators can preserve authority under overload. Failed target means narrow event size or corrective work, not a silent gate waiver.

## P11 - Release and event operations

Depends: P10. Slices: release packaging; dress rehearsal; post-event reconciliation.

- [ ] P11-01 Produce reproducible release tag, migration/config manifest, deployment and rollback instructions, dependency/SBOM record, and operator access list.
- [ ] P11-02 Verify required CI/reviews, protected refs, credential scopes, backups, caps, alerts, and incident ownership in the actual release environment.
- [ ] P11-03 Run dress rehearsal from invitation to final playable release, including appeal, outage, budget pause, and rollback.
- [ ] P11-04 Freeze the seed-derived event constitution/oracles/selection rules authored in P00-10 and implemented/pinned in P04-08, communicate capacity and onboarding policy, and publish participant documentation.
- [ ] P11-05 Run event with live budget/queue/build health, moderator coverage, incident log, and release/blocker decisions.
- [ ] P11-06 Reconcile all promotions, credit accounts/reservations, winning decisions, artifacts, and unresolved tasks after event closure.
- [ ] P11-07 Review participant/operator feedback and failure data; prioritize fixes by causal impact, not code volume or speculative future architecture.
- [ ] P11-C Dedicate post-event consolidation to temporary overrides, accumulated complexity, flaky checks, stale fixtures, and abandoned features.
- [ ] P11-D Update actual architecture, runbooks, API/client docs, measured capacity, change log, and next-phase backlog before new features resume.
- [ ] P11-R Complete release and post-event authority/security reviews; record remaining limits and blocker closure evidence.

Exit evidence: reproducible release and auditable event record; post-event cleanup is completed rather than deferred behind another feature wave.

## P12 - Evidence-led expansion

Depends: P11 and demonstrated need. Not a precondition for the first event.

- [ ] P12-01 Identify measured limiting resources before proposing horizontal decomposition, sharding, multi-region reads, or larger events.
- [ ] P12-02 Preserve single canonical authority and hierarchical budgets through topology changes; benchmark migration and rollback.
- [ ] P12-03 For each new domain define automated/hybrid/human-only oracle, inputs, pass threshold, reliability, check cost, and human review allocation.
- [ ] P12-04 Trial one new domain at bounded scale; do not inherit software-event throughput claims without validation economics measurements.
- [ ] P12-05 Revisit permissionless admission only with an explicit owner/escrow/ramp/appeal policy and abuse experiment.
- [ ] P12-C Retire superseded adapters and duplicated domain policy; resist genericization without two proven uses.
- [ ] P12-D Publish new ADRs, migration/rollback plans, domain oracle limits, and measured scale profile.
- [ ] P12-R Review revised threat/cost/authority model and evidence for every expanded claim.

Exit evidence: expansion improves a demonstrated outcome without weakening the original invariants.

## Recurring checkpoint RC-n (repeat every three feature PRs or weekly)

- [ ] RC-n-01 Inspect the accumulated main-branch change as one system; review authority, concurrency, failure paths, and state transitions across PR boundaries.
- [ ] RC-n-02 Triage debt ledger; close blockers, choose concrete cleanup, and assign owner/phase/deadline for justified deferrals.
- [ ] RC-n-03 Remove duplicate abstractions, copy-paste policy, unused dependencies/configuration, dead paths, misleading placeholders, and broad exception swallowing.
- [ ] RC-n-04 Review test value: preserve meaningful failure/concurrency checks; repair flaky checks; remove implementation-mirroring or obsolete tests where appropriate.
- [ ] RC-n-05 Reconcile README, architecture, ADRs, API/CLI schemas, runbooks, TODO states, examples, and supported setup commands against current main.
- [ ] RC-n-06 Run checks appropriate to cleanup; review the cleanup PR(s); record evidence and whether feature work may resume.

## Milestone records

- [ ] M1 P00-P03 reviewed: bounded coordination/admission foundation.
- [ ] M2 P04-P05 reviewed: real isolated verification and recoverable exact-tree promotion.
- [ ] M3 P06-P07 reviewed: complete governed deployment and operational recovery.
- [ ] M4 P08-P11 reviewed: complete collaboration, measured full-event readiness, release, and post-event consolidation.
- [ ] M5 P12 reviewed when needed: justified expansion with domain-specific evidence.
