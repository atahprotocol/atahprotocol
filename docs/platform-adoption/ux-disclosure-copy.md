# UX disclosure copy library

**Status:** model strings AI platforms can adapt for their own UI. Not normative — the protocol mandates that the `presentation_disclosure` block is surfaced; it does not mandate the exact wording. This document supplies model phrasings calibrated against the ATAH non-recommendation framing and the platform presentation obligations in [PRD §11](../../PRD-v0_8.md).

**Audience:** AI platform UX teams writing the copy that wraps ATAH match results in user-facing surfaces.

**Companion to:** [PRD §11 Platform presentation obligations](../../PRD-v0_8.md), [spec §4.12 Match Response Schema](../../spec/v0.8/full-spec.md), [`presentation_disclosure` block in `match-response.schema.json`](../../spec/v0.8/schemas/match-response.schema.json).

---

## How to use this document

The `presentation_disclosure` block on every match response carries four required fields:

- `atah_is_not_recommending` — boolean, always `true`
- `ordering_policy` — object declaring the ordering mode (`stratified_random` when no explicit ordering preference is supplied; named modes such as `nearest`, `soonest_available`, `language`, `verification_confidence` when one is) and the hard filters applied
- `commercial_weighting` — boolean, always `false`
- `user_facing_disclosure` — string the platform should surface

Platforms can use the `user_facing_disclosure` string verbatim, adapt it to their UI voice, or substitute their own copy — provided the substitute preserves the non-recommendation framing and meets the platform presentation obligations. The model strings below are calibrated examples for each common surface.

The constraint is the framing, not the wording. ATAH does not recommend; results are candidate options based on declared need, verification evidence, category rules, and availability; the user remains responsible for selection. Any platform copy that preserves that meaning is acceptable.

## Core disclosures

Six common surfaces, each with a model string and a brief note on when to use it.

### 1. Verified options, not recommendations

**Use when:** the user is first shown the ATAH-sourced shortlist, before they pick a professional.

**Model string:**
> These are verified options based on what you've described, not recommendations. Each has been verified against the source listed alongside their name. You choose whether to proceed.

**Variants for terser surfaces:**
> Verified options for what you've described. Selection is yours.

> Candidate professionals matching your needs. We've shown the verification basis for each.

### 2. Professional appears licensed because

**Use when:** the user clicks through to a specific professional's profile or hovers on a verification badge.

**Model string:**
> [Professional name] appears licensed because [verification source] confirmed their licence on [verification date]. This verification is single-source [or: corroborated by N sources]. Verifications are refreshed within [freshness window] of any change.

**Notes:**
- Substitute the actual `verification_status_summary` source name from the match response.
- Use the `verification_confidence` signal (per spec §4.12, schema lands v0.8.2) to populate single-source vs multi-source. Until the schema lands, infer from the `_provenance` map.
- Freshness window is category-specific; surface the actual window from category metadata.

### 3. Self-declared fields included

**Use when:** the profile shown contains fields labelled `self-declared` in the `_provenance` map (specialism details, fee model, response-time commitment, languages, etc.).

**Model string:**
> Some details on this profile are self-declared by the professional and not independently verified — including [list the self-declared fields]. Verified facts (licence, qualifications, professional body membership) are labelled separately.

**Notes:**
- Pull the field list from the `_provenance` map; do not invent or summarise.
- Keep the verified/self-declared distinction explicit. Self-declared is honest, not weak — but it's different from verified.

### 4. Result suppressed because verification data conflicts

**Use when:** the user has asked about a specific professional or expected to see one in the results, and that professional's record is currently in conflict-suppression state (`pending-rollup`, `pending-integrity-review`, or similar).

**Model string:**
> [Professional name]'s record is currently held while a data conflict is being resolved. They aren't being shown in match results until the conflict is resolved. This protects you from acting on data that may be inaccurate. The professional has been notified and the dispute has a published resolution timeline.

**Notes:**
- Don't expose the nature of the conflict at this surface — the conflict resolution flow is admin-mediated and partner-mediated.
- Don't speculate ("this might be a problem with the professional"). Suppression is data-quality management, not a finding against the professional.

### 5. ATAH does not warrant outcome

**Use when:** the user is about to initiate the introduction (Stage 1), or about to release contact details (Stage 3).

**Model string:**
> ATAH supplies verification evidence, freshness, and conflict checks for the professionals shown. ATAH does not warrant that any specific professional is right for your matter, and does not guarantee outcomes. Suitability is your decision (and your AI's, in supporting you to decide).

**Variant for shorter surfaces:**
> Verification is checked. Suitability is your call.

### 6. You choose whether to proceed

**Use when:** the user has reviewed a professional's profile and the platform is presenting the proceed/decline action.

**Model string:**
> You can proceed to introduction with [professional name], or decline and pick someone else. Proceeding means sharing the small set of details listed below — your AI will keep your data minimised and ATAH will release it only for this introduction.

**Notes:**
- The "small set of details listed below" refers to the consent-receipt scope. Show the actual `data_categories` from the consent receipt being submitted.
- Make decline as visible as proceed. Privacy-first means revoke must be as easy as submit (per EXPLAINER consent receipts section).

## Worked examples by category

How the disclosures cohere when assembled for a specific category. The examples assume a v0.8.1 launch state with single-source partner verification (typical at protocol launch as the partner network builds — see PRD §8.5 verification confidence signal).

### Legal services (lawyer matching)

**Initial shortlist surface:**
> Three lawyers match what you've described — verified options, not recommendations. Each has been verified against the source listed. You choose whether to proceed.

**Profile click-through:**
> Sarah Chen appears licensed because the [State Bar of representative example] confirmed her active licence on 28 March 2026. This verification is single-source. Verifications are refreshed within seven days of any change. Some details on this profile are self-declared by the professional and not independently verified — including specialism in family law, response time commitment, and language fluency. Verified facts (licence, jurisdiction, disciplinary history) are labelled separately.

**Pre-introduction surface:**
> ATAH supplies verification evidence, freshness, and conflict checks. ATAH does not warrant that Sarah Chen is right for your matter, and does not guarantee outcomes. Suitability is your decision. Stage 2 will run a preliminary conflict check; that does not replace Sarah's own conflict-checking obligations before engagement.

### Property and casualty insurance

**Initial shortlist surface:**
> Two insurance agents match the coverage you've described — verified options, not recommendations. Each has been verified against the source listed. You choose whether to proceed.

**Profile click-through:**
> [Agent name] is currently licensed in [jurisdiction] for property and casualty lines per [licensing data source — illustrative example] on 12 April 2026. This verification is single-source. Lines of authority and carrier appointments are partner-verified; specific specialisms (farm, commercial, personal) are self-declared by the professional.

### Financial advice

**Initial shortlist surface:**
> Two advisers match what you've described — verified options, not recommendations. Each has been verified against the source listed. You choose whether to proceed.

**Profile click-through:**
> [Adviser name] is currently registered with [authoritative US register — illustrative example] per the most recent verification on [date]. This verification is single-source. Disclosed disciplinary history is included where the source surfaces it. Some details (advice scope, fee model, asset minimums) are self-declared by the adviser.

## Anti-patterns

Language platforms must not use. Drawn from the platform presentation obligations (PRD §11) and the non-recommendation framing throughout the package.

### Words ATAH does not endorse

| Don't say | Why |
|---|---|
| "Recommended by ATAH" | ATAH does not recommend. Match responses are candidate options against query parameters and disclosed verification signals. |
| "Approved by ATAH" | ATAH does not approve professionals. Approval is the relevant regulator's role; ATAH surfaces what the regulator has already approved. |
| "Best for [matter]" | ATAH does not rank candidates or produce a global match score. Within hard-filtered candidate sets, ATAH presents results in a stratified-random order within transparent bands reflecting verification confidence, category fit, availability, and contact freshness. There is no "best" for ATAH to surface; platforms must surface this ordering policy to consumers in their presentation of results. |
| "Endorsed by ATAH" | ATAH does not endorse. Provenance is visible per field; the user's AI applies context; the user makes the call. |
| "Vouched for by ATAH" | ATAH does not vouch for individual professionals. ATAH attests to verification evidence, source, and freshness — not to any individual's competence or character. |
| "Top-rated" / "Premier" / "Elite" | These imply ranking ATAH does not perform. Review-derived signals are capped per category and never blended into a single trust score. |

### Framings that misread ATAH

**Don't:** present ATAH match results as a directory the user can browse. ATAH is not consumer-facing; the user is reaching it through their AI. The shortlist is a response to the matter described, not a list to scroll.

**Don't:** describe ATAH as a regulator or as conferring regulatory status. Regulators are partners (where relevant); ATAH surfaces what they know.

**Don't:** treat review-derived signals as equivalent to credential verification. Review platforms are signal providers, not authority providers; review-derived signals complement but do not substitute for credential verification (per PRD §6.3 partner taxonomy).

**Don't:** treat single-source verification as a weakness. Single-source is common at protocol launch, reflects partner-network maturity, and is honestly labelled. The `verification_confidence` signal is informational, not a quality judgement.

### Presentation hygiene

**Don't:** strip the `presentation_disclosure` block from the user-facing surface. The conformance requirement is that platforms surface its content; visual treatment is the platform's choice but the content must be present.

**Don't:** soften "ATAH does not warrant outcomes" into "ATAH may not warrant outcomes" or similar hedging. The non-warranty stance is structural, not contingent.

**Don't:** invent verification sources that ATAH didn't return. If the `_provenance` map says the licence is `partner-verified` by partner X, the surface should name partner X — not "verified by leading legal authorities" or any composite phrasing.

## Reference

For the protocol-level definition of `presentation_disclosure`, see [spec §4.12 Match Response Schema](../../spec/v0.8/full-spec.md). For the platform presentation obligations, see [PRD §11](../../PRD-v0_8.md). For the partner taxonomy that distinguishes authoritative regulatory sources from signal providers, see [PRD §6.3](../../PRD-v0_8.md). For consent receipts and the data-categories list a platform must surface at proceed-or-decline points, see [spec §4.10](../../spec/v0.8/full-spec.md).
