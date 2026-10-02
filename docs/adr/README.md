# Architecture decisions

Task P00-01 records the initial design choices. These decisions become the implementation baseline when their PR merges; they are not deployed behavior or tested provider compatibility. Later phases carry the concrete verification work. A new decision supersedes an old ADR with links and migration/reversal evidence; do not rewrite accepted history silently.

| ADR | Choice | Verification / revisit gate |
| --- | --- | --- |
| [0001](0001-control-plane-runtime.md) | Python 3.12 compatibility baseline and FastAPI adapters | P01 install/pins; P10 profiling; interpreter support |
| [0002](0002-postgresql-authority.md) | PostgreSQL authority, locked budget/event mutations | P01/P03 concurrency; P07 restore; P10 contention |
| [0003](0003-module-boundaries.md) | Modular codebase with separately privileged processes | P04/P07 boundaries; P08 transport parity |
| [0004](0004-repository-authority.md) | Exact Git identity and recoverable restricted remote writer | P05 real CAS/fencing/crash drills |
| [0005](0005-trusted-execution.md) | External disposable VM or proven equivalent | P04 provider pilot, isolation and cost caps |
| [0006](0006-object-storage.md) | S3-compatible artifacts with verified immutable completion | P03 provider behavior; P07 restore/retention |
| [0007](0007-identity-admission.md) | Account authentication plus sponsor/owner admission | P01 stable identity; P02/P07 abuse and revocation |
| [0008](0008-container-optional-development.md) | Tested primary setup without required Docker | P01/P06 clean setup; P04 independent VM qualification |

Each ADR supplies context, decision, alternatives/consequences, verification/reversal conditions, and sources. Provider names/examples are not provisioning decisions. Source inputs are preserved in [the source manifest](../sources/README.md). Standards documentation supports technical facts; architecture selections and acceptance criteria are project decisions.

## P00-01 review status

Implementation: Codex authored the ADR set and checked internal consistency, source links and local references. Review: a separate author review checks all eight decisions against the merged plan and its authority/budget/recovery invariants; external PR review remains pending. No second reviewer or runtime experiment is claimed. P00 phase C/D/R gates remain open.

Acceptance evidence: eight ADRs cover every P00-01 topic, record alternatives and reversal triggers, assign unresolved package/provider/version compatibility to specific later tasks, and preserve the early-skeleton versus production distinction. Documentation-only work does not advance the feature PR counter. P00-12 will initialize its durable home; no cadence/debt/checkpoint infrastructure is claimed complete here.
