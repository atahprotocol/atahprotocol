# ATAH Roadmap

This document describes where ATAH is now, what's planned next, and what's longer-term. Granular work tracking lives in [GitHub Issues](https://github.com/atahprotocol/atahprotocol/issues); this roadmap is the narrative.

## Where we are

**v0.8.2 is the release candidate.** The specification, Charter, and supporting documentation are published for technical review and reference implementation work. The aim of publishing now is to get v0.8.2 in front of standards reviewers, AI platforms, professional bodies, and prospective partners — and to iterate openly based on what they say.

The v0.8.2 publication includes:

- The full protocol specification (`spec/v0.8/full-spec.md`) covering schemas, lifecycle, MCP and REST bindings, matching, verification, transparency-as-conformance, governance commitments, and the conformance classes.
- The Charter — the public governance covenant.
- README, EXPLAINER, PRD, CHANGELOG, CONFORMANCE.md, ROADMAP.md.
- The full set of JSON Schema files plus `openapi.yaml` and `mcp-tools.json`.
- The stress-test matrix as the published verification artifact for the release.
- GOVERNANCE.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, LICENSE.

## Release strategy

- **v0.8.2.x — patch-level fixes.** Editorial corrections, schema clarifications, minor inconsistencies, and small spec amendments that don't change semantics are reviewed for incorporation in subsequent patch versions as community review surfaces issues suitable for patch-level treatment. There is no committed cadence; patches land when meaningful work has accumulated.
- **v0.8.3 — substantive next patch round.** Items listed below.
- **v0.9 — substantive next release.** Items listed below. Timing depends on feedback volume, partner integration progress, and the resources available to pick up deferred work.
- **v1.0 — stable.** When at least one third-party conforming implementation exists, governance has transitioned to an independent structure per the Charter triggers, and the deferred items below are addressed (or deliberately deferred further with stated reasons).

Breaking changes are possible up to v1.0; the version negotiation and deprecation policy in [spec Section 7.1](spec/v0.8/full-spec.md#71-versioning-and-backward-compatibility) governs how those changes are managed.

## v0.8.3 candidates

Items where the v0.8.2 spec defines the obligation but the endpoint implementation MAY follow in v0.8.3, plus follow-on normative work surfaced during the v0.8.2 cycle:

- **Professional-facing visibility-explanation endpoint.** `GET /v1/professionals/me/visibility-explanations` (REST) and `get_my_visibility_explanations` (MCP). Spec §11A.4 defines the rules-derived behaviour, required fields, authority controls, audit linkage, and the MUST NOT rule on query-history exposure. Implementations MAY declare `x-implementation-deferred-to: v0.8.3` and return 501 in v0.8.2. The obligation itself is part of v0.8.2; only the endpoint implementation is permitted to defer.
- **Normative computation algorithms for band-input signals.** v0.8.2 normatively defines the band-assignment logic, the stratified-randomisation rule within bands, and the structural constraint that band assignment must not reference commercial signals. Computation algorithms for `category_fit`, `availability_score`, and contact-freshness scoring are not normatively specified at v0.8.2 — implementations document their own approach in their conformance statement. v0.8.3 will specify normative computation algorithms for these signals.
- **Conditional `consent_receipt_id` requirements by `request_intent` in Discovery.** v0.8.2 requires `consent_receipt_id` on every Discovery query. Conditional requirements that vary by `request_intent` (e.g. narrower scope for `on_behalf_of_client` queries acting under a professional's own delegated authority) are targeted for v0.8.3 refinement.
- **Engagement liveness (response-rate tracking).** Stress-test scenario 7.3. Phase 7 covers contact-channel verification only; response-rate tracking is `deferred-to-v0.8.3`.
- **Withdrawal-cycle harassment monitoring.** Stress-test scenario 6.1 — malicious actor initiates and withdraws repeated handoffs. Withdrawal-as-state-transition plus audit retention closes the structural side; concrete rate-limit thresholds and pattern detection on cancel/withdraw cycles are `deferred-to-v0.8.3`.
- **Stress-testing as ongoing publication discipline.** The [stress-test matrix](spec/v0.8/stress-test-matrix.md) is the verification artifact for v0.8.2 publication. Stress-testing is a publication discipline, not a one-off artifact. v0.8.3 candidate workstream: matrix maintenance — adding scenarios as they emerge from implementation experience, peer review, or partner conversations; reviewing previously-resolved scenarios for status drift; promoting `partially-resolved` scenarios to `resolved` as their residual work completes. The matrix is a living verification artifact; updates land as patch releases.

## v0.8.2.x candidates

Items expected to be small enough to land as patch releases rather than wait for v0.8.3 or v0.9:

- Editorial corrections from community review
- Schema clarifications where wording has been ambiguous in practice
- Cross-document consistency fixes surfaced after publication
- Repository directory restructure into `core/`, `bindings/`, `registry-profile/` subdirectories — deferred to avoid schema-reference reflow risk pre-publication

These are best-effort; patch versions land when meaningful work has accumulated rather than on a fixed cadence.

## v0.9 candidates

These are acknowledged in the [CHANGELOG](CHANGELOG.md) as known v0.9 candidates rather than v0.8 omissions:

### Federation mechanics

Cross-registry trust, atah_id resolution across registries, handoff routing between registries, federated governance, registry namespace resolution, duplicate professional records, consent receipt portability, registry operator approval and audit. The v0.8 architecture is federation-ready (atah_id namespace prefixes reserved, no single-registry assumption in the data model), but the cross-registry trust mechanics are not yet specified.

### Claim authentication hardening

v0.8 specifies the minimum claim authentication requirement (verified email match against partner records or equivalent partner-attested identifier). v0.9 will specify the fuller hardening: step-up authentication for claims, anomaly detection on claim attempts (rapid succession from new IPs, attempts to immediately edit fee or contact fields after claim), notification to the partner-known channel with an objection window before claim takes effect, and rate limiting on claim attempts per record and per IP.

The reference registry is expected to apply these controls operationally at launch even though they are not yet protocol-mandated for all conforming registries.

### Platform-light consent mode

In v0.8, the asserting AI platform is responsible for capturing structured consent and producing the consent receipt; ATAH stores the hash and metadata. Some AI platforms have indicated they would prefer a model where ATAH provides more of the consent ceremony — for example, a canonical-consent-text endpoint plus a consent-flow-orchestration helper — so the platform invokes a standard ATAH consent flow rather than constructing receipts itself. This would reduce the platform's interpretation burden without changing where legal responsibility sits.

v0.9 will specify an optional platform-light consent mode that platforms can adopt as an alternative to the v0.8 self-asserted-receipt model. The v0.8 self-asserted model continues to be supported.

### Per-category matching weight profiles populated with researched values

v0.8 ships with default weights plus exemplar overrides for representative regulated and non-regulated categories. v0.9 will add researched per-category weights for all launch categories, requiring domain expertise the v0.8 work didn't have.

### Full STRIDE-level threat model

v0.8 SECURITY.md ships with threat categories listed. The full STRIDE-level threat model — spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege — is deferred to v0.9.

### Conformance test suite

v0.8 ships the discovery endpoints (`/.well-known/atah`, `/.well-known/atah-conformance`, `/v1/capabilities`). The conformance test suite — schema-validating, lifecycle-exercising, behaviour-confirming — is v0.9.

### Finer-grained conformance profiles

v0.8 primarily specifies the full reference registry profile. Finer-grained conformance profiles (a `profile-provider` profile, a `matching-registry` profile distinct from a `full-handoff-registry` profile) are v0.9 candidates.

### Edge-case lifecycle handling

Detailed handling for jurisdiction changes, partner acquisitions, verifier acquisitions, mid-flight category compliance changes, and similar lifecycle edges that are predictable but not yet specified.

### Detailed dispute resolution process

Beyond the v0.8 baseline, including timelines, escalation criteria, evidence handling, and resolution publication.

### Full DSAR machinery for federation scenarios

v0.8 satisfies right-to-erasure through the transient-only design for ATAH-held data. Full Data Subject Access Request machinery for cross-registry scenarios (where a consumer's data may have passed through multiple registries) is deferred to v0.9 alongside federation.

### Multilingual support

v0.8 ships in English. Multilingual category metadata, multilingual consent text versioning, and multilingual conformance-statement requirements are deferred to v0.9.

### Behavioural-neutrality and conformance-audit model

v0.8.2 introduces a three-level conformance-status distinction in [`CONFORMANCE.md`](CONFORMANCE.md): protocol-compatible / ATAH-conformant / ATAH-recognised neutral implementation. The naming is provisional; the distinction is locked. v0.8.2's contribution is the framing. v0.9 operationalises the verification regime through a behavioural-neutrality / conformance-audit model. Items the model should consider:

- Public ordering-policy metadata exposure tests — the response-level `presentation_disclosure.ordering_policy.mode` and `commercial_weighting: false` `const` are checkable today; v0.9 extends this to operational metadata covering ordering decisions across query volume.
- Decision-explanation object validation across required response shapes — the Phase 6 Transparency Class makes `decision_explanation` required at response and per-candidate level; v0.9 audits whether implementations produce explanations that are structurally valid AND substantively non-trivial (i.e. not stub responses).
- Commercial-neutrality attestation — annual signed attestation by the implementing organisation that no commercial weighting was applied in matching outcomes during the audit window, with auditable evidence on request.
- Auditability of matching / banding behaviour — sampled query replay against the documented `band_definitions` and within-band fairness policy to verify observable non-determinism and band-rule consistency.
- Public partner and verifier scope registry — the `verification-scope.schema.json` data is structured today; v0.9 publishes a public read-only channel showing approved partners, scopes, and freshness commitments.
- Disclosed conflict-of-interest policies — Charter §7 / GOVERNANCE.md §4.1 require related-party disclosure today; v0.9 publishes the disclosed register on a regular cadence with audit evidence.
- Revocable conformance mark issuance and revocation procedure — the mark is revocable per CHARTER Part Two; v0.9 specifies the issuance criteria, monitoring cadence, the published-revocation process, and the remediation path back to compliance.
- Right of ATAH governance to publicly contest misleading claims — added to GOVERNANCE.md §5.1 in v0.8.2 as a stated commitment; v0.9 may formalise the public-statement template and the response window.

The audit regime should not require public release of proprietary scoring code from any conforming implementation. It should require enough observable metadata, audit output, and governance disclosure to make hidden commercial weighting harder to hide behind the protocol brand. The goal is **not** to stop forks of the open Apache 2.0 protocol — those are permitted by the licence and contribute to the protocol's resilience. The goal is to preserve the meaning of official ATAH conformance, neutrality, and recognised-implementation status.

### `presentation_disclosure.verification_tier` field for consumer-facing rendering

Brand-dilution mitigation requires AI platforms to surface the credentialled-vs-established verification-tier distinction to consumers. v0.8.2 carries the structural distinction through the existing `professional_tier` field on professional-identity schemas, the `_provenance` map, and the Transparency Class. A dedicated `presentation_disclosure.verification_tier` field with values like `regulator_verified` / `body_verified` / `self_declared` would let AI platforms render the distinction prominently without inferring it from the underlying schema. Decision deferred to v0.9 — review whether the existing structural distinction (Transparency Class plus `professional_tier`) is operationally sufficient for consistent consumer-facing rendering across AI-platform UX, or whether a dedicated `presentation_disclosure.verification_tier` field is needed to harden the rendering obligation.

## v1.0 target

v1.0 is the stable target. The criteria for declaring v1.0 are not yet final, but the working list is:

- At least one external conforming implementation exists and is operational
- Federation mechanics are specified and at least one federated registry pair operates conformantly
- Governance has transitioned to an independent structure (Charter §Part Three)
- A conformance test suite is published and used
- The major deferred-to-v0.9 items above are either addressed or deliberately deferred further with stated reasons
- The Charter has been reviewed by external counsel and any structural amendments needed for v1.0 are made

v1.0 may iterate through v0.9.x and v1.0-rc steps before stabilising.

## How to engage

If you want to influence the roadmap:

- **Propose a specific change** — open an issue describing it. Use issue templates if provided.
- **Pick up a v0.9 candidate** — comment on the relevant issue. Larger items are likely to need a small working group.
- **Critique a v0.8 decision** — open a discussion or issue. Architectural decisions made in the v0.8 cycle are documented; the case for any change should engage with the original reasoning.
- **Contribute a conforming implementation** — see [CONTRIBUTING.md](CONTRIBUTING.md) and the conformance requirements in [CONFORMANCE.md](CONFORMANCE.md).

For governance-level engagement (Charter changes, transition timing, partnership terms), see [GOVERNANCE.md](GOVERNANCE.md).

For security disclosures, follow the process in [SECURITY.md](SECURITY.md). Roadmap items affecting security posture are tracked alongside other v0.9 work but security disclosures themselves are not handled through public roadmap channels.
