# ADR 0002: One PostgreSQL authority

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

Leases, hierarchical reservations, candidate transitions and resumable events require concurrent mutations with explicit recovery. Maintaining SQLite and PostgreSQL authority paths would duplicate the hardest semantics.

## Decision

Use PostgreSQL for authoritative control-plane state from the first real loop. Choose a supported stable server release and pin its major version in P01; do not select beta software merely because it is newer. Use constraints and transactions for invariants, bounded connection pools, migrations, and explicit row-lock ordering. Read Committed plus locked invariant rows is the initial approach; any transaction requiring stronger isolation must declare it and handle bounded whole-transaction retries.

Allocate each project's durable event number by updating a project counter row in the same transaction as state/event/outbox. Hold that row lock through commit; standard database sequences alone do not establish commit-visible order. Lock owner/org/event credit rows in canonical order before reserving. Do not make a universal lock order implicit: P01 records the order across credit/project/entity locks and tests contention/deadlocks.

Derived indexes, presence, and edge caches are disposable. A skeleton's temporary store has no production authority. Git remains source-history authority; PostgreSQL does not replace it.

## Alternatives and consequences

SQLite is useful locally but would introduce a second authority implementation and weaker multi-worker portability. Redis/event-bus authority creates cross-store invariants before need. PostgreSQL row locking creates contention, especially per-project event ordering; measure that cost rather than weakening visibility guarantees. Presence stays outside the durable mutation sequence.

## Verification and reversal

P01/P03 prove no event inversion, lost update, double settlement, or concurrent overspend on real PostgreSQL. P07 restores backups; P10 measures locks/pools. Revisit allocation/partition strategy if measured contention misses objectives. Change database only with equivalent invariant/recovery tests and a migration/rollback plan.

## References

- [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html) and [locking](https://www.postgresql.org/docs/current/explicit-locking.html) (checked 2026-09-30).
- Spec §§29-30; review 01; P01-07/P01-09/P03-07/P03-11.
