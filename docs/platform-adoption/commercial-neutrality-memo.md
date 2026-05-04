# Commercial neutrality memo

**Status:** explanatory memo for AI platform legal and integration teams. Not normative — the binding commitments live in the [Charter](../../CHARTER.md) (Part One core commitments and Part Two partner principles). This memo gathers the evidence in one place so a platform classifying ATAH against its own commercial-relationship policies can see the structure quickly.

**Audience:** AI platform legal teams, partnership teams, and policy reviewers assessing whether ATAH falls within their existing partner / integration / lead-generation / ad-tech classifications.

**Companion to:** [Charter Part One core commitments](../../CHARTER.md), [Charter Part Two partner principles](../../CHARTER.md), [PRD §11 The Commercial Model](../../PRD-v0_8.md), [PRD §6.3 Trusted Partners and partner taxonomy](../../PRD-v0_8.md), [GOVERNANCE.md](../../GOVERNANCE.md).

---

## Why this memo exists

AI platform reviewers classify integrations against existing buckets — search providers, content partners, advertising networks, lead-generation services, regulated-product directories. The classification matters because it determines which user-facing rules apply: disclosure requirements, paid-placement labelling, ad-policy review, lead-gen consumer protections, and so on.

ATAH does not fit cleanly into any of those buckets. It is closer to neutral infrastructure — comparable in structure to standards bodies that publish open specifications and operate reference implementations than to commercial directories. The risk if a platform mis-classifies ATAH is that the platform applies disclosure rules calibrated for paid placement (which ATAH structurally cannot have) or lead-generation (which ATAH structurally cannot do).

This memo sets out the structural evidence so the classification conversation can be quick.

## What ATAH does not do

The following are not policy commitments that could be loosened later. They are entrenched core commitments in [Charter Part One](../../CHARTER.md) and require the supermajority core-commitment amendment process to change.

### No commercial weighting in matching

The matching logic carries no commercial weighting at any point. Partner integration fees, professional registration fees, AI platform relationships, enhanced verification fees, review platform integration arrangements, and any other commercial relationship in or around ATAH have no influence on:

- Whether a professional is returned in a match result
- The position of a professional in a candidate set
- The trust signals attached to a professional's profile
- The recommendation status of any professional or partner

Matching is determined by hard filters (eligibility), transparent bands (verification confidence, category fit, availability window, contact freshness), and within-band randomisation under a documented fairness policy — never by who has paid what to whom. There is no global match score in `match-response.schema.json` and the conformance test verifies that within-band ordering is observably non-deterministic across repeated queries. Where paid services generate verification evidence, that evidence is assessed under the same public rubric available to all approved sources, regardless of who commissioned or paid for it. (Source: [Charter Part One core commitment 2](../../CHARTER.md); [spec §9.1 matching pipeline](../../spec/v0.8/full-spec.md).)

### No per-query monetisation of the MCP endpoint

The ATAH-operated reference MCP endpoint remains free for any AI platform to query. There is no per-query fee, no licensing fee for AI platform implementation of the protocol, and no commercial gate on protocol access at the platform layer. Free does not mean unauthenticated: querying still requires authentication, rate limits, and compliance with developer terms to protect consumers and registry integrity. The commitment is that authentication and operational controls exist solely to protect the system, not to monetise platform access. (Source: [Charter Part One core commitment 4](../../CHARTER.md).)

### No paid placement, no paid ranking

There is no paid-placement mechanism in ATAH. A professional cannot pay to be returned in matches they would not otherwise be returned in, cannot pay to rank higher than they otherwise would, and cannot pay to suppress competitors. Listing fees (where they apply, for individual self-registration after the founding-period free window closes) are flat fees for registry presence — not referral fees, not contingent on consumer engagement or outcome, not paid on a per-introduction basis. (Source: [Charter Part Two partner principles](../../CHARTER.md), [PRD §6.1](../../PRD-v0_8.md), [PRD §11](../../PRD-v0_8.md).)

### No referral fees, no per-introduction fees

ATAH does not permit per-referral fees, outcome-contingent fees, or ranking based on payment. This is particularly load-bearing for legal services, where many jurisdictions' professional rules prohibit referral arrangements; it is enforced as a category-neutral structural rule rather than category-specific. (Source: [PRD §9 lawyers entry](../../PRD-v0_8.md), [Charter Part Two](../../CHARTER.md).)

### No exclusive partner arrangements

No exclusivity in any category. Multiple regulatory bodies, professional bodies, verifiers, and review platforms may operate in any category. (Source: [Charter Part Two partner principles](../../CHARTER.md).)

### No use of platform query data to compete with platforms

The matter described in a query, the consumer reference, and the consumer-side context that the AI platform supplies are used solely for the matching the platform is asking for. ATAH does not use platform-query data to derive insight, to populate competing products, or to share with third parties beyond what is needed to fulfil the matching request. (Source: [Charter Part One core commitment 5 — privacy floor](../../CHARTER.md), [PRD §9](../../PRD-v0_8.md).)

## What ATAH charges for, and why those charges are cost-recovery

The protocol is royalty-free to implement. Commercial charges exist only at the implementation or operational layer of the ATAH-operated reference registry. They fund the work of running the registry, governing the partner and verifier networks, maintaining integrations, and supporting the introduction lifecycle.

### Trusted partner integration fees

Partners pay ATAH for integration access. Partner participation funds the work of building and maintaining the protocol. Partner status is granted on the basis of data quality, freshness commitments, and verification capability — never on the basis of payment alone.

The partner principles applying to these fees:

- A published partner fee schedule with cost-recovery rationale
- Fee waivers or deferred fees for regulatory bodies and public-interest organisations
- No exclusive arrangements within any category
- A public partner registry showing partner status, scope, and approval date
- Data quality criteria independent of payment level
- Equal data-handling and provenance treatment regardless of fee tier

(Source: [Charter Part Two trusted partner participation](../../CHARTER.md).)

### Independent verifier governance fees

Verifiers pay ATAH a non-refundable assessment fee at application (covering the suitability review whether or not approval is granted) and an annual audit fee once approved (covering yearly methodology adherence audit). Both fees are tiered by the verifier's declared scope — narrow scope, multi-scope, or global scope.

ATAH does not take a share of the verification fees the verifier charges its commissioning parties. The verifier sets that price; ATAH governance fees cover the governance and audit work that keeps the verifier model credible. (Source: [PRD §6.3 enhanced verification](../../PRD-v0_8.md).)

### Individual self-registration listing fee

Free during the protocol's initial period after launch. After the initial period closes, individual self-registration remains available with a small annual fee. Hardship waivers and fee caps are published in operational documentation. The fee is for registry presence — not a referral fee, not contingent on consumer engagement or outcome, not paid on a per-introduction basis. (Source: [Charter Part Two professional participation](../../CHARTER.md).)

## The operational service provider model

The protocol — the open specification — is governed by ATAH governance under the Charter. The reference registry that operates the protocol's first endpoint is a separate concern: it is software and operations, not the protocol itself.

ATAH governance contracts with one or more **operational service providers** to run the reference registry and provide adjacent operational support. The relationship is structured around a replaceability principle: any operational service provider operates under terms equally available to others, can be replaced or terminated, and is non-exclusive. The protocol does not depend on any single operational service provider for its continued existence.

### Allowed fee models for operational service providers

Operational service providers may receive cost reimbursement, fixed-fee, time-and-materials, cost-plus, or market-rate managed-service fees. These are commercial arrangements for operational work, structured transparently, and do not affect protocol matching, provenance presentation, or governance decisions.

### Forbidden fee models for operational service providers

Operational service providers may NOT receive:

- Protocol royalty
- Paid placement or any commercial weighting in matching
- Referral fees or per-introduction fees
- Success fees contingent on registry growth or outcome
- Permanent revenue share

A service provider operating the registry has the same relationship to matching outcomes as any other professional or partner: none.

### Founder-affiliated entities

Any founder-affiliated entity that participates as an operational service provider operates under terms equally available to any other service provider. The replaceability principle applies. The Charter's conflict-of-interest, recusal, and disclosure rules ([Charter §7](../../CHARTER.md)) apply to the founder personally and to any founder-affiliated entity in the ecosystem — annual public disclosure, recusal from governance decisions affecting the entity, public register of related-party transactions, no privileged access to non-public partner, platform, or registry data beyond what is available to other governance body members.

The framework applies to any present or future founder-affiliated entity. The intent is to make the ecosystem auditable rather than to assert that no commercial relationship between governance and operations may exist. The auditability is the structural protection.

## Auditable commitments

Several commitments are auditable from the published material rather than from internal documents — meaning a platform reviewer can verify them against the public artefacts.

| Commitment | Auditable from |
|---|---|
| Public partner fee schedule | [Charter Part Two](../../CHARTER.md); operational fee schedule published in registry documentation |
| Cost-recovery rationale | [Charter Part Two](../../CHARTER.md); fee schedule notes |
| Public verifier acceptance criteria and scoring rubric | [Charter Part Two](../../CHARTER.md); [PRD §6.3 enhanced verification](../../PRD-v0_8.md); [spec §5.7](../../spec/v0.8/full-spec.md) |
| Public review platform acceptance criteria | [Charter Part Two](../../CHARTER.md); [PRD §6.3 review platforms](../../PRD-v0_8.md); [spec §5.8](../../spec/v0.8/full-spec.md) |
| Public matching algorithm rubric | [PRD §8.5 matching engine](../../PRD-v0_8.md); [spec §9](../../spec/v0.8/full-spec.md) |
| Provenance visibility per field | [`provenance-map.schema.json`](../../spec/v0.8/schemas/provenance-map.schema.json); [spec §4.2](../../spec/v0.8/full-spec.md) |
| `presentation_disclosure` block on every match response | [`match-response.schema.json`](../../spec/v0.8/schemas/match-response.schema.json); [spec §4.12](../../spec/v0.8/full-spec.md) |
| `commercial_weighting: false` machine-readable | `presentation_disclosure.commercial_weighting` field, `const: false` |
| Founder conflict-of-interest disclosures | [Charter §7](../../CHARTER.md); annual public disclosure published in operational documentation |
| Related-party transactions register | [Charter §7](../../CHARTER.md); public register published in operational documentation |

The `commercial_weighting: false` field deserves a closer look: it is a JSON Schema constant in `match-response.schema.json` — a machine-readable, conformance-test-checkable assertion that the matching engine that produced a given response did not apply commercial weighting. Any conforming implementation that violates this would be producing schema-invalid responses.

## Conformance status

ATAH does not currently issue certifications, badges, or formal endorsements of any implementation. At v0.8.2, conformance statements are self-declarations by the implementing party. The published [CONFORMANCE.md](../../CONFORMANCE.md) describes the five conformance classes (Core Object, Binding, Registry, Governance, Transparency) and the conformance statement requirement. The Transparency Class makes machine-readable explanations a normative obligation: response-level and per-candidate `decision_explanation` on every match response, plus aggregate `exclusion_summary` for excluded candidates and a dedicated governance-only retrieval endpoint for named-exclusion detail.

The v0.9 conformance test suite, when it ships, will provide schema-validating, lifecycle-exercising, behaviour-confirming tests that move conformance from self-declaration toward verifiable testing. Until then, ATAH may publicly contest conformance statements that misrepresent compliance with the protocol or with the conformance principles in the specification.

For the purposes of platform classification, the practical consequence is: the reference registry's conformance statement (and any other implementation's statement) is currently a published claim that platforms can read and assess against the protocol's normative text, not a certification ATAH has independently audited.

## Forks and official conformance recognition

The protocol is published under Apache 2.0. Anyone may implement, fork, or extend it. Forks are permitted by the licence and do not require ATAH approval.

A separate question is which implementations may claim "official ATAH conformance" recognition. Recognition is reserved for implementations that meet the conformance criteria in the specification (Section 14) and publish a conformance statement at the discoverable location. The founder advisory seat described in the Charter applies to the official ATAH governance line — not to forks. A fork may diverge in any way the fork's maintainers choose, but a divergent fork cannot claim conformance with the official ATAH protocol unless it preserves the conformance principles.

For platform reviewers: when an integration claims "ATAH-conformant," the relevant question is whether the implementation has published a conformance statement at the discoverable location and whether the matching response objects validate against the published schemas. Both are checkable from public artefacts.

## Behavioural neutrality, not licence-derived neutrality

> Open licensing strengthens ATAH as a protocol, but it also means the reference registry must compete on trusted governance, neutrality, auditability, and recognised conformance — not merely on authorship of the standard.

Use of the open Apache 2.0 protocol alone does not prove neutrality. Neutrality depends on — and is verifiable through — the governance commitments, ordering-policy disclosure, decision-explanation discipline, partner scope discipline, conflict-of-interest rules, and audit posture documented across the [specification](../../spec/v0.8/full-spec.md), [`CONFORMANCE.md`](../../CONFORMANCE.md), and [`CHARTER.md`](../../CHARTER.md). A purely technical conformance regime may not catch hidden commercial weighting or biased ordering in proprietary implementation logic. The role of ATAH governance is therefore not to prevent forks (Apache 2.0 permits them), but to preserve the meaning of official ATAH conformance, neutrality, and recognised implementation status.

The Transparency Class is the operational backbone of behavioural neutrality. The response-level and per-candidate `decision_explanation` plus aggregate `exclusion_summary` make ordering and exclusion decisions inspectable; the audit-event-anchored governance retrieval endpoint preserves named-excluded-candidate detail for auditors without leaking it through the consumer surface. Combined with the `presentation_disclosure.commercial_weighting: false` `const` (a schema-checkable assertion) and the `_provenance` map's per-field source labels, the v0.8.2 protocol layer makes commercial weighting structurally visible if it is ever attempted — neutrality is not a self-declared posture, it is a behaviour the published artefacts let any auditor or reviewer test against.

[`CONFORMANCE.md`](../../CONFORMANCE.md) introduces a three-level conformance-status distinction: **protocol-compatible** (uses some ATAH schemas/concepts but does not claim conformance), **ATAH-conformant** (satisfies published technical and behavioural conformance, including transparency and ordering-policy obligations), and **ATAH-recognised neutral implementation** (additionally satisfies the strongest governance, neutrality, auditability, public-disclosure, and conflict-of-interest requirements; may use the strongest official trust mark, revocable). The naming is provisional; the distinction is locked. v0.9 work operationalises the verification regime through the behavioural-neutrality / conformance-audit model. For the v0.8.2 audience, the practical consequence: a platform reviewer asking "is this implementation neutral?" should look at the implementation's claimed conformance level and at the audit posture behind that claim, not at the licence under which the protocol is published.

## Recommended user-facing language for AI platforms presenting ATAH results

This section gives short user-facing copy that helps platforms communicate the commercial-neutrality posture without overstating it. For the full UX disclosure copy library (six common surfaces, model strings, anti-patterns), see [`ux-disclosure-copy.md`](ux-disclosure-copy.md).

**Recommended:**
- "These are verified options based on what you've described, not paid placements."
- "Ranking is based on relevance, verification, availability, and completeness — not on commercial relationships."
- "ATAH does not earn from your selection. Your AI shows you these options because they match what you said you need."

**Avoid:**
- "ATAH-recommended" / "ATAH-approved" / "ATAH-endorsed" — ATAH does not recommend, approve, or endorse individual professionals (per [PRD §11 platform presentation obligations](../../PRD-v0_8.md)).
- "Sponsored result" / "Paid placement" — these labels misdescribe how ATAH results are produced. There is no paid-placement mechanism.
- "Verified by ATAH" — ATAH defines the verification framework and surfaces verification evidence; the verification itself is performed by the relevant authoritative source (regulator, professional body, independent verifier). Better: "Verified by [source]" with the source named per the `_provenance` map.

## Reference

For the full normative commitments, see [CHARTER.md](../../CHARTER.md). For the operational implementation of the partner taxonomy and verifier model, see [PRD §6.3](../../PRD-v0_8.md). For the matching engine rubric and `presentation_disclosure` requirement, see [PRD §8.5](../../PRD-v0_8.md) and [spec §9](../../spec/v0.8/full-spec.md). For governance procedures, see [GOVERNANCE.md](../../GOVERNANCE.md). For the consolidated platform/ATAH liability allocation, see [PRD §11 Liability allocation and platform/ATAH responsibilities](../../PRD-v0_8.md).
