# Governance

**Date: April 2026**
**Status: Interim governance (pre-transition).**

This document describes how decisions are made for the ATAH protocol and the
ATAH-operated reference registry during the interim period before transition to
independent governance. It is the operational companion to the
[Charter](CHARTER.md). Where the Charter specifies *what* commitments bind the
project, this document specifies *how* decisions are made and recorded.

## 1. Purpose

The Charter is the public governance covenant and founding commitment. This document is operational. Where the two disagree, the Charter prevails. Substantive amendments to either are made through the processes below.

### 1.1 Three governance layers (per Charter Core Commitment 8)

Per Charter Core Commitment 8 (strengthened in v0.8.2 with Paolo Piponi's F1.6 verbatim normative rule), ATAH governance is structured in three deliberate layers:

- **Protocol governance body.** The independent not-for-profit or equivalent public-interest entity. Owns the specification, conformance marks, category annex process, partner / verifier admission rules, transparency rules, and neutrality audits. **Form is mandated.** This layer is what this GOVERNANCE.md document operates today (during the interim period before transition to independent governance per Charter §"Transition to independent governance"); the form is committed to in the Charter.
- **Reference registry operator.** May be not-for-profit, public-benefit, community-interest company, foundation-owned subsidiary, or commercial under strict constraints. Critical requirements: enforceable neutrality, public fee schedules, auditability, data-use limits, structural separation from the protocol governance body. **Form is constrained, not mandated.** The initial reference registry is operated by ATAH-the-organisation in v0.8.x; future reference registries (e.g. operated by partners or successor organisations) follow the same constraint set.
- **Third-party conforming implementations.** May be commercial, public-sector, professional-body, or not-for-profit, provided they meet conformance requirements (Core Object / Binding / Registry / Governance / Transparency — see CONFORMANCE.md). **Form is unconstrained.**

The mandate above applies to the protocol governance body specifically. Legal form alone does not solve neutrality — a not-for-profit can be captured by funders or incumbents; a commercial entity can behave neutrally if constrained, audited, and transparent. The Charter Core Commitment 8 wording reflects this: form mandate at the protocol layer; structural constraints at the reference-registry layer; conformance requirements (uniform across implementations of any form) at the third-party layer.

The hard-artifacts list (per Charter Part Two and Paolo's F1.7) is the structural complement to this layered governance: the trust floor lives in concrete published artifacts (verification scopes, partner admission criteria, mandatory audit events, conformance tests, revocable conformance mark, etc.), not in committee discretion. Governance discretion applies to evolution and edge cases; the trust floor stays uniform.

## 2. Current state — interim governance

Until transition to independent governance:

- **The founder operates as decision-maker** for governance matters not
  requiring interim advisory consultation. The founder is named and the
  founder's role, conflicts, recusal duties, and compensation are public per
  Charter §"Founder accountability".
- **An interim advisory group will be convened** when suitable members are identified and available to serve. Composition principles, member identities, and operating procedures will be published here when finalised.
- **Decisions requiring interim advisory consultation** (per Charter §"Interim
  advisory structure"): approval of the first independent verifier; approval
  of the first review platform; first commercial partner agreement; first
  category compliance annex; any Charter change.

**Maintenance posture.** ATAH at v0.8.1 publication is operating with the resourcing typical of a release-candidate open standard: a small founding team, publishing the protocol openly to attract reviewers, partners, implementers, and advisors. The SLAs published in this document and in SECURITY.md are best-effort targets at release-candidate stage. As partner integrations and external implementations mature, governance and operational capacity grow alongside — which is exactly what the trigger-based transition framework anticipates. The protocol's path to sustained, well-staffed maintenance runs through partner conversations, external implementer engagement, and the eventual transition to independent governance.

## 3. Decision types

| Decision | Pre-transition process | Post-transition process |
|---|---|---|
| Operational decisions (day-to-day) | Founder | Independent governance body operating under published procedures |
| Charter Part Two (operational commitments) amendments | Interim advisory consultation; public PR; merge after public review window | Independent body decision under published procedures |
| Charter Part One (core commitments) amendments | Interim advisory consultation **and** 30-day public notice; founder may not amend without explicit advisory consultation | 75% supermajority of independent governance body **and** 30-day public notice |
| Compliance annex approvals | Interim advisory consultation | Independent body |
| Partner approvals (after first) | Public criteria; founder decision; public registry | Independent body decision; public registry |
| Verifier approvals (after first) | As partners, plus annual audit requirement | As partners, plus annual audit requirement |
| Specification changes | Public PR; substantive changes through governance review per CONTRIBUTING.md | Public PR; independent body review |

## 4. Public registers

The following are published and kept current:

- Trusted partner registry (`tp-*` records)
  - Standard partner agreement template (including data-protection terms
    appropriate to the partner's role under GDPR and equivalent frameworks)
- Independent verifier registry (`iv-*` records)
- Review platform classifications and their anti-gaming attestations
- Category compliance annexes
- Fee schedules (for any paid registry feature, partner agreement, or verifier
  approval cost)
- Founder conflict-of-interest disclosures (annual)
- Founder compensation (annual)
- Related-party transactions involving any ATAH ecosystem entity in which the
  founder, advisors, or governance members hold a material interest

### 4.1 Operational support, service providers, and related-party reimbursement

ATAH and any ATAH-operated reference registry may be supported and operated by service providers performing real operational work. This includes — without limitation — operational, administrative, technical, legal, PR, outreach, funding, partner-introduction, reference-implementation, platform-build, managed-service, hosting, security, data-onboarding, professional-support, compliance-operations, or governance-formation work. Service providers may include independent contractors, technology partners, implementation partners, managed-service providers, employees, contributors, or other supporting organisations.

Permitted commercial arrangements include documented cost reimbursement, fixed-fee service contracts, time-and-materials arrangements, cost-plus arrangements, market-rate managed-service fees, and other documented service-provider fees approved or ratified through the applicable governance process. Payments compensate work performed; they do not purchase ranking, visibility, approval, conformance, access to non-public data, or influence over protocol decisions.

Where a service provider is the founder, a founder-affiliated entity, an advisor-affiliated entity, or any other related party, the arrangement is treated as a related-party transaction. It is recorded in the public related-party register established in §4 above, with: the identity of the related party; the nature of the work or support provided; the basis of payment or reimbursement; the governance review, approval, or ratification process applied; and any recusal by the founder, advisor, or governance participant with a material interest. Reimbursement and payment are limited to reasonable, documented work or costs actually incurred for ATAH-related purposes.

No service-provider arrangement — whether or not the provider is a related party — may be structured as a protocol royalty, paid-placement fee, referral fee, success fee, permanent revenue share, or commercial return linked to registry growth, professional ranking, partner approval, introduction volume, or user-facing visibility. No service-provider arrangement may affect matching, ranking, provenance, conformance status, partner approval, platform access, professional visibility, data access, or any other decision governed by the Charter's neutrality commitments.

Per Part One of the Charter (Operational independence), every operational service-provider arrangement must be terminable on reasonable notice, replaceable, and not exclusive. A founder-affiliated or related-party service provider may be selected during the pre-transition period where doing so is reasonable for incubation, continuity, cost, speed, expertise, or availability — the rationale is documented in the related-party register. Once independent governance is in prospect, material ongoing related-party service arrangements are reviewed and, where appropriate, re-tendered or made terminable on reasonable notice so that independent governance is not locked into a non-replaceable provider.

The existence of a paid service-provider relationship does not confer ownership of the protocol or any right to control its evolution. The ATAH specification remains open and royalty-free to implement, and conforming implementations may be built or operated by other parties subject to the conformance requirements.

## 5. Charter compliance objections

Anyone may raise an objection that a decision or action contradicts the
Charter. The process:

- **How to raise:** open a GitHub issue with the `governance` label, **and**
  email `governance@atahprotocol.org`. Both routes accepted; the email
  guarantees acknowledgement.
- **Acknowledgement:** within 7 days.
- **Decision:** within 30 days of acknowledgement, unless complexity requires
  an explicit time extension communicated publicly.
- **Public remedy on confirmed breach:** the action is reversed, the
  decision-maker publishes an explanation, and a remediation step is recorded
  in the public register.

Objections are handled in good faith. Vexatious or bad-faith objections may be
closed; the closure rationale is recorded publicly.

## 6. Transition to independent governance

The Charter sets the transition framework: transition is triggered when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated. Independent governance is appropriate when there is an ecosystem to govern; the trigger conditions reflect that the protocol works with cross-industry organisations whose decision cycles are inherently slower than software-industry timelines.

The transition process, once triggered:

1. The interim advisory group (or, if it has not yet been convened by trigger time, an advisory body convened at that point) takes legal advice on candidate structures (foundation, association, multi-stakeholder body, etc.), and conducts public consultation on the chosen structure.
2. Decision is made transparently, with reasons published.
3. The independent body is constituted and assumes the decisions listed in §3 above.
4. The founder seat continues per Charter §"Founder's role and compensation" — a single seat, no veto, with the same recusal duties as any other seat-holder.
5. The advisory group is dissolved or restructured into a permanent role of the new body, as appropriate.

Annual review of transition progress is published per the Charter when there is meaningful progress to report. The review reports on progress toward the trigger conditions, the state of partner integrations, and any factors affecting the path to independent governance. Where circumstances change in a way that affects the transition approach, the change is explained publicly.

## 7. Concern flag data handling

ATAH stores concern flags raised against named professionals as part of the
ecosystem-integrity mechanism described in spec §5.10. Because concern flag
data identifies individuals and may carry derogatory content, the following
data handling rules apply. For the lifecycle (intake through escalation) see
PRD §8.11.

**Lawful basis (UK GDPR / EU GDPR).** Processing rests on legitimate interests
in maintaining the integrity of the professional handoff ecosystem and the
safety of consumers using AI systems that rely on ATAH-conformant registries.
A balancing test is performed and documented at the point each new flag type
or escalation pathway is introduced. The balancing test considers the
professional's reasonable expectations, the proportionality of processing,
the necessity of identifiable data, and the availability of less intrusive
alternatives.

**Retention.** Unresolved flags are retained for 12 months from the date of
submission unless extended by an active investigation. Flags resolved as
substantiated and forming part of a disclosed pattern are retained for
3 years. Flags resolved as unsubstantiated or withdrawn are pseudonymised
within 30 days of resolution and deleted within 90 days. Flags determined to
be bad-faith (per spec §5.10's bad-faith detection) are deleted or
pseudonymised at the point of determination.

**Access.** Concern flag content is admin-only. Role-restricted, audit-logged,
no public exposure. Partners and verifiers do not see flag content.

**Professional rights.**
- **Notification.** The professional named is notified of any flag against
  them within 7 days of submission, unless notification would compromise an
  active safety investigation, in which case notification is deferred until
  the investigation completes.
- **Right of reply.** 30 days from notification.
- **Rectification.** The professional may submit a correction request at any
  time. Material disputes follow the dispute resolution process in spec
  §8.10.
- **Erasure.** Where a flag is determined unsubstantiated or bad-faith, the
  professional may request immediate deletion rather than awaiting the
  scheduled pseudonymisation/deletion timeline.

**Escalation.** Flags are escalated to the relevant regulatory or
professional body only where (i) a documented threshold is met (multiple
substantiated flags, severity threshold, or pattern indicating systemic
concern), (ii) the escalation has been reviewed by the governance body or
its delegated authority, and (iii) the professional has been notified of
the escalation. ATAH does not adjudicate professional conduct; escalation
delivers what ATAH knows to the body with authority to act.

**Wording control.** All flag content distinguishes "unverified concern"
from "finding." ATAH does not surface flag content to anyone outside the
admin role; if any future surfacing is proposed, it requires a Charter
amendment under the Part One core commitments process.

Detailed operational procedures (the threshold values, escalation criteria,
review cadence) are maintained as administrative procedure documents and may
evolve through governance review without amendment to this section.

## 8. Amendment of this document

This document is operational. Substantive amendments are made through public
PR and consulted with the interim advisory group (post-transition: with the
independent governance body). Editorial corrections may be merged without
consultation but are listed in CHANGELOG.

## 9. Contact

- Governance: `governance@atahprotocol.org`
- General: `hello@atahprotocol.org`
- Security: `security@atahprotocol.org`
