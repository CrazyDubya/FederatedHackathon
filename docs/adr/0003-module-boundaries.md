# ADR 0003: Modular monolith and restricted workers

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

One policy path is needed across multiple transports. Validation and canonical writing require different privileges from ordinary coordination. Premature service decomposition would make invariant changes span more systems.

## Decision

Keep one repository and a modular control-plane codebase: identity, projects, work, governor, candidates, validation, integration, events, collaboration and operations. Use concrete application services with module-owned mutations; adapters do not mutate another module's tables directly. Cross-module workflows commit authoritative state and events together where possible.

Deploy the API, outbox/job workers and canonical writer as separate processes/roles as required, without turning each module into a service. The API cannot execute participant code or write protected Git refs. The external runner cannot access the authoritative database or writer credentials. The writer accepts only validated integration identities and cannot be called as a general shell service. Integration construction that parses hostile source metadata must also use bounded disposable workspaces.

Use narrow explicit interfaces where an external boundary actually exists: repository, objects, runner, identity provider. No generic plugin registry, arbitrary agent hosting, or invented universal persistence framework.

## Alternatives and consequences

Microservices provide independent scaling but add network failure and distributed mutation costs before measurement. One privileged all-purpose process is easier initially but collapses trust boundaries. The chosen design keeps cross-module changes reviewable while still requiring deployment permissions and tests; Python module organization alone is not a security boundary.

## Verification and reversal

P01/P06 verify shared service authorization and transport parity. P04/P07 prove privilege separation. Each checkpoint removes duplicate policy and state. Split a service only for measured capacity, separate lifecycle or a demonstrated security requirement; preserve event/idempotency contracts and document the new recovery boundary.

## References

Spec §§3, 20, 24, 29, 34; plan §4; P01-06/P04/P05/P08-08. This topology is an engineering choice, not a source mandate.
