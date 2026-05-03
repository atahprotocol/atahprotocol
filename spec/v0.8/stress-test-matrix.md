# Stress-Test Matrix — ATAH Protocol v0.8.2

**Status:** Skeleton populated in Phase 0A; iteratively completed through Phases 1–10A; finalised in Phase 11.

**Purpose.** This matrix is the verification harness for v0.8.2 protocol design. It captures the abuse, mistake, commercial-conflict, and edge-case behaviours that ATAH must address — at the protocol level, not only as implementation QA. Each subsequent phase updates the matrix as part of its completion checklist: scenarios mapped to the phase get their resolution columns filled in and their status updated.

**Scope.** Protocol-design risks: what the spec permits, what it leaves ambiguous, and what conforming implementations might do differently. Stress testing is a publication discipline, not a later implementation afterthought (Paolo's source note `PAOLO-stress-tests.md`).

**Inputs.** F1.18 and F1.19 from the consolidated Paolo peer review; `PAOLO-stress-tests.md` for the seed scenarios.

---

## Status vocabulary (F-17)

Every scenario carries one of the following statuses. The discipline: do not use `resolved` when `bounded-by-protocol` or `allocated-to-platform-responsibility` is more honest. Press-cycle reviewers will scrutinise this matrix; overclaimed `resolved` markers attract criticism.

- **`skeleton`** — populated in Phase 0A; not yet validated against phase output. Initial state for every seed scenario.
- **`resolved`** — protocol behaviour fully addresses the scenario. The protocol's stated rules, when followed, prevent or detect the failure mode. Reserve for cases where ATAH's own behaviour closes the loop.
- **`bounded-by-protocol`** — protocol detects, limits, or makes verifiable a scenario without fully eliminating it. Example: "Platform submits consent for a user who did not consent" — ATAH verifies receipt integrity (tampering, replay, mismatch) but cannot prove the platform's underlying consent ceremony was valid. The protocol bounds the abuse without resolving it.
- **`allocated-to-platform-responsibility`** — scenario is real but its resolution belongs to the AI platform, partner, or verifier under their own obligations. ATAH documents the obligation and exposes verifiable signals; the scenario itself is closed by another party's behaviour, not ATAH's. Example: "AI platform presents ATAH result one as 'best'" — ATAH publishes ordering policy in `presentation_disclosure`; the platform's reordering and presentation is its own responsibility.
- **`partially-resolved`** — scenario is partly addressed by v0.8.2 work, partly pending further work. Use for scenarios that span phases (resolution in one phase + further work in a later phase) and for scenarios where v0.8.2 does substantial-but-incomplete work. Should always carry a reference to either the completing phase or the deferral version.
- **`deferred`** — scenario is acknowledged but not addressed in v0.8.2. Carries an explicit deferral version (`deferred-to-v0.8.3`, `deferred-to-v0.9`, `deferred-to-v1.0`) and a one-line reason.

---

## Scenario format

Each scenario uses the following columns:

- **Scenario** — concrete description.
- **Abuse / failure mode** — what goes wrong.
- **Expected protocol behaviour** — what conforming implementations MUST do.
- **Required audit events** — which audit events MUST be recorded.
- **Required user / professional disclosure** — what is surfaced to which roles.
- **Required conformance test** — how to verify the behaviour.
- **Phase mapping** — which planned Phase(s) of v0.8.2 will resolve this scenario.
- **Status** — one of the F-17 statuses above.

In Phase 0A all scenarios are populated with the **Scenario**, **Phase mapping**, and **Status: skeleton** fields. The other fields are filled in by the corresponding phase as part of its checkpoint discipline.

---

## Category 1 — Authentication and Delegation Abuse

**Phase mapping:** Phase 1.

### 1.1 Agent falsely declares `request_intent`

- **Scenario.** A consumer-mediated AI platform issues a query to ATAH and declares an inaccurate `request_intent`, either to obtain a different result class than the user authorised or to mask the true purpose.
- **Abuse / failure mode.** Misrepresentation of authority basis; downstream protocol decisions (eligibility filters, transparency obligations, audit semantics) are taken under a false premise.
- **Expected protocol behaviour.** Every protocol action carries a `principal-delegation.schema.json` block recording `authenticated_actor`, `client_application`, and `authority_context.represented_principal_type` / `authority_basis` / `permitted_scopes`. ATAH validates the declared `authority_basis` against the credential class actually presented (e.g. `user_session` actually backed by an authorization-code-with-PKCE token); a mismatch results in `403`. Phase 2 introduces server-side validation of `request_intent` against the action class permitted under that authority basis, completing the loop.
- **Required audit events.** `event_type: security_event` (or the action's normal `event_type` with structured metadata indicating `intent_validation: rejected`), recording the declared `request_intent`, the authority basis presented, and the rejection reason.
- **Required user / professional disclosure.** None to consumer (an attacker should not learn the validation logic). Audit trail is accessible to the asserting platform on request, per §13.5.
- **Required conformance test.** Conformance suite includes a request with mismatched `request_intent` / `authority_basis` and asserts `403` plus an audit event matching the metadata schema.
- **Phase 1 contribution.** `authenticated_actor` + `client_application` + `authority_context` shape established; per-operation authority-basis validation enforced via §7.3 matrix and OpenAPI per-operation `security` blocks.
- **Phase 2 contribution.** `request_intent` is now a required parameter on `query.schema.json` (`self` / `on_behalf_of_client` / `referral_partner_search`) and is the architectural lever gating Component 2 and Component 3 availability. Component 2 endpoints reject calls whose originating Discovery did not declare `request_intent: 'self'`; Component 3 endpoints reject calls under any authority basis other than `professional_delegated_token` / `firm_delegation`. The principal-delegation context recording the declared intent is persisted on the audit-event for every protocol action, so retroactive review of intent declarations is possible.
- **Status: `resolved`** by Phase 2 — declaration is required, downstream component access is gated server-side, and the declared intent is recorded in the audit trail.

### 1.2 Platform submits consent for a user who did not consent

- **Scenario.** An AI platform presents a consent receipt at a Stage 2 / Stage 3 boundary for a consumer who did not perform the required consent ceremony.
- **Abuse / failure mode.** Fabricated or recycled consent; downstream professionals receive data on a false consent basis.
- **Expected protocol behaviour.** Per spec §4.10, ATAH verifies the receipt's integrity (hash + continuity binding) at submission and at every consumption point. Per F-3, the stored receipt carries `client_id`, `pseudonymous_consumer_ref` (HMAC), and `data_categories_hash` (for `disclosure_consent`); on consumption, the consuming session's `client_id` and pseudonymous reference must match the stored values. Per the Paolo receipt-hash limitation framing (verbatim in spec §4.10): "ATAH verifies integrity of the submitted consent receipt. The asserting AI platform remains responsible for obtaining valid user consent to disclose data."
- **Required audit events.** `consent_receipt_submitted` records the receipt with the principal-delegation context. `consent_continuity_mismatch` is emitted on any consumption attempt whose binding does not match the stored values. `consent_revoked` flips `revocation_status` per §4.10.
- **Required user / professional disclosure.** Mismatched-binding consumption attempts surface as `consent_mismatch` errors to the consuming AI platform. Asserting platforms remain responsible for the underlying consent ceremony; ATAH disclaims that responsibility explicitly through the verbatim receipt-hash limitation framing.
- **Required conformance test.** Conformance suite verifies all six F-3 continuity-binding mismatches (client_id mismatch, pseudonymous_consumer_ref mismatch, scope mismatch, data_categories_hash mismatch, expired receipt, revoked receipt) produce rejections with `consent_continuity_mismatch` audit events.
- **Phase 1 contribution.** `authority_context` records the basis the asserting platform claims; `consent-receipt.schema.json` `asserted_by` uses the principal-delegation `AuthenticatedActor` + `ClientApplication` $defs so the asserter identity is recorded with the same shape ATAH validates everywhere else.
- **Phase 4 contribution.** F-3 continuity binding fields (`client_id`, `pseudonymous_consumer_ref`, `data_categories_hash`) added to `consent-receipt-stored.schema.json`. The verbatim receipt-hash limitation framing is in spec §4.10. The submission lifecycle (Option B — `POST /v1/consent-receipts`) makes every consumption traceable to a recorded submission.
- **Status: `bounded-by-protocol`** (per F-17 settled status). ATAH detects tampering, replay, and continuity mismatch via receipt hash + continuity binding fields. ATAH cannot and does not prove that the platform's underlying consent ceremony was valid (whether the user actually clicked "I consent" in the platform's UI). The protocol bounds the abuse without resolving it. The platform retains the legal responsibility for the underlying consent ceremony.

### 1.3 Professional delegated-agent token is stolen

- **Scenario.** An attacker obtains a delegated agent's credentials or token and acts on a professional's behalf without that professional's knowledge.
- **Abuse / failure mode.** Unauthorised actions taken under a real professional's identity; profile changes, withdrawal, response-on-behalf-of without the principal's authority.
- **Expected protocol behaviour.** Authority basis `professional_delegated_token` carries `expires_at` (mandatory for time-bounded credentials per `principal-delegation.schema.json`) and is bound to `object_constraints` of `professional_id` (own). Tokens are short-lived and rotated; the §7.3 matrix permits them only for `actor_type: professional` actions on the bound professional id.
- **Required audit events.** Every action under `professional_delegated_token` records the token's `expires_at` and the bound `professional_id` in `authority_context`. Anomaly detection on the audit stream flags token use after revocation, from unexpected client applications, or against object constraints other than the bound principal.
- **Required user / professional disclosure.** Professional-facing audit feed (`GET /v1/professionals/me/disputes` and equivalent visibility endpoints) surfaces actions taken under their delegated tokens with the token's basis and constraints.
- **Required conformance test.** Conformance suite verifies that a request with `authority_basis: professional_delegated_token` whose `expires_at` is past, or whose `object_constraints[professional_id]` does not match the targeted principal, is rejected.
- **Status: `resolved`** by `authority_basis: professional_delegated_token` with mandatory `expires_at` and `object_constraints`, combined with audit-event recording of the basis. Operational hardening (token rotation cadence, anomaly detection thresholds) is registry-implementation responsibility.

### 1.4 Firm admin withdraws or changes a professional profile improperly

- **Scenario.** A firm admin (delegated authority over multiple professionals' profiles) withdraws or amends a professional's record outside the scope of the authority that professional granted.
- **Abuse / failure mode.** Authority creep at the firm level; individual professional's protocol-relevant state changed without their explicit assent.
- **Expected protocol behaviour.** Firm admin actions carry `authenticated_actor.actor_type: professional` (or `admin` for ATAH-side firm-admin roles where applicable) with `authority_context.represented_principal_type: firm_admin` and `authority_basis: firm_delegation`. `object_constraints` MUST narrow the action to the specific professional ids the firm admin is authorised to manage; ATAH rejects actions targeting professionals outside the constrained set.
- **Required audit events.** Every firm-admin action records `authenticated_actor`, the `firm_delegation` basis, and the `object_constraints` set; the affected professional's audit feed surfaces the firm-admin action distinctly from their own actions.
- **Required user / professional disclosure.** The affected professional's `GET /v1/professionals/me/disputes` and notification stream surface firm-admin-originated changes with the firm and the action explicitly labelled.
- **Required conformance test.** Conformance suite verifies that a `firm_delegation` request without `object_constraints` is rejected, and that an action targeting a `professional_id` outside the constraint set is rejected.
- **Phase 1 contribution.** `firm_delegation` is in the `authority_basis` enum; `object_constraints` shape supports per-professional narrowing. **Status: `partially-resolved`** — Phase 1 supplies the structural basis; Phase 3's step-up authentication for high-impact withdrawal actions completes the resolution.

### 1.5 Consumer handoff is attempted from a different platform/session without continuity authority

- **Scenario.** A handoff (Stage 2 / Stage 3) is attempted using identifiers or receipt that originated on a different AI platform or session than the one whose continuity binding the receipt records.
- **Abuse / failure mode.** Cross-platform replay; consent receipt or principal-delegation context being reused outside its issuing platform/session.
- **Expected protocol behaviour.** _To be filled in during Phase 4 (continuity binding)._
- **Required audit events.** _To be filled in during Phase 4._
- **Required user / professional disclosure.** _To be filled in during Phase 4._
- **Required conformance test.** _To be filled in during Phase 4._
- **Phase 1 contribution.** `authority_context` records the asserting `client_application` (`platform_id`, `client_id`) and the `authority_basis` (`user_session` for the original asserter, or `handoff_access_token` for stage-changing calls). The §7.3 matrix requires original-asserter match for stage-changing calls. **Status: `partially-resolved`** — Phase 1 supplies the structural carrier; Phase 4 adds the continuity binding fields on the stored consent receipt that detect cross-platform replay end-to-end.

### 1.6 Component 3 proposal spam via looking-toggle gaming

- **Scenario.** An attacker (or a poorly-bounded automated proposer) issues large volumes of Component 3 proposals targeting professionals who have set `looking_for_referral_partners: true`, harassing the targets or saturating their inbound queues. A variant: an attacker repeatedly toggles `looking_for_referral_partners` on briefly to receive proposals from waiting proposers, then off, gaming the de-duplication exclusion.
- **Abuse / failure mode.** Proposal-flow abuse not present in v0.8.1; introduced in v0.8.2 by Component 3.
- **Expected protocol behaviour.** Per-period rate limits on proposals per `proposing_professional_id` (configurable; default category-tier-appropriate). Audit-event records every `referral_proposal_*` action with the principal-delegation context, so abuse patterns are visible. The `looking_for_referral_partners` toggle is default-off; proposals to professionals not currently looking are rejected. Reciprocal/dense-cluster toggle patterns flagged for review (see v0.8.3 deferral below).
- **Required audit events.** `event_type: referral_proposal_created`, `referral_proposal_resolved`, with the resolution reason recorded; for toggle changes, `event_type: profile_updated` with the field-level diff.
- **Required user / professional disclosure.** Targets MAY surface declined / silently-lapsed proposal counts in their professional portal. Proposers MAY surface their own outbound proposal status. Implementation-side, not protocol-mandated.
- **Required conformance test.** Conformance suite verifies that proposals to professionals with `looking_for_referral_partners: false` are rejected, and that the per-period rate limit returns `429` after threshold.
- **Phase mapping.** Phase 2 (introduces the surface and per-period rate limits); residual harassment-monitoring deferred to v0.8.3.
- **Status: `partially-resolved`** — Phase 2 closes the structural side (toggle gating, rate limits, audit recording). The dense-cluster / reciprocal-toggle pattern detection is `deferred-to-v0.8.3` for monitoring.

---

## Category 2 — Consent Boundary Failure

**Phase mapping:** Phase 4.

### 2.1 ATAH handoff consent is misrepresented as engagement consent

- **Scenario.** A Stage 2 or Stage 3 handoff consent is presented to a downstream professional or surface as if it were ongoing engagement consent (e.g. authorisation to retain, contact further, or open a relationship).
- **Abuse / failure mode.** Scope creep at the consent boundary; consumer's narrow authorisation expanded into broader permissions.
- **Expected protocol behaviour.** Per spec §1.5, §4.10, and Charter Part Two operational commitments, ATAH supports two consent types (`query_authorization`, `disclosure_consent`); engagement consent is excluded by design and the `consent_type` enum has no value for it. The Paolo MUST NOT rule applies verbatim across spec, EXPLAINER, PRD, and Charter: ATAH consent receipts MUST NOT be represented as professional engagement consent; conforming implementations MUST disclose that any professional-client relationship arises only through the professional's own onboarding, engagement, regulatory, or contractual process.
- **Required audit events.** `consent_receipt_submitted` records the `consent_type` (one of the two supported values). Any attempt to issue an `engagement_consent` receipt fails validation at submission.
- **Required user / professional disclosure.** AI platforms MUST surface the engagement-consent boundary at point of consent capture; the canonical disclosure language is published in the protocol documentation. EXPLAINER and PRD §9 carry the same framing for downstream readers.
- **Required conformance test.** Conformance suite verifies that a receipt with `consent_type: engagement_consent` is rejected at `POST /v1/consent-receipts` with a validation error.
- **Status: `resolved`** by F1.4 normative rule (verbatim adopted across spec / Charter / EXPLAINER / PRD), the `consent_type` enum restriction on `consent-receipt.schema.json`, and the canonical disclosure-language requirement.

### 2.2 User consents to one professional but data is sent to another

- **Scenario.** Consumer consents at Stage 2 / Stage 3 in respect of a specific candidate; the platform routes data to a different professional (substituted, additional, or the wrong record).
- **Abuse / failure mode.** Mismatch between consented scope and actual recipient; consent receipt's `data_categories_hash` and / or recipient binding does not match the delivery target.
- **Expected protocol behaviour.** Per spec §4.10 (continuity binding, F-3), the stored consent receipt records `professional_id` as part of its captured binding (where the receipt is professional-bound). On consumption, ATAH verifies the consuming session targets the same `professional_id`; mismatch is a continuity-binding violation. For `disclosure_consent`, the requested data categories are also verified against `data_categories_hash`; mismatch is a continuity-binding violation.
- **Required audit events.** `consent_continuity_mismatch` on the failed consumption attempt, with the principal-delegation context, the receipt id, and the mismatched binding field.
- **Required user / professional disclosure.** Mismatch surfaces as a `consent_mismatch` error to the consuming AI platform; the consumer's session is not advanced.
- **Required conformance test.** Conformance suite issues a receipt for professional A then attempts a Stage 2 submission targeting professional B; verifies the submission is rejected with `consent_continuity_mismatch` audit event.
- **Status: `resolved`** by F-3 continuity binding (`professional_id` binding + `data_categories_hash`).

### 2.3 Consent text differs from stored receipt metadata

- **Scenario.** What the consumer was shown at the consent ceremony does not match the metadata recorded on the stored consent receipt (different scope, recipients, data categories, or duration).
- **Abuse / failure mode.** Drift between displayed and recorded consent; later disputes are unresolvable on protocol grounds.
- **Expected protocol behaviour.** Per §4.10, the receipt's `consent_text_version` records the canonical consent text the consumer was shown; the published consent-text version is part of the receipt and is hashed alongside the rest of the receipt body. On dispute, the asserting platform produces the receipt and the consent-text version; ATAH verifies the receipt hash matches the stored hash, confirming the receipt itself was not altered after the fact. The receipt-hash limitation framing applies (Paolo verbatim): ATAH verifies receipt integrity, not the validity of the underlying consent ceremony.
- **Required audit events.** `consent_receipt_submitted` records the `consent_text_version`. Future `consent_continuity_mismatch` events triggered by drift detection are recorded with the divergent fields.
- **Required user / professional disclosure.** Asserting platforms MUST show the consent text matching the recorded `consent_text_version`. Drift is a platform-side conformance failure surfaced through dispute resolution.
- **Required conformance test.** Conformance suite verifies that submitting a receipt with `consent_text_version` not matching any published canonical text produces a validation error.
- **Status: `bounded-by-protocol`** — receipt integrity is verified by hash; ATAH cannot prove what the consumer actually saw on the platform's UI. The platform's consent ceremony remains its own responsibility per the receipt-hash limitation framing.

### 2.4 Consent is revoked after Stage 2 or Stage 3

- **Scenario.** Consumer revokes consent after data has already been delivered to a downstream professional or surface.
- **Abuse / failure mode.** Ambiguous semantics for "consent revoked after delivery" — what must downstream parties do, and what does ATAH itself record?
- **Expected protocol behaviour.** Per §11.9 (Phase 3 — withdrawal as state transition) and §4.10, the receipt's `revocation_status` flips to `revoked`; on the consuming-side, `POST /v1/introductions/{handoff_id}/revoke-consent` performs vault crypto-erasure for any remaining vault payload and stops ATAH processing. ATAH cannot force deletion from the professional's systems where data has already been retrieved (§11.9 scenario 3 documents this acknowledgement). The `revocation_status: active` check on consumption ensures revoked receipts cannot drive new actions.
- **Required audit events.** `consent_revoked` on the receipt; `data_erased` for any remaining vault payload; `withdrawal_recorded` if the revocation came in via the §11.9 scenario 3 path.
- **Required user / professional disclosure.** AI platforms surface the revocation to the consumer; the professional receives a notification that consent has been revoked and that their downstream data-handling obligations under their own privacy framework apply.
- **Required conformance test.** Conformance suite verifies that any consenting endpoint rejects consumption of a revoked receipt with `consent_receipt_revoked` error.
- **Status: `resolved`** by §4.10 `revocation_status` checks at every consumption point + Phase 3 §11.9 scenario 3 (consumer-withdrawal-after-Stage-3-retrieval semantics + vault crypto-erasure).

### 2.5 Receipt replay across users (cross-user confusion)

- **Scenario.** Platform X submits a receipt for user A (`pseudonymous_consumer_ref` HMAC-A); subsequently attempts to consume it for an action concerning user B (whose pseudonymous reference HMACs to a different value).
- **Abuse / failure mode.** Receipt is valid (hash check passes); consumption context does not match captured context.
- **Expected protocol behaviour.** Per F-3 continuity binding, ATAH verifies the consuming session's `pseudonymous_consumer_ref` matches the stored HMAC at consumption time. Mismatch is a continuity-binding violation.
- **Required audit events.** `consent_continuity_mismatch` on the failed consumption with the receipt id and the mismatched binding field (`pseudonymous_consumer_ref`).
- **Required user / professional disclosure.** Failed consumption surfaces as a `consent_mismatch` error to the consuming AI platform.
- **Required conformance test.** Conformance suite issues a receipt for user A then attempts consumption with a different pseudonymous reference; verifies rejection with the `consent_continuity_mismatch` audit event.
- **Status: `resolved`** by F-3 continuity binding (`pseudonymous_consumer_ref` HMAC).

### 2.6 Receipt replay across sessions (same user, different action)

- **Scenario.** A receipt issued for one consenting action is consumed for a different action without re-consent.
- **Abuse / failure mode.** Captured `scope` does not match the consuming endpoint's required scope; or `data_categories_hash` does not match the data being shared at consumption.
- **Expected protocol behaviour.** Per §4.10 continuity binding, ATAH verifies the consuming endpoint's required scope matches the receipt's captured `scope`, and (for `disclosure_consent`) that the requested data categories match the receipt's `data_categories_hash`. Mismatch is a continuity-binding violation. Additionally, the receipt's `expires_at` is enforced.
- **Required audit events.** `consent_continuity_mismatch` on the failed consumption with the mismatched binding field (`scope` or `data_categories_hash`).
- **Required user / professional disclosure.** Failed consumption surfaces as a `consent_mismatch` error.
- **Required conformance test.** Conformance suite issues a receipt with scope `stage_2_prehandoff_submission` then attempts consumption at `stage_3_contact_release`; verifies rejection. Also: receipt with one set of data categories, attempt to share a different set; verifies rejection.
- **Status: `resolved`** by F-3 continuity binding (`scope` and `data_categories_hash` checks) + receipt expiry.

### 2.7 Component 3 endpoint accidentally accepts a consumer consent receipt

- **Scenario.** An implementation, by oversight or attempted escalation, accepts a `consent_receipt_id` parameter on a Component 3 endpoint and treats it as authorising a referral connection.
- **Abuse / failure mode.** Category error: consumer disclosure consent is treated as professional referral-connection authorisation. Could escalate consumer consent into broader professional-side actions.
- **Expected protocol behaviour.** Per F-4 / spec §6.4 / Charter Part Two operational commitments: Component 3 actions MUST be authorised through authenticated professional delegation; they MUST NOT use consumer disclosure-consent receipts. The `scope` enum on `consent-receipt.schema.json` and `consent-receipt-stored.schema.json` does NOT contain `type_2_referral_participation` or `type_3_referred_client`. Component 3 OpenAPI endpoints and MCP tools have no `consent_receipt_id` parameter.
- **Required audit events.** Any attempt to register a `consent_receipt_id` against a Component 3 action fails validation; the failed attempt is recorded as a `security_event`.
- **Required user / professional disclosure.** Failed attempt surfaces to the calling agent; the action does not proceed.
- **Required conformance test.** Conformance suite attempts to call a Component 3 endpoint with a `consent_receipt_id` parameter; verifies rejection (the parameter is not part of the request schema). Conformance suite also verifies that no consent receipt can be issued with `consent_type: engagement_consent` or with the v0.8.1 `type_2_referral_participation` / `type_3_referred_client` scopes (those values are absent from the enum).
- **Status: `resolved`** by F-4: `consent_receipt_id` is structurally absent from Component 3 endpoints/tools; the scope enum excludes the v0.8.1 referral values.

---

## Category 3 — Ordering and Recommendation Drift

**Phase mapping:** Phase 5.

### 3.1 Candidates are always returned in the same order

- **Scenario.** Implementations return the same candidate ordering for substantially equivalent queries, leading to entrenched advantage for consistently top-listed professionals.
- **Abuse / failure mode.** Ordering becomes a de facto recommendation; stratified-randomisation requirement bypassed in practice.
- **Expected protocol behaviour.** Per spec §9.1 step 4, ordering within bands MUST use a documented fairness policy: `uniform_random`, `round_robin_rotation`, or a published `documented_implementation_policy`. The randomisation seed is not disclosed (`not_disclosed_to_prevent_gaming`). Per §9.5, conformance verifies that the returned `band_assignment` data is consistent with `band_definitions` and that within-band ordering is observably non-deterministic across repeated queries.
- **Required audit events.** Discovery responses record the `ordering_policy.mode` per the response-level `presentation_disclosure` and the per-candidate `ordering_policy.within_band_policy`; pattern detection of identical orderings across identical queries is a registry-implementation operational concern.
- **Required user / professional disclosure.** AI platforms reordering downstream MUST preserve the response-level `presentation_disclosure.ordering_policy` per §9.4.
- **Required conformance test.** Conformance suite issues the same query repeatedly and verifies that within-band candidate ordering is observably different across at least N invocations (N tuned per band size).
- **Status: `resolved`** by §9.1 stratified-randomisation default + the §9.5 conformance requirement on observable non-determinism.

### 3.2 Hidden score influences order despite non-recommendation claims

- **Scenario.** Implementation publishes "no global match score" framing while internally computing a score and using it to bias ordering.
- **Abuse / failure mode.** Recommendation in disguise; the published ordering policy does not reflect the actual ranking basis.
- **Expected protocol behaviour.** Per spec §9.1, v0.8.2 removes `match_score`, `match_factors`, `presentation_disclosure.ranking_basis`, and every reference to `inbound_referral_signal` from `match-response.schema.json` in full. The "no global score" claim is structurally observable: no `match_score` field exists at any layer of the schema. Per F1.11, "a hidden global score plus randomised display would look cosmetic and undermine the claim that ATAH is not ranking by preference"; the structural removal closes that vector.
- **Required audit events.** None specific (the structural absence is the verification surface, not an audit-event surface).
- **Required user / professional disclosure.** Response-level `presentation_disclosure.ordering_policy` declares the mode (`stratified_random` by default) and `atah_expresses_preference: false`.
- **Required conformance test.** Conformance suite verifies that no `match_score` or `match_factors` field is present in any match-response example or in implementation responses; that `band_assignment` is populated for every candidate; and (per §9.5) that within-band ordering is observably non-deterministic.
- **Phase 2 contribution.** Removed `inbound_referral_signal` from `profession-category.matching_weight_profile` and from match-response example outputs.
- **Phase 5 contribution.** Removed `match_score`, `match_factors`, `presentation_disclosure.ranking_basis` from the schema entirely; replaced with `filters_passed`, `band_assignment`, `ordering_policy`. The transitional four-component weighted model is gone.
- **Status: `resolved`** by Phase 5's full removal of the weighted-score architecture from the schema.

### 3.3 Commercial partner candidates appear disproportionately

- **Scenario.** Professionals associated with commercial partners (paying or otherwise) appear in eligible result sets at frequencies inconsistent with their share of the eligible population.
- **Abuse / failure mode.** Commercial bias entering ordering through a back channel — band assignment, eligibility filters, or stratification weights — without disclosure.
- **Expected protocol behaviour.** Per spec §9.6 (Commercial neutrality), no weight is assigned to ATAH-partner commercial relationships, partner status itself, enhanced-verification payment, review-platform identity beyond `review_platform_class`, or registration route (partner vs individual). Per F-7 / §9.2, review-derived signals MAY supplement transparency but MUST NOT promote candidates into higher bands in regulated categories. The response-level `presentation_disclosure.ordering_policy.commercial_weighting: false` is `const`-asserted in the schema (the field is structurally not a free value). Phase 6's transparency-as-conformance work adds per-candidate `decision_explanation` so the basis of band assignment is auditable per response.
- **Required audit events.** Audit log records every `partner_data_pushed` and `verifier_data_submitted` event with the principal-delegation context; pattern detection of partner-affiliated overrepresentation is an operational concern.
- **Required user / professional disclosure.** `commercial_weighting: false` is asserted in every match response. AI platforms reordering MUST preserve this.
- **Required conformance test.** Conformance suite verifies (a) `commercial_weighting: false` is asserted in every response; (b) per Phase 6, `decision_explanation` exposes the band-assignment basis for each candidate so audit can verify no partner-relationship signal entered the assignment.
- **Status: `resolved`** by §9.6 commercial-neutrality MUST + §9.2 review-signal band cap + Phase 6 per-candidate decision_explanation transparency.

### 3.4 AI platform presents ATAH result one as "best"

- **Scenario.** ATAH returns a stratified set with a published ordering policy; the AI platform reorders, summarises, or labels the first result as "best" or "top recommendation" downstream.
- **Abuse / failure mode.** Platform-side presentation undermines ATAH's stratification and equal-treatment posture; users experience a recommendation despite the protocol's stance.
- **Expected protocol behaviour.** Per spec §9.4: "AI platforms reordering the ATAH response MUST preserve the `presentation_disclosure.ordering_policy` field verbatim and surface its content to the user. AI platforms MUST NOT present ATAH's response as a recommendation or rank by ATAH's preference; ATAH expresses no preference among eligible candidates by default." Failure to preserve the disclosure is a binding-conformance failure (Section 14). The structural surface ATAH provides — `presentation_disclosure.ordering_policy` with `atah_expresses_preference: false` — is the platform's obligation to surface verbatim.
- **Required audit events.** Not directly auditable from ATAH's side (the platform is downstream of ATAH's surface). Platform conformance is verified through binding-conformance review, not audit-stream pattern detection.
- **Required user / professional disclosure.** AI platforms display `presentation_disclosure.ordering_policy` content to users. Where the platform reorders for user-context reasons, the disclosure that ATAH applied non-preferential ordering MUST survive.
- **Required conformance test.** Binding-conformance review verifies platform implementations preserve and surface the `presentation_disclosure.ordering_policy` block.
- **Status: `allocated-to-platform-responsibility`** per F-17. ATAH publishes the ordering-policy disclosure; downstream presentation is the AI platform's obligation. ATAH's role is to provide the structural disclosure surface and the binding-conformance MUST; the platform's role is to honour it. (This was the F-17 worked example for `allocated-to-platform-responsibility`.)

### 3.5 Review signals quietly upgrade a candidate's band in a regulated category

- **Scenario.** A regulated-category implementation lets review-platform signals push a candidate into a higher verification-confidence or category-fit band, despite the absence of corroborating regulator-source evidence.
- **Abuse / failure mode.** Review-platform signal becomes a back-door promotion mechanism in high-stakes categories; the v0.8.1 `review_signal_weight_cap` original safeguard is bypassed.
- **Expected protocol behaviour.** Per spec §9.2 (F-7 verbatim): review-derived signals MAY supplement transparency and confidence metadata, but for high-stakes regulated categories MUST NOT move a candidate into a higher eligibility or verification band unless corroborated by authoritative credential or regulator-source evidence. Encoded per category in `profession-categories.json` `review_signal_band_cap`: regulated categories set `may_upgrade_band: false` and `regulated_category_max_effect: no_band_upgrade`.
- **Required audit events.** Band-assignment changes (where the implementation supports re-binding) record the basis on the audit event; pattern detection of review-driven band promotions in regulated categories is an operational concern.
- **Required user / professional disclosure.** `review_signal_summary` on `match-response.schema.json` is presentational; the protocol's framing in §9.2 makes clear that for regulated categories the signal is supplemental-only.
- **Required conformance test.** Conformance suite verifies that for any regulated category (`may_upgrade_band: false`), an attempt to construct a band assignment that promotes a candidate based on review signals alone is rejected; the candidate remains in the band determined by authoritative credential or regulator-source evidence.
- **Status: `resolved`** by §9.2 verbatim MUST rule + per-category `review_signal_band_cap` configuration.

### 3.6 Implementation chooses a non-uniform random algorithm that systematically favours certain candidates

- **Scenario.** An implementation declares stratified-randomisation conformance but uses a within-band randomisation algorithm that systematically favours certain candidates (biased PRNG, weighted-by-something-else, fixed seed across queries).
- **Abuse / failure mode.** Within-band ordering is observably non-uniform across queries; the structural "no preference" claim is undermined at the algorithmic level.
- **Expected protocol behaviour.** Per §9.5, v0.8.2 specifies the model (hard filters → bands → within-band randomisation) and the within-band fairness policy enumeration (`uniform_random`, `round_robin_rotation`, `documented_implementation_policy`). The randomisation seed is `not_disclosed_to_prevent_gaming`. v0.8.2 does NOT mandate a specific algorithm; that is implementation choice. The conformance test verifies returned `band_assignment` is consistent with `band_definitions` and within-band ordering is observably non-deterministic across repeated identical queries.
- **Required audit events.** None specific (the structural verification surface is the conformance test, not the audit log).
- **Required user / professional disclosure.** Response-level `presentation_disclosure.ordering_policy.randomization_seed_disclosure` is `not_disclosed_to_prevent_gaming`; per-candidate `ordering_policy.within_band_policy` declares the policy applied.
- **Required conformance test.** Conformance suite issues the same query repeatedly and verifies within-band candidate ordering is observably different across N invocations. Phase 6 / Phase 11 may add a stronger statistical test; v0.9 may standardise the algorithm.
- **Status: `partially-resolved`** — Phase 5 establishes the structural model and the conformance non-determinism test; full statistical-distribution conformance and algorithm standardisation are deferred. Concrete next step: Phase 6 / Phase 11 algorithm-conformance test; v0.9 algorithm standardisation.

---

## Category 4 — Transparency and Explainability Failure

**Phase mapping:** Phase 6.

### 4.1 Professional asks why they were excluded

- **Scenario.** A verified professional, expecting to appear in eligible result sets, queries the protocol about exclusion reasons.
- **Abuse / failure mode.** No mechanism for professional-facing transparency; or the mechanism leaks query-history / volume data inappropriately.
- **Expected protocol behaviour.** Per spec §11A.4 (Phase 6), a conforming implementation MUST provide a mechanism for an authenticated professional to retrieve their own visibility-decision view. The view is **rules-derived**: representative inclusion rules that apply, representative exclusion reason categories that could apply, current band assignment from `band_definitions`, and the implementation's published ordering policy. Per F-18 verbatim MUST NOT, the view does NOT expose actual query-count or query-history data. Endpoint: `GET /v1/professionals/me/visibility-explanations` (REST) and `get_my_visibility_explanations` (MCP). Implementations MAY defer the endpoint implementation to v0.8.3 via `x-implementation-deferred-to`; the obligation is part of v0.8.2.
- **Required audit events.** Each retrieval generates a `security_event` (or equivalent) recording the principal-delegation context, the category and jurisdiction queried, and the timestamp.
- **Required user / professional disclosure.** Authority basis: `professional_delegated_token` or `firm_delegation`. Rate-limited per professional account.
- **Required conformance test.** Conformance suite verifies the endpoint returns the rules-derived shape (representative inclusion rules, representative exclusion reason categories, current band assignment, published ordering policy); verifies absence of query-history fields in the response; verifies authority-basis enforcement.
- **Status: `resolved`** by §11A.4 verbatim normative rule + dedicated endpoint specification (implementation may defer to v0.8.3 but obligation is v0.8.2).

### 4.2 Consumer asks why a candidate appeared

- **Scenario.** A consumer (via the AI platform) asks why a particular professional was returned in the eligible set, or why one candidate appeared above another in the stratified ordering.
- **Abuse / failure mode.** Decision basis is opaque; ordering looks like a recommendation; consumer trust depends on a transparent rationale.
- **Expected protocol behaviour.** Per spec §11A.2 (Phase 6), the per-candidate `decision_explanation` (Layer 2) on `match-response.schema.json` documents the rules each candidate passed, the band assignment basis, and the ordering policy applied. The response-level `decision_explanation` (Layer 1) documents the query-level rules and ordering policy. Both layers reference the audit event for traceability.
- **Required audit events.** Discovery responses produce audit events linked from `decision_explanation.audit_event_id`; the audit event records the principal-delegation context.
- **Required user / professional disclosure.** AI platforms surface the per-candidate `decision_explanation` to the consumer through their UI. Per §9.4, the response-level `presentation_disclosure.ordering_policy` MUST be preserved verbatim through any AI-platform reordering.
- **Required conformance test.** Conformance suite verifies both layers of `decision_explanation` are populated on every Discovery response with non-empty results; verifies content is internally consistent with the implementation's published rules.
- **Status: `resolved`** by §11A.2 two-layer model + per-candidate `decision_explanation` required field on `match-response.schema.json`.

### 4.3 Auditor asks whether commercial weighting affected results

- **Scenario.** A regulator, conformance auditor, or governance reviewer asks whether commercial relationships influenced eligibility, ordering, or presentation in a given response.
- **Abuse / failure mode.** Without per-response transparency artifacts, the question is unanswerable from the protocol record alone.
- **Expected protocol behaviour.** Per `decision-explanation.schema.json`, every `decision_explanation` carries `ordering_policy.commercial_weighting: false` (const-asserted). Per §9.6, no weight is assigned to ATAH-partner commercial relationships, partner status, enhanced-verification payment, review-platform identity, or registration route. Auditors retrieve the full per-decision rationale via `GET /v1/decision-explanations/{audit_event_id}` under `governance_admin_role` authority — this returns the named-excluded-candidate detail and the full explanation object.
- **Required audit events.** Audit log retains the principal-delegation context and the linked `decision_explanation_id` reverse reference for cross-correlation.
- **Required user / professional disclosure.** None additional — the auditor surface is governance-only and itself audit-logged.
- **Required conformance test.** Conformance suite verifies (a) `commercial_weighting: false` is asserted on every `decision_explanation` and at the response-level `presentation_disclosure`; (b) the auditor retrieval endpoint returns the full explanation including any excluded-candidate detail; (c) auditor retrievals themselves produce audit events.
- **Status: `resolved`** by `decision_explanation.ordering_policy.commercial_weighting: false` (const) + `GET /v1/decision-explanations/{audit_event_id}` governance endpoint.

### 4.4 Partner data conflict suppresses a profile and the professional appeals

- **Scenario.** A trusted-partner data feed reports a conflict (suspended licence, sanction, complaint) that suppresses or downgrades a professional's appearance; the professional appeals on the grounds the conflict record is wrong.
- **Abuse / failure mode.** Partner errors propagate without correction path; or the appeal mechanism is opaque.
- **Expected protocol behaviour.** Per §11A.2 / §11A.4, the suppression is recorded as a `decision_explanation` with `decision_type: suppression`; the affected professional sees the suppression reason category through their visibility-explanation view. The partner data conflict records (`conflict-record.schema.json`) and dispute machinery (`dispute-record.schema.json` + `POST /v1/professionals/me/disputes`) are the existing v0.8.1 appeal surfaces; v0.8.2 §11A makes the transparency component required.
- **Required audit events.** `meaningful_conflict_detected` (existing event_type) records the suppression with `decision_explanation_id` reverse reference; the dispute lifecycle produces `dispute_raised` and `dispute_resolved` events.
- **Required user / professional disclosure.** The professional's visibility-explanation view shows the suppression reason category; the dispute portal (existing v0.8.1) gives the professional a structured appeal path.
- **Required conformance test.** Conformance suite verifies suppression events generate `decision_explanation` with `decision_type: suppression`; verifies the dispute machinery produces audit-linked events.
- **Phase 6 contribution.** Transparency on suppression reason — `resolved`. Detailed appeal-workflow and timeline-of-resolution work — flagged as v0.9 deeper-dispute-resolution work.
- **Status: `partially-resolved`** — Phase 6 closes the transparency side (suppression reason visible; dispute path visible). The fuller dispute-resolution timeline / escalation criteria / evidence-handling specification is v0.9 work (already on ROADMAP under "Detailed dispute resolution process").

### 4.5 Implementation provides response-level explanation only and claims conformance

- **Scenario.** An implementation produces a response-level `decision_explanation` but omits per-candidate `decision_explanation` on individual results, claiming response-level alone is sufficient.
- **Abuse / failure mode.** Per-candidate "why this specific candidate" question becomes unanswerable from the response alone; conformance gap exploited.
- **Expected protocol behaviour.** Per §11A.2 (F-6), the Transparency Class requires both layers. `match-response.schema.json` `MatchResult` `required` includes `decision_explanation`; a response without per-candidate explanations fails schema validation.
- **Required audit events.** Schema-validation failures produce conformance-test failures; not directly audit-logged at runtime.
- **Required user / professional disclosure.** Conformance test failure surfaces through the conformance test suite (v0.9).
- **Required conformance test.** Conformance suite picks a Discovery response with non-empty `results` and verifies every result item has `decision_explanation` populated. Schema validation rejects response payloads missing the field.
- **Status: `resolved`** by Transparency Class conformance requirement + schema-level required field.

### 4.6 Exclusion summary leaks excluded professional identities

- **Scenario.** An implementation populates `exclusion_summary` with named excluded professionals or with reason-category granularity small enough to single-out individual professionals (e.g. one entry per excluded professional with a reason-specific name).
- **Abuse / failure mode.** Exclusion data leaks excluded professionals' identities and match-status to a consumer who didn't ask.
- **Expected protocol behaviour.** Per §11A.2, `exclusion_summary` is **aggregate-only**: a `total_excluded` count and a `reason_categories` map from category identifier to count. The schema's `exclusion_summary` shape does not permit per-candidate detail. Named-excluded-candidate detail is available only through the dedicated `GET /v1/decision-explanations/{audit_event_id}` endpoint under `governance_admin_role` authority.
- **Required audit events.** Auditor retrievals via `GET /v1/decision-explanations/{audit_event_id}` are audit-logged.
- **Required user / professional disclosure.** The aggregate `exclusion_summary` is surfaced to consumers; named detail is not.
- **Required conformance test.** Conformance suite verifies `exclusion_summary` matches the schema's aggregate shape (no per-candidate detail leakage); verifies named-detail access requires `governance_admin_role`.
- **Status: `resolved`** by aggregate-only `exclusion_summary` schema structure + governance-only retrieval for named detail.

### 4.7 Professional-facing endpoint is deferred and the obligation becomes invisible

- **Scenario.** An implementation marks `GET /v1/professionals/me/visibility-explanations` as `x-implementation-deferred-to: v0.8.3` and treats the obligation itself as deferred; the professional-facing transparency commitment is not honoured at v0.8.2 launch.
- **Abuse / failure mode.** "Deferred endpoint implementation" is misread as "deferred obligation"; v0.8.2 ships without the substance of the F-6 commitment.
- **Expected protocol behaviour.** Per §11A.4 / F-6 spec discipline, the v0.8.2 spec defines the required behaviour, fields, authority controls, audit linkage, and the F-18 MUST NOT rule. The deferral covers the dedicated endpoint implementation; it does NOT cover the obligation itself. Implementations MAY return 501 in v0.8.2 — but the spec is complete enough that an implementer can build from v0.8.2 alone. The CHANGELOG and ROADMAP record the deferral honestly.
- **Required audit events.** None specific (the structural verification is via spec-completeness review).
- **Required user / professional disclosure.** The CHANGELOG entry and the ROADMAP "v0.8.3 candidates" section make the deferral visible.
- **Required conformance test.** Conformance suite review verifies that the v0.8.2 spec section §11A.4 contains all five required components (behaviour, fields, authority controls, audit linkage, F-18 MUST NOT). The endpoint implementation deferral is a registry-side decision verified through binding-conformance review.
- **Status: `bounded-by-protocol`** — F-6 spec discipline keeps the obligation visible at the spec level even when implementation defers. The protocol bounds the deferral risk; implementations choosing to defer are explicit about it.

### 4.8 Professional-facing endpoint exposes actual query data, leaking consumer/platform information

- **Scenario.** An implementation builds the visibility-explanation endpoint by aggregating actual query traffic ("here's how many queries this category received last month and what fraction included you").
- **Abuse / failure mode.** Real query patterns expose information about consumers, AI platform integrations, demand signals, matter types, urgency, and platform activity that does not belong in a per-professional report. Even aggregated counts can leak in low-volume categories.
- **Expected protocol behaviour.** Per F-18 verbatim MUST NOT in §11A.4: professional-facing visibility explanations MUST NOT expose actual query-count data, query-history data, observed demand patterns, or any per-query information derived from consumer or platform traffic. The view MUST be rules-derived from the implementation's documented rules and the professional's own profile data.
- **Required audit events.** Implementations producing query-history-derived visibility views fail conformance review.
- **Required user / professional disclosure.** N/A — the failure mode is implementation-side and the spec rule is the verification surface.
- **Required conformance test.** Conformance suite reviews the implementation's visibility-explanation response payloads for any field derived from query traffic; the absence of query-history fields is the structural test. The reviewer also examines the implementation's documentation and source where available.
- **Status: `resolved`** by F-18 verbatim MUST NOT in spec §11A.4 + cross-references in CHANGELOG, EXPLAINER, PRD, and `docs/professional/visibility-report.md`.

### 4.9 Professional-facing endpoint exposes inferred query patterns via low-volume category leakage

- **Scenario.** An implementation provides aggregated query counts at category level only, not per-professional, but the category is low-volume enough that "lawyers in Wyoming, county court matters" effectively names individual professionals.
- **Abuse / failure mode.** Low-volume aggregation leaks demand information for narrow categories or jurisdictions, indirectly identifying professionals.
- **Expected protocol behaviour.** The rules-derived approach (per F-18) is structurally immune to this failure mode: the explanation is derived from documented rules and the professional's own profile, not from observed query traffic. There are no aggregate counts in the visibility-explanation response (the schema does not include any field that could carry them); low-volume leakage is impossible by construction.
- **Required audit events.** N/A.
- **Required user / professional disclosure.** N/A.
- **Required conformance test.** Conformance suite verifies the visibility-explanation response schema does not include query-count fields; the field absence is the structural test.
- **Status: `resolved`** by F-18 rules-derived approach + structural absence of query-count fields in the response schema.

---

## Category 5 — Vault, Erasure, and Audit Abuse

**Phase mapping:** Phase 3.

### 5.1 Payload is erased but abuse investigation is needed

- **Scenario.** A handoff payload is erased per protocol's erasure rules; subsequently a complaint, dispute, or regulator request requires reconstruction or evidence of what was sent.
- **Abuse / failure mode.** Tension between privacy (erasure) and accountability (investigation). Three-concept separation must keep audit-event metadata viable after payload erasure.
- **Expected protocol behaviour.** Per §11.8, crypto-erasure applies to vault payload content, not to audit metadata. The audit log retains tamper-evident records sufficient to investigate abuse, disputes, authentication failures, and consent-integrity claims, without retaining consumer personal data in plaintext. The retain-field list (§11.8) covers `event_id`, `handoff_id`, principal-delegation context, `request_intent`, `stage`, `payload_type` (not content), `consent_receipt_id`, `receipt_hash`, timestamps, retrieval actor, auth tier, abuse flags, HMAC-form contact and device references.
- **Required audit events.** All lifecycle events for the handoff: `introduction_initiated`, `stage_advanced`, `consent_asserted`, `data_retrieved`, `data_erased`, `key_destroyed`, plus any `consent_revoked` or withdrawal events that occurred. Each carries the principal-delegation context per §4.9A.
- **Required user / professional disclosure.** ATAH admin and the subject of audit (professional, partner, verifier) have access to entries concerning them on request, per §13.5.
- **Required conformance test.** Conformance suite simulates a handoff lifecycle through completion + payload erasure, then queries the audit log and verifies that the retain-field list is populated for each lifecycle event without any consumer personal data in plaintext.
- **Status: `resolved`** by §11.8 audit retention rules and the retain-field list on `audit-event.schema.json`.

### 5.2 Plaintext appears in logs or notification providers

- **Scenario.** Despite vault and crypto-erasure architecture, plaintext (consumer query content, professional contact details, decision rationale) leaks into application logs, notification provider records, or third-party telemetry.
- **Abuse / failure mode.** Privacy posture undermined at the operational margin; secondary persistence stores defeat the primary erasure design.
- **Expected protocol behaviour.** Per §11.6 implementation requirements, conforming implementations MUST satisfy: no plaintext in logs, queues, analytics, indexes, or notification providers; keys stored separately from ciphertext; destroyed keys excluded from recoverable backups. Per §11.8, audit-event records MUST NOT contain consumer-identifying fields in plaintext; references to contact identifiers (email, phone) MUST use HMACs with protected audit keys, not plain hashes. Per §11.2 deletion table, the personal-data-allowed column is "None" for SMS/email notifications, notification provider logs, webhook delivery queue, analytics/metrics, and dead-letter queue.
- **Required audit events.** If plaintext is detected in a store where it is not allowed, the entry is automatically purged and a critical audit event is logged (per §11.2).
- **Required user / professional disclosure.** Critical-severity alerting per §13.5; security event recorded.
- **Required conformance test.** Conformance suite includes a "no plaintext" test against logs, queues, analytics indices, and notification provider snapshots after a complete handoff lifecycle. Plain-hashed email or phone in the audit log MUST cause a test failure.
- **Status: `bounded-by-protocol`** — the protocol's normative requirements close the failure mode at the spec level. ATAH cannot fully prevent operational deployments from leaking plaintext into infrastructure outside its direct control (e.g. a misconfigured notification provider that logs full message bodies); the protocol's role is to specify the requirement and reject non-conformance via the conformance test suite. Operational-side enforcement is a registry-implementation responsibility.

### 5.3 Key deletion is not auditable

- **Scenario.** Crypto-erasure relies on key destruction; deletion of the key itself is performed but produces no auditable event, so erasure cannot be verified after the fact.
- **Abuse / failure mode.** Erasure is a claim, not a verifiable fact; conformance and dispute resolution cannot rely on it.
- **Expected protocol behaviour.** Per §11.6, "auditable key-destruction events" is one of the six normative implementation requirements. The `audit-event.schema.json` `event_type` enum includes `key_destroyed`; key-destruction events carry the destroyed key identifier, the linked vault-payload event id, and the destruction method.
- **Required audit events.** `key_destroyed` event for each DEK destruction; linked to the corresponding `data_erased` event by the `linked_event_id` field in `metadata`.
- **Required user / professional disclosure.** ATAH admin and the subject of audit have access on request per §13.5.
- **Required conformance test.** Conformance suite verifies that every `data_erased` event has a matching `key_destroyed` event with a synchronous-or-near-synchronous timestamp and a populated `linked_event_id`.
- **Status: `resolved`** by §11.6 implementation requirement (auditable key-destruction events) and the `key_destroyed` `event_type` on the audit-event schema.

### 5.4 Retrieval token is reused or leaked

- **Scenario.** A retrieval token issued for single-use access to a payload is reused, replayed by an attacker, or leaks into a system where it is consumed by an unintended party.
- **Abuse / failure mode.** Retrieval becomes effectively multi-use; downstream parties access content without distinct authority.
- **Expected protocol behaviour.** Per §11.6 implementation requirements, retrieval tokens are short-TTL and single-use. The `audit-event.schema.json` records `retrieved_at`, `retrieval_actor`, and `auth_tier` for every retrieval; reuse is detected at the vault layer (single-use enforcement) and at the audit layer (anomaly: same token id appearing twice).
- **Required audit events.** `data_retrieved` event with `retrieval_actor`, `auth_tier`, and the principal-delegation context; on reuse attempt, a `security_event` with the failure reason.
- **Required user / professional disclosure.** Retrieval failure surfaces to the asserter platform and to the affected professional through the existing notification channels.
- **Required conformance test.** Conformance suite verifies that a second retrieval using the same token returns an authentication failure and produces a `security_event` audit record.
- **Status: `resolved`** by §11.6 single-use retrieval-token requirement and the `data_retrieved` / `security_event` audit-event recording.

---

## Category 6 — Withdrawal Abuse

**Phase mapping:** Phase 3.

### 6.1 Malicious actor initiates and withdraws repeated handoffs

- **Scenario.** An attacker or harasser initiates handoffs to a target professional and withdraws each one before completion, exploiting the withdrawal mechanism for harassment, denial-of-service, or reputational signal noise.
- **Abuse / failure mode.** Withdrawal as harassment vector; rate-limit gaps and lack of immutable metadata enable repeat abuse.
- **Expected protocol behaviour.** Per §11.9, withdrawal is a state transition and does NOT erase audit records. Every initiate-then-withdraw cycle generates a structured audit event with the principal-delegation context, `previous_state`, `resulting_state`, and `reason_code`. Pattern detection (repeat withdrawals from the same `authenticated_actor`) is therefore visible in the audit stream. Per §13.3, rate limits on per-period proposal/initiation actions apply (concrete numbers populated by registry implementations; spec mandates the requirement).
- **Required audit events.** `introduction_initiated` and `withdrawal_recorded` (or the equivalent state-transition event_type) for each cycle, all carrying the same `authenticated_actor.actor_id` and `client_application` for pattern detection.
- **Required user / professional disclosure.** The target professional MAY surface withdrawal patterns through their professional portal; harassment-monitoring is a v0.8.3 deferred mechanism (per scenario 1.6 deferral).
- **Required conformance test.** Conformance suite verifies that withdrawal generates an audit event with the F1.17 fields and that rate-limit enforcement returns `429` after threshold.
- **Status: `partially-resolved`** — Phase 3 closes the structural side via withdrawal-as-state-transition + audit retention. Concrete rate-limit thresholds and harassment-monitoring (dense-cluster pattern detection) are `deferred-to-v0.8.3`.

### 6.2 Professional withdraws profile after concern flags

- **Scenario.** A professional withdraws their ATAH profile (or a critical part of it) immediately after concern flags are raised, attempting to evade ongoing scrutiny.
- **Abuse / failure mode.** Withdrawal as evidence escape; without immutable audit metadata of the flags and the withdrawal, future re-registration could erase prior signals.
- **Expected protocol behaviour.** Per §11.9 normative rule: withdrawal stops future processing but does NOT erase audit records, consent receipt metadata, state history, abuse signals, dispute records, or legally required retention records. Per §11.9 scenario 5: professional withdrawal from matching retains the record's required audit trail, dispute records, regulatory metadata, and existing concern flags. Per §13.2A: the withdrawal is high-impact and requires step-up authentication at the point of action. On future re-registration, the audit and concern-flag history is still present and discoverable through the dispute / admin surfaces.
- **Required audit events.** `professional_withdrew_from_matching` with previous_state, resulting_state (`withdrawn`), reason_code, principal-delegation context, and `auth_tier` reflecting step-up. Existing `concern_flag_raised` events are unaffected.
- **Required user / professional disclosure.** ATAH admin retains visibility on the audit trail; the professional themselves can view their own concern-flag history through the existing professional portal endpoints.
- **Required conformance test.** Conformance suite simulates concern-flag-then-withdrawal-then-re-registration; verifies that prior concern-flag history is still discoverable post-re-registration.
- **Status: `resolved`** by §11.9 withdrawal scenarios 5 (audit and dispute records preserved) + §13.2A step-up authentication requirement.

### 6.3 Consumer withdraws after data retrieval and expects data to be unseen

- **Scenario.** Consumer withdraws after the professional has already retrieved the handoff payload and expects this to render the data "unseen" or unusable.
- **Abuse / failure mode.** Mismatch between the consumer's mental model of withdrawal and what protocol withdrawal can actually achieve once data has been delivered to a third party's own systems.
- **Expected protocol behaviour.** Per §11.9 scenario 3: erase any remaining vault payload immediately and stop ATAH processing; **acknowledge that ATAH cannot force deletion from the professional's systems** — that obligation rests with the professional under their own privacy framework. The handoff's `withdrawal_state` becomes `consumer_withdrew_post_stage3_retrieval`. The audit event records the transition; the consent receipt's `revocation_status` flips to `revoked`.
- **Required audit events.** `consent_revoked` and `data_erased` for any remaining vault payload; `withdrawal_recorded` with previous_state, resulting_state, reason_code.
- **Required user / professional disclosure.** Consumer-facing AI platforms MUST surface, at the consent-capture stage, that ATAH cannot retract data already delivered to professionals. The professional receives a notification that consent has been revoked; their own data-handling obligations apply under their privacy framework.
- **Required conformance test.** Conformance suite simulates Stage 3 retrieval followed by consumer revocation; verifies vault crypto-erasure of any remaining payload and the audit-event sequence.
- **Status: `resolved`** by §11.9 scenario 3 (with the explicit acknowledgement that ATAH cannot force deletion from the professional's systems — the protocol is honest about its limits).

### 6.4 Delegated agent withdraws a professional from matching after account compromise

- **Scenario.** A delegated agent token, compromised by an attacker, is used to withdraw the principal professional from matching (or modify their profile to render them ineligible).
- **Abuse / failure mode.** Compromise of one credential class causes business damage to the principal; recovery semantics unclear.
- **Expected protocol behaviour.** Per §13.2A: professional withdrawal from matching is a high-impact action requiring step-up authentication at the point of action. A compromised delegated-agent token alone is therefore insufficient — the step-up authentication factor (re-auth via OAuth, second factor via TOTP, out-of-band push, or equivalent) must also be presented. Per §11.9: revocation of a professional delegated-agent token is itself a high-impact action requiring step-up; the principal can recover by revoking the compromised token and reissuing.
- **Required audit events.** Failed step-up authentication produces a `security_event`; successful withdrawal produces `professional_withdrew_from_matching` with `auth_tier` recording the step-up method.
- **Required user / professional disclosure.** Failed step-up attempts on a professional's account MAY surface to the principal through their professional portal; out-of-band notification (email or SMS to the partner-known channel) is a registry-implementation pattern.
- **Required conformance test.** Conformance suite verifies that an attempt to call `withdraw_from_matching` (or POST `/v1/professionals/me/withdraw`) without a valid `step_up_token` returns `403` with the appropriate error code.
- **Status: `resolved`** by §13.2A step-up authentication requirement on high-impact withdrawals.

### 6.5 Component 3 connection records persist after professional withdrawal from matching

- **Scenario.** Professional A withdraws their record from matching (or their `matching_status` flips to `withdrawn`). Active Component 3 referral connections involving A continue to exist as `referral-connection` records — they were not what A's withdrawal was supposed to end.
- **Abuse / failure mode.** Mismatch between A's mental model of withdrawal ("I am no longer in ATAH") and what protocol withdrawal removes (matching pool, not Component 3 connections); audit interpretation may be ambiguous.
- **Expected protocol behaviour.** _To be filled in during Phase 3 (withdrawal as state transition)._ Phase 3's withdrawal mechanics will need to specify whether a `withdrawn` matching_status implicitly withdraws Component 3 connections or whether explicit `POST /v1/referral-connections/:connection_id/withdraw` is required for each.
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Phase mapping.** Phase 2 introduces Component 3 connection records; Phase 3 specifies how withdrawal-from-matching interacts with them.
- **Status: `skeleton`** — discovered during Phase 2; resolution falls to Phase 3.

---

## Category 7 — Contact Freshness and Engagement Liveness

**Phase mapping:** Phase 7 (channel-validity verification). Engagement liveness (response-rate tracking) is **deferred to v0.8.3** — covered by Phase 7's documented deferral.

### 7.1 Licensed professional has stale email or phone

- **Scenario.** A verified, eligible professional's recorded contact channels (email, phone, notification endpoint) point to addresses that are no longer monitored or owned.
- **Abuse / failure mode.** Notifications go unreceived; consumer expects a live route, professional misses opportunity, ATAH appears to recommend a non-reachable contact.
- **Expected protocol behaviour.** _To be filled in during Phase 7._
- **Required audit events.** _To be filled in during Phase 7._
- **Required user / professional disclosure.** _To be filled in during Phase 7._
- **Required conformance test.** _To be filled in during Phase 7._
- **Status.** `skeleton`

### 7.2 Notification goes to an abandoned channel

- **Scenario.** A handoff notification is dispatched to a channel whose underlying account / address has been abandoned by the recipient; delivery succeeds at the transport layer but no human ever sees it.
- **Abuse / failure mode.** Silent delivery failure; no surfacing of the staleness condition until the consumer's wait time has elapsed.
- **Expected protocol behaviour.** _To be filled in during Phase 7._
- **Required audit events.** _To be filled in during Phase 7._
- **Required user / professional disclosure.** _To be filled in during Phase 7._
- **Required conformance test.** _To be filled in during Phase 7._
- **Status.** `skeleton`

### 7.3 Professional repeatedly misses Stage 1 response commitments

- **Scenario.** A professional appears in eligible result sets but consistently fails to respond within their declared response window at Stage 1.
- **Abuse / failure mode.** Engagement-liveness signal absent; professional remains in eligible set despite a real-world unresponsive pattern.
- **Expected protocol behaviour.** *Engagement liveness is **deferred to v0.8.3**.* Phase 7 covers channel-validity verification only; response-rate tracking is a separate mechanism not in scope for v0.8.2.
- **Required audit events.** _Deferred to v0.8.3._
- **Required user / professional disclosure.** _Deferred to v0.8.3._
- **Required conformance test.** _Deferred to v0.8.3._
- **Status.** `deferred` *(`deferred-to-v0.8.3` — engagement liveness mechanism out of v0.8.2 scope; v0.8.2 covers contact-channel verification only.)*

### 7.4 Professional appears active but is not reachable

- **Scenario.** Profile metadata (last activity, recent updates) suggests the professional is active, but the verified contact channels are not currently functioning (mailbox full, number disconnected, integration broken).
- **Abuse / failure mode.** Profile-level "active" signal diverges from channel-level "reachable" signal; the latter is what the consumer experiences.
- **Expected protocol behaviour.** _To be filled in during Phase 7._
- **Required audit events.** _To be filled in during Phase 7._
- **Required user / professional disclosure.** _To be filled in during Phase 7._
- **Required conformance test.** _To be filled in during Phase 7._
- **Status.** `skeleton`

---

## Category 8 — Partner and Governance Capture

**Phase mapping:** Phase 8.

### 8.1 Paying partner seeks broader verification scope than justified

- **Scenario.** A trusted partner who pays for, or sponsors, the partnership relationship requests an expansion of their authoritative-data scope beyond what their underlying authority justifies.
- **Abuse / failure mode.** Verification-scope creep; partner's authority over a narrow domain (e.g. licence data) leveraged into broader implicit endorsement.
- **Expected protocol behaviour.** _To be filled in during Phase 8._
- **Required audit events.** _To be filled in during Phase 8._
- **Required user / professional disclosure.** _To be filled in during Phase 8._
- **Required conformance test.** _To be filled in during Phase 8._
- **Status.** `skeleton`

### 8.2 Review platform with weak anti-gaming wants admission

- **Scenario.** A review platform with insufficient anti-gaming controls applies for admission as a partner data source.
- **Abuse / failure mode.** Low-quality review signal entering ATAH band assignment; gameable signals influencing professional appearance.
- **Expected protocol behaviour.** _To be filled in during Phase 8._
- **Required audit events.** _To be filled in during Phase 8._
- **Required user / professional disclosure.** _To be filled in during Phase 8._
- **Required conformance test.** _To be filled in during Phase 8._
- **Status.** `skeleton`

### 8.3 Verifier sells enhanced verification as a purchasable badge

- **Scenario.** An independent verifier offers professionals a paid "enhanced verification" badge whose substantive verification rigour is not commensurate with the badge's prominence.
- **Abuse / failure mode.** Verification becomes a marketing channel; signal devaluation across the ecosystem.
- **Expected protocol behaviour.** _To be filled in during Phase 8._
- **Required audit events.** _To be filled in during Phase 8._
- **Required user / professional disclosure.** _To be filled in during Phase 8._
- **Required conformance test.** _To be filled in during Phase 8._
- **Status.** `skeleton`

### 8.4 Founder-affiliated or commercially connected entity seeks preferential treatment

- **Scenario.** An entity related to ATAH's founders, governance body, or significant commercial counterparties seeks preferential admission, scope, or visibility.
- **Abuse / failure mode.** Governance capture; perceived neutrality of the protocol layer compromised even if the technical effect is small.
- **Expected protocol behaviour.** _To be filled in during Phase 8._
- **Required audit events.** _To be filled in during Phase 8._
- **Required user / professional disclosure.** _To be filled in during Phase 8._
- **Required conformance test.** _To be filled in during Phase 8._
- **Status.** `skeleton`

---

## Category 9 — Category and Jurisdiction Misfit

**Phase mapping:** existing v0.8.1 work + Phase 8 (category annex template cross-reference).

### 9.1 Legal conflict rules vary by jurisdiction

- **Scenario.** Conflict-of-interest rules for legal practitioners differ materially across jurisdictions; a one-size implementation either over-blocks or under-detects conflicts.
- **Abuse / failure mode.** Jurisdiction mismatch; conflict rules from one country mis-applied in another.
- **Expected protocol behaviour.** _To be filled in via existing v0.8.1 category annex work referenced from Phase 8._
- **Required audit events.** _To be filled in via Phase 8._
- **Required user / professional disclosure.** _To be filled in via Phase 8._
- **Required conformance test.** _To be filled in via Phase 8._
- **Status.** `skeleton`

### 9.2 Healthcare data requires stronger compliance controls

- **Scenario.** Healthcare-category queries and handoffs carry data subject to compliance regimes (HIPAA, GDPR special categories, equivalent national frameworks) stricter than ATAH's default posture.
- **Abuse / failure mode.** Default protocol controls insufficient for the category; conformance achieved without compliance.
- **Expected protocol behaviour.** _To be filled in via Phase 8 (category annex template) referencing existing v0.8.1 work._
- **Required audit events.** _To be filled in via Phase 8._
- **Required user / professional disclosure.** _To be filled in via Phase 8._
- **Required conformance test.** _To be filled in via Phase 8._
- **Status.** `skeleton`

### 9.3 Financial-advice terminology differs by country

- **Scenario.** "Financial adviser", "investment professional", "wealth manager" carry different licensure, scope, and consumer-protection meanings across jurisdictions.
- **Abuse / failure mode.** Cross-jurisdiction term confusion; consumers misled about scope.
- **Expected protocol behaviour.** _To be filled in via Phase 8 / existing category annex work._
- **Required audit events.** _To be filled in via Phase 8._
- **Required user / professional disclosure.** _To be filled in via Phase 8._
- **Required conformance test.** _To be filled in via Phase 8._
- **Status.** `skeleton`

### 9.4 Established-practitioner categories lack authoritative regulators

- **Scenario.** Categories where no single authoritative regulator exists (some coaching, advisory, or therapeutic professions) require verification approaches different from credentialled categories.
- **Abuse / failure mode.** Verification model designed for credentialled categories applied weakly or inappropriately to established-practitioner categories.
- **Expected protocol behaviour.** _To be filled in via Phase 8 / existing category annex work._
- **Required audit events.** _To be filled in via Phase 8._
- **Required user / professional disclosure.** _To be filled in via Phase 8._
- **Required conformance test.** _To be filled in via Phase 8._
- **Status.** `skeleton`

---

## Category 10 — Conformance Divergence

**Phase mapping:** Phase 8 + Phase 6 (transparency conformance class).

### 10.1 Third-party registry claims ATAH compatibility but omits transparency metadata

- **Scenario.** A third-party registry implements ATAH-compatible endpoints but omits or weakens transparency artifacts (decision-explanation, presentation-disclosure, visibility report).
- **Abuse / failure mode.** "ATAH compatible" claim made without the transparency posture that gives the protocol its accountability shape.
- **Expected protocol behaviour.** _To be filled in during Phase 8 (governance) and Phase 6 (transparency-as-conformance class)._
- **Required audit events.** _To be filled in during Phase 8 / Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 8 / Phase 6._
- **Required conformance test.** _To be filled in during Phase 8 / Phase 6._
- **Status.** `skeleton`

### 10.2 Registry applies commercial ordering while preserving provenance

- **Scenario.** A third-party registry preserves provenance metadata correctly but applies commercial ordering, treating the protocol's stratified-randomisation rule as advisory rather than normative.
- **Abuse / failure mode.** Conformance gap exploited; ordering framing weakened across implementations.
- **Expected protocol behaviour.** _To be filled in during Phase 8 / Phase 6._
- **Required audit events.** _To be filled in during Phase 8 / Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 8 / Phase 6._
- **Required conformance test.** _To be filled in during Phase 8 / Phase 6._
- **Status.** `skeleton`

### 10.3 Implementation changes consent or withdrawal semantics

- **Scenario.** A conforming-by-name implementation alters consent-receipt scope semantics or withdrawal state-transition rules, producing user-visible behaviour that diverges from ATAH's documented semantics.
- **Abuse / failure mode.** Same protocol surface, different meanings; users and reviewers cannot rely on documented semantics across implementations.
- **Expected protocol behaviour.** _To be filled in during Phase 8 / Phase 6._
- **Required audit events.** _To be filled in during Phase 8 / Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 8 / Phase 6._
- **Required conformance test.** _To be filled in during Phase 8 / Phase 6._
- **Status.** `skeleton`

### 10.4 Federation introduces inconsistent identity or audit handling

- **Scenario.** Federated implementations exchange handoffs across instances; identity resolution, audit-event recording, or principal-delegation handling diverges between participating implementations.
- **Abuse / failure mode.** Cross-instance audit gaps; handoffs that look conforming on each side but produce broken end-to-end accountability.
- **Expected protocol behaviour.** _To be filled in during Phase 8 / Phase 6._
- **Required audit events.** _To be filled in during Phase 8 / Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 8 / Phase 6._
- **Required conformance test.** _To be filled in during Phase 8 / Phase 6._
- **Status.** `skeleton`

---

## Phase 0A summary

- **Categories defined:** 10.
- **Seed scenarios populated:** 41 (5 / 4 / 4 / 4 / 4 / 4 / 4 / 4 / 4 / 4).
- **Status distribution at end of Phase 0A:** 40 × `skeleton`, 1 × `deferred` (scenario 7.3, `deferred-to-v0.8.3` per Phase 7's documented engagement-liveness deferral).
- **Phase mapping coverage:** every scenario carries a phase mapping. Phase 1 → Cat 1; Phase 4 → Cat 2; Phase 5 → Cat 3; Phase 6 → Cat 4; Phase 3 → Cats 5 and 6; Phase 7 → Cat 7 (with v0.8.3 deferral); Phase 8 → Cat 8; existing v0.8.1 work + Phase 8 → Cat 9; Phase 8 + Phase 6 → Cat 10.

## Phase 1 update

Category 1 (Authentication and Delegation Abuse) scenarios marked per Phase 1's contribution.

- **Status updates:**
  - 1.1 (`request_intent` falsely declared) → `partially-resolved` (becomes `resolved` after Phase 2 server-side `request_intent` validation lands).
  - 1.2 (platform submits consent the user did not give) → `partially-resolved` here (Phase 1 records the asserter shape via `principal-delegation`); fully addressed by Phase 4's continuity binding (`client_id`, `pseudonymous_consumer_ref`, `data_categories_hash`); per F-17, eventual settled status is `bounded-by-protocol` — ATAH verifies receipt integrity, not the validity of the underlying consent ceremony.
  - 1.3 (delegated-agent token stolen) → `resolved` by `authority_basis: professional_delegated_token` with mandatory `expires_at` and `object_constraints`. Operational hardening (rotation cadence, anomaly detection thresholds) is registry-implementation responsibility.
  - 1.4 (firm admin acts beyond delegated authority) → `partially-resolved` here (Phase 1 supplies `firm_delegation` basis + per-professional `object_constraints`); fully resolved by Phase 3's step-up authentication for high-impact actions.
  - 1.5 (cross-platform / cross-session handoff replay) → `partially-resolved` here (Phase 1 records the asserter `client_application` and the §7.3 original-asserter match); fully resolved by Phase 4's continuity-binding fields on the stored consent receipt.
- **New scenarios discovered during Phase 1:** none.
- **Status distribution after Phase 1:** 35 × `skeleton`, 1 × `resolved` (1.3), 4 × `partially-resolved` (1.1, 1.2, 1.4, 1.5), 1 × `deferred` (7.3, unchanged).

## Phase 2 update

Architectural simplification (GKC-COMMENTS-01) ships in Phase 2. Status changes:

- **1.1 (`request_intent` falsely declared)** → **`resolved`**. Phase 2 makes `request_intent` a required parameter on the Discovery query, gates Component 2 and Component 3 endpoints on the originating intent, and persists the principal-delegation context recording the declared intent in every audit-event. Mismatched intent is rejected at endpoint level.
- **3.2 (hidden score influences order despite non-recommendation claims)** → **`partially-resolved`**. Phase 2 removes the `inbound_referral_signal` matching component and example output (one signal previously available for hidden weighting is gone). The four-component weighted model that remains is still weighted-score architecture; Phase 5 supersedes it with stratified randomisation, completing resolution.
- **3.1, 3.3, 3.4** unchanged — Phase 5 territory.
- **Cat 2, Cat 4, Cats 5–7, Cat 8, Cat 9, Cat 10** unchanged in Phase 2 — their resolutions live in later phases.

New scenarios discovered during Phase 2:

- **1.6 — Component 3 proposal spam via looking-toggle gaming.** Status `partially-resolved` (Phase 2 closes structural side via toggle gating + rate limits + audit recording; dense-cluster pattern detection `deferred-to-v0.8.3`).
- **6.5 — Component 3 connection records persist after professional withdrawal from matching.** Status `skeleton` — discovered in Phase 2 but resolution falls to Phase 3's withdrawal-as-state-transition work.

**Status distribution after Phase 2:** 35 × `skeleton`, 2 × `resolved` (1.1, 1.3), 5 × `partially-resolved` (1.2, 1.4, 1.5, 1.6, 3.2), 1 × `deferred` (7.3). New total: 43 scenarios (41 seed + 2 discovered in Phase 2).

## Phase 3 update

Three-concept separation of payload erasure / audit retention / withdrawal-as-state-transition (Paolo's F1.14–F1.17) ships in Phase 3. Status changes:

- **Category 5 (Vault, Erasure, and Audit Abuse):**
  - 5.1 (payload erased but abuse investigation needed) → **`resolved`** by §11.8 audit retention rules and the retain-field list on `audit-event.schema.json`.
  - 5.2 (plaintext appears in logs or notification providers) → **`bounded-by-protocol`** — the protocol's normative requirements (§11.6 implementation requirements + §11.8 HMAC requirement) close the failure mode at the spec level; ATAH cannot fully prevent operational deployments from leaking plaintext into infrastructure outside its direct control. Operational-side enforcement is a registry-implementation responsibility.
  - 5.3 (key deletion not auditable) → **`resolved`** by §11.6 normative requirement (auditable key-destruction events) and the `key_destroyed` `event_type` on the audit-event schema.
  - 5.4 (retrieval token reused or leaked) → **`resolved`** by §11.6 single-use retrieval-token requirement and the `data_retrieved` / `security_event` audit recording.
- **Category 6 (Withdrawal Abuse):**
  - 6.1 (malicious actor initiates and withdraws repeated handoffs) → **`partially-resolved`**. Phase 3 closes the structural side via withdrawal-as-state-transition + audit retention. Concrete rate-limit thresholds and harassment-monitoring (dense-cluster pattern detection) `deferred-to-v0.8.3`.
  - 6.2 (professional withdraws profile after concern flags) → **`resolved`** by §11.9 scenario 5 (audit and dispute records preserved) + §13.2A step-up authentication requirement.
  - 6.3 (consumer withdraws after data retrieval and expects data to be unseen) → **`resolved`** by §11.9 scenario 3, with the explicit acknowledgement that ATAH cannot force deletion from the professional's systems — the protocol is honest about its limits.
  - 6.4 (delegated agent withdraws after account compromise) → **`resolved`** by §13.2A step-up authentication requirement on high-impact withdrawals.
  - 6.5 (Component 3 connection records persist after professional withdrawal from matching, discovered in Phase 2) → **`resolved`** by §11.9 scenario 5's explicit framing: withdrawal-from-matching does NOT imply withdrawal-from-Component-3-connections; either can be performed independently. The interaction is documented and intentional.

New scenarios discovered during Phase 3: none.

**Status distribution after Phase 3:** 26 × `skeleton`, 9 × `resolved` (1.1, 1.3, 5.1, 5.3, 5.4, 6.2, 6.3, 6.4, 6.5), 1 × `bounded-by-protocol` (5.2), 6 × `partially-resolved` (1.2, 1.4, 1.5, 1.6, 3.2, 6.1), 1 × `deferred` (7.3). Total: 43 scenarios.

## Phase 4 update

Consent boundaries (Paolo's F1.3 / F1.4 / F1.5 + F-3 continuity binding + F-4 Component 3 authority) ship in Phase 4. Status changes:

- **F-17 correction to 1.2 (platform submits consent for a user who did not consent)** → settled at **`bounded-by-protocol`** (was `partially-resolved` after Phase 1; the eventual settled status was flagged in Phase 1's notes). Per the receipt-hash limitation framing: ATAH detects tampering, replay, and continuity mismatch via receipt hash + F-3 continuity binding (`client_id`, `pseudonymous_consumer_ref`, `data_categories_hash`); ATAH cannot prove the platform's underlying consent ceremony was valid.
- **2.1 (handoff consent misrepresented as engagement consent)** → **`resolved`** by F1.4 normative rule (verbatim across spec / Charter / EXPLAINER / PRD), the `consent_type` enum restriction, and the canonical disclosure-language requirement.
- **2.2 (user consents to one professional but data sent to another)** → **`resolved`** by F-3 continuity binding (`professional_id` binding + `data_categories_hash`).
- **2.3 (consent text differs from stored receipt metadata)** → **`bounded-by-protocol`** — receipt integrity verified by hash; ATAH cannot prove what the consumer actually saw on the platform's UI per the receipt-hash limitation framing.
- **2.4 (consent is revoked after Stage 2 or Stage 3)** → **`resolved`** by §4.10 `revocation_status` checks at every consumption point + Phase 3's §11.9 scenario 3 (consumer-withdrawal-after-Stage-3-retrieval semantics).

New scenarios discovered during Phase 4 work and added to Cat 2:

- **2.5 — Receipt replay across users (cross-user confusion).** **`resolved`** by F-3 `pseudonymous_consumer_ref` HMAC binding.
- **2.6 — Receipt replay across sessions (same user, different action).** **`resolved`** by F-3 `scope` and `data_categories_hash` binding + receipt expiry.
- **2.7 — Component 3 endpoint accidentally accepts a consumer consent receipt.** **`resolved`** by F-4: `consent_receipt_id` is structurally absent from Component 3 endpoints/tools; the scope enum excludes the v0.8.1 referral values.

**Status distribution after Phase 4:** 22 × `skeleton`, 15 × `resolved` (1.1, 1.3, 2.1, 2.2, 2.4, 2.5, 2.6, 2.7, 5.1, 5.3, 5.4, 6.2, 6.3, 6.4, 6.5), 3 × `bounded-by-protocol` (1.2, 2.3, 5.2), 5 × `partially-resolved` (1.4, 1.5, 1.6, 3.2, 6.1), 1 × `deferred` (7.3). Total: **46 scenarios** (43 prior + 3 new in Phase 4 — 2.5, 2.6, 2.7).

## Phase 5 update

Stratified randomisation, no global match_score, review-signal band cap (Paolo's F1.10 / F1.11 / F1.12 / F1.13 + F-7 + F-13 + F-14) ship in Phase 5. Status changes:

- **3.1 (candidates always returned in same order)** → **`resolved`** by §9.1 stratified-randomisation default + §9.5 conformance requirement on observable non-determinism.
- **3.2 (hidden score influences order)** → promoted from `partially-resolved` to **`resolved`** by Phase 5's full removal of the weighted-score architecture from `match-response.schema.json` (no `match_score` / `match_factors` / `ranking_basis` field exists at any layer).
- **3.3 (commercial partner candidates appear disproportionately)** → **`resolved`** by §9.6 commercial-neutrality MUST + §9.2 review-signal band cap + Phase 6 per-candidate `decision_explanation` transparency (Phase 5 sets up the structural surface; Phase 6 makes it required).
- **3.4 (AI platform presents result one as "best")** → **`allocated-to-platform-responsibility`** per F-17. ATAH publishes the `presentation_disclosure.ordering_policy` disclosure and the §9.4 MUST; downstream presentation is the platform's obligation.

New scenarios discovered during Phase 5 (entries 3.5, 3.6 added in Cat 3 above):

- **3.5 — Review signals quietly upgrade a candidate's band in a regulated category** → **`resolved`** by §9.2 verbatim MUST rule + `review_signal_band_cap.may_upgrade_band: false` for regulated categories.
- **3.6 — Implementation chooses a non-uniform random algorithm that systematically favours certain candidates** → **`partially-resolved`** by §9.5 (model + non-determinism conformance test); full statistical-distribution conformance lands in Phase 6 / Phase 11; v0.9 may standardise the algorithm.

**Status distribution after Phase 5:** 19 × `skeleton`, 19 × `resolved` (1.1, 1.3, 2.1, 2.2, 2.4, 2.5, 2.6, 2.7, 3.1, 3.2, 3.3, 3.5, 5.1, 5.3, 5.4, 6.2, 6.3, 6.4, 6.5), 3 × `bounded-by-protocol` (1.2, 2.3, 5.2), 1 × `allocated-to-platform-responsibility` (3.4), 5 × `partially-resolved` (1.4, 1.5, 1.6, 3.6, 6.1), 1 × `deferred` (7.3). Total: **48 scenarios** (46 prior + 2 new in Phase 5).

## Phase 6 update

Transparency-as-conformance and decision-explanation (Paolo's F1.8 / F1.9 + F-5 inner-object-only + F-6 two-layer + F-18 rules-derived professional-facing view) ship in Phase 6. Status changes:

- **4.1 (professional asks why excluded)** → **`resolved`** by §11A.4 verbatim normative rule + dedicated endpoint specification (implementation may defer to v0.8.3 but obligation is v0.8.2).
- **4.2 (consumer asks why a candidate appeared)** → **`resolved`** by §11A.2 two-layer model + per-candidate `decision_explanation` required on `match-response.schema.json`.
- **4.3 (auditor asks whether commercial weighting affected results)** → **`resolved`** by `decision_explanation.ordering_policy.commercial_weighting: false` (const) + `GET /v1/decision-explanations/{audit_event_id}` governance endpoint.
- **4.4 (partner data conflict suppresses, professional appeals)** → **`partially-resolved`**. Phase 6 closes the transparency side (suppression reason visible; existing dispute path visible). Detailed dispute-resolution timeline / escalation criteria remains v0.9 work (already on ROADMAP).

New scenarios discovered during Phase 6 work (added to Cat 4 above):

- **4.5 (response-level only, claims conformance)** → **`resolved`** by Transparency Class conformance + schema-level required field on `MatchResult`.
- **4.6 (exclusion summary leaks excluded professional identities)** → **`resolved`** by aggregate-only `exclusion_summary` schema structure + governance-only retrieval for named detail.
- **4.7 (endpoint deferred; obligation becomes invisible)** → **`bounded-by-protocol`**. F-6 spec discipline keeps the obligation visible at the spec level; deferring implementations are explicit about it via the `x-implementation-deferred-to` extension and the CHANGELOG / ROADMAP record.
- **4.8 (endpoint exposes actual query data)** → **`resolved`** by F-18 verbatim MUST NOT + cross-references across spec, CHANGELOG, EXPLAINER, PRD, and `docs/professional/visibility-report.md`.
- **4.9 (low-volume category leakage of inferred query patterns)** → **`resolved`** by F-18 rules-derived approach + structural absence of query-count fields in the visibility-explanation response schema.

**Status distribution after Phase 6:** 14 × `skeleton`, 26 × `resolved` (1.1, 1.3, 2.1, 2.2, 2.4, 2.5, 2.6, 2.7, 3.1, 3.2, 3.3, 3.5, 4.1, 4.2, 4.3, 4.5, 4.6, 4.8, 4.9, 5.1, 5.3, 5.4, 6.2, 6.3, 6.4, 6.5), 4 × `bounded-by-protocol` (1.2, 2.3, 4.7, 5.2), 1 × `allocated-to-platform-responsibility` (3.4), 6 × `partially-resolved` (1.4, 1.5, 1.6, 3.6, 4.4, 6.1), 1 × `deferred` (7.3). Total: **53 scenarios** (48 prior + 5 new in Phase 6).

Phase 11 finalises the matrix as the verification artifact for v0.8.2 publication.
