# ATAH

**Agent to Authenticated Human Protocol**

**Technical Specification**

| Version | 0.8 |
| --- | --- |
| Status | Release Candidate |
| Date | April 2026 |
| Founded by | Grahame Cohen |
| Audience | Developers, AI coding agents, partner integrators, peer reviewers |

---

## Version History

**v0.8** — Release-candidate specification incorporating pre-publication peer review remediation. Adds: structured consent receipts replacing boolean assertions; tiered handoff_id security model with separate handoff_access_token; transient encrypted vault delivery model with notification + authenticated retrieval (no PII through SMS or email); full Type 2 and Type 3 introduction schemas; parallel provenance map for per-field provenance; explicit standards composition (OAuth 2.1, OpenID Connect, W3C Verifiable Credentials, DIDs); three-layer protocol/registry/implementation framing; federation deferral with federation-ready architecture; opaque atah_id format; idempotency requirement on mutating endpoints; authorisation matrix; deletion scope table; non-goals section; glossary; standards-composition section; concern flag protections. Earlier v0.8 changes preserved: Stage 2 reframed as a category-flexible pre-handoff check; partner records declare verification depth and vetting strength; two registration routes (partner and individual) with roll-up logic; profession category metadata; individual self-registration fee model; enhanced verification layer with independent verifier model; review platforms formalised as a specialised partner class with anti-gaming requirements and differential treatment in matching. Verification status enum tightened — profile-level states moved to a dedicated `matching_status` field. Matching engine Step 3 restructured to expose four sub-components with transparent contribution metadata. Protocol renamed from "Agent to Accredited Human Protocol" to "Agent to Authenticated Human Protocol" — acronym ATAH unchanged.

**v0.7** — Five issues resolved: atomic deletion compensating action pattern specified; MCP endpoint authentication mechanism defined (OAuth 2.0); verification batching strategy made concrete; simultaneous introduction rules specified explicitly; schema versioning policy added.

**v0.6** — Targeted corrections: state bar verification scoped to priority states at MVP; webhooks and MCP push repositioned as Phase 2. Conflict field classification made explicit. Referral signal discount threshold defined concretely. Notification model corrected.

**v0.5** — Initial technical specification.

---

## 1. Purpose of This Document

This document is the technical specification for ATAH (Agent to Authenticated Human Protocol). It should be read after the [README](../../README.md) and [EXPLAINER](../../EXPLAINER.md). It covers the full schema definitions, API endpoints, MCP server specification, trusted partner model, matching engine design, introduction lifecycle state machine, verification architecture, data governance implementation, notification architecture, repository structure, and build sequence.

This is a release-candidate specification intended for technical, governance, and ecosystem review. It defines the core data model, introduction lifecycle, provenance model, matching principles, consent stages, and governance commitments. The machine-readable schemas, API contracts, and reference implementation will be hardened through public review and implementation work. Breaking changes are possible until v1.0 — the version negotiation and deprecation policy in Section 7.1 governs how those changes are managed.

The governance commitments that constrain how this specification may evolve are in [CHARTER.md](../../CHARTER.md). The most important of those commitments — that matching logic carries no commercial weighting, that provenance is always visible, that the MCP endpoint is free to query, that consumer personal data is never centrally retained, that no professional category is surfaced before its compliance annex is approved — are reflected throughout this document. Implementations are expected to honour them.

## 1.5 What ATAH Is and Is Not

ATAH is an open specification for AI-to-authenticated-human professional handoff. The protocol defines:

- A standardised data model for professional identity, with verification provenance
- A trusted partner model for surfacing regulatory and professional-body data
- A staged introduction lifecycle with explicit consent at each stage
- A matching engine producing provenance-visible shortlists
- Commercial neutrality and privacy commitments enforced by Charter

### Three layers

ATAH operates across three layers, and clarity about which layer is being discussed is important:

1. **The open specification.** The protocol itself, defined in this document and the schema files, published under Apache 2.0. Anyone may implement, fork, or extend it.

2. **Conforming implementations.** Software systems that implement the specification. Conformance requires preserving ATAH's mandatory principles: provenance visibility, no commercial weighting, staged consent, category compliance gating, and data minimisation.

3. **The initial reference registry.** ATAH operates an initial reference registry and MCP/REST endpoints that demonstrate and operate the protocol. The reference registry is not the protocol itself. Other registries may exist, provided they preserve the conformance principles.

The initial registry exists because launching a protocol with no operational instance produces no real-world adoption. As federation develops (deferred to v0.9 or v1.0 — see Section 1.6), atah_id namespaces and cross-registry trust mechanics will be specified to support multiple conforming registries.

### Protocol core, bindings, and registry profile

ATAH v0.8 separates the protocol into three specification surfaces:

1. **Core protocol objects.** Professional records, provenance maps, category metadata, consent receipts, handoff records, outcome reports, audit events, and presentation disclosures. These objects are transport-neutral. The protocol does not require MCP, REST, A2A, or any single hosting model at the core object level.

2. **Transport bindings.** v0.8 defines MCP and REST bindings. A binding defines how core protocol objects are requested, transmitted, and updated. Bindings do not change core semantics. Future versions may define A2A-compatible, push, webhook, or federation bindings.

3. **Registry operation profile.** The set of behaviours required for systems that store professional records, ingest partner evidence, run matching, and manage introduction lifecycle state. A registry implementing this profile is a *conforming registry*.

The initial ATAH reference registry implements all three surfaces. A future conforming implementation may implement the core protocol and one or more bindings, subject to the conformance principles and federation rules then in force. See Section 14 (Conformance Classes) for how implementations may declare which surfaces they support.

### What ATAH is

- A specification for the layer above identity and credentialing standards
- An open protocol for AI-to-human professional handoff
- A trust framework with explicit provenance for every data point
- A neutrality commitment with structural guards against commercial capture

### What ATAH is not (non-goals)

- **ATAH is not a recommendation engine.** It returns provenance-visible shortlists based on declared need, verification evidence, category rules, and availability. The user or calling AI platform remains responsible for selection.
- **ATAH is not a regulator, enforcement body, or complaints adjudicator.** Professional conduct concerns are routed to the relevant regulatory or professional body. ATAH does not investigate, adjudicate, or sanction professionals based on consumer complaints.
- **ATAH is not a payment processor or marketplace.** It carries no transaction or commerce capability.
- **ATAH is not a directory service or AEO replacement.** Professionals are not "promoted" or "ranked for visibility" in any commercial sense — matching is determined entirely by the rubric in Section 9.
- **ATAH does not warrant outcomes.** Matching is best-effort within the published rubric and verification freshness windows. AI platforms and consumers retain responsibility for selection and use. ATAH commits to specific data freshness windows, dispute resolution timelines, and concern-routing protocols, but does not warrant the suitability, competence, or behaviour of any individual professional.
- **ATAH is not a personal data repository.** Consumer personal data passes through ATAH only as transient handoff data and is deleted per the protocol in Section 11. ATAH does not retain, profile, or derive insight from consumer data.
- **ATAH is not a consumer interface.** Consumers do not interact with ATAH directly. They interact with AI systems, which use ATAH on their behalf.

## 1.6 Relationship to Existing Standards

ATAH composes with established identity, credential, and authorisation standards rather than replacing them. ATAH's specific contribution is the layer above — professional categorisation, the staged introduction lifecycle, the matching engine, the trusted-partner trust model, the commercial-neutrality and provenance-visibility commitments — none of which exist in the underlying identity and credential standards.

**OAuth 2.1** is the basis for ATAH's MCP and REST API authentication. AI platforms, partners, and verifiers authenticate via standard OAuth flows with ATAH-specific scopes (Section 8 and Section 13). Where MCP authorisation guidance evolves (protected resource metadata, authorization server metadata, resource indicators, audience validation), ATAH conforms to the current MCP authorisation specification at the time of implementation.

**OpenID Connect (OIDC)** is supported for professional portal authentication and may be used by partners and verifiers. Magic-link and SMS authentication remain available for professionals without an OIDC provider.

**W3C Verifiable Credentials (VCs)** are an accepted format for trusted partner data submissions. A regulator or professional body that issues data as Verifiable Credentials may push them directly through the partner API; ATAH verifies the cryptographic proof and stores the claim with full provenance. Partners without VC infrastructure may submit structured JSON via the standard partner data API. Both routes produce the same provenance representation in the registry.

**Decentralized Identifiers (DIDs)** are not required by v0.8 but the `atah_id` format and partner identifier model are designed to be compatible with future DID resolution.

**W3C Consent Receipts (Kantara CR 1.1, ISO/IEC 29184)** inform ATAH's consent receipt structure (Section 4.10). ATAH consent receipts may be expressed in formats compatible with these standards.

**Sigstore / RFC 6962-style transparency logs** inform the consent receipt verification model (Section 11.5): ATAH stores receipt hashes and metadata; the asserting platform stores the full receipt; verification at dispute time confirms the platform has not altered the receipt after the fact.

### Federation

ATAH v0.8 operates as a single reference registry. The protocol is structured to support federation in future versions:

- The `atah_id` format includes a registry namespace prefix
- No part of the data model assumes a single registry
- The matching engine is profile-driven rather than implementation-locked
- Partner and verifier identifiers are not implicitly scoped to a single registry

Federation mechanics — cross-registry trust, atah_id resolution across registries, handoff routing between registries, and federated governance — are deferred to v0.9 or v1.0. Implementations conforming to v0.8 are expected to interoperate with the reference registry until federation is specified.

## 2. Reference Implementation Architecture Overview

This section describes the recommended architecture for the initial ATAH reference registry. It is not the only permitted architecture for conforming implementations. A conforming implementation may use different infrastructure if it preserves the core protocol semantics, security requirements, privacy requirements, and conformance principles set out in Section 14.

The reference implementation consists of eight core components.

- **Registry service** — stores and manages professional profiles, verification records, trusted partner data, and referral relationship data. Does not store consumer personal data.
- **Trusted partner engine** — manages partner onboarding, data ingestion (including Verifiable Credentials where applicable), freshness tracking, conflict detection, and partner scoring.
- **Matching engine** — in the reference implementation, runs within the ATAH registry service. Processes queries, scores candidates, returns ranked shortlists with full trusted partner data payloads and per-sub-component contribution metadata. A conforming registry MAY implement matching internally or through a delegated service, provided the match response preserves ATAH's scoring transparency, provenance, presentation disclosure, and no-commercial-weighting requirements.
- **Introduction lifecycle manager** — manages state for all active introductions across all three types, handles transitions, enforces timeouts, triggers notifications, and enforces simultaneous introduction rules. Issues handoff_access_tokens for write operations.
- **Transient data vault** — encrypted, time-limited store for consumer personal data passing through introductions. Strictly transient. Separate from the main registry. Crypto-erased per category retention rules.
- **MCP server** — exposes ATAH capabilities as MCP tools to any MCP-compatible AI agent. Authenticated per current MCP authorisation guidance (OAuth 2.1-compatible). This is the v0.8 MCP binding.
- **REST API** — full programmatic access for non-MCP clients. Authenticated via OAuth 2.1 client credentials for AI platforms; signed-request mechanisms for partners and verifiers; magic-link/OIDC for professionals. This is the v0.8 REST binding.
- **Notification service** — sends non-PII notifications to professionals on state changes via SMS and email. Notifications carry only metadata and authenticated retrieval references — never personal data. Supports polling by consumer AI agents. MCP push and webhook delivery to consumer AI agents are Phase 2.

Supporting infrastructure: Postgres database, Redis for session and queue management, transient encrypted store for consumer personal data, object storage for documents and audit logs, webhook delivery system (Phase 2).

## 3. Recommended Technology Stack

These are recommendations for the reference implementation, not requirements of the protocol. Any stack capable of meeting the spec's behaviour and security requirements is acceptable.

- Backend — TypeScript, Node.js
- API framework — Express or Fastify
- Database — Postgres
- Cache and queue — Redis
- Transient encrypted store — Redis or Postgres with row-level encryption and TTL-based crypto-erasure
- SMS — Twilio or equivalent (notifications only — no PII)
- Email — Resend, Postmark, or equivalent (notifications only — no PII)
- Object storage — AWS S3 or equivalent
- Webhooks — standard HTTP POST with retry logic and signature verification (Phase 2 for consumer AI agent delivery; available in Phase 1 for partner notifications)
- Authentication — magic link, SMS one-time code, or OIDC for professionals (no passwords). OAuth 2.1 for AI platform MCP and REST access. Signed-request mechanisms (HMAC or signed JWT client assertion) for partner and verifier data writes.
- Hosting — AWS, Railway, Fly.io, or equivalent. High availability required for MCP endpoint.
- Repository — GitHub, public, Apache 2.0 licence

## 4. Core Data Schemas

### 4.1 Schema Conventions

All ATAH schemas conform to the following conventions:

- **Schema language:** JSON Schema Draft 2020-12.
- **Identifier format:** opaque ULID-style identifiers with a typed prefix. The reference registry uses the following prefixes (non-exhaustive): `atah-` (professionals), `tp-` (trusted partners — including review platforms, identified via their `partner_id`), `iv-` (independent verifiers), `cr-` (consent receipts), `hf-` (handoffs), `ev-` (enhanced verifications), `q-` (queries), `att-` (professional attestations), `dispute-` (dispute records), `flag-` (concern flags), `vault-` (transient vault references), `ref-` (Type 3 referrals), and `proposal-` (Type 2 proposals). Additional prefixes used in records and audit events (such as `evt-`, `req-`, `conflict-`, `rollup-`, `batch-`, `cdim-`, `conf-`) follow the same convention. Regional or future federated registries use distinct prefixes (e.g. `atah-eu-`, `atah-uk-`). Identifiers are case-sensitive and must match the regex `^[a-z]{2,8}-[a-z0-9-]{6,64}$`.
- **Timestamps:** RFC 3339 in UTC, e.g. `"2026-04-10T14:21:00Z"`. All response time SLAs are measured in elapsed wall-clock hours, not business hours. Availability is expressed with the timezone of operation.
- **Schema versioning:** every schema-bearing object carries a `schema_version` field. The protocol-level version is carried in `protocol_version`.
- **Closed enums by default.** Open enums are explicitly marked.
- **Strict additional properties.** Schemas use `additionalProperties: false` unless a specific extensibility need is documented.
- **Nullability:** explicit. Optional fields are either omitted or carry an explicit `null`. The schema specifies which.
- **Provenance:** carried in a parallel `_provenance` map (Section 4.2), not by wrapping individual fields.

### 4.2 Provenance Model

ATAH's headline trust commitment is that every data point carries its source and verification status. This is implemented through a parallel provenance map on every object that contains verifiable claims.

The provenance map (`_provenance`) is a top-level property whose keys are field paths within the object (using dot-notation for nested fields and array index syntax for arrays). Each entry records `verification_status`, `source`, and `as_of`:

```json
{
  "atah_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
  "name": "Jane Smith",
  "firm": "Smith Legal LLC",
  "jurisdictions": ["US-TX", "US-NY"],
  "specialisms": ["commercial_contracts", "employment_law"],

  "_provenance": {
    "name": {
      "verification_status": "partner-verified",
      "source": "tp-statebar-tx-001",
      "as_of": "2026-04-01T00:00:00Z"
    },
    "firm": {
      "verification_status": "self-declared",
      "source": "professional",
      "as_of": "2026-03-15T00:00:00Z"
    },
    "jurisdictions": {
      "verification_status": "partner-verified",
      "source": "tp-statebar-tx-001",
      "as_of": "2026-04-01T00:00:00Z"
    },
    "jurisdictions[1]": {
      "verification_status": "self-declared",
      "source": "professional",
      "as_of": "2026-03-15T00:00:00Z"
    },
    "specialisms": {
      "verification_status": "self-declared",
      "source": "professional",
      "as_of": "2026-03-15T00:00:00Z"
    }
  }
}
```

Every field that carries a verifiable claim must have a corresponding entry in `_provenance`. Fields that are purely informational (e.g. response time preferences) may be omitted.

When an entry in `_provenance` is missing for a field that should have one, AI platforms and consuming systems must treat the field as `not-provided` for trust purposes, regardless of the field's value.

### 4.3 Verification Status Codes

Applied to fields via `_provenance`. These statuses describe how an individual data point was verified — they are not profile-level states. Profile-level states are recorded in the `matching_status` field on the profile.

- `registry-verified` — confirmed active against an authoritative source by ATAH directly
- `partner-verified` — confirmed by a named trusted partner with active partner status
- `self-declared` — professional has stated this, not yet verified. Always surfaced as such. Never presented with false confidence.
- `verification-failed` — verification attempted but could not be confirmed
- `verification-deferred` — verification due but external source unavailable or rate-limited. Profile remains active in matching. Status flagged as stale after 7 days beyond due date.
- `vc-verified` — confirmed via cryptographic verification of a W3C Verifiable Credential signed by an approved issuer. Treated as equivalent to `partner-verified` for matching purposes; the issuer is recorded in `source`.
- `not-provided` — field not declared or uploaded

Enhanced verification is a separate concept and does not appear in this enum. It is recorded as a structured object on the profile (Section 4.9) and contributes to the verification quality score in matching (Section 9). It is not a status applied to individual data points.

*AI agents receiving match results should present the verification status of each data point transparently. Self-declared data is surfaced as such. The absence of partner-verified data is not penalised — only its presence rewards.*

#### Profile-level `matching_status`

Distinct from per-field verification status, every profile carries a `matching_status` field describing whether the profile is eligible for matching. Values:

- `active` — profile is eligible to be returned in match results (subject to filters)
- `compliance-pending` — profession category requires a compliance annex before matching is enabled. Profile exists and is maintained. Professional is not returned in match results.
- `regulatory-suspended` — regulator-reported adverse status (licence revoked, disciplinary action, etc.). Profile suppressed from matching pending review.
- `admin-suspended` — ATAH admin suppression for protocol-level concerns (data integrity, conflict pending resolution). Profile suppressed from matching pending review.
- `pending-rollup` — individual self-registered profile flagged for low-confidence rollup match against a partner roster (Section 10.4). Profile remains active in matching while review is pending.
- `pending-integrity-review` — profile flagged at registration by integrity checks (Section 10.3). Held pending review before becoming active.
- `withdrawn` — professional has withdrawn from the registry. Profile retained for audit; not surfaced.
- `deceased` — professional reported as deceased. Profile retained for audit; not surfaced.

### 4.4 Profession Category Metadata

Profession categories are referenced throughout the spec (lawyer, pr_professional, physician, tax_planner, and so on). Each supported category has a metadata record describing its tier, body membership structure, default pre-handoff check type, compliance status, and category-specific matching weight profile.

The full set of supported profession categories is maintained in `profession-categories.json` in the repository. New categories are added through the standard governance pull request process described in [GOVERNANCE.md](../../GOVERNANCE.md). Two representative examples are shown inline:

```json
{
  "category_id": "lawyer",
  "category_name": "Lawyer",
  "professional_tier": "credentialled",
  "body_membership_type": "mandatory_per_jurisdiction",
  "compliance_status": "active",
  "default_pre_handoff_check": "conflict_check",
  "matching_weight_profile": {
    "relevance": 0.30,
    "verification_quality": 0.40,
    "availability_response": 0.15,
    "profile_completeness": 0.05,
    "inbound_referral_signal": 0.10
  },
  "review_signal_weight_cap": 0.10,
  "stage2_auth_tier": "tier_3"
}
```

```json
{
  "category_id": "pr_professional",
  "category_name": "Public Relations Professional",
  "professional_tier": "established",
  "body_membership_type": "optional_with_strong_signal",
  "compliance_status": "active",
  "default_pre_handoff_check": "scope_confirmation",
  "matching_weight_profile": {
    "relevance": 0.30,
    "verification_quality": 0.25,
    "availability_response": 0.20,
    "profile_completeness": 0.10,
    "inbound_referral_signal": 0.15
  },
  "review_signal_weight_cap": 0.20,
  "stage2_auth_tier": "tier_2"
}
```

The `body_membership_type` field has values:

- `mandatory_per_jurisdiction` — body membership is mandatory by jurisdiction (e.g. lawyers must be admitted to a bar to practice in that jurisdiction)
- `mandatory_global` — single global body required to practice
- `optional_with_strong_signal` — body membership is optional but carries strong professional weight (e.g. CIPR Fellow status)
- `optional_with_weak_signal` — body membership is optional and carries limited signal
- `none` — no recognised professional body for the category

The `compliance_status` field has values: `active` (open for matching), `compliance-pending` (registration permitted, matching suppressed pending compliance annex), and `not-supported` (registration not accepted).

The `default_pre_handoff_check` field has values: `conflict_check`, `scope_confirmation`, `capacity_confirmation`, `fee_alignment`, or `none`. See Section 6 for what each pre-handoff check type contains.

The `matching_weight_profile` field defines the category-specific final score weights used in Section 9. Where a category does not specify the field, the protocol defaults apply (Section 9.7). Category-specific weights are governance-approved and may not be set unilaterally.

The `review_signal_weight_cap` field defines the maximum contribution of review platform signals within the verification quality score for that category. Regulated and high-stakes categories use lower caps (≤0.10); lower-regulatory categories may use higher caps. Review weighting may not exceed the cap.

The `stage2_auth_tier` field defines the authentication tier required for Stage 2 data retrieval (Section 11.6). Values: `tier_1` (notification metadata only — used for low-sensitivity scope confirmation), `tier_2` (single-token authenticated retrieval — standard), `tier_3` (step-up authentication required — used for healthcare, sensitive legal matters, or any category where Stage 2 contains highly sensitive data).

### 4.5 Professional Tiers

Every professional in the ATAH registry belongs to one of two tiers. The tier determines what verification looks like. The protocol, matching engine, introduction lifecycle, and trusted partner model work identically for both.

**Credentialled professionals** hold a formal licence, certification, or accreditation from a recognised regulatory or professional body. Verification is against an authoritative regulatory source. The `licence` object is present in their profile.

**Established professionals** are recognised practitioners in fields where formal licensing does not exist but professional standing is real and verifiable. Verification is via trusted partner data. The `membership` object is present in their profile in place of licence.

### 4.6 Professional Identity Object — Credentialled Tier

Example: lawyer. The `licence` object is present. The `professional_tier` field is set to `credentialled`. `protocol_version` is included in every profile object and every API response.

```json
{
  "atah_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "professional_tier": "credentialled",
  "category": "lawyer",
  "name": "Jane Smith",
  "firm": "Smith Legal LLC",
  "licensed_jurisdictions": ["US-TX", "US-NY"],
  "service_locations": ["US-TX", "US-NY"],
  "physical_locations": ["Austin, TX"],
  "remote_service_available": true,
  "specialisms": ["commercial_contracts", "employment_law"],
  "matter_types": ["contract_review", "employment_dispute"],
  "matching_status": "active",
  "registration_route": "individual",
  "licence": {
    "number": "TX-BAR-12345",
    "issuing_body": "State Bar of Texas",
    "status": "active",
    "verified_at": "2026-04-01T00:00:00Z",
    "disciplinary_record": "none_found"
  },
  "experience": { "years_in_practice": 18, "years_in_specialism": 12 },
  "response_time": {
    "declared_hours": 24,
    "applies_to": "working_hours",
    "operating_timezone": "America/Chicago"
  },
  "engagement": {
    "model": "fixed_fee_and_hourly",
    "virtual": true,
    "languages": ["en", "es"],
    "availability": "accepting_new_matters"
  },
  "referral_network": {
    "inbound_relationship_count": 7,
    "inbound_relationship_quality_score": 0.84
  },
  "trusted_partner_data": [
    {
      "partner_id": "tp-statebar-tx-001",
      "partner_name": "State Bar of Texas",
      "partner_type": "regulatory_database",
      "data_points": {
        "licence_status": "active",
        "disciplinary_record": "none_found",
        "admission_date": "2008-09-15"
      },
      "last_updated": "2026-04-01T00:00:00Z",
      "trust_tier": "partner-verified"
    }
  ],
  "enhanced_verification": [],
  "data_conflicts": [],
  "ai_assistant_connected": true,
  "profile_completeness_score": 0.92,

  "_provenance": {
    "name": {"verification_status": "partner-verified", "source": "tp-statebar-tx-001", "as_of": "2026-04-01T00:00:00Z"},
    "firm": {"verification_status": "self-declared", "source": "professional", "as_of": "2026-03-15T00:00:00Z"},
    "licensed_jurisdictions": {"verification_status": "partner-verified", "source": "tp-statebar-tx-001", "as_of": "2026-04-01T00:00:00Z"},
    "specialisms": {"verification_status": "self-declared", "source": "professional", "as_of": "2026-03-15T00:00:00Z"},
    "licence.number": {"verification_status": "registry-verified", "source": "atah-registry", "as_of": "2026-04-01T00:00:00Z"},
    "licence.status": {"verification_status": "registry-verified", "source": "atah-registry", "as_of": "2026-04-01T00:00:00Z"},
    "licence.disciplinary_record": {"verification_status": "partner-verified", "source": "tp-statebar-tx-001", "as_of": "2026-04-01T00:00:00Z"}
  }
}
```

The `registration_route` field is required and has values `partner` or `individual`. See Section 10.4 for the full registration route model.

The location-related fields are split for clarity:

- `licensed_jurisdictions` — where the professional is legally authorised to practice
- `service_locations` — where the professional accepts clients (may differ from licensed jurisdictions for remote service)
- `physical_locations` — physical offices, if relevant
- `remote_service_available` — whether the professional offers remote service

Category-specific matching rules govern how each is used (e.g. for lawyers, `licensed_jurisdictions` is required for matching; for established-tier professionals it may not apply).

### 4.7 Professional Identity Object — Established Tier

Example: PR professional. The `membership` object is present in place of `licence`. The `trusted_partner_data` array is the primary source of verifiable standing. Provenance map omitted for brevity but follows the same pattern.

```json
{
  "atah_id": "atah-01HQXR8M2P3QRV5WTYZAB6CDEF",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "professional_tier": "established",
  "category": "pr_professional",
  "name": "Alex Johnson",
  "firm": "Johnson Communications Ltd",
  "service_locations": ["GB-LON"],
  "physical_locations": ["London, UK"],
  "remote_service_available": true,
  "specialisms": ["corporate_communications", "crisis_management"],
  "matching_status": "active",
  "registration_route": "partner",
  "membership": {
    "body": "Chartered Institute of Public Relations",
    "level": "Fellow",
    "member_number": "FCIPR-12345",
    "verified_at": "2026-04-01T00:00:00Z"
  },
  "trusted_partner_data": [
    {
      "partner_id": "tp-cipr-001",
      "partner_name": "Chartered Institute of Public Relations",
      "partner_type": "professional_body",
      "data_points": {
        "membership_status": "fellow",
        "member_since": "2011-03-01",
        "cpd_compliant": true,
        "disciplinary_record": "none_found"
      },
      "last_updated": "2026-04-01T00:00:00Z",
      "trust_tier": "partner-verified"
    }
  ],
  "enhanced_verification": [],
  "data_conflicts": [],
  "ai_assistant_connected": false,
  "profile_completeness_score": 0.88,
  "_provenance": { "...": "...as in Section 4.6..." }
}
```

### 4.8 Compliance-Pending Professional Example

```json
{
  "atah_id": "atah-01HQXR9N3Q4RSW6XUZBC7DEFGH",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "professional_tier": "credentialled",
  "category": "physician",
  "name": "Dr. Sarah Lee",
  "matching_status": "compliance-pending",
  "matching_status_reason": "Healthcare category pending HIPAA compliance annex sign-off",
  "profile_maintained": true,
  "registration_route": "individual"
}
```

### 4.9 Enhanced Verification Object

Enhanced verification provides a deeper, ongoing, independently-audited verification layer. It is optional. A professional may have multiple enhanced verifications (commissioned by their body, by their employer, or by themselves). It is administered by approved independent verifiers — see Section 5.7.

**Commercial neutrality:** Enhanced verification is a paid route to a stronger trust signal. ATAH scores enhanced verification under a public rubric available equally to all approved verifiers. No weight is assigned to which party paid for the verification or to the verifier's commercial relationship with the professional. Where this rubric is changed, the change is governance-approved and announced publicly. Paid verification is not a promoted-placement mechanism.

Each entry in the `enhanced_verification` array has the following structure:

```json
{
  "verification_id": "ev-01HQXRA0R5STXY7Z8ABC9DEFG",
  "verifier_id": "iv-globalcheck-001",
  "verifier_name": "GlobalCheck Independent Verification",
  "verifier_approved_by": "atah-governance-2026-q1",
  "commissioned_by": "self",
  "verified_dimensions": [
    { "dimension": "identity", "depth": "evidenced", "last_confirmed": "2026-03-15T00:00:00Z" },
    { "dimension": "qualifications", "depth": "evidenced", "last_confirmed": "2026-03-15T00:00:00Z" },
    { "dimension": "professional_standing", "depth": "audited", "last_confirmed": "2026-03-15T00:00:00Z" },
    { "dimension": "insurance", "depth": "evidenced", "last_confirmed": "2026-03-15T00:00:00Z" },
    { "dimension": "sanctions_screening", "depth": "confirmed", "last_confirmed": "2026-03-15T00:00:00Z" }
  ],
  "verified_at": "2026-03-15T00:00:00Z",
  "next_review_due": "2027-03-15T00:00:00Z",
  "current_status": "active",
  "breadth_score": 0.62,
  "depth_score": 0.78,
  "freshness_score": 0.95
}
```

**Field definitions:**

`verifier_id` and `verifier_name` — identify the approved independent verifier (see Section 5.7 for the verifier model).

`verifier_approved_by` — reference to the protocol governance approval that authorised this verifier.

`commissioned_by` — who commissioned the verification: `self`, `body:<partner_id>`, `employer`, or `other`. Recorded for transparency only; not used in matching weighting.

`verified_dimensions` — an array of dimensions that were verified. The seed dimension list is: `identity`, `qualifications`, `professional_standing`, `professional_history`, `insurance`, `references`, `client_feedback`, `compliance_obligations`, `sanctions_screening`, `custom`.

For `custom` dimensions the verifier must supply a structured `custom_dimension` block:

```json
{
  "dimension": "custom",
  "custom_dimension": {
    "custom_dimension_id": "cdim-globalcheck-pi-coverage-2026",
    "description": "Confirmation of professional indemnity insurance coverage at or above $5M",
    "evidence_type": "document_review",
    "scoring_mapping": "evidenced",
    "governance_approval_ref": "atah-governance-2026-q2-cdim-001",
    "category_scope": ["lawyer", "tax_planner"]
  },
  "depth": "evidenced",
  "last_confirmed": "2026-03-15T00:00:00Z"
}
```

The dimension list is extensible. New core dimensions are proposed by verifiers and ratified through governance review. Custom dimensions require governance approval before they may contribute to matching.

`depth` values for each dimension: `confirmed` (binary check passed), `evidenced` (documentary evidence on file), `audited` (active independent audit conducted), `not_assessed` (dimension was not assessed in this verification).

`current_status` values: `active`, `expired`, `revoked`, `under_review`. An expired verification remains in the profile but is surfaced as expired and does not contribute positively to matching. A revoked or under_review verification does not contribute to matching.

`breadth_score`, `depth_score`, `freshness_score` — computed by ATAH from the structured data. Verifiers cannot set these scores directly. The matching engine uses them as inputs to the verification quality component (Section 9).

- `breadth_score` is the proportion of dimensions relevant to the category that were assessed (the relevant dimension set per category is defined in `profession-categories.json`)
- `depth_score` is a weighted average of the depth values across assessed dimensions (`audited` = 1.0, `evidenced` = 0.75, `confirmed` = 0.5, `not_assessed` = 0)
- `freshness_score` is calculated from the time elapsed relative to `next_review_due`, with full score until 75% of the way to the review date and degrading linearly to zero at the review date

### 4.10 Consent Receipt Object

Every consent action in ATAH — consent to a query, to Stage 2 pre-handoff data submission, to Stage 3 contact release, to Type 2 referral participation, to Type 3 referral as a referred client — is captured as a structured consent receipt rather than a boolean flag. A boolean consent flag is not sufficient for ATAH conformance.

ATAH stores a hash of the consent receipt and its metadata. The full receipt is stored by the asserting platform (typically the AI platform that captured the consent). At dispute time, the platform produces the receipt and ATAH verifies the hash matches what was stored at consent time, confirming the receipt has not been altered.

```json
{
  "consent_receipt_id": "cr-01HQXRB1S6TUVZ8A9BCD0EFGHI",
  "schema_version": "0.8",
  "asserted": true,
  "asserted_by": {
    "actor_type": "ai_platform",
    "platform_id": "platform_openai_example",
    "client_id": "client_123"
  },
  "consumer_ref": "opaque-platform-user-ref",
  "scope": "stage_2_prehandoff_submission",
  "professional_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
  "handoff_id": "hf-01HQXRC2T7UVWZ9ABCDE1FGHIJ",
  "data_categories": [
    "matter_summary",
    "jurisdiction",
    "contact_preference"
  ],
  "consent_text_version": "stage2-conflict-check-v0.8",
  "captured_at": "2026-04-10T14:21:00Z",
  "expires_at": "2026-04-10T18:21:00Z",
  "revocation_supported": true,
  "revocation_endpoint": "https://platform.example/atah/revoke",
  "revocation_status": "active"
}
```

**Field definitions:**

`consent_receipt_id` — opaque identifier issued by ATAH at consent capture.

`asserted_by` — the actor responsible for capturing and asserting consent. For Type 1 consumer introductions this is the AI platform. For Type 2 referral proposals the asserter is the initiating professional (or their AI assistant acting on their behalf). For Type 3 referrals the asserter is the referring professional, with the underlying client consent represented by a separate nested receipt or attestation.

`consumer_ref` — opaque platform-side reference to the consumer. Not a personally identifiable identifier. ATAH does not learn the identity of the consumer from this field.

`scope` — what the consent covers. Defined values: `query_submission`, `stage_1_introduction`, `stage_2_prehandoff_submission`, `stage_3_contact_release`, `type_2_referral_participation`, `type_3_referred_client`, `outcome_reporting`. Each scope has corresponding required `data_categories` defined in the schema.

`data_categories` — list of categories of data covered by the consent. Defined values include `matter_summary`, `jurisdiction`, `contact_details`, `consumer_name`, `other_party_name`, `engagement_timeline`, `budget_range`, `client_summary`, and `contact_preference`. The set is closed and expanded through governance review.

`consent_text_version` — identifier for the canonical consent text the consumer was shown. Versions are published in the protocol documentation. Asserting platforms must show the consumer text matching this version.

`captured_at` and `expires_at` — RFC 3339 timestamps. Default expiry per scope is defined in schema.

`revocation_supported` and `revocation_endpoint` — whether the consumer can revoke this consent and the platform-side endpoint that supports revocation.

`revocation_status` — current revocation state. Values: `active`, `revoked`, `expired`. ATAH must check this status before any data action covered by the receipt.

When ATAH stores the receipt hash, the stored record is:

```json
{
  "consent_receipt_id": "cr-01HQXRB1S6TUVZ8A9BCD0EFGHI",
  "receipt_hash": "sha256:9f8a2c1b4d6e7a3c5b8d9f0e1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b",
  "asserted_by_platform_id": "platform_openai_example",
  "scope": "stage_2_prehandoff_submission",
  "professional_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
  "handoff_id": "hf-01HQXRC2T7UVWZ9ABCDE1FGHIJ",
  "captured_at": "2026-04-10T14:21:00Z",
  "expires_at": "2026-04-10T18:21:00Z",
  "revocation_status": "active"
}
```

ATAH never stores the `consumer_ref`, `data_categories`, or any consent text. The asserting platform retains the full receipt for its records.

The stored form is defined as a separate schema (`consent-receipt-stored.schema.json`) from the full receipt (`consent-receipt.schema.json`). Defining them as distinct schemas makes the privacy boundary structurally enforceable: build-time tests verify the stored-form schema does not declare any of the personal-data fields above.

### 4.11 Query Schema — Type 1 Consumer Introduction

```json
{
  "query_id": "q-01HQXRD3U8VWXAB0CDEF2GHIJK",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "query_type": "consumer_introduction",
  "requesting_agent": { "platform_id": "platform_openai_example", "client_id": "client_123" },
  "consent_receipt_id": "cr-01HQXRB1S6TUVZ8A9BCD0EFGHI",
  "matter": {
    "category": "pr_professional",
    "matter_type": "crisis_response",
    "location": "GB-LON",
    "urgency": "urgent"
  },
  "filters": {
    "professional_tier": "established",
    "partner_verified_only": true,
    "enhanced_verification_required": false,
    "max_response_time_hours": 4,
    "virtual": true
  },
  "shortlist_size": 3
}
```

The `consent_receipt_id` references a previously-issued consent receipt with scope `query_submission`. The asserting platform must have captured consumer consent before issuing the query.

The `enhanced_verification_required` filter, when true, returns only professionals with at least one active enhanced verification record.

### 4.12 Match Response Schema

Every match response includes `protocol_version`, a `deprecation_warning` field, and a `presentation_disclosure` object that AI platforms must surface to the user.

```json
{
  "query_id": "q-01HQXRD3U8VWXAB0CDEF2GHIJK",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "deprecation_warning": null,
  "presentation_disclosure": {
    "atah_is_not_recommending": true,
    "ranking_basis": ["relevance", "verification_quality", "availability_response", "profile_completeness", "inbound_referral_signal"],
    "commercial_weighting": false,
    "user_facing_disclosure": "ATAH does not recommend or endorse any professional. This is a provenance-visible shortlist based on the matter described and verification evidence available."
  },
  "result_count": 1,
  "results": [
    {
      "atah_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
      "professional_tier": "credentialled",
      "registration_route": "individual",
      "match_score": 0.94,
      "verification_status_summary": {
        "licence_status": "registry-verified",
        "disciplinary_record": "partner-verified",
        "pi_insurance": "partner-verified"
      },
      "enhanced_verification_summary": {
        "active_count": 1,
        "highest_combined_score": 0.78,
        "verifier_names": ["GlobalCheck Independent Verification"]
      },
      "review_signal_summary": {
        "platforms_contributing": ["Avvo"],
        "total_review_count": 47,
        "time_decayed_score": 0.86,
        "highest_review_platform_class": "verified_account"
      },
      "match_factors": {
        "relevance": 0.96,
        "verification_quality": 0.95,
        "verification_quality_subcomponents": {
          "credential_verification": 0.97,
          "enhanced_score": 0.78,
          "review_signals": 0.86,
          "scope_completeness": 1.0
        },
        "profile_completeness": 0.92,
        "availability_score": 1.0,
        "inbound_referral_signal": 0.84
      },
      "trusted_partner_data": ["...full payload..."],
      "_provenance": "...full provenance map for the profile..."
    }
  ]
}
```

The `presentation_disclosure` block must be present on every match response. AI platforms are expected to surface its content to end users at point of selection. This is a conformance requirement.

`enhanced_verification_summary` is present only if the professional has at least one active enhanced verification record. `review_signal_summary` is present only if at least one review platform is contributing data above the volume threshold. The full payload remains available via `GET /v1/professionals/:atah_id`.

### Verification confidence signal (v0.8.1, schema commitment for v0.8.2)

Match responses for regulated categories with single-source verification SHOULD carry a `verification_confidence` field separate from the verification-quality score components, surfacing to AI platforms whether the verification basis comes from a single source or is corroborated by multiple sources.

Defined values:

- `single_source_regulator` — verified against one authoritative regulatory source (e.g. one state bar lookup, one regulator register), with no corroborating partner data, no enhanced verification record, and no independent registry cross-reference.
- `single_source_membership` — verified through one professional membership body, with no corroborating regulatory source, no enhanced verification, and no independent cross-reference.
- `single_source_self_declared` — registered through individual self-registration with no partner verification (always returned with appropriate provenance labelling per the `_provenance` map).
- `multi_source_corroborated` — at least two independent sources agree on the verifiable fields (regulatory + membership; regulatory + enhanced verification; membership + independent verifier; or any other combination of independent sources).
- `multi_source_with_enhanced_verification` — multi-source corroboration that includes an active enhanced verification record from an approved independent verifier.

The signal is independent of the verification-quality score; a profile may have a high verification-quality score from a single high-strength source (regulator) yet remain `single_source_regulator` until corroborated. AI platforms presenting ATAH match results are expected to surface the confidence signal alongside the verification-quality information.

**v0.8.1 commitment.** The field name, value enumeration, and intended semantics are committed in this section. Schema implementation in `match-response.schema.json` and the matching engine logic that populates it land in v0.8.2 alongside the schema work tracked in the v0.8.2 / v0.9 ROADMAP. Until schema implementation lands, AI platforms inferring confidence from the `_provenance` map are not in conflict with this specification.

## 5. Trusted Partner Model

Trusted partners are organisations that contribute verified, maintained data about professionals to the ATAH registry. They are the core data quality mechanism for both professional tiers and the primary commercial model for the protocol.

### 5.1 Partner Tiers

- `partner-verified` — active, meeting freshness and quality thresholds, data surfaced in profiles and match results
- `probationary-partner` — recently connected or under review, data surfaced but flagged
- `suspended-partner` — failing to meet thresholds, data suppressed until remediated
- `revoked-partner` — partner status permanently removed, all contributed data removed from profiles within 30 days

### 5.2 Partner Verification Scope

Different partners verify different things. The protocol records what each partner actually verifies, rather than treating all partner-verified data as a single uniform claim.

Each partner record carries a `verification_scope` field describing what the partner verifies and how:

```json
{
  "verification_scope": {
    "verifies_membership": true,
    "verifies_active_licence": false,
    "verifies_disciplinary_record": true,
    "verifies_cpd_compliance": true,
    "verifies_insurance": false,
    "verification_method": "scheduled_data_push",
    "verification_frequency_days": 90,
    "supports_verifiable_credentials": false
  }
}
```

`verification_method` values: `api_lookup`, `scheduled_data_push`, `manual_audit`, `verifiable_credential_push`. `verification_frequency_days` is the partner's declared maximum interval between updates. `supports_verifiable_credentials` indicates whether the partner can submit data as W3C Verifiable Credentials.

The `verification_scope` is set when the partner is onboarded. It may be updated by the partner through governance review.

### 5.3 Partner Vetting Strength

Different partners apply different standards to membership or accreditation. A state bar association striking off a member for misconduct represents a different evidentiary weight from a body that admits anyone willing to pay the joining fee. The protocol records this distinction explicitly.

Each partner record carries a `vetting_strength` field with three possible values:

- `regulatory` — a regulator with statutory or quasi-statutory authority. Representative examples include state bar associations, NIPR, FINRA, state medical boards, and NAIC; equivalent regulatory bodies exist across other jurisdictions. Each is illustrative of the partner type, not a committed partner at v0.8 publication.
- `strong_membership` — a professional body with strict membership standards: chartered or fellow status, CPD requirements, active disciplinary processes. Representative examples include CIPR, ICF, PMI (for senior credentials), the CPCU Society, and the Big "I" (IIABA) and PIA (for independent insurance agents); equivalent bodies exist across other jurisdictions. Each is illustrative.
- `open_membership` — a body where membership is essentially open to anyone who pays the joining fee. Membership confirms identity but is a weaker signal of professional standing.

The `vetting_strength` is set when the partner is onboarded, by the protocol governance body, and may be revised through governance review. It is surfaced in match response metadata and is used as a weighting input in the matching engine's verification quality scoring (Section 9). It is not amendable by the partner unilaterally.

### 5.4 Partner Scoring

Every trusted partner has a continuously maintained partner score covering three components.

- Freshness score (0–1) — proportion of contributed data points updated within the partner's declared SLA
- Completeness score (0–1) — proportion of professionals within the partner's declared scope who have data in the registry
- Conflict rate (0–1, lower is better) — proportion of data points contributing to active conflicts with another source

Thresholds: above 0.80 — active in good standing. 0.60 to 0.80 — on notice, 30 days to remediate. Below 0.60 — suspended. Below 0.60 for more than 90 days — escalated to revocation review.

### 5.5 Data Freshness

Partners declare a data SLA on onboarding. ATAH monitors compliance continuously. Data points older than the declared SLA are flagged as stale and deprioritised. Partners push data updates via the trusted partner API. ATAH does not poll partners. If a partner has not pushed any update within their declared SLA window, their freshness score degrades automatically.

### 5.6 Conflict Detection and Resolution

The following fields are classified as **meaningful** — a conflict on any of these triggers immediate profile suppression from matching: `licence_status`, `disciplinary_record`, `matching_status`, `membership_status`. All other fields default to **benign** and are resolved automatically by recency.

- Benign conflict — resolved automatically by taking the most recently updated value. No profile impact.
- Meaningful conflict — profile immediately suppressed from matching (status `admin-suspended`). Professional notified. Both contributing partners notified. Conflict placed in manual review queue.

Resolution paths: one partner retracts; both update to consistent value; ATAH admin makes determination; or professional provides documentation. Suppression lifted only when conflict is resolved. All resolutions audit-logged.

### 5.7 Independent Verifiers

Independent verifiers are a specialised partner type that provide enhanced verification (Section 4.9). They are treated as trusted partners with two additional requirements: explicit governance approval and annual audit.

Each independent verifier is recorded with:

```json
{
  "verifier_id": "iv-globalcheck-001",
  "verifier_name": "GlobalCheck Independent Verification",
  "approved_by": "atah-governance-2026-q1",
  "approved_at": "2026-02-15T00:00:00Z",
  "categories_in_scope": ["lawyer", "tax_planner", "accountant"],
  "jurisdictions_in_scope": ["US", "UK", "EU"],
  "scope_tier": "global_scope",
  "verification_methodology_url": "https://globalcheck.example/methodology",
  "annual_audit_status": "passed",
  "annual_audit_last_completed": "2026-01-15T00:00:00Z",
  "vetting_strength_for_outputs": "regulatory"
}
```

**Approval requirement.** Independent verifiers require explicit approval by the protocol governance body before they may operate. Approval is granted on the basis of demonstrated methodology, independence, and audit capability.

**Vetting strength of verifier outputs.** Each approved verifier carries a `vetting_strength_for_outputs` value (`regulatory`, `strong_membership`, or `open_membership`) set at approval time. This is the vetting strength applied to enhanced verification records produced by this verifier when those records contribute to matching.

**Categories and jurisdictions in scope.** Each verifier declares the professional categories and jurisdictions they verify in. ATAH rejects verification submissions outside the declared scope.

**Scope tier.** Each verifier is classified by `scope_tier`: `narrow_scope`, `multi_scope`, or `global_scope`. The classification determines the fee tier the verifier sits in and does not influence matching weight.

**Annual audit requirement.** Each verifier is audited annually for methodology adherence. A failed audit suspends the verifier and flags all their active enhanced verification records as `under_review` until remediated.

**Independence requirement.** An independent verifier may not also operate as a professional body for any of the categories they verify in. A professional body wishing to commission enhanced verification for its members must engage an approved independent verifier separate from itself.

**Change of control.** A verifier acquired by another organisation must notify ATAH within 30 days. ATAH automatically suspends affected enhanced verification records pending an independence re-check. Public status updated.

**Commercial model.** Independent verifiers are commissioned by professional bodies, employers, or individual professionals. The fee for verification work is set by the verifier and paid by the commissioning party. ATAH does not receive a share of verification fees and does not set verifier prices.

Verifiers pay two distinct fees to ATAH:

- A **non-refundable assessment fee** paid at application. Covers methodology assessment, independence checks, conflict screening — payable whether or not approval is granted.
- An **annual audit fee** paid once approved, ongoing for as long as the verifier remains active. Covers the yearly methodology adherence audit.

Both fees are tiered by declared scope (`narrow_scope`, `multi_scope`, `global_scope`). The fee schedule is published transparently in the protocol's operational documentation. These fees are cost-recovery for governance work — they do not influence matching, do not affect approval likelihood beyond the assessment outcome itself, and do not create any financial incentive in a verifier's commercial success. ATAH publishes fee waiver criteria for public-interest organisations.

### 5.8 Review Platforms

Review platforms — representative examples include broad-spectrum review platforms (Google reviews, Trustpilot), legal review platforms (Avvo, Martindale-Hubbell), and healthcare review platforms (Healthgrades) — are a specialised partner class. The protocol treats review platform data as a distinct signal class with specific requirements. Each named platform is illustrative; none is a committed partner at v0.8 publication.

Each approved review platform is classified by `review_platform_class` based on anti-gaming controls and verification rigour:

- `verified_transaction` — reviews tied to verified completed engagements (highest weight)
- `verified_account` — reviews tied to authenticated accounts but not specific transactions
- `open_review` — reviews from any visitor (lowest weight)

**Anti-gaming attestation.** Each review platform must complete an anti-gaming attestation at registration covering: review provenance verification, sock-puppet detection, paid-review prevention, removal-on-fraud-detection process, transparency reporting cadence. Attestation is reviewed and classified by ATAH governance. Annual re-attestation required.

**Acceptance criteria, audit rights, suspension.** Review platform acceptance criteria are public. ATAH retains audit rights, including the ability to request transparency reports and conduct sample-based reviews. Failure to maintain attested controls results in classification downgrade or partner suspension. Public classification rationale is published.

**Volume thresholds and time decay.** Reviews contribute to matching only above a volume threshold (default 10 reviews). Reviews more than 24 months old contribute at reduced weight; reviews more than 60 months old do not contribute at all.

**Review weight cap by category.** The weight contribution of review signals to verification quality is capped per category in `profession-categories.json`. For regulated and high-stakes categories (lawyer, physician, financial_advisor, insurance_agent), the cap is ≤0.10. For lower-regulatory categories the cap may be higher, up to 0.20 in v0.8.

### 5.9 Disputes

Professionals may dispute partner data, enhanced verification records, or roll-up merges through the dispute API. Dispute records are created synchronously and assigned to the relevant partner, verifier, or admin queue. The dispute flow is described in Section 7.

### 5.10 Concern Flags

Concern flags are created when an introduction outcome includes `consumer-reported-concern`. Concern flag handling:

- Concern flags are **admin-only visibility**. They are not published, not surfaced in match results, and not used in matching weighting.
- The professional named in a concern flag is **notified** when a flag is created against them and has a **right of reply** within 30 days.
- A single flag does not result in profile suppression. **Patterns of flags** (defined operationally in admin documentation) trigger admin review and may result in escalation to the relevant regulatory or professional body.
- ATAH does not adjudicate professional conduct. Where escalation is appropriate, ATAH notifies the relevant regulator or professional body and records the notification.
- Flags submitted in apparent bad faith (rapid succession from same consumer, contradictory content) are flagged for review and may be discarded.

This protects against both consumer harm (from bad actors gaming the flag system) and professional harm (from defamation through unverified flags).

## 6. Introduction Lifecycle

ATAH supports three introduction types, each with its own state machine. All three share the same data governance protocol (Section 11).

### 6.1 Type 1 — Consumer Introduction

```
query → shortlist-returned → introduction-initiated
  → stage-1-pending → stage-1-accepted | stage-1-declined | stage-1-timeout
  (if stage-1-accepted)
  → pending-stage-2 → stage-2-cleared | stage-2-blocked | stage-2-timeout
  (if stage-2-cleared OR category requires no pre-handoff check)
  → pending-stage-3 → stage-3-released | stage-3-cancelled
  → introduction-complete → outcome-reported → closed
```

Consumer cancellation states reachable from any pending state: `consumer-cancelled`, `consent-revoked`, `data-deleted`.

Each state transition is atomic and logged. The introduction lifecycle manager issues a fresh `handoff_access_token` for each authenticated state-changing call (Section 11.5).

### 6.2 Pre-Handoff Check Types (Stage 2)

The Stage 2 pre-handoff check is category-flexible. The check type is determined by the category's `default_pre_handoff_check` setting, with optional override by the AI agent:

- `conflict_check` — used for lawyers and other categories where conflict screening is required. Required `prehandoff_data`: `consumer_name`, optional `other_party_name`.
- `scope_confirmation` — used for PR professionals, tax planners, and other established-tier categories. Required: `consumer_name`, `scope_summary`.
- `capacity_confirmation` — used where availability is the gating concern. Required: `consumer_name`, `engagement_timeline`.
- `fee_alignment` — used where fee fit is a primary concern. Required: `consumer_name`, `budget_range`.
- `none` — categories where no pre-handoff check is required. The lifecycle skips Stage 2 and goes directly from `stage-1-accepted` to `pending-stage-3`.

The default pre-handoff check for each category is set in the profession category metadata. The consumer's AI agent may override the default if appropriate for the matter; the override is recorded on the introduction record.

A `stage-2-blocked` outcome means the professional has determined the introduction cannot proceed for a category-specific reason. The consumer's AI agent may reroute to another professional or initiate a new search.

### 6.3 Simultaneous Introduction Rules

These rules govern concurrent introductions across all three types. Hard constraints, not advisory.

Rules for professionals:

- A professional may receive any number of simultaneous Stage 1 introductions.
- A professional may only be in one active Stage 2 introduction at a time. Concurrent attempts are held at `stage-1-accepted`.
- A professional may only be in one active Stage 3 introduction at a time, by the same rule.
- Once Stage 2 on introduction A resolves, introduction B automatically advances. No manual action required.

This rule applies uniformly across all categories in v0.8. Per-category configurability is a candidate for v0.9.

Rules for consumers:

- Multiple simultaneous active introductions to different professionals — no cap.
- Not more than one active introduction to the same professional at the same time. A second attempt returns 409 Conflict with the existing `handoff_id`.
- Each introduction has its own independent `handoff_id`.

Rules for the shortlist:

- Default `find_professional` shortlist size is 1. Configurable up by the querying AI agent. Maximum 5.
- ATAH does not enforce a single-introduction-per-query rule.

### 6.4 Type 2 — Referral Partner Establishment Schema and States

Type 2 establishes a referral relationship between two professionals (or their AI assistants) for future client referrals. Type 2 does not transmit client personal data; it is a professional-to-professional relationship establishment.

```json
{
  "proposal_id": "proposal-01HQXRE4V9WXYABCDE3FGHIJK",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "introduction_type": "type_2_referral_proposal",
  "initiating_professional_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
  "target_professional_id": "atah-01HQXRF5W0XYZABCDE4GHIJKL",
  "consent_receipt_id": "cr-01HQXRG6X1YZAB0CDEF5HIJKLM",
  "proposed_relationship": {
    "category_alignment": ["lawyer", "tax_planner"],
    "jurisdiction_overlap": ["US-TX"],
    "rationale": "Cross-referral relationship for clients with combined legal and tax planning needs"
  },
  "current_state": "pending-mutual-opt-in",
  "expires_at": "2026-04-17T14:21:00Z"
}
```

State machine:

```
proposal-created → pending-mutual-opt-in
  → partner-1-accepted
    → partner-2-accepted → relationship-established
    → partner-2-declined → relationship-declined
  → partner-1-declined → relationship-declined
  → expired (no response in 7 days) → relationship-declined
```

### 6.5 Type 3 — Professional-Initiated Client Referral Schema and States

Type 3 occurs when Professional A refers their existing client to Professional B. The client's contact details are transmitted; client consent is required.

Type 3 is the most sensitive introduction type because it transmits PII initiated by a professional rather than by the consumer themselves. It uses one of two consent paths:

**Path 1 — Client consent receipt.** The client has used ATAH or a connected AI to consent to the referral. A consent receipt with scope `type_3_referred_client` is included in the referral.

**Path 2 — Professional attestation of client consent.** The referring professional attests that they have obtained client consent through professional practice (e.g. a signed client engagement letter authorising referrals). The attestation is structured:

```json
{
  "client_consent_attestation": {
    "attestation_id": "att-01HQXRH7Y2ZABCDEFG6IJKLMN",
    "attesting_professional_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
    "attestation_text_version": "type3-attestation-v0.8",
    "captured_at": "2026-04-10T14:00:00Z",
    "evidence_basis": "signed_engagement_letter",
    "data_retention_compliant": true
  }
}
```

Attestations carry the same legal-assertion model as platform-asserted consumer consent: the attesting professional takes responsibility for client consent capture and retention. ATAH logs the attestation and the professional identity.

Type 3 referral object:

```json
{
  "referral_id": "ref-01HQXRJ8Z3ABCDEFGHIJKLMNO0",
  "schema_version": "0.8",
  "protocol_version": "0.8",
  "introduction_type": "type_3_client_referral",
  "referring_professional_id": "atah-01HQXR7K9NBVZ8M3PXFG2YT5WA",
  "target_professional_id": "atah-01HQXRF5W0XYZABCDE4GHIJKL",
  "consent": {
    "type": "client_consent_receipt",
    "consent_receipt_id": "cr-01HQXRK9A4BCDEFGHIJKLMNOPQ"
  },
  "client_data_vault_ref": "vault-01HQXRL0B5CDEFGHIJKLMNOPQR",
  "current_state": "referral-initiated"
}
```

The actual client contact details are stored in the transient encrypted vault (Section 11.6) and referenced by `client_data_vault_ref`. Client data is not present in the referral object itself.

State machine:

```
referral-initiated → referral-delivered
  → contact-details-deleted (immediate after retrieval — Section 11)
    → referral-recorded (anonymised event logged)
```

## 7. API Endpoints

### 7.1 Versioning and Backward Compatibility

All REST API endpoints include the major API version in the path (`/v1/`). The API version is independent of the protocol version: the API surface is more stable than protocol semantics during the v0.x phase. The `protocol_version` field in request and response bodies carries the full semantic protocol version (e.g. `0.8`).

If a client supplies a `protocol_version` in the request body that differs from the `Accept-Version` header, the request body's `protocol_version` is authoritative.

Version negotiation:

- Clients may include an `Accept-Version` header specifying the protocol version they expect (e.g. `Accept-Version: 0.8`). If absent, the latest protocol version is assumed.
- Responses always include a `protocol_version` field reflecting the version used.
- When a client requests a deprecated version, the response includes a `Deprecation` header with the sunset date and a `deprecation_warning` field in the response body.

Deprecation policy:

- Breaking changes (field removals, type changes, behaviour changes) require a new minor protocol version increment (e.g. `0.8` → `0.9`).
- Non-breaking additions do not require a version increment.
- A deprecated protocol version is supported for a minimum of 90 days after the new version is published.
- After the sunset date, requests to the deprecated version receive 410 Gone with a migration guide URL.

### 7.2 Idempotency

All mutating endpoints require an `Idempotency-Key` header. The server returns the original result for repeated keys within a 24-hour retention window.

Required on:

- `POST /v1/introductions`
- `POST /v1/introductions/:handoff_id/stage-2`
- `POST /v1/introductions/:handoff_id/stage-3`
- `POST /v1/introductions/:handoff_id/outcome`
- `POST /v1/introductions/:handoff_id/cancel`
- `POST /v1/introductions/:handoff_id/revoke-consent`
- `POST /v1/referral-proposals`
- `POST /v1/referral-proposals/:proposal_id/respond`
- `POST /v1/partners/:partner_id/data` (per-batch)
- `POST /v1/verifiers/:verifier_id/verifications`

Optional on `POST /v1/query` (queries are read-like but stateful enough to support idempotency).

### 7.3 Authorisation Matrix

| Actor | Endpoint | Scope | Object-level constraint |
|---|---|---|---|
| AI platform | `POST /v1/query` | `find` | platform/client must be authenticated |
| AI platform | `POST /v1/introductions` | `introductions:create` | only under authenticated platform/client |
| AI platform | `GET /v1/introductions/:handoff_id` | `atah:introductions:read` | platform/client matches asserter, OR consumer_ref matches handoff record |
| AI platform | `POST /v1/introductions/:handoff_id/stage-*` | `atah:introductions:write` | platform/client matches original asserter, with valid handoff_access_token |
| AI platform | `POST /v1/introductions/:handoff_id/outcome` | `atah:outcomes:write` | platform/client matches original asserter or target professional |
| AI platform | `POST /v1/introductions/:handoff_id/cancel` | `atah:introductions:write` | platform/client matches original asserter |
| AI platform | `POST /v1/introductions/:handoff_id/revoke-consent` | `atah:introductions:write` | platform/client matches original asserter |
| Professional | `GET /v1/professionals/me` | `atah:professionals:self_read` | only own profile |
| Professional | `PUT /v1/professionals/me` | `atah:professionals:self_write` | only own profile, only fields not partner-managed |
| Professional | `GET /v1/introductions/received` | `atah:introductions:read` | only introductions addressed to this professional |
| Professional | `POST /v1/introductions/:handoff_id/respond` | `atah:introductions:write` | only introductions addressed to this professional |
| Partner | `POST /v1/partners/:partner_id/data` | `atah:partner:data.write` | only own `partner_id` |
| Partner | `POST /v1/partners/:partner_id/retract` | `atah:partner:data.write` | only own `partner_id` |
| Verifier | `POST /v1/verifiers/:verifier_id/verifications` | `atah:verifier:verification.write` | only own `verifier_id`, only approved categories |
| Admin | `POST /v1/admin/professionals/:atah_id/suspend` | `atah:admin` | governance/admin role only |
| Admin | `POST /v1/admin/concern-flags/:flag_id/review` | `atah:admin` | governance/admin role only |

### 7.4 External API — Consumer and AI Agent Facing

```
POST   /v1/query                                       Find professionals. Returns ranked shortlist.
GET    /v1/professionals/:atah_id                      Get public profile.
POST   /v1/introductions                               Initiate new introduction (Type 1 or Type 3).
GET    /v1/introductions/:handoff_id                   Get current state. Stateless caller support per Section 11.5.
POST   /v1/introductions/:handoff_id/stage-2           Submit Stage 2 pre-handoff check data.
POST   /v1/introductions/:handoff_id/stage-3           Submit Stage 3 consent and release.
POST   /v1/introductions/:handoff_id/outcome           Submit outcome report.
POST   /v1/introductions/:handoff_id/cancel            Consumer cancellation of introduction.
POST   /v1/introductions/:handoff_id/revoke-consent    Consumer revocation of consent.
POST   /v1/referral-proposals                          Initiate Type 2 referral partner proposal.
GET    /v1/referral-proposals/:proposal_id             Get proposal status.
POST   /v1/webhooks/register                           Register webhook endpoint (Phase 2).
GET    /.well-known/atah                               Implementation discovery (capabilities, version, MCP endpoint).
GET    /v1/capabilities                                Detailed implementation capabilities.
```

### 7.5 Professional API

```
POST   /v1/professionals/register
GET    /v1/professionals/me
PUT    /v1/professionals/me
POST   /v1/professionals/me/verify-document
POST   /v1/professionals/me/mcp-connect
GET    /v1/introductions/received
POST   /v1/introductions/:handoff_id/respond
GET    /v1/referral-network
POST   /v1/referral-proposals/:proposal_id/respond
POST   /v1/professionals/me/disputes
GET    /v1/professionals/me/disputes
GET    /v1/professionals/me/enhanced-verifications
GET    /v1/professionals/me/concern-flags          List concern flags against this professional
POST   /v1/professionals/me/concern-flags/:flag_id/reply   Submit right-of-reply response
```

`POST /v1/professionals/register` requires `acknowledged_rollup_terms: true` for individual self-registration.

### 7.6 Partner API

```
POST   /v1/partners/register
GET    /v1/partners/:partner_id
PUT    /v1/partners/:partner_id
POST   /v1/partners/:partner_id/data
GET    /v1/partners/:partner_id/score
GET    /v1/partners/:partner_id/conflicts
POST   /v1/partners/:partner_id/retract
```

The partner data push schema (`partner-data-push.schema.json`) supports:

- `batch_id` (required, idempotency key)
- `schema_version`
- `submitted_at`
- `partner_signature` (signed JWT or HMAC, applied at the batch level)
- `partial_failure_policy`: `all_or_nothing` | `accept_valid`
- `records[]`, where each record has:
  - `operation`: `upsert | retract | replace`
  - `record_type`: identifies the payload shape — `professional_profile`, `enhanced_verification`, or `verification_data_point`
  - `record_id`
  - `source_last_updated`
  - `payload`: the record content, with shape determined by `record_type`
- Returns: validation report with accepted/rejected counts and per-record outcomes

The detailed per-record payload structure is specified in the schema design (see `partner-data-push.schema.json`).

### 7.7 Independent Verifier API

```
POST   /v1/verifiers/register                          (admin only — verifier approval)
GET    /v1/verifiers/:verifier_id
POST   /v1/verifiers/:verifier_id/verifications
PUT    /v1/verifiers/:verifier_id/verifications/:id
POST   /v1/verifiers/:verifier_id/revoke/:id
GET    /v1/verifiers/:verifier_id/audit-status
```

### 7.8 Admin API

```
GET    /v1/admin/professionals
POST   /v1/admin/professionals/:atah_id/verify
POST   /v1/admin/professionals/:atah_id/suspend
GET    /v1/admin/introductions
GET    /v1/admin/metrics
GET    /v1/admin/disputes
POST   /v1/admin/disputes/:dispute_id/resolve
GET    /v1/admin/concern-flags
POST   /v1/admin/concern-flags/:flag_id/review
GET    /v1/admin/rollup-queue
POST   /v1/admin/rollup-queue/:match_id/decide
GET    /v1/admin/partners
POST   /v1/admin/partners/:partner_id/suspend
POST   /v1/admin/partners/:partner_id/resolve-conflict/:conflict_id
```

### 7.9 Error Model

All errors return a structured response:

```json
{
  "error": {
    "code": "consent_receipt_invalid",
    "message": "The consent receipt referenced has expired",
    "request_id": "req-01HQXRM1C6DEFGHIJKLMNOPQRS",
    "details": { "consent_receipt_id": "cr-...", "expired_at": "2026-04-10T18:21:00Z" }
  }
}
```

Defined error codes are listed in `error.schema.json`. Error responses include a `request_id` for correlation with audit logs.

## 8. MCP Server Specification

The ATAH MCP server exposes tools to any MCP-compatible AI agent. MCP server endpoint: `/mcp`.

### 8.1 MCP Authentication

The ATAH MCP server conforms to the current MCP authorisation specification at the time of implementation. The v0.8 baseline requires:

- **OAuth 2.1-compatible authorisation.** Authorization Code flow with PKCE for interactive flows; client credentials grant for server-to-server.
- **Protected Resource Metadata** at `/.well-known/oauth-protected-resource` declaring the authorisation server, scopes, and resource indicators.
- **Authorization Server Metadata** discoverable via `/.well-known/oauth-authorization-server` or OIDC discovery (`/.well-known/openid-configuration`).
- **Resource indicators (RFC 8707).** Tokens must declare the ATAH MCP endpoint as the resource. Tokens issued for other resources are rejected.
- **Audience validation.** ATAH validates the `aud` claim on every token.
- **Explicit scopes.** ATAH defines scopes including `atah:find`, `atah:introductions:create`, `atah:introductions:read`, `atah:introductions:write`, and `atah:outcomes:write`. Consent revocation is authorised under `atah:introductions:write`. Tokens must declare the minimum scopes required.
- **Short-lived access tokens.** Default lifetime 1 hour. Refresh tokens supported with rotation.
- **Dynamic Client Registration** supported per RFC 7591 for AI platforms wishing to integrate without manual onboarding.

Manual onboarding remains available at `atahprotocol.org/developers` for platforms preferring static client registration.

Where MCP authorisation guidance evolves between v0.8 publication and the actual implementation, ATAH conforms to the current MCP authorisation specification. Implementers should treat this section as defining minimum requirements that may be tightened by current MCP guidance, never weakened.

### 8.2 MCP Tools

v0.8 defines nine MCP tools. `cancel_introduction`, `revoke_consent`, and `get_consent_requirements` are new in v0.8; `submit_stage2_consent` and `submit_stage3_consent` were renamed to `submit_stage2_data` and `submit_stage3_data` respectively.

Full structured input and output schemas are published in `mcp-tools.json`. The summaries below describe tool purpose and the key inputs.

**Tool safety metadata.** Each tool definition in `mcp-tools.json` declares structured safety metadata for AI platforms and MCP clients to surface to end users:

- `is_read_only` — true for tools that do not change state on the registry side (`find_professional`, `check_introduction_status`, `get_consent_requirements`); false for state-changing tools.
- `transmits_personal_data` — true for tools that carry consumer personal data into the transient vault (`submit_stage2_data`, `submit_stage3_data`); false otherwise. `find_professional` is false because queries carry only matter type, jurisdiction, and category — not consumer-identifying data.
- `requires_explicit_user_approval` — true for tools that release consumer data, change state on the consumer's behalf, or commit consent (`initiate_introduction`, `submit_stage2_data`, `submit_stage3_data`, `cancel_introduction`, `revoke_consent`); false for read-only tools.
- `state_change_scope` — `none` (read-only), `handoff` (changes a single handoff's state), or `consent` (creates or revokes a consent receipt).

These flags are advisory metadata for client-side surfacing, not authorisation enforcement. Authorisation rules are enforced server-side per Section 7.3 and Section 11.5 regardless of the metadata. The metadata exists so clients can show the user appropriate "you are about to share data" or "this will change a handoff's state" prompts before invocation.

#### Tool: `find_professional`

Find a credentialled or established professional for a matter requiring human expertise. Returns a ranked shortlist with full trusted partner data payload, `presentation_disclosure`, and `protocol_version`.

- Required inputs: `matter` (object containing `category`, `matter_type`, `location`, and `urgency`) and `consent_receipt_id`
- Optional inputs: `professional_tier`, additional `filters`, `shortlist_size`

Professionals with `matching_status` of `compliance-pending`, `regulatory-suspended`, or `admin-suspended` are excluded from all results regardless of other filters.

#### Tool: `initiate_introduction`

Initiate a structured introduction. Returns a `handoff_id` and `handoff_access_token` for subsequent state-changing calls. For Type 3, includes referral-specific consent payload.

- Required inputs: `atah_id`, `matter_type`, `location_or_jurisdiction`, `consent_receipt_id`
- Returns: `handoff_id`, `handoff_access_token`, `current_state`, `protocol_version`, `pre_handoff_check_required`
- Returns 409 Conflict if the consumer already has an active introduction to this professional.

#### Tool: `check_introduction_status`

Check the current status of an active introduction. Available to the original asserter platform and to authenticated AI platforms presenting a consumer reference matching the handoff record (Section 11.5).

- Required inputs: `handoff_id`
- Optional inputs: `consumer_ref` (for cross-platform status check)
- Returns: `handoff_id`, `current_state`, `state_history`, `protocol_version`, `last_updated`

#### Tool: `submit_stage2_data`

Submit Stage 2 pre-handoff check data after the professional has accepted Stage 1. Renamed from `submit_stage2_consent` because this tool transmits both consent and data.

- Required inputs: `handoff_id`, `handoff_access_token`, `prehandoff_data`, `consent_receipt_id`
- The structure of `prehandoff_data` depends on the pre-handoff check type required by the category.
- Personal data is stored in the transient encrypted vault per Section 11.6; the professional retrieves it through authenticated retrieval. The data does not leave the vault via SMS or email.

#### Tool: `submit_stage3_data`

Release consumer contact details to the selected professional after explicit Stage 3 consent. Renamed from `submit_stage3_consent`.

- Required inputs: `handoff_id`, `handoff_access_token`, `consumer_contact`, `consent_receipt_id`
- Optional inputs: `matter_summary`
- Personal data is stored in the transient encrypted vault and retrieved by the professional through authenticated retrieval.

#### Tool: `cancel_introduction`

Consumer cancellation of an active introduction. Triggers data deletion in the transient vault and notifies the professional that the introduction has been cancelled.

- Required inputs: `handoff_id`, `handoff_access_token`, `cancellation_reason` (optional)
- Returns: confirmation and final state `consumer-cancelled`

#### Tool: `revoke_consent`

Consumer revocation of a previously-issued consent receipt. Triggers data deletion in the transient vault for any data covered by the revoked receipt and prevents further actions under that receipt.

- Required inputs: `consent_receipt_id`, `handoff_access_token` (if associated with a handoff)
- Returns: confirmation and updated `revocation_status`

#### Tool: `get_consent_requirements`

Retrieve the consent requirements for a specific scope and category. Allows AI platforms to determine what consent text and data categories must be captured before issuing a consent receipt.

- Required inputs: `scope`, `category`
- Returns: required `data_categories`, `consent_text_version`, `consent_text` (canonical wording), `default_expiry_seconds`

#### Tool: `report_introduction_outcome`

Report the outcome of an introduction when it concludes.

- Required inputs: `handoff_id`, `handoff_access_token`, `outcome_code`

Outcome codes: `introduction-successful`, `professional-declined-conflict`, `professional-declined-scope`, `professional-declined-capacity`, `professional-declined-fee`, `professional-declined-other`, `consumer-chose-alternative`, `matter-not-proceeding`, `no-response-from-consumer`, `consumer-reported-concern`, `consumer-cancelled`, `consent-revoked`.

On `consumer-reported-concern`: routes to the concern flag mechanism per Section 5.10. ATAH does not adjudicate; escalation to the relevant regulatory or professional body is the route for serious matters.

## 9. Matching Engine Design

The matching engine runs entirely on the registry's servers. AI agents receive a ranked shortlist with the full trusted partner data payload, the `presentation_disclosure` block, and per-sub-component contribution metadata.

Professionals with `matching_status` of `compliance-pending`, `regulatory-suspended`, or `admin-suspended` are excluded at Step 1 regardless of any other filter.

### 9.1 Matching Pipeline

**Step 1 — Hard exclusion filters.** Wrong location or jurisdiction, wrong category, not accepting, response time exceeds urgency threshold, profile suppressed due to meaningful data conflict, `matching_status` not `active`.

**Step 2 — Relevance scoring (0–1).** Exact match on matter type, specialism tags, matter complexity.

**Step 3 — Verification quality score (0–1).** Four sub-components combine to produce the final verification quality score.

**Sub-component A: Credential and standing verification.** For credentialled professionals: licence verification status, PI insurance, disciplinary record. For established professionals: partner-verified membership status, CPD compliance, disciplinary record via partner data. Registry-verified, partner-verified, and vc-verified data treated as equivalent in evidentiary type, but weighted by the contributing partner's `vetting_strength`. Self-declared at lower weight throughout.

**Sub-component B: Enhanced verification.** Weighted average of `breadth_score`, `depth_score`, `freshness_score` of active enhanced verification records. Zero for professionals without enhanced verification.

**Sub-component C: Review platform signals.** Time-decayed average review score, weighted by `review_platform_class`. Capped per category by `review_signal_weight_cap` in `profession-categories.json`. For high-stakes regulated categories the cap is ≤0.10.

**Sub-component D: Profile data completeness within verification scope.** Split into:

- `declared_scope_completeness` — has the professional populated verification-relevant fields
- `verified_scope_completeness` — have those fields been verified (registry, partner, or VC)

Only `verified_scope_completeness` materially affects this sub-component. Declared completeness contributes minimally.

The four sub-components combine into the verification quality score, capped at 1.0:

```
verification_quality = min(1.0,
  credential_verification × w_a +
  enhanced_score × w_b +
  review_signals × w_c +
  scope_completeness × w_d
)
```

Default weights: `w_a = 0.50`, `w_b = 0.20`, `w_c = 0.20` (subject to category cap), `w_d = 0.10`. All four weights are configurable per category.

**Step 4 — Availability and response time score (0–1).**

**Step 5 — Profile completeness score (0–1).**

**Step 6 — Inbound referral signal score (0–1).**

Discounted during early operation. Threshold: fewer than 20% of active professionals in the registry have at least one inbound referral relationship. Below threshold, the inbound referral weight redistributes proportionally.

Anti-gaming controls applied to inbound referrals:

- Reciprocal referrals capped (closed pairs and triangles discounted)
- Time decay applied (older referrals weighted less)
- Referrals weighted by referrer verification quality
- Dense clusters (Sybil-like patterns) flagged for review and discounted pending review
- Evidence of actual successful referral outcomes (Type 2 relationship-established or Type 3 referral-recorded events) required for full weight
- Referral provenance exposed in match metadata

Sybil/ring detection more sophisticated than the above is deferred to v0.9.

**Step 7 — Final score:**

```
final_score = (
  relevance               × w_relevance +
  verification_quality    × w_verification +
  availability_response   × w_availability +
  profile_completeness    × w_completeness +
  inbound_referral_signal × w_referral
)
```

Weights default: relevance 0.30, verification 0.25, availability 0.20, completeness 0.10, referral 0.15. Per-category overrides via `matching_weight_profile` in `profession-categories.json`. Categories with no override use the default profile.

**Commercial neutrality (Section 9.2).** No weight may be assigned to:

- any commercial relationship between ATAH and a partner, verifier, review platform, or AI platform
- trusted partner status itself (only the data contributed by partners influences ranking, and only through verification quality)
- which party paid for an enhanced verification (only the structured verification depth, breadth, freshness influence ranking)
- which review platform contributed data beyond the published `review_platform_class`
- the registration route (partner vs. individual paid annual fee). A paid individual listing and a partner-route listing receive identical matching treatment.

Where paid services generate verification evidence, that evidence is scored under the same public rubric available to all approved sources.

## 10. Verification Architecture

### 10.1 Verification by Professional Tier

**Credentialled professionals — insurance agents (United States):**

Authoritative US insurance licensing data — NIPR is a representative example of the type of source that can support automated verification across US states and territories for licensed insurance producers, where the relevant partner agreement is in place. Such integrations verify licensing and appointment data where available; category fit (lines of authority, carrier appointments, P&C focus, farm/commercial/personal specialisation) still depends on declared and partner-verified specialisms, product lines, and engagement scope. Category-specific metadata is captured in `profession-categories.json` and via partner data from professional bodies (representative examples include the Big "I", PIA, CPCU Society, and NAIC; equivalent bodies exist across other jurisdictions).

**Credentialled professionals — lawyers (United States):**

State bar databases — operational scope is limited to jurisdictions where partner agreements exist. Where state bar APIs are accessible under partner agreement, automated verification runs directly. Where only lookup tools exist, a scraping layer may be used subject to the relevant terms. Where neither is reliable, manual review within 24 hours applies.

Where verification routes are not yet in place, profiles are self-declared and clearly labelled as such. No false verification claims are made.

The current operational scope for priority states is published in the protocol's documentation, not embedded in the spec.

**Credentialled professionals — other categories:** verification approach is category-specific and depends on the availability of regulatory APIs or trusted partner data. Each category's verification approach is documented in `profession-categories.json`.

**Established professionals:** verification is via trusted partner data. W3C Verifiable Credentials issued by professional bodies are accepted where supported.

**Compliance-pending categories:** profiles created and maintained normally, excluded from all match results until the compliance annex is complete.

### 10.2 Verification Job Runner and Batching Strategy

**On registration:** verification triggered immediately for credentialled professionals where automated verification is available. Result within 24 hours.

**Scheduled re-verification (90-day cycle):** processed as a rolling daily batch. Batch size = total active credentialled profiles ÷ 90, rounded up. Distributes load evenly across the cycle.

**Concurrency limit:** maximum 10 concurrent external verification calls. Configurable. Calls beyond the limit are queued in Redis.

**Rate-limit handling:** on HTTP 429, exponential backoff starting at 60 seconds, doubling on each retry, maximum 5 retries. If exhausted, profile marked `verification-deferred` and retried in the next day's batch. Profile remains active in matching but scores lower on verification quality. After 7 days beyond due date, flagged for admin review.

**Query-triggered verification:** if last verification was more than 30 days ago, re-verification is triggered before the result is included in a shortlist. If the external source is unavailable, the query proceeds with existing verification status; deferred verification is queued.

**Adverse status detection:** if verification returns an adverse status, the profile is immediately suspended (`regulatory-suspended`) and the professional is notified. Never deferred.

**Jurisdiction change events:** when a professional's licensed jurisdictions change (verified through partner data or self-declared with re-verification), a jurisdiction-change event is logged. Active introductions in the affected jurisdiction are flagged for AI platform attention but not automatically cancelled. A grace period of 30 days applies to transitioning practice before historical-authority labels are applied. Detailed jurisdiction-change handling is operational documentation.

### 10.3 Registry Integrity and Fraud Prevention

Any professional may register with a self-declared profile via the individual self-registration route. The protocol is open by design. Basic credibility checks at registration: email verification, name and firm findable in at least one publicly accessible source, flagging of profiles where multiple data points appear very recently created or where review histories show anomalous patterns.

Flagged profiles are held as `pending-integrity-review` and queued for review before becoming active in matching. In a future phase, this layer will be extended with more sophisticated signal analysis.

### 10.4 Registration Routes

ATAH supports two routes for a professional to be present in the registry: the partner route and individual self-registration. Both routes coexist permanently.

#### Partner route

When a trusted partner joins ATAH for a professional category and contributes member data, the professionals it covers are surfaced through that partner's data. Their `atah_id` is created automatically; `registration_route` is recorded as `partner`.

A partner-route professional may claim their profile via verified email match against partner records and gain editing access for fields not covered by partner data. Fields covered by partner data are partner-managed and not editable directly — disputes go through the dispute flow.

#### Individual self-registration

A professional may register directly via `POST /v1/professionals/register`. `registration_route` recorded as `individual`. Profile data is self-declared and verified subsequently per Section 10.

Individual self-registration requires `acknowledged_rollup_terms: true` in the registration request.

#### Roll-up logic

When a trusted partner joins ATAH for a category, ATAH compares the partner's contributed member roster against existing individual self-registered profiles in the same category. Matching criteria:

- Name match (allowing common variants)
- Jurisdiction match
- One corroborating data point: firm, verified email, or licence number

A match meeting all three: high-confidence merge — automatic, professional notified.

A match meeting two of three: low-confidence — held in review queue. Profile flagged `pending-rollup`. **Pre-merge notification.** For high-risk categories (regulated tier), the professional is notified before the merge completes and given a 7-day objection window. Low-risk categories may merge without pre-notification but the professional is notified after.

A match meeting one or fewer: not a match.

After roll-up, the professional may dispute the merge through the standard dispute flow if incorrect. Disputed fields are temporarily suppressed.

**Roll-up acknowledgement at registration.** Individual self-registered profiles MUST set `acknowledged_rollup_terms: true` at registration to record that the professional has been informed of the roll-up process, the 7-day pre-merge notification window, and the right to object as described in this section. The field is optional or `false` for partner-route registrations (where the partner has its own member terms covering data contribution). This applies to both credentialled and established professional schemas.

#### Handling roll-up objections

When a professional objects within the 7-day pre-merge notification window for
high-risk regulated-tier categories, the following branch applies:

1. **Canonical merge is paused.** The partner-managed roll-up does not
   complete. The professional's existing self-registered record remains
   live.

2. **Partner-provided fields are held as disputed.** The fields the partner
   would have made canonical are recorded against the professional's record
   with verification status `partner-disputed` and are not used in matching
   weighting until resolved.

3. **Conflict assessment.** Admin assesses whether the partner data and
   self-declared data conflict on material fields (regulatory standing,
   licence number, jurisdictional coverage, disciplinary history).
   - If no material conflict: the self-declared record remains live;
     partner data is held in `partner-disputed` state pending the
     professional's response or admin resolution.
   - If material conflict in a regulated-tier category: the profile is
     temporarily suppressed from matching pending resolution, with
     `matching_status: admin-suspended` and a reason code indicating
     roll-up dispute.

4. **Resolution.** Admin reviews the dispute under the dispute resolution
   process in §5.9. Resolution outcomes:
   - Partner data confirmed canonical: roll-up completes, partner-managed
     fields become canonical, self-declared fields that do not contradict
     partner-verified fields are retained.
   - Self-declared data confirmed correct: partner data is recorded as
     disputed, the partner is notified of the conflict, the professional's
     record continues with self-declared canonical fields.
   - Neither resolves cleanly: the profile remains suppressed from matching
     pending further evidence; the professional and partner are both
     notified.

5. **Professional rights during suppression.** The professional retains
   access to their portal, may continue to update self-declared fields
   (subject to the conflict not widening), and is notified of the dispute
   resolution outcome within the SLA published in operational documentation.

The professional's right to maintain self-declared fields that do not
contradict partner-verified fields is preserved through the resolution. The
roll-up does not extinguish self-declared content; it normalises canonical
authority for fields where the partner is the authoritative source.

#### Fee model

Individual self-registration is free during the initial period after launch. After the initial period closes, individual self-registration remains available with a small annual fee. Hardship waivers and fee caps published in operational documentation.

Partner-route registration carries no ATAH fee.

## 11. Data Governance Implementation

### 11.1 Privacy Architecture Overview

ATAH handles consumer personal data as a transient conduit, not a repository. Consumer personal data is **never written to the main registry database**. All consumer personal data flows through the **transient encrypted vault** (Section 11.6), is retrieved through authenticated access, and is deleted or crypto-erased per category retention rules.

**Notification channels (SMS and email) carry no consumer personal data.** They carry only non-sensitive notification metadata and authenticated retrieval references.

### 11.2 Deletion Scope Table

| Store | Personal data allowed? | Deletion mechanism | Max retention |
|---|---|---|---|
| Main registry database | No | Rejected at validation | n/a |
| Transient encrypted vault | Yes (during active handoff) | Crypto-erasure on retrieval, expiry, cancellation, or revocation | Category-specific; max 72h post-resolution |
| Consent receipt store | Hash and metadata only — no PII | TTL on receipt expiry + 90 days | Per receipt expiry |
| Audit log | Pseudonymous identifiers only — no PII | Retention per audit policy | 7 years |
| Object storage (documents) | No consumer documents stored | n/a | n/a |
| SMS/email notifications | No PII allowed in payload | Provider-managed | n/a (no PII) |
| Notification provider logs | No PII (because no PII in payload) | Provider-managed | Provider-managed |
| Webhook delivery queue | No consumer PII; metadata only | Standard queue retention | 7 days |
| Analytics / metrics | Anonymised outcome data only | Standard | 2 years |
| Backups | Encrypted; mirror primary retention | Crypto-erase on key rotation | Mirror primary |
| Dead-letter queue | Sanitised — no PII | Standard | 30 days |

If a personal data field is detected in a store where it is not allowed, the entry is automatically purged and a critical audit event is logged.

### 11.3 Consumer Personal Data Categories and Retention

**Stage 2 pre-handoff check data:** stored in the transient encrypted vault. Crypto-erased on Stage 2 resolution. Maximum retention: 72 hours after Stage 2 timeout. Never written to the main registry database or notification providers.

**Stage 3 contact details:** stored in the transient encrypted vault. Crypto-erased immediately after successful authenticated retrieval by the professional.

**Type 3 referral contact details:** same handling as Stage 3.

**Post-introduction data:** anonymised outcome data only. Retained two years. Schema:

```json
{
  "record_id": "anon-2026-04-10-001",
  "professional_category": "lawyer",
  "professional_tier": "credentialled",
  "registration_route": "individual",
  "matter_type": "contract_review",
  "jurisdiction": "US-TX",
  "outcome_code": "introduction-successful",
  "introduction_type": "consumer_introduction",
  "stage_reached": "stage_3",
  "pre_handoff_check_used": "conflict_check",
  "response_time_hours": 2.5,
  "created_date": "2026-04-10",
  "referring_relationship_existed": false,
  "enhanced_verification_present": true
}
```

### 11.4 Trusted Partner Data Governance

Partner data is tagged with its source. When a partner is suspended, data is suppressed pending review. When revoked, data is removed within 30 days. Professionals may dispute partner data via the professional portal. Partners may not contribute consumer personal data; the partner API accepts only professional credential, standing, and quality data.

### 11.5 Handoff Access Control Model (Tiered)

`handoff_id` identifies an introduction. It does not, on its own, grant access to introduction state or data.

ATAH operates a two-tier access model:

**Tier 1 — Read-like operations.** `GET /v1/introductions/:handoff_id` and `check_introduction_status` are available to:

- The original asserter platform and client (matched against the handoff record)
- Any authenticated AI platform that supplies a `consumer_ref` matching the consumer reference recorded on the handoff (allows cross-platform user experience: a consumer using a different AI mid-flow can check status if their platform-side consumer reference matches)

Read access returns `current_state`, `state_history`, `last_updated`, and reduced metadata. It does **not** return personal data.

**Tier 2 — Write/state-changing operations.** All `POST` operations that advance state, submit data, cancel, revoke, or report outcomes require a valid `handoff_access_token` in addition to the `handoff_id`.

The `handoff_access_token` is:

- Issued at `initiate_introduction` to the asserter platform/client
- Bound to the asserting platform/client identity
- Scoped to the specific handoff
- Time-limited (default 4 hours, refreshed on each successful state-changing call up to the introduction's lifetime cap)
- Single-use within state transitions where appropriate
- Revocable on consent revocation or consumer cancellation
- Logged on every use

A `handoff_access_token` is opaque, rotatable, and never logged in URLs (passed only in `Authorization` headers). Possession of a `handoff_access_token` plus a `handoff_id` is required for any write operation.

This model preserves cross-platform user experience for status checks while preventing possession of `handoff_id` alone from granting state-changing power.

#### consumer_ref constraints (v0.8.1)

The `consumer_ref` field used in cross-platform status checks is a continuity handle, not an identity. v0.8.1 specifies what `consumer_ref` MUST NOT be; the positive specification of how `consumer_ref` is generated, salted, and bound to user-mediated continuity consent is deferred to a subsequent patch version.

A v0.8.1-conforming implementation MUST NOT use any of the following as `consumer_ref`:

- A bare hash of the consumer's email address
- A bare hash of the consumer's phone number
- A bare hash of any government-issued identifier (national ID, driving licence, passport)
- Any other stable personal identifier without per-handoff salting and explicit user-mediated continuity consent

The risk these constraints close: if `consumer_ref` were a bare hash of a stable personal identifier, any AI platform holding that identifier could read any handoff that consumer has open with any other AI platform. That is a worse cross-platform correlation handle than the bearer-token problem the tiered handoff access model was designed to solve.

The reference registry implements `consumer_ref` as a per-handoff salted derivation bound to user-mediated continuity consent captured at the original Stage 3 consent receipt. The full mechanism is documented in the reference registry's implementation notes and will be promoted to spec text in a subsequent patch version once the field has been exercised in production.

### 11.6 Transient Encrypted Vault and Authenticated Retrieval

Consumer personal data is stored in the transient encrypted vault. The vault has the following properties:

- **Per-record encryption.** Each record uses a unique data encryption key (DEK) wrapped by the vault's master key.
- **Crypto-erasure.** Deletion erases the DEK; the encrypted ciphertext becomes unrecoverable. Crypto-erasure is the primary deletion mechanism.
- **Time-bounded.** Each record has a TTL based on its handoff stage and category.
- **Authenticated access.** Records are retrieved via authenticated retrieval with a single-use retrieval token.
- **Auditable.** Every retrieval is logged with timestamp, retriever identity, and record identifier.

#### Authenticated retrieval flow

1. **Notification dispatch.** When personal data is placed in the vault for a professional, ATAH sends a notification to the professional via SMS or email. The notification carries:
   - Notification metadata only (introduction reference, urgency tag, category, jurisdiction)
   - An authenticated retrieval URL with a single-use token in the URL fragment (not query string, to avoid logs)
   - **No personal data**, no consumer name, no matter description, no contact details
2. **Retrieval.** The professional taps the link, which opens the ATAH portal. The portal authenticates the professional per the `stage2_auth_tier` set on the category:
   - `tier_1`: token alone is sufficient (no further auth — used only for low-sensitivity scope confirmation)
   - `tier_2`: token + device-bound session (most categories — the SMS-bound device is treated as the trust signal)
   - `tier_3`: token + step-up authentication (SMS one-time code or email confirmation in addition to the original notification — required for healthcare, sensitive legal matters)
3. **Retrieval validates.** The vault validates the token, the auth tier, the retrieval window, and the professional identity match. On success, the data is returned to the portal.
4. **Crypto-erasure.** After successful retrieval, the DEK is erased synchronously. Audit log records `delivered-and-deleted`.

#### Failure handling

- **Retrieval window expiry without retrieval.** Token expires (default 24 hours for standard, 4 hours for urgent). Vault crypto-erases the data automatically. Handoff marked `delivery-failed`. Consumer's AI platform notified.
- **Retrieval failure (e.g. failed authentication).** Up to 3 retrieval attempts within the window. After 3 failed attempts, retrieval window suspended; manual support request required.
- **Crypto-erasure failure.** Critical alert. P0 incident. Manual erasure initiated. Further introductions for the consumer are blocked until resolved.

This model ensures that no consumer personal data is ever transmitted through SMS or email; only retrieval references are. It also ensures that even if a notification provider is compromised, the personal data is not exposed.

### 11.7 Right to Information and Erasure (Limited Surface)

Because ATAH does not retain consumer personal data, the standard data subject right to erasure is satisfied by ATAH's transient-only model. Consumer personal data is held only in the vault during active handoffs and is crypto-erased per the protocol.

For audit log entries containing pseudonymous identifiers (handoff IDs, consent receipt hashes), ATAH retains 7-year audit records consistent with regulatory expectations. These records contain no PII.

For data held by asserting platforms (full consent receipts) and by professionals (data they retrieved during a handoff), erasure obligations rest with those parties under their own privacy frameworks.

Detailed DSAR machinery for federation scenarios is deferred to v0.9.

## 12. Notification Service

### 12.1 Phase 1 — Professional Notifications

Notifications are non-PII alerts only. Personal data is never carried in notifications.

- **Primary (Phase 1):** SMS. Notification metadata + authenticated retrieval link.
- **Parallel:** email with same content.
- **Best-effort (Phase 1):** MCP notification to professional's AI assistant if connected.
- **Phase 2 primary:** MCP push notifications, once MCP push standards are established and stable.

Example SMS body:

```
ATAH: New urgent referral available (lawyer, US-TX). Tap to view: https://atahprotocol.org/p/r#tok_xxxxxx
```

### 12.2 Phase 1 — Consumer AI Agent Updates

Consumer AI agents track introduction status by polling `check_introduction_status`.

- The asserter platform polls with `handoff_id` + handoff_access_token (read access)
- Cross-platform status check available to authenticated AI platforms with matching `consumer_ref`
- Phase 2: structured webhook events pushed to registered callback URLs

### 12.3 Notification Authentication Tokens

All notification retrieval links use single-use tokens with TTLs aligned to category sensitivity. Tokens are stored in Redis with TTL. Tokens are invalidated after use.

### 12.4 Partner and Verifier Notifications

Partners receive webhook notifications on conflict detection, partner score threshold events, conflict resolution outcomes, and dispute escalations.

Verifiers receive webhook notifications on enhanced verification disputes, audit status events, and approval changes.

## 13. Security Model

### 13.1 Authentication Mechanisms by Actor

- **AI platforms (MCP and REST):** OAuth 2.1-compatible (Section 8.1). Resource indicators, audience validation, explicit scopes, short-lived tokens with refresh and rotation. Dynamic Client Registration supported.
- **Trusted partners and independent verifiers (writes):** signed JWT client assertion or HMAC-signed requests with timestamp, nonce, and replay protection. API keys may be used for read-only operations but not for data writes.
- **Professionals:** magic link via email or SMS one-time code, OIDC where supported. No passwords. Long-lived sessions expiring after 90 days of inactivity.
- **Consent assertion:** structured consent receipts (Section 4.10), not boolean flags. Asserting platforms take legal responsibility for consent capture; ATAH stores receipt hash and metadata.
- **Handoff access:** tiered model per Section 11.5.

### 13.2 Cryptography and Transport

- All data encrypted in transit using TLS 1.3 minimum.
- All data encrypted at rest.
- Separate encryption keys for transient vault (per-record DEK wrapped by master key).
- Master key rotation every 90 days.
- Crypto-erasure as primary deletion mechanism for transient vault.

### 13.3 Rate Limiting and Abuse Prevention

- Rate limits on all public endpoints, configurable per registered platform.
- Stricter limits on `POST /v1/query` to prevent registry scraping.
- Partner data push: per-batch rate limiting; bulk pushes must be batched.
- Consent assertion abuse monitoring: platforms with anomalous consent assertion patterns are reviewed.

### 13.4 Threat Model Summary

A separate `SECURITY.md` provides the full threat model, responsible disclosure policy, and security contact information. The summary threat categories addressed:

- Malicious AI platform asserting false consent
- Malicious or compromised professional
- Malicious or compromised partner
- Review-gaming rings and Sybil professionals
- Verifier capture or compromise
- Scraping and data exfiltration
- PII leakage through notification channels (mitigated by Section 11.6)
- Consent forgery (mitigated by hash storage per Section 11.5)
- Partner data poisoning
- Insider admin abuse
- Handoff_id leakage (mitigated by tiered access per Section 11.5)

Each threat category is addressed in the controls described in this Section and in `SECURITY.md`. Residual risks and acceptance rationale are documented in `SECURITY.md`.

### 13.5 Audit Logging

All state transitions, data accesses, consent receipts, retrieval events, partner data writes, verifier writes, admin actions, and security events are recorded in the audit log. Audit log entries:

- Carry pseudonymous identifiers only (no PII)
- Are tamper-evident (hash-chained)
- Are retained for 7 years
- Are accessible to ATAH admin and to subjects of audit (professional, partner, verifier) for entries concerning them, on request

## 14. Conformance Classes

ATAH conformance is not a claim that an implementation uses the same codebase or infrastructure as the ATAH reference registry. It is a claim that the implementation preserves the protocol's mandatory semantics.

ATAH v0.8 defines four conformance classes. Implementations may declare which classes they support.

### 14.1 Core Object Conformance

An implementation conforms at the core-object level if it produces and consumes ATAH schema-valid:

- professional records;
- provenance maps;
- consent receipt references;
- handoff records (Type 1, Type 2, Type 3);
- match responses;
- presentation disclosures;
- outcome reports;
- audit events.

Core object conformance is the minimum a non-registry implementation may claim — for example, a partner-side tool that prepares ATAH-valid data for submission, or an analytics tool that consumes ATAH match-response payloads.

### 14.2 Binding Conformance

A binding conforms if it exposes ATAH operations through a defined interface — such as MCP or REST — while preserving the core protocol semantics. Bindings do not change which fields are required, which provenance must be returned, or which consent stages apply. They only change how those operations are invoked.

v0.8 defines two reference bindings: the MCP binding (Section 8) and the REST binding (Section 7). A future implementation MAY define additional bindings (A2A-compatible, push, federation) and claim binding conformance for the binding(s) it implements.

### 14.3 Registry Conformance

A registry conforms if it:

- stores or resolves professional records under an `atah_id` namespace;
- preserves field-level provenance via the `_provenance` map;
- enforces category compliance gating (no surfacing in match results before the relevant compliance annex is approved);
- applies no commercial weighting in matching;
- supports record correction and dispute flows;
- returns ATAH-valid match responses with the required `presentation_disclosure` block;
- supports consent receipt references in introduction creation;
- manages handoff state if it supports introductions;
- implements data minimisation as set out in Section 11;
- publishes a conformance statement listing supported categories, jurisdictions, bindings, and conformance classes.

### 14.4 Governance Conformance

A governed implementation conforms if it publishes:

- partner approval criteria;
- verifier approval criteria;
- review-platform classification criteria;
- fee principles (cost-recovery rationale, waivers for public-interest organisations);
- the process for correcting inaccurate professional data;
- the process for handling concern flags routed to it.

Governance conformance is what allows downstream parties — AI platforms, professional bodies, regulators — to assess whether an implementation operates the protocol on terms consistent with the Charter Part One core commitments.

### 14.5 Reference Registry Scope

The initial ATAH reference registry is expected to conform to all four conformance classes. Future implementations may declare narrower conformance — for example, a profile-only implementation that supports core object and binding conformance but does not run the introduction lifecycle.

Finer-grained conformance profiles (such as a `profile-provider` profile or a `matching-registry` profile distinct from a `full-handoff-registry` profile) are a v0.9 candidate. v0.8 primarily specifies the full reference registry profile.

### 14.6 Record Portability

Record portability between conforming registries — including export format, duplicate resolution, and cross-registry provenance trust — is deferred to federation work in v0.9 or v1.0.

### 14.7 Conformance Statement Requirement

A conforming implementation publishes a public conformance statement at a discoverable location (recommended: `/.well-known/atah-conformance`) listing:

- protocol version supported;
- schema version supported;
- bindings supported (with binding versions);
- conformance classes claimed;
- categories supported;
- jurisdictions supported;
- governance documents URL (charter or equivalent);
- contact for conformance enquiries.

The reference registry publishes its conformance statement as part of the v0.8 launch package.

### 14.8 AI Platform Minimal Profile

For AI platforms integrating with an ATAH-conformant registry, this section defines the smallest safe integration. An AI platform meeting this profile can support consumer-initiated (Type 1) introductions end-to-end without supporting Type 2 referral establishment, Type 3 client referrals, or partner/verifier write paths.

**Required tool support (from `mcp-tools.json` or REST equivalents):**

- `find_professional`
- `initiate_introduction`
- `check_introduction_status`
- `submit_stage2_data`
- `submit_stage3_data`
- `cancel_introduction`
- `revoke_consent`

**Optional tool support at this profile level:**

- `get_consent_requirements` — recommended; allows the platform to fetch canonical consent text rather than maintaining its own.
- `report_introduction_outcome` — recommended; supports the introduction lifecycle's outcome state.

**Required behaviours:**

- Surface the `presentation_disclosure` block from every match response to the end user. The protocol's non-recommendation framing must be visible.
- Capture and assert structured consent receipts (Section 4.10), not boolean flags.
- Support consent revocation and introduction cancellation as first-class user actions (the corresponding tools must not be hidden behind harder-to-find UX than the submission tools).
- Treat `handoff_id` as non-bearer; use `handoff_access_token` for state-changing calls (Section 11.5).
- Never transmit consumer personal data through SMS or email; rely on ATAH's transient vault and authenticated retrieval (Section 11.6).

**Not required at this profile level:**

- Type 2 (professional-to-professional referral establishment) tool support.
- Type 3 (professional-initiated client referral) tool support.
- Partner data write paths.
- Verifier write paths.
- Hosting an MCP server (the platform is a client; it consumes the registry's MCP server).
- Running matching internally (the registry runs matching).

A platform meeting this profile claims **minimal binding conformance** in its conformance statement and may state: "Supports ATAH v0.8 AI Platform Minimal Profile."

The minimal profile is intentionally narrow. Platforms that want to support professional-to-professional referrals, partner data submission, or verifier integration extend beyond it as their use case requires.

## 15. Professional Records and Registration

ATAH requires professional records, but those records do not have to originate in only one way. This section clarifies the relationship between protocol, registry, and professional record, and answers the question often raised at first reading: "do professionals register with ATAH itself?"

### 15.1 Records, registries, and the protocol

Professionals do not register "with the protocol." A protocol is a contract; it has no membership.

Professionals are *represented* in an ATAH-conformant registry. At v0.8, that means the ATAH reference registry. As federation develops in v0.9 or v1.0, other conforming registries may exist, and a professional may be represented in any of them, subject to the federation rules then in force.

### 15.2 Three record creation routes

At v0.8, the initial ATAH reference registry supports three record creation routes:

#### Partner-created records

A trusted partner contributes records about professionals it covers. The partner-managed fields (regulatory standing, membership status, formal credential evidence) remain controlled by the partner and identified with the partner's name in the `_provenance` map.

The professional may later claim the record to add self-declared fields, manage handoff preferences, dispute partner-managed data through the dispute process (Section 5.9), or connect their AI assistant.

#### Professional-claimed records

A professional claims an existing record created from partner data, seed data, or earlier registration. Partner-managed fields remain controlled by the partner. Self-declared fields are editable by the professional. Verified credentials remain verified.

A claimed record is not a different record from the original — it is the same record with a control layer attached.

#### Direct self-registered records

A professional creates a record directly where:

- no trusted partner exists for their field;
- partner participation is optional and the professional has chosen not to belong to a partnered body;
- the relevant partner has not yet joined an ATAH-conformant registry; or
- the professional is outside any partnered network.

Direct self-registered records are permitted but their fields are clearly labelled as `self-declared` in the `_provenance` map until and unless they are independently verified through partner data, Verifiable Credentials, or enhanced verification.

### 15.3 Record versus account

A professional record is not the same as a professional account.

- A *professional record* is a data object representing the professional, conforming to the professional profile schemas (Sections 4.6 and 4.7).
- A *professional account* is a control layer that lets a professional log in and act on the record — claim it, edit self-declared fields, dispute partner data, manage handoff preferences, respond to introductions.

Records may be partner-created, seeded, claimed by the professional, or directly registered. An account is only required where the professional needs to interact actively with the registry. A professional with a partner-created record may exist in the registry without ever creating an account, and may be returned in match results based on partner data alone.

### 15.4 Provenance honesty

Each professional record carries field-level provenance:

- A self-declared field is always labelled as `self-declared`.
- Partner-managed fields identify the partner source.
- Verified credentials identify the issuer.
- Enhanced verification identifies the independent verifier.

Professionals do not need to understand protocol mechanics. The protocol ensures every claim about them is labelled by source and verification status.

### 15.5 Future external conforming registry records

For v0.8, all records live in the reference registry. Future conforming registries may hold professional records under their own namespace and expose ATAH-compatible responses, once federation mechanics — registry namespace resolution, cross-registry trust, duplicate professional records, consent receipt portability — are specified in v0.9 or v1.0.

### 15.6 Claim authentication

When a professional claims a partner-created record, that claim must be authenticated to prevent identity theft against a real professional's record. v0.8 specifies a minimum claim authentication requirement and flags fuller claim-authentication hardening as v0.9 work.

**v0.8 minimum requirement.** A professional claiming a partner-created record must demonstrate possession of an identifier that the partner record carries — typically a verified email match against the partner-supplied email address, or an equivalent partner-attested identifier. Where the partner record does not carry such an identifier, claiming requires either document upload (reviewed by the registry operator against the partner's published verification standard, or by the partner directly where the partner has agreed to provide that review) or partner-side attestation that the claimant is the named professional.

**Scope of editing rights after claim.** Partner-managed fields remain controlled by the partner and are not editable by the claiming professional. Disputes about partner-managed data are routed through the dispute flow (Section 5.9). Self-declared fields become editable by the claiming professional but remain labelled `self-declared` in the `_provenance` map.

**Deferred to v0.9.** Full claim-authentication hardening including step-up authentication for claims, anomaly detection on claim attempts (rapid succession from new IPs, attempts to immediately edit fee or contact fields after claim), notification to the *partner-known channel* (the contact channel — typically the email address — recorded on the partner-supplied record at the time of partner data ingestion) with an objection window before claim takes effect, and rate limiting on claim attempts per record and per IP. The reference registry SHOULD apply these controls operationally at launch; they are not yet protocol-mandated for all conforming registries pending v0.9 specification.

## 16. Glossary

- **AI platform** — software system (e.g. ChatGPT, Claude, Gemini, custom agents) that interacts with consumers and uses ATAH on their behalf
- **Asserter / asserting platform** — the AI platform that captures consumer consent and asserts it through a consent receipt
- **ATAH Core Protocol** — the schemas, semantics, lifecycle, consent receipts, provenance rules, and conformance principles. Transport-neutral. Defined in this specification.
- **atah_id** — opaque identifier for a professional in an ATAH-conformant registry
- **Binding** — a way to expose ATAH operations through a defined interface (e.g. MCP, REST, future A2A or push). Does not change core semantics.
- **Claimed record** — an existing professional record that has been claimed by the professional, who can then edit self-declared fields and manage handoff preferences
- **Concern flag** — record raised against a professional based on a `consumer-reported-concern` outcome. Admin-only visibility, never used in matching weighting, with right of reply for the named professional. Named "concern" rather than "conduct" because ATAH does not adjudicate professional conduct (see Section 5.10).
- **Conformance** — implementations conforming to the ATAH specification preserve mandatory principles: provenance visibility, no commercial weighting, staged consent, category compliance gating, data minimisation. See Section 14 for the four conformance classes.
- **Conforming registry** — a registry implementation that meets the registry conformance class (Section 14.3). At v0.8, the ATAH reference registry is the only conforming registry; future implementations may follow once federation is specified.
- **Consent receipt** — structured record of consent capture (Section 4.10), referenced by `consent_receipt_id`. Replaces the boolean `consumer_consent`.
- **Consumer** — protocol-neutral term for the party seeking professional services. Equivalent terms in real-world contexts: client (legal, accounting), patient (healthcare), policyholder (insurance), customer. ATAH never communicates with the consumer directly; it is always mediated by an AI platform.
- **Credentialled professional** — professional holding a formal licence, certification, or accreditation from a regulatory or professional body
- **Direct self-registered record** — a record created by the professional directly, where no trusted partner covers them or where partner participation is optional
- **Established professional** — recognised practitioner in a field where formal licensing does not exist but professional standing is real and verifiable
- **Federation** — future cross-registry operation, including registry namespace resolution, cross-registry trust, duplicate professional records, consent receipt portability, handoff routing, and registry operator governance. Deferred to v0.9 or v1.0; v0.8 is single-registry.
- **Handoff** — the staged process of moving a consumer from AI interaction to a verified professional
- **handoff_id** — identifier for a handoff. Does not on its own grant access; tier-1 read access available with consumer_ref match, tier-2 write access requires handoff_access_token (Section 11.5)
- **handoff_access_token** — short-lived token bound to platform/client/scope/expiry. Required for state-changing operations on a handoff.
- **Partner / Trusted partner** — organisation contributing verified data about professionals (regulator, professional body, etc.)
- **Partner-created record** — a professional record created from trusted partner data, with partner-managed fields controlled by the partner
- **Pre-handoff check (Stage 2)** — category-flexible check performed before contact details are released
- **Professional account** — a control layer that lets a professional log in and act on a professional record (claim, edit, dispute, manage handoff preferences). Distinct from a record; see Section 15.3.
- **Professional record** — a data object representing a professional, conforming to the professional profile schemas. May be partner-created, claimed, or direct self-registered.
- **Protocol-conformant** — preserving the ATAH Core Protocol's mandatory semantics; see Section 14
- **Provenance** — the source and verification status of every data point, made visible through the parallel `_provenance` map
- **Reference registry** — the initial ATAH-operated implementation. Demonstrates and operates the protocol. Implements all four conformance classes. Not the protocol itself.
- **Registry operator** — organisation running a registry implementation
- **Registry profile / Registry operation profile** — the set of behaviours required for systems that store professional records, ingest partner evidence, run matching, and manage introduction lifecycle state. See Section 14.3.
- **Review platform** — specialised partner class providing aggregated review/feedback data with anti-gaming controls
- **Stage 1 / Stage 2 / Stage 3** — the three stages of a Type 1 introduction: introduction acceptance, pre-handoff check, contact release
- **Transient vault** — the encrypted, time-limited store for consumer personal data passing through introductions
- **Type 1 / Type 2 / Type 3** — the three introduction types: consumer-initiated (Type 1), professional-to-professional referral relationship establishment (Type 2), professional-initiated client referral (Type 3)
- **VC / Verifiable Credential** — W3C Verifiable Credential. Accepted as a partner data submission format.
- **Vetting strength** — characterisation of a partner's standards: `regulatory`, `strong_membership`, or `open_membership`. Affects matching weight.

## 17. Repository Structure

```
atahprotocol/
  README.md
  EXPLAINER.md
  CHARTER.md
  CONFORMANCE.md
  ROADMAP.md
  GOVERNANCE.md
  CONTRIBUTING.md
  CODE_OF_CONDUCT.md
  SECURITY.md
  CHANGELOG.md
  CHANGES-FROM-V0_7-TO-V0_8.md
  LICENSE
  docs/
    decisions/                       ← Architecture Decision Records
  spec/
    v0.8/
      00-overview.md
      01-architecture.md
      02-schemas.md
      03-trusted-partners.md
      04-introduction-lifecycle.md
      05-api.md
      06-mcp-server.md
      07-matching-engine.md
      08-verification.md
      09-data-governance.md
      10-notifications.md
      11-security.md
      12-build-sequence.md
      full-spec.md
      CHANGELOG.md
      openapi.yaml
      mcp-tools.json
      schemas/
        professional-identity-credentialled.schema.json
        professional-identity-established.schema.json
        provenance-map.schema.json
        profession-category.schema.json
        profession-categories.json
        query.schema.json
        match-response.schema.json
        consent-receipt.schema.json
        consent-receipt-stored.schema.json
        handoff-type1.schema.json
        handoff-type2.schema.json
        handoff-type3.schema.json
        stage2-prehandoff.schema.json
        stage3-contact.schema.json
        trusted-partner.schema.json
        partner-data-push.schema.json
        verification-scope.schema.json
        independent-verifier.schema.json
        enhanced-verification.schema.json
        review-platform.schema.json
        conflict-record.schema.json
        dispute-record.schema.json
        concern-flag-record.schema.json
        rollup-match-record.schema.json
        audit-event.schema.json
        error.schema.json
  press/
    media-release.md
    associations-release.md
    ai-platforms-release.md
```

The reference implementation code does not live in the specification repository.

## 18. Build Sequence

This build sequence describes the work to implement v0.8 from scratch. For implementations migrating from v0.7, only the v0.8 changes need to be applied.

### Phase A — Foundation (Weeks 1–2)

Protocol specification v0.8 finalised and published. All schemas including consent-receipt, handoff-type1/2/3, provenance-map, audit-event, error written and validated. Profession category metadata populated with launch categories including `matching_weight_profile`, `review_signal_weight_cap`, and `stage2_auth_tier`. Independent verifier schema. Repository structure created. Domain registered. GitHub repository public. Legal advice on structure and jurisdiction commissioned.

### Phase B — Core Infrastructure (Weeks 2–4)

Postgres schema for all entities. Transient encrypted vault built (per-record DEK, crypto-erasure, TTL, audit logging). Registry service built with both registration routes. Verification job runner — illustrative integrations such as NIPR-style licensing data (subject to partner agreement) and state bar database integrations (subject to partner agreement, in jurisdictions where state bar partners are in place). Rolling daily batch with concurrency limit and exponential backoff. Professional registration flow with magic link/OIDC, basic registry integrity checks. Full delivery-and-deletion protocol from Section 11 with all failure paths. OAuth 2.1 endpoints (authorization server metadata, token endpoint, dynamic client registration). Audit logging service with tamper-evident hash chaining. `Accept-Version` and `deprecation_warning` support.

### Phase C — Trusted Partner Engine (Weeks 3–5)

Partner API with `verification_scope` and `vetting_strength`. VC submission acceptance path. Independent verifier API including verifier approval flow (admin-only), assessment fee handling, annual audit fee tracking, `scope_tier` classification. Review platform partner class with anti-gaming attestation, `review_platform_class`. Conflict detection with meaningful/benign field classification. Partner scoring engine. Partner score threshold monitoring. Conflict review queue. Dispute flow. Partner webhook notifications. Roll-up logic.

### Phase D — Matching and Query (Weeks 4–6)

Matching engine — all scoring components, compliance-pending hard exclusion, referral signal discount with anti-gaming controls (reciprocal cap, time decay, referrer weighting, evidence requirement), verification quality with four sub-components, vetting-strength weighted partner data, review-platform-class-weighted review signals (with category caps), declared/verified scope completeness split. Query API. Match response includes `presentation_disclosure`, full per-sub-component contribution metadata, and full provenance map.

### Phase E — Introduction Lifecycle (Weeks 5–7)

Type 1, Type 2, Type 3 state machines with full schemas (Section 6). Tiered handoff access control (Section 11.5) — handoff_access_token issuance, validation, rotation. Stage 2 reframed as category-flexible pre-handoff check. Simultaneous introduction rules. Stage 2 and Stage 3 data flow through transient vault. Cancel and revoke endpoints/tools. Outcome reporting. Consumer-reported-concern routing to admin-only flag flow with professional notification and right of reply (Section 5.10).

### Phase F — MCP Server (Weeks 6–8)

OAuth 2.1 conformance: authorization server metadata, protected resource metadata, resource indicators, audience validation. All nine MCP tools (Section 8.2) implemented and tested with structured input/output schemas. `mcp-tools.json` published. Cancellation and revocation tools.

### Phase G — Notification Service (Weeks 6–8)

SMS and email integrated with strict no-PII policy. Notification metadata + authenticated retrieval link only. Authenticated retrieval portal with tiered auth (`tier_1` / `tier_2` / `tier_3`). MCP notification to professional AI assistants where connected. Consumer AI agent polling. Webhook system for partners and verifiers.

### Phase H — Professional Portal (Weeks 7–9)

Profile management UI for both tiers and routes. Concern flag inbox with right of reply. Authenticated retrieval portal for Stage 2/3 data. Pre-handoff check display. Referral network management. AI assistant MCP connection. Magic link/OIDC authentication. Partner data dispute flow. Enhanced verification display. Roll-up notification flow with pre-merge objection window for high-risk categories.

### Phase I — Testing and Launch Preparation (Weeks 8–11)

End-to-end testing: all three introduction types, both tiers, both registration routes. Consent receipt generation, hash storage, revocation. Tiered handoff access control. Vault retrieval at all three auth tiers. Delivery-and-deletion protocol with failure paths. Compliance-pending exclusion. Referral signal anti-gaming. Priority-state lawyer verification + non-priority self-declared. Verification batching with 429 handling. OAuth 2.1 MCP authentication. `Accept-Version` and `Deprecation`. Trusted partner integration with VC submission. Enhanced verification end-to-end. Concern flag with right of reply. Security review with threat model exercise. GDPR audit. Performance testing. Founding cohort onboarded. First trusted partners. First independent verifier. Press materials finalised. Public launch.

## 19. AI Coding Agent Instructions

This section is written specifically for AI coding agents building ATAH. Read the README, EXPLAINER, and CHARTER before writing any code.

### What to build first

Phase A and Phase B. Schemas, registry, transient vault, OAuth 2.1 endpoints, and delivery-and-deletion protocol — all foundational and must precede other work. Do not defer the vault or the consent receipt store.

### What not to build

- Do not build commercial referral analytics within the protocol surface
- Do not build payment processing
- Do not build AI/LLM functionality inside ATAH; matching is deterministic
- Do not build partner ranking or promotion logic of any kind
- Do not build commercial weighting into matching, ever, in any form
- Do not add a third professional tier
- Do not open compliance-pending categories for matching
- Do not build a conduct adjudication system; route concerns per Section 5.10
- Do not transmit consumer PII through SMS or email under any circumstances
- Do not store consumer PII in the main registry database under any circumstances
- Do not treat `consumer_consent: true` boolean as valid consent; require structured consent receipts
- Do not allow `handoff_id` possession alone to grant write access; require `handoff_access_token`
- Do not allow verifiers to set their own `breadth_score`, `depth_score`, or `freshness_score`
- Do not allow professionals to opt out of roll-up rule
- Do not allow `vetting_strength` or `review_platform_class` to be amended unilaterally
- Do not refund the verifier assessment fee
- Do not adjudicate concern flags; ATAH only routes patterns to relevant bodies

### Key design rules

- Every introduction requires a fresh consent receipt. Never re-use consent.
- Consumer personal data must never be written to the main registry database.
- Consumer personal data must never be transmitted through SMS or email — only notifications.
- Follow the delivery-and-deletion protocol in Section 11.6 exactly. Build and test all failure paths.
- All state transitions must be atomic and logged.
- The matching engine must never include any commercial weighting.
- The MCP server must conform to current MCP authorisation guidance (OAuth 2.1-compatible) and require valid tokens with audience validation.
- The REST API uses OAuth 2.1 for AI platforms; signed-request mechanisms for partner/verifier writes; magic-link/OIDC for professionals.
- No passwords anywhere.
- All data points must be tagged with provenance via `_provenance` map.
- Meaningful data conflicts must suppress the professional from matching until resolved.
- `compliance-pending` professionals must be excluded at the hard exclusion step.
- Simultaneous introduction rules in Section 6.3 are hard constraints.
- 409 Conflict for duplicate consumer+professional pairs.
- `Accept-Version` and `deprecation_warning` from day one.
- Idempotency keys required on all mutating endpoints.
- Verification batching uses rolling daily batch with concurrency limit and backoff.
- Inbound referral signal subject to anti-gaming controls per Section 9.
- Pre-handoff check type set per category, with optional override; lifecycle skips Stage 2 for `none`.
- Roll-up logic with pre-merge notification for high-risk categories.
- Independent verifier approval is admin-only.
- Review platforms must complete anti-gaming attestation; review weight subject to category cap.
- Concern flags admin-only visibility, professional notified with right of reply.

### When in doubt

Choose the narrower scope. Choose more auditability. Choose more explicit consent. Choose less data retention. If a professional's tier is ambiguous, choose `established`. If a category's compliance status is unclear, treat it as `compliance-pending`. If a conflict field classification is unclear, treat it as meaningful. If a roll-up match confidence is borderline, route to review queue. If a category's auth tier is unclear, choose `tier_3`.

## 20. Definition of Done for v0.8 MVP

The v0.8 MVP is complete when all of the following are true.

- All schemas published, validated, and example payloads validate against their schemas
- `openapi.yaml` published and validates clean
- `mcp-tools.json` published with structured input/output schemas
- Consent receipt generation, hash storage, and revocation working end-to-end
- Tiered handoff access control working: read access via consumer_ref match; write access via handoff_access_token
- Transient vault working with crypto-erasure, all three auth tiers, audit logging
- Notifications confirmed PII-free across SMS and email
- Type 1 introduction runs end-to-end for credentialled and established professionals across at least two categories
- Type 2 referral relationship establishment runs end-to-end with structured consent
- Type 3 client referral runs end-to-end with both consent paths (receipt and attestation) and immediate vault crypto-erasure after retrieval
- Cancel and revoke endpoints/tools working with mandatory data deletion on revocation
- Compliance-pending exclusion confirmed at Step 1 of matching
- Inbound referral signal anti-gaming controls validated
- Roll-up logic with pre-merge notification for high-risk categories
- OAuth 2.1 MCP authentication working with audience validation
- Idempotency working on all mutating endpoints
- `Accept-Version` and `Deprecation` working
- Concern flag mechanism with admin-only visibility, professional notification, right of reply
- Two registration routes live; partner-route and individual self-registration both functional
- At least one trusted partner integration live for credentialled professionals (representative examples of the type include licensing data sources such as NIPR or a state bar API, subject to partner agreement)
- At least one trusted partner integration live for established professionals
- Partner scoring running and queryable
- Conflict detection end-to-end including meaningful suppression
- Dispute resolution flow end-to-end (partner data and enhanced verification disputes)
- Registry integrity checks at registration
- At least twenty founding professionals across both tiers and routes
- MCP server queryable by at least two AI platforms
- Stateless cross-platform status check support validated
- Public MCP endpoint live and accessible
- End-to-end audit trail reconstructable for any introduction
- `SECURITY.md`, `GOVERNANCE.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`, `CHANGES-FROM-V0_7-TO-V0_8.md` published
- Interim advisory group convened per Charter (Section: Transition to independent governance) when suitable members are identified and available to serve
- Press materials updated to reflect v0.8

---

*Specification ends. The accompanying [README](../../README.md), [EXPLAINER](../../EXPLAINER.md), and [CHARTER](../../CHARTER.md) provide the framing, plain-language explanation, and binding governance commitments. The detailed governance machinery is in [GOVERNANCE.md](../../GOVERNANCE.md). Security disclosure and threat model are in [SECURITY.md](../../SECURITY.md).*
