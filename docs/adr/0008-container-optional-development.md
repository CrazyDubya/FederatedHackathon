# ADR 0008: Reproducible development without required Docker

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

Contributor edge environments are deliberately heterogeneous. The project needs reproducible setup without turning a container runtime into an entry requirement or confusing environment packaging with hostile-code isolation.

## Decision

Document a primary path using Python venv, installed Git and an explicit local or remote PostgreSQL service. CLI/client participation requires only its supported runtime and credentials; participants do not need the control plane or trusted runner locally. Pin supported versions and provide actual install/start/check commands in P01/P06. Production process supervision and service configuration must be reproducible without Docker.

Provide the filesystem object adapter and local bare Git tests for development. Fake runner/temporary state adapters remain explicitly development-only and disabled for production readiness. The trusted VM service is external; lack of local virtualization must not prevent ordinary contributor or API development. Tests requiring PostgreSQL or the real isolated runner state their prerequisites and cannot silently pass through replacement mocks.

Optional containers may be documented later as conveniences, but they are not the sole tested installation path and cannot imply security equivalence to a VM runner. Preserve one authoritative production DB path instead of adding SQLite merely to make setup easier.

## Alternatives and consequences

Docker-only onboarding simplifies some packaging but violates the chosen workflow and excludes otherwise capable contributors. Running every component on every laptop creates needless privilege/setup burdens. An entirely mocked developer environment hides integration mistakes; real PostgreSQL/Git tests remain necessary. This decision creates maintenance responsibility for non-container instructions and supported host environments.

## Verification and reversal

P01 verifies installation/migrations and P06 rehearses a clean documented setup. P04 separately qualifies runner isolation. Revisit process packaging only with concrete support cost/host evidence and an explicit ADR amendment; do not remove the non-Docker path incidentally through a dependency or script change.

## References

Spec §§4, 24, 33-34; plan §3; P01-01/P04-01/P06-06/P06-D. The non-Docker baseline is a project workflow choice, not a spec requirement.
