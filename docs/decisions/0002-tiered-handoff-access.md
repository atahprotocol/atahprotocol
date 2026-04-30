# 2. Tiered handoff access — `handoff_id` is not a bearer token

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

A handoff is a record of an in-flight introduction between an AI platform's
consumer and a professional. It has a `handoff_id` and goes through a state
machine including consent capture, pre-handoff checks, and contact release.

Earlier drafts treated `handoff_id` as a bearer reference: knowing the id
was enough to read or modify the handoff. This is convenient — every actor
who needs to interact has the id from the initial response — but it puts
the entire state of the introduction one log line away from compromise.

`handoff_id` will leak. It travels through audit logs, error reports, AI
platform telemetry, professional CRMs, and conversation transcripts. We
have to assume any leaked id is in adversary hands.

We considered three options:

1. Treat `handoff_id` as a bearer token (the early-draft default).
2. Require strict access tokens for **all** operations, including read.
3. A tiered model: read-like operations available with weaker
   authentication, write-like operations require an unforgeable token.

## Decision

We adopted (3). The model is:

- **Read tier.** Available to (a) the original asserting platform/client
  pair, or (b) an authenticated AI platform presenting a `consumer_ref`
  matching the handoff record. Returns state and history; does not return
  personal data.
- **Write tier.** Requires `handoff_access_token` — opaque, short-lived,
  bound to platform/client/scope/expiry. Returned at handoff creation and
  required on every state-changing call (`stage-2`, `stage-3`, `cancel`,
  `revoke-consent`, `outcome`).

`handoff_id` alone never grants write access. Knowing the id is necessary
but not sufficient.

## Alternatives considered

- **`handoff_id` as bearer.** Rejected. Logs leak, and there is no way to
  rotate a leaked id without invalidating the introduction the consumer is
  trying to complete. The asymmetry between casual exposure paths and the
  damage of write access is too large.
- **Strict access tokens for all operations.** Rejected. The cross-platform
  use case — a consumer who started in Platform A asks Platform B "what's
  the status of my introduction?" — is core. If Platform B has to acquire
  a token from Platform A to read the state, the cross-platform UX
  collapses. Read-tier with consumer-reference matching preserves that UX
  without trusting `handoff_id` alone.

## Consequences

- The `handoff_access_token` is generated server-side at
  `POST /v1/introductions` and returned alongside `handoff_id`.
- Every state-changing endpoint requires the token in a header
  (`X-Handoff-Access-Token`).
- The token is bound to platform/client and is not portable across
  platforms — preventing a leaked token from being used by an attacker
  acting from a different platform identity.
- Read access is wider than write access by design. Audit log emits
  separate event types for read-tier vs write-tier access.

## References

- Spec §11.5 (Tiered handoff access)
- OpenAPI: `X-Handoff-Access-Token` parameter; `responses` 401/403 on
  state-changing introduction endpoints
- MCP: `handoff_access_token` is a required input on
  `submit_stage2_data`, `submit_stage3_data`, `cancel_introduction`, and
  `report_introduction_outcome`
