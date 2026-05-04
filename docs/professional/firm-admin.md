# Firm and team administration

**Status:** operational documentation for the ATAH-operated reference registry. Describes how firm-level concepts are handled at v0.8.2, where firms appear in professional records, the operational mechanisms available to firm administrators, and what is not yet supported. Firms and teams as first-class entities in the protocol data model are deferred to v0.9; this document describes the v0.8.2 reference-registry posture, not protocol-mandated behaviour.

**Audience:** firm administrators, intake managers, compliance partners, and IT/operations leads at firms where one or more professionals are (or will be) represented in an ATAH-conformant registry.

**Companion to:** [PRD §6.1 Credentialled and Established Professionals](../../PRD-v0_8.md), [spec §10.4 Registration Routes](../../spec/v0.8/full-spec.md), [spec §15 Professional Records and Registration](../../spec/v0.8/full-spec.md), [docs/professional/visibility-controls.md](visibility-controls.md).

---

## Scope

At v0.8.2 the protocol's data model treats professionals as the primary entity. The `firm` field appears on each professional record as a string identifying the firm; it is a property of the professional, not a separate first-class object with its own identifier, governance, or schema. This is a deliberate v0.8.2 simplification — firm and team as first-class protocol entities (with firm-level identifiers, firm-level data flow, firm-level governance, firm-managed visibility settings expressed through the data model) are a v0.9 candidate.

What that means in practice: at v0.8.2, firm administration in the reference registry runs through operational conventions on top of the per-professional record. Several of the patterns below — delegated administrator, compliance approver, branch controls — are reference-registry features layered on the underlying schema, not protocol-mandated behaviour that other conforming registries are required to implement identically.

This document describes what the reference registry provides. Firms operating with another conforming registry should consult that registry's operational documentation for its equivalents.

## Delegated administrator

The reference registry supports a delegated-administrator role: a firm member (typically a partner, practice manager, or operations lead) with authority to manage records on behalf of colleagues at the firm.

**What the role can do.**
- Update partner-managed-but-firm-controlled fields on colleague records (e.g. firm address, branch assignment, intake routing preferences).
- Set firm-level intake routing on colleagues' records (the "only accept via firm intake" Stage 1 control described in [PRD §6.1](../../PRD-v0_8.md)).
- Submit registration requests for new colleagues (subject to the colleagues completing claim authentication themselves).
- View aggregate visibility status for the firm's listed professionals — who is currently visible, who is suppressed, who has pending disputes — without seeing per-colleague match-by-match analytics.

**What the role cannot do.**
- Set or override partner-managed fields where the partner is the canonical source. Partner-managed fields remain controlled by the partner; firm administrators dispute through the standard dispute flow, like any other professional.
- Change a colleague's `acknowledged_rollup_terms` flag (where set) on the colleague's behalf — this is the colleague's own affirmative acknowledgement, and the firm cannot acknowledge for them.
- Withdraw a colleague's record without the colleague's affirmative authorisation. Withdrawal of a personal professional record is a personal-account-level action.
- Read or reveal Stage 2 / Stage 3 data flowing to colleagues. Per-introduction data is per-professional; the firm administrator does not get a read on the colleague's individual matter intake unless the firm's own intake routing is set to surface introductions through the firm.

**How the role is granted.** Through the professional portal: a colleague nominates the firm administrator on their record. The firm administrator accepts the nomination through their own portal session. The relationship is recorded against both records and is revocable by the colleague at any time.

## Compliance approver

A separate firm-level role used for sign-off on matter-related introductions in firms where compliance review precedes engagement (regulated services in particular):

**What the role does.** Sees firm-managed introductions at Stage 2 (or earlier, depending on firm intake configuration) and approves or declines on the firm's behalf before the introduction proceeds to the engaged professional's individual handling.

**Where the role applies.** Firms can configure the compliance-approver checkpoint into their firm intake routing. Where configured, introductions inbound to firm members route to the compliance approver before reaching the individual professional. The approver acts within the published Stage 2 SLA; declined introductions are returned with an outcome code that does not expose firm internals to the consumer's AI platform.

**Where it does not apply.** Firms not configuring firm intake routing do not have a compliance-approver checkpoint. Individual professionals receive introductions directly per their own Stage 1 preferences, with no firm-level intermediation.

## Branch and location controls

Firms operating in multiple jurisdictions or branches use the location and licensing fields on each colleague's record to manage per-location presence:

- `licensed_jurisdictions` — where the colleague is licensed to practise. Per-jurisdiction.
- `service_locations` — where the colleague is willing to serve clients (which may be broader than where they are licensed for some categories, narrower for others).
- `physical_locations` — where the colleague has a physical office presence. Used for queries with proximity preferences.
- `remote_service_available` — whether the colleague handles matters remotely.

A multi-jurisdiction firm typically sets these per-colleague to reflect each colleague's actual licensing and operating posture. The firm administrator can update branch-related fields on behalf of colleagues; jurisdictional licensing fields are partner-managed where the relevant regulator integration exists, otherwise self-declared.

Cross-jurisdiction matching follows the protocol-level default ([PRD §8.5](../../PRD-v0_8.md)): no cross-jurisdiction matches by default for regulated categories; opt-in requires explicit query parameters and surfaces the licensing-coverage caveat to consumers. Firm administrators do not need to configure cross-jurisdiction defaults; they apply per category at the protocol level.

## Firm-level conflict and intake routing

Firms with central intake or central conflict-checking processes can route ATAH introductions through that infrastructure rather than directly to individual professionals:

**Firm intake routing.** When configured, Stage 1 introductions to any colleague at the firm are first routed to the firm intake address. The firm decides which individual colleague will handle the matter and forwards (or, where the firm configuration supports it, the introduction proceeds to the named colleague after firm intake review). Useful for firms with central intake practices and for firms whose conflict-of-interest checks must precede any individual matter handling.

**Firm conflict checks at Stage 2.** For categories where Stage 2 is a conflict check (legal services in particular), the firm-level conflict check supplements rather than replaces the individual professional's conflict-checking obligation. The Stage 2 mechanism in the protocol provides preliminary screening; the firm's own conflict process — and the individual lawyer's professional duty — remain the controlling check.

**Coordination with individual Stage 1 preferences.** Firm intake routing composes with the individual-professional Stage 1 controls (max introductions per week, urgency levels, fee model filters, etc., described in [PRD §6.1](../../PRD-v0_8.md)). Firms generally configure firm intake to apply firm-wide policies (categories of matter not handled, jurisdictions not served) while individual Stage 1 preferences apply colleague-specific availability and capacity constraints. Both layers run; either can decline an introduction.

## "Claim request pending firm approval"

A workflow for cases where an individual professional wants to claim a record that the firm regards as firm-controlled (typically: the firm's marketing or operations have set up the colleague's record, and the colleague is now joining or claiming it):

**The flow.**

1. The colleague initiates a claim through the standard claim authentication process (verified email match against partner records, or the equivalent partner-attested identifier — per [spec §15.6](../../spec/v0.8/full-spec.md)).
2. Where the firm has set the relevant flag on the firm's records, the claim is paused pending firm approval rather than proceeding directly to the colleague.
3. The firm administrator (or designated role) reviews the claim — confirming the colleague is who they say they are, and that the firm endorses the claim.
4. On approval, the claim completes and the colleague gains portal access and ability to update self-declared fields.
5. On rejection (e.g. the colleague has left the firm and is mistakenly claiming an old record), the claim is denied and the colleague is notified with an explanation and an appeal route.

**Where this matters.** Firms with multiple departing-and-arriving professionals over time, or firms using ATAH for marketing presence in advance of individual professionals joining, both benefit from the firm-approval-before-claim workflow. Firms that prefer the simpler "colleague claims directly" model can leave the flag unset.

## Private firm-internal instances

Some firms ask whether they can run a private firm-internal instance of an ATAH-conformant registry — to handle internal referral relationships between colleagues at a large firm, internal directory functionality, or AI-tooling integrations that are firm-specific. The protocol's licensing posture covers this:

**Apache 2.0 permits forks.** The protocol is published under Apache 2.0. A firm may fork the specification and the schemas, modify them, and run a private instance for internal use. No ATAH approval is needed for the fork itself.

**Conforming-implementation status is a separate question.** A private firm-internal instance is not the same thing as operating a conforming registry. Conformance — claiming "ATAH-conformant" recognition — requires meeting the conformance criteria in [spec §14](../../spec/v0.8/full-spec.md), publishing a conformance statement at the discoverable location, and (when v0.9 ships the conformance test suite) passing the published tests. Most private firm-internal use cases do not need conformance recognition; they need internal functionality.

**Implications for firm-internal data.** Data held in a firm-internal instance is firm data, not data flowing through the ATAH ecosystem. Firms running internal instances should treat them as internal infrastructure subject to their own data protection obligations, not as ATAH-mediated data.

**Bridging firm-internal and the ATAH-operated reference registry.** If a firm wants its colleagues to be visible to AI platforms that query ATAH-conformant registries, the colleagues need records in a conforming registry — at v0.8.2, that means the ATAH-operated reference registry or another conforming registry once federation lands. Firm-internal instances do not bridge to AI-platform queries unless the firm has also published a conformance statement and is operating as a public conforming registry, which is a different undertaking.

## What is not yet supported at v0.8.2

A non-exhaustive list of v0.9 candidates for firm and team work:

**Firm/team as first-class protocol entity.** A `firm` schema with its own identifier, governance fields, contact details, branch structure, and audit history. Today, the `firm` field is a string property on each professional record. v0.9 would introduce `firm-{ULID}` identifiers, a `firm` schema, and per-firm provenance and dispute flows.

**Cross-firm permissioning.** Workflows for one professional listed at multiple firms (a barrister with chamber affiliations and consulting roles, a lawyer with both law-firm and corporate-counsel listings, etc.). At v0.8.2, the `firm` string is single-valued; multi-affiliation scenarios are handled by professional-side disclosure rather than first-class data-model support.

**Firm-level analytics.** Aggregated visibility analytics across the firm's listed professionals — query patterns the firm sees, conversion through Stage 1 / Stage 2 / Stage 3 by category, comparative ranking across the firm's specialisms. Analytics of this kind raise privacy considerations (consumer-side query patterns are sensitive even in aggregate); v0.9 work will scope what is and is not appropriate.

**Firm-level matching weight overrides.** Whether a firm can override matching weights for its colleagues (e.g. weighting the firm's senior partners higher in firm-routed introductions). Likely deliberately not supported even in v0.9 — overrides interact awkwardly with the no-commercial-weighting commitment — but the boundary will be made explicit.

**Firm-level concern flag visibility.** Whether firm administrators see concern flags raised against firm members. Currently concern flags are admin-only at the ATAH governance level (per [GOVERNANCE §7](../../GOVERNANCE.md)); whether a firm administrator should see flags against firm members is a sensitive design question that v0.9 will work through.

These items are tracked in the v0.8.3 / v0.9 candidates section of [ROADMAP.md](../../ROADMAP.md) where they are scoped.

## Reference

For the protocol-level handling of professional records and registration, see [spec §15](../../spec/v0.8/full-spec.md). For roll-up and the registration routes, see [spec §10.4](../../spec/v0.8/full-spec.md). For Stage 1 appetite-check controls and how individual preferences compose with firm intake routing, see [PRD §6.1](../../PRD-v0_8.md). For visibility classes and how they apply to firm-managed records, see [`visibility-controls.md`](visibility-controls.md). For conformance criteria a private firm-internal instance would need to meet to claim conforming-implementation status, see [spec §14](../../spec/v0.8/full-spec.md) and [CONFORMANCE.md](../../CONFORMANCE.md).
