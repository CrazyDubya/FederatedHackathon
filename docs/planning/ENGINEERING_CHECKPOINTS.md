# Engineering review, consolidation, and documentation checkpoints

## Cadence and responsibility

| Trigger | Required checkpoint | Gate owner | Record |
| --- | --- | --- | --- |
| Every PR | Scope, code, invariant, test, debt, documentation review | Named reviewer | PR checklist and linked evidence |
| Every 3 merged feature PRs or weekly during active development | Integrated main review and consolidation | Maintainer | RC-n record and cleanup PRs |
| Every phase exit | Phase-R, Phase-C, Phase-D | Phase reviewer and maintainer | Phase gate record |
| Every milestone/release | End-to-end, security, operations, capacity as applicable | Release owner/operator | Go/no-go record |
| After incident or material authority change | Focused re-review of affected boundaries | Incident owner and reviewer | Incident follow-up |
| After event | Ledger/promotion reconciliation and consolidation | Event owner | Post-event report and closed cleanup tasks |

Assign real people or explicitly identified reviewing tools at task activation; names are not invented by this plan. If automated review assists, record its scope and findings and who adjudicated them. No requirement to spawn agents. Maintainers review their combined system, not only isolated diffs. High-impact auth, credit, runner, migration, and promotion work requires a second reviewer where available; if unavailable, record the limitation and require a separate review pass before release.

Count feature PRs since the last RC record. Cleanup-only and documentation-only PRs do not increment the feature counter. At a cadence trigger, pause additional feature merges, perform the integrated review, complete required corrective cleanup, update docs, and explicitly reset the counter. Emergency incident fixes may proceed with an incident record and a mandatory follow-up checkpoint before normal feature work resumes.

## Definition of ready

A slice is ready when its desired behavior, dependencies, acceptance evidence, authority boundary, owner/reviewer, and documentation target are explicit. Interface changes have contract examples. Data changes have migration/recovery plans. Execution changes have budget/isolation plans. Unknown critical assumptions get a bounded experiment, not optimistic implementation.

## Per-PR review rubric

1. **Outcome and scope:** One concrete concern; no opportunistic refactor concealed inside a feature. Before/after behavior is understandable without conversation history.
2. **Authority:** All mutations pass auth, scope, governance, and audit; transports cannot bypass policy; edge evidence does not become trusted merely by parsing it.
3. **Concurrency:** Identify locks, uniqueness, fencing, idempotency, visibility order, and retry effects. Use real DB/Git tests where those systems provide the guarantee.
4. **Failure behavior:** Crash, cancellation, timeout, stale policy, revoked credentials, unavailable dependencies, and partial completion have explicit outcomes.
5. **Budgets:** Reservations/settlements conserve capacity; hierarchical ceilings survive credential multiplication; retries and output cannot grow without bound.
6. **Code quality:** Prefer concrete, small responsibilities. Remove unexplained indirection, generic frameworks, duplicate state machines, dead code, hidden global state, catch-all success fallbacks, and misleading placeholders.
7. **Test quality:** Tests prove externally meaningful behavior and failure cases. Avoid asserting private implementation structure or writing tests solely to pad counts. Flaky required checks block release until repaired or replaced with a justified check.
8. **Security/privacy:** Participant text/code/artifacts are untrusted; secrets and private abuse linkage remain out of public responses/logs.
9. **Documentation:** Contracts, examples, runbooks, architecture, configuration and changelog reflect the change. State explicitly when none need changing and why.
10. **Debt:** Any deliberate shortcut has impact, owner, expiry/phase, and removal trigger. Authority, security, budget and data-loss shortcuts cannot be deferred across their required gate.

## Deslopping/consolidation rubric

Consolidation is a behavior-preserving simplification backed by evidence. It is not beautification by volume or a periodic rewrite.

Look for:

- Policy repeated in API, MCP, CLI, worker, and UI rather than enforced once in application services.
- The same entity/status stored independently in authoritative tables, projections, caches, and client state without reconciliation rules.
- Overgeneralized base classes, adapter registries, convenience wrappers, and configuration options with no concrete second use.
- Code generated around imagined future needs; unused schemas, endpoints, imports, dependencies, branches, and feature flags.
- Broad exceptions that convert failed verification/accounting into success; speculative retry loops with no caps.
- Giant modules that combine role checks, state mutations, execution, and remote Git writes without clear boundaries.
- Test suites that mirror helper implementations, depend on time sleeps, or pass only through unrealistic mocks.
- Comments/docs describing aspirations as shipped behavior; stale ADRs, commands, diagrams, and TODO statuses.
- Dependency/migration drift, duplicated validation rules, and setup steps that cannot be reproduced.

Do not change public behavior under a cleanup label. If simplification changes semantics, declare the change and apply feature-level review. Do not introduce a dependency merely to reduce a small amount of straightforward code. Refactor only areas with observed complexity, defects, or a near-term concrete need.

## Debt ledger

Create docs/engineering/DEBT.md during P00. Each entry records ID, discovery checkpoint, affected module, concrete defect/shortcut, user or operational impact, evidence, severity, owner, resolution phase/deadline, removal trigger, and linked PR. Closing requires evidence of removal or a documented decision that the item is not debt.

| Class | Treatment |
| --- | --- |
| Authority/security/budget/data-loss blocker | Fix before dependent phase or release; no budget waiver substitutes for correctness |
| Reliability or current capacity impairment | Resolve before the applicable rehearsal/gate |
| Duplicated code or architecture drift | Address at next recurring checkpoint unless evidence supports a specific deferral |
| Speculative future optimization | Track as backlog hypothesis, not debt that forces premature work |

Reserve a consolidation delivery slot after each three feature PRs. Actual effort follows findings; a reviewed no-change record is valid. Repeated deferral requires renewed evidence and explicit maintainer ownership, not silent checkbox carryover.

## Documentation contract

Keep one authoritative location for each fact and link from other pages:

- README: shipped capabilities, setup entry points, and current limitations.
- Architecture/domain docs: boundaries, authority, transactions, states, and recovery behavior.
- ADRs: chosen decisions, alternatives, evidence, and reversal conditions.
- API/OpenAPI, CLI and MCP catalog: current commands/contracts, scopes, errors, idempotency, versioning.
- Operator runbooks: deployment, migration, freeze, restore, rollback, credential rotation, incident, and appeals.
- Contributor guide: onboarding, leases, candidate/evidence publication, budgets, selection policy, and troubleshooting.
- Benchmark reports: actual workload, hardware, configuration, costs, measurements, and unsupported limits.
- Planning TODO and debt ledger: current delivery state and evidence links.

At each phase-D gate, verify links, examples, schema names, supported command paths, configuration defaults, and at least the changed setup/workflow. At release, a fresh operator executes the documented environment setup and a contributor completes the documented loop. Never document fake adapters or placeholder UI as production capabilities.

## Checkpoint record template

Create docs/engineering/checkpoints/RC-NNN.md or a phase gate record when work begins:

```markdown
# Checkpoint <ID>
Date:
Reviewed commit / PR range:
Implementer(s):
Reviewer / gate owner:
Trigger: PR / cadence / phase / release / incident
Outcomes and invariants reviewed:
Checks run and evidence links:
Failure/concurrency scenarios exercised:
Findings by severity:
Debt closed / added / explicitly deferred:
Cleanup PRs and behavior impact:
Documentation updated and commands verified:
Unresolved risks and owners:
Decision: PASS / BLOCKED / NOT APPLICABLE (with reason)
Next allowed phase/slice:
```

A phase passes only after R/C/D all pass. Records refer to an exact commit or reviewed range, so subsequent changes do not inherit a stale approval. Evidence can be concise logs, reproducible command results, scenarios, or benchmark artifacts; do not dump sensitive logs.

## Release gate

Release requires the applicable phase records; real isolated validation; exact-tree Git evidence; promotion/rollback crash drills; bounded owner/org/event credits; moderation availability; restore rehearsal; required checks; current contributor/operator documentation; and measured capacity appropriate to the announced event size.

Do not use a passing test count as a substitute for those claims. When a gate fails, specify which guarantee failed, corrective task, owner, and evidence needed to resume. Narrowing the event size may resolve capacity limits; it does not excuse unsafe authority or unbounded execution.
