# 12. Stratified randomisation, no global match_score, review-signal band cap

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

This decision draws on Paolo Piponi's peer review (May 2026). v0.8.1's matching engine was a weighted-score architecture: a `match_score` per candidate computed from a `match_factors` object (relevance, verification quality, availability, profile completeness, inbound referral signal), with `presentation_disclosure.ranking_basis` listing the components, and per-category weights tunable via `matching_weight_profile`. The v0.8.1 spec also declared that ATAH was not a recommendation engine.

The reviewer (F1.10, F1.11, F1.12, F1.13) identified that the two positions were in tension. Even if ATAH says it is not a recommendation engine, ordering still matters: users and AI systems may interpret the first returned candidate as the best candidate. If ATAH applies hidden ranking — even via a published score and weights — the "not a recommendation" position is weakened. The reviewer's framing: "A hidden global score plus randomised display would look cosmetic and undermine the claim that ATAH is not ranking by preference." Pure randomisation is not sufficient either — it can produce poor outcomes and can still place a weak candidate first by chance.

Three further problems compounded:

1. **Inbound referral signal as a recommendation hook.** The `inbound_referral_signal` component required reciprocal-cap, time-decay, dense-cluster, and Sybil-detection controls. The fact that those controls were necessary indicates the signal was a meaningful gaming vector. v0.8.2 Phase 2 had already removed the signal per `GKC-COMMENTS-01`; Phase 5 makes the architectural point that no single signal of this type belongs in matching.

2. **Review-signal cap as a band-incompatible safeguard.** v0.8.1's `review_signal_weight_cap` capped the contribution of review-derived signals to the final score in regulated categories (≤0.10). The cap was the original safeguard against review-platform dominance in high-stakes contexts. Paolo's review (F-7 in the v3 review pass) flagged that simply deleting the field, as v2's plan proposed, would create a back door — a non-conforming registry could let review signals dominate band assignment in regulated categories with no protocol-level prohibition. The cap needs to be retained, but in a band-compatible form.

3. **AI-platform reordering obligation.** ATAH controls only the returned list. The consuming AI platform may reorder based on user context, but the disclosure that ATAH applied non-preferential ordering needs to survive that reordering. v0.8.1 had no explicit MUST on this; v0.8.2 needs one (F1.13).

## Decision

v0.8.2 replaces the weighted-score architecture in full with hard-filters-then-stratified-randomisation, with verbatim normative wording adopted from Paolo (per the v0.8.2 standing instruction):

**Four-step matching pipeline (§9.1).**

1. Hard filters (category, jurisdiction, matching_status, compliance_status, contact freshness, availability, category-required verification thresholds).
2. Band assignment (eligible candidates grouped into transparent bands per `profession-categories.json` `band_definitions`; default bands: verification confidence, category fit, availability window, contact freshness).
3. Threshold exclusion (where the category requires a minimum band position).
4. Randomisation within bands (lower band position first; within band, `uniform_random`, `round_robin_rotation`, or a published `documented_implementation_policy`). Randomisation seed is `not_disclosed_to_prevent_gaming`.

Verbatim normative rule:

> ATAH may determine eligibility and exclusion, but MUST NOT express preference among eligible candidates unless the ordering basis is explicit, non-commercial, and disclosed. Where no user-requested ordering criterion is supplied, candidate order MUST be randomised or rotated using a documented fairness policy.

**No global match_score.** `match-response.schema.json` removes in full: `match_score` (per-result), `match_factors` (per-result object and all sub-properties), `presentation_disclosure.ranking_basis` (array enum), every reference to `inbound_referral_signal`. Replaced with `filters_passed`, `band_assignment` (with bands list and `position_in_response`), per-candidate `ordering_policy` (`mode` + `within_band_policy`), and response-level `presentation_disclosure.ordering_policy` (the canonical AI-platform-facing disclosure). The "no recommendation engine" commitment is structural, not just declarative — observable through schema absence of `match_score` and through conformance-test verification of within-band non-determinism.

**User-requested ordering (§9.3).** `query.schema.json` adds optional `ordering_preference` parameter with eight modes (`nearest`, `soonest_available`, `remote_available`, `specific_language`, `highest_verification_confidence`, `specific_category_qualification`, `professional_type`, `jurisdiction`). When omitted, default is `stratified_random`. When supplied, ATAH applies the named mode and records it on the response. The user-requested mode does not override hard filters or threshold exclusion; it changes ordering only.

**Review-signal band cap (§9.2, F-7).** `profession-category.schema.json` replaces `review_signal_weight_cap` with `review_signal_band_cap`, preserving the original safeguard under the band-based model:

> Review-derived signals MAY supplement transparency and confidence metadata, but for high-stakes regulated categories they MUST NOT move a candidate into a higher eligibility or verification band unless corroborated by authoritative credential or regulator-source evidence.

Field shape: `max_band_influence` (`supplemental_only` / `not_applicable`); `may_upgrade_band` (boolean); `regulated_category_max_effect` (`no_band_upgrade` / `documented_corroboration_required` / `not_applicable`). High-stakes regulated categories (credentialled tier with mandatory body membership) set `may_upgrade_band: false`. Lower-stakes categories may permit `may_upgrade_band: true` with documented authoritative-corroboration requirements.

**Schema and data atomicity (F-13).** `profession-category.schema.json` and `profession-categories.json` are updated atomically. The schema removes `matching_weight_profile` and `review_signal_weight_cap` from properties and required, and adds `band_definitions` and `review_signal_band_cap`. All 15 launch categories in the data file are rewritten with the new shape. The data file validates against the schema; non-atomic updates would fail validation for the duration of any gap.

**AI platform presentation obligation (§9.4, F1.13).**

> AI platforms reordering the ATAH response MUST preserve the `presentation_disclosure.ordering_policy` field verbatim and surface its content to the user. AI platforms MUST NOT present ATAH's response as a recommendation or rank by ATAH's preference.

Failure to preserve the disclosure is a binding-conformance failure (Section 14). Charter Part Two operational commitments mirror the §9 normative rules.

**Implementation algorithm deferral (§9.5).** v0.8.2 specifies the four-step model and the within-band fairness policy enumeration but does not mandate a specific randomisation algorithm. That is implementation choice. Conformance verifies that returned `band_assignment` data is consistent with `band_definitions` and that within-band ordering is observably non-deterministic across repeated queries. v0.9 may standardise the algorithm.

## Alternatives considered

**Pure randomisation without bands.** Considered: drop weighting and bands altogether; randomise eligible candidates uniformly. Rejected as insufficient quality control — random ordering can place a single-source-self-declared candidate above a multi-source-corroborated candidate by chance, undermining the verification-quality posture even when ATAH has the data to do better. Stratified randomisation (random within bands of equivalent quality, but with bands ordered by quality) keeps the quality posture without expressing preference within the band.

**Retain weighted scoring with caps.** Considered: keep `match_score` but tighten caps and disclosures. Rejected per F1.10 sharper position — a hidden global score plus randomised display would look cosmetic and undermine the non-recommendation claim. The cap-and-disclosure approach kicks the can; structural removal is the honest position.

**Delete `review_signal_weight_cap` entirely.** Considered (v2 plan): remove the cap as part of the broader removal of `matching_weight_profile`. Rejected per F-7 — deleting the cap creates a back-door for review platforms to influence high-stakes regulated categories with no protocol-level prohibition. The band-compatible replacement (`review_signal_band_cap`) preserves the original safeguard.

**Standardise the randomisation algorithm in v0.8.2.** Considered: mandate `uniform_random` (or another specific algorithm). Rejected — implementation contexts differ (some implementations may want round-robin rotation for fairness across queries; some may have category-specific reasons to use a documented per-implementation policy). v0.8.2 specifies the model and the enumeration of permitted policies; v0.9 may standardise.

**Disclose the randomisation seed.** Considered: include the seed on the response for reproducibility / auditability. Rejected — disclosing the seed creates a gaming surface (an attacker who can predict the seed can predict ordering). The conformance test verifies non-determinism without requiring seed disclosure.

## Consequences

- v0.8.1 partner integration documentation referencing `match_score` directly is now broken. Phase 12 (documentation cascade) catches this; Phase 5 has done a preliminary grep and found no in-spec references outside the retiring narrative sections.
- The "no hidden global score" claim is structurally enforced only if the implementation actually uses the bands-and-randomisation model. A non-conforming implementation could still apply a hidden score internally. The conformance test in Phase 6 / Phase 11 needs to verify that returned `band_assignment` data is consistent with the documented `band_definitions` and that within-band ordering is observably non-deterministic.
- The `review_signal_band_cap` is a new field shape. Future revisions of the field need coordinated updates across the schema, the data file, and the matching engine implementation.
- The AI platform presentation obligation introduces a binding-conformance requirement. Implementations that strip or alter `presentation_disclosure.ordering_policy` before surfacing to the user are non-conformant; the conformance test verifies the disclosure survives the ATAH-to-platform handoff.
- The match-response schema's `decision_explanation` field is left as an open object placeholder in Phase 5; Phase 6 introduces `decision-explanation.schema.json` and replaces the placeholder with a `$ref`. Phase 5 must not block Phase 6 by hardcoding response shapes.
- Per-category band definitions for v0.8.2 use a templated tier-based default (credentialled vs established). Researched per-category refinements are a v0.9 task that requires domain expertise.

## References

- Spec §4.12 (Match Response Schema — example end-to-end refresh), §9 (full refactor: §9.1 four-step pipeline with verbatim MUST NOT; §9.2 review-signal band cap with F-7 verbatim MUST; §9.3 user-requested ordering modes; §9.4 AI platform presentation obligation; §9.5 algorithm deferral; §9.6 commercial neutrality).
- Schemas: [`match-response.schema.json`](../../spec/v0.8/schemas/match-response.schema.json) (full rewrite — `OrderingPolicy` and `BandAssignment` `$defs`, weighted-score fields removed, `decision_explanation` placeholder added); [`profession-category.schema.json`](../../spec/v0.8/schemas/profession-category.schema.json) (`band_definitions` and `review_signal_band_cap` added, old fields removed); [`query.schema.json`](../../spec/v0.8/schemas/query.schema.json) (`ordering_preference` added).
- Data: `profession-categories.json` — all 15 launch categories rewritten with band definitions and review-signal band cap.
- OpenAPI: `MatchResponseExample` updated end-to-end; `POST /v1/query` accepts `ordering_preference` via the schema.
- MCP tools: `find_professional` accepts `ordering_preference` via the schema.
- Charter Part Two operational commitments mirror the §9 normative rules.
- Master patch plan v3.1.1, Phase 5 (§7) and Paolo's findings F1.10 (stratified randomisation), F1.11 (no global match_score), F1.12 (user-requested ordering), F1.13 (presentation obligation); F-7 (review-signal band cap, not deletion); F-13 (atomic schema + data update); F-14 (full match-response cleanup).
- Source review: `PAOLO-PEER-REVIEW-FINDINGS-01-v0_8_1.md` and `PAOLO-ordering-and-randomisation.md`.
