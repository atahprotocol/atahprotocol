# 12. Stratified randomisation, no global match_score, review-signal band cap

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

This decision draws on Paolo Piponi's peer review (May 2026). ATAH says it is not a recommendation engine, but ordering still matters: users and AI systems may interpret the first returned candidate as the best candidate. If ATAH applied hidden ranking — even via a published score and weights — the "not a recommendation" position would be weakened. The reviewer's framing (F1.10, F1.11, F1.12, F1.13): "A hidden global score plus randomised display would look cosmetic and undermine the claim that ATAH is not ranking by preference." Pure randomisation is not sufficient either — it can produce poor outcomes and can still place a weak candidate first by chance.

Two further considerations compounded:

1. **Review-signal cap as a band-compatible safeguard.** Review-derived signals can dominate matching in regulated categories if not constrained. The original safeguard against review-platform dominance is a per-category cap. Paolo's review (F-7) flagged that the cap needs to be expressed in a band-compatible form — without an explicit cap, a non-conforming registry could let review signals upgrade band assignment in regulated categories with no protocol-level prohibition.

2. **AI-platform reordering obligation.** ATAH controls only the returned list. The consuming AI platform may reorder based on user context, but the disclosure that ATAH applied non-preferential ordering needs to survive that reordering — an explicit MUST is required (F1.13).

## Decision

ATAH adopts a hard-filters-then-stratified-randomisation matching pipeline, with verbatim normative wording adopted from Paolo's review:

**Four-step matching pipeline (§9.1).**

1. Hard filters (category, jurisdiction, matching_status, compliance_status, contact freshness, availability, category-required verification thresholds).
2. Band assignment (eligible candidates grouped into transparent bands per `profession-categories.json` `band_definitions`; default bands: verification confidence, category fit, availability window, contact freshness).
3. Threshold exclusion (where the category requires a minimum band position).
4. Randomisation within bands (lower band position first; within band, `uniform_random`, `round_robin_rotation`, or a published `documented_implementation_policy`). Randomisation seed is `not_disclosed_to_prevent_gaming`.

Verbatim normative rule:

> ATAH may determine eligibility and exclusion, but MUST NOT express preference among eligible candidates unless the ordering basis is explicit, non-commercial, and disclosed. Where no user-requested ordering criterion is supplied, candidate order MUST be randomised or rotated using a documented fairness policy.

**No global match_score.** `match-response.schema.json` carries `filters_passed`, `band_assignment` (with bands list and `position_in_response`), per-candidate `ordering_policy` (`mode` + `within_band_policy`), and response-level `presentation_disclosure.ordering_policy` (the canonical AI-platform-facing disclosure). There is no global score field. The "no recommendation engine" commitment is structural, not just declarative — observable through schema absence of any global score and through conformance-test verification of within-band non-determinism.

**User-requested ordering (§9.3).** `query.schema.json` adds optional `ordering_preference` parameter with eight modes (`nearest`, `soonest_available`, `remote_available`, `specific_language`, `highest_verification_confidence`, `specific_category_qualification`, `professional_type`, `jurisdiction`). When omitted, default is `stratified_random`. When supplied, ATAH applies the named mode and records it on the response. The user-requested mode does not override hard filters or threshold exclusion; it changes ordering only.

**Review-signal band cap (§9.2, F-7).** `profession-category.schema.json` carries `review_signal_band_cap`, preserving the safeguard under the band-based model:

> Review-derived signals MAY supplement transparency and confidence metadata, but for high-stakes regulated categories they MUST NOT move a candidate into a higher eligibility or verification band unless corroborated by authoritative credential or regulator-source evidence.

Field shape: `max_band_influence` (`supplemental_only` / `not_applicable`); `may_upgrade_band` (boolean); `regulated_category_max_effect` (`no_band_upgrade` / `documented_corroboration_required` / `not_applicable`). High-stakes regulated categories (credentialled tier with mandatory body membership) set `may_upgrade_band: false`. Lower-stakes categories may permit `may_upgrade_band: true` with documented authoritative-corroboration requirements.

**Schema and data atomicity (F-13).** `profession-category.schema.json` and `profession-categories.json` move together: the schema declares `band_definitions` and `review_signal_band_cap` as the canonical fields, and all 15 launch categories in the data file carry the corresponding shape. The data file validates against the schema; any future change moves both surfaces atomically.

**AI platform presentation obligation (§9.4, F1.13).**

> AI platforms reordering the ATAH response MUST preserve the `presentation_disclosure.ordering_policy` field verbatim and surface its content to the user. AI platforms MUST NOT present ATAH's response as a recommendation or rank by ATAH's preference.

Failure to preserve the disclosure is a binding-conformance failure (Section 14). Charter Part Two operational commitments mirror the §9 normative rules.

**Implementation algorithm deferral (§9.5).** The protocol specifies the four-step model and the within-band fairness policy enumeration but does not mandate a specific randomisation algorithm. That is implementation choice. Conformance verifies that returned `band_assignment` data is consistent with `band_definitions` and that within-band ordering is observably non-deterministic across repeated queries. v0.9 may standardise the algorithm.

## Alternatives considered

**Pure randomisation without bands.** Considered: drop weighting and bands altogether; randomise eligible candidates uniformly. Rejected as insufficient quality control — random ordering can place a single-source-self-declared candidate above a multi-source-corroborated candidate by chance, undermining the verification-quality posture even when ATAH has the data to do better. Stratified randomisation (random within bands of equivalent quality, but with bands ordered by quality) keeps the quality posture without expressing preference within the band.

**Weighted scoring with caps and disclosures.** Considered: a global score with tightened caps and explicit disclosures. Rejected per F1.10 — a hidden global score plus randomised display would look cosmetic and undermine the non-recommendation claim. The cap-and-disclosure approach kicks the can; structural absence of a global score is the honest position.

**No protocol-level review-signal cap.** Considered: leave review-signal influence to per-implementation policy. Rejected per F-7 — without a protocol-level cap, a non-conforming registry could let review signals upgrade band assignment in regulated categories with no protocol prohibition. The `review_signal_band_cap` field codifies the safeguard.

**Standardise the randomisation algorithm now.** Considered: mandate `uniform_random` (or another specific algorithm) at the protocol layer. Rejected — implementation contexts differ (some implementations may want round-robin rotation for fairness across queries; some may have category-specific reasons to use a documented per-implementation policy). The protocol specifies the model and the enumeration of permitted policies; v0.9 may standardise.

**Disclose the randomisation seed.** Considered: include the seed on the response for reproducibility / auditability. Rejected — disclosing the seed creates a gaming surface (an attacker who can predict the seed can predict ordering). The conformance test verifies non-determinism without requiring seed disclosure.

## Consequences

- The "no hidden global score" claim is structurally enforced only if the implementation actually uses the bands-and-randomisation model. A non-conforming implementation could still apply a hidden score internally. The conformance test verifies that returned `band_assignment` data is consistent with the documented `band_definitions` and that within-band ordering is observably non-deterministic.
- The `review_signal_band_cap` field shape requires coordinated updates across the schema, the data file, and the matching engine implementation when revised.
- The AI platform presentation obligation is a binding-conformance requirement. Implementations that strip or alter `presentation_disclosure.ordering_policy` before surfacing to the user are non-conformant; the conformance test verifies the disclosure survives the ATAH-to-platform handoff.
- The match-response schema references `decision-explanation.schema.json` for response-level and per-candidate explanations; the per-candidate `decision_explanation` field carries the rules-derived reasons supporting band assignment.
- Per-category band definitions use a templated tier-based default (credentialled vs established). Researched per-category refinements are a v0.9 task that requires domain expertise.

## References

- Spec §4.12 (Match Response Schema), §9 (§9.1 four-step pipeline with verbatim MUST NOT; §9.2 review-signal band cap with F-7 verbatim MUST; §9.3 user-requested ordering modes; §9.4 AI platform presentation obligation; §9.5 algorithm deferral; §9.6 commercial neutrality).
- Schemas: [`match-response.schema.json`](../../spec/v0.8/schemas/match-response.schema.json) (`OrderingPolicy` and `BandAssignment` `$defs`); [`profession-category.schema.json`](../../spec/v0.8/schemas/profession-category.schema.json) (`band_definitions` and `review_signal_band_cap`); [`query.schema.json`](../../spec/v0.8/schemas/query.schema.json) (`ordering_preference`).
- Data: `profession-categories.json` — 15 launch categories with band definitions and review-signal band cap.
- OpenAPI: `MatchResponseExample`; `POST /v1/query` accepts `ordering_preference` via the schema.
- MCP tools: `find_professional` accepts `ordering_preference` via the schema.
- Charter Part Two operational commitments mirror the §9 normative rules.
- Findings: F1.10 (stratified randomisation), F1.11 (no global match_score), F1.12 (user-requested ordering), F1.13 (presentation obligation); F-7 (review-signal band cap); F-13 (atomic schema + data); F-14 (match-response shape).
