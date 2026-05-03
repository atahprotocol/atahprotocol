# 9. Principal and Delegation Model

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

v0.8.1 records actor information at a coarse level. The audit-event schema carries a single `actor` object with `actor_type` and `actor_id`; `query.schema.json` and `handoff-type1.schema.json` carry a flat `requesting_agent` with `platform_id` and `client_id`; `consent-receipt.schema.json` carries an `asserted_by` object that conflates authenticated identity with platform identification.

This is enough to know roughly who acted, but not enough to express the trust boundaries the protocol depends on:

- It does not distinguish the **authenticated party** from the **principal whose authority is being exercised**. A consumer-mediated AI platform call and a professional-delegated agent call both look like "platform_id, client_id" pairs.
- It does not record the **credential class** establishing the authority. The same actor type can act under different bases (a professional via session, via delegated agent token, via firm delegation) with materially different trust implications.
- It does not record **object-level constraints**. A handoff_access_token grants narrow authority over a specific handoff, not blanket authority over all handoffs of the same actor.
- It does not give downstream parties (audit, dispute, conformance) a uniform shape to reason about. Each schema invented its own actor block.

Paolo's review identified this as the foundational v0.8.2 architectural change (F1.1 in the consolidated peer-review findings). The v2 review added a refinement (F-2): the model must not be mashed into a single combined actor object on `audit-event.schema.json`, because partner / verifier / system events would then be forced to look like AI-platform actions (with required `platform_id` / `client_id` fields they do not have).

## Decision

v0.8.2 introduces a single shared schema, `principal-delegation.schema.json`, containing three named sub-objects under `$defs`:

- **`AuthenticatedActor`** — the directly authenticated party. `actor_type` ∈ {`ai_platform`, `professional`, `partner`, `verifier`, `admin`, `system`}; `actor_id` opaque. Consumers are not a valid `actor_type` (per F-16); consumer-mediated calls authenticate as `ai_platform` and identify the consumer in `AuthorityContext.represented_principal_type`.
- **`ClientApplication`** — platform/client identification. Required when `AuthenticatedActor.actor_type` is `ai_platform`; optional otherwise.
- **`AuthorityContext`** — the delegation context: `represented_principal_type` ∈ {`consumer`, `professional`, `firm_admin`, `partner`, `verifier`, `governance_admin`, `none`}; `authority_basis` ∈ {`user_session`, `professional_delegated_token`, `firm_delegation`, `partner_credential`, `handoff_access_token`, `governance_admin_role`, `system_role`}; optional `permitted_scopes`, `expires_at`, and `object_constraints`.

The top-level shape composes the three: `authenticated_actor` and `authority_context` always required, `client_application` conditionally required (enforced via JSON Schema `if/then` keyed on `authenticated_actor.actor_type == "ai_platform"`).

Schemas updated to use the shared shape:

- `query.schema.json` — `requesting_agent` now `$ref`s `principal-delegation.schema.json`.
- `handoff-type1.schema.json` — same.
- `consent-receipt.schema.json` — `asserted_by` redefined as a composite of `AuthenticatedActor` + (conditionally) `ClientApplication` from the shared `$defs`. Authority for the consent ceremony lives in the receipt's `scope` field; ATAH verifies receipt integrity, not the validity of the underlying consent ceremony.
- `audit-event.schema.json` — the v0.8.1 generic `actor` object is replaced by three top-level fields (`authenticated_actor`, `client_application`, `authority_context`) referencing the shared `$defs`. `client_application` is conditionally required for `ai_platform` events.

Spec §4.9A documents the model in prose. Spec §7.3 (authorisation matrix) is refactored to express permissions per `authenticated_actor.actor_type` × `authority_context.represented_principal_type` × `authority_basis` × OAuth scope × object-level constraint, grouped by actor type. OpenAPI `securitySchemes.oauth2.description` now documents the authority-basis-to-OAuth-scope mapping. MCP tools declare `permitted_authority_bases` per tool.

## Alternatives considered

**Single combined actor object.** v1 of the master patch plan proposed mashing the new model into the existing `actor` object. This was rejected per F-2 because it would have forced every audit-event actor record to declare `platform_id` and `client_id` (required fields), breaking partner / verifier / system events that have no client application. The three-object split is the structural fix.

**Per-actor-type schemas (one schema per actor_type).** Considered for greater type-specificity. Rejected: excessive proliferation (six schemas), no real benefit over JSON Schema `if/then` conditional requirements on a single composite, and significant cross-schema reference burden.

**Adding `consumer` to `actor_type`.** v1 proposed this to address `_schema_delta_report.md` item 15 (consumer-triggered events). Reversed in v3 per F-16: consumers do not authenticate directly to ATAH, and adding `consumer` to `actor_type` would contradict the principal/delegation model this ADR establishes. Consumer-triggered events are recorded with `authenticated_actor.actor_type: ai_platform` and `authority_context.represented_principal_type: consumer`. The v0.8.1 ROADMAP entry proposing the addition is closed as superseded (Phase 8 ROADMAP update).

## Consequences

- Every schema that previously inlined actor or asserter information references `principal-delegation.schema.json` (`query`, `handoff-type1`, `consent-receipt`, `audit-event`). Phase 1 covers these. Schemas added or refactored in later phases adopt the same model where they record actor information (Phase 2's renamed Component 3 record will add a principal-delegation reference for the initiating professional; Phase 3's withdrawal records will use it for the withdrawing actor; Phase 4's continuity-binding additions live alongside it on the receipt).
- `audit-event.schema.json` is a breaking schema change relative to v0.8.1. Since v0.8.1 is not published, no migration is required externally; internal test fixtures using the old `actor` shape must be updated.
- The §7.3 authorisation matrix is materially larger and is grouped per `authenticated_actor.actor_type`. The structure scales by adding rows; later phases that add operations append rows under the appropriate actor type.
- OAuth-scope-to-authority-basis mapping is documented in `openapi.yaml`'s `securitySchemes.oauth2.description`. Where a clean mapping does not exist, it is documented explicitly in §13 rather than fudged.
- The `if/then` conditional for `client_application` requires JSON Schema 2020-12 implementations that handle conditional validation correctly. The `examples` arrays on `principal-delegation.schema.json` and `audit-event.schema.json` cover both branches (with and without `client_application`) so validation behaviour is exercised.
- The professional-facing audit traceability now includes the `authority_basis` under which an action was taken — important for disputes (e.g. "this profile change was made under `firm_delegation`, not under the professional's own `professional_delegated_token`").

## References

- Spec §4.9A (Principal and Delegation Model), §7.3 (Authorisation Matrix), §13.3 (Rate Limiting and Abuse Prevention).
- Schemas: [`principal-delegation.schema.json`](../../spec/v0.8/schemas/principal-delegation.schema.json), and the four updated consumers (`query`, `handoff-type1`, `consent-receipt`, `audit-event`).
- OpenAPI: `spec/v0.8/openapi.yaml` `components.securitySchemes.oauth2.description`.
- MCP tools: `spec/v0.8/mcp-tools.json` `tools[*].permitted_authority_bases`.
- Master patch plan v3.1.1, Phase 1 (§3) and findings F-2 (audit actor split), F-16 (no `consumer` in actor_type).
- Source review: `PAOLO-PEER-REVIEW-FINDINGS-01-v0_8_1.md` F1.1 (principal/delegation model), F1.2 (recommended permission posture), and `PAOLO-authentication-and-consent-boundaries.md`.
