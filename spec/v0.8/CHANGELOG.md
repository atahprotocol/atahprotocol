# Spec v0.8 Changelog

This is the spec-specific changelog for v0.8. The top-level
[CHANGELOG.md](../../CHANGELOG.md) covers repository-wide changes including
governance and supporting documentation; this file mirrors the entries that
affect the specification, schemas, OpenAPI contract, and MCP binding
artefacts.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
Breaking changes are possible until v1.0.

A detailed file-by-file change log between v0.7 and v0.8 is in
[CHANGES-FROM-V0_7-TO-V0_8.md](../../CHANGES-FROM-V0_7-TO-V0_8.md).

---

## [v0.8.1] — April 2026

**Status:** Release candidate. First public publication.

Spec-level changes from the pre-publication review-remediation patch round
that produced v0.8.1 (full repository-wide changes in the top-level
[CHANGELOG.md](../../CHANGELOG.md)):

### Added

- **Verification confidence signal (§4.12).** Match responses for regulated
  categories will carry a `verification_confidence` field separate from the
  verification-quality score components, surfacing single-source vs
  multi-source verification basis. Field name, value enumeration
  (`single_source_regulator`, `single_source_membership`,
  `single_source_self_declared`, `multi_source_corroborated`,
  `multi_source_with_enhanced_verification`), and intended semantics
  committed in v0.8.1; schema implementation in
  `match-response.schema.json` lands in v0.8.2.
- **`consumer_ref` MUST NOT specification (§11.5).** Negative constraints on
  `consumer_ref` to close cross-platform correlation risk: bare hashes of
  email, phone, or government-issued identifier prohibited without
  per-handoff salting and explicit user-mediated continuity consent. The
  positive specification of the generation/salting mechanism is deferred to
  a subsequent patch version.
- **Roll-up objection branch (§10.4).** Five-step canonical-merge-paused /
  partner-disputed / conflict-assessment / resolution / professional-rights-
  during-suppression flow.
- **`acknowledged_rollup_terms` field on both professional schemas.**
  Optional boolean top-level field on
  `professional-identity-credentialled.schema.json` and
  `professional-identity-established.schema.json` with conditional-required
  semantics: MUST be true when `registration_route: individual`. Closes a
  contract gap between OpenAPI registration documentation and the schemas
  with `additionalProperties: false`.

### Changed

- **§7.3 authorisation matrix scopes aligned to OpenAPI scope names.**
  All scopes now carry the `atah:` prefix consistent with `securitySchemes`
  in `openapi.yaml`; `introductions:read_received`,
  `introductions:respond`, and `admin:*` collapsed into the OpenAPI scopes
  that actually exist (`atah:introductions:read`,
  `atah:introductions:write`, `atah:admin`).
- **§8.1 OAuth scope list.** Removed orphan `atah:consent:revoke` scope;
  clarified that consent revocation is authorised under
  `atah:introductions:write` (matching OpenAPI and MCP).
- **§8.2 `find_professional` input shape.** Corrected to match the actual
  `query.schema.json` structure (matter sub-object containing category,
  matter_type, location, urgency — all required); excluded
  `matching_status` list expanded to `compliance-pending`,
  `regulatory-suspended`, `admin-suspended`.
- **§4.10 `data_categories` enum.** Prose list aligned with schema (added
  `contact_preference`).
- **§4.1 prefix list.** Removed unused `rp-` prefix; reframed list as
  non-exhaustive and added example-only prefixes used in records.
- **§6.4 Type 2 example.** Corrected `proposal_id` prefix from `rp-` to
  `proposal-` (matching schema and OpenAPI).
- **§10.4 cross-reference.** Dispute resolution cross-reference corrected
  from §8.10 (PRD section) to §5.9 (spec dispute resolution section).
- **OpenAPI Stage 2 endpoint.** Empty 202 response removed; `held_status`
  block (`held_until`, `reason`) folded into the 200 response, matching
  MCP `submit_stage2_data.output_schema` semantics.

### Known limitations (deferred to v0.8.2)

- **MCP `initiate_introduction` Type 3 support.** The MCP tool description
  references Type 3 referrals but the input schema only models Type 1.
  REST `POST /v1/introductions` handles both Type 1 and Type 3 via the
  `introduction_type` discriminator and is the v0.8.1 workaround for MCP
  clients needing Type 3. v0.8.2 will either split the MCP tool or add an
  `introduction_type` discriminator.
- **REST `get_consent_requirements` endpoint.** MCP-native platforms have
  the `get_consent_requirements` tool; REST-only platforms in v0.8.1
  should source consent text and required `data_categories` from spec
  §4.10 and `consent-receipt.schema.json` directly. v0.8.2 will add a
  `GET /v1/consent-requirements` endpoint with the same response shape as
  the MCP tool.

---

## [v0.8.0] — April 2026

**Status:** Release candidate. First public publication.

### Added

- **New spec Section 14 — Conformance Classes.** Four classes: Core Object,
  Binding, Registry, Governance. The reference-registry-specific behaviour is
  now distinguished from protocol-level requirements throughout.
- **New spec Section 15 — Professional Records and Registration.** Three
  record creation routes (partner-created, claimed, direct self-registered)
  and a minimum claim authentication requirement (verified email match
  against partner records or equivalent). Full claim-authentication hardening
  is on the v0.9 list.
- **Stored consent receipt as a separate schema.**
  `consent-receipt-stored.schema.json` defined as a distinct schema from
  `consent-receipt.schema.json` so the privacy boundary (no
  consumer-identifying fields, no free-text consent in the stored form) is
  structurally enforceable through schema-validation tests.
- **Schema conventions block.** Section 4.1 of the spec defines opaque
  ULID-style identifiers, RFC 3339 UTC timestamps, closed enums,
  additionalProperties policy, and schema versioning.
- **Parallel provenance map.** Per-field provenance is delivered through a
  `_provenance` map rather than wrapping individual fields. The map
  references `provenance-map.schema.json` from any schema carrying verifiable
  claims.
- **Type 2 and Type 3 introduction schemas.** Full lifecycle schemas, consent
  flows (including structured client consent receipts and structured
  professional attestations of client consent for Type 3), and state
  machines.
- **Consent receipts.** Boolean `consumer_consent` replaced with structured
  consent receipt objects following W3C Consent Receipt and Kantara CR 1.1
  patterns.
- **Tiered handoff access control.** `handoff_id` no longer grants access on
  possession alone (spec §11.5). Read-like operations available to original
  asserter or to authenticated AI platforms with matching consumer
  reference. Write operations require a `handoff_access_token` bound to
  platform/client/scope/expiry.
- **Transient encrypted vault and authenticated retrieval.** Section 11.6 of
  the spec. Personal data never transmitted through SMS or email — only
  notification metadata and authenticated retrieval references. Three
  authentication tiers (`tier_1`, `tier_2`, `tier_3`) per category
  sensitivity. Crypto-erasure as primary deletion mechanism.
- **Idempotency requirement.** All mutating endpoints require an
  `Idempotency-Key` header (spec §7.2).
- **Authorisation matrix.** Section 7.3 defines who can do what with which
  scope and object-level constraints.
- **Cancel and revoke as first-class operations.** New `cancel_introduction`
  and `revoke_consent` MCP tools and corresponding REST endpoints. New
  `get_consent_requirements` tool to support consent capture. v0.8 defines
  **nine** MCP tools (was eight in earlier drafts).
- **Per-category matching weight profiles.** Categories may define
  `matching_weight_profile` in `profession-categories.json`. Regulated
  categories weight verification quality higher than availability.
- **Review weight cap by category.** `review_signal_weight_cap` field caps
  review platform contribution to verification quality per category. ≤0.10
  for high-stakes regulated categories.
- **Anti-gaming controls on inbound referral signal.** Reciprocal cap, time
  decay, referrer verification quality weighting, evidence of successful
  outcomes for full weight, dense-cluster discounting.
- **Declared/verified scope completeness split** in verification quality
  Sub-component D. Only verified materially contributes.
- **Concern flag protections.** Renamed from "conduct flag" because ATAH
  does not adjudicate conduct. Admin-only visibility, never used in matching,
  professional notification with right of reply, pattern-based review,
  bad-faith flag detection.
- **Presentation disclosure block** in every match response. Machine-readable
  non-recommendation framing AI platforms must surface to users.
- **Opaque atah_id format.** ULID-style identifiers; country, profession,
  and category are stored as attributes rather than encoded in the ID.
- **Profile-level matching status** split: `regulatory-suspended` and
  `admin-suspended` distinguished. New states: `withdrawn`, `deceased`.
- **Location field split.** `licensed_jurisdictions`, `service_locations`,
  `physical_locations`, `remote_service_available`.
- **Custom verification dimension structure.** Required structured
  `custom_dimension` block including governance approval reference.
- **Pre-merge notification for high-risk roll-up matches.** 7-day objection
  window for regulated-tier categories.
- **Glossary section** in the spec, extended with new terms for the
  protocol-layer framing.

### Changed

- **MCP authentication updated.** OAuth 2.1-compatible authorisation;
  protected resource metadata; authorization server metadata; resource
  indicators (RFC 8707); audience validation; explicit scopes; short-lived
  access tokens with refresh and rotation; Dynamic Client Registration (RFC
  7591). Conforms to current MCP authorisation guidance at the time of
  implementation.
- **MCP tool naming.** `submit_stage2_consent` renamed to `submit_stage2_data`
  and `submit_stage3_consent` renamed to `submit_stage3_data` to reflect
  that these tools transmit both consent and data.
- **Authentication mechanisms by actor.** Partners and verifiers writing
  high-trust data now require signed JWT client assertion or HMAC-signed
  requests. API keys retained for read-only operations only.
- **Glossary section number** bumped (16 in v0.8) following the addition of
  Sections 14 and 15.

### Deferred to v0.9 / v1.0

The following are acknowledged as known v0.9 candidates rather than v0.8
omissions:

- **Federation mechanics.** Cross-registry trust, atah_id resolution across
  registries, handoff routing between registries, federated governance.
- **Claim authentication hardening.** Step-up authentication, anomaly
  detection on claim attempts, partner-known-channel notification with
  objection window, rate limiting (spec §15.6 lists the deferred set). The
  reference registry SHOULD apply these controls operationally at launch
  even though they are not yet protocol-mandated.
- **Sybil and referral-ring detection.** Beyond the v0.8 anti-gaming
  controls.
- **Platform-light consent mode.** Optional alternative to the v0.8
  self-asserted-receipt model.
- **Per-category matching weight profiles populated with researched values**
  for all launch categories. v0.8 ships with defaults plus exemplar
  overrides.
- **Full STRIDE-level threat model.** SECURITY.md ships with threat
  categories listed; full modelling deferred.
- **Conformance test suite.** Discovery endpoints (`/.well-known/atah`,
  `/v1/capabilities`) shipped; the test suite itself is v0.9.
- **Edge-case lifecycle handling.** Detailed handling for jurisdiction
  changes, partner acquisitions, verifier acquisitions, mid-flight category
  compliance changes.
- **Detailed dispute resolution process.** Beyond v0.8 baseline.
- **Full DSAR machinery for federation scenarios.**
- **Multilingual support.**

### Security

- Tiered handoff access control mitigates `handoff_id` leakage as a
  bearer-token vulnerability.
- Personal data never transmitted through notification channels (SMS, email)
  mitigates PII leakage through providers.
- Consent receipt hash storage mitigates consent forgery and platform-side
  receipt tampering.
- Authentication mechanism strengthened for high-trust partner and verifier
  writes.
- Audit log made tamper-evident (hash-chained) and retains pseudonymous
  identifiers only.

### Privacy

- Personal data never written to the main registry database.
- Notification channels carry no consumer personal data — only metadata and
  authenticated retrieval references.
- Crypto-erasure as primary deletion mechanism for the transient vault.
- Right to erasure satisfied by the transient-only design for ATAH-held
  data.

### Binding artefact status

- **`spec/v0.8/openapi.yaml`** — OpenAPI 3.1 contract for the REST binding.
  External API, Auth, and Discovery endpoints carry full operation
  definitions with examples. Professional, Partner, Independent Verifier,
  and Admin endpoints are documented at **stub level** for v0.8.0 — operation
  identifier, scopes, and key body/response refs only. Per-endpoint examples
  and full per-operation 4XX responses for those four surfaces are scheduled
  for v0.8.1.
- **`spec/v0.8/mcp-tools.json`** — MCP binding artefact: the nine v0.8 tools
  with input/output schemas (referencing the JSON Schemas via `$ref` where
  the input/output object has a published schema, and inline schemas
  otherwise), error codes, safety metadata, and at least one example per
  tool.
- **`spec/v0.8/schemas/`** — 25 JSON Schema (Draft 2020-12) files plus the
  `profession-categories.json` data file (15 launch categories).
  Cross-schema `$ref`s use relative file paths so that both ajv and Redocly
  can resolve them; the schema `$id` URLs are unchanged.

### Cross-references

- Top-level changelog: [CHANGELOG.md](../../CHANGELOG.md)
- v0.7 → v0.8 detailed change log:
  [CHANGES-FROM-V0_7-TO-V0_8.md](../../CHANGES-FROM-V0_7-TO-V0_8.md)
