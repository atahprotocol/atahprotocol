# 13. Transparency as conformance, layered decision explanations, rules-derived professional-facing view

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

This decision draws on Paolo Piponi's peer review (May 2026). v0.8.1's transparency posture was real but fragmented. Provenance maps, presentation disclosures, match-factor metadata, visibility reports, audit logs, and commercial-neutrality commitments were all in the spec, but they were not expressed as a single hard protocol commitment. The reviewer's framing: "Transparency is one of ATAH's strongest potential differentiators from private recommendation engines... The concern is that these are fragmented. They should be expressed as a single hard protocol commitment: every meaningful protocol decision should be explainable."

Three review passes converged on the v0.8.2 design:

1. **F1.8 / F1.9 (Paolo).** Make transparency a top-level conformance requirement; introduce a structured `decision_explanation` object covering discovery, exclusion, ordering, handoff, withdrawal, suppression, and data-sharing events; surface the explanation role-specifically (consumers, professionals, platforms, governance/auditors, public).

2. **F-5 / F-6 (v2 review).** Two refinements: (F-5) the `decision-explanation.schema.json` defines the inner object only — re-wrapping it inside a `decision_explanation` field at schema level produces a double-nesting bug; (F-6) response-level explanation alone is insufficient; both response-level and per-candidate explanations are required; the professional-facing transparency obligation is part of v0.8.2 even where the dedicated endpoint implementation is deferred to v0.8.3.

3. **F-18 (v3 review).** The professional-facing visibility view is rules-derived, not query-history-derived. Real query patterns expose information about consumers, AI platform integrations, demand signals, matter types, urgency, and platform activity that does not belong in a per-professional report — even aggregated query counts can leak in low-volume categories. The existing `docs/professional/visibility-report.md` posture (which already excludes "aggregate query volume against the professional's profile") is the right one; v0.8.2 codifies it as a normative MUST NOT in the spec.

## Decision

v0.8.2 makes transparency a top-level conformance requirement, with verbatim normative wording adopted from Paolo (per the v0.8.2 standing instruction):

**Transparency-as-conformance MUST (§11A, Paolo verbatim).**

> Conforming implementations MUST produce machine-readable explanations for discovery, exclusion, ordering, handoff, withdrawal, suppression, and data-sharing events, subject to privacy, security, and anti-gaming limits.

A new fifth conformance class — **Transparency Class** — is added to `CONFORMANCE.md` alongside Core Object, Binding, Registry, and Governance. The Transparency Class requires both layers of `decision_explanation` plus the aggregate `exclusion_summary` plus the professional-facing visibility-explanation behaviour.

**`decision-explanation.schema.json` (F-5 inner-object-only).** Schema defining the explanation object directly. Required fields: `decision_type` (enum: `discovery_result`, `exclusion`, `ordering`, `handoff`, `withdrawal`, `suppression`, `data_sharing`), `rules_applied` (open array of rule identifiers; the canonical initial set ships with the schema description), `ordering_policy`, `authority_basis`, `audit_event_id`. Optional: `exclusion_reason_category` (for `decision_type: exclusion`), `withholding_basis` (default `no_withholding`). The schema description text declares the F-5 rule explicitly: "This schema defines the decision-explanation object DIRECTLY... the field name in the consuming response is `decision_explanation`, but this schema does NOT wrap its own contents in a `decision_explanation` key." Implementers MUST NOT re-wrap.

**Three transparency layers in match responses (F-6).**

- **Layer 1.** Response-level `decision_explanation` (required on `match-response.schema.json`).
- **Layer 2.** Per-candidate `decision_explanation` (required for every candidate in the results array). The Phase 5 placeholder is promoted to required and references `decision-explanation.schema.json`.
- **Layer 3.** Aggregate `exclusion_summary` with reason-category counts (required when there were exclusions). Names of excluded candidates are NOT surfaced through this object.

The three-layer model protects excluded candidates from leakage while still providing aggregate-level explainability. Named-excluded-candidate detail is available only through the dedicated auditor / governance endpoint `GET /v1/decision-explanations/{audit_event_id}` under `governance_admin_role` authority.

**Role-specific transparency (§11A.3).** Consumers and consumer agents see Layer 2 + Layer 3; do not see named excluded candidates. Professionals see their own visibility view (rules-derived per §11A.4). AI platforms receive all three layers; per §9.4, they MUST preserve the response-level disclosure when reordering downstream. Governance and auditors have deeper audit access via `GET /v1/decision-explanations/{audit_event_id}`. The public sees high-level rules, fee schedules, partner scopes, category annexes, and conformance status.

**Professional-facing transparency obligation (§11A.4, F-6 + F-18).** The protocol obligation is part of v0.8.2. The dedicated endpoint `GET /v1/professionals/me/visibility-explanations` (REST) and `get_my_visibility_explanations` (MCP) is fully specified in v0.8.2; implementations MAY declare `x-implementation-deferred-to: v0.8.3` and return 501. The view is rules-derived per F-18:

> Professional-facing visibility explanations MUST NOT expose actual query-count data, query-history data, observed demand patterns, or any per-query information derived from consumer or platform traffic. They MUST present representative explanation categories at the category/jurisdiction level, derived from the implementation's documented rules and the professional's own profile data, not from observed query traffic.

Required fields in the view: representative inclusion rules (which apply to the professional in this category/jurisdiction); representative exclusion reason categories (which could apply if conditions changed); the professional's current band assignment (derived from `band_definitions`, NOT from observed query traffic); the implementation's published ordering policy. Authority basis: `professional_delegated_token` or `firm_delegation`. Each retrieval is audit-logged. Endpoint rate-limited per professional account.

**Honest gap acknowledgement.** The rules-derived view tells a professional what would-or-would-not include them, not whether they were actually included in a real query. v0.8.2 explicitly does not provide that. v0.9 may revisit with privacy-preserving aggregate mechanisms (k-anonymity floors, differential privacy, or category-level aggregates with high-volume thresholds). v0.8.2's position is rules-derived only.

**Privacy / security / anti-gaming withholding (§11A.5).** Withholding is permitted only where disclosure would (a) leak personal data, (b) reveal a security vulnerability, or (c) enable gaming of the matching engine. The reason something happened MUST still be explainable at category level. Where a detail is withheld, `decision-explanation.schema.json` `withholding_basis` records the basis. Implementations declaring withholding on a high proportion of explanations are subject to conformance review.

**Schema-level transparency hooks.** `match-response.schema.json` requires `decision_explanation` at both layers and `exclusion_summary` when exclusions exist. `handoff-component2.schema.json` carries optional `decision_explanation` for lifecycle events. `audit-event.schema.json` carries optional reverse reference `decision_explanation_id` so the audit log can cross-reference the explanation it corresponds to.

## Alternatives considered

**Response-level explanation only.** Considered: include `decision_explanation` at the top of `match-response.schema.json` only; do not require per-candidate explanations. Rejected per F-6: response-level alone is insufficient because the protocol cannot answer "why this specific candidate appeared" without per-candidate explanations. The two-layer model (plus aggregate exclusion summary) is the F-6 minimum.

**Full implementation in v0.8.2 including the professional-facing endpoint.** Preferred but accepted with v0.8.3 deferral as fallback. The F-6 reviewer constraint: spec defines the required behaviour, fields, authority controls, and audit linkage in v0.8.2 even if the implementation may follow in v0.8.3. v0.8.2 satisfies this — the OpenAPI endpoint is fully specified; the MCP tool is fully specified; the F-18 MUST NOT rule is in the spec; the obligation is part of v0.8.2 regardless of which release implements the endpoint.

**Named-excluded-candidate visibility to consumers.** Considered: surface excluded-candidate names alongside the included candidates so consumers can see "X was excluded for jurisdiction mismatch." Rejected on privacy grounds: this leaks excluded professionals' identities and their match-status to a consumer who didn't ask. The aggregate `exclusion_summary` provides explainability without leakage; named detail is reserved for governance / auditor retrieval.

**Actual query history with k-anonymity threshold (F-18 alternative).** Considered: relax the rules-derived constraint and allow actual query data above a k-anonymity threshold (e.g. only show category-level aggregates where N ≥ 50). Rejected as more complex, harder to specify, and still creating low-volume edge cases (e.g. specialism + jurisdiction combinations that fall below threshold). The rules-derived approach is a simpler structural rule that doesn't require threshold tuning.

**Real aggregate query counts at category level only.** Considered: show category-level aggregates of actual query traffic to the professional. Rejected as still leaking demand information for low-volume jurisdictions or specialisms. Even "lawyers in Wyoming, county court matters" is leakable when query counts are small.

**Inlining `decision_explanation` content directly into the response schema.** Considered: define the explanation fields inline on `match-response.schema.json` rather than a separate schema. Rejected: the explanation is reused across discovery, handoff, and withdrawal responses; a separate schema avoids duplication and makes the F-5 inner-object discipline easier to enforce.

**Re-wrapping `decision_explanation` inside the schema (F-5 anti-pattern).** Considered: define `decision-explanation.schema.json` as `{ "decision_explanation": { ... } }` so the schema is self-documenting. Rejected per F-5 — this produces double-nesting when the schema is referenced from a response schema using `decision_explanation` as a field name. The schema description text now explicitly warns implementers against re-wrapping.

## Consequences

- Every relevant response shape carries `decision_explanation` (required on `match-response.schema.json` at both layers; optional on `handoff-component2` for lifecycle events). Conformance verification includes a check that every response shape that should have it does have it.
- The `rules_applied` array is open-ended (an array of strings, not an enum). v0.8.2 ships the canonical initial set in the schema description; implementations MAY extend; the field is open so v0.8.3 / v0.9 can add canonical values without a breaking schema change.
- The `decision_type` enum is closed for v0.8.2; new types require a schema-version bump.
- Privacy / security / anti-gaming withholding has a structured `withholding_basis` field. Implementations declaring withholding on a high proportion of explanations are subject to conformance review.
- The professional-facing endpoint is one of the v0.8.3-deferred implementation items. The protocol obligation remains in force at v0.8.2; only the endpoint implementation may defer.
- The existing `docs/professional/visibility-report.md` posture is preserved and codified as a normative MUST NOT in spec §11A.4. Implementations cannot use observed query traffic to populate the professional-facing view; the view is derived from `band_definitions`, the professional's own profile, and the implementation's published rules.
- The honest gap (rules-derived view doesn't tell professionals whether they were actually included in real queries) is documented in §11A.4 and EXPLAINER. v0.9 may revisit; v0.8.2 doesn't pretend the gap doesn't exist.
- The conformance test suite (v0.9) verifies all five conformance classes including the new Transparency Class. The class definition in `CONFORMANCE.md` lists the requirements concretely so the test surface is well-defined.

## References

- Spec §11A (full new section: §11A.1 what an explanation answers; §11A.2 three transparency layers; §11A.3 role-specific transparency; §11A.4 professional-facing visibility-explanation obligation with verbatim Paolo MUST and verbatim F-18 MUST NOT; §11A.5 privacy / security / anti-gaming withholding; §11A.6 conformance class definition).
- Schema: [`decision-explanation.schema.json`](../../spec/v0.8/schemas/decision-explanation.schema.json) — new, F-5 inner-object-only with explicit description-text warning.
- Schema updates: [`match-response.schema.json`](../../spec/v0.8/schemas/match-response.schema.json) (Layer 1 + Layer 2 + Layer 3 — `decision_explanation` required at both layers; `exclusion_summary` added); [`handoff-component2.schema.json`](../../spec/v0.8/schemas/handoff-component2.schema.json) — optional `decision_explanation` for lifecycle events; [`audit-event.schema.json`](../../spec/v0.8/schemas/audit-event.schema.json) — optional reverse reference `decision_explanation_id`.
- OpenAPI: `GET /v1/decision-explanations/{audit_event_id}` (governance retrieval); `GET /v1/professionals/me/visibility-explanations` (professional self, fully specified, implementation may defer to v0.8.3 via `x-implementation-deferred-to`).
- MCP tools: `get_my_visibility_explanations`.
- [`CONFORMANCE.md`](../../CONFORMANCE.md) — Transparency Class added as the fifth class.
- Charter Part Two operational commitments mirror the §11A normative rules.
- [`docs/professional/visibility-report.md`](../professional/visibility-report.md) — existing posture cross-referenced; the "aggregate query volume" exclusion is preserved and tightened as a citation of the spec MUST NOT.
- ROADMAP.md — new "v0.8.3 candidates" section listing the deferred professional-facing endpoint implementation.
- Master patch plan v3.1.1, Phase 6 (§8) and Paolo's findings F1.8 (transparency as conformance, verbatim), F1.9 (decision_explanation as first-class artefact); F-5 (inner object only); F-6 (two-layer + aggregate; obligation defined in v0.8.2 even if endpoint is v0.8.3); F-18 (rules-derived professional-facing view, verbatim MUST NOT).
- Source review: `PAOLO-PEER-REVIEW-FINDINGS-01-v0_8_1.md` and `PAOLO-transparency-and-explainability.md`.
