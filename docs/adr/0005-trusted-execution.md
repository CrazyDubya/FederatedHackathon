# ADR 0005: External disposable VM validation

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

Participant code is hostile, and timeout-only subprocesses do not isolate credentials, files, network or resource exhaustion. The first deployable loop requires a real isolated runner; fake evidence belongs only to P00-11.

## Decision

Require an external disposable VM or equivalent independently verified isolation boundary for trusted jobs, outside API/integrator/database hosts. Prefer a managed ephemeral VM interface for the first deployment if it meets security/cost requirements. P04-01 chooses the concrete implementation by pilot; no provider, VM engine, operating system image, or benchmark result is claimed here. Firecracker/KVM is a candidate for a controlled Linux host, not a requirement for contributors or laptops.

Jobs identify immutable candidate/tree, suite, environment and nonce. Enforce CPU/memory/time/process/disk/output/network limits externally; default-deny network, with explicit allowlisted dependency access where required. Use disposable storage and separately isolated dependency caches. VM teardown must complete on cancellation/failure; measured usage settles reserved credits. Reject jobs when required isolation/caps or funded reservation cannot be established.

No control-plane, database, OAuth or Git-writer credentials enter guests. Artifact channels are scoped, expiring and bounded. The job controller authenticates results but untrusted guest assertions cannot alone prove trusted usage or successful isolation. P04 specifies host-side measurement and result binding.

## Alternatives and consequences

A local shell with resource limits is useful for benign tests, not trusted hostile execution. Hardened containers might satisfy an equivalent threat model after evidence, but introduce no Docker dependency. Self-hosted microVMs offer control with host patching/virtualization operations; managed VMs trade some visibility for simpler operation. Isolation does not solve admission economics.

## Verification and reversal

P04 runs the adversarial suite and representative build cost pilot; P07 reviews containment and teardown failures. Replace the provider/engine if required caps, network policy, clean teardown, environment reproducibility or reservation-bounded costs cannot be verified. Do not downgrade trust to clear a queue.

## References

- [Firecracker production host guidance](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md) (checked 2026-09-30; candidate implementation only).
- Spec §§11, 22-24; review 01; P04-01/P04-03/P04-09.
