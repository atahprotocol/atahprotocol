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
- **Phase 1 contribution.** `authenticated_actor` + `client_application` + `authority_context` shape established; per-operation authority-basis validation enforced via §7.3 matrix and OpenAPI per-operation `security` blocks. **Status: `partially-resolved`** — Phase 1 closes the structural side (declaration + binding); Phase 2's server-side `request_intent` validation closes the semantic side. Will become `resolved` after Phase 2.

### 1.2 Platform submits consent for a user who did not consent

- **Scenario.** An AI platform presents a consent receipt at a Stage 2 / Stage 3 boundary for a consumer who did not perform the required consent ceremony.
- **Abuse / failure mode.** Fabricated or recycled consent; downstream professionals receive data on a false consent basis.
- **Expected protocol behaviour.** _To be filled in during Phase 4._
- **Required audit events.** _To be filled in during Phase 4._
- **Required user / professional disclosure.** _To be filled in during Phase 4._
- **Required conformance test.** _To be filled in during Phase 4._
- **Phase 1 contribution.** `authority_context` records the basis the asserting platform claims; `consent-receipt.schema.json` `asserted_by` now uses the principal-delegation `AuthenticatedActor` + `ClientApplication` $defs so the asserter identity is recorded with the same shape ATAH validates everywhere else. **Status: `partially-resolved`** — Phase 1 records the basis; Phase 4 adds the continuity binding (`client_id`, `pseudonymous_consumer_ref`, `data_categories_hash`) that detects replay and cross-user confusion. Per F-17 the eventual settled status is `bounded-by-protocol`: ATAH verifies receipt integrity (tampering, replay, mismatch) but cannot prove the platform's underlying consent ceremony was valid.

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

---

## Category 2 — Consent Boundary Failure

**Phase mapping:** Phase 4.

### 2.1 ATAH handoff consent is misrepresented as engagement consent

- **Scenario.** A Stage 2 or Stage 3 handoff consent is presented to a downstream professional or surface as if it were ongoing engagement consent (e.g. authorisation to retain, contact further, or open a relationship).
- **Abuse / failure mode.** Scope creep at the consent boundary; consumer's narrow authorisation expanded into broader permissions.
- **Expected protocol behaviour.** _To be filled in during Phase 4._
- **Required audit events.** _To be filled in during Phase 4._
- **Required user / professional disclosure.** _To be filled in during Phase 4._
- **Required conformance test.** _To be filled in during Phase 4._
- **Status.** `skeleton`

### 2.2 User consents to one professional but data is sent to another

- **Scenario.** Consumer consents at Stage 2 / Stage 3 in respect of a specific candidate; the platform routes data to a different professional (substituted, additional, or the wrong record).
- **Abuse / failure mode.** Mismatch between consented scope and actual recipient; consent receipt's `data_categories_hash` and / or recipient binding does not match the delivery target.
- **Expected protocol behaviour.** _To be filled in during Phase 4._
- **Required audit events.** _To be filled in during Phase 4._
- **Required user / professional disclosure.** _To be filled in during Phase 4._
- **Required conformance test.** _To be filled in during Phase 4._
- **Status.** `skeleton`

### 2.3 Consent text differs from stored receipt metadata

- **Scenario.** What the consumer was shown at the consent ceremony does not match the metadata recorded on the stored consent receipt (different scope, recipients, data categories, or duration).
- **Abuse / failure mode.** Drift between displayed and recorded consent; later disputes are unresolvable on protocol grounds.
- **Expected protocol behaviour.** _To be filled in during Phase 4._
- **Required audit events.** _To be filled in during Phase 4._
- **Required user / professional disclosure.** _To be filled in during Phase 4._
- **Required conformance test.** _To be filled in during Phase 4._
- **Status.** `skeleton`

### 2.4 Consent is revoked after Stage 2 or Stage 3

- **Scenario.** Consumer revokes consent after data has already been delivered to a downstream professional or surface.
- **Abuse / failure mode.** Ambiguous semantics for "consent revoked after delivery" — what must downstream parties do, and what does ATAH itself record?
- **Expected protocol behaviour.** _To be filled in during Phase 4._
- **Required audit events.** _To be filled in during Phase 4._
- **Required user / professional disclosure.** _To be filled in during Phase 4._
- **Required conformance test.** _To be filled in during Phase 4._
- **Status.** `skeleton`

---

## Category 3 — Ordering and Recommendation Drift

**Phase mapping:** Phase 5.

### 3.1 Candidates are always returned in the same order

- **Scenario.** Implementations return the same candidate ordering for substantially equivalent queries, leading to entrenched advantage for consistently top-listed professionals.
- **Abuse / failure mode.** Ordering becomes a de facto recommendation; stratified-randomisation requirement bypassed in practice.
- **Expected protocol behaviour.** _To be filled in during Phase 5._
- **Required audit events.** _To be filled in during Phase 5._
- **Required user / professional disclosure.** _To be filled in during Phase 5._
- **Required conformance test.** _To be filled in during Phase 5._
- **Status.** `skeleton`

### 3.2 Hidden score influences order despite non-recommendation claims

- **Scenario.** Implementation publishes "no global match score" framing while internally computing a score and using it to bias ordering.
- **Abuse / failure mode.** Recommendation in disguise; the published ordering policy does not reflect the actual ranking basis.
- **Expected protocol behaviour.** _To be filled in during Phase 5._
- **Required audit events.** _To be filled in during Phase 5._
- **Required user / professional disclosure.** _To be filled in during Phase 5._
- **Required conformance test.** _To be filled in during Phase 5._
- **Status.** `skeleton`

### 3.3 Commercial partner candidates appear disproportionately

- **Scenario.** Professionals associated with commercial partners (paying or otherwise) appear in eligible result sets at frequencies inconsistent with their share of the eligible population.
- **Abuse / failure mode.** Commercial bias entering ordering through a back channel — band assignment, eligibility filters, or stratification weights — without disclosure.
- **Expected protocol behaviour.** _To be filled in during Phase 5._
- **Required audit events.** _To be filled in during Phase 5._
- **Required user / professional disclosure.** _To be filled in during Phase 5._
- **Required conformance test.** _To be filled in during Phase 5._
- **Status.** `skeleton`

### 3.4 AI platform presents ATAH result one as "best"

- **Scenario.** ATAH returns a stratified set with a published ordering policy; the AI platform reorders, summarises, or labels the first result as "best" or "top recommendation" downstream.
- **Abuse / failure mode.** Platform-side presentation undermines ATAH's stratification and equal-treatment posture; users experience a recommendation despite the protocol's stance.
- **Expected protocol behaviour.** _To be filled in during Phase 5._
- **Required audit events.** _To be filled in during Phase 5._
- **Required user / professional disclosure.** _To be filled in during Phase 5._
- **Required conformance test.** _To be filled in during Phase 5._
- **Status.** `skeleton` *(expected to settle as `allocated-to-platform-responsibility` per F-17 — ATAH publishes ordering policy in `presentation_disclosure`; downstream presentation is the platform's obligation.)*

---

## Category 4 — Transparency and Explainability Failure

**Phase mapping:** Phase 6.

### 4.1 Professional asks why they were excluded

- **Scenario.** A verified professional, expecting to appear in eligible result sets, queries the protocol about exclusion reasons.
- **Abuse / failure mode.** No mechanism for professional-facing transparency; or the mechanism leaks query-history / volume data inappropriately.
- **Expected protocol behaviour.** _To be filled in during Phase 6._ Per F-18: protocol response MUST surface representative explanation categories at category / jurisdiction level, derived from documented rules — MUST NOT expose actual query-count or query-history data.
- **Required audit events.** _To be filled in during Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 6._
- **Required conformance test.** _To be filled in during Phase 6._
- **Status.** `skeleton`

### 4.2 Consumer asks why a candidate appeared

- **Scenario.** A consumer (via the AI platform) asks why a particular professional was returned in the eligible set, or why one candidate appeared above another in the stratified ordering.
- **Abuse / failure mode.** Decision basis is opaque; ordering looks like a recommendation; consumer trust depends on a transparent rationale.
- **Expected protocol behaviour.** _To be filled in during Phase 6._
- **Required audit events.** _To be filled in during Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 6._
- **Required conformance test.** _To be filled in during Phase 6._
- **Status.** `skeleton`

### 4.3 Auditor asks whether commercial weighting affected results

- **Scenario.** A regulator, conformance auditor, or governance reviewer asks whether commercial relationships influenced eligibility, ordering, or presentation in a given response.
- **Abuse / failure mode.** Without per-response transparency artifacts, the question is unanswerable from the protocol record alone.
- **Expected protocol behaviour.** _To be filled in during Phase 6._
- **Required audit events.** _To be filled in during Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 6._
- **Required conformance test.** _To be filled in during Phase 6._
- **Status.** `skeleton`

### 4.4 Partner data conflict suppresses a profile and the professional appeals

- **Scenario.** A trusted-partner data feed reports a conflict (suspended licence, sanction, complaint) that suppresses or downgrades a professional's appearance; the professional appeals on the grounds the conflict record is wrong.
- **Abuse / failure mode.** Partner errors propagate without correction path; or the appeal mechanism is opaque.
- **Expected protocol behaviour.** _To be filled in during Phase 6._
- **Required audit events.** _To be filled in during Phase 6._
- **Required user / professional disclosure.** _To be filled in during Phase 6._
- **Required conformance test.** _To be filled in during Phase 6._
- **Status.** `skeleton`

---

## Category 5 — Vault, Erasure, and Audit Abuse

**Phase mapping:** Phase 3.

### 5.1 Payload is erased but abuse investigation is needed

- **Scenario.** A handoff payload is erased per protocol's erasure rules; subsequently a complaint, dispute, or regulator request requires reconstruction or evidence of what was sent.
- **Abuse / failure mode.** Tension between privacy (erasure) and accountability (investigation). Three-concept separation must keep audit-event metadata viable after payload erasure.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

### 5.2 Plaintext appears in logs or notification providers

- **Scenario.** Despite vault and crypto-erasure architecture, plaintext (consumer query content, professional contact details, decision rationale) leaks into application logs, notification provider records, or third-party telemetry.
- **Abuse / failure mode.** Privacy posture undermined at the operational margin; secondary persistence stores defeat the primary erasure design.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

### 5.3 Key deletion is not auditable

- **Scenario.** Crypto-erasure relies on key destruction; deletion of the key itself is performed but produces no auditable event, so erasure cannot be verified after the fact.
- **Abuse / failure mode.** Erasure is a claim, not a verifiable fact; conformance and dispute resolution cannot rely on it.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

### 5.4 Retrieval token is reused or leaked

- **Scenario.** A retrieval token issued for single-use access to a payload is reused, replayed by an attacker, or leaks into a system where it is consumed by an unintended party.
- **Abuse / failure mode.** Retrieval becomes effectively multi-use; downstream parties access content without distinct authority.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

---

## Category 6 — Withdrawal Abuse

**Phase mapping:** Phase 3.

### 6.1 Malicious actor initiates and withdraws repeated handoffs

- **Scenario.** An attacker or harasser initiates handoffs to a target professional and withdraws each one before completion, exploiting the withdrawal mechanism for harassment, denial-of-service, or reputational signal noise.
- **Abuse / failure mode.** Withdrawal as harassment vector; rate-limit gaps and lack of immutable metadata enable repeat abuse.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

### 6.2 Professional withdraws profile after concern flags

- **Scenario.** A professional withdraws their ATAH profile (or a critical part of it) immediately after concern flags are raised, attempting to evade ongoing scrutiny.
- **Abuse / failure mode.** Withdrawal as evidence escape; without immutable audit metadata of the flags and the withdrawal, future re-registration could erase prior signals.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

### 6.3 Consumer withdraws after data retrieval and expects data to be unseen

- **Scenario.** Consumer withdraws after the professional has already retrieved the handoff payload and expects this to render the data "unseen" or unusable.
- **Abuse / failure mode.** Mismatch between the consumer's mental model of withdrawal and what protocol withdrawal can actually achieve once data has been delivered to a third party's own systems.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

### 6.4 Delegated agent withdraws a professional from matching after account compromise

- **Scenario.** A delegated agent token, compromised by an attacker, is used to withdraw the principal professional from matching (or modify their profile to render them ineligible).
- **Abuse / failure mode.** Compromise of one credential class causes business damage to the principal; recovery semantics unclear.
- **Expected protocol behaviour.** _To be filled in during Phase 3._
- **Required audit events.** _To be filled in during Phase 3._
- **Required user / professional disclosure.** _To be filled in during Phase 3._
- **Required conformance test.** _To be filled in during Phase 3._
- **Status.** `skeleton`

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

Phase 11 finalises the matrix as the verification artifact for v0.8.2 publication.
