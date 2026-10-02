# ADR 0001: Python control plane

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

The center coordinates shared mutations and I/O; it must not execute participant builds or inference. The team needs a small deployable codebase with CLI/REST/MCP semantics that agree. Spec §33 permits Python/FastAPI; the implementation plan proposes Python 3.12.

## Decision

Use Python 3.12 as the initial compatibility baseline and FastAPI for HTTP/WebSocket adapters. Use a venv, explicit dependency pins, and separate worker processes for durable jobs. Database access uses an async-capable PostgreSQL driver; ORM and migration package selection/pins belong to P01-01/P01-02 after compatibility checks. Blocking Git operations and CPU-heavy parsing never execute in an API event loop. Participant code runs only in the external runner.

Application services own policy and transactions. HTTP, CLI and MCP are callers, not independent implementations. Workers acquire durable jobs rather than relying on in-process background tasks to survive restart.

## Alternatives and consequences

Go/Rust are valid alternatives but add an initial implementation/tooling shift without measured need. A sync-only API simplifies some operations but would require a deliberate bounded blocking execution model. Python's ergonomics do not establish capacity; multiple processes and async I/O are execution tools, not throughput claims. Python 3.12's maintenance state requires patched distributions and a planned version upgrade; do not freeze it indefinitely.

## Verification and reversal

P01 proves pinned dependencies install and migrate on the supported interpreter. P06 verifies restart behavior; P10 measures latency/event-loop responsiveness under actual load. Revisit the interpreter if dependencies drop support or patch availability is unsuitable. Consider a language change only after profiling identifies a bottleneck that bounded workers/index changes cannot address; preserve protocol and authority tests during migration.

## References

- [FastAPI concurrency](https://fastapi.tiangolo.com/async/) and [Python version status](https://devguide.python.org/versions/) (checked 2026-09-30).
- Spec §33; plan §§3-4; P01-01/P01-02/P06/P10.
