# ATAH Charter

**Founding governance commitments of the Agent to Authenticated Human Protocol**

*Version 1.3 — April 2026*

---

## Purpose of this Charter

This Charter sets out the founding governance commitments of the Agent to Authenticated Human Protocol (ATAH). It is intended to operate as a public governance covenant. Until governance transfers to an independent entity, the founder undertakes to administer ATAH consistently with this Charter. [GOVERNANCE.md](GOVERNANCE.md) identifies who may raise a Charter compliance objection, the review process, and the public remedy for breach.

The Charter applies to all versions of the protocol, regardless of the corporate or foundation structure that holds governance over time.

The Charter is divided into two parts. **Core commitments** are entrenched and amendable only by the process set out in the Amendments section below. They define the protocol's neutrality, openness, and minimum privacy commitments, and exist to make those properties durable across changes in leadership. **Operational commitments** can evolve through normal governance processes as described in [GOVERNANCE.md](GOVERNANCE.md).

The Charter is published in the protocol's public repository. It takes effect from the date of its first commit. Amendments to operational commitments are made through pull request to this document, following the process set out in GOVERNANCE.md. Amendments to core commitments require the supermajority threshold defined below and are recorded in the document's revision history.

---

## Part One: Core commitments

These commitments are entrenched. They may be amended only through the supermajority core-commitment process set out in the Amendments section below. They define what makes ATAH neutral infrastructure rather than a commercial product, and they protect those properties against future drift.

### 1. Open specification

The ATAH protocol specification is published under the [Apache License 2.0](LICENSE). Anyone may implement, build on, fork, or extend the protocol. No protocol royalty, licence fee, or permission requirement applies to implementing ATAH. Standard Apache 2.0 attribution requirements apply when redistributing the specification.

The ATAH-operated reference MCP endpoint is free to query subject to authentication, rate limits, and abuse controls (see Core commitment 4 below).

Commercial charges may exist only at the implementation or operational layer (such as the ATAH-operated reference registry or a future conforming registry) and must not affect matching, provenance, or conformance status.

The specification will remain published under an open licence approved by the Open Source Initiative for as long as the protocol exists. Any future change of licence requires the same supermajority core-commitment amendment process.

A fork or derivative work created under the open licence may not represent itself as the official ATAH protocol, the ATAH-operated reference registry, or an ATAH-governed implementation, and may not claim official ATAH conformance recognition, certification, endorsement, governance continuity, or founder participation where none exists. The open licence permits derivative works; it does not extend the official ATAH governance line, the founder advisory seat, or the ATAH project identity to those derivative works. Conforming implementations, by contrast, operate the ATAH protocol as published under the conformance principles set out in this Charter and the specification, and remain part of the ATAH ecosystem.

### 2. No commercial weighting in matching

The protocol's matching logic carries no commercial weighting at any point. Partner integration fees, professional registration fees, AI platform relationships, enhanced verification fees, review platform integration arrangements, and any other commercial relationship in or around ATAH have no influence on:

- Whether a professional is returned in a match result
- The position of a professional in a ranked shortlist
- The trust signals attached to a professional's profile
- The recommendation status of any professional or partner

Matching is determined by relevance, verification quality, availability, completeness, and inbound referral signal — never by who has paid what to whom. Where paid services generate verification evidence, that evidence must be scored under the same public rubric available to all approved sources, regardless of who commissioned or paid for it.

### 3. Provenance always visible

Every data point in a professional's profile, and every claim returned in a match result, is tagged with its source and verification status. Trusted partner contributions are always identified by partner name. Self-declared data is always labelled as such. Provenance is never obscured, aggregated away, or collapsed into an opaque trust score.

### 4. Open MCP endpoint

The ATAH-operated reference MCP endpoint remains free for any AI platform to query. There is no per-query fee, no licensing fee for AI platform implementation of the protocol, and no commercial gate on protocol access at the platform layer. Free does not mean unauthenticated: querying still requires authentication, rate limits, and compliance with developer terms to protect consumers and registry integrity. The commitment is that authentication and operational controls exist solely to protect the system, not to monetise platform access.

The ATAH protocol itself remains royalty-free to implement. Future third-party conforming implementations may operate their own endpoints, subject to the conformance principles set out in the specification (Section 14), but may not use commercial payment to influence matching or provenance presentation.

References to registry operations elsewhere in this Charter apply to the ATAH-operated reference registry and any governance-controlled successor. They do not prevent independent parties from implementing the open specification under Apache 2.0, provided they do not claim ATAH conformance unless they meet the conformance criteria.

### 5. Privacy floor

ATAH must not operate as a central repository of consumer personal data. Consumer personal data may be processed only for explicit, staged handoff purposes and must be minimised, time-limited, and auditable. Notification channels (such as SMS and email) must not carry consumer personal data; they may carry only non-sensitive notification text and authenticated retrieval references. The detailed data governance commitments are set out in the specification and may be strengthened, but not weakened, through governance review.

This privacy floor concerns consumer personal data. Professional data is
also personal data where it relates to identifiable professionals, and is
governed by the operational commitments on professional data protection set
out in Part Two and in the specification.

### 6. Compliance gating floor

No professional category may be surfaced in matching unless category-specific minimum verification, compliance, and disclosure requirements have been approved and published as a compliance annex. Categories may be open for registration before the annex is complete, but professionals in such categories must not be returned in match results until compliance gating has been satisfied.

### 7. Founder's role and compensation

The founder retains a continuing advisory seat on the governance body of the protocol. This seat:

- Carries no veto rights over protocol decisions
- Confers no preferential commercial advantage to any entity the founder is involved in — any such entity participates in the ATAH ecosystem on terms equally available to any other participant
- Exists to preserve continuity of intent across governance transitions, not to confer ongoing control

The founder is subject to the following conflict-of-interest, recusal, and disclosure rules:

- Annual public conflict-of-interest disclosure, listing all entities in which the founder or members of the founder's immediate family hold a material interest and which are or may become participants in the ATAH ecosystem
- Disclosure of any founder-affiliated entity participating in ATAH as a partner, verifier, review platform, or registered professional
- Recusal from governance decisions affecting any founder-affiliated entity
- Public register of related-party transactions involving ATAH and any founder-affiliated entity
- No privileged access to non-public partner, platform, or registry data beyond that available to other governance body members

Reasonable compensation for the founder's time spent on protocol work is permitted, set transparently and reviewed by the governance body once established. The founder does not receive a personal share of partner integration fees, professional registration fees, enhanced verification fees, or any commercial benefit conditional on the registry's growth. Where the founder draws compensation for protocol work, the amount and basis are publicly disclosed.

The founder advisory seat is a personal seat. It is non-transferable and non-inheritable. After transition to independent governance, the seat may be occupied only by the founder. If the founder is unavailable, becomes incapacitated, or withdraws, the seat lapses; the independent governance body may, at its discretion, convert it into a non-voting historical adviser role appointed at its own decision. No founder nominee may hold approval rights over core amendments.

The right to reasonable compensation for protocol work set out in this commitment — applicable to the founder, and equivalently to any future advisor or governance member — is distinct from operational service-provider arrangements governed by Part Two and GOVERNANCE.md §4.1. Governance compensation and operational service-provider compensation are separate categories. The founder advisory seat established by this commitment applies to the official ATAH governance body and any governance-controlled successor; it does not extend to independent forks or derivative protocols that are not governed by ATAH. Where the term "founder-affiliated entity" is used in this Charter and in GOVERNANCE.md, it means any organisation in which the founder holds a material interest, controls, or with which the founder has a substantive ongoing commercial relationship.

### 8. Operational independence

ATAH services may be operated by service providers — contractors, technology partners, implementation partners, managed-service providers, employees, contributors, or other supporting organisations — on commercial or paid terms. No such operational arrangement may confer ownership of the protocol, exclusive control over its evolution, or any right to influence matching, ranking, provenance, conformance, partner approval, platform access, professional visibility, data access, or any other Charter-protected commitment.

Any operational service-provider arrangement must be terminable on reasonable notice, replaceable, and not exclusive. Independent governance must be structurally able to replace any operational service provider. The existence of a paid service-provider relationship does not confer protocol ownership.

This commitment applies equally to all operational service providers, including the founder, founder-affiliated entities, advisor-affiliated entities, contributors, contractors, and any other related party. Where the operator is a related party, the additional disclosure obligations in GOVERNANCE.md §4.1 apply.

This commitment applies to operational service-provider arrangements only. It does not affect the founder's continuing advisory seat as set out in Core commitment 7 above, nor the right to reasonable compensation for protocol work as set out in Core commitment 7 above. Governance roles and compensation for protocol/governance work are separately governed by Core commitment 7.

---

## Part Two: Operational commitments

These commitments describe how the protocol operates today and may evolve through the governance process described in GOVERNANCE.md.

### Transition to independent governance

It is the founding commitment that governance of the ATAH protocol will transition to an independent structure — a not-for-profit, foundation, standards body, or equivalent — once the protocol has demonstrated real-world adoption.

**Transition triggers.** Transition is triggered when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated. These triggers reflect that independent governance is appropriate when there is an ecosystem to govern, rather than when an arbitrary calendar deadline is reached. The protocol works with cross-industry organisations whose decision cycles are inherently slower than software-industry timelines, and the transition framework recognises that.

**Interim period.** Until the transition is complete, the protocol operates under this Charter, with the founder undertaking to administer ATAH consistently with its commitments. An interim advisory group will be convened when suitable members are identified and available to serve. The composition, member identities, and operating procedures will be published when finalised.

**Annual review.** The status of governance transition will be reviewed periodically and published as warranted. The review will report on progress toward the trigger conditions, the state of partner integrations, and any factors affecting the path to independent governance. If circumstances change in a way that affects the transition approach, the change will be explained publicly.

**Continuity.** The choice of governance structure will be made transparently and in consultation with active contributors and trusted partners at that time. The protocol specification will remain open and licensed under Apache 2.0 regardless of the corporate structure chosen for governance.

### Interim advisory structure

Before the interim governance body is formed, the founder will convene an interim advisory group when suitable members are identified and available to serve. Once convened, the interim advisory group must be consulted before any of the following decisions:

- Approval of the first independent verifier
- Approval of the first review platform
- Approval of the first commercial partner agreement
- Approval of the first category compliance annex
- Any change to the Charter

The interim advisory group's composition, terms, and operating principles will be published in [GOVERNANCE.md](GOVERNANCE.md).

### Operational support and service providers

ATAH may be supported and operated by service providers — independent contractors, technology partners, implementation partners, managed-service providers, employees, contributors, or other supporting organisations. Permitted commercial arrangements include documented cost reimbursement, fixed-fee build work, time-and-materials work, cost-plus support, market-rate managed-service fees, and similar service-provider compensation structures. These payments compensate work performed; they do not buy ranking, visibility, approval, conformance, access to non-public data, or influence over protocol decisions.

Where a service provider is the founder, a founder-affiliated entity, an advisor-affiliated entity, or any other related party, the arrangement is governed by the related-party disclosure rules in GOVERNANCE.md §4.1, and is subject to the operational independence commitment in Part One (Core commitment 8). Operational service-provider arrangements must be terminable on reasonable notice, replaceable, and not exclusive.

No service-provider arrangement may be structured as a protocol royalty, paid-placement fee, referral fee, success fee, permanent revenue share, or commercial return linked to registry growth, professional ranking, partner approval, introduction volume, or user-facing visibility.

### Trusted partner participation

Partners pay ATAH for integration access. Partner participation funds the work of building and maintaining the protocol. Partner status is granted on the basis of data quality, freshness commitments, and verification capability — never on the basis of payment alone.

The following partner principles apply:

- A published partner fee schedule with cost-recovery rationale
- Fee waivers or deferred fees for regulatory bodies and public-interest organisations
- No exclusive arrangements within any category
- A public partner registry showing partner status, scope, and approval date
- Data quality criteria independent of payment level
- Equal data-handling and provenance treatment regardless of fee tier
- Partner agreements include appropriate data-protection terms, including
  controller/processor or joint-controller arrangements where required

Partner contributions are subject to data quality and freshness obligations. Partners failing to meet these obligations may have their data deprioritised, suspended, or removed under the conflict and integrity processes described in the specification. Detailed partner terms are documented in [GOVERNANCE.md](GOVERNANCE.md) and may evolve through governance review.

### Professional participation

Two routes are available for professionals to be present in the registry: via a trusted partner, or by individual self-registration. Individual self-registration is free during the protocol's initial period after launch. After that initial period closes, individual self-registration remains available but a small annual fee applies to maintain the listing.

The following professional participation principles apply:

- A hardship waiver process for individual self-registration fees
- A public fee cap or cost-recovery principle for individual fees
- Free listing for categories lacking partner coverage until a partner route exists
- No ranking difference between paid individual listings and partner-route listings

Where a trusted partner joins ATAH for a category, individual listings of professionals who are members of that partner are absorbed into the partner's data through the roll-up process described in the specification. The terms of professional participation are documented separately and may evolve through governance review.

### Independent verifier and review platform principles

Independent verifiers and review platforms are approved by ATAH governance under public criteria. The following principles apply:

- Public verifier acceptance criteria and scoring rubric
- Public review platform acceptance criteria, including anti-gaming controls
- Annual re-attestation for both verifiers and review platforms
- Audit rights and suspension criteria
- Public classification rationale for each approved verifier and review platform
- Verifiers may not also operate as professional bodies for categories they verify
- Change-of-control notification requirements for both verifiers and review platforms

Detailed verifier and review platform terms are set out in the specification and [GOVERNANCE.md](GOVERNANCE.md).

### Privacy and data minimisation operational commitments

These operational commitments implement the Part One privacy floor. They may be strengthened but not weakened through governance review.

- Notification channels carry no consumer personal data; only non-sensitive notification text and authenticated retrieval references
- Consumer personal data is held only in transient encrypted stores, retrieved through authenticated access, and deleted or crypto-erased per category retention rules
- Every introduction requires fresh explicit consent, captured as a structured consent receipt
- ATAH never re-uses personal data from a previous introduction
- ATAH operates the three-concept separation (payload erasure / audit retention / withdrawal-as-state-transition) — payload content is crypto-erased per the spec; tamper-evident audit metadata is retained without consumer personal data in plaintext; withdrawal stops future protocol processing without erasing audit history. Detailed semantics live in spec §11.
- Matching is hard-filters-then-stratified-randomisation, not a recommendation engine. ATAH may determine eligibility and exclusion, but MUST NOT express preference among eligible candidates unless the ordering basis is explicit, non-commercial, and disclosed. Where no user-requested ordering criterion is supplied, candidate order MUST be randomised or rotated using a documented fairness policy. Review-derived signals MUST NOT move a candidate into a higher band in high-stakes regulated categories unless corroborated by authoritative credential or regulator-source evidence. Detailed semantics live in spec §9.
- Transparency is a conformance requirement. Conforming implementations MUST produce machine-readable explanations for discovery, exclusion, ordering, handoff, withdrawal, suppression, and data-sharing events, subject to privacy, security, and anti-gaming limits. Professional-facing visibility explanations MUST NOT expose actual query-count or query-history data; they MUST present representative explanation categories derived from the implementation's documented rules and the professional's own profile data. Detailed semantics live in spec §11A.
- ATAH supports two consent types — query authorisation and disclosure consent. A third consent concept exists in the world — engagement consent — which is the consent that creates a professional-client relationship. Engagement consent is outside ATAH by design. ATAH consent receipts MUST NOT be represented as professional engagement consent. Conforming implementations MUST disclose that any professional-client relationship arises only through the professional's own onboarding, engagement, regulatory, or contractual process. Detailed semantics live in spec §4.10.
- Component 3 (referral connection-making) actions are professional-to-professional and MUST be authorised through authenticated professional delegation. They MUST NOT use consumer disclosure-consent receipts and MUST NOT imply consumer involvement. Detailed semantics live in spec §6.4 and §7.3.
- Detailed data governance is set out in the specification
- Professional data is also personal data where it relates to identifiable
  professionals. Operational commitments on professional data protection
  (controller role, lawful basis, professional rights, partner notice,
  erasure and objection) are set out in the PRD §9 and the specification §11.

### Code of conduct

The protocol community operates under the [Code of Conduct](CODE_OF_CONDUCT.md) published in the repository. The Code of Conduct may evolve through governance review.

### Accessibility commitment

ATAH commits to accessibility-conscious operation. Documentation, public interfaces, and partner-facing tools should be developed with attention to accessibility standards. Specific accessibility requirements are set out in the specification and may evolve through governance review.

---

## Amendments

**Core commitments (Part One)** may be amended only through the following process:

- During the interim governance period: amendment requires the agreement of the founder and the interim advisory group, with a 30-day public notice period and public explanation of the reason for change. Emergency amendments for legal compliance only may proceed on a shorter timeline with retrospective public explanation.
- After transition to independent governance: amendment requires a supermajority of at least 75% of the independent governance body, following a 30-day public notice period. The founder advisory seat carries consultation rights but not approval rights for core amendments after transition.

Amendments to core commitments are recorded in the Charter's revision history with an explanation of the reason for change.

**Operational commitments (Part Two)** may be amended through the standard governance process described in [GOVERNANCE.md](GOVERNANCE.md). Pull requests proposing changes are reviewed publicly. Approved changes are merged and a new Charter version is published.

A complete revision history of this Charter is maintained in the repository's commit log.

---

## Effective date

This Charter takes effect from the date of its first commit to the public repository. The founder undertakes to administer ATAH consistently with its commitments from that date forward.

---

## Version history

- **v1.3 — April 2026.** Governance transition reworked: removed the 90-day interim advisory group deadline and the 12-month transition deadline; replaced with trigger-based transition (two trusted partner integrations + one external conforming implementation) and review of transition progress published as warranted. Aligns governance commitments with the realistic adoption timelines of cross-industry standards work. Added Core commitment 8 to Part One (Operational independence): operational service providers permitted on commercial terms; arrangements must be terminable, replaceable, and not exclusive; cannot affect Charter neutrality commitments. Added a clarifying paragraph to Core commitment 1 (Open specification) addressing the boundary between forks/derivative works (permitted under the open licence, but not "official ATAH") and conforming implementations (operating the protocol, part of the ecosystem). Added a clarifying paragraph to Core commitment 7 distinguishing governance compensation from operational service-provider compensation, confirming the founder advisory seat applies to the official ATAH governance line only (not to forks or derivative protocols), and defining "founder-affiliated entity" once for use throughout the Charter and GOVERNANCE.md. Softened the headline reference in Core commitment 7 from "permanent advisory seat" to "continuing advisory seat" (substance unchanged; structural references where permanence is operative remain). Added "Operational support and service providers" subsection to Part Two with cross-reference to Core commitment 8 and to GOVERNANCE.md §4.1. Operational machinery (permitted work types, payment structures, related-party register requirements, replaceability application) lives in the new GOVERNANCE.md §4.1.
- **v1.2 — April 2026.** Protocol-layer refinement. Updated Open specification (Core commitment 1) to make royalty-free protocol implementation explicit and clarify that commercial charges live at the implementation or operational layer. Updated Open MCP endpoint (Core commitment 4) to distinguish the ATAH-operated reference MCP endpoint from the protocol itself, and to clarify that future third-party conforming implementations may operate their own endpoints subject to conformance principles. Added scope clarification that registry-operations references in the Charter apply to the ATAH-operated reference registry. No changes to substantive commitments.
- **v1.1 — April 2026.** Added privacy floor and compliance gating floor as core commitments. Added founder conflict-of-interest, recusal, and disclosure rules. Added concrete transition covenant with deadline. Added interim advisory structure. Added partner principles, professional participation principles, and verifier/review platform principles. Strengthened amendment process. Added accessibility commitment. Clarified the Charter's operation as a public governance covenant.
- **v1.0 — April 2026.** Initial Charter.

---

*Founded by Grahame Cohen. Published at [github.com/atahprotocol/atahprotocol](https://github.com/atahprotocol/atahprotocol). Contact: [grahame@atahprotocol.org](mailto:grahame@atahprotocol.org).*
