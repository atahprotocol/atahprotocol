# 8. Claim authentication — minimum at v0.8, hardening deferred to v0.9

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

When a partner-created record is **claimed** by the professional, the
person claiming gets the ability to update self-managed fields on the
record (fees, contact details, response time, language proficiency,
specialisms). Those fields then influence matching.

This is an attractive target. An identity thief who can pass the claim
flow gains the ability to redirect introductions intended for the real
professional — or to set fees and availability that misrepresent the
professional in the market. The threat is concrete and widely
understood (cf. impersonation attacks against professional-directory
services).

The right protections — step-up authentication, anomaly detection on
claim attempts, notification through partner-known channels with an
objection window, rate limiting — are well-understood individually but
require operational data and partner integration to tune correctly.

The choice we faced: ship v0.8 with full hardening mandated; ship v0.8
with no minimum (let implementers decide); or ship v0.8 with a minimum
that closes the obvious doors and defer the full hardening to v0.9 with
explicit operational guidance for the reference registry.

## Decision

v0.8 specifies a **minimum** claim authentication requirement:

- **Verified email match against partner records** (the partner submitted
  the professional's email; the claimant must demonstrate control of
  that email by completing a verification flow), **or**
- **Equivalent partner-attested identifier** when email is not what the
  partner holds (e.g. an active-licence number plus a separate channel
  check the partner can confirm).

The fuller hardening — step-up authentication on first claim, anomaly
detection on claim attempts (geo, device, velocity), notification
through a partner-known channel with an objection window, and rate
limiting — is **listed in spec §15.6 as deferred to v0.9**.

The reference registry SHOULD apply the deferred hardening operationally
at launch even though it is not yet protocol-mandated for all conforming
implementations. This is a "MUST for the reference registry,
recommended for others, normative in v0.9" pattern.

## Alternatives considered

- **Mandate full hardening at v0.8.** Rejected. Premature: anomaly
  detection and rate-limit thresholds need real operational data to
  set defensibly. Mandating specific implementations now risks
  encoding bad defaults into the standard.
- **Specify nothing — let implementers decide.** Rejected. Leaves the
  door open to identity-theft-by-claim with no protocol-level floor.
  Implementers under cost pressure will reach for "send a magic link
  to the email on file" with no further controls.
- **Mandate the minimum AND specify the hardening as RECOMMENDED.**
  Considered. Functionally similar to what we did. We chose to mark
  hardening as DEFERRED rather than RECOMMENDED so that v0.9 can
  promote it to MUST without a deprecation cycle for "RECOMMENDED but
  not done it" implementations.

## Consequences

- Spec §15.6 is the normative reference for v0.8 behaviour.
- The CHANGELOG explicitly lists claim-authentication hardening under
  "Deferred to v0.9 / v1.0".
- SECURITY.md flags claim-authentication hardening as out of scope at
  v0.8.
- The reference registry's launch checklist includes the deferred
  hardening as a launch-day operational requirement.
- An identity-theft-by-claim disclosure during the v0.8 → v0.9 window
  would be handled through the security disclosure process and may
  accelerate v0.9.

## References

- Spec §15 (Professional Records and Registration), §15.6 (Claim
  Authentication)
- CHANGELOG.md "Deferred to v0.9 / v1.0" — claim authentication
  hardening
- SECURITY.md threat-model baseline (out-of-scope items)
