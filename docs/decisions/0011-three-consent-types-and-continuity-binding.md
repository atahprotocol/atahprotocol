# 11. Three consent types (engagement excluded by design) and continuity binding

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

This decision draws on Paolo Piponi's peer review (May 2026). Four weaknesses converged into a single coherent fix:

1. **Consent types were conflated.** Treating all consent uniformly hides three meaningfully distinct consent concepts: query authorisation (the user permits their AI to request candidate professionals), disclosure consent (the user permits specific data to be shared with a specific professional for a specific introduction stage), and engagement consent (the user agrees to a fee letter, instructs the professional to act, signs the engagement letter, gives treatment consent, waives a conflict). Conflating them legally is dangerous: a professional or regulator reading "ATAH consent" can interpret it as engagement consent, which would make ATAH appear to be granting the relationship rather than the handoff. This is the legally-most-important boundary the protocol crosses.

2. **The receipt issuer must be unambiguous.** Whoever issues the consent receipt determines who carries the legal accountability for the underlying consent ceremony. A protocol surface that is unclear on the issuer would let implementations diverge on a load-bearing detail (F1.5 in the consolidated peer-review findings).

3. **A submission path is required.** Without an explicit API surface to submit a captured consent receipt, the schema describes a shape with no workflow to produce instances of it (F1.3 in the consolidated peer-review findings).

4. **Continuity binding closes a class of replay attack.** Receipt replay (Platform X submits a receipt for user A, then attempts to consume it for an action concerning user B) and cross-user confusion need to be closed. A receipt's hash protects against post-submission tampering, but does not bind the receipt to the consumption context.

## Decision

v0.8.2 separates the consent landscape into three distinct concepts, with verbatim normative rules adopted from Paolo (per the v0.8.2 standing instruction on verbatim adoption):

**Three consent types, two supported, one excluded by design.** The `consent_type` enum on `consent-receipt.schema.json` is restricted to `query_authorization` and `disclosure_consent`. Engagement consent is documented in spec §4.10 as outside ATAH; the schema has no enum value for it. Spec §4.10 carries Paolo's MUST NOT rule verbatim:

> ATAH consent receipts MUST NOT be represented as professional engagement consent. Conforming implementations MUST disclose that any professional-client relationship arises only through the professional's own onboarding, engagement, regulatory, or contractual process.

The receipt-hash limitation framing is also verbatim:

> ATAH verifies integrity of the submitted consent receipt. The asserting AI platform remains responsible for obtaining valid user consent to disclose data.

**Submission via dedicated endpoint (Option B).** v0.8.2 adopts the dedicated-submission-endpoint pattern: `POST /v1/consent-receipts` accepts the full receipt body, returns `consent_receipt_id` and the stored-form metadata. Subsequent consenting endpoints reference by id only. Reasons: simple stable consenting-endpoint shapes; submission lifecycle separated from consenting-action lifecycle; supports later batch submission; the ID-by-reference pattern matches the Verifiable Credentials and Sigstore patterns the spec already cites; cleaner idempotency. The new MCP tool `submit_consent_receipt` mirrors the REST endpoint. Symmetric `POST /v1/consent-receipts/{consent_receipt_id}/revoke` covers query-scope and pre-handoff revocations.

**Issuer model resolved.** Consent receipts are issued by the asserting AI platform, submitted to ATAH, and stored as hash + continuity-binding metadata. The full receipt is retained by the asserting platform for dispute resolution. The asserting platform's `client_application.platform_id` MUST match the authenticated platform on submission — this enforces issuer identity at the API surface.

**Continuity binding (F-3).** The stored receipt (per `consent-receipt-stored.schema.json`) carries a non-identifying continuity binding: `client_id` (the asserting platform's client identifier, mirrored from the source receipt); `pseudonymous_consumer_ref` (HMAC of the platform-side consumer reference, under a per-context salted HMAC key per the F1.16 / §11.8 HMAC-not-plain-hash requirement; required for consumer-tied scopes); `data_categories_hash` (SHA-256 of the canonicalised data-categories array; required when `consent_type` is `disclosure_consent`). On consumption, ATAH verifies the consuming session's `client_id` and pseudonymous reference match the stored values, and that the requested action falls within the captured scope and data-categories hash. Mismatch produces a consent-mismatch error and a `consent_continuity_mismatch` audit event. The build-time privacy-floor test verifies these fields are HMAC-shaped or hash-shaped, not plaintext.

Charter Part Two operational commitments mirror the engagement-consent normative rule; spec §4.10 is the canonical source.

## Alternatives considered

**Option A — full-receipt-inline.** Every consenting endpoint accepts the full receipt body inline. Rejected per the start-of-phase decision (Option B selected). Reasons: Option A produces five different request shapes that all carry the same big receipt body; it conflates submission and consumption lifecycles; it complicates idempotency (one Idempotency-Key per consenting action vs one per receipt submission); the ID-by-reference pattern is already what VC and Sigstore use.

**Storing more receipt context for richer auditability.** Considered: store additional fields on the stored form (e.g. the asserting platform's request IP, the user-agent string at consent capture, the consent-text payload itself). Rejected on privacy grounds: ATAH stores hash and continuity-binding metadata only, with HMAC-protected references where audit needs them. The full receipt is retained by the asserting platform.

**Plain hash of `consumer_ref` for continuity binding.** Considered: a SHA-256 of the platform-side `consumer_ref` would be simpler than HMAC. Rejected: plain hashes of platform-side consumer references are guessable across the platform's user population and effectively reversible if the consumer-ref space is enumerable. Per F1.16 / §11.8, HMAC with a protected audit key is the standard mitigation.

**Inlining `engagement_consent` as a documented-but-unsupported enum value.** Considered: list `engagement_consent` in the enum with a doc note that it's not supported. Rejected: a schema enum with an unsupported value invites misuse. The schema's `consent_type` enum lists only the two supported values; the spec text explains why engagement consent is excluded.

## Consequences

- The submission-then-consumption pattern means every consenting flow is two API calls. Implementations must ensure the `consent_receipt_id` came from a prior submission, not from any other source. The OpenAPI documents this, and the audit log records both submission and consumption events with the same receipt id.
- The two-call pattern introduces a small race surface: a platform submits a receipt; the consumer revokes; the platform attempts to consume the now-revoked receipt. The `revocation_status: active` check on consumption is required at every consumption point in the OpenAPI documentation.
- The HMAC continuity binding requires an HMAC key-management lifecycle separate from application authentication keys (per §13.2 and §11.8). v0.8.2 specifies the requirement; key-management mechanics are an implementation choice and a candidate for v0.9 standardisation.
- The engagement-consent exclusion is the legally-most-important sentence in the protocol. It appears prominently in spec §1.5, §4.10, EXPLAINER, PRD §9, CHANGELOG, and Charter Part Two operational commitments — five surfaces, the same wording, so a state bar reading any one of them gets the same answer.
- The stored receipt's continuity-binding fields are non-identifying by construction (`client_id` is platform-opaque; `pseudonymous_consumer_ref` is HMAC-protected; `data_categories_hash` is over the canonical category list). The build-time privacy-floor test verifies the continuity-binding fields are HMAC-shaped or hash-shaped, not plaintext.
- The `audit-event.schema.json` `event_type` enum gains `consent_continuity_mismatch` so consumption-time verification failures are first-class audit events, not opaque rejections.

## References

- Spec §1.5 (three consent types overview), §4.10 (consent receipt object), §7.3 (authorisation matrix rows for the consent-receipt endpoints), §11.2 (deletion table consent-receipt store row).
- Schemas: [`consent-receipt.schema.json`](../../spec/v0.8/schemas/consent-receipt.schema.json) (`consent_type` enum, asserter framing); [`consent-receipt-stored.schema.json`](../../spec/v0.8/schemas/consent-receipt-stored.schema.json) (continuity binding fields).
- OpenAPI: `POST /v1/consent-receipts`; `POST /v1/consent-receipts/{consent_receipt_id}/revoke`; the §7.3 matrix expressions of these.
- MCP tools: `submit_consent_receipt`.
- Charter: Part Two operational commitments — engagement-consent MUST NOT rule mirrored from spec §4.10.
- Findings: F1.3 (three consent types), F1.4 (engagement-consent normative rule), F1.5 (receipt-hash limitation framing); F-3 (continuity binding).
