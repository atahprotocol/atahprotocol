# 9. Principal and Delegation Model

- Status: Accepted
- Date: 2026-05 (v0.8.2)

## Context

ATAH protocol actions need to express trust boundaries that a flat "actor" record cannot:

- Distinguish the **authenticated party** from the **principal whose authority is being exercised**. A consumer-mediated AI platform call and a professional-delegated agent call both involve a platform_id / client_id, but the principal differs.
- Record the **credential class** establishing the authority. The same actor type can act under different bases (a professional via session, a professional via delegated agent token) with materially different trust implications.
- Record **object-level constraints**. A handoff_access_token grants narrow authority over a specific handoff, not blanket authority over all handoffs of the same actor.
- Give downstream parties (audit, dispute, conformance) a uniform shape to reason about. Without a shared shape each schema invents its own actor block.

This decision draws on Paolo Piponi's peer review (May 2026). The review identified the principal/delegation model as the foundational architectural change (F1.1 in the consolidated peer-review findings). A structural refinement (F-2) was added: the model must not be mashed into a single combined actor object on `audit-event.schema.json`, because partner / verifier / system events would then be forced to look like AI-platform actions (with required `platform_id` / `client_id` fields they do not have).

## Decision

ATAH defines a single shared schema, `principal-delegation.schema.json`, containing three named sub-objects under `$defs`:

- **`AuthenticatedActor`** — the directly authenticated party. `actor_type` ∈ {`ai_platform`, `professional`, `partner`, `verifier`, `admin`, `system`}; `actor_id` opaque. Consumers are not a valid `actor_type` (per F-16); consumer-mediated calls authenticate as `ai_platform` and identify the consumer in `AuthorityContext.represented_principal_type`.
- **`ClientApplication`** — platform/client identification. Required when `AuthenticatedActor.actor_type` is `ai_platform`; optional otherwise.
- **`AuthorityContext`** — the delegation context: `represented_principal_type` ∈ {`consumer`, `professional`, `partner`, `verifier`, `governance_admin`, `none`}; `authority_basis` ∈ {`user_session`, `professional_delegated_token`, `partner_credential`, `handoff_access_token`, `governance_admin_role`, `system_role`}; optional `permitted_scopes`, `expires_at`, and `object_constraints`.

The top-level shape composes the three: `authenticated_actor` and `authority_context` always required, `client_application` conditionally required (enforced via JSON Schema `if/then` keyed on `authenticated_actor.actor_type == "ai_platform"`).

Schemas using the shared shape:

- `query.schema.json` — `requesting_agent` `$ref`s `principal-delegation.schema.json`.
- `handoff-component2.schema.json` — same.
- `consent-receipt.schema.json` — `asserted_by` defined as a composite of `AuthenticatedActor` + (conditionally) `ClientApplication` from the shared `$defs`. Authority for the consent ceremony lives in the receipt's `scope` field; ATAH verifies receipt integrity, not the validity of the underlying consent ceremony.
- `audit-event.schema.json` — three top-level fields (`authenticated_actor`, `client_application`, `authority_context`) referencing the shared `$defs`. `client_application` is conditionally required for `ai_platform` events.

Spec §4.9A documents the model in prose. Spec §7.3 (authorisation matrix) expresses permissions per `authenticated_actor.actor_type` × `authority_context.represented_principal_type` × `authority_basis` × OAuth scope × object-level constraint, grouped by actor type. OpenAPI `securitySchemes.oauth2.description` documents the authority-basis-to-OAuth-scope mapping. MCP tools declare `permitted_authority_bases` per tool.

## Alternatives considered

**Single combined actor object.** Considered. Rejected per F-2 because it would have forced every audit-event actor record to declare `platform_id` and `client_id` (required fields), breaking partner / verifier / system events that have no client application. The three-object split is the structural fix.

**Per-actor-type schemas (one schema per actor_type).** Considered for greater type-specificity. Rejected: excessive proliferation (six schemas), no real benefit over JSON Schema `if/then` conditional requirements on a single composite, and significant cross-schema reference burden.

**Adding `consumer` to `actor_type`.** Considered for consumer-triggered events. Rejected per F-16: consumers do not authenticate directly to ATAH, and adding `consumer` to `actor_type` would contradict the principal/delegation model this ADR establishes. Consumer-triggered events are recorded with `authenticated_actor.actor_type: ai_platform` and `authority_context.represented_principal_type: consumer`.

## Consequences

- Every schema that records actor or asserter information references `principal-delegation.schema.json` (`query`, `handoff-component2`, `consent-receipt`, `audit-event`). Schemas added later that record actor information adopt the same model.
- The §7.3 authorisation matrix is grouped per `authenticated_actor.actor_type`. The structure scales by adding rows.
- OAuth-scope-to-authority-basis mapping is documented in `openapi.yaml`'s `securitySchemes.oauth2.description`. Where a clean mapping does not exist, it is documented explicitly in §13 rather than fudged.
- The `if/then` conditional for `client_application` requires JSON Schema 2020-12 implementations that handle conditional validation correctly. The `examples` arrays on `principal-delegation.schema.json` and `audit-event.schema.json` cover both branches (with and without `client_application`) so validation behaviour is exercised.
- The professional-facing audit traceability includes the `authority_basis` under which an action was taken — important for disputes.

## References

- Spec §4.9A (Principal and Delegation Model), §7.3 (Authorisation Matrix), §13.3 (Rate Limiting and Abuse Prevention).
- Schemas: [`principal-delegation.schema.json`](../../spec/v0.8/schemas/principal-delegation.schema.json), and the four consumers (`query`, `handoff-component2`, `consent-receipt`, `audit-event`).
- OpenAPI: `spec/v0.8/openapi.yaml` `components.securitySchemes.oauth2.description`.
- MCP tools: `spec/v0.8/mcp-tools.json` `tools[*].permitted_authority_bases`.
- Findings: F1.1 (principal/delegation model), F1.2 (recommended permission posture), F-2 (audit actor split), F-16 (no `consumer` in actor_type).
