# ATAH Conformance

ATAH conformance is the preservation of mandatory protocol semantics, not the reuse of any particular codebase or infrastructure. An implementation conforms by producing and consuming ATAH-valid objects, preserving the protocol's privacy and commercial-neutrality requirements, and publishing a discoverable conformance statement.

This document summarises the conformance model. The full normative text is in the specification's [Section 14](spec/v0.8/full-spec.md#14-conformance-classes).

## Four conformance classes

ATAH v0.8 defines four conformance classes. Implementations may declare which classes they support — the classes are layered rather than hierarchical, so an implementation can claim some without claiming all.

### Core Object Conformance

An implementation conforms at the core-object level if it produces and consumes ATAH schema-valid objects: professional records, provenance maps, consent receipt references, handoff records (Type 1, 2, and 3), match responses, presentation disclosures, outcome reports, and audit events.

This is the minimum a non-registry implementation may claim. Examples: a partner-side tool that prepares ATAH-valid data for submission, an analytics tool that consumes ATAH match-response payloads, a compliance tool that audits ATAH-formatted records.

### Binding Conformance

A binding conforms if it exposes ATAH operations through a defined interface — such as MCP or REST — while preserving core protocol semantics. Bindings do not change which fields are required, which provenance must be returned, or which consent stages apply. They change only how those operations are invoked.

v0.8 defines two reference bindings: the MCP binding and the REST binding. A future implementation may define additional bindings (A2A-compatible, push, federation, webhook) and claim binding conformance for the binding(s) it implements.

### Registry Conformance

A registry conforms if it stores or resolves professional records under an `atah_id` namespace, preserves field-level provenance via the `_provenance` map, enforces category compliance gating, applies no commercial weighting in matching, supports record correction and dispute flows, returns ATAH-valid match responses with the required `presentation_disclosure` block, supports consent receipt references in introduction creation, manages handoff state if it supports introductions, implements data minimisation, and publishes a conformance statement.

### Governance Conformance

A governed implementation conforms if it publishes its partner approval criteria, verifier approval criteria, review-platform classification criteria, fee principles (with cost-recovery rationale and waivers for public-interest organisations), the process for correcting inaccurate professional data, and the process for handling concern flags routed to it.

Governance conformance is what allows downstream parties — AI platforms, professional bodies, regulators — to assess whether an implementation operates the protocol on terms consistent with the Charter Part One core commitments.

## Reference Registry Scope

The initial ATAH reference registry is expected to conform to all four classes. Future implementations may declare narrower conformance — for example, a profile-only implementation that supports core object and binding conformance but does not run the introduction lifecycle.

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

## Where to find more

For the full normative conformance specification, see [spec/v0.8/full-spec.md, Section 14](spec/v0.8/full-spec.md#14-conformance-classes).

For the architectural framing — what counts as the protocol versus a binding versus a registry profile versus an implementation — see [spec/v0.8/full-spec.md, Section 1.5](spec/v0.8/full-spec.md#15-what-atah-is-and-is-not).

For the governance commitments that conformance is anchored to, see [CHARTER.md](CHARTER.md).

For roadmap items that affect conformance (full conformance test suite, finer-grained conformance profiles, federation conformance), see [ROADMAP.md](ROADMAP.md).
