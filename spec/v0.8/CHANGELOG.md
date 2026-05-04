# Spec v0.8 Changelog

This is the spec-specific changelog for v0.8. The top-level
[CHANGELOG.md](../../CHANGELOG.md) covers repository-wide changes including
governance and supporting documentation; this file mirrors the entries that
affect the specification, schemas, OpenAPI contract, and MCP binding
artefacts.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
Breaking changes are possible until v1.0.

---

## [v0.8.2] — May 2026

**Status:** Initial public release. The release-candidate specification, schemas,
OpenAPI contract, and MCP binding are published for technical review and reference
implementation work.

### Spec surfaces

- **Schemas (`schemas/`).** 25 JSON Schema files covering professional identity
  (credentialled and established tiers), provenance map, query and match-response,
  consent receipts (full and stored forms), principal/delegation, handoff-component2
  with `flow_variant` (`off_protocol` | `contact_share` | `full_lifecycle`), stage-2
  pre-handoff and stage-3 contact transient-vault payloads, trusted-partner and
  partner-data-push, verification-scope, profession-category metadata and the
  `profession-categories.json` data file with band definitions, independent-verifier
  and enhanced-verification, review-platform, conflict-record and dispute-record and
  concern-flag-record, rollup-match-record, audit-event, decision-explanation, and
  error.
- **Full specification (`full-spec.md`).** Core data schemas, principal/delegation
  model (§4.9A), consent receipt object (§4.10), match response schema (§4.12),
  trusted partner model (§5), introduction lifecycle (§6), API endpoints with
  authorisation matrix (§7), MCP server specification (§8), matching engine
  pipeline (§9), verification architecture (§10), data governance and the
  three-concept separation (§11), transparency and explainability (§11A),
  notification service (§12), contact-detail freshness mechanism (§12A),
  security model (§13), conformance classes (§14), professional records and
  registration (§15), repository structure (§17).
- **OpenAPI contract (`openapi.yaml`).** External / consumer-facing endpoints,
  professional endpoints, partner endpoints, verifier endpoints, admin endpoints,
  and well-known discovery endpoints. OAuth-scope-to-authority-basis mapping
  documented in `securitySchemes.oauth2.description`.
- **MCP binding (`mcp-tools.json`).** 14 tools exposing the protocol operations,
  each declaring `permitted_authority_bases`.
- **Stress-test matrix (`stress-test-matrix.md`).** The published verification
  artifact for v0.8.2 publication.
- **Spectral ruleset (`.spectral.yaml`).** Extends `spectral:oas` so OpenAPI lint
  is a uniform invocation; `oas3-valid-media-example` downgraded to `warn` per
  the F1.4 deferral; `operation-description` disabled per the F1.7 deferral.

### Spec features

- **Discovery query.** `query.schema.json` carries required `request_intent`
  (`self`, `on_behalf_of_client`) and required `limit` (1–100). `request_intent`
  gates whether Component 2 is available next.
- **Matching engine (§9.1).** Four-step pipeline: hard filters → band assignment
  → threshold exclusion → stratified-random ordering within bands. No global
  match score. Within-band ordering is observably non-deterministic; the
  randomisation seed is not disclosed.
- **Review-signal band cap (§9.2).** `profession-category.schema.json` carries
  `review_signal_band_cap`; for high-stakes regulated categories review-derived
  signals MUST NOT move a candidate into a higher band unless corroborated by
  authoritative credential or regulator-source evidence.
- **AI platform presentation obligation (§9.4).** AI platforms reordering the
  ATAH response MUST preserve the `presentation_disclosure.ordering_policy`
  field verbatim and surface its content to the user. Failure is a
  binding-conformance failure.
- **Transparency obligations (§11A).** Conforming implementations MUST produce
  machine-readable explanations for discovery, exclusion, ordering, handoff,
  withdrawal, suppression, and data-sharing events. Three transparency layers
  in match responses: response-level `decision_explanation`, per-candidate
  `decision_explanation`, and aggregate `exclusion_summary`. Named-excluded-
  candidate detail is available only through governance / auditor retrieval
  (`GET /v1/decision-explanations/{audit_event_id}`). Professional-facing
  visibility view is rules-derived per the §11A.4 MUST NOT rule, not derived
  from observed query traffic.
- **Three-concept separation (§11).** Payload erasure, audit retention, and
  withdrawal-as-state-transition are deliberately separate operations on
  data, each with its own rules and retention semantics.
- **Five withdrawal scenarios (§11.9).** Consumer withdrawal before data
  sharing; consumer withdrawal after Stage 2 viewed; consumer withdrawal after
  Stage 3 retrieval; professional withdrawal from an introduction; professional
  withdrawal from matching. High-impact withdrawals require step-up
  authentication at the point of action.
- **Contact-detail freshness mechanism (§12A).** Three-layer model — periodic
  verification on a per-category cadence, escalation when challenges go
  unanswered, and a just-in-time check before high-stakes introductions. Stale
  channels flip the professional's `matching_status` to `contact_unverified`
  and exclude them from Discovery results until re-verified.
- **Two consent types, one excluded by design (§4.10).** `query_authorization`
  and `disclosure_consent`. Engagement consent is outside ATAH by design. ATAH
  consent receipts MUST NOT be represented as professional engagement consent.
  Continuity binding (HMAC of platform-side consumer reference, hash of
  canonical `data_categories` array) prevents receipt replay and cross-user
  confusion.
- **Five conformance classes (§14).** Core Object, Binding, Registry, Governance,
  Transparency.

### Deferred

- **Professional-facing visibility-explanation endpoint** — implementation MAY
  defer to v0.8.3 (the obligation is part of v0.8.2 per §11A.4).
- **Normative computation algorithms for band-input signals** (`category_fit`,
  `availability_score`, contact-freshness scoring) — targeted for v0.8.3.
- **Conditional `consent_receipt_id` requirements by `request_intent`** —
  targeted for v0.8.3.
- **Engagement liveness response-rate tracking** — targeted for v0.8.3.
- **Withdrawal-cycle harassment monitoring rate-limit thresholds** — targeted
  for v0.8.3.
- **Anti-phishing format for verification challenges** — targeted for v0.8.3 / v0.9.
- **F1.4 Spectral resolver tooling-layer issue** on `MatchResponseExample` —
  rule downgraded to `warn`; targeted for v0.8.3.
- **F1.7 OpenAPI operation-description stubs** — rule disabled; descriptions
  best written with implementation experience to draw on; targeted for v0.8.3.
- **Behavioural-neutrality / conformance-audit operational regime** — v0.8.2
  ships the three-level conformance-status framing; v0.9 operationalises the
  verification regime.
- **Federation mechanics** — v0.8.2 architecture is federation-ready;
  cross-registry trust mechanics are v0.9 / v1.0 work.
- **`presentation_disclosure.verification_tier` field** — v0.9 reviews whether
  a dedicated presentation field is needed.
