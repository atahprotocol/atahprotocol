# ATAH Conformance

ATAH conformance is the preservation of mandatory protocol semantics, not the reuse of any particular codebase or infrastructure. An implementation conforms by producing and consuming ATAH-valid objects, preserving the protocol's privacy and commercial-neutrality requirements, and publishing a discoverable conformance statement.

This document summarises the conformance model. The full normative text is in the specification's [Section 14](spec/v0.8/full-spec.md#14-conformance-classes).

## Five conformance classes

ATAH v0.8.2 defines five conformance classes (the v0.8.1 four-class set plus the new Transparency Class introduced in Phase 6). Implementations may declare which classes they support — the classes are layered rather than hierarchical, so an implementation can claim some without claiming all.

### Core Object Conformance

An implementation conforms at the core-object level if it produces and consumes ATAH schema-valid objects: professional records, provenance maps, consent receipt references, Component 2 handoff records, Component 3 referral-proposal and referral-connection records, match responses, presentation disclosures, decision-explanation objects, outcome reports, and audit events.

This is the minimum a non-registry implementation may claim. Examples: a partner-side tool that prepares ATAH-valid data for submission, an analytics tool that consumes ATAH match-response payloads, a compliance tool that audits ATAH-formatted records.

### Binding Conformance

A binding conforms if it exposes ATAH operations through a defined interface — such as MCP or REST — while preserving core protocol semantics. Bindings do not change which fields are required, which provenance must be returned, or which consent stages apply. They change only how those operations are invoked.

v0.8 defines two reference bindings: the MCP binding and the REST binding. A future implementation may define additional bindings (A2A-compatible, push, federation, webhook) and claim binding conformance for the binding(s) it implements.

### Registry Conformance

A registry conforms if it stores or resolves professional records under an `atah_id` namespace, preserves field-level provenance via the `_provenance` map, enforces category compliance gating, applies no commercial weighting in matching, supports record correction and dispute flows, returns ATAH-valid match responses with the required `presentation_disclosure` block, supports consent receipt references in introduction creation, manages handoff state if it supports introductions, implements data minimisation, and publishes a conformance statement.

### Governance Conformance

A governed implementation conforms if it publishes its partner approval criteria, verifier approval criteria, review-platform classification criteria, fee principles (with cost-recovery rationale and waivers for public-interest organisations), the process for correcting inaccurate professional data, and the process for handling concern flags routed to it.

Governance conformance is what allows downstream parties — AI platforms, professional bodies, regulators — to assess whether an implementation operates the protocol on terms consistent with the Charter Part One core commitments.

### Transparency Conformance (new in v0.8.2)

Per spec §11A and Paolo's F1.8 / F1.9 normative position, transparency is a top-level conformance requirement. An implementation conforms at the transparency level if every relevant response carries a valid response-level `decision_explanation` (Layer 1); every candidate in a Discovery response carries a valid per-candidate `decision_explanation` (Layer 2); every Discovery response with exclusions carries a valid `exclusion_summary` with aggregated reason categories (Layer 3); the professional-facing visibility-explanation behaviour (per §11A.4 / F-18 — rules-derived, NOT query-history-derived) is implementable from the v0.8.2 spec (whether the dedicated endpoint is implemented in v0.8.2 or follows in v0.8.3); the `rules_applied` list is non-empty for inclusion decisions; exclusion responses include reason categories; and the `band_definitions` for the implementation are publicly accessible.

> **MUST.** Conforming implementations MUST produce machine-readable explanations for discovery, exclusion, ordering, handoff, withdrawal, suppression, and data-sharing events, subject to privacy, security, and anti-gaming limits.

> **MUST NOT.** Professional-facing visibility explanations MUST NOT expose actual query-count data, query-history data, observed demand patterns, or any per-query information derived from consumer or platform traffic. They MUST present representative explanation categories at the category/jurisdiction level, derived from the implementation's documented rules and the professional's own profile data, not from observed query traffic.

The conformance test for the Transparency Class (formalised in the v0.9 conformance test suite) picks three sample queries — one inclusion-only, one with exclusions, one with explicit `ordering_preference` — and verifies each response includes both layers of `decision_explanation` and (where applicable) `exclusion_summary`, with contents internally consistent with the implementation's published rules.

## Three governance layers

Distinct from the five conformance classes (which describe what an implementation must do), the governance model has three layers that describe who runs each part on what terms (per Charter Core Commitment 8 and Paolo Piponi's F1.6 peer-review finding):

- **Protocol governance body.** ATAH protocol governance MUST be held by an independent not-for-profit or equivalent public-interest entity. Owns the specification, conformance marks, category annex process, partner / verifier admission rules, transparency rules, and neutrality audits. Form is mandated.
- **Reference registry operator.** May be not-for-profit, public-benefit, community-interest company, foundation-owned subsidiary, or commercial under strict constraints. Critical requirements: enforceable neutrality, public fee schedules, auditability, data-use limits, structural separation from the protocol governance body. Form is constrained, not mandated.
- **Third-party conforming implementations.** May be commercial, public-sector, professional-body, or not-for-profit, provided they meet conformance requirements (the five classes above). Form is unconstrained.

The protocol governance body holds the conformance marks; conforming implementations earn them through self-declaration during the release-candidate stage and through the conformance test suite (v0.9). Conformance is uniform across implementations of any form; governance form is uniform only at the protocol layer.

Note that the three governance layers above are a different axis from the Phase 10A conformance-status distinction (protocol-compatible / ATAH-conformant / ATAH-recognised neutral implementation — see Phase 10A in the master patch plan and ROADMAP for v0.9 work). Governance layers answer "what kind of actor or layer is this?"; conformance status answers "what level of recognition does this implementation have?"

## Reference Registry Scope

The initial ATAH reference registry is expected to conform to all five classes. Future implementations may declare narrower conformance — for example, a profile-only implementation that supports core object and binding conformance but does not run the introduction lifecycle.

Finer-grained conformance profiles (such as a `profile-provider` profile or a `matching-registry` profile distinct from a `full-handoff-registry` profile) are a v0.9 candidate. v0.8 primarily specifies the full reference registry profile.

## Conformance Statement Requirement

A conforming implementation publishes a public conformance statement at a discoverable location. The recommended location is `/.well-known/atah-conformance`. The statement lists:

- protocol version supported
- schema version supported
- bindings supported (with binding versions)
- conformance classes claimed
- categories supported
- jurisdictions supported
- governance documents URL (charter or equivalent)
- contact for conformance enquiries

The reference registry publishes its conformance statement as part of the v0.8 launch package.

**Self-certification status until v0.9.** Until the v0.9 conformance test
suite is published, conformance statements are self-declarations by their
publishers. They are not ATAH certification or endorsement of any
implementation. ATAH may publicly contest conformance statements that
misrepresent compliance with the protocol or with the conformance principles
in §14 of the specification.

## Record portability across registries

Record portability between conforming registries — including export format, duplicate resolution, and cross-registry provenance trust — is deferred to federation work in v0.9 or v1.0. The v0.8 architecture is federation-ready (atah_id namespaces are reserved, no part of the data model assumes single-registry operation), but the cross-registry trust mechanics are not yet specified.

## Self-declaration during release-candidate stage

Until an official conformance test suite or certification process exists, conformance statements made by implementers are self-declarations by the implementing party. ATAH does not currently issue conformance certifications, conformance badges, or formal endorsements; the v0.9 conformance test suite is the planned mechanism for that future state (see ROADMAP.md).

ATAH may publicly challenge or correct misleading claims of conformance, certification, endorsement, or official status. Implementers making conformance statements should accurately describe what their implementation does, against which version of the specification, and which conformance principles their implementation preserves. Conforming implementations are encouraged to publish a conformance statement at a discoverable location (recommended: `/.well-known/atah-conformance`) detailing protocol version, schema version, bindings supported, conformance classes claimed, categories supported, jurisdictions supported, and contact for conformance enquiries.

## Where to find more

For the full normative conformance specification, see [spec/v0.8/full-spec.md, Section 14](spec/v0.8/full-spec.md#14-conformance-classes).

For the architectural framing — what counts as the protocol versus a binding versus a registry profile versus an implementation — see [spec/v0.8/full-spec.md, Section 1.5](spec/v0.8/full-spec.md#15-what-atah-is-and-is-not).

For the governance commitments that conformance is anchored to, see [CHARTER.md](CHARTER.md).

For roadmap items that affect conformance (full conformance test suite, finer-grained conformance profiles, federation conformance), see [ROADMAP.md](ROADMAP.md).
