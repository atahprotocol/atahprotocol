# Changelog

All notable changes to the ATAH protocol specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Breaking changes are possible until v1.0.

A detailed file-by-file change log between v0.7 and v0.8 is maintained separately in [CHANGES-FROM-V0_7-TO-V0_8.md](CHANGES-FROM-V0_7-TO-V0_8.md).

---

## [v0.8.2] — May 2026 (in progress)

**Status:** Patch round in progress. v0.8.1 is not separately published; v0.8.2 publishes directly to GitHub when the patch round completes review.

### Summary

v0.8.2 absorbs three rounds of post-v0.8.1 review work: GKC-COMMENTS-01 (architectural simplification to a three-component model), Paolo's peer review (formal principal/delegation model, three-concept separation of erasure / audit / withdrawal, stratified-randomisation-based ordering, transparency-as-conformance, contact-detail freshness), and three rounds of AI peer review (technical contract fixes, residual cleanup, strategic-risk positioning). The protocol's purpose, actors, and lifecycle direction are unchanged; v0.8.2 sharpens trust boundaries, retires the v0.8.1 "Type 1 / Type 2 / Type 3" framing in favour of a three-component model, and commits to a stratified ordering posture in place of weighted scoring.

### Changed

- **Three-component architecture (GKC-COMMENTS-01).** Type 1 / Type 2 / Type 3 terminology is retired entirely; v0.8.2 uses Component 1 (Discovery), Component 2 (Consumer-self handoff with three flow variants — `off_protocol`, `contact_share`, `full_lifecycle`), Component 3 (Referral connection-making). No historical aliases retained — the rename is comprehensive across spec, schemas, OpenAPI, MCP tools, and documentation. The professional-on-behalf-of-client case formerly modelled as "Type 3" is served by Discovery alone (Component 1 with `request_intent: 'on_behalf_of_client'`); ATAH does not deliver client contact details based on professional attestation. The associated `professional_attestation_of_client_consent` consent path is removed.
- **Schema renames and additions.** `handoff-type1.schema.json` → `handoff-component2.schema.json` (with new required `flow_variant` field). `handoff-type2.schema.json` → `referral-proposal.schema.json` (rewritten for the persistent-then-lapsed lifecycle; per F-4, `consent_receipt_id` is NOT carried — Component 3 actions are governed by professional delegated authority recorded via `principal-delegation.schema.json`). `handoff-type3.schema.json` deleted. New: `referral-connection.schema.json` for the de-duplication record. New: `principal-delegation.schema.json` defining the shared `AuthenticatedActor` / `ClientApplication` / `AuthorityContext` shape used wherever actor or asserter information is recorded.
- **Discovery query.** `query.schema.json` adds required `request_intent` (`self`, `on_behalf_of_client`, `referral_partner_search`) and required `limit` (1–100). `request_intent` gates which next-step components are available. `shortlist_size` is removed; `limit` is the single candidate-set size parameter.
- **Matching engine simplified.** Inbound referral signal (and its reciprocal-cap, time-decay, dense-cluster, and Sybil controls) removed. Connection records are kept for de-duplication of `request_intent: 'referral_partner_search'` Discovery only; they do NOT feed matching as a competence or trust signal. The four-component weighted model (relevance / verification quality / availability and response time / profile completeness) in v0.8.2 is a transitional state superseded by Phase 5's stratified-randomisation `band_definitions` model.
- **Looking-toggle for Component 3.** Both professional-identity schemas add `looking_for_referral_partners` (boolean, default `false`); only professionals with the toggle set to `true` are in the candidate pool for inbound Component 3 proposals.
- **Authorisation matrix refactor.** Spec §7.3 is regrouped per `authenticated_actor.actor_type` with subtables expressing permissions in principal/delegation terms (per F-2). Component 3 endpoint rows require professional delegated authority and MUST NOT accept `consent_receipt_id` (per F-4).
- **Audit-event schema split.** The v0.8.1 generic `actor` object on `audit-event.schema.json` is replaced by three top-level fields (`authenticated_actor`, `client_application`, `authority_context`) referencing the shared `principal-delegation.schema.json` `$defs`. `client_application` is conditionally required for `ai_platform` events (per F-2). Per F-16, `consumer` is not added to `authenticated_actor.actor_type`; consumer-triggered events are recorded as `actor_type: ai_platform` with `represented_principal_type: consumer`.

### Added

- **Spec §4.9A — Principal and Delegation Model.** Documents the three-object shape and combination rules; cross-referenced from §4.10 and §7.3.
- **Component 3 endpoints (8 new MCP tools, 4 new OpenAPI paths).** `propose_referral_connection`, `list_inbound_referral_proposals`, `list_outbound_referral_proposals`, `respond_to_referral_proposal`, `withdraw_referral_proposal`, `list_my_referral_connections`, `withdraw_referral_connection`, `set_looking_for_referral_partners`. REST: `POST /v1/referral-proposals` (updated), `GET /v1/referral-proposals/inbound`, `GET /v1/referral-proposals/outbound`, `POST /v1/referral-proposals/{id}/withdraw`, `GET /v1/referral-connections`, `POST /v1/referral-connections/{id}/withdraw`. `/v1/referral-network` is removed in favour of `/v1/referral-connections`.
- **`referral_proposal_expiry_months` per category.** New field on `profession-category.schema.json` with default 6; v0.8.2 ships with 6 months across all categories.
- **MCP `permitted_authority_bases` per tool.** Each MCP tool declares the authority bases under which it may be invoked (per F1.2).
- **OpenAPI authority-basis-to-OAuth-scope mapping.** The `securitySchemes.oauth2.description` documents the mapping from each `authority_basis` value to the OAuth grant flow.
- **ADR 0009.** Records the principal/delegation model decision and the rejected alternatives (single combined object, per-actor-type schemas, `consumer` as `actor_type`).
- **Stress-test matrix skeleton.** New `spec/v0.8/stress-test-matrix.md` with 10 categories, 41 seed scenarios, F-17 status vocabulary (`skeleton` / `resolved` / `bounded-by-protocol` / `allocated-to-platform-responsibility` / `partially-resolved` / `deferred`), and per-phase update discipline.
- **Spectral ruleset.** New `.spectral.yaml` extending `spectral:oas` so per-phase OpenAPI lint is a uniform invocation.

### Removed

- The professional-attestation-of-client-consent path. The associated v0.8.1 ROADMAP item proposing `consumer` in `actor_type` is closed as superseded.
- `inbound_referral_signal` from the matching engine and from `profession-category.matching_weight_profile`.
- `shortlist_size` from `query.schema.json` (replaced by `limit`).

### Charter and governance updates (Phase 8)

Strengthens Charter Core Commitment 8 with Paolo Piponi's verbatim normative rule on protocol-vs-implementation governance (per F1.6):

> ATAH protocol governance MUST be held by an independent not-for-profit or equivalent public-interest entity. Conforming implementations MAY be operated by commercial, public, professional-body, or not-for-profit entities, provided they satisfy conformance, transparency, neutrality, audit, and conflict-of-interest requirements.

The Charter now expresses a deliberate three-layer governance distinction:

- **Protocol governance body** — independent not-for-profit or equivalent public-interest entity. Form is mandated.
- **Reference registry operator** — not-for-profit, public-benefit, community-interest company, foundation-owned subsidiary, or commercial under strict constraints (enforceable neutrality, public fee schedules, auditability, data-use limits, structural separation). Form is constrained, not mandated.
- **Third-party conforming implementations** — commercial, public-sector, professional-body, or not-for-profit. Form is unconstrained; conformance requirements (the five conformance classes — Core Object, Binding, Registry, Governance, Transparency) are uniform.

Mirrored in spec §1.5 (Three layers of conforming implementations + new "Three governance layers" subsection), `CONFORMANCE.md` (new "Three governance layers" section), `GOVERNANCE.md` §1.1 (new), `EXPLAINER.md` (Governance section expanded), and `PRD-v0_8.md` §12 (new "Three governance layers" subsection).

Per Paolo's F1.7, Charter Part Two adds a **hard-artifacts list** — the trust floor lives in concrete published artifacts, not committee discretion. Eleven items enumerated, each cross-referenced to the existing artifact (where applicable) or noted as v0.9 work (where applicable):

- Field-level verification scopes per partner — existing (`verification-scope.schema.json`)
- Partner class admission criteria — v0.9
- Review-platform minimum criteria — existing (`review-platform.schema.json`)
- Verifier conflict and audit rules — partial; v0.9 fuller specification
- Category annex templates — existing process; v0.9 canonical template
- Mandatory audit events — existing (`audit-event.schema.json` extended in Phases 1, 3, 4, 6, 7)
- Professional appeal state machines — partial (`dispute-record.schema.json`); v0.9 fuller specification
- Public governance-decision register — v0.9 operational
- Public registry of approved partners with scopes and freshness commitments — partial; v0.9 publication channel
- Public conformance tests — Phase 6's `CONFORMANCE.md` defines them; v0.9 executable test suite
- Revocable conformance mark — existing concept; v0.9 revocation mechanism

ROADMAP closes the v0.8.1 "consumer actor_type in audit-event schema" item explicitly as superseded by the v0.8.2 principal/delegation model (per F-16). The closure is explicit so the item doesn't drift back into v0.9 candidates.

No ADR for Phase 8 per master plan (Charter strengthening of existing commitments rather than a new architectural decision).

### Contact-detail freshness mechanism (Phase 7)

Implements `GKC-COMMENTS-02` in full as a three-layer mechanism. Spec §12A documents the design.

- **Layer 1 — periodic channel verification** with category-tiered cadence: `quarterly` for high-stakes regulated categories (credentialled tier with mandatory body membership); `biannual` for mid-stakes regulated; `annual` for established practitioners; `annual` as the protocol-level default. Cadence stored on `profession-categories.json` `contact_verification_cadence` (per-category configurable).
- **Layer 2 — escalation if verification fails.** Confirmation deadline 14 days; grace before escalation a further 30 days; grace before `matching_status` flips to `contact_unverified` a further 30 days. Total 74 days from initial challenge. Partner-route professionals are notified through their partner organisation; individuals self-registered are escalated via other registered channels + in-portal alert + connected AI assistant alert.
- **Layer 3 — pre-handoff freshness check (high-stakes categories only).** Per-category `pre_handoff_freshness_window_days` (30 days for high-stakes in v0.8.2). Before sending the introduction notification, if the relevant channel hasn't been verified within the window, ATAH issues a fresh verification challenge and waits up to 24 hours. On timeout, the introduction is deferred back to the consumer's AI with status `professional_contact_unverified`.

Per F-15, both professional identity schemas (credentialled and established) gain a new top-level `notification_channels` array. Each channel record carries `channel_id`, `channel_type` (enum: `email`, `phone_voice`, `phone_sms`, `mcp_push`, `webhook`), `destination_reference` (HMAC of the destination per §11.8 — NOT plaintext), `delivery_purpose`, `is_primary`, `last_verified` (nullable for not-yet-verified), `verification_status` (lifecycle: `not_yet_verified` → `verification_pending` → `verified` → `escalation_pending` → `unverified`), and `created_at`. Channels are stored on the professional record itself (not in a separate operational schema) per the v3.1.1 locked decision.

`matching_status` enum on both professional identity schemas adds `contact_unverified`. `audit-event.schema.json` `event_type` enum adds `channel_verification_challenged`, `channel_verification_confirmed`, `channel_verification_escalated`, `channel_marked_unverified`, `matching_status_flipped_to_contact_unverified`, `matching_status_restored_from_contact_unverified`.

New OpenAPI endpoints: `GET /v1/professionals/me/notification-channels`, `POST /v1/professionals/me/notification-channels/{channel_id}/verify`, `GET /v1/admin/partners/{partner_id}/professionals/verification-status`. New MCP tools: `list_my_notification_channels_with_verification_status`, `confirm_notification_channel_verification`. `initiate_introduction` documents the new `professional_contact_unverified` deferred-back status and adds the corresponding error code.

Per spec §12A.4, verification challenges follow a documented anti-phishing format (distinctive sender domain, content-template enforcement, no embedded redirect URLs, single-use tokens with short TTL, confirmation surface displays the professional's `atah_id` and challenge timestamp). The standardised format itself is a v0.8.3 / v0.9 standardisation candidate.

**Migration note.** v0.8.1 records have no `notification_channels` array. v0.8.2 implementations migrate on first read: create the array, populate channel records from existing notification configuration data, set `last_verified: null` and `verification_status: not_yet_verified` on migrated channels. The first verification cycle then transitions them through `verification_pending` to `verified`. v0.8.1 is unpublished, so no externally-deployed records require migration; the path is documented for partner and reference-registry implementations with v0.8.1-shaped internal data.

**Engagement liveness deferred to v0.8.3.** v0.8.2 verifies channel reachability. It does not verify that a professional with verified contact details actually responds to introductions. Response-rate tracking, missed-Stage-1 patterns, and engagement-liveness review remain v0.8.3 work — listed in the ROADMAP "v0.8.3 candidates" section.

### Transparency-as-conformance and decision-explanation (Phase 6)

Per Paolo's F1.8 / F1.9, transparency becomes a top-level conformance requirement. v0.8.2 adds a fifth conformance class (Transparency Class) alongside Core Object, Binding, Registry, and Governance. Spec §11A carries Paolo's MUST verbatim:

> Conforming implementations MUST produce machine-readable explanations for discovery, exclusion, ordering, handoff, withdrawal, suppression, and data-sharing events, subject to privacy, security, and anti-gaming limits.

New `decision-explanation.schema.json` carries machine-readable explanations. Per F-5, the schema defines the inner object only — the field name `decision_explanation` is the wrapper in consuming response schemas, NOT inside the schema itself; implementers MUST NOT re-wrap when copying or referencing.

Per F-6, the transparency model has three layers in match responses:

- **Layer 1** — response-level `decision_explanation` (required on `match-response.schema.json`).
- **Layer 2** — per-candidate `decision_explanation` (required for every candidate in the results array).
- **Layer 3** — aggregate `exclusion_summary` with reason-category counts (required when there were exclusions). Names of excluded candidates are NOT surfaced through Layer 3; the dedicated auditor endpoint is the only surface for named-excluded-candidate detail.

Per F-6, the professional-facing transparency obligation is part of v0.8.2 even where the dedicated endpoint implementation is deferred to v0.8.3:

- New OpenAPI endpoint: `GET /v1/professionals/me/visibility-explanations` (fully specified; implementations MAY return 501 in v0.8.2 by declaring `x-implementation-deferred-to: v0.8.3`).
- New MCP tool: `get_my_visibility_explanations`.
- Authority basis: `professional_delegated_token` (or `firm_delegation`); each retrieval is audit-logged; rate-limited per professional account.

Per F-18 (rules-derived, not query-history-derived):

> Professional-facing visibility explanations MUST NOT expose actual query-count data, query-history data, observed demand patterns, or any per-query information derived from consumer or platform traffic. They MUST present representative explanation categories at the category/jurisdiction level, derived from the implementation's documented rules and the professional's own profile data, not from observed query traffic.

The existing `docs/professional/visibility-report.md` posture is preserved and codified as a normative MUST NOT in spec §11A.4. Implementations cannot use observed query traffic to populate the professional-facing view; the view is derived from `band_definitions`, the professional's own profile, and the implementation's published rules. v0.9 may revisit with privacy-preserving aggregate mechanisms (k-anonymity floors, differential privacy) but v0.8.2's position is rules-derived only.

New OpenAPI endpoint `GET /v1/decision-explanations/{audit_event_id}` (governance / auditor only) returns named-excluded-candidate detail and full per-decision rationale; restricted to `governance_admin_role` authority; each retrieval is itself audit-logged.

`audit-event.schema.json` adds optional reverse reference `decision_explanation_id`. `handoff-component2.schema.json`, `referral-proposal.schema.json`, and `referral-connection.schema.json` add optional `decision_explanation` for lifecycle events. `match-response.schema.json` makes `decision_explanation` required at both response level and per-candidate level (replacing the Phase 5 placeholder), and adds `exclusion_summary`.

Privacy / security / anti-gaming withholding (per §11A.5) is permitted only for those three reasons and is recorded on the explanation via `withholding_basis` (`personal_data_protection`, `security_vulnerability_protection`, `anti_gaming_protection`, `no_withholding`); the reason something happened MUST still be explainable at category level.

ADR 0013 records the decision and acknowledges Paolo Piponi's peer review as the source.

### Ordering, randomisation, and no global match_score (Phase 5)

Replaces the v0.8.1 weighted-scoring matching engine with hard-filters-then-stratified-randomisation per Paolo's F1.10 normative position:

> ATAH may determine eligibility and exclusion, but MUST NOT express preference among eligible candidates unless the ordering basis is explicit, non-commercial, and disclosed. Where no user-requested ordering criterion is supplied, candidate order MUST be randomised or rotated using a documented fairness policy.

The four-step model: hard filters → band assignment → threshold exclusion → randomisation within bands. Per-category bands are declared in `profession-categories.json` `band_definitions` (default bands: verification confidence, category fit, availability window, contact freshness). Within-band ordering uses a documented fairness policy (`uniform_random`, `round_robin_rotation`, or a published `documented_implementation_policy`). The randomisation seed is not disclosed (`not_disclosed_to_prevent_gaming`) — disclosing it would create a gaming surface.

`match-response.schema.json` removes in full: `match_score` (global), `match_factors` (per-result object and all sub-properties), `presentation_disclosure.ranking_basis` (array enum), every reference to `inbound_referral_signal`. Replaced with: per-candidate `filters_passed` (array), `band_assignment` (object with bands and `position_in_response`), `ordering_policy` (object with `mode` and `within_band_policy`); response-level `presentation_disclosure.ordering_policy` (the canonical disclosure AI platforms reordering downstream MUST preserve). Examples updated end-to-end. The "no global score" claim is structurally enforced: no `match_score` field exists at any layer of the schema.

`query.schema.json` adds optional `ordering_preference` parameter: when omitted, the default is stratified randomisation; when supplied, ATAH applies the named mode (`nearest`, `soonest_available`, `remote_available`, `specific_language`, `highest_verification_confidence`, `specific_category_qualification`, `professional_type`, `jurisdiction`) and records it on the response.

Per F-7, `profession-category.schema.json` replaces `review_signal_weight_cap` with `review_signal_band_cap`:

> Review-derived signals MAY supplement transparency and confidence metadata, but for high-stakes regulated categories they MUST NOT move a candidate into a higher eligibility or verification band unless corroborated by authoritative credential or regulator-source evidence.

Encoded as `max_band_influence: supplemental_only`, `may_upgrade_band: false`, `regulated_category_max_effect: no_band_upgrade` for high-stakes regulated categories. Lower-stakes categories permit `may_upgrade_band: true` with documented corroboration.

Per F-13, `profession-category.schema.json` is updated atomically with `profession-categories.json`: `matching_weight_profile` and `review_signal_weight_cap` removed from properties and required; `band_definitions` and `review_signal_band_cap` added to properties and required. All 15 launch categories rewritten with the new shape; data file validates against schema.

Spec §9 fully refactored as four-step pipeline with verbatim normative MUST NOT (§9.1) and F-7 MUST (§9.2). New §9.3 documents user-requested ordering modes; new §9.4 documents the AI platform presentation obligation (preserve `presentation_disclosure.ordering_policy` verbatim; no recommendation framing). §9.5 acknowledges the v0.8.2 algorithm-agnostic posture; v0.9 may standardise. EXPLAINER and PRD §8.5 carry plain-language framings. Charter Part Two operational commitments mirror the §9 normative rules.

ADR 0012 records the decision and acknowledges Paolo Piponi's peer review as the source.

### Consent boundaries (Phase 4)

Adopts Paolo's three consent types: `query_authorization`, `disclosure_consent`, and **`engagement_consent` excluded by design**. The `consent_type` enum on `consent-receipt.schema.json` carries only the two ATAH-supported values; engagement consent (any professional-client agreement, fee agreement, treatment consent, regulated advice consent, conflict waiver, scope-of-work acceptance, or instruction to act) is the professional's responsibility and outside ATAH. Spec §4.10 carries Paolo's MUST NOT rule verbatim:

> ATAH consent receipts MUST NOT be represented as professional engagement consent. Conforming implementations MUST disclose that any professional-client relationship arises only through the professional's own onboarding, engagement, regulatory, or contractual process.

The receipt-hash limitation framing is also adopted verbatim:

> ATAH verifies integrity of the submitted consent receipt. The asserting AI platform remains responsible for obtaining valid user consent to disclose data.

The submission lifecycle is now Option B: the asserting platform calls `POST /v1/consent-receipts` with the full receipt body; ATAH validates, computes the SHA-256 hash, computes the F-3 continuity-binding metadata, stores the hash and metadata, and returns the `consent_receipt_id`. Subsequent consenting endpoints reference by id only; the full receipt body is never re-submitted. The new `submit_consent_receipt` MCP tool mirrors the REST endpoint. Symmetric `POST /v1/consent-receipts/{consent_receipt_id}/revoke` covers query-scope and pre-handoff revocations (handoff-bound revocations continue to use `/v1/introductions/{handoff_id}/revoke-consent`).

The F-3 continuity-binding fields are added to `consent-receipt-stored.schema.json`: `client_id` (asserting platform's client identifier), `pseudonymous_consumer_ref` (HMAC of the platform-side consumer reference), `data_categories_hash` (SHA-256 of the canonicalised data-categories list, for `disclosure_consent`). On consumption, ATAH verifies the consuming session's `client_id` and pseudonymous reference match the stored values, and that the requested action falls within the captured scope and data-categories hash. Mismatch produces a consent-mismatch error and a `consent_continuity_mismatch` audit event. The continuity-binding fields are non-identifying; the build-time privacy-floor test verifies they are HMAC-shaped or hash-shaped.

Per F-4, the `scope` enum on `consent-receipt.schema.json` and `consent-receipt-stored.schema.json` removes `type_2_referral_participation` and `type_3_referred_client`. Component 3 referral-connection actions are governed by professional delegated authority (per spec §6.4 and §7.3) and do NOT use consumer consent receipts. Spec §6.4 carries the Paolo-style MUST rule verbatim:

> Component 3 professional referral-connection actions MUST be authorised through authenticated professional delegation. They MUST NOT use consumer disclosure-consent receipts and MUST NOT imply consumer involvement.

`audit-event.schema.json` `event_type` enum gains `consent_continuity_mismatch` for consumption-time verification failures. Charter Part Two operational commitments mirror the engagement-consent and Component 3 normative rules (spec §4.10 and §6.4 are the canonical source).

### Fixed (Phase 4 absorbs from `AI-PEER-REVIEW-FINDINGS-01`)

- **F1.3 (no consent receipt submission path).** Resolved by `POST /v1/consent-receipts` (and the `submit_consent_receipt` MCP tool). Every consenting endpoint now traces back to a recorded submission.
- **F1.5 (spec/schema contradiction on consent receipt issuer).** Resolved by §4.10: consent receipts are issued by the asserting AI platform; ATAH receives the full receipt at submission, computes and stores the hash plus continuity-binding metadata, and returns the `consent_receipt_id`. Spec text and schema agree.

ADR 0011 records the decision and acknowledges Paolo Piponi's peer review as the source.

### Three-concept separation (Phase 3)

Refactored §11 around the three-concept separation of payload erasure, audit retention, and withdrawal-as-state-transition. v0.8.2 distinguishes:

- **Payload erasure** (§11.6) — adopts Paolo's verbatim normative wording on crypto-erasure: vault payloads are crypto-erased by destroying the data-encryption key; encrypted ciphertext may remain in storage or backups until ordinary retention expiry, but is treated as unrecoverable once the corresponding key material has been destroyed. Six implementation requirements are normative (unique key per object; keys stored separately from ciphertext; destroyed keys excluded from recoverable backups; no plaintext in logs / queues / analytics / indexes / notification providers; auditable key-destruction events; short retrieval-token TTLs and single-use retrieval tokens).
- **Audit retention** (§11.8) — adopts Paolo's verbatim normative rule: crypto-erasure applies to vault payload content, not audit metadata; conforming implementations MUST retain tamper-evident audit records sufficient to investigate abuse, disputes, authentication failures, and consent-integrity claims, without retaining consumer personal data in plaintext. The §11.8 erase-fields list and retain-fields list are documented verbatim. References to contact identifiers and device identifiers in audit records MUST use HMACs with protected audit keys; plain hashes of email or phone numbers are guessable and explicitly excluded.
- **Withdrawal-as-state-transition** (§11.9) — adopts Paolo's verbatim normative rule: withdrawal stops future processing but does not erase audit records, consent receipt metadata, state history, abuse signals, dispute records, or legally required retention records. Six distinct withdrawal scenarios are documented (consumer pre-share / post-Stage-2 / post-Stage-3-retrieval, professional withdrawal from an introduction, professional withdrawal from matching, Component 3 proposal/connection withdrawal). Step-up authentication is mandatory at the point of action for high-impact withdrawals (§11.9 / §13.2A).

`audit-event.schema.json` extended with the F1.16 retain-field list (`handoff_id`, `query_id`, `professional_id`, `proposal_id`, `connection_id`, `pseudonymous_consumer_ref` HMAC-form, `request_intent`, `stage`, `payload_type`, `consent_receipt_id`, `receipt_hash`, `retrieved_at`, `erased_at`, `retrieval_actor`, `auth_tier`, `abuse_flags`, `source_ip_hmac`, `user_agent_hmac`) and the F1.17 withdrawal-context fields (`previous_state`, `resulting_state`, `reason_code`). `event_type` enum extended with Component 3 events, professional withdrawal events, and `key_destroyed`. `subject_type` enum extended with `referral_proposal`, `referral_connection`, `vault_object`, `delegated_token`. `handoff-component2.schema.json` adds `withdrawal_state` summary classification.

New OpenAPI / MCP: `POST /v1/professionals/me/withdraw` (and `withdraw_from_matching` MCP tool) for Component 2 withdrawal scenario 5 (professional withdrawal from matching), with mandatory step-up authentication. `/v1/introductions/{handoff_id}/cancel` and `/revoke-consent` descriptions updated to map to Paolo's withdrawal scenarios 1, 2, and 3.

ADR 0010 records the three-concept separation decision and acknowledges Paolo Piponi's peer review as the source.

---

## [v0.8.1] — April 2026

**Status:** Release candidate. First public publication.

### Summary

v0.8.1 incorporates substantive pre-publication review remediation across legal/regulatory risk, market readiness/coherence, and adoption blockers. The protocol's purpose, actors, and lifecycle are unchanged from v0.8.0; v0.8.1 tightens governance commitments, corrects readiness-overstating language across the package, addresses GDPR posture for professional data, expands the security model with worked threat scenarios and incident response, and adds a small set of platform-facing and professional-facing operational documents.

### Changed

- **Governance transition reworked.** Removed the 90-day interim advisory group deadline and the 12-month transition deadline. Replaced with trigger-based transition (two trusted partner integrations live + one external conforming implementation demonstrated) and periodic public review when there is meaningful progress to share. Aligns with the realistic adoption timelines of cross-industry standards work and with the protocol's actual maintenance posture. Charter bumped to v1.3.
- **Roadmap phase dates removed.** PRD §14 Roadmap reframed as direction of work rather than delivery schedule. Quarterly dates (Q2 2026, Q3 2026, Q4 2026, 2027, 2027–2028) removed from all phases. Named-partner deliverables (NIPR, FINRA, SRA, FCA, etc.) reframed as illustrative examples. "Independent governance entity established" removed as Phase 3 deliverable — independent governance is now trigger-based per Charter, not phase-based. Phase 1 rewritten to describe only the publication-period activity (publication, repo live, community review, outreach) rather than aspirational reference-registry deliverables.
- **"Validated" → "verified" in consumer-facing surfaces.** Across README, EXPLAINER, PRD, and CHANGELOG. Preserved "validator stack" as a technical term of art.
- **"Permanent" → "entrenched" in CHARTER core-commitment framing.** Removes the implication of unamendable permanence; the supermajority amendment process remains in place.
- **Named regulators and bodies reframed as illustrative examples.** Throughout README, EXPLAINER, PRD, CHANGES-FROM-V0_7-TO-V0_8, and spec. Named bodies (NIPR, FINRA, state bars, PRSA, CIPR, the International Coaching Federation, etc.) appear as representative examples of the *types* of organisation the protocol works with — not as committed partners. New explanatory paragraph added to README and EXPLAINER making this convention explicit.
- **Notification transport reframed as binding-level.** SMS, email, MCP push, and other transports are now described as conforming-implementation choices rather than protocol mandates. The protocol-level requirement is no-PII through any notification channel.
- **MCP tool count corrected.** PRD §8.13 corrected from "eight" to "nine" tools.
- **"Press launch" → "publication announcement"** in PRD §14, aligning with the release-candidate framing.
- **Visible separation of regulatory sources from review platforms.** README and EXPLAINER trusted-partner passages rewritten to distinguish authoritative regulatory sources and professional membership bodies (the credential-verification authority layer) from review platforms (signal providers, not authority providers, with anti-gaming attestation requirements and per-category review weight caps). "Review-derived signals complement but do not substitute for credential verification" wording added at meaningful surfaces.
- **AEO equivalency framing softened.** "Without mastering Answer Engine Optimisation" reframed as "as part of their AI-discoverability infrastructure" across README, EXPLAINER, PRD. Clarifying sentence added to README and EXPLAINER: ATAH is not a replacement for a professional's existing marketing or visibility work; there is no guarantee of introduction volume, ranking, or work; the protocol is designed to become a trusted handoff rail as platforms integrate.

### Added

- **Professional Data Protection section (PRD §9).** GDPR posture for professional data: ATAH as data controller, lawful bases by source, professional notice and rights (access, rectification, objection, erasure subject to public-task carve-outs, restriction, portability), partner-route opt-out, partner agreement DPA terms, audit log treatment.
- **Platform presentation obligations (PRD §11, EXPLAINER mirror).** Platforms must not describe ATAH results as "recommended," "approved," "best," "endorsed," or "vouched for" by ATAH.
- **Liability allocation freshness paragraph (PRD §11).** Explicit statement that platforms are expected to surface verification freshness; ATAH does not warrant accuracy beyond published freshness windows.
- **Concern flag lifecycle (PRD §8.11) and Concern flag data handling (GOVERNANCE §7).** Full lifecycle from intake through escalation; lawful basis, retention, professional rights, escalation thresholds.
- **Consent receipt limitation (EXPLAINER, PRD §8.7).** ATAH verifies receipt integrity, not consent capture; platform retains responsibility for the consent ceremony.
- **Revocation by stage (EXPLAINER, PRD §8.6).** Clarifies what revocation actually achieves at each stage of the introduction.
- **Cross-jurisdiction default rule (PRD §8.5).** No cross-jurisdiction matching by default for regulated categories; explicit query parameter and disclosure required.
- **Vetting strength surfaced in match responses (EXPLAINER, PRD §6.3).** Explicit `vetting_strength` enum (`regulatory`, `strong_membership`, `open_membership`) with public-facing classification names. Six public-facing partner classes added: authoritative regulatory source, professional membership body, open membership body, review signal provider, independent verifier, public registry source.
- **Worked threat scenarios in SECURITY.md.** Three scenarios — leaked handoff_id, malicious AI platform asserting forged consent receipt, compromised partner submitting poisoned data.
- **Incident response in SECURITY.md.** Confirmed professional fraud and professional account compromise procedures.
- **Conformance self-certification disclaimer (CONFORMANCE.md).** Conformance statements are self-declared until v0.9 conformance test suite ships.
- **Founder COI extended to immediate family (CHARTER §7).**
- **Founder seat language tightened (CHARTER §7).** Seat is personal, non-transferable, non-inheritable; lapses on founder unavailability.
- **DPA acknowledgement (CHARTER Part Two, GOVERNANCE §4).** Partner agreements include data-protection terms.
- **Disciplinary-data treatment subsection (PRD §8.4).** Severity categorisation, spent/expired discipline, withdrawal of complaints, professional right of reply.
- **Mid-handoff status change handling (PRD §8.6).** Procedure when a professional's matching status changes mid-handoff.
- **Conflicting credentials hierarchy (PRD §8.4).** Resolution order: regulator > statutory register > mandatory professional body > strong voluntary body > independent verifier > review platform > self-declared.
- **Capacity assumption (PRD §9).** ATAH assumes the asserting platform has determined consumer capacity.
- **Roll-up objection branch (spec §10.4).** What happens after a professional objects within the 7-day pre-merge window. Conditional-required `acknowledged_rollup_terms` field added to both professional schemas to close the contract gap with OpenAPI registration documentation.
- **`consumer_ref` MUST NOT specification (spec §11.5).** Negative constraints on `consumer_ref` to close cross-platform correlation risk.
- **Acknowledgements rewrite (README).** Leads with human authorship; AI tools framed as drafting/stress-testing/editorial assistants with human-led structural decisions.
- **COVERAGE.md.** Launch coverage matrix by category × jurisdiction × verification route × matchable-at-launch.
- **`presentation_disclosure` mention in README.**
- **"Isn't this just X?" FAQ in EXPLAINER.**
- **Consent Storage Rationale doc (`docs/consent-storage-rationale.md`).** Design rationale for the consent receipt split between AI platform and ATAH.
- **Issue templates (`.github/ISSUE_TEMPLATE/`).** spec-change, category-annex, bug, config.
- **Maintenance posture note (README, GOVERNANCE).** Release-candidate-stage maintenance framing added to README and GOVERNANCE.md. Published SLAs framed as best-effort at release-candidate stage; project actively seeking reviewers, partners, implementers, and advisors. Resourcing posture tied to the trigger-based transition framework — capacity grows alongside partner integrations and external implementations.
- **Operational independence and service-provider framework (CHARTER Part One Core commitment 8, Part Two new subsection, GOVERNANCE §4.1).** General-principle framing: ATAH services may be operated by service providers (contractors, technology partners, managed-service providers, contributors, employees, supporting organisations) on commercial or paid terms. Operational arrangements never confer protocol ownership or exclusive control. Every operational service-provider arrangement must be terminable on reasonable notice, replaceable, and not exclusive. Independent governance must be structurally able to replace any operational service provider. Where the service provider is the founder, a founder-affiliated entity, an advisor-affiliated entity, or any other related party, the arrangement is governed by GOVERNANCE.md §4.1 (related-party register, payment structure permitted, negative list, replaceability application). Clarifying sentence added to Core commitment 7 confirming governance compensation and operational service-provider compensation are separate categories. "Founder-affiliated entity" defined once in Core commitment 7 for use throughout. No version bump — recorded as part of CHARTER v1.3.
- **Forks, derivatives, and official ATAH status (CHARTER Core commitment 1, Core commitment 7, README new subsection).** Clarifying language added addressing the boundary between forks/derivative works (permitted under Apache 2.0, but not the official ATAH protocol) and conforming implementations (operating the protocol under conformance criteria, part of the ATAH ecosystem). The Charter clarifies that the open licence permits derivatives but does not extend the official ATAH governance line, the founder advisory seat, or the ATAH project identity to those derivatives. The README adds a short subsection making the boundary explicit for general readers. Forks may not represent themselves as the official ATAH protocol or claim ATAH conformance, certification, governance continuity, or founder participation where none exists.
- **Verification confidence signal (spec §4.12, PRD §8.5).** Match responses for regulated categories will carry a `verification_confidence` field separate from verification-quality score components, surfacing single-source vs multi-source verification basis. Field name, value enumeration, and semantics committed in v0.8.1; schema implementation lands in v0.8.2.
- **Consolidated liability allocation subsection (PRD §11).** New "Liability allocation and platform/ATAH responsibilities" subsection consolidating what ATAH commits to, what ATAH does not warrant, what platforms take on, and worked examples of common cases. EXPLAINER mirror paragraph and README "What ATAH is *not*" bullet added.
- **Adoption Blockers in-scope items.** Six new operational documents: `docs/platform-adoption/handoff-trigger-guidance.md` (P2), `docs/platform-adoption/ux-disclosure-copy.md` (P7), `docs/platform-adoption/commercial-neutrality-memo.md` (P8), `docs/professional/visibility-controls.md` (P22), `docs/professional/visibility-report.md` (P29), `docs/professional/firm-admin.md` (P31). Plus inline patches: integration cost honesty in README (P12), public partner taxonomy paragraph in PRD §6.3 (P14), Stage 1 appetite-check controls in PRD §6.1 (P25), field-level visibility classes in PRD §8.2 (P27), regulatory-vs-review separation wording sweep (P15), AEO equivalency wording sweep (P32). P10 is satisfied by COVERAGE.md.
- **Schema/OpenAPI delta corrections.** Prose-only corrections aligning spec text with schemas, OpenAPI, and MCP tools file: §4.1 prefix list (removed unused `rp-`, reframed as non-exhaustive), §4.10 `data_categories` (added `contact_preference` to prose list to match schema), §6.4 Type 2 example (`rp-` → `proposal-`), §7.3 authorisation matrix (scope names aligned to OpenAPI's actual `securitySchemes`), §8.1 OAuth scopes (removed orphan `atah:consent:revoke`), §8.2 `find_professional` input shape (matches `query.schema.json` matter sub-object), §10.4 dispute-resolution cross-reference (§8.10 → §5.9). OpenAPI Stage 2 endpoint: empty 202 removed, `held_status` folded into 200 response matching MCP semantics.
- **ROADMAP v0.8.2 candidates expanded.** Six candidates tracked: `verification_confidence` schema implementation, MCP `initiate_introduction` Type 3 support, REST `get_consent_requirements` endpoint, `presentation_disclosure` standalone schema refactor, `check_introduction_status` response shape alignment between OpenAPI and MCP, non-handoff-bound consent revocation endpoint, `consumer` actor_type addition to audit-event schema.

### Held

The following items from the three pre-publication reviews are held for post-publication work (some tracked externally in non-published planning artefacts):

- Items needing legal review (Platform Legal Integration Pack, Model partner data-sharing agreement, AI Platform Developer Terms, Published partner indemnity framework, Category-specific ethics memos and referral-fee legal opinions)
- Items needing partner conversations (Member Communications Pack, Partner approval criteria with worked examples, Lower-tier partner ingestion + onboarding process, Worked compliance annexes for specific categories)
- Items needing operational decisions not yet made (Individual self-registration fee number, Operational SLA / RTO/RPO, Operational claim authentication procedure)
- Items requiring schema changes or new code (those listed in the v0.8.2 candidates section above)
- Operational tasks for the founder (seeding GitHub Discussions before launch; identifying interim advisory group members; press cycle and association outreach)

### Known limitations (deferred to v0.8.2)

- **MCP `initiate_introduction` Type 3 support.** The MCP tool description references Type 3 referrals but the input schema only models Type 1. The REST endpoint (`POST /v1/introductions`) handles both Type 1 and Type 3 via the `introduction_type` discriminator and is the v0.8.1 workaround for MCP clients needing Type 3. Full MCP Type 3 support lands in v0.8.2.
- **REST `GET /v1/consent-requirements` endpoint.** MCP-native platforms have the `get_consent_requirements` tool. REST-only platforms in v0.8.1 should source consent text and required `data_categories` from spec §4.10 and `consent-receipt.schema.json` directly. A native REST endpoint with the same response shape as the MCP tool lands in v0.8.2.

### Security

Tiered handoff access control, no-PII-in-notifications, consent receipt hash storage, audit log pseudonymisation, and the rest of the v0.8.0 security commitments are unchanged. v0.8.1 adds worked threat scenarios and incident response procedures (SECURITY.md §4 and §8 respectively).

### Privacy

Privacy floor for consumer data (Charter Part One core commitment 5) is unchanged. v0.8.1 adds the Professional Data Protection section addressing professional data — the larger volume surface area — with controller role, lawful basis, professional rights, partner notice, erasure handling, and partner DPA acknowledgement.

---

## [v0.8.0] — April 2026

**Status:** Release candidate. First public publication.

### Summary

v0.8.0 is the release-candidate version of ATAH, incorporating substantial pre-publication peer review remediation across architecture, security, privacy, governance, and documentation. The protocol's purpose, actors, and lifecycle are unchanged from v0.7; v0.8 tightens, completes, and clarifies what was previously implied or under-specified.

### Added

- **Protocol-layer refinement.** Clarified ATAH Core Protocol, transport bindings, registry operation profile, and reference registry boundaries. The initial ATAH registry remains the first operational implementation, but the specification is now explicitly transport-compatible and implementation-neutral at the core object layer. README, EXPLAINER, and spec language updated to distinguish protocol from reference implementation throughout. New spec Section 14 (Conformance Classes — four classes: Core Object, Binding, Registry, Governance) and Section 15 (Professional Records and Registration — three record creation routes: partner-created, claimed, direct self-registered). Charter v1.2: Open specification commitment made explicitly royalty-free; Open MCP endpoint commitment scoped to the ATAH-operated reference endpoint and clarified for future conforming implementations.
- **Three-layer framing.** Specification, conforming implementations, and initial reference registry are now distinguished explicitly throughout the documentation.
- **Standards composition section.** Explicit treatment of OAuth 2.1, OpenID Connect, W3C Verifiable Credentials, and Decentralized Identifiers — what ATAH uses, what it accepts, and where the boundaries are.
- **Federation framing.** v0.8 is single-registry by design; the architecture is federation-ready (atah_id namespace, no single-registry assumption in the data model). Federation mechanics deferred to v0.9 / v1.0.
- **Non-goals section.** Explicit list of what ATAH is not — recommendation engine, regulator, payment processor, directory product, personal data repository, consumer interface — added to README, spec, EXPLAINER, and PRD.
- **Glossary.** Section 16 in the spec, extended with new terms for the protocol-layer framing.
- **CONFORMANCE.md.** Top-level repository file summarising the four conformance classes (Section 14 of the spec) for discoverability by standards reviewers and implementers.
- **ROADMAP.md.** Top-level repository file describing v0.8.1, v0.9, and v1.0 candidate work in narrative form. Granular tracking moves to GitHub Issues once the repository is live.
- **Stored consent receipt as separate schema.** `consent-receipt-stored.schema.json` defined as a distinct schema from `consent-receipt.schema.json` so the privacy boundary (no `consumer_ref`, no `data_categories`, no consent text in the stored form) is structurally enforceable through schema-validation tests.
- **Schema conventions block.** Section 4.1 of the spec defines opaque ULID-style identifiers, RFC 3339 UTC timestamps, closed enums, additionalProperties policy, and schema versioning.
- **Parallel provenance map.** Per-field provenance is now delivered through a `_provenance` map rather than wrapping individual fields. Matches the README's headline trust commitment without bloating schemas.
- **Type 2 and Type 3 introduction schemas.** Full lifecycle schemas, consent flows (including structured client consent receipts and structured professional attestations of client consent for Type 3), and state machines.
- **Consent receipts.** Boolean `consumer_consent` replaced with structured consent receipt objects following W3C Consent Receipt and Kantara CR 1.1 patterns. ATAH stores the receipt hash and metadata; the asserting platform stores the full receipt (Sigstore / RFC 6962 transparency log pattern).
- **Tiered handoff access control.** `handoff_id` no longer grants access on possession alone. Read-like operations available to original asserter or to authenticated AI platforms with matching consumer reference (preserving cross-platform user experience). Write operations require a `handoff_access_token` bound to platform/client/scope/expiry.
- **Transient encrypted vault and authenticated retrieval.** Section 11.6 of the spec. Personal data never transmitted through SMS or email — only notification metadata and authenticated retrieval references. Three authentication tiers (`tier_1`, `tier_2`, `tier_3`) per category sensitivity. Crypto-erasure as primary deletion mechanism.
- **Idempotency requirement.** All mutating endpoints require an `Idempotency-Key` header.
- **Authorisation matrix.** Section 7.3 of the spec defines who can do what with which scope and object-level constraints.
- **Cancel and revoke as first-class operations.** New `cancel_introduction` and `revoke_consent` MCP tools and corresponding REST endpoints. New `get_consent_requirements` tool to support consent capture.
- **Per-category matching weight profiles.** Categories may define `matching_weight_profile` in `profession-categories.json`. Regulated categories weight verification quality higher than availability. Category-specific overrides require governance approval.
- **Review weight cap by category.** `review_signal_weight_cap` field caps review platform contribution to verification quality per category. ≤0.10 for high-stakes regulated categories.
- **Anti-gaming controls on inbound referral signal.** Reciprocal cap, time decay, referrer verification quality weighting, evidence of successful outcomes for full weight, dense-cluster discounting.
- **Declared/verified scope completeness split.** Sub-component D of verification quality split into declared and verified. Only verified materially contributes.
- **Concern flag protections.** Admin-only visibility, never used in matching, professional notification with 30-day right of reply, pattern-based review, bad-faith flag detection.
- **Founder conflict-of-interest, recusal, and disclosure rules.** Annual public disclosure, recusal from decisions affecting founder-affiliated entities, public register of related-party transactions.
- **Concrete governance transition covenant.** 90-day interim plan, 12-month transition deadline or earlier on triggers, public-explanation safety valve for any deadline revision.
- **Interim advisory structure.** Required consultation before approving the first verifier, review platform, partner agreement, category compliance annex, or governance change.
- **Privacy and compliance gating floors as Charter Part One core commitments.** Promoted from operational to permanent.
- **Partner principles, professional participation principles, verifier/review platform principles** added to Charter Part Two.
- **Hardship waivers and fee caps** for individual self-registration. **Fee waivers** for public-interest partners.
- **Presentation disclosure block** in every match response. Machine-readable non-recommendation framing AI platforms must surface to users.
- **Failure-mode honesty section** in PRD. ATAH does not warrant outcomes; AI platforms and consumers retain responsibility.
- **Capture resistance section** in PRD.
- **Opaque atah_id format.** ULID-style identifiers; country, profession, and category are stored as attributes rather than encoded in the ID.
- **Profile-level matching status** split: `regulatory-suspended` and `admin-suspended` distinguished. New states: `withdrawn`, `deceased`.
- **Location field split.** `licensed_jurisdictions`, `service_locations`, `physical_locations`, `remote_service_available`.
- **Missing supporting documents.** GOVERNANCE.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, CHANGELOG.md (this file), CHANGES-FROM-V0_7-TO-V0_8.md added to repository.
- **Custom verification dimension structure.** Required structured `custom_dimension` block including governance approval reference.
- **Pre-merge notification for high-risk roll-up matches.** 7-day objection window for regulated-tier categories.

### Changed

- **MCP authentication updated.** OAuth 2.1-compatible authorisation; protected resource metadata; authorization server metadata; resource indicators (RFC 8707); audience validation; explicit scopes; short-lived access tokens with refresh and rotation; Dynamic Client Registration (RFC 7591). Conforms to current MCP authorisation guidance at the time of implementation.
- **MCP tool naming.** `submit_stage2_consent` renamed to `submit_stage2_data` and `submit_stage3_consent` renamed to `submit_stage3_data` to reflect that these tools transmit both consent and data.
- **Authentication mechanisms by actor.** Partners and verifiers writing high-trust data now require signed JWT client assertion or HMAC-signed requests. API keys retained for read-only operations only.
- **Documentation tone.** Replaced "complete and implementable as written" with "ready for technical review and reference implementation." Replaced "commercial model is clean" with "designed to separate payment from ranking." Removed "the timing is right" / "the window will not stay open" pitch language from PRD.
- **Charter version bumped to v1.2.** v1.1 reframed Charter as a public governance covenant; founder undertakes to administer ATAH consistently with its commitments. v1.2 adds protocol-layer refinement: royalty-free protocol implementation made explicit; Open MCP endpoint commitment scoped to the ATAH-operated reference endpoint with future conforming implementations clarified.
- **PRD bumped to v0.8.** Aligned with the v0.8 spec.

### Deferred to v0.9 / v1.0

The following are acknowledged as known v0.9 candidates rather than v0.8 omissions:

- **Federation mechanics.** Cross-registry trust, atah_id resolution across registries, handoff routing between registries, federated governance.
- **Claim authentication hardening.** v0.8 specifies the minimum (verified email match against partner records or equivalent partner-attested identifier). Step-up authentication, anomaly detection on claim attempts, partner-known-channel notification with objection window, and rate limiting are deferred to v0.9. The reference registry SHOULD apply these controls operationally at launch.
- **Sybil and referral-ring detection.** Beyond the v0.8 anti-gaming controls.
- **Platform-light consent mode.** Optional alternative to the v0.8 self-asserted-receipt model. v0.9 will specify a model where ATAH provides more of the consent ceremony (canonical consent text endpoint, consent flow orchestration helper) for AI platforms that prefer to invoke a standard ATAH consent flow rather than constructing receipts themselves. The v0.8 self-asserted model continues to be supported.
- **Per-category matching weight profiles populated with researched values** for all launch categories. v0.8 ships with defaults plus exemplar overrides.
- **Full STRIDE-level threat model.** SECURITY.md ships with threat categories listed; full modelling deferred.
- **Conformance test suite.** Discovery endpoints (`/.well-known/atah`, `/v1/capabilities`) shipped; the test suite itself is v0.9.
- **Edge-case lifecycle handling.** Detailed handling for jurisdiction changes, partner acquisitions, verifier acquisitions, mid-flight category compliance changes.
- **Detailed dispute resolution process.** Beyond v0.8 baseline.
- **Full DSAR machinery for federation scenarios.**
- **Multilingual support.**

### Security

- Tiered handoff access control mitigates handoff_id leakage as a bearer-token vulnerability.
- Personal data never transmitted through notification channels (SMS, email) mitigates PII leakage through providers.
- Consent receipt hash storage mitigates consent forgery and platform-side receipt tampering.
- Authentication mechanism strengthened for high-trust partner and verifier writes.
- Audit log made tamper-evident (hash-chained) and retains pseudonymous identifiers only.

### Privacy

- Personal data never written to the main registry database.
- Notification channels carry no consumer personal data — only metadata and authenticated retrieval references.
- Crypto-erasure as primary deletion mechanism for the transient vault.
- Deletion scope table (Section 11.2) enumerates every store and its personal-data policy.
- Right to erasure satisfied by the transient-only design for ATAH-held data.

---

## [v0.7] — Pre-publication, internal

Five issues resolved: atomic deletion compensating action pattern specified; MCP endpoint authentication mechanism defined (OAuth 2.0); verification batching strategy made concrete; simultaneous introduction rules specified explicitly; schema versioning policy added.

The PRD-v0.7.md document was retained alongside v0.8 spec work and is replaced by PRD-v0.8.md at v0.8 publication.

## [v0.6] — Pre-publication, internal

Targeted corrections: state bar verification scoped to priority states at MVP; webhooks and MCP push repositioned as Phase 2. Conflict field classification made explicit. Referral signal discount threshold defined concretely. Notification model corrected.

## [v0.5] — Pre-publication, internal

Initial technical specification.

---

*This changelog tracks the protocol specification. Reference implementation changes are tracked in the reference implementation repository.*
