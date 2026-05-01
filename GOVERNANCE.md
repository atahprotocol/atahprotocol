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

**Maintenance posture.** ATAH at v0.8.1 publication is maintained by a small team — currently a single founder with significant other commitments. The SLAs published in this document and in SECURITY.md are best-effort targets reflecting that posture. Where response times are missed because of capacity constraints, that is itself information worth surfacing in any review of governance status. The protocol's path to sustained, well-staffed maintenance runs through partner integrations, external implementer engagement, and the eventual transition to independent governance.

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

## 7. Amendment of this document

This document is operational. Substantive amendments are made through public
PR and consulted with the interim advisory group (post-transition: with the
independent governance body). Editorial corrections may be merged without
consultation but are listed in CHANGELOG.

## 8. Contact

- Governance: `governance@atahprotocol.org`
- General: `hello@atahprotocol.org`
- Security: `security@atahprotocol.org`
