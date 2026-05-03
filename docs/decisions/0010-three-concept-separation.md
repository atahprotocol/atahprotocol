# 10. Three-concept separation: payload erasure, audit retention, withdrawal-as-state-transition

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

This decision draws on Paolo Piponi's peer review (May 2026). v0.8.1's §11 conflated three different operations on data:

- **Payload erasure** — the actual crypto-erasure of sensitive content from the transient vault.
- **Audit retention** — the tamper-evident record of what happened, used to investigate abuse, disputes, and consent-integrity claims.
- **Withdrawal** — the user-facing notion of stopping future processing.

Conflating them produced two real failure modes the review identified:

1. **Crypto-erasure was loosely worded.** The vault model relies on cryptographic erasure: destroy the key and the payload becomes practically unrecoverable. This is a valid engineering pattern, but it is not literal deletion of every byte. Ciphertext may remain in storage, backups, queues, or replicas until ordinary retention expiry. v0.8.1 did not say this precisely, and it did not specify the implementation requirements (unique keys per object, keys stored separately from ciphertext, no plaintext in logs/queues/analytics, auditable key-destruction events, short single-use retrieval tokens) that make crypto-erasure work as advertised.
2. **Withdrawal could become an evidence-deletion abuse primitive.** If withdrawal also removed audit metadata, ATAH would become harder to investigate when abuse, spam, phishing, bad consent assertions, weak authentication, or disputes occur. A professional facing concern flags could withdraw their record and simultaneously erase the lifecycle history that justified the flags. A consumer claiming "this never happened" could withdraw and remove the protocol's own evidence. This is a real abuse vector.

Separately, v0.8.1's audit log fields were under-specified: which fields are payload (must be erased) and which fields are metadata (must be retained) was not enumerated. References to contact identifiers (email addresses, phone numbers) defaulted to plain hashes — which are guessable across the population they identify and effectively reversible at scale.

## Decision

v0.8.2 separates the three concepts deliberately, with verbatim normative wording adopted from Paolo's review (per the v0.8.2 standing instruction on verbatim adoption of Paolo's MUST / MUST NOT rules):

**Payload erasure (§11.6).** Crypto-erasure is defined verbatim:

> Vault payloads are crypto-erased by destroying the data-encryption key. Encrypted ciphertext may remain in storage or backups until ordinary retention expiry, but is treated as unrecoverable once the corresponding key material has been destroyed.

Six implementation requirements are normative: unique key per vault object, keys stored separately from ciphertext, destroyed keys excluded from recoverable backups, no plaintext in logs/queues/analytics/indexes/notification providers, auditable key-destruction events, short retrieval-token TTLs and single-use retrieval tokens.

**Audit retention (§11.8).** A separate normative rule applies:

> Crypto-erasure applies to vault payload content, not to audit metadata. Conforming implementations MUST retain tamper-evident audit records sufficient to investigate abuse, disputes, authentication failures, and consent-integrity claims, without retaining consumer personal data in plaintext.

The erase-fields list (contact details, Stage 2 conflict/scope data, sensitive matter summaries, free text, retrieval tokens, exact notification links) and the retain-fields list (event ID, handoff/query/professional IDs, asserting platform and authenticated client, pseudonymous consumer reference, request_intent, stage, payload type, consent receipt ID and receipt hash, timestamps, retrieval actor, auth tier, abuse flags, HMAC source IP/user-agent) are documented verbatim. The HMAC requirement is normative: where audit records reference contact identifiers or device identifiers, they MUST use HMACs with protected audit keys; plain hashes of email or phone numbers are explicitly excluded.

**Withdrawal-as-state-transition (§11.9).** A third normative rule applies:

> Withdrawal stops future processing but does not erase audit records, consent receipt metadata, state history, abuse signals, dispute records, or legally required retention records.

Six distinct withdrawal scenarios are documented: consumer withdrawal before data sharing, consumer withdrawal after Stage 2 viewed, consumer withdrawal after Stage 3 retrieval, professional withdrawal from an introduction, professional withdrawal from matching, Component 3 proposal/connection withdrawal. Each carries its own semantics and reason-code space. Withdrawal action records carry F1.17 fields: authenticated actor, authority basis, timestamp, affected object, previous state, resulting state, reason code. High-impact withdrawals (professional withdrawal from matching; revocation of a professional delegated-agent token; professional record deletion where supported) MUST require step-up authentication at the point of action.

The decision is recorded across the spec (§11.1 overview, §11.2 deletion table refactored along the three concepts, §11.6 crypto-erasure with implementation requirements, §11.8 audit retention, §11.9 withdrawal semantics, §13.2A step-up authentication for high-impact withdrawals); the audit-event schema (`audit-event.schema.json` extended with the F1.16 retain fields and F1.17 withdrawal-context fields); the handoff-component2 schema (`withdrawal_state` summary classification); the OpenAPI surface (existing `/cancel` and `/revoke-consent` endpoints documented per scenario; new `POST /v1/professionals/me/withdraw` with step-up); and the MCP binding (`cancel_introduction` and `revoke_consent` descriptions updated; new `withdraw_from_matching` tool).

## Alternatives considered

**Single retention model with field-level rules.** Considered: a single "retention" concept with rules attached to each field. Rejected: too unclear at the conceptual level — readers conflate "this field is retained for 7 years" with "this object is retained for 7 years". The three-concept separation tells the reader what is happening to the data (payload-vs-metadata) before getting into per-field retention rules.

**Full erasure on withdrawal.** Considered: when a user (consumer or professional) withdraws, erase all associated records. Rejected as an evidence-deletion abuse primitive. The asymmetry — consumer or professional gains the ability to retroactively erase the protocol's record of their actions — would break dispute resolution, abuse investigation, and consent-integrity verification.

**Separate `withdrawal-record.schema.json`.** Considered: introduce a dedicated record type for withdrawals, separate from the audit log. Rejected as duplication: every withdrawal already generates a structured audit event with the F1.17 required fields. A separate record would carry the same information twice and create synchronisation complexity. Future revisions can reconsider if a richer withdrawal-record shape becomes useful.

**Plain hash of email and phone in audit records.** Considered: continue using plain hashes (as v0.8.1 did implicitly). Rejected because plain hashes of email addresses and phone numbers are guessable across the population they identify — effectively reversible at scale. HMAC with a protected audit key is the standard mitigation.

## Consequences

- `audit-event.schema.json` carries materially more fields (`handoff_id`, `query_id`, `professional_id`, `proposal_id`, `connection_id`, `pseudonymous_consumer_ref`, `request_intent`, `stage`, `payload_type`, `consent_receipt_id`, `receipt_hash`, `retrieved_at`, `erased_at`, `retrieval_actor`, `auth_tier`, `abuse_flags`, `source_ip_hmac`, `user_agent_hmac`, `previous_state`, `resulting_state`, `reason_code`). All optional except where noted; population is event-type-dependent.
- The schema's description text now encodes the §11.8 prohibition on plaintext consumer-identifying fields and the HMAC requirement.
- Implementations need a separate key-management lifecycle for audit HMAC keys (distinct from application authentication keys). Key rotation, key access control, and key-destruction logging are operational concerns documented in deployment notes; v0.8.2 spec mandates the requirement, v0.9 may standardise.
- Step-up authentication is now required at the spec level for three high-impact withdrawal types. The protocol does NOT standardise the step-up mechanism in v0.8.2 (re-auth via OAuth flow, second factor via TOTP, out-of-band push approval, or equivalent). This is a documented v0.8.2 gap and a v0.9 standardisation candidate.
- The §11.2 deletion table is wider (5 columns instead of 4 — Store / Payload / Audit metadata / Withdrawal state / Retention rule) and longer to read. The trade-off is correctness over conciseness.
- The Charter Part Two privacy operational commitment now includes a one-line reference to the three-concept separation, with detailed semantics in spec §11. Phase 8 may extend this further.

## References

- Spec §11 (full refactor): §11.1 (overview with three-concept framing), §11.2 (deletion table), §11.6 (vault + crypto-erasure verbatim wording + implementation requirements), §11.8 (audit retention with verbatim rule + HMAC requirement), §11.9 (withdrawal semantics with verbatim rule + six scenarios + step-up requirement).
- Spec §13.2A (step-up authentication for high-impact withdrawals).
- Schema: [`audit-event.schema.json`](../../spec/v0.8/schemas/audit-event.schema.json) — F1.16 retain fields and F1.17 withdrawal-context fields.
- Schema: [`handoff-component2.schema.json`](../../spec/v0.8/schemas/handoff-component2.schema.json) — `withdrawal_state` field.
- OpenAPI: `POST /v1/professionals/me/withdraw` (new, high-impact, step-up required); `/cancel` and `/revoke-consent` descriptions updated per §11.9 scenarios.
- MCP tools: `withdraw_from_matching` (new), `cancel_introduction` and `revoke_consent` descriptions updated.
- Master patch plan v3.1.1, Phase 3 (§5) and Paolo's findings F1.14 (three-concept separation), F1.15 (crypto-erasure precise wording), F1.16 (audit retain/erase field lists + HMAC), F1.17 (withdrawal as state transition).
- Source review: `PAOLO-PEER-REVIEW-FINDINGS-01-v0_8_1.md` and `PAOLO-erasure-audit-and-withdrawal.md`.
