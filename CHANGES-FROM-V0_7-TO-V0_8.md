# Changes from v0.7 to v0.8

This document is a file-by-file change log between the v0.7 internal package and the v0.8 release-candidate publication. It exists so reviewers can quickly see what moved, what stayed, and where the substantive work happened, without diffing every file by hand.

For the high-level release notes, see [CHANGELOG.md](CHANGELOG.md).

The v0.8 package incorporates pre-publication peer review remediation. The protocol's purpose, actors, and lifecycle are unchanged from v0.7. v0.8 tightens, completes, and clarifies what was previously implied or under-specified.

## Architecture review note (final)

After further architecture review prior to publication, v0.8 was sharpened to distinguish the **Core Protocol**, **Bindings**, the **Registry Operation Profile**, and the **Reference Registry** as four distinct things — three specification surfaces (the protocol's transport-neutral objects, the bindings that expose it, and the behaviours required of a conforming registry) plus the first operational implementation. This did not change the protocol purpose, the introduction lifecycle, the privacy architecture, the trust model, or the governance commitments. It clarified adoption boundaries and made the package easier for AI platforms, professional bodies, and third-party implementers to evaluate.

The substantive refinements introduced by this review are:

- the spec now contains an explicit Conformance Classes section (Core Object, Binding, Registry, Governance);
- the spec now contains an explicit Professional Records and Registration section (partner-created, claimed, direct self-registered);
- the Charter (v1.2) makes royalty-free protocol implementation explicit, and scopes the Open MCP endpoint commitment to the ATAH-operated reference endpoint;
- README, EXPLAINER, PRD, and spec language consistently distinguishes "the protocol" from "the reference registry";
- MCP and REST are explicitly framed as v0.8 *bindings* of a transport-neutral core protocol, rather than as the protocol itself.

The reference registry remains the first operational implementation and is essential for v0.8 launch. The change is to make clear that it is the first implementation, not the boundary of the protocol.

---

## CHARTER.md (v1.0 → v1.2)

**Lines: 105 → 205**

### Changed in v1.2 (final pre-publication)

- **Open specification commitment** clarified: protocol implementation is explicitly royalty-free; commercial charges may exist only at the implementation or operational layer.
- **Open MCP endpoint commitment** scoped: the ATAH-operated reference MCP endpoint remains free; the protocol itself is royalty-free; future third-party conforming implementations may operate their own endpoints subject to conformance principles, but may not use commercial payment to influence matching or provenance.
- Added scope clarification that registry-operations references in the Charter apply to the ATAH-operated reference registry.

### Changed in v1.1

- Reframed the Charter explicitly as a public governance covenant the founder undertakes to administer. Removed claims of legal "binding" without legal machinery in place; flagged for legal review before v1.0.
- Strengthened the amendment process. Pre-transition: founder + interim advisory group with 30-day public notice. Post-transition: 75% supermajority of independent governance body with 30-day public notice. Founder advisory seat carries consultation rights but not approval rights for core amendments after transition.

### Added — Part One (Core commitments)

- **New core commitment 5: Privacy floor.** ATAH must not operate as a central repository of consumer personal data. Notification channels carry no PII.
- **New core commitment 6: Compliance gating floor.** No professional category surfaced in matching without an approved compliance annex.
- **Founder conflict-of-interest rules.** Annual public disclosure; recusal from decisions affecting founder-affiliated entities; public register of related-party transactions; no privileged access to non-public data.
- **Founder seat post-transition rules.** No nominee with approval rights over core amendments.

### Added — Part Two (Operational commitments)

- **Concrete transition covenant.** 90-day interim governance plan; 12-month transition deadline or earlier on triggers (two trusted partner integrations + one external conforming implementation); public-explanation safety valve for any deadline revision.
- **Interim advisory structure.** Required consultation before approving the first verifier, review platform, partner agreement, category compliance annex, or any Charter change.
- **Partner principles.** Published fee schedule; cost-recovery rationale; fee waivers for regulators and public-interest organisations; no exclusivity; public partner registry.
- **Professional participation principles.** Hardship waiver process; public fee cap; free listing for categories without partner coverage; no ranking difference between paid individual and partner-route listings.
- **Verifier and review platform principles.** Public acceptance criteria; annual re-attestation; audit rights; suspension criteria; public classification rationale; verifiers may not also operate as professional bodies for categories they verify; change-of-control notification.
- **Privacy and data minimisation operational commitments.** Implements the Part One privacy floor.
- **Accessibility commitment.**

---

## spec-v0.8-full.md (1,317 → 2,017 lines)

### New sections (final pre-publication review)

- **Section 1.5 — "Protocol core, bindings, and registry profile" subsection added.** Distinguishes Core Protocol Objects, Transport Bindings, and Registry Operation Profile.
- **Section 2 — Renamed to "Reference Implementation Architecture Overview"** with explicit qualifier that the section describes the recommended architecture for the initial reference registry, not the only permitted architecture for conforming implementations. "ATAH servers" language replaced with reference-implementation language.
- **Section 14 — Conformance Classes (NEW).** Four classes: Core Object, Binding, Registry, Governance. Conformance statement requirement at `/.well-known/atah-conformance`.
- **Section 15 — Professional Records and Registration (NEW).** Three record creation routes (partner-created, claimed, direct self-registered). Distinction between professional record and professional account. Provenance honesty principle.
- **Section 16 — Glossary extended.** New terms: ATAH Core Protocol, Binding, Claimed record, Conforming registry, Direct self-registered record, Partner-created record, Professional account, Professional record, Protocol-conformant, Registry operator, Registry profile.

(Sections 14–18 of the v0.7-derived spec became Sections 16–20.)

### New sections

- **Section 1.5 — What ATAH Is and Is Not.** Three-layer framing (specification / conforming implementations / initial reference registry); non-goals list (recommendation engine, regulator, payment processor, etc.).
- **Section 1.6 — Relationship to Existing Standards.** OAuth 2.1, OIDC, W3C Verifiable Credentials, DIDs, W3C Consent Receipts, Sigstore-style transparency. Federation deferral with federation-ready architecture.
- **Section 4.1 — Schema Conventions.** Opaque ULID-style identifiers; RFC 3339 UTC timestamps; closed enums; additionalProperties policy; schema versioning.
- **Section 4.2 — Provenance Model.** Parallel `_provenance` map (replaces field-wrapping concept).
- **Section 4.10 — Consent Receipt Object.** Structured consent receipt schema; ATAH-stored hash + metadata; platform-stored full receipt.
- **Section 5.10 — Concern Flags.** Admin-only visibility; professional notification with right of reply; pattern-based review; bad-faith detection.
- **Section 6.4 — Type 2 Schema and States.** Full referral partner establishment schema with structured consent.
- **Section 6.5 — Type 3 Schema and States.** Two consent paths (client receipt and professional attestation).
- **Section 7.2 — Idempotency.** Required on all mutating endpoints.
- **Section 7.3 — Authorisation Matrix.** Actor × endpoint × scope × object-level constraint.
- **Section 7.9 — Error Model.** Structured error responses with codes and request_id.
- **Section 11.1 — Privacy Architecture Overview.** Headline statement of the transient-conduit model.
- **Section 11.2 — Deletion Scope Table.** Every store, personal-data policy, deletion mechanism, retention.
- **Section 11.5 — Handoff Access Control Model (Tiered).** Tier 1 read-like access; Tier 2 write access requires `handoff_access_token`.
- **Section 11.6 — Transient Encrypted Vault and Authenticated Retrieval.** Per-record DEK; crypto-erasure; three auth tiers (`tier_1`, `tier_2`, `tier_3`); failure handling.
- **Section 11.7 — Right to Information and Erasure (Limited Surface).** How the transient model satisfies erasure rights.
- **Section 13 — Security Model.** Authentication mechanisms by actor; cryptography; rate limiting; threat model summary; audit logging. (Was previously a single sentence in the security section of v0.7.)
- **Section 16 — Glossary.**

### Substantively rewritten

- **Section 2 — System Architecture.** Added transient encrypted vault as a core component; clarified registry never stores consumer personal data; strengthened notification service description.
- **Section 4.3 — Verification Status Codes.** Added `vc-verified` for W3C Verifiable Credentials. Profile-level `matching_status` split into more granular states (`regulatory-suspended` vs `admin-suspended`; `withdrawn`; `deceased`).
- **Section 4.4 — Profession Category Metadata.** Added `matching_weight_profile` (per-category weights), `review_signal_weight_cap` (per-category review weight cap), `stage2_auth_tier` (which auth tier required for Stage 2 retrieval).
- **Section 4.6 — Credentialled Profile Example.** Opaque atah_id format; location fields split (`licensed_jurisdictions`, `service_locations`, `physical_locations`, `remote_service_available`); parallel `_provenance` map example.
- **Section 4.9 — Enhanced Verification Object.** Commercial neutrality clarification; structured `custom_dimension` block requirements including governance approval reference.
- **Section 4.11 — Query Schema (Type 1).** Now references `consent_receipt_id` instead of boolean.
- **Section 4.12 — Match Response Schema.** Added `presentation_disclosure` block (machine-readable non-recommendation framing).
- **Section 5.1 — Partner Tiers.** Specified 30-day deletion on revocation.
- **Section 5.2 — Partner Verification Scope.** Added `supports_verifiable_credentials` field.
- **Section 5.7 — Independent Verifiers.** Change-of-control notification requirement; fee waivers for public-interest organisations; explicit commercial neutrality framing.
- **Section 5.8 — Review Platforms.** Annual re-attestation; audit rights; public classification rationale; per-category review weight cap.
- **Section 6.2 — Pre-Handoff Check Types.** Substantially unchanged but cross-references the new vault model.
- **Section 6.3 — Simultaneous Introduction Rules.** Unchanged — preserved as a strong v0.7 design.
- **Section 7.4 — External API.** Added `cancel`, `revoke-consent`, `.well-known/atah`, `/v1/capabilities` endpoints.
- **Section 7.5 — Professional API.** Added concern flag inbox and right-of-reply endpoints.
- **Section 7.6 — Partner API.** Added partner data push schema requirements (batch_id, signature, partial failure policy, etc.).
- **Section 8.1 — MCP Authentication.** OAuth 2.1-compatible; protected resource metadata; authorization server metadata; resource indicators; audience validation; explicit scopes; short-lived tokens with rotation; Dynamic Client Registration.
- **Section 8.2 — MCP Tools.** Nine tools (was six). New: `cancel_introduction`, `revoke_consent`, `get_consent_requirements`. Renamed: `submit_stage2_consent` → `submit_stage2_data`; `submit_stage3_consent` → `submit_stage3_data`. All tools now require `handoff_access_token` for state-changing calls.
- **Section 9 — Matching Engine.** Per-category matching weight profiles via `matching_weight_profile`; review weight cap per category; declared/verified scope completeness split; anti-gaming controls on inbound referral signal (reciprocal cap, time decay, referrer weighting, evidence requirement, dense-cluster discounting).
- **Section 10.4 — Roll-up Logic.** Pre-merge notification for high-risk categories with 7-day objection window.
- **Section 11.3 — Data Types and Retention.** Substantially rewritten to reference the transient vault.
- **Section 12 — Notification Service.** Substantially rewritten — strict no-PII policy across SMS and email; tiered authentication for retrieval.
- **Section 17 — Repository Structure.** Added `openapi.yaml`, `mcp-tools.json`, `consent-receipt.schema.json`, `consent-receipt-stored.schema.json`, `audit-event.schema.json`, `error.schema.json`, `provenance-map.schema.json`. Added top-level CONFORMANCE, ROADMAP, GOVERNANCE, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, CHANGELOG, CHANGES-FROM-V0_7-TO-V0_8 files.
- **Section 18 — Build Sequence.** Substantially rewritten across all phases to reflect v0.8 architectural changes.
- **Section 19 — AI Coding Agent Instructions.** Substantially expanded with v0.8-specific design rules.
- **Section 20 — Definition of Done for v0.8 MVP.** Substantially rewritten.

### Documentation tone

- "Complete and implementable as written" → "ready for technical review and reference implementation"
- All overclaiming pitch language softened.

---

## PRD-v0.7.md → PRD-v0.8.md

**Renamed and bumped from v0.7 to v0.8** to align with spec versioning. Substance updated to reflect v0.8 architectural changes. Content largely preserved where still accurate.

### Added

- Three-layer framing
- Standards composition (OAuth 2.1, OIDC, VCs, DIDs)
- Non-goals section
- Failure modes and accountability section ("ATAH does not warrant outcomes")
- Capture resistance section
- Concern flag mechanism with right of reply
- Tiered handoff access description
- Consent receipts model description
- Per-category matching weight profile description
- Concrete governance transition covenant

### Changed

- All references to boolean `consumer_consent` updated to consent receipts
- All references to PII-in-notifications updated to notification + authenticated retrieval
- "Commercial model is clean" → "designed to separate payment from ranking"
- "The timing is right" / "window will not stay open" — removed
- Roadmap updated to reflect v0.8 deliverables and post-publication 90-day governance plan commitment

---

## EXPLAINER.md

### Changed

- Softened "complete and implementable as written" claim to "ready for technical review and reference implementation"
- Plain-language description of the three-layer framing (specification / implementations / reference registry)
- Plain-language description of the federation deferral
- Plain-language description of standards composition
- Plain-language explanation of consent receipts
- Plain-language explanation of the transient vault and authenticated retrieval
- Glossary note on "consumer" terminology
- Failure-mode honesty

---

## README.md

### Added

- "What ATAH is *not*" section — explicit non-goals
- "Three layers" section — specification, conforming implementations, initial reference registry
- "Where ATAH sits" table updated to include OAuth 2.1, OIDC, and W3C VCs
- "Standards composition" framing throughout
- Concrete governance transition deadlines (12 months or earlier on triggers)
- "Status: Release candidate" badge

### Changed

- "Complete and implementable as written" → "ready for technical review and reference implementation"
- File references updated to include all new supporting documents (GOVERNANCE, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, CHANGELOG, CHANGES-FROM-V0_7-TO-V0_8)
- "Free to query" framed correctly (no per-query fee; authentication still required)

---

## New files in v0.8

These files did not exist in v0.7 and are added at v0.8 publication:

- **GOVERNANCE.md** — current decision-making, interim advisory group, transition plan
- **CONTRIBUTING.md** — how to propose specification changes, category annexes, bug reports, security disclosures
- **CODE_OF_CONDUCT.md** — Contributor Covenant 2.1 with ATAH-specific addenda (commercial pressure, conflicts of interest, antitrust expectations, responsible handling of security and privacy reports)
- **SECURITY.md** — threat model summary, responsible disclosure policy, security contact, partner/verifier authentication requirements
- **CHANGELOG.md** — release history (this file's parent)
- **CHANGES-FROM-V0_7-TO-V0_8.md** — this file
- **LICENSE** — Apache 2.0
- **spec/v0.8/openapi.yaml** — OpenAPI 3.1 contract
- **spec/v0.8/mcp-tools.json** — MCP tool input/output schemas
- **spec/v0.8/schemas/\*.schema.json** — JSON Schema files for all data structures (21+ schemas)

---

## Files unchanged or trivially updated

- **LICENSE** — Apache 2.0 (no change to terms; year/holder fields populated)
- **press/\*** — version references updated; no substantive changes (handled separately during press release update workflow)

---

## Summary

The architectural decisions made in v0.8 — and the rationale for each — are also captured in the project's session handover documents, which are maintained outside the public repository.

The most consequential v0.8 decisions, for reference:

1. **Tiered handoff access** preserves cross-platform user experience while closing the bearer-token vulnerability
2. **Parallel provenance map** delivers the per-field provenance promise without bloating schemas
3. **Consent receipt with hash storage** provides demonstrable consent without making ATAH a personal data store
4. **Transient vault with authenticated retrieval** ensures consumer PII never transits SMS or email
5. **Three-layer framing** clarifies the protocol/registry distinction without forcing federation design now
6. **Federation deferral** is intellectually honest — the architecture is federation-ready; mechanics will be specified in v0.9 / v1.0
7. **Per-category matching weight profiles** acknowledge that one weighting cannot serve all professional categories, while shipping defaults that are defensible
8. **Concern flag protections** balance consumer voice with professional defamation protection
9. **Concrete governance transition covenant** prevents the "as soon as a suitable form is established" governance drift trap
10. **Standards composition section** pre-empts the "why not VCs / OIDC?" critique by saying explicitly where each fits

---

*v0.8 — April 2026 — release candidate.*
