# 1. Protocol / Bindings / Registry Operation Profile split

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

ATAH could be presented as a single artefact: "the ATAH service" — one
registry, one set of endpoints, one MCP server, all conflated. Early drafts
read that way in places.

The trouble is that ATAH is asking for a lot from very different audiences:
AI platforms must trust the matching engine, regulators must accept the
verification model, professionals must accept the data lifecycle, and future
implementers must be able to interoperate. None of those audiences will
accept "ATAH is a service we run" as the answer to "what is the standard?".

We had three choices:

1. Spec as MCP-only — define the protocol as a set of MCP tools and call it
   done.
2. Spec as a central-database service — describe the ATAH-operated registry
   in detail and make conformance equal to "talks to that registry".
3. Spec as a transport-neutral protocol with named conformance classes,
   with the ATAH-operated registry as the first implementation.

## Decision

We adopted (3). The specification is partitioned into three layers:

- **ATAH Core Protocol** — the transport-neutral object model and consent /
  introduction / outcome semantics. Required for any conforming
  implementation that produces or consumes ATAH objects.
- **Transport bindings** — the MCP binding (`mcp-tools.json`) and the REST
  binding (`openapi.yaml`) are two of the v0.8 bindings. Additional
  bindings may be defined in future. A binding is conformant if it
  preserves the protocol's mandatory semantics.
- **Registry Operation Profile** — what a conforming registry stores,
  scores, and returns. Required for implementations claiming Registry
  Conformance.

The ATAH-operated registry is the **reference registry**. It is one
conforming implementation; the specification does not require any other
implementation to be that registry.

## Alternatives considered

- **Spec as MCP-only.** Rejected. MCP is one transport. The protocol's
  consent and lifecycle semantics matter regardless of transport. Locking
  the spec to MCP would have made the protocol look like an MCP-vendor
  product rather than a protocol.
- **Spec as a central-database service.** Rejected. This makes ATAH a
  proprietary referral database with a standards label on it. Standards
  reviewers would not accept conformance defined as "calls our service".
  Federation and alternative registries would be effectively impossible.

## Consequences

- The conformance text (spec §14) is structured around four classes — Core
  Object, Binding, Registry, Governance — which map to the layers above
  plus governance commitments.
- Every schema's `description` field labels its class explicitly. Every
  binding artefact's description identifies it as a binding.
- README, EXPLAINER, and Charter v1.2 use protocol-level language; "the
  reference registry" is the term used when reference-implementation
  behaviour is being described, not "ATAH".
- The architecture is federation-ready (atah_id namespace, no
  single-registry assumption in the data model) even though federation
  mechanics are deferred to v0.9.

## References

- Spec §1.5 (Protocol vs implementation), §14 (Conformance Classes)
- Charter v1.2: open specification commitment, royalty-free
- README "Three layers" section
