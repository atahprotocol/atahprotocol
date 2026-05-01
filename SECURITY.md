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

## 4. Worked threat scenarios

The full STRIDE-level model per asset is deferred to v0.9. The three
scenarios below illustrate how the v0.8 controls handle representative
threats. They are illustrative, not exhaustive.

### 4.1 Leaked handoff_id

**Threat.** A `handoff_id` leaks into a screenshot, support ticket,
log file, or URL captured by a third party. The third party attempts to
read or modify the handoff state.

**Attempted exploit.** Third party calls `check_introduction_status`
with the leaked `handoff_id`, then attempts to call `submit_stage2_data`
or `cancel_introduction` on the same handoff.

**Detection and mitigation.** The tiered handoff access control model
(§11.5 of the spec) requires a `handoff_access_token` for all
state-changing operations. The token is bound to platform/client/scope/
expiry and is not present in the leaked artefact. Read-like access via
`check_introduction_status` succeeds only for the original asserter
platform or for an authenticated AI platform supplying a matching
`consumer_ref` (which the third party does not have). Audit log records
the read attempt with pseudonymous identifiers.

**Residual risk.** Read-tier access to the handoff metadata (state,
category, professional reference) is possible if the third party also
holds a valid `consumer_ref`. The `consumer_ref` constraints in §11.5
limit this to actors who have already obtained user-mediated continuity
consent. Monitoring for anomalous read patterns is operational; formal
specification deferred to v0.9.

### 4.2 Malicious AI platform asserts forged consent receipt

**Threat.** A compromised or malicious AI platform attempts to submit
an `initiate_introduction` call with a fabricated consent receipt for a
consumer who did not actually consent.

**Attempted exploit.** Platform generates a structured consent receipt
locally, computes its hash, and submits the hash to ATAH at introduction
creation. Platform retains the locally-fabricated full receipt to
produce on demand.

**Detection and mitigation.** The receipt hash matches the locally
fabricated receipt. The exploit is not detectable at introduction time
purely on the basis of hash verification. However:

- The asserting platform is OAuth 2.1 authenticated with audience
  validation; the exploit requires platform-side compromise, not
  external forgery.
- The consumer can revoke consent at any time via `revoke_consent`,
  which immediately crypto-erases vault contents and halts the handoff.
- Post-hoc audit: the consumer (through any AI platform with their
  authenticated continuity grant) can request the introduction history
  associated with their `consumer_ref` and identify introductions they
  did not initiate.
- Pattern detection on platform-side anomalies (introductions to
  professionals the consumer has no plausible relationship with;
  introductions outside the consumer's normal jurisdiction; clustered
  introductions in short time windows) flags suspicious assertion.
- A platform shown to have asserted forged consent has its OAuth
  credentials revoked and is suspended pending investigation.

**Residual risk.** A single forged assertion is detectable only post-hoc
or by pattern. A platform with sustained access to a consumer's account
and a coordinated forgery campaign could cause real harm before
detection. The v0.9 platform-light consent mode mitigates this further
by moving the consent ceremony to ATAH, reducing the platform-side
attack surface.

### 4.3 Compromised partner submits poisoned data

**Threat.** A compromised trusted partner (or one whose source data is
itself compromised) submits poisoned professional data — for example,
fabricated credentials for individuals who do not exist, or fabricated
disciplinary records against legitimate professionals.

**Attempted exploit.** Partner submits a batch via the partner data
push schema (§7.6). The submission is signed and authenticated.

**Detection and mitigation.**
- Partner submissions are signed; the signature confirms partner
  identity, not partner data integrity.
- Conflict detection: where partner-provided data conflicts with
  another source (another partner, an authoritative registry, the
  professional's own self-declared record, an enhanced verification
  record), the affected profile is suppressed from matching pending
  resolution.
- Anomaly detection on partner submissions: large batches with
  unusually high rates of new-record creation, batches affecting a
  disproportionate number of records in a single jurisdiction, or
  batches that contradict prior submissions trigger admin review
  before propagation.
- Affected professionals are notified and given the dispute resolution
  process (§8.10).
- A partner shown to have submitted poisoned data is suspended; their
  data is rolled back where rollback is technically feasible; the
  rollback is audit-logged and disclosed publicly.

**Residual risk.** Single poisoned records consistent with the
partner's normal submission pattern may not be detected at submission
time. Detection relies on conflict with another source or on a
professional disputing the data. Sophisticated coordinated poisoning
across multiple partners is research-grade adversarial work and
detection is partial; full Sybil/coordinated-attack detection is
deferred to v0.9.

## 5. Controls (overview)

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

## 6. Partner and verifier authentication

High-trust writes by partners and verifiers require:

- Signed JWT client assertion **or** HMAC-signed requests with
  `Date`/`X-ATAH-Timestamp`, nonce, replay-protection window, and
  request-body signature
- API keys are accepted only for read-only operations, never for high-trust
  writes

## 7. Audit policy

The following events are recorded in the audit log:

- All state transitions on handoffs and proposals
- Consent assertions and revocations
- Data retrievals through authenticated retrieval
- Partner data writes and retractions
- Verifier writes and revocations
- Admin actions (suspensions, conflict resolutions, dispute decisions, flag
  reviews)

Audit log entries are hash-chained and use minimised, pseudonymous identifiers. They are not directly identifying within ATAH alone but may constitute personal data where linkable by ATAH, the asserting platform, a partner, or a regulator. Retention period is 7 years, justified by the purposes of fraud detection, dispute resolution, and regulator engagement. The lawful basis is legitimate interest, with the balancing test reflected in the operational privacy policy. Subjects of audit (professionals, partners,
verifiers) may request entries concerning themselves through the
`/v1/professionals/me/disputes` and admin paths.

## 8. Incident response

ATAH maintains an incident-response posture for two specific scenarios that
have a material consumer impact: confirmed professional fraud and confirmed
professional account compromise.

### Confirmed professional fraud

On confirmed fraud (e.g. a professional uses ATAH-routed introductions to
defraud consumers, or a professional record is shown to be fraudulent in
ways the registration credibility checks did not catch):

- **Immediate suspension** of the affected professional record, removing it
  from match results.
- **Partner notification** where the professional was represented through a
  partner.
- **Affected-introduction review** of recent and in-flight introductions
  involving the professional.
- **Consumer notification** through the asserting AI platforms for affected
  introductions.
- **Cooperation** with law enforcement and regulators where legally
  appropriate.
- **Public disclosure** through the security advisory channel where the
  fraud was facilitated by a protocol-level weakness.

ATAH does not commit to compensation for losses arising from professional
fraud. Consumer-side compensation considerations sit with the consumer,
their AI platform, the professional's regulator (where applicable), and any
relevant law enforcement.

### Professional account compromise — sensitive change controls

Sensitive changes to a professional's record — contact details, payout or
payment details, licence jurisdiction, firm affiliation, fee model, and
availability routing — require step-up authentication beyond the standard
magic link or SMS one-time code. The change is also notified to the
prior verified channel, allowing the professional to detect unauthorised
changes through a channel that has not itself been compromised.

This control is operational from launch. Full claim-authentication
hardening with anomaly detection on attempted changes (rapid succession from
new IPs, attempts to immediately edit fee or contact fields after a claim)
is deferred to v0.9 per ROADMAP.

## 9. Vulnerability hall of fame

This section will be populated as disclosures are resolved. Reporters who wish
to be credited will be listed with the disclosure date and a one-line
description of the issue.

## 10. Bounty programme

ATAH does not currently operate a paid bounty programme. We commit to
handling responsible disclosures in good faith, crediting reporters where
they wish, and to publishing a transparent post-disclosure summary for any
substantive issue. A bounty programme may be considered post-transition.

## 11. Contact

- Security: `security@atahprotocol.org`
- General: `hello@atahprotocol.org`
- Governance: `governance@atahprotocol.org`
