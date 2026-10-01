# ADR 0006: Immutable artifacts behind a narrow object adapter

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

Logs, replays, screenshots and builds have different retention and size profiles from transactional metadata. Upload success must not imply complete, trusted evidence. Participant-chosen object paths are unsafe.

## Decision

Use an S3-compatible production object interface with a filesystem development adapter. PostgreSQL owns artifact metadata, authorization, expected digest/size, completion and lifecycle; object bytes are not a second workflow authority. Choose provider and tested capabilities in P03-05/P06, not in this documentation task.

Use server-generated scoped keys. Upload to staging, verify full bytes/checksum/size, then publish immutable identity. Deny overwrite of completed artifacts through platform policy and tested provider behavior; keys/digests alone do not enforce immutability. Avoid exposing private moderation logs or credentials in public canonical builds. Bound per-artifact and event storage, logs and download/egress pressure.

Presigned URLs are bearer access and may be reusable until expiry, not inherently single-use. Their scope/expiry do not alone enforce every upload limit. Use signed upload conditions where supported or a bounded ingestion gateway, then independently verify before completion. Isolate artifact origins and safe content disposition from the operator UI. Clean aborted/multipart uploads and reconcile DB/object orphans; backups document both metadata and bytes.

## Alternatives and consequences

Database blobs simplify transactions but impose backup/query pressure. Git is appropriate for source, not high-volume disposable evidence. A production local filesystem needs its own durability/concurrency/backup plan. S3 compatibility is not evidence that a provider implements every checksum/lifecycle/access feature identically.

## Verification and reversal

P03 tests tamper/overwrite/oversize/partial uploads and expiry; P07 tests object outage, retention and restore; P10 measures storage/egress. Change provider if necessary features or cost limits fail. Preserve checksummed identities and metadata when migrating.

## References

- [S3 presigned URL behavior](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html) (checked 2026-09-30; verify chosen provider separately).
- Spec §§24-25, 29; P03-05/P07-03/P07-06.
