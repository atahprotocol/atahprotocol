# Stress-Test Matrix — ATAH Protocol v0.8.2

**Status:** Published as the verification artifact for v0.8.2.

**Purpose.** This matrix is the verification harness for v0.8.2 protocol design. It captures the abuse, mistake, commercial-conflict, and edge-case behaviours that ATAH must address — at the protocol level, not only as implementation QA.

**Scope.** Protocol-design risks: what the spec permits, what it leaves ambiguous, and what conforming implementations might do differently. Stress testing is a publication discipline, not a later implementation afterthought.

---

## Summary table

**Total scenarios: 52** across 10 categories. Final v0.8.2 status distribution:

| Status | Count | % | What it means |
|---|---:|---:|---|
| `resolved` | 33 | 63.5% | Protocol behaviour fully addresses the scenario; conforming implementations close the loop. |
| `bounded-by-protocol` | 5 | 9.6% | Protocol detects, limits, or makes verifiable; does not fully eliminate (e.g. consent-receipt integrity vs underlying-ceremony validity). |
| `allocated-to-platform-responsibility` | 1 | 1.9% | Resolution belongs to AI platform / partner / verifier under their own obligations; ATAH publishes the disclosure surface. |
| `partially-resolved` | 11 | 21.2% | Substantially addressed in v0.8.2; remaining work in a later v0.8.3 / v0.9 phase, with explicit reference. |
| `deferred` | 2 | 3.8% | Acknowledged but not addressed in v0.8.2; explicit target version. |
| `skeleton` | **0** | 0% | **All seed-scenario skeleton statuses processed.** |

**Per-category distribution:**

| Cat | Title | Total | Statuses |
|---:|---|---:|---|
| 1 | Authentication and Delegation Abuse | 4 | 2 resolved · 1 bounded-by-protocol · 1 partially-resolved |
| 2 | Consent Boundary Failure | 6 | 5 resolved · 1 bounded-by-protocol |
| 3 | Ordering and Recommendation Drift | 6 | 4 resolved · 1 allocated-to-platform-responsibility · 1 partially-resolved |
| 4 | Transparency and Explainability Failure | 9 | 7 resolved · 1 bounded-by-protocol · 1 partially-resolved |
| 5 | Vault, Erasure, and Audit Abuse | 4 | 3 resolved · 1 bounded-by-protocol |
| 6 | Withdrawal Abuse | 4 | 3 resolved · 1 partially-resolved |
| 7 | Contact Freshness and Engagement Liveness | 4 | 3 resolved · 1 deferred (7.3 → v0.8.3) |
| 8 | Partner and Governance Capture | 7 | 2 resolved · 1 bounded-by-protocol · 4 partially-resolved |
| 9 | Category and Jurisdiction Misfit | 4 | 1 resolved · 3 partially-resolved |
| 10 | Conformance Divergence | 4 | 3 resolved · 1 deferred (10.4 → v1.0) |

**Deferrals — explicit:**

- **7.3** `deferred-to-v0.8.3` — engagement-liveness mechanism (response-rate tracking, missed-Stage-1 patterns) is out of v0.8.2 scope; v0.8.2 covers contact-channel verification only.
- **10.4** `deferred-to-v1.0` — federation mechanics. v0.8.2 architecture does not preclude federation; it does not provide federation. Documented in spec §1.6 and ROADMAP "v0.9 candidates" / "v1.0 target".

**`partially-resolved` scenarios with explicit residual-work targets:**

- **1.5** Structural carrier present in v0.8.2; continuity-binding fields complete cross-platform-replay detection.
- **3.6** Stratified-randomisation model + non-determinism conformance test in v0.8.2; **algorithm standardisation → v0.9**.
- **4.4** Transparency surface + dispute path in v0.8.2; **fuller dispute-resolution timeline / escalation criteria / evidence handling → v0.9**.
- **6.1** Withdrawal-as-state-transition + audit retention in v0.8.2; **concrete rate-limit thresholds and pattern-detection harassment monitoring → v0.8.3**.
- **8.2** Review-platform schema + §9.2 review-signal control in v0.8.2; **fuller minimum-criteria specification → v0.9** (Charter Part Two hard-artifacts list).
- **8.3** Verifier schema + §9.6 commercial-neutrality framing in v0.8.2; **fuller verifier conflict and audit rules → v0.9** (Charter Part Two hard-artifacts list).
- **8.5** Three-level conformance-status framing in `CONFORMANCE.md`; **Charter-spirit conformance audit → v0.9 behavioural-neutrality / conformance-audit model**.
- **8.6** Three-level distinction + revocability commitment + right-to-contest in v0.8.2; **recognised-neutral revocation procedure → v0.9**.
- **9.1** Category-annex work + transparency obligations in v0.8.2; **per-jurisdiction legal-conflict canonicalisation → v0.9 category-annex template**.
- **9.2** `compliance_status` mechanism + §9.1 hard filter in v0.8.2; **per-jurisdiction healthcare compliance → v0.9 category-annex template**.
- **9.3** Cross-jurisdiction default + per-category metadata in v0.8.2; **per-jurisdiction financial-advice terminology → v0.9 category-annex template**.

**Verification check.** Every scenario has either a concrete resolution reference or an explicit deferral with target version. Zero scenarios use ad-hoc status words outside the published status vocabulary (`resolved`, `bounded-by-protocol`, `allocated-to-platform-responsibility`, `partially-resolved`, `deferred`).

---

## Status vocabulary

Every scenario carries one of the following statuses. The discipline: do not use `resolved` when `bounded-by-protocol` or `allocated-to-platform-responsibility` is more honest. Press-cycle reviewers will scrutinise this matrix; overclaimed `resolved` markers attract criticism.

- **`resolved`** — protocol behaviour fully addresses the scenario. The protocol's stated rules, when followed, prevent or detect the failure mode. Reserve for cases where ATAH's own behaviour closes the loop.
- **`bounded-by-protocol`** — protocol detects, limits, or makes verifiable a scenario without fully eliminating it. Example: "Platform submits consent for a user who did not consent" — ATAH verifies receipt integrity (tampering, replay, mismatch) but cannot prove the platform's underlying consent ceremony was valid. The protocol bounds the abuse without resolving it.
- **`allocated-to-platform-responsibility`** — scenario is real but its resolution belongs to the AI platform, partner, or verifier under their own obligations. ATAH documents the obligation and exposes verifiable signals; the scenario itself is closed by another party's behaviour, not ATAH's. Example: "AI platform presents ATAH result one as 'best'" — ATAH publishes ordering policy in `presentation_disclosure`; the platform's reordering and presentation is its own responsibility.
- **`partially-resolved`** — scenario is partly addressed by v0.8.2 work, partly pending further work. Use for scenarios where v0.8.2 does substantial-but-incomplete work. Should always carry a reference to the deferral version.
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
- **Status** — one of the statuses above.

---

## Category 1 — Authentication and Delegation Abuse


### 1.1 Agent falsely declares `request_intent`

- **Scenario.** A consumer-mediated AI platform issues a query to ATAH and declares an inaccurate `request_intent`, either to obtain a different result class than the user authorised or to mask the true purpose.
- **Abuse / failure mode.** Misrepresentation of authority basis; downstream protocol decisions (eligibility filters, transparency obligations, audit semantics) are taken under a false premise.
- **Expected protocol behaviour.** Every protocol action carries a `principal-delegation.schema.json` block recording `authenticated_actor`, `client_application`, and `authority_context.represented_principal_type` / `authority_basis` / `permitted_scopes`. ATAH validates the declared `authority_basis` against the credential class actually presented (e.g. `user_session` actually backed by an authorization-code-with-PKCE token); a mismatch results in `403`. The protocol introduces server-side validation of `request_intent` against the action class permitted under that authority basis, completing the loop.
- **Required audit events.** `event_type: security_event` (or the action's normal `event_type` with structured metadata indicating `intent_validation: rejected`), recording the declared `request_intent`, the authority basis presented, and the rejection reason.
- **Required user / professional disclosure.** None to consumer (an attacker should not learn the validation logic). Audit trail is accessible to the asserting platform on request, per §13.5.
- **Required conformance test.** Conformance suite includes a request with mismatched `request_intent` / `authority_basis` and asserts `403` plus an audit event matching the metadata schema.
- **Status: `resolved`** — declaration is required, downstream component access is gated server-side, and the declared intent is recorded in the audit trail.

### 1.2 Platform submits consent for a user who did not consent

- **Scenario.** An AI platform presents a consent receipt at a Stage 2 / Stage 3 boundary for a consumer who did not perform the required consent ceremony.
- **Abuse / failure mode.** Fabricated or recycled consent; downstream professionals receive data on a false consent basis.
- **Expected protocol behaviour.** Per spec §4.10, ATAH verifies the receipt's integrity (hash + continuity binding) at submission and at every consumption point. the stored receipt carries `client_id`, `pseudonymous_consumer_ref` (HMAC), and `data_categories_hash` (for `disclosure_consent`); on consumption, the consuming session's `client_id` and pseudonymous reference must match the stored values. Per spec §4.10: "ATAH verifies integrity of the submitted consent receipt. The asserting AI platform remains responsible for obtaining valid user consent to disclose data."
- **Required audit events.** `consent_receipt_submitted` records the receipt with the principal-delegation context. `consent_continuity_mismatch` is emitted on any consumption attempt whose binding does not match the stored values. `consent_revoked` flips `revocation_status` per §4.10.
- **Required user / professional disclosure.** Mismatched-binding consumption attempts surface as `consent_mismatch` errors to the consuming AI platform. Asserting platforms remain responsible for the underlying consent ceremony; ATAH disclaims that responsibility explicitly through the verbatim receipt-hash limitation framing.
- **Required conformance test.** Conformance suite verifies all six continuity-binding mismatches (client_id mismatch, pseudonymous_consumer_ref mismatch, scope mismatch, data_categories_hash mismatch, expired receipt, revoked receipt) produce rejections with `consent_continuity_mismatch` audit events.
- **Status: `bounded-by-protocol`**. ATAH detects tampering, replay, and continuity mismatch via receipt hash + continuity binding fields. ATAH cannot and does not prove that the platform's underlying consent ceremony was valid (whether the user actually clicked "I consent" in the platform's UI). The protocol bounds the abuse without resolving it. The platform retains the legal responsibility for the underlying consent ceremony.

### 1.3 Professional delegated-agent token is stolen

- **Scenario.** An attacker obtains a delegated agent's credentials or token and acts on a professional's behalf without that professional's knowledge.
- **Abuse / failure mode.** Unauthorised actions taken under a real professional's identity; profile changes, withdrawal, response-on-behalf-of without the principal's authority.
- **Expected protocol behaviour.** Authority basis `professional_delegated_token` carries `expires_at` (mandatory for time-bounded credentials per `principal-delegation.schema.json`) and is bound to `object_constraints` of `professional_id` (own). Tokens are short-lived and rotated; the §7.3 matrix permits them only for `actor_type: professional` actions on the bound professional id.
- **Required audit events.** Every action under `professional_delegated_token` records the token's `expires_at` and the bound `professional_id` in `authority_context`. Anomaly detection on the audit stream flags token use after revocation, from unexpected client applications, or against object constraints other than the bound principal.
- **Required user / professional disclosure.** Professional-facing audit feed (`GET /v1/professionals/me/disputes` and equivalent visibility endpoints) surfaces actions taken under their delegated tokens with the token's basis and constraints.
- **Required conformance test.** Conformance suite verifies that a request with `authority_basis: professional_delegated_token` whose `expires_at` is past, or whose `object_constraints[professional_id]` does not match the targeted principal, is rejected.
- **Status: `resolved`** by `authority_basis: professional_delegated_token` with mandatory `expires_at` and `object_constraints`, combined with audit-event recording of the basis. Operational hardening (token rotation cadence, anomaly detection thresholds) is registry-implementation responsibility.

### 1.5 Consumer handoff is attempted from a different platform/session without continuity authority

- **Scenario.** A handoff (Stage 2 / Stage 3) is attempted using identifiers or receipt that originated on a different AI platform or session than the one whose continuity binding the receipt records.
- **Abuse / failure mode.** Cross-platform replay; consent receipt or principal-delegation context being reused outside its issuing platform/session.
- **Expected protocol behaviour.** _See spec section referenced below._
- **Required audit events.** _See spec section referenced below._
- **Required user / professional disclosure.** _See spec section referenced below._
- **Required conformance test.** _See spec section referenced below._
- **Status: `partially-resolved`** — The protocol supplies the structural carrier; The protocol adds the continuity binding fields on the stored consent receipt that detect cross-platform replay end-to-end.

## Category 2 — Consent Boundary Failure


### 2.1 ATAH handoff consent is misrepresented as engagement consent

- **Scenario.** A Stage 2 or Stage 3 handoff consent is presented to a downstream professional or surface as if it were ongoing engagement consent (e.g. authorisation to retain, contact further, or open a relationship).
- **Abuse / failure mode.** Scope creep at the consent boundary; consumer's narrow authorisation expanded into broader permissions.
- **Expected protocol behaviour.** Per spec §1.5, §4.10, and Charter Part Two operational commitments, ATAH supports two consent types (`query_authorization`, `disclosure_consent`); engagement consent is excluded by design and the `consent_type` enum has no value for it. The MUST NOT rule applies verbatim across spec, EXPLAINER, PRD, and Charter: ATAH consent receipts MUST NOT be represented as professional engagement consent; conforming implementations MUST disclose that any professional-client relationship arises only through the professional's own onboarding, engagement, regulatory, or contractual process.
- **Required audit events.** `consent_receipt_submitted` records the `consent_type` (one of the two supported values). Any attempt to issue an `engagement_consent` receipt fails validation at submission.
- **Required user / professional disclosure.** AI platforms MUST surface the engagement-consent boundary at point of consent capture; the canonical disclosure language is published in the protocol documentation. EXPLAINER and PRD §9 carry the same framing for downstream readers.
- **Required conformance test.** Conformance suite verifies that a receipt with `consent_type: engagement_consent` is rejected at `POST /v1/consent-receipts` with a validation error.
- **Status: `resolved`** by normative rule (verbatim adopted across spec / Charter / EXPLAINER / PRD), the `consent_type` enum restriction on `consent-receipt.schema.json`, and the canonical disclosure-language requirement.

### 2.2 User consents to one professional but data is sent to another

- **Scenario.** Consumer consents at Stage 2 / Stage 3 in respect of a specific candidate; the platform routes data to a different professional (substituted, additional, or the wrong record).
- **Abuse / failure mode.** Mismatch between consented scope and actual recipient; consent receipt's `data_categories_hash` and / or recipient binding does not match the delivery target.
- **Expected protocol behaviour.** Per spec §4.10 (continuity binding), the stored consent receipt records `professional_id` as part of its captured binding (where the receipt is professional-bound). On consumption, ATAH verifies the consuming session targets the same `professional_id`; mismatch is a continuity-binding violation. For `disclosure_consent`, the requested data categories are also verified against `data_categories_hash`; mismatch is a continuity-binding violation.
- **Required audit events.** `consent_continuity_mismatch` on the failed consumption attempt, with the principal-delegation context, the receipt id, and the mismatched binding field.
- **Required user / professional disclosure.** Mismatch surfaces as a `consent_mismatch` error to the consuming AI platform; the consumer's session is not advanced.
- **Required conformance test.** Conformance suite issues a receipt for professional A then attempts a Stage 2 submission targeting professional B; verifies the submission is rejected with `consent_continuity_mismatch` audit event.
- **Status: `resolved`** by continuity binding (`professional_id` binding + `data_categories_hash`).

### 2.3 Consent text differs from stored receipt metadata

- **Scenario.** What the consumer was shown at the consent ceremony does not match the metadata recorded on the stored consent receipt (different scope, recipients, data categories, or duration).
- **Abuse / failure mode.** Drift between displayed and recorded consent; later disputes are unresolvable on protocol grounds.
- **Expected protocol behaviour.** Per §4.10, the receipt's `consent_text_version` records the canonical consent text the consumer was shown; the published consent-text version is part of the receipt and is hashed alongside the rest of the receipt body. On dispute, the asserting platform produces the receipt and the consent-text version; ATAH verifies the receipt hash matches the stored hash, confirming the receipt itself was not altered after the fact. The receipt-hash limitation framing applies: ATAH verifies receipt integrity, not the validity of the underlying consent ceremony.
- **Required audit events.** `consent_receipt_submitted` records the `consent_text_version`. Future `consent_continuity_mismatch` events triggered by drift detection are recorded with the divergent fields.
- **Required user / professional disclosure.** Asserting platforms MUST show the consent text matching the recorded `consent_text_version`. Drift is a platform-side conformance failure surfaced through dispute resolution.
- **Required conformance test.** Conformance suite verifies that submitting a receipt with `consent_text_version` not matching any published canonical text produces a validation error.
- **Status: `bounded-by-protocol`** — receipt integrity is verified by hash; ATAH cannot prove what the consumer actually saw on the platform's UI. The platform's consent ceremony remains its own responsibility per the receipt-hash limitation framing.

### 2.4 Consent is revoked after Stage 2 or Stage 3

- **Scenario.** Consumer revokes consent after data has already been delivered to a downstream professional or surface.
- **Abuse / failure mode.** Ambiguous semantics for "consent revoked after delivery" — what must downstream parties do, and what does ATAH itself record?
- **Expected protocol behaviour.** Per §11.9 and §4.10, the receipt's `revocation_status` flips to `revoked`; on the consuming-side, `POST /v1/introductions/{handoff_id}/revoke-consent` performs vault crypto-erasure for any remaining vault payload and stops ATAH processing. ATAH cannot force deletion from the professional's systems where data has already been retrieved (§11.9 scenario 3 documents this acknowledgement). The `revocation_status: active` check on consumption ensures revoked receipts cannot drive new actions.
- **Required audit events.** `consent_revoked` on the receipt; `data_erased` for any remaining vault payload; `withdrawal_recorded` if the revocation came in via the §11.9 scenario 3 path.
- **Required user / professional disclosure.** AI platforms surface the revocation to the consumer; the professional receives a notification that consent has been revoked and that their downstream data-handling obligations under their own privacy framework apply.
- **Required conformance test.** Conformance suite verifies that any consenting endpoint rejects consumption of a revoked receipt with `consent_receipt_revoked` error.
- **Status: `resolved`** by §4.10 `revocation_status` checks at every consumption point + §11.9 scenario 3 (consumer-withdrawal-after-Stage-3-retrieval semantics + vault crypto-erasure).

### 2.5 Receipt replay across users (cross-user confusion)

- **Scenario.** Platform X submits a receipt for user A (`pseudonymous_consumer_ref` HMAC-A); subsequently attempts to consume it for an action concerning user B (whose pseudonymous reference HMACs to a different value).
- **Abuse / failure mode.** Receipt is valid (hash check passes); consumption context does not match captured context.
- **Expected protocol behaviour.** Per continuity binding, ATAH verifies the consuming session's `pseudonymous_consumer_ref` matches the stored HMAC at consumption time. Mismatch is a continuity-binding violation.
- **Required audit events.** `consent_continuity_mismatch` on the failed consumption with the receipt id and the mismatched binding field (`pseudonymous_consumer_ref`).
- **Required user / professional disclosure.** Failed consumption surfaces as a `consent_mismatch` error to the consuming AI platform.
- **Required conformance test.** Conformance suite issues a receipt for user A then attempts consumption with a different pseudonymous reference; verifies rejection with the `consent_continuity_mismatch` audit event.
- **Status: `resolved`** by continuity binding (`pseudonymous_consumer_ref` HMAC).

### 2.6 Receipt replay across sessions (same user, different action)

- **Scenario.** A receipt issued for one consenting action is consumed for a different action without re-consent.
- **Abuse / failure mode.** Captured `scope` does not match the consuming endpoint's required scope; or `data_categories_hash` does not match the data being shared at consumption.
- **Expected protocol behaviour.** Per §4.10 continuity binding, ATAH verifies the consuming endpoint's required scope matches the receipt's captured `scope`, and (for `disclosure_consent`) that the requested data categories match the receipt's `data_categories_hash`. Mismatch is a continuity-binding violation. Additionally, the receipt's `expires_at` is enforced.
- **Required audit events.** `consent_continuity_mismatch` on the failed consumption with the mismatched binding field (`scope` or `data_categories_hash`).
- **Required user / professional disclosure.** Failed consumption surfaces as a `consent_mismatch` error.
- **Required conformance test.** Conformance suite issues a receipt with scope `stage_2_prehandoff_submission` then attempts consumption at `stage_3_contact_release`; verifies rejection. Also: receipt with one set of data categories, attempt to share a different set; verifies rejection.
- **Status: `resolved`** by continuity binding (`scope` and `data_categories_hash` checks) + receipt expiry.

## Category 3 — Ordering and Recommendation Drift


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
- **Expected protocol behaviour.** Per spec §9.1, the "no global score" claim is structurally observable: no global score field exists at any layer of `match-response.schema.json`. "a hidden global score plus randomised display would look cosmetic and undermine the claim that ATAH is not ranking by preference"; the structural absence closes that vector.
- **Required audit events.** None specific (the structural absence is the verification surface, not an audit-event surface).
- **Required user / professional disclosure.** Response-level `presentation_disclosure.ordering_policy` declares the mode (`stratified_random` by default) and `atah_expresses_preference: false`.
- **Required conformance test.** Conformance suite verifies that no global score field is present in any match-response example or in implementation responses; that `band_assignment` is populated for every candidate; and (per §9.5) that within-band ordering is observably non-deterministic.
- **Status: `resolved`** by the structural absence of any global score in `match-response.schema.json` and the band-and-stratify architecture in §9.1.

### 3.3 Commercial partner candidates appear disproportionately

- **Scenario.** Professionals associated with commercial partners (paying or otherwise) appear in eligible result sets at frequencies inconsistent with their share of the eligible population.
- **Abuse / failure mode.** Commercial bias entering ordering through a back channel — band assignment, eligibility filters, or stratification weights — without disclosure.
- **Expected protocol behaviour.** Per spec §9.6 (Commercial neutrality), no weight is assigned to ATAH-partner commercial relationships, partner status itself, enhanced-verification payment, review-platform identity beyond `review_platform_class`, or registration route (partner vs individual). Per / §9.2, review-derived signals MAY supplement transparency but MUST NOT promote candidates into higher bands in regulated categories. The response-level `presentation_disclosure.ordering_policy.commercial_weighting: false` is `const`-asserted in the schema (the field is structurally not a free value). The protocol's transparency-as-conformance work adds per-candidate `decision_explanation` so the basis of band assignment is auditable per response.
- **Required audit events.** Audit log records every `partner_data_pushed` and `verifier_data_submitted` event with the principal-delegation context; pattern detection of partner-affiliated overrepresentation is an operational concern.
- **Required user / professional disclosure.** `commercial_weighting: false` is asserted in every match response. AI platforms reordering MUST preserve this.
- **Required conformance test.** Conformance suite verifies (a) `commercial_weighting: false` is asserted in every response; (b) `decision_explanation` exposes the band-assignment basis for each candidate so audit can verify no partner-relationship signal entered the assignment.
- **Status: `resolved`** by §9.6 commercial-neutrality MUST + §9.2 review-signal band cap + per-candidate decision_explanation transparency.

### 3.4 AI platform presents ATAH result one as "best"

- **Scenario.** ATAH returns a stratified set with a published ordering policy; the AI platform reorders, summarises, or labels the first result as "best" or "top recommendation" downstream.
- **Abuse / failure mode.** Platform-side presentation undermines ATAH's stratification and equal-treatment posture; users experience a recommendation despite the protocol's stance.
- **Expected protocol behaviour.** Per spec §9.4: "AI platforms reordering the ATAH response MUST preserve the `presentation_disclosure.ordering_policy` field verbatim and surface its content to the user. AI platforms MUST NOT present ATAH's response as a recommendation or rank by ATAH's preference; ATAH expresses no preference among eligible candidates by default." Failure to preserve the disclosure is a binding-conformance failure (Section 14). The structural surface ATAH provides — `presentation_disclosure.ordering_policy` with `atah_expresses_preference: false` — is the platform's obligation to surface verbatim.
- **Required audit events.** Not directly auditable from ATAH's side (the platform is downstream of ATAH's surface). Platform conformance is verified through binding-conformance review, not audit-stream pattern detection.
- **Required user / professional disclosure.** AI platforms display `presentation_disclosure.ordering_policy` content to users. Where the platform reorders for user-context reasons, the disclosure that ATAH applied non-preferential ordering MUST survive.
- **Required conformance test.** Binding-conformance review verifies platform implementations preserve and surface the `presentation_disclosure.ordering_policy` block.
- **Status: `allocated-to-platform-responsibility`**. ATAH publishes the ordering-policy disclosure; downstream presentation is the AI platform's obligation. ATAH's role is to provide the structural disclosure surface and the binding-conformance MUST; the platform's role is to honour it. (This was the worked example for `allocated-to-platform-responsibility`.)

### 3.5 Review signals quietly upgrade a candidate's band in a regulated category

- **Scenario.** A regulated-category implementation lets review-platform signals push a candidate into a higher verification-confidence or category-fit band, despite the absence of corroborating regulator-source evidence.
- **Abuse / failure mode.** Review-platform signal becomes a back-door promotion mechanism in high-stakes categories; the v0.8.1 `review_signal_weight_cap` original safeguard is bypassed.
- **Expected protocol behaviour.** Per spec §9.2: review-derived signals MAY supplement transparency and confidence metadata, but for high-stakes regulated categories MUST NOT move a candidate into a higher eligibility or verification band unless corroborated by authoritative credential or regulator-source evidence. Encoded per category in `profession-categories.json` `review_signal_band_cap`: regulated categories set `may_upgrade_band: false` and `regulated_category_max_effect: no_band_upgrade`.
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
- **Required conformance test.** Conformance suite issues the same query repeatedly and verifies within-band candidate ordering is observably different across N invocations. A future revision may add a stronger statistical test; v0.9 may standardise the algorithm.
- **Status: `partially-resolved`** — The protocol establishes the structural model and the conformance non-determinism test; full statistical-distribution conformance and algorithm standardisation are deferred. Concrete next step: algorithm-conformance test; v0.9 algorithm standardisation.

---

## Category 4 — Transparency and Explainability Failure


### 4.1 Professional asks why they were excluded

- **Scenario.** A verified professional, expecting to appear in eligible result sets, queries the protocol about exclusion reasons.
- **Abuse / failure mode.** No mechanism for professional-facing transparency; or the mechanism leaks query-history / volume data inappropriately.
- **Expected protocol behaviour.** Per spec §11A.4, a conforming implementation MUST provide a mechanism for an authenticated professional to retrieve their own visibility-decision view. The view is **rules-derived**: representative inclusion rules that apply, representative exclusion reason categories that could apply, current band assignment from `band_definitions`, and the implementation's published ordering policy. Per MUST NOT, the view does NOT expose actual query-count or query-history data. Endpoint: `GET /v1/professionals/me/visibility-explanations` (REST) and `get_my_visibility_explanations` (MCP). Implementations MAY defer the endpoint implementation to v0.8.3 via `x-implementation-deferred-to`; the obligation is part of v0.8.2.
- **Required audit events.** Each retrieval generates a `security_event` (or equivalent) recording the principal-delegation context, the category and jurisdiction queried, and the timestamp.
- **Required user / professional disclosure.** Authority basis: `professional_delegated_token`. Rate-limited per professional account.
- **Required conformance test.** Conformance suite verifies the endpoint returns the rules-derived shape (representative inclusion rules, representative exclusion reason categories, current band assignment, published ordering policy); verifies absence of query-history fields in the response; verifies authority-basis enforcement.
- **Status: `resolved`** by §11A.4 verbatim normative rule + dedicated endpoint specification (implementation may defer to v0.8.3 but obligation is v0.8.2).

### 4.2 Consumer asks why a candidate appeared

- **Scenario.** A consumer (via the AI platform) asks why a particular professional was returned in the eligible set, or why one candidate appeared above another in the stratified ordering.
- **Abuse / failure mode.** Decision basis is opaque; ordering looks like a recommendation; consumer trust depends on a transparent rationale.
- **Expected protocol behaviour.** Per spec §11A.2, the per-candidate `decision_explanation` (Layer 2) on `match-response.schema.json` documents the rules each candidate passed, the band assignment basis, and the ordering policy applied. The response-level `decision_explanation` (Layer 1) documents the query-level rules and ordering policy. Both layers reference the audit event for traceability.
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
- **Status: `partially-resolved`** — protocol behaviour closes the transparency side (suppression reason visible; dispute path visible). The fuller dispute-resolution timeline / escalation criteria / evidence-handling specification is v0.9 work (already on ROADMAP under "Detailed dispute resolution process").

### 4.5 Implementation provides response-level explanation only and claims conformance

- **Scenario.** An implementation produces a response-level `decision_explanation` but omits per-candidate `decision_explanation` on individual results, claiming response-level alone is sufficient.
- **Abuse / failure mode.** Per-candidate "why this specific candidate" question becomes unanswerable from the response alone; conformance gap exploited.
- **Expected protocol behaviour.** Per §11A.2, the Transparency Class requires both layers. `match-response.schema.json` `MatchResult` `required` includes `decision_explanation`; a response without per-candidate explanations fails schema validation.
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
- **Abuse / failure mode.** "Deferred endpoint implementation" is misread as "deferred obligation"; v0.8.2 ships without the substance of the commitment.
- **Expected protocol behaviour.** Per §11A.4 / spec discipline, the v0.8.2 spec defines the required behaviour, fields, authority controls, audit linkage, and the MUST NOT rule. The deferral covers the dedicated endpoint implementation; it does NOT cover the obligation itself. Implementations MAY return 501 in v0.8.2 — but the spec is complete enough that an implementer can build from v0.8.2 alone. The CHANGELOG and ROADMAP record the deferral honestly.
- **Required audit events.** None specific (the structural verification is via spec-completeness review).
- **Required user / professional disclosure.** The CHANGELOG entry and the ROADMAP "v0.8.3 candidates" section make the deferral visible.
- **Required conformance test.** Conformance suite review verifies that the v0.8.2 spec section §11A.4 contains all five required components (behaviour, fields, authority controls, audit linkage, MUST NOT). The endpoint implementation deferral is a registry-side decision verified through binding-conformance review.
- **Status: `bounded-by-protocol`** — spec discipline keeps the obligation visible at the spec level even when implementation defers. The protocol bounds the deferral risk; implementations choosing to defer are explicit about it.

### 4.8 Professional-facing endpoint exposes actual query data, leaking consumer/platform information

- **Scenario.** An implementation builds the visibility-explanation endpoint by aggregating actual query traffic ("here's how many queries this category received last month and what fraction included you").
- **Abuse / failure mode.** Real query patterns expose information about consumers, AI platform integrations, demand signals, matter types, urgency, and platform activity that does not belong in a per-professional report. Even aggregated counts can leak in low-volume categories.
- **Expected protocol behaviour.** Per §11A.4: professional-facing visibility explanations MUST NOT expose actual query-count data, query-history data, observed demand patterns, or any per-query information derived from consumer or platform traffic. The view MUST be rules-derived from the implementation's documented rules and the professional's own profile data.
- **Required audit events.** Implementations producing query-history-derived visibility views fail conformance review.
- **Required user / professional disclosure.** N/A — the failure mode is implementation-side and the spec rule is the verification surface.
- **Required conformance test.** Conformance suite reviews the implementation's visibility-explanation response payloads for any field derived from query traffic; the absence of query-history fields is the structural test. The reviewer also examines the implementation's documentation and source where available.
- **Status: `resolved`** by MUST NOT in spec §11A.4 + cross-references in CHANGELOG, EXPLAINER, PRD, and `docs/professional/visibility-report.md`.

### 4.9 Professional-facing endpoint exposes inferred query patterns via low-volume category leakage

- **Scenario.** An implementation provides aggregated query counts at category level only, not per-professional, but the category is low-volume enough that "lawyers in Wyoming, county court matters" effectively names individual professionals.
- **Abuse / failure mode.** Low-volume aggregation leaks demand information for narrow categories or jurisdictions, indirectly identifying professionals.
- **Expected protocol behaviour.** The rules-derived approach is structurally immune to this failure mode: the explanation is derived from documented rules and the professional's own profile, not from observed query traffic. There are no aggregate counts in the visibility-explanation response (the schema does not include any field that could carry them); low-volume leakage is impossible by construction.
- **Required audit events.** N/A.
- **Required user / professional disclosure.** N/A.
- **Required conformance test.** Conformance suite verifies the visibility-explanation response schema does not include query-count fields; the field absence is the structural test.
- **Status: `resolved`** by rules-derived approach + structural absence of query-count fields in the response schema.

---

## Category 5 — Vault, Erasure, and Audit Abuse


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


### 6.1 Malicious actor initiates and withdraws repeated handoffs

- **Scenario.** An attacker or harasser initiates handoffs to a target professional and withdraws each one before completion, exploiting the withdrawal mechanism for harassment, denial-of-service, or reputational signal noise.
- **Abuse / failure mode.** Withdrawal as harassment vector; rate-limit gaps and lack of immutable metadata enable repeat abuse.
- **Expected protocol behaviour.** Per §11.9, withdrawal is a state transition and does NOT erase audit records. Every initiate-then-withdraw cycle generates a structured audit event with the principal-delegation context, `previous_state`, `resulting_state`, and `reason_code`. Pattern detection (repeat withdrawals from the same `authenticated_actor`) is therefore visible in the audit stream. Per §13.3, rate limits on per-period proposal/initiation actions apply (concrete numbers populated by registry implementations; spec mandates the requirement).
- **Required audit events.** `introduction_initiated` and `withdrawal_recorded` (or the equivalent state-transition event_type) for each cycle, all carrying the same `authenticated_actor.actor_id` and `client_application` for pattern detection.
- **Required user / professional disclosure.** The target professional MAY surface withdrawal patterns through their professional portal; harassment-monitoring is a v0.8.3 deferred mechanism (per scenario 1.6 deferral).
- **Required conformance test.** Conformance suite verifies that withdrawal generates an audit event with the fields and that rate-limit enforcement returns `429` after threshold.
- **Status: `partially-resolved`** — closes the structural side via withdrawal-as-state-transition + audit retention. Concrete rate-limit thresholds and harassment-monitoring (dense-cluster pattern detection) are `deferred-to-v0.8.3`.

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

## Category 7 — Contact Freshness and Engagement Liveness


### 7.1 Licensed professional has stale email or phone

- **Scenario.** A verified, eligible professional's recorded contact channels (email, phone, notification endpoint) point to addresses that are no longer monitored or owned.
- **Abuse / failure mode.** Notifications go unreceived; consumer expects a live route, professional misses opportunity, ATAH appears to recommend a non-reachable contact.
- **Expected protocol behaviour.** Per spec §12A.1 (Layer 1 periodic verification), every channel on the professional record carries `last_verified` and `verification_status`. Verification challenges are issued on a per-category cadence (`quarterly` / `biannual` / `annual` / `none`) per `profession-categories.json` `contact_verification_cadence`. Missed challenges escalate per §12A.2; after the full grace period (~74 days), `matching_status` flips to `contact_unverified` and the professional stops appearing in Discovery results until re-verified.
- **Required audit events.** `channel_verification_challenged` (challenge issued), `channel_verification_confirmed` (challenge confirmed), `channel_verification_escalated` (Layer 2 escalation triggered), `channel_marked_unverified` (full grace period elapsed), `matching_status_flipped_to_contact_unverified`.
- **Required user / professional disclosure.** Professional's portal surfaces channel verification status; partner-route professionals receive Layer 2 escalation through their partner organisation; individual self-registered receive escalation across other registered channels and an in-portal alert.
- **Required conformance test.** Conformance suite simulates a missed verification cycle; verifies the lifecycle `verified` → `verification_pending` → `escalation_pending` → `unverified` and the corresponding `matching_status` transitions; verifies the audit event sequence.
- **Status: `resolved`** by §12A.1 Layer 1 periodic verification + §12A.2 Layer 2 escalation path.

### 7.2 Notification goes to an abandoned channel

- **Scenario.** A handoff notification is dispatched to a channel whose underlying account / address has been abandoned by the recipient; delivery succeeds at the transport layer but no human ever sees it.
- **Abuse / failure mode.** Silent delivery failure; no surfacing of the staleness condition until the consumer's wait time has elapsed.
- **Expected protocol behaviour.** Per §12A.1 Layer 1, the periodic verification cadence detects the abandoned channel before the next handoff (channel won't confirm; status flips through `verification_pending` → `escalation_pending` → `unverified`). Per §12A.3 Layer 3, for high-stakes categories a fresh verification challenge runs just before the introduction notification; if the challenge times out within 24 hours, the introduction is deferred back to the consumer's AI with status `professional_contact_unverified` rather than dispatching to the abandoned channel silently.
- **Required audit events.** `channel_verification_challenged` and the eventual lifecycle events; for Layer 3 timeouts, the audit event records the deferred-back outcome with the principal-delegation context.
- **Required user / professional disclosure.** Consumer's AI surfaces the deferred-back status to the consumer with a meaningful message; the professional's portal flags the failed verification.
- **Required conformance test.** Conformance suite issues a challenge to a non-responsive channel and verifies (a) Layer 1 lifecycle transitions occur on schedule; (b) for high-stakes categories with `pre_handoff_freshness_window_days` populated, an introduction attempt within the window for a stale channel triggers the Layer 3 challenge and returns `professional_contact_unverified` on timeout.
- **Status: `resolved`** by §12A.1 Layer 1 + §12A.3 Layer 3.

### 7.3 Professional repeatedly misses Stage 1 response commitments

- **Scenario.** A professional appears in eligible result sets but consistently fails to respond within their declared response window at Stage 1.
- **Abuse / failure mode.** Engagement-liveness signal absent; professional remains in eligible set despite a real-world unresponsive pattern.
- **Expected protocol behaviour.** *Engagement liveness is **deferred to v0.8.3**.* The protocol covers channel-validity verification only; response-rate tracking is a separate mechanism not in scope for v0.8.2.
- **Required audit events.** _Deferred to v0.8.3._
- **Required user / professional disclosure.** _Deferred to v0.8.3._
- **Required conformance test.** _Deferred to v0.8.3._
- **Status: `deferred`** — `deferred-to-v0.8.3` — engagement liveness mechanism out of v0.8.2 scope; v0.8.2 covers contact-channel verification only.

### 7.4 Professional appears active but is not reachable

- **Scenario.** Profile metadata (last activity, recent updates) suggests the professional is active, but the verified contact channels are not currently functioning (mailbox full, number disconnected, integration broken).
- **Abuse / failure mode.** Profile-level "active" signal diverges from channel-level "reachable" signal; the latter is what the consumer experiences.
- **Expected protocol behaviour.** Per §12A.3 Layer 3 pre-handoff freshness check, for high-stakes categories ATAH issues a fresh verification challenge before sending the introduction notification when the relevant channel hasn't been verified within `pre_handoff_freshness_window_days` (30 days in v0.8.2). On timeout (24 hours) the introduction is deferred back to the consumer's AI with `professional_contact_unverified` status. This catches the divergence between profile-level "active" and channel-level "reachable" — the professional may have an active profile and recent activity, but the channel-level check fails just-in-time.
- **Required audit events.** `channel_verification_challenged` for the Layer 3 challenge; on timeout, the introduction-initiated audit event records the deferred-back outcome with the principal-delegation context.
- **Required user / professional disclosure.** Consumer's AI surfaces the deferred-back status; the professional's portal shows the failed Layer 3 challenge and prompts re-verification.
- **Required conformance test.** Conformance suite simulates a high-stakes-category introduction attempt where the channel was last verified outside the `pre_handoff_freshness_window_days` window; verifies Layer 3 challenge is issued; verifies on timeout the introduction returns `professional_contact_unverified`.
- **Status: `resolved`** by §12A.3 Layer 3 pre-handoff freshness check for high-stakes categories.

---

## Category 8 — Partner and Governance Capture


### 8.1 Paying partner seeks broader verification scope than justified

- **Scenario.** A trusted partner who pays for, or sponsors, the partnership relationship requests an expansion of their authoritative-data scope beyond what their underlying authority justifies.
- **Abuse / failure mode.** Verification-scope creep; partner's authority over a narrow domain (e.g. licence data) leveraged into broader implicit endorsement.
- **Expected protocol behaviour.** Per Charter Part Two hard-artifacts list: field-level verification scopes per partner are an existing hard artifact (`verification-scope.schema.json`). Partner data submissions outside the declared scope are rejected at validation; scope expansion requires governance review and is logged in the public governance-decision register (v0.9 operational artifact). Per Charter Core Commitment 8, the protocol governance body holds the partner admission and scope rules; per Core Commitment 7 / GOVERNANCE.md §4.1, founder-affiliated or commercially connected admissions trigger related-party disclosure.
- **Required audit events.** `partner_data_pushed` events outside scope produce validation failures recorded as `security_event` audit entries. `admin_action` events record scope-expansion governance decisions.
- **Required user / professional disclosure.** Scope-expansion decisions surface through the public governance-decision register; professionals see partner-attributed data with provenance per the `_provenance` map.
- **Required conformance test.** Conformance suite verifies that partner data submissions outside the partner's declared `verification-scope.schema.json` are rejected; verifies that the partner's declared scopes are publicly accessible.
- **Status: `resolved`** by hard-artifact (`verification-scope.schema.json`) + Charter Core Commitment 8 protocol-governance independence + GOVERNANCE.md §4.1 related-party disclosure rules.

### 8.2 Review platform with weak anti-gaming wants admission

- **Scenario.** A review platform with insufficient anti-gaming controls applies for admission as a partner data source.
- **Abuse / failure mode.** Low-quality review signal entering ATAH band assignment; gameable signals influencing professional appearance.
- **Expected protocol behaviour.** Per Charter Part Two hard-artifacts list: review-platform minimum criteria are an existing hard artifact (`review-platform.schema.json` defines the partner-class shape; admission requires declared anti-gaming controls and review-platform-class self-classification). Per §9.2 /: review-derived signals MUST NOT promote candidates into higher bands in regulated categories regardless of their anti-gaming posture; the structural protection applies even if a weak review platform is admitted.
- **Required audit events.** Review platform admission decisions produce `admin_action` audit events; review-platform-attributed data carries `_provenance` for inspection.
- **Required user / professional disclosure.** Review-platform-class self-classification is publicly visible; per §9.2, review signals are presentational and supplemental.
- **Required conformance test.** Conformance suite verifies that review platforms outside the published criteria are rejected; verifies that review-derived signals do not promote candidates into higher bands in regulated categories (the §9.2 / MUST rule).
- **Status: `partially-resolved`** — The protocol references existing hard-artifact (`review-platform.schema.json`) + §9.2 structural protection. **The fuller minimum-criteria specification (anti-gaming attestation requirements, audit cadence, revocation triggers) is a v0.9 work item**, named in the Charter Part Two hard-artifacts list and on the ROADMAP.

### 8.3 Verifier sells enhanced verification as a purchasable badge

- **Scenario.** An independent verifier offers professionals a paid "enhanced verification" badge whose substantive verification rigour is not commensurate with the badge's prominence.
- **Abuse / failure mode.** Verification becomes a marketing channel; signal devaluation across the ecosystem.
- **Expected protocol behaviour.** Per Charter Part Two hard-artifacts list: verifier conflict and audit rules are partial in v0.8.2 (`independent-verifier.schema.json` defines the verifier shape); the fuller specification is v0.9. Per the matching engine §9.6 commercial-neutrality rule: no weight is assigned to which party paid for an enhanced verification; only the structured verification depth, breadth, and freshness influence ranking. Per Charter §"Independent verifier and review platform principles": verifiers commit to neutrality, conflict declaration, and revocable conformance.
- **Required audit events.** `verifier_data_submitted` events record the verification record's structured depth/breadth/freshness; `verifier_suspended` events record revocation.
- **Required user / professional disclosure.** Enhanced verification records carry full provenance per the `_provenance` map; the verifier identity, the verification scope, and the verification rigour are visible to consumers and auditors.
- **Required conformance test.** Conformance suite verifies that paid-for enhanced verifications do not get higher matching weight than equivalently-rigorous unpaid verifications; verifies that verifier conflict-of-interest declarations are publicly accessible.
- **Status: `partially-resolved`** — The protocol references existing hard-artifact (`independent-verifier.schema.json`) + §9.6 commercial-neutrality rule. **The fuller verifier conflict and audit rules (cadence, revocation triggers, conflict-declaration shape) are a v0.9 work item**, named in the Charter Part Two hard-artifacts list and on the ROADMAP.

### 8.4 Founder-affiliated or commercially connected entity seeks preferential treatment

- **Scenario.** An entity related to ATAH's founders, governance body, or significant commercial counterparties seeks preferential admission, scope, or visibility.
- **Abuse / failure mode.** Governance capture; perceived neutrality of the protocol layer compromised even if the technical effect is small.
- **Expected protocol behaviour.** Per Charter Core Commitment 7 (founder accountability) and Core Commitment 8 (protocol governance is held by an independent not-for-profit or equivalent public-interest entity, with operational service-provider arrangements terminable / replaceable / non-exclusive). Per GOVERNANCE.md §4.1: related-party arrangements (including founder-affiliated entities) carry mandatory disclosure obligations and recusal rules. The strengthened Core Commitment 8 in v0.8.2 makes the form mandate at the protocol governance layer explicit.
- **Required audit events.** `admin_action` events record related-party admission decisions with the principal-delegation context; recusal is recorded.
- **Required user / professional disclosure.** Public governance-decision register (v0.9 operational artifact) records related-party decisions; founder-affiliated entity register is published per Charter §"Founder accountability".
- **Required conformance test.** Conformance suite reviews the public governance-decision register and the related-party disclosure register; verifies recusal patterns where founder-affiliated entities applied for partner admission, scope expansion, or commercial agreement.
- **Status: `resolved`** by Charter Core Commitments 7 and 8 + GOVERNANCE.md §4.1 related-party disclosure and recusal rules.

### 8.5 Implementation claims ATAH conformance using only the technical protocol while violating Charter spirit

- **Scenario.** An implementation publishes a conformance statement at `/.well-known/atah-conformance` and validates against the published schemas, but applies opaque ordering, undisclosed commercial weighting, or other behaviours that violate the Charter's spirit. Apache 2.0 permits the use of the protocol; the conformance claim is only a self-declaration during the release-candidate stage.
- **Abuse / failure mode.** "Conformance laundering" — technical conformance is used to imply governance neutrality, blurring the distinction between protocol implementation and recognised neutrality.
- **Expected protocol behaviour.** Per `CONFORMANCE.md` (the three-level distinction): such an implementation is `protocol-compatible` (uses ATAH schemas) or, at most, `ATAH-conformant` if the technical conformance assertions hold — but it is NOT `ATAH-recognised neutral implementation`, which requires behavioural conformance to governance, neutrality, auditability, public-disclosure, and conflict-of-interest requirements. Per The protocol's Transparency Class, ordering policy and decision explanations are observable; commercial-weighting hidden in implementation logic becomes detectable through audit and decision-explanation discrepancies.
- **Required audit events.** Implementation-side audit events (covered by the Transparency Class). Cross-implementation comparison is the basis on which an ATAH-recognised-neutral claim is evaluated against Charter-spirit conformance.
- **Required user / professional disclosure.** `presentation_disclosure` carries the ordering-policy disclosure; `decision_explanation` per response and per candidate documents the rules applied; aggregate `exclusion_summary` surfaces exclusion-reason categories. A reviewer can compare the published conformance statement against the observable behaviour.
- **Required conformance test.** v0.8.2 ships the schema-level checks (schema validity, presence of `presentation_disclosure`, presence of `decision_explanation` at both layers, structural absence of `match_score` / `match_factors`). v0.9 audit regime adds the behavioural-neutrality / conformance-audit model — sampled query replay, decision-explanation substantive-validity tests, commercial-neutrality attestation cadence — that catches Charter-spirit violations a purely-technical regime would miss.
- **Status: `partially-resolved`** — The protocol's three-level distinction makes the framing clear in documentation (the recognised level is reserved for behavioural conformance, not just technical). Fully resolved when the v0.9 behavioural-neutrality / conformance-audit model is implemented per the new ROADMAP item.

### 8.6 Implementation falsely claims ATAH-recognised neutral status

- **Scenario.** An implementation that is at most `ATAH-conformant` (per the three-level distinction) publicly claims `ATAH-recognised neutral implementation` status without meeting the strongest governance, neutrality, auditability, public-disclosure, and conflict-of-interest requirements. The claim is misleading rather than technically wrong.
- **Abuse / failure mode.** Trust-mark laundering; consumers, partners, and reviewers may mistake technical conformance for recognised neutrality.
- **Expected protocol behaviour.** Per CHARTER Part Two (a v0.8.2 commitment), the official ATAH trust mark and "ATAH-recognised neutral implementation" status are revocable; they depend on continuing behavioural conformance. Per GOVERNANCE.md §5.1 (a v0.8.2 commitment), ATAH governance reserves the right to publicly contest claims of ATAH conformance, neutrality, or recognised-implementation status that misrepresent compliance. Both are stated commitments; v0.9 operationalises the revocation procedure.
- **Required audit events.** Public governance-decision register (v0.9 operational artifact per Charter Part Two hard-artifacts list) records contest actions, revocations, and remediation outcomes.
- **Required user / professional disclosure.** Public statement issued by ATAH governance when a claim is contested. The published-revocation process under v0.9 makes status transitions visible.
- **Required conformance test.** v0.8.2 ships the framework (the three-level distinction is documented; the revocability and right-to-contest are stated). v0.9 ships the issuance and revocation procedure (v0.9 ROADMAP item).
- **Status: `partially-resolved`** — The protocol creates the framework for revocability (CHARTER) and the public-contest right (GOVERNANCE §5.1). Fully addressed when v0.9 operationalises the revocation procedure.

### 8.7 Well-resourced player forks the registry, complies with technical conformance, weakens governance in practice

- **Scenario.** A well-resourced player implements an ATAH-compatible registry under its own brand, bundles it with existing distribution, complies with the letter of the technical protocol, weakens or avoids the Charter's spirit in practice, and out-competes the ATAH reference registry on operational quality, integrations, brand, and reach.
- **Abuse / failure mode.** Open licensing is exploited to capture the protocol's value while evading the governance commitments; the meaning of "ATAH" drifts under commercial pressure.
- **Expected protocol behaviour.** Per `CONFORMANCE.md` the three-level distinction: forks fall in `protocol-compatible` by default and cannot self-claim `ATAH-recognised neutral implementation` without meeting the governance/audit requirements (form is uniform at the protocol-governance layer Charter Core Commitment 8; conformance is uniform at the implementation layer per the five conformance classes plus the three-level recognition framework). Per the note in `commercial-neutrality-memo.md`: open licensing strengthens ATAH as a protocol but means the reference registry must compete on trusted governance, neutrality, auditability, and recognised conformance — not merely on authorship of the standard. Sophisticated reviewers (partners, regulators, enterprise platforms) can read the conformance level claimed and the audit posture behind it; the framing makes Charter-spirit divergence visible.
- **Required audit events.** Per the v0.9 audit regime: public ordering-policy metadata exposure tests, decision-explanation discipline checks, commercial-neutrality attestation. The audit regime is what makes hidden weakening detectable.
- **Required user / professional disclosure.** Conformance-status declarations are public; the three-level distinction is publicly documented; the right-to-contest is a stated governance commitment.
- **Required conformance test.** v0.8.2 ships the documented distinction. v0.9 ships the operational audit regime per the ROADMAP item. The protocol-level ceiling is the framing's visibility to reviewers; ATAH-the-organisation does not have legal-licence levers to prevent forks (Apache 2.0 permits them), and the goal claims.
- **Status: `bounded-by-protocol`** — the framing makes the distinction visible to sophisticated reviewers and to the v0.9 audit regime; the protocol cannot prevent forks (Apache 2.0 permits them), and the goal is meaning-preservation rather than fork-prevention. The protocol-level boundary is what's available; v0.9 audit verification adds the operational backbone.

---

## Category 9 — Category and Jurisdiction Misfit


### 9.1 Legal conflict rules vary by jurisdiction

- **Scenario.** Conflict-of-interest rules for legal practitioners differ materially across jurisdictions; a one-size implementation either over-blocks or under-detects conflicts.
- **Abuse / failure mode.** Jurisdiction mismatch; conflict rules from one country mis-applied in another.
- **Expected protocol behaviour.** Per existing v0.8.1 category-annex work + Charter Part Two hard-artifacts list (v0.9 canonical category-annex template). Per spec §9.1 hard filters: jurisdiction matching is a step-1 filter (a candidate not licensed in the jurisdiction is excluded). Per `profession-categories.json`: each category declares its own `band_definitions`; jurisdiction-specific behaviour is encoded per category through the annex process.
- **Required audit events.** Hard-filter exclusions for jurisdiction mismatch produce `decision_explanation` entries with `decision_type: exclusion` and `exclusion_reason_category: jurisdiction_mismatch`.
- **Required user / professional disclosure.** Aggregate `exclusion_summary` on match responses surfaces jurisdiction-mismatch counts; per-candidate `decision_explanation` documents the rules applied.
- **Required conformance test.** Conformance suite verifies jurisdiction-mismatch exclusions appear in the aggregate summary; verifies the published category annex documents jurisdiction-specific rules.
- **Status: `partially-resolved`** — existing v0.8.1 category-annex work + / transparency surface the structural mechanism. Per-category jurisdiction-rule canonicalisation (legal conflict rules, healthcare compliance, etc.) is v0.9 work via the canonical category-annex template named in the Charter Part Two hard-artifacts list.

### 9.2 Healthcare data requires stronger compliance controls

- **Scenario.** Healthcare-category queries and handoffs carry data subject to compliance regimes (HIPAA, GDPR special categories, equivalent national frameworks) stricter than ATAH's default posture.
- **Abuse / failure mode.** Default protocol controls insufficient for the category; conformance achieved without compliance.
- **Expected protocol behaviour.** Per `profession-categories.json` `compliance_status` field (existing v0.8.1): healthcare categories carry `compliance-pending` until category-specific compliance work (HIPAA-compliant handling, equivalent national frameworks) is signed off via the category-annex process. Per §9.1 hard filters, candidates with `matching_status: compliance-pending` are excluded from Discovery. The category-annex template (v0.9 work) will canonicalise per-jurisdiction healthcare-compliance requirements.
- **Required audit events.** `compliance_status` flips on category metadata produce `admin_action` events.
- **Required user / professional disclosure.** Categories in `compliance-pending` are publicly visible through the published profession-category metadata.
- **Required conformance test.** Conformance suite verifies that no candidate in a `compliance-pending` category appears in Discovery results.
- **Status: `partially-resolved`** — existing v0.8.1 `compliance_status` mechanism + §9.1 hard filter. **Per-jurisdiction healthcare compliance specification is v0.9 work** via the canonical category-annex template.

### 9.3 Financial-advice terminology differs by country

- **Scenario.** "Financial adviser", "investment professional", "wealth manager" carry different licensure, scope, and consumer-protection meanings across jurisdictions.
- **Abuse / failure mode.** Cross-jurisdiction term confusion; consumers misled about scope.
- **Expected protocol behaviour.** Per `profession-categories.json` per-category `category_id` and the per-jurisdiction body-membership-type structure: each category is canonicalised per jurisdiction through the category-annex process. Per spec §1.5 cross-jurisdiction default rule (existing v0.8.1): ATAH does not return cross-jurisdiction matches by default for regulated categories; cross-jurisdiction queries require explicit query parameters and a `presentation_disclosure` note.
- **Required audit events.** Cross-jurisdiction queries produce `decision_explanation` entries documenting the explicit cross-jurisdiction request.
- **Required user / professional disclosure.** Cross-jurisdiction `presentation_disclosure` text is surfaced to the consumer.
- **Required conformance test.** Conformance suite verifies that cross-jurisdiction requests without the explicit parameter are rejected; verifies the disclosure surfaces with cross-jurisdiction matches.
- **Status: `partially-resolved`** — existing v0.8.1 cross-jurisdiction default + per-category metadata. **Per-jurisdiction financial-advice terminology canonicalisation is v0.9 work** via the canonical category-annex template.

### 9.4 Established-practitioner categories lack authoritative regulators

- **Scenario.** Categories where no single authoritative regulator exists (some coaching, advisory, or therapeutic professions) require verification approaches different from credentialled categories.
- **Abuse / failure mode.** Verification model designed for credentialled categories applied weakly or inappropriately to established-practitioner categories.
- **Expected protocol behaviour.** Per the two-tier model (`professional_tier` enum on professional identity schemas: `credentialled` and `established`): established-tier categories use a different verification shape — partner-verified body membership, CPD compliance, disciplinary record via partner data, optionally enhanced verification through approved independent verifiers. `band_definitions`: established categories use different threshold rules from credentialled (the established-tier defaults set `verification_confidence == multi_source_corroborated OR multi_source_with_enhanced_verification` for band position 1, with `single_source_membership OR single_source_self_declared` for band position 2).
- **Required audit events.** Verification mechanism is recorded per `_provenance` map on each professional record; partner-attributed data carries the partner identifier and `verification_status`.
- **Required user / professional disclosure.** The professional's tier, registration route, and verification provenance are surfaced through the match response and the `_provenance` map.
- **Required conformance test.** Conformance suite verifies that established-tier categories use the established-tier band definitions and that partner-attributed data carries full provenance.
- **Status: `resolved`** by the two-tier model + tier-templated `band_definitions` + provenance map. v0.9 may refine per-category band definitions for established categories, but the structural mechanism is in place in v0.8.2.

---

## Category 10 — Conformance Divergence


### 10.1 Third-party registry claims ATAH compatibility but omits transparency metadata

- **Scenario.** A third-party registry implements ATAH-compatible endpoints but omits or weakens transparency artifacts (decision-explanation, presentation-disclosure, visibility report).
- **Abuse / failure mode.** "ATAH compatible" claim made without the transparency posture that gives the protocol its accountability shape.
- **Expected protocol behaviour.** Per §11A.6 Transparency Class (the new fifth conformance class): every relevant response includes a valid response-level `decision_explanation` (Layer 1); every candidate in a Discovery response includes a valid per-candidate `decision_explanation` (Layer 2); every Discovery response with exclusions includes a valid `exclusion_summary` (Layer 3); the professional-facing visibility-explanation behaviour is implementable from the v0.8.2 spec. Per Charter Part Two hard-artifacts list: public conformance tests are part of the trust floor; revocable conformance mark is the enforcement mechanism. Per Charter Core Commitment 8: the protocol governance body holds the conformance marks and may revoke them.
- **Required audit events.** Conformance test failures produce reportable conformance-mark revocation triggers.
- **Required user / professional disclosure.** Conformance marks are publicly visible per `/.well-known/atah-conformance`; revocations are recorded in the public governance-decision register (v0.9 operational artifact).
- **Required conformance test.** Conformance suite verifies all five classes; an implementation missing the Transparency Class fails the test.
- **Status: `resolved`** by The protocol's Transparency Class + Charter Part Two hard-artifacts list (revocable conformance mark) + Charter Core Commitment 8 (protocol governance body holds the conformance marks).

### 10.2 Registry applies commercial ordering while preserving provenance

- **Scenario.** A third-party registry preserves provenance metadata correctly but applies commercial ordering, treating the protocol's stratified-randomisation rule as advisory rather than normative.
- **Abuse / failure mode.** Conformance gap exploited; ordering framing weakened across implementations.
- **Expected protocol behaviour.** Per §9.1 verbatim MUST NOT: ATAH may determine eligibility and exclusion, but MUST NOT express preference among eligible candidates unless the ordering basis is explicit, non-commercial, and disclosed. Per §9.6 commercial neutrality. Per §9.5 conformance: returned `band_assignment` must be consistent with `band_definitions` and within-band ordering must be observably non-deterministic across repeated queries. An implementation applying commercial ordering fails the structural conformance test.
- **Required audit events.** Conformance test failures produce reportable conformance-mark revocation triggers.
- **Required user / professional disclosure.** `presentation_disclosure.ordering_policy.commercial_weighting: false` is `const`-asserted at the schema level; an implementation cannot return a different value through a conformant response.
- **Required conformance test.** Conformance suite verifies non-determinism within bands across repeated queries; verifies `commercial_weighting: false` is const-asserted on every response.
- **Status: `resolved`** by Charter + stratified-randomisation rule + §9.5 conformance non-determinism test.

### 10.3 Implementation changes consent or withdrawal semantics

- **Scenario.** A conforming-by-name implementation alters consent-receipt scope semantics or withdrawal state-transition rules, producing user-visible behaviour that diverges from ATAH's documented semantics.
- **Abuse / failure mode.** Same protocol surface, different meanings; users and reviewers cannot rely on documented semantics across implementations.
- **Expected protocol behaviour.** (consent boundaries) + (withdrawal as state transition): consent receipt scope enum is closed (`consent_type` enum: `query_authorization`, `disclosure_consent`); withdrawal scenarios are six explicitly-documented patterns in §11.9; receipt continuity-binding fields enforce the binding. An implementation altering these semantics fails the Core Object Conformance class. Per Charter Part Two hard-artifacts list: public conformance tests + revocable conformance mark.
- **Required audit events.** Conformance test failures produce reportable conformance-mark revocation triggers.
- **Required user / professional disclosure.** Conformance marks publicly visible.
- **Required conformance test.** Conformance suite verifies the consent-type enum and the six withdrawal scenarios produce the expected outputs; verifies receipt continuity-binding mismatch produces `consent_continuity_mismatch` audit events.
- **Status: `resolved`** by schema-enforced semantics + Charter conformance test surface + revocable conformance mark.

### 10.4 Federation introduces inconsistent identity or audit handling

- **Scenario.** Federated implementations exchange handoffs across instances; identity resolution, audit-event recording, or principal-delegation handling diverges between participating implementations.
- **Abuse / failure mode.** Cross-instance audit gaps; handoffs that look conforming on each side but produce broken end-to-end accountability.
- **Expected protocol behaviour.** Federation is **deferred to v0.9 / v1.0** per spec §1.6. v0.8.x is single-registry. The architecture is federation-ready (atah_id namespaces are reserved, no part of the data model assumes single-registry operation), but the cross-registry trust mechanics — namespace resolution, handoff routing, audit reconciliation, federated identity — are not yet specified.
- **Required audit events.** Federation-specific audit semantics are part of the v1.0 work.
- **Required user / professional disclosure.** Federation is documented as deferred so reviewers don't expect a federation answer in v0.8.2.
- **Required conformance test.** Federation conformance test is part of the v1.0 conformance test suite.
- **Status: `deferred`** — `deferred-to-v1.0` (federation mechanics). The v0.8.2 architecture does not preclude federation; it does not provide federation. Documented in spec §1.6 and ROADMAP "v0.9 candidates" (federation mechanics).
