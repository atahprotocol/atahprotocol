# Security Policy

ATAH treats security and privacy as design commitments, not afterthoughts. The
mechanisms are described in detail in the [specification](spec/v0.8/full-spec.md)
(Section 13 Security Considerations and Section 11 Privacy and Data
Lifecycle); this document covers reporting, disclosure policy, and the v0.8
threat-model baseline.

## 1. Reporting a security issue

**Do NOT open a public GitHub issue for security disclosures.**

Email `security@atahprotocol.org` with the details of the issue. Where you can,
encrypt with our published PGP key (the fingerprint is published at
`https://atahprotocol.org/.well-known/security-pgp-key.asc` — verify the
fingerprint via the same URL).

Include:

- Nature of the issue and the affected component (spec section, schema,
  endpoint, MCP tool, reference registry behaviour, etc.)
- Steps to reproduce, where applicable
- Affected versions
- Your contact information and whether you wish to be credited

We acknowledge security reports within **72 hours** and will provide a
substantive response within **14 days** of acknowledgement.

## 2. Disclosure policy

We follow coordinated disclosure. The default window is **90 days** from
acknowledgement to public disclosure, negotiable based on severity and
remediation complexity.

- Reporters are credited in the public disclosure unless they prefer
  otherwise.
- We will work with reporters on remediation timeline and content of the
  disclosure.
- For severe issues affecting the reference registry, ATAH may coordinate
  with affected partners, verifiers, or AI platforms before public
  disclosure.

## 3. Threat-model baseline (v0.8)

The full STRIDE-level model per asset is **deferred to v0.9**. v0.8 ships with
the following baseline.

**Trust boundaries:**

- AI platform ↔ ATAH
- ATAH ↔ trusted partner
- ATAH ↔ independent verifier
- ATAH ↔ professional
- ATAH ↔ transient encrypted vault
- ATAH ↔ audit log

**In-scope threat actors and concerns:**

- A malicious or compromised AI platform asserting false consent
- A malicious or compromised professional
- A malicious or compromised partner submitting false data
- Review-gaming rings and Sybil professionals
- Verifier capture or compromise
- Scrapers and data exfiltrators
- PII leakage attempts (notification channels, query results, audit logs)
- Consent-receipt forgery attempts
- Partner data poisoning
- Insider admin abuse
- `handoff_id` leakage attempts (the tiered handoff-access model assumes
  `handoff_id` may leak; possession alone does not grant access)

**Out of scope at v0.8 (deferred to v0.9):**

- Full STRIDE modelling per asset
- Federation-related cross-registry threats
- Comprehensive Sybil and referral-ring detection
- Step-up authentication and anomaly detection on professional claim attempts
  (the minimum is mandated; the hardening is deferred per spec §15.6)

## 4. Controls (overview)

Full detail is in spec §13. Summary:

- **Authentication.** OAuth 2.1 with audience validation, explicit scopes,
  short-lived access tokens with refresh and rotation. Signed JWT client
  assertions or HMAC-signed requests for partner and verifier writes. Magic
  link or OIDC for professional sessions.
- **Tiered handoff access.** `handoff_id` alone does not grant write access.
  Read-tier requires consumer-reference match. Write-tier requires a
  `handoff_access_token` bound to platform/client/scope/expiry (spec §11.5).
- **Consent receipts.** Hash-and-metadata stored at ATAH; full receipt
  retained by the asserting platform. ATAH never stores consumer-identifying
  fields in receipt records (spec §4.10 and the
  `consent-receipt-stored.schema.json` privacy floor).
- **Transient encrypted vault.** Per-record DEKs; crypto-erasure as primary
  deletion mechanism; PII never transmitted via SMS or email — only
  notification metadata and authenticated retrieval references (spec §11.6).
- **Audit log.** Tamper-evident hash chaining; pseudonymous identifiers only;
  7-year retention. Subjects of audit may request entries concerning
  themselves.
- **Rate limiting** on public endpoints; stricter on `/v1/query` to make
  scraping unattractive.
- **Encryption.** TLS 1.3 in transit. Encryption at rest. 90-day master key
  rotation. HSM-backed signing keys for high-trust operations.

## 5. Partner and verifier authentication

High-trust writes by partners and verifiers require:

- Signed JWT client assertion **or** HMAC-signed requests with
  `Date`/`X-ATAH-Timestamp`, nonce, replay-protection window, and
  request-body signature
- API keys are accepted only for read-only operations, never for high-trust
  writes

## 6. Audit policy

The following events are recorded in the audit log:

- All state transitions on handoffs and proposals
- Consent assertions and revocations
- Data retrievals through authenticated retrieval
- Partner data writes and retractions
- Verifier writes and revocations
- Admin actions (suspensions, conflict resolutions, dispute decisions, flag
  reviews)

Audit log entries are hash-chained, store pseudonymous identifiers only, and
are retained for 7 years. Subjects of audit (professionals, partners,
verifiers) may request entries concerning themselves through the
`/v1/professionals/me/disputes` and admin paths.

## 7. Vulnerability hall of fame

This section will be populated as disclosures are resolved. Reporters who wish
to be credited will be listed with the disclosure date and a one-line
description of the issue.

## 8. Bounty programme

ATAH does not currently operate a paid bounty programme. We commit to
handling responsible disclosures in good faith, crediting reporters where
they wish, and to publishing a transparent post-disclosure summary for any
substantive issue. A bounty programme may be considered post-transition.

## 9. Contact

- Security: `security@atahprotocol.org`
- General: `hello@atahprotocol.org`
- Governance: `governance@atahprotocol.org`
