# Changelog

All notable changes to the ATAH protocol specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Breaking changes are possible until v1.0.

A detailed file-by-file change log between v0.7 and v0.8 is maintained separately in [CHANGES-FROM-V0_7-TO-V0_8.md](CHANGES-FROM-V0_7-TO-V0_8.md).

---

## [v0.8.1] — April 2026

**Status:** Release candidate. First public publication.

### Summary

v0.8.1 incorporates substantive pre-publication review remediation across legal/regulatory risk, market readiness/coherence, and adoption blockers. The protocol's purpose, actors, and lifecycle are unchanged from v0.8.0; v0.8.1 tightens governance commitments, corrects readiness-overstating language across the package, addresses GDPR posture for professional data, expands the security model with worked threat scenarios and incident response, and adds a small set of platform-facing and professional-facing operational documents.

### Changed

- **Governance transition reworked.** Removed the 90-day interim advisory group deadline and the 12-month transition deadline. Replaced with trigger-based transition (two trusted partner integrations live + one external conforming implementation demonstrated) and periodic public review when there is meaningful progress to share. Aligns with the realistic adoption timelines of cross-industry standards work and with the protocol's actual maintenance posture. Charter bumped to v1.3.
- **Roadmap phase dates removed.** PRD §14 Roadmap reframed as direction of work rather than delivery schedule. Quarterly dates (Q2 2026, Q3 2026, Q4 2026, 2027, 2027–2028) removed from all phases. Named-partner deliverables (NIPR, FINRA, SRA, FCA, etc.) reframed as illustrative examples. "Independent governance entity established" removed as Phase 3 deliverable — independent governance is now trigger-based per Charter, not phase-based. Phase 1 rewritten to describe only the publication-period activity (publication, repo live, community review, outreach) rather than aspirational reference-registry deliverables.
- **"Validated" → "verified" in consumer-facing surfaces.** Across README, EXPLAINER, PRD, and CHANGELOG. Preserved "validator stack" as a technical term of art.
- **"Permanent" → "entrenched" in CHARTER core-commitment framing.** Removes the implication of unamendable permanence; the supermajority amendment process remains in place.
- **Named regulators and bodies reframed as illustrative examples.** Throughout README, EXPLAINER, PRD, CHANGES-FROM-V0_7-TO-V0_8, and spec. Named bodies (NIPR, FINRA, state bars, PRSA, CIPR, the International Coaching Federation, etc.) appear as representative examples of the *types* of organisation the protocol works with — not as committed partners. New explanatory paragraph added to README and EXPLAINER making this convention explicit.
- **Notification transport reframed as binding-level.** SMS, email, MCP push, and other transports are now described as conforming-implementation choices rather than protocol mandates. The protocol-level requirement is no-PII through any notification channel.
- **MCP tool count corrected.** PRD §8.13 corrected from "eight" to "nine" tools.
- **"Press launch" → "publication announcement"** in PRD §14, aligning with the release-candidate framing.
- **Visible separation of regulatory sources from review platforms.** README and EXPLAINER trusted-partner passages rewritten to distinguish authoritative regulatory sources and professional membership bodies (the credential-verification authority layer) from review platforms (signal providers, not authority providers, with anti-gaming attestation requirements and per-category review weight caps). "Review-derived signals complement but do not substitute for credential verification" wording added at meaningful surfaces.
- **AEO equivalency framing softened.** "Without mastering Answer Engine Optimisation" reframed as "as part of their AI-discoverability infrastructure" across README, EXPLAINER, PRD. Clarifying sentence added to README and EXPLAINER: ATAH is not a replacement for a professional's existing marketing or visibility work; there is no guarantee of introduction volume, ranking, or work; the protocol is designed to become a trusted handoff rail as platforms integrate.

### Added

- **Professional Data Protection section (PRD §9).** GDPR posture for professional data: ATAH as data controller, lawful bases by source, professional notice and rights (access, rectification, objection, erasure subject to public-task carve-outs, restriction, portability), partner-route opt-out, partner agreement DPA terms, audit log treatment.
- **Platform presentation obligations (PRD §11, EXPLAINER mirror).** Platforms must not describe ATAH results as "recommended," "approved," "best," "endorsed," or "vouched for" by ATAH.
- **Liability allocation freshness paragraph (PRD §11).** Explicit statement that platforms are expected to surface verification freshness; ATAH does not warrant accuracy beyond published freshness windows.
- **Concern flag lifecycle (PRD §8.11) and Concern flag data handling (GOVERNANCE §7).** Full lifecycle from intake through escalation; lawful basis, retention, professional rights, escalation thresholds.
- **Consent receipt limitation (EXPLAINER, PRD §8.7).** ATAH verifies receipt integrity, not consent capture; platform retains responsibility for the consent ceremony.
- **Revocation by stage (EXPLAINER, PRD §8.6).** Clarifies what revocation actually achieves at each stage of the introduction.
- **Cross-jurisdiction default rule (PRD §8.5).** No cross-jurisdiction matching by default for regulated categories; explicit query parameter and disclosure required.
- **Vetting strength surfaced in match responses (EXPLAINER, PRD §6.3).** Explicit `vetting_strength` enum (`regulatory`, `strong_membership`, `open_membership`) with public-facing classification names. Six public-facing partner classes added: authoritative regulatory source, professional membership body, open membership body, review signal provider, independent verifier, public registry source.
- **Worked threat scenarios in SECURITY.md.** Three scenarios — leaked handoff_id, malicious AI platform asserting forged consent receipt, compromised partner submitting poisoned data.
- **Incident response in SECURITY.md.** Confirmed professional fraud and professional account compromise procedures.
- **Conformance self-certification disclaimer (CONFORMANCE.md).** Conformance statements are self-declared until v0.9 conformance test suite ships.
- **Founder COI extended to immediate family (CHARTER §7).**
- **Founder seat language tightened (CHARTER §7).** Seat is personal, non-transferable, non-inheritable; lapses on founder unavailability.
- **DPA acknowledgement (CHARTER Part Two, GOVERNANCE §4).** Partner agreements include data-protection terms.
- **Disciplinary-data treatment subsection (PRD §8.4).** Severity categorisation, spent/expired discipline, withdrawal of complaints, professional right of reply.
- **Mid-handoff status change handling (PRD §8.6).** Procedure when a professional's matching status changes mid-handoff.
- **Conflicting credentials hierarchy (PRD §8.4).** Resolution order: regulator > statutory register > mandatory professional body > strong voluntary body > independent verifier > review platform > self-declared.
- **Capacity assumption (PRD §9).** ATAH assumes the asserting platform has determined consumer capacity.
- **Roll-up objection branch (spec §10.4).** What happens after a professional objects within the 7-day pre-merge window. Conditional-required `acknowledged_rollup_terms` field added to both professional schemas to close the contract gap with OpenAPI registration documentation.
- **`consumer_ref` MUST NOT specification (spec §11.5).** Negative constraints on `consumer_ref` to close cross-platform correlation risk.
- **Acknowledgements rewrite (README).** Leads with human authorship; AI tools framed as drafting/stress-testing/editorial assistants with human-led structural decisions.
- **COVERAGE.md.** Launch coverage matrix by category × jurisdiction × verification route × matchable-at-launch.
- **`presentation_disclosure` mention in README.**
- **"Isn't this just X?" FAQ in EXPLAINER.**
- **Consent Storage Rationale doc (`docs/consent-storage-rationale.md`).** Design rationale for the consent receipt split between AI platform and ATAH.
- **Issue templates (`.github/ISSUE_TEMPLATE/`).** spec-change, category-annex, bug, config.
- **Maintenance posture note (README, GOVERNANCE).** Honest acknowledgement that ATAH is currently maintained by a small team with limited bandwidth. Published SLAs are best-effort targets reflecting that posture.
- **Verification confidence signal (spec §4.12, PRD §8.5).** Match responses for regulated categories will carry a `verification_confidence` field separate from verification-quality score components, surfacing single-source vs multi-source verification basis. Field name, value enumeration, and semantics committed in v0.8.1; schema implementation lands in v0.8.2.
- **Consolidated liability allocation subsection (PRD §11).** New "Liability allocation and platform/ATAH responsibilities" subsection consolidating what ATAH commits to, what ATAH does not warrant, what platforms take on, and worked examples of common cases. EXPLAINER mirror paragraph and README "What ATAH is *not*" bullet added.
- **Adoption Blockers in-scope items.** Six new operational documents: `docs/platform-adoption/handoff-trigger-guidance.md` (P2), `docs/platform-adoption/ux-disclosure-copy.md` (P7), `docs/platform-adoption/commercial-neutrality-memo.md` (P8), `docs/professional/visibility-controls.md` (P22), `docs/professional/visibility-report.md` (P29), `docs/professional/firm-admin.md` (P31). Plus inline patches: integration cost honesty in README (P12), public partner taxonomy paragraph in PRD §6.3 (P14), Stage 1 appetite-check controls in PRD §6.1 (P25), field-level visibility classes in PRD §8.2 (P27), regulatory-vs-review separation wording sweep (P15), AEO equivalency wording sweep (P32). P10 is satisfied by COVERAGE.md.
- **Schema/OpenAPI delta corrections.** Prose-only corrections aligning spec text with schemas, OpenAPI, and MCP tools file: §4.1 prefix list (removed unused `rp-`, reframed as non-exhaustive), §4.10 `data_categories` (added `contact_preference` to prose list to match schema), §6.4 Type 2 example (`rp-` → `proposal-`), §7.3 authorisation matrix (scope names aligned to OpenAPI's actual `securitySchemes`), §8.1 OAuth scopes (removed orphan `atah:consent:revoke`), §8.2 `find_professional` input shape (matches `query.schema.json` matter sub-object), §10.4 dispute-resolution cross-reference (§8.10 → §5.9). OpenAPI Stage 2 endpoint: empty 202 removed, `held_status` folded into 200 response matching MCP semantics.
- **ROADMAP v0.8.2 candidates expanded.** Six candidates tracked: `verification_confidence` schema implementation, MCP `initiate_introduction` Type 3 support, REST `get_consent_requirements` endpoint, `presentation_disclosure` standalone schema refactor, `check_introduction_status` response shape alignment between OpenAPI and MCP, non-handoff-bound consent revocation endpoint, `consumer` actor_type addition to audit-event schema.

### Held

The following items from the three pre-publication reviews are held for post-publication work (some tracked externally in non-published planning artefacts):

- Items needing legal review (Platform Legal Integration Pack, Model partner data-sharing agreement, AI Platform Developer Terms, Published partner indemnity framework, Category-specific ethics memos and referral-fee legal opinions)
- Items needing partner conversations (Member Communications Pack, Partner approval criteria with worked examples, Lower-tier partner ingestion + onboarding process, Worked compliance annexes for specific categories)
- Items needing operational decisions not yet made (Individual self-registration fee number, Operational SLA / RTO/RPO, Operational claim authentication procedure)
- Items requiring schema changes or new code (those listed in the v0.8.2 candidates section above)
- Operational tasks for the founder (seeding GitHub Discussions before launch; identifying interim advisory group members; press cycle and association outreach)

### Known limitations (deferred to v0.8.2)

- **MCP `initiate_introduction` Type 3 support.** The MCP tool description references Type 3 referrals but the input schema only models Type 1. The REST endpoint (`POST /v1/introductions`) handles both Type 1 and Type 3 via the `introduction_type` discriminator and is the v0.8.1 workaround for MCP clients needing Type 3. Full MCP Type 3 support lands in v0.8.2.
- **REST `GET /v1/consent-requirements` endpoint.** MCP-native platforms have the `get_consent_requirements` tool. REST-only platforms in v0.8.1 should source consent text and required `data_categories` from spec §4.10 and `consent-receipt.schema.json` directly. A native REST endpoint with the same response shape as the MCP tool lands in v0.8.2.

### Security

Tiered handoff access control, no-PII-in-notifications, consent receipt hash storage, audit log pseudonymisation, and the rest of the v0.8.0 security commitments are unchanged. v0.8.1 adds worked threat scenarios and incident response procedures (SECURITY.md §4 and §8 respectively).

### Privacy

Privacy floor for consumer data (Charter Part One core commitment 5) is unchanged. v0.8.1 adds the Professional Data Protection section addressing professional data — the larger volume surface area — with controller role, lawful basis, professional rights, partner notice, erasure handling, and partner DPA acknowledgement.

---

## [v0.8.0] — April 2026

**Status:** Release candidate. First public publication.

### Summary

v0.8.0 is the release-candidate version of ATAH, incorporating substantial pre-publication peer review remediation across architecture, security, privacy, governance, and documentation. The protocol's purpose, actors, and lifecycle are unchanged from v0.7; v0.8 tightens, completes, and clarifies what was previously implied or under-specified.

### Added

- **Protocol-layer refinement.** Clarified ATAH Core Protocol, transport bindings, registry operation profile, and reference registry boundaries. The initial ATAH registry remains the first operational implementation, but the specification is now explicitly transport-compatible and implementation-neutral at the core object layer. README, EXPLAINER, and spec language updated to distinguish protocol from reference implementation throughout. New spec Section 14 (Conformance Classes — four classes: Core Object, Binding, Registry, Governance) and Section 15 (Professional Records and Registration — three record creation routes: partner-created, claimed, direct self-registered). Charter v1.2: Open specification commitment made explicitly royalty-free; Open MCP endpoint commitment scoped to the ATAH-operated reference endpoint and clarified for future conforming implementations.
- **Three-layer framing.** Specification, conforming implementations, and initial reference registry are now distinguished explicitly throughout the documentation.
- **Standards composition section.** Explicit treatment of OAuth 2.1, OpenID Connect, W3C Verifiable Credentials, and Decentralized Identifiers — what ATAH uses, what it accepts, and where the boundaries are.
- **Federation framing.** v0.8 is single-registry by design; the architecture is federation-ready (atah_id namespace, no single-registry assumption in the data model). Federation mechanics deferred to v0.9 / v1.0.
- **Non-goals section.** Explicit list of what ATAH is not — recommendation engine, regulator, payment processor, directory product, personal data repository, consumer interface — added to README, spec, EXPLAINER, and PRD.
- **Glossary.** Section 16 in the spec, extended with new terms for the protocol-layer framing.
- **CONFORMANCE.md.** Top-level repository file summarising the four conformance classes (Section 14 of the spec) for discoverability by standards reviewers and implementers.
- **ROADMAP.md.** Top-level repository file describing v0.8.1, v0.9, and v1.0 candidate work in narrative form. Granular tracking moves to GitHub Issues once the repository is live.
- **Stored consent receipt as separate schema.** `consent-receipt-stored.schema.json` defined as a distinct schema from `consent-receipt.schema.json` so the privacy boundary (no `consumer_ref`, no `data_categories`, no consent text in the stored form) is structurally enforceable through schema-validation tests.
- **Schema conventions block.** Section 4.1 of the spec defines opaque ULID-style identifiers, RFC 3339 UTC timestamps, closed enums, additionalProperties policy, and schema versioning.
- **Parallel provenance map.** Per-field provenance is now delivered through a `_provenance` map rather than wrapping individual fields. Matches the README's headline trust commitment without bloating schemas.
- **Type 2 and Type 3 introduction schemas.** Full lifecycle schemas, consent flows (including structured client consent receipts and structured professional attestations of client consent for Type 3), and state machines.
- **Consent receipts.** Boolean `consumer_consent` replaced with structured consent receipt objects following W3C Consent Receipt and Kantara CR 1.1 patterns. ATAH stores the receipt hash and metadata; the asserting platform stores the full receipt (Sigstore / RFC 6962 transparency log pattern).
- **Tiered handoff access control.** `handoff_id` no longer grants access on possession alone. Read-like operations available to original asserter or to authenticated AI platforms with matching consumer reference (preserving cross-platform user experience). Write operations require a `handoff_access_token` bound to platform/client/scope/expiry.
- **Transient encrypted vault and authenticated retrieval.** Section 11.6 of the spec. Personal data never transmitted through SMS or email — only notification metadata and authenticated retrieval references. Three authentication tiers (`tier_1`, `tier_2`, `tier_3`) per category sensitivity. Crypto-erasure as primary deletion mechanism.
- **Idempotency requirement.** All mutating endpoints require an `Idempotency-Key` header.
- **Authorisation matrix.** Section 7.3 of the spec defines who can do what with which scope and object-level constraints.
- **Cancel and revoke as first-class operations.** New `cancel_introduction` and `revoke_consent` MCP tools and corresponding REST endpoints. New `get_consent_requirements` tool to support consent capture.
- **Per-category matching weight profiles.** Categories may define `matching_weight_profile` in `profession-categories.json`. Regulated categories weight verification quality higher than availability. Category-specific overrides require governance approval.
- **Review weight cap by category.** `review_signal_weight_cap` field caps review platform contribution to verification quality per category. ≤0.10 for high-stakes regulated categories.
- **Anti-gaming controls on inbound referral signal.** Reciprocal cap, time decay, referrer verification quality weighting, evidence of successful outcomes for full weight, dense-cluster discounting.
- **Declared/verified scope completeness split.** Sub-component D of verification quality split into declared and verified. Only verified materially contributes.
- **Concern flag protections.** Admin-only visibility, never used in matching, professional notification with 30-day right of reply, pattern-based review, bad-faith flag detection.
- **Founder conflict-of-interest, recusal, and disclosure rules.** Annual public disclosure, recusal from decisions affecting founder-affiliated entities, public register of related-party transactions.
- **Concrete governance transition covenant.** 90-day interim plan, 12-month transition deadline or earlier on triggers, public-explanation safety valve for any deadline revision.
- **Interim advisory structure.** Required consultation before approving the first verifier, review platform, partner agreement, category compliance annex, or governance change.
- **Privacy and compliance gating floors as Charter Part One core commitments.** Promoted from operational to permanent.
- **Partner principles, professional participation principles, verifier/review platform principles** added to Charter Part Two.
- **Hardship waivers and fee caps** for individual self-registration. **Fee waivers** for public-interest partners.
- **Presentation disclosure block** in every match response. Machine-readable non-recommendation framing AI platforms must surface to users.
- **Failure-mode honesty section** in PRD. ATAH does not warrant outcomes; AI platforms and consumers retain responsibility.
- **Capture resistance section** in PRD.
- **Opaque atah_id format.** ULID-style identifiers; country, profession, and category are stored as attributes rather than encoded in the ID.
- **Profile-level matching status** split: `regulatory-suspended` and `admin-suspended` distinguished. New states: `withdrawn`, `deceased`.
- **Location field split.** `licensed_jurisdictions`, `service_locations`, `physical_locations`, `remote_service_available`.
- **Missing supporting documents.** GOVERNANCE.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, CHANGELOG.md (this file), CHANGES-FROM-V0_7-TO-V0_8.md added to repository.
- **Custom verification dimension structure.** Required structured `custom_dimension` block including governance approval reference.
- **Pre-merge notification for high-risk roll-up matches.** 7-day objection window for regulated-tier categories.

### Changed

- **MCP authentication updated.** OAuth 2.1-compatible authorisation; protected resource metadata; authorization server metadata; resource indicators (RFC 8707); audience validation; explicit scopes; short-lived access tokens with refresh and rotation; Dynamic Client Registration (RFC 7591). Conforms to current MCP authorisation guidance at the time of implementation.
- **MCP tool naming.** `submit_stage2_consent` renamed to `submit_stage2_data` and `submit_stage3_consent` renamed to `submit_stage3_data` to reflect that these tools transmit both consent and data.
- **Authentication mechanisms by actor.** Partners and verifiers writing high-trust data now require signed JWT client assertion or HMAC-signed requests. API keys retained for read-only operations only.
- **Documentation tone.** Replaced "complete and implementable as written" with "ready for technical review and reference implementation." Replaced "commercial model is clean" with "designed to separate payment from ranking." Removed "the timing is right" / "the window will not stay open" pitch language from PRD.
- **Charter version bumped to v1.2.** v1.1 reframed Charter as a public governance covenant; founder undertakes to administer ATAH consistently with its commitments. v1.2 adds protocol-layer refinement: royalty-free protocol implementation made explicit; Open MCP endpoint commitment scoped to the ATAH-operated reference endpoint with future conforming implementations clarified.
- **PRD bumped to v0.8.** Aligned with the v0.8 spec.

### Deferred to v0.9 / v1.0

The following are acknowledged as known v0.9 candidates rather than v0.8 omissions:

- **Federation mechanics.** Cross-registry trust, atah_id resolution across registries, handoff routing between registries, federated governance.
- **Claim authentication hardening.** v0.8 specifies the minimum (verified email match against partner records or equivalent partner-attested identifier). Step-up authentication, anomaly detection on claim attempts, partner-known-channel notification with objection window, and rate limiting are deferred to v0.9. The reference registry SHOULD apply these controls operationally at launch.
- **Sybil and referral-ring detection.** Beyond the v0.8 anti-gaming controls.
- **Platform-light consent mode.** Optional alternative to the v0.8 self-asserted-receipt model. v0.9 will specify a model where ATAH provides more of the consent ceremony (canonical consent text endpoint, consent flow orchestration helper) for AI platforms that prefer to invoke a standard ATAH consent flow rather than constructing receipts themselves. The v0.8 self-asserted model continues to be supported.
- **Per-category matching weight profiles populated with researched values** for all launch categories. v0.8 ships with defaults plus exemplar overrides.
- **Full STRIDE-level threat model.** SECURITY.md ships with threat categories listed; full modelling deferred.
- **Conformance test suite.** Discovery endpoints (`/.well-known/atah`, `/v1/capabilities`) shipped; the test suite itself is v0.9.
- **Edge-case lifecycle handling.** Detailed handling for jurisdiction changes, partner acquisitions, verifier acquisitions, mid-flight category compliance changes.
- **Detailed dispute resolution process.** Beyond v0.8 baseline.
- **Full DSAR machinery for federation scenarios.**
- **Multilingual support.**

### Security

- Tiered handoff access control mitigates handoff_id leakage as a bearer-token vulnerability.
- Personal data never transmitted through notification channels (SMS, email) mitigates PII leakage through providers.
- Consent receipt hash storage mitigates consent forgery and platform-side receipt tampering.
- Authentication mechanism strengthened for high-trust partner and verifier writes.
- Audit log made tamper-evident (hash-chained) and retains pseudonymous identifiers only.

### Privacy

- Personal data never written to the main registry database.
- Notification channels carry no consumer personal data — only metadata and authenticated retrieval references.
- Crypto-erasure as primary deletion mechanism for the transient vault.
- Deletion scope table (Section 11.2) enumerates every store and its personal-data policy.
- Right to erasure satisfied by the transient-only design for ATAH-held data.

---

## [v0.7] — Pre-publication, internal

Five issues resolved: atomic deletion compensating action pattern specified; MCP endpoint authentication mechanism defined (OAuth 2.0); verification batching strategy made concrete; simultaneous introduction rules specified explicitly; schema versioning policy added.

The PRD-v0.7.md document was retained alongside v0.8 spec work and is replaced by PRD-v0.8.md at v0.8 publication.

## [v0.6] — Pre-publication, internal

Targeted corrections: state bar verification scoped to priority states at MVP; webhooks and MCP push repositioned as Phase 2. Conflict field classification made explicit. Referral signal discount threshold defined concretely. Notification model corrected.

## [v0.5] — Pre-publication, internal

Initial technical specification.

---

*This changelog tracks the protocol specification. Reference implementation changes are tracked in the reference implementation repository.*
