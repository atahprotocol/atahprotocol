# "Your visibility" report — feature concept

**Status:** feature description for a planned professional-portal feature. Not yet built. Not protocol-mandated — this is a reference registry feature concept; conforming registries may implement an equivalent or not. The protocol-level mechanisms the report would draw on (matching engine rubric, verification quality components, partner taxonomy) are defined in the [specification](../../spec/v0.8/full-spec.md) and [PRD](../../PRD-v0_8.md).

**Audience:** professionals listed (or expecting to be listed) in the ATAH-operated reference registry, plus implementers considering whether to offer a similar feature in their own conforming registry.

**Companion to:** [PRD §8.5 Matching Engine](../../PRD-v0_8.md), [spec §9 Matching Engine](../../spec/v0.8/full-spec.md), [PRD §6.1 Credentialled and Established Professionals](../../PRD-v0_8.md), [docs/professional/visibility-controls.md](visibility-controls.md).

---

## Purpose

Professionals using ATAH face a question their existing marketing tools don't answer: *when an AI system queries on behalf of someone in my category, do I appear in the candidate set, and which band have I been placed in?* The matching engine is publicly documented (hard filters, band definitions, stratified-random ordering within bands) and the `_provenance` map and per-candidate `decision_explanation` carry the rules-derived reasons for inclusion, exclusion, and band placement — so the underlying signals are not secret. But assembling them into a useful self-service answer requires running representative queries and breaking down the result.

The "Your visibility" report is the planned professional-portal feature that does this. It runs representative queries against the professional's own profile, surfaces whether the professional was included or excluded, which band they were assigned to, and which rules drove that placement — so the professional can understand their AI-discoverability without manual AEO work or third-party tooling.

The report is descriptive, not prescriptive. It tells a professional how their profile is currently presented; it does not tell them what to do about it. Where band placement reflects factors the professional can affect (profile completeness, scope details, partner verification), those become useful inputs to their own decisions about updating the profile or joining a body. Where band placement reflects factors outside their control (no partner integration in their category yet, low review volume in a category where review weight is capped), the report makes that visible too — which is itself useful information.

## What the report shows

For each representative query the report runs against the professional's profile, the output shows three layers:

**The query.** The category, matter type, location, urgency, and any filters used. The report runs a small set of queries representative of the professional's profile (their declared category, the matter types and specialisms they have indicated, the jurisdictions they cover) — typically eight to twelve queries spanning the routine cases.

**The placement.** Whether the professional appeared in the candidate set, which band they were assigned to, and the candidate-set size and band composition for that query. "Top band of three, in a candidate set of fifteen across two bands" is more informative than "appeared" alone.

**The decision-explanation.** For each appearance, the rules-derived reasons supporting band assignment: which hard filters were applied, which band-input signals (verification confidence, category fit, availability, contact freshness) drove the placement, and which rules — if any — could have placed the professional in a higher or lower band. The decision-explanation is the same machine-readable artefact AI platforms see on every match response, presented to the professional for their own profile.

The decision-explanation also surfaces the `verification_confidence` signal (single-source vs multi-source), the partner-class summary for the professional's verifications, and any active enhanced verification records. These are the signals the AI platform sees on every match response — the report shows the professional what their own match response looks like.

## What the report does not show

A short list of deliberate omissions:

**Specific competing professionals.** The report is about the professional's own positioning, not about other professionals. Where the professional shared a band with other candidates, the report does not name them. AI-discoverability transparency is for the professional reading their own report; competitive intelligence on other professionals is not the purpose.

**Aggregate query volume against the professional's profile.** The report shows representative queries, not actual queries the registry has handled. Real query patterns would expose information about consumers and AI platform integrations that doesn't belong in a per-professional report. **This is codified as a normative MUST NOT in spec §11A.4:** professional-facing visibility explanations MUST NOT expose actual query-count data, query-history data, observed demand patterns, or any per-query information derived from consumer or platform traffic; they MUST present representative explanation categories at the category/jurisdiction level, derived from the implementation's documented rules and the professional's own profile data. The professional-facing visibility view delivered by the `GET /v1/professionals/me/visibility-explanations` endpoint (whether implemented in v0.8.2 or as a v0.8.3 fast-follow per ROADMAP.md) is rules-derived for the same reason — even aggregated query counts can leak in low-volume categories.

**Match decisions made by AI platforms.** Once a match response leaves ATAH, the AI platform applies its own contextual layer — re-ordering, filtering, presenting only some of the candidate set. The report shows what ATAH returned, not what the AI platform did with the response. Platform-side decisions are platform business.

**Predictive estimates of future introductions.** The report does not predict how many introductions the professional will receive. Introduction volume depends on actual query traffic, which depends on AI platform integrations advancing, which is outside what a per-professional report can responsibly forecast.

## Decision-explanation breakdown — what each band-input signal means in plain language

For professionals who haven't read the matching engine specification (most of you), here is what each band-input signal represents:

**Verification confidence.** How well the professional's verification base is corroborated. Single-source verification (only one partner confirms credentials) sits in a different band from multi-source corroboration (several partners agree). Mostly outside the professional's control; reflects which authoritative sources cover the category and whether the relevant partner agreement is in place. Enhanced verification — commissioned by the professional, an employer, or a professional body — adds an additional confidence layer. The partner taxonomy's `vetting_strength` field is what makes regulatory verification distinct from open-membership-body verification.

**Category fit.** How well the professional's declared scope (category, matter types, specialisms, jurisdictions, languages) fits the query. Improvable by making your declared scope more accurate — too narrow misses queries you could handle; too broad weakens fit on queries you actually fit. Only verified scope (specialisms confirmed by the relevant body, lines of authority for insurance, etc.) materially contributes.

**Availability.** Whether the professional is currently accepting introductions, the declared response-time commitment, and timezone alignment with the query. Improvable by keeping availability current and setting realistic response-time commitments.

**Contact freshness.** Whether the professional's notification channels have been verified within the per-category cadence (quarterly for high-stakes regulated categories; annual for established practitioners). Stale channels flip `matching_status` to `contact_unverified` and remove the professional from candidate sets until re-verified. Improvable directly: respond to verification challenges promptly and keep contact details current.

**Review platform signals — where present.** Supplement the band-input picture but do not move a candidate into a higher band in regulated categories unless corroborated by an authoritative source like a regulator or professional body. Capped per category — for high-stakes regulated categories, the review-derived contribution is bounded by the category-specific `review_signal_band_cap` regardless of review volume. Reviews complement but do not substitute for credential verification.

## Frequency and access

The planned design:

**On demand through the professional portal.** Run the report whenever you want to. The representative queries are deterministic given your profile; running the report twice in a row gives the same result unless something on your profile or in the partner data has changed.

**Notified updates.** Where a material change happens to your contribution breakdown — a new partner integration covering your category goes live, an enhanced verification record activates, a roll-up merge completes — the portal surfaces a notification with a pointer to re-run the report.

**Historical comparison.** Optionally, the portal can show how your visibility has evolved over time. This is opt-in (some professionals will not want a longitudinal record) and is stored against the professional's record, not aggregated across professionals.

## Data privacy

The report uses the professional's own data and the public matching rubric. Other professionals' specific data is not exposed. The aggregate category data the report references ("the typical fifteen-candidate pool size for this query") is statistical and category-level, not per-professional.

The report runs server-side; representative queries are not actual queries from real consumers. Running the report does not consume registry resources beyond the report computation itself, and does not affect or appear in any consumer's match response.

## Where this fits

The "Your visibility" report sits alongside the visibility controls described in [`visibility-controls.md`](visibility-controls.md). Visibility controls answer "am I visible, and to whom?" The report answers "given that I am visible, what does my visibility actually look like?"

For implementers building a conforming registry, an equivalent feature is encouraged but not protocol-mandated. The matching engine pipeline (hard filters → band assignment → stratified-random ordering within bands) is public, the per-candidate `decision_explanation` is in every match response, and the `_provenance` map carries field-level verification status — so a conforming registry has all the inputs needed to build an equivalent report.

## Status

**At v0.8.2 publication:** feature concept. The matching engine pipeline and per-candidate `decision_explanation` are specified and present in match responses. The professional-portal feature itself is reference-registry build work that proceeds alongside partner conversations and implementation work (per [PRD §14 Phase 2](../../PRD-v0_8.md)).

**When the feature ships in the reference registry:** this document will be expanded to cover the actual portal surface, the specific representative-query set per category, and any operational considerations (rate limiting on report generation, retention of historical reports, professional consent for storing the longitudinal series).

## Reference

For the matching engine pipeline (hard filters, band definitions, stratified-random ordering within bands), see [PRD §8.5](../../PRD-v0_8.md) and [spec §9](../../spec/v0.8/full-spec.md). For the per-candidate `decision_explanation` shape, see [spec §11A](../../spec/v0.8/full-spec.md). For the `_provenance` map and verification status enum, see [spec §4.2 and §4.3](../../spec/v0.8/full-spec.md). For the partner taxonomy that determines how `vetting_strength` qualifies credential verification, see [PRD §6.3](../../PRD-v0_8.md). For visibility classes and how they interact with whether the report is meaningful, see [`visibility-controls.md`](visibility-controls.md).
