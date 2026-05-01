# ATAH Roadmap

This document describes where ATAH is now, what's planned next, and what's longer-term. Granular work tracking lives in [GitHub Issues](https://github.com/atahprotocol/atahprotocol/issues); this roadmap is the narrative.

## Where we are

**v0.8 is the release candidate.** The specification, Charter, and supporting documentation are published for technical review and reference implementation work. The aim of publishing now is to get v0.8 in front of standards reviewers, AI platforms, professional bodies, and prospective partners — and to iterate openly based on what they say.

The v0.8 publication includes:

- The full protocol specification (`spec/v0.8/full-spec.md`) covering schemas, lifecycle, MCP and REST bindings, matching, verification, governance commitments, and four conformance classes.
- The Charter (v1.2) — the public governance covenant.
- README, EXPLAINER, PRD, CHANGELOG, CHANGES-FROM-V0_7-TO-V0_8, CONFORMANCE.md, ROADMAP.md.
- The full set of JSON Schema files plus `openapi.yaml` and `mcp-tools.json`.
- GOVERNANCE.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, LICENSE.

## Release strategy

v0.8.1 is published as the release-candidate version. The release plan handles ongoing iteration openly:

- **v0.8.1.x — patch-level fixes.** Editorial corrections, schema clarifications, minor inconsistencies, and small spec amendments that don't change semantics may be incorporated in patch versions as feedback arrives and time and resources allow. There is no committed cadence; patches land when meaningful work has accumulated.
- **v0.9 — substantive next release.** Items listed below. Timing depends on feedback volume, partner integration progress, and the resources available to pick up deferred work.
- **v1.0 — stable.** When at least one third-party conforming implementation exists, governance has transitioned to an independent structure per the Charter triggers, and the deferred items below are addressed (or deliberately deferred further with stated reasons).

Breaking changes are possible up to v1.0; the version negotiation and deprecation policy in [spec Section 7.1](spec/v0.8/full-spec.md#71-versioning-and-backward-compatibility) governs how those changes are managed.

## v0.8.1.x and v0.8.2 candidates

Items expected to be small enough to land as patch releases rather than wait for v0.9:

- Editorial corrections from peer review
- Schema clarifications where wording has been ambiguous in practice
- Cross-document consistency fixes surfaced after publication
- Repository directory restructure into `core/`, `bindings/`, `registry-profile/` subdirectories — deferred from v0.8.1 to avoid schema-reference reflow risk pre-publication
- **`verification_confidence` schema implementation.** Add the field to `match-response.schema.json` with the enumeration and semantics specified in spec §4.12. Update matching engine logic to populate the field. Add validation tests for each enumeration value. Update example payloads in `examples/`. (Field name and semantics committed in spec §4.12 at v0.8.1; schema implementation lands in v0.8.2.)

These are best-effort; patch versions land when meaningful work has accumulated rather than on a fixed cadence.

## v0.9 candidates

These are acknowledged in the [CHANGELOG](CHANGELOG.md) as known v0.9 candidates rather than v0.8 omissions:

### Federation mechanics

Cross-registry trust, atah_id resolution across registries, handoff routing between registries, federated governance, registry namespace resolution, duplicate professional records, consent receipt portability, registry operator approval and audit. The v0.8 architecture is federation-ready (atah_id namespace prefixes reserved, no single-registry assumption in the data model), but the cross-registry trust mechanics are not yet specified.

### Claim authentication hardening

v0.8 specifies the minimum claim authentication requirement (verified email match against partner records or equivalent partner-attested identifier). v0.9 will specify the fuller hardening: step-up authentication for claims, anomaly detection on claim attempts (rapid succession from new IPs, attempts to immediately edit fee or contact fields after claim), notification to the partner-known channel with an objection window before claim takes effect, and rate limiting on claim attempts per record and per IP.

The reference registry is expected to apply these controls operationally at launch even though they are not yet protocol-mandated for all conforming registries.

### Sybil and referral-ring detection

Beyond the v0.8 anti-gaming controls (reciprocal cap, time decay, referrer verification quality weighting, evidence of successful referral outcomes for full weight, dense-cluster discounting), v0.9 will specify more sophisticated sybil and referral-ring detection.

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
