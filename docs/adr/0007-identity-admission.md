# ADR 0007: Account authentication plus invite-based owner admission

Date: 2026-09-30 | Task: P00-01 | Status: proposed for merge; accepted as design baseline when this PR merges

## Context

The review requires owner-level budget inheritance and resistance to account multiplication. OAuth account ownership cannot establish a unique human; automatic new-account grants would create a budget reset path.

## Decision

Use an allowlisted OAuth/OIDC provider to authenticate account control and sponsor-issued event invitations to establish admission. P01 chooses/configures the provider; follow its supported authorization-code flow and applicable OAuth security guidance, including PKCE and CSRF/session protections. OIDC identities use issuer plus subject; plain OAuth providers use their documented stable account identifier, not invented OIDC claims. Do not use mutable email/display name as identity authority or silently link accounts by email.

Map authenticated accounts to an admitted internal owner through explicit approved linkage. Each invitation ties to a sponsor/admission record and a bounded starter allocation. Linking multiple accounts to an owner preserves consumed capacity; org membership and subordinate credentials never create fresh pools. Admission linkage and abuse decisions are private and appealable. IP address is not owner proof. Invitation control reduces abuse opportunities but does not mathematically eliminate collusion/Sybil identities.

Issue event-scoped subordinate credentials with expiry, hashed secret storage, individual revocation and least privilege. Owner/org permissions are checked on every shared mutation; OAuth tokens are not agent credentials. P00-02/P00-05 specify role/ramp/linkage policy; P01 implements; P07 tests compromise/rotation.

## Alternatives and consequences

Permissionless OAuth signup reduces onboarding friction but does not meet owner-budget guarantees alone. Mandatory universal identity verification adds privacy/operational burden not justified for an invited first event. Password hosting adds another security responsibility. Invitations require moderator/sponsor capacity and clear appeals.

## Verification and reversal

Test duplicate account grants, credential multiplication, explicit linking, revoked membership, and compromised owners before release. Consider permissionless admission only in P12-05 with measured abuse economics and privacy/appeal policy. Revisit providers if stable identity, supported security flow or operational availability cannot be verified.

## References

- [OIDC Core](https://openid.net/specs/openid-connect-core-1_0.html) and [OAuth security BCP, RFC 9700](https://www.rfc-editor.org/rfc/rfc9700) (checked 2026-09-30).
- Spec §§5, 14-16; review 03; P00-02/P01-03/P02-10.
