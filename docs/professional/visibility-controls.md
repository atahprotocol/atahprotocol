# Visibility controls

**Status:** operational documentation for professionals listed in an ATAH-conformant registry. Describes the visibility controls available to you and how to use them. Not normative protocol text — the underlying mechanisms are defined in the [specification](../../spec/v0.8/full-spec.md), and the operational implementation lives in the reference registry.

**Audience:** professionals — credentialled or established — who appear (or expect to appear) in an ATAH-conformant registry, whether through a trusted partner integration or by individual self-registration.

**Companion to:** [spec §10.4 Registration Routes](../../spec/v0.8/full-spec.md), [PRD §6.1 Credentialled and Established Professionals](../../PRD-v0_8.md), [Charter Part Two professional participation principles](../../CHARTER.md).

---

## What this document covers

If you appear in an ATAH-conformant registry, you have meaningful control over how, when, and to whom you are visible. This document describes the visibility classes available, how to move between them, and the limits — what ATAH can and cannot remove from your record.

The controls work together with the Stage 1 appetite-check preferences (described in [PRD §6.1](../../PRD-v0_8.md)) — those preferences shape which introductions are routed to you when you are visible. Visibility controls described here are the prior question of whether you are visible at all.

## Visibility classes

The reference registry recognises four visibility classes, set per record:

**Visible in matching.** Your record is returned in match responses to AI platforms when the query parameters fit. This is the default state for an active record (`matching_status: active` in the data model).

**Visible to registry administrators only.** Your record is held in the registry but is not returned in match responses. Administrators can see it for governance and dispute purposes; AI platforms cannot. This state is used while a dispute is being resolved or while a roll-up merge is being assessed (`matching_status: pending-rollup` or `pending-integrity-review`).

**Suppressed pending dispute.** A specific case of administrator-only visibility: your record is suppressed from matching because data on it is being disputed (a partner-supplied field, a disciplinary entry, a roll-up merge you've objected to). Suppression continues until the dispute resolves. You retain access to your portal and can continue to update self-declared fields throughout (`matching_status: admin-suspended` with a reason code indicating the dispute).

**Withdrawn.** Your record is marked withdrawn and is not returned in match responses. The record is retained for audit and dispute-history purposes but is not active. Withdrawal is reversible — you can reinstate visibility later (`matching_status: withdrawn`).

There are also two regulator-driven states the professional cannot self-set, listed for completeness: `regulatory-suspended` (a regulator has suspended the licence and the registry reflects that fact) and `compliance-pending` (a category-level compliance annex has not yet been completed for this professional's category, so the record is open for registration but excluded from matching pending the annex).

## How to change your visibility

Three routes:

**Through the professional portal.** Sign in to your portal and update your visibility setting. Changes take effect within minutes. The portal also shows you the current state and the history of state changes on your record.

**Through your connected AI assistant.** If you have connected an AI assistant to your professional record (an MCP-connected assistant operating on your behalf), you can ask it to change your visibility. The assistant invokes the relevant ATAH operation through the protocol, with the same effect as a portal change.

**Through your partner-managed record (where applicable).** If you appear in the registry through a trusted partner — a regulator, your professional body, or another approved partner — your partner-managed fields cannot be changed by you directly (partner data is canonical for those fields, per the partner's data integration). You can still set your own visibility class for the record, but partner-managed factual data continues to flow from the partner. If you want to dispute a partner-managed field, see the disputed-inclusion section below.

## Right to be excluded

You have several flavours of exclusion available:

**Full opt-out.** Withdraw your record from the registry entirely. The record is marked withdrawn, no longer returned in match responses, and retained only as needed for audit and dispute history. You can re-register later if you choose.

**Category opt-out.** Stay visible in some categories but not others. Useful if you are a professional with multiple specialisms (a tax planner who is also qualified to handle estate matters, for example) and you want to receive introductions in some specialisms but not others. Configured through the matter-types and specialisms fields on your profile.

**Jurisdictional opt-out.** Stay visible in some jurisdictions but not others. Useful if you are licensed in multiple jurisdictions but only practising in some. Configured through the licensed-jurisdictions and service-locations fields. The cross-jurisdiction default ([PRD §8.5](../../PRD-v0_8.md)) means cross-jurisdiction matches require explicit query parameters anyway, so jurisdictional opt-out at the professional side combines with the protocol-level default to keep cross-jurisdictional exposure controlled.

**"Not currently accepting introductions."** A softer state than withdrawal. You remain in the registry, your record is visible to AI platforms in match responses, but the response indicates you are not currently accepting new introductions. Useful for capacity-managed periods, holiday, parental leave, or while you are concentrating on existing matters. Configured through the availability-status field.

**Firm-managed only.** If your record is connected to a firm administration, you can set the record to receive introductions only via firm intake, not directly. Useful for firms with central intake teams and conflict-checking processes the firm wants all introductions to flow through.

## Disputed inclusion

A record may have been created without your knowledge — typically through a partner-route registration where a body you belong to has joined ATAH as a trusted partner and contributed member data, or through an automated verification against a public regulatory database. The reference registry handles this:

**Pre-merge notification window.** For high-risk regulated-tier categories, you are notified before partner data is merged into a canonical record about you, with a 7-day window to object (per [spec §10.4](../../spec/v0.8/full-spec.md)). If you object, the merge does not complete until the dispute resolves.

**Right of objection at any time.** Even outside the pre-merge window — for example, if you discover an existing record about you and disagree with how it was created — you can object through the dispute flow. Disputes are logged, the contributing partner is notified, and the disputed fields are flagged in the profile (and may be suppressed from matching where the conflict is material).

**Resolution timeline.** Disputes have a published SLA in the operational documentation. Where the partner doesn't respond, ATAH admin reviews and may resolve in your favour. The professional retains right of reply throughout.

**What gets resolved.** A dispute resolves either (a) the partner data is confirmed canonical and the merge or update completes; (b) your self-declared data is confirmed correct and the partner data is recorded as disputed (the partner is notified of the conflict); or (c) the conflict cannot be resolved cleanly, in which case the affected fields remain suppressed from matching pending further evidence.

## Reinstating visibility after suppression

Suppression states are not permanent — they reflect a current data quality or dispute condition. When that condition resolves, your visibility reinstates:

**After dispute resolution.** When a disputed field resolves in your favour or the partner update is corrected, the record returns to `active` and visibility resumes.

**After roll-up merge completes (or is rejected).** When a roll-up merge that triggered `pending-rollup` either completes (with your participation, per the pre-merge notification window) or is rejected (because you objected), the record returns to `active` or remains as your existing self-declared record.

**After category compliance annex completion.** A `compliance-pending` record becomes matchable when the category annex completes and your category opens for matching. You don't need to take action — the registry surfaces you when the gating condition resolves.

**After voluntary withdrawal.** If you withdrew your record and later want to reinstate, sign in to the portal (or use your connected AI assistant) and reactivate. Existing data on the record is preserved unless you opted to delete the record entirely.

**After regulatory suspension lifts.** If your regulator's data shows your licence reinstated after suspension, the registry's verification cycle picks this up and your `regulatory-suspended` state changes to `active` automatically.

## What ATAH cannot remove

Two limits are worth being clear about, both flowing from the protocol's interaction with regulatory structures rather than from internal policy:

**Regulator-sourced data covered by public-task carve-outs.** Where a regulator's record is the canonical data source — your licence number and active status, for example — the lawful basis ATAH relies on is the regulator's public-task or legal-obligation processing. You can object to ATAH surfacing this data, and ATAH will assess that objection, but the underlying regulator's record is outside ATAH's control. Where you successfully object, ATAH may suppress your record from matching while retaining the minimal data required for audit and conflict-resolution purposes (per [PRD §9 professional data protection](../../PRD-v0_8.md)).

**Audit log entries.** Audit log entries on state transitions and consent assertions are pseudonymous and retained for seven years on a legitimate-interest basis (fraud detection, dispute resolution, regulator engagement). They do not directly identify you within ATAH alone but may constitute personal data where linkable. They are not removed on visibility change or withdrawal — withdrawal stops the record being surfaced; the audit trail of past actions on it remains. (Per [SECURITY.md §7](../../SECURITY.md) and [PRD §9](../../PRD-v0_8.md).)

## A note on referral connections (Component 3)

If you have established Component 3 referral connections with other professionals through ATAH, withdrawing your record from matching does not automatically end those connections. Component 3 connections persist while both parties remain connected; either party may withdraw at any time via `POST /v1/referral-connections/:connection_id/withdraw` or the equivalent MCP tool, and the connection record is then deleted.

Component 3 connections are kept for one operational purpose only — de-duplication of `request_intent: 'referral_partner_search'` Discovery results so that already-connected professionals do not see each other in their referral-partner candidate sets — and **do not feed matching as a competence or trust signal** anywhere in the protocol. The looking-toggle (`looking_for_referral_partners`, default off) is the visibility control: setting it to `false` excludes you from the candidate pool for new inbound Component 3 proposals without affecting existing connections; toggling back to `true` puts you back in the pool.

For full detail on Component 3 referral connection-making, see spec §6.4 and the README section on referral relationships between professionals.

## How to act on these controls

The shortest path: sign in to your professional portal. Every visibility class above is configurable from the portal. The portal also surfaces a status panel showing current visibility, pending disputes, and any roll-up merges in flight.

If you do not yet have portal access — for example, because you have just become aware of a record about you that you did not create — start with the dispute flow. The portal supports unauthenticated dispute submission for exactly this case: you provide enough information for ATAH admin to identify the record and connect it to your verified channel, after which dispute resolution proceeds and (where appropriate) you are invited to claim the record and gain full portal access.

For specific operational steps — claiming a record, setting a visibility class, configuring Stage 1 appetite preferences, submitting a dispute — see the operational guides published alongside the reference registry. This document covers the framework; the operational guides cover the buttons.

## Reference

For the protocol-level definition of registration routes and roll-up handling, see [spec §10.4](../../spec/v0.8/full-spec.md). For Stage 1 appetite-check preferences (which compose with visibility controls to shape what reaches you), see [PRD §6.1](../../PRD-v0_8.md). For the dispute-resolution flow, see [spec §5.9](../../spec/v0.8/full-spec.md). For audit log treatment, see [PRD §9 professional data protection](../../PRD-v0_8.md) and [SECURITY.md §7](../../SECURITY.md). For the foundational professional participation principles, see [Charter Part Two](../../CHARTER.md).
