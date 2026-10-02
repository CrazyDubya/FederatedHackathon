# ADR 0004: Git identity and recoverable canonical writes

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

Source candidate commits must remain immutable while integrations target a moving parent. Git and database changes cannot share an atomic transaction. A GitHub API call without expected-parent semantics is insufficient to prove promotion safety.

## Decision

Use Git objects and refs as source-history authority. Use real local bare repositories for development/protocol tests and a protected GitHub-compatible remote for production. This platform's own repository and an event's game repository are distinct roles; contributors do not receive canonical write permission merely by joining an event.

Define repository operations narrowly: resolve/fetch objects, construct integration revision, read canonical parent, compare-and-swap ref, and reconcile result. Local CAS uses expected-old-object semantics. The remote adapter must demonstrate equivalent server-side behavior; never assume an ordinary ref-update HTTP endpoint provides CAS. Pin and verify the tested commit/tree after merge/rebase, before promotion.

Persist promotion intent/evidence/policy before a remote write and reconcile remote success before finalizing state/outbox. Only the dedicated canonical writer holds remote write credentials. Fence writer epochs and stop failover while prior writes are unresolved; CAS alone does not fence a stale worker that can issue new writes. Unexpected ref movement freezes integration. P05 must prove both fencing and remote adapter semantics, not substitute a database lease for remote enforcement.

Rollback creates an audited forward commit restoring a known-good tree; no force-reset shortcut.

## Alternatives and consequences

Normal PR merge buttons abstract tree construction but cannot replace exact-tree validation. A self-hosted Git service gives more control but adds operations. GitHub is initial compatibility, not permanent vendor authority. Reconciliation and restricted writer access add work but make lost acknowledgments recoverable.

## Verification and reversal

P05 tests real Git CAS, competing writers, crash after remote success, lost response, rollback and external movement. P07 verifies permissions and failure drills. If the chosen remote cannot meet tested CAS/fencing requirements, replace the remote adapter or use a controlled Git service before release; never weaken exact-tree evidence.

## References

- [Git update-ref](https://git-scm.com/docs/git-update-ref) (checked 2026-09-30).
- Spec §§9, 12-13, 30; review 02; P05-05/P05-06/P05-10.
