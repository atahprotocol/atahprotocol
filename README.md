# ATAH

**Agent to Authenticated Human Protocol**

An open protocol layer for AI-to-authenticated-human professional handoff. ATAH defines the trust, provenance, consent, and lifecycle contract that conforming registries and AI platforms can implement — exposed through MCP and REST bindings, with the ATAH reference registry as the first operational implementation.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Spec Version](https://img.shields.io/badge/spec-v0.8-orange.svg)](spec/v0.8/)
[![Status: Release Candidate](https://img.shields.io/badge/status-release_candidate-yellow.svg)](#status)

---

## The problem

AI systems can now book flights, buy products, complete transactions, and run multi-step workflows on behalf of users. But when an AI identifies that human professional expertise is genuinely needed — a lawyer for a contract, a financial advisor for a regulated decision, a PR consultant for a crisis, a structural engineer for a building concern — the journey breaks. The AI typically stops at a disclaimer and leaves the user with a search box.

That broken handoff is the gap ATAH is designed to fill.

## What ATAH is

ATAH is an open protocol layer that gives AI systems a structured, trusted way to connect users to credentialled and established human professionals when AI reaches the limit of what it can helpfully provide.

It is a **trust and handoff contract** — a common machine-readable standard defining the data model, consent receipts, provenance rules, handoff lifecycle, and conformance requirements for AI-to-authenticated-human professional handoff. The protocol is transport-compatible: it can be exposed through MCP tools, REST APIs, A2A-compatible agents, or future bindings. The v0.8 package defines MCP and REST bindings and an initial reference registry.

ATAH separates the standard from the service that first implements it. AI systems integrate with an ATAH-compatible endpoint and receive validated professional options together with transparent credential, provenance, and trust signals — supporting a clean handoff without leaving users stranded.

The protocol works for two categories of professional, through identical infrastructure:

- **Credentialled professionals** — those whose standing can be verified against formal licences, certifications, or regulatory records. Lawyers, property and casualty insurance agents, financial advisors, doctors, engineers, tax planners, accountants, architects.
- **Established professionals** — those whose standing is verifiable through professional body membership, peer recognition, and trusted partner data, even where formal licensing does not apply. PR specialists, management consultants, executive coaches, HR professionals, project managers, and many others.

Every data point in a profile carries its own verification status — registry-verified, partner-verified, VC-verified, or self-declared — returned transparently in every response via a parallel provenance map. AI systems receive granular trust signals and can communicate them clearly to users. ATAH is designed to make the basis of trust visible, not to collapse it into an opaque score.

Two routes exist for professionals to be represented in an ATAH-conformant registry: through their professional body when that body has joined as a trusted partner, or by individual self-registration where no partnered body covers them. Both routes coexist permanently, and matching treatment is identical between them. At launch, those records live in the ATAH reference registry; future conforming registries may exist once federation mechanics are specified.

The protocol supports three introduction types: consumer introductions (AI to professional), professional-to-professional referral relationships, and professional-initiated client referrals. All three use the same underlying infrastructure with structured consent receipts at every stage.

## What ATAH is *not*

- **Not a recommendation engine.** ATAH returns a provenance-visible shortlist based on declared need, verification evidence, category rules, and availability. The user or calling AI platform remains responsible for selection.
- **Not a regulator, enforcement body, or complaints adjudicator.** Professional conduct concerns are routed to the relevant regulatory or professional body.
- **Not a payment processor or marketplace.** No transaction or commerce capability.
- **Not a directory product or AEO replacement.** Professionals are not "promoted" or "ranked for visibility" in any commercial sense.
- **Not a personal data repository.** Consumer personal data passes through ATAH only as transient handoff data, held in a transient encrypted vault, and crypto-erased per protocol.
- **Not a consumer interface.** Consumers interact with AI systems, which use ATAH on their behalf.
- **Does not warrant outcomes.** ATAH commits to specific data freshness windows and dispute timelines, but does not warrant the suitability, competence, or behaviour of any individual professional.

## Three layers

Clarity about which layer is being discussed matters:

1. **The open specification.** The protocol itself, defined in this repository and published under Apache 2.0. Anyone may implement, fork, or extend it.
2. **Conforming implementations.** Software systems implementing the specification. Conformance requires preserving ATAH's mandatory principles: provenance visibility, no commercial weighting, staged consent, category compliance gating, and data minimisation.
3. **The initial reference registry.** ATAH operates an initial reference registry and MCP/REST endpoints that demonstrate and operate the protocol. The reference registry is *not* the protocol itself. As federation develops (deferred to v0.9 or v1.0), other conforming registries may exist, provided they preserve the conformance principles.

The initial registry exists because launching a protocol with no operational instance produces no real-world adoption. v0.8 is single-registry by design; the architecture is federation-ready.

## Implementation model

ATAH separates the standard from the service that first implements it.

The **ATAH Core Protocol** defines the data structures, consent receipts, provenance rules, handoff states, presentation disclosures, and conformance requirements.

The **ATAH bindings** define how those objects are exchanged. v0.8 defines MCP and REST bindings. Future versions may define A2A-compatible or federation bindings.

The **ATAH reference registry** is the first operational implementation. It stores professional records, ingests trusted partner data, runs matching, manages handoff lifecycle state, and exposes MCP/REST endpoints. It demonstrates and operates the protocol — but it is not a requirement that all ATAH records live in one database forever.

Future conforming registries may exist once federation and cross-registry trust mechanics are specified. Until then, v0.8 is single-registry by design and federation-ready by architecture.

## Where ATAH sits

ATAH is the missing layer above the existing agent-web stack and composes with established identity standards rather than replacing them:

| Layer | Purpose |
|---|---|
| **OAuth 2.1 / OIDC** | Authentication (used by ATAH for AI platforms and professionals) |
| **W3C Verifiable Credentials** | Cryptographically verifiable credential format (accepted by ATAH for partner data) |
| **MCP** (Model Context Protocol) | Tool and data access for AI agents |
| **A2A** (Agent2Agent Protocol) | Agent-to-agent communication |
| **ACP / UCP / AP2** | Agentic commerce and agent-mediated payments |
| **ATAH** | **AI-to-authenticated-human professional handoff** |

ATAH does not duplicate the work of identity, credential, or commerce standards. Its specific contribution is the layer above — professional categorisation, the staged introduction lifecycle, the matching engine, the trusted-partner trust model, and the commercial-neutrality and provenance-visibility commitments — none of which exist in the underlying standards.

## Who this is for

**AI platforms.** A common ATAH-compatible handoff contract, available through the v0.8 MCP and REST bindings, that returns validated professional options when your users need human expertise. No bespoke integration per category or jurisdiction. Free to query (no per-query or licensing fee — querying still requires authentication for protocol integrity). Conforms to current MCP authorisation guidance (OAuth 2.1-compatible). The initial reference registry provides the first live endpoint; future conforming implementations may expose the same protocol.

**Professional associations.** A practical answer to the question every member is asking: *how do I stay visible and trusted when AI becomes the first call for professional advice?* Associations participate as trusted partners — contributing what they already know about their members, and giving those members structured machine-readable presence in AI-mediated environments. Public partner fee schedules; waivers available for public-interest bodies.

**Credentialled and established professionals.** A way to be discoverable by AI systems without mastering Answer Engine Optimisation. No platform lock-in. Standing verified and kept current. Concern flag protections (admin-only visibility, right of reply).

**Trusted partners** — regulatory databases, professional bodies, review platforms, independent verifiers. A clean way to make verified data useful at the exact point users are deciding whether to trust a professional. Provenance always visible.

## Documentation

- **Read the spec:** [spec/v0.8/full-spec.md](spec/v0.8/full-spec.md) — full technical specification
- **Quick read:** [EXPLAINER.md](EXPLAINER.md) — plain-language summary
- **Charter:** [CHARTER.md](CHARTER.md) — founding governance commitments
- **Product requirements:** [PRD-v0_8.md](PRD-v0_8.md) — what ATAH is and what success looks like

## What's in this repository

- **[README.md](README.md)** — this file
- **[EXPLAINER.md](EXPLAINER.md)** — plain-language summary
- **[CHARTER.md](CHARTER.md)** — founding governance commitments
- **[PRD-v0_8.md](PRD-v0_8.md)** — product requirements
- **[GOVERNANCE.md](GOVERNANCE.md)** — current decision-making and transition plan
- **[CONTRIBUTING.md](CONTRIBUTING.md)** — how to propose changes
- **[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)** — community conduct standards
- **[SECURITY.md](SECURITY.md)** — security disclosure and threat model
- **[CHANGELOG.md](CHANGELOG.md)** — release history
- **[CHANGES-FROM-V0_7-TO-V0_8.md](CHANGES-FROM-V0_7-TO-V0_8.md)** — detailed change log between v0.7 and v0.8
- **[spec/v0.8/](spec/v0.8/)** — full technical specification, OpenAPI definition, MCP tool schemas, JSON Schema files
- **[press/](press/)** — launch announcements for media, professional associations, and AI platforms

## Status

**v0.8 — Release candidate.**

The v0.8 specification is intended to be implementable and is ready for technical review and reference implementation. It defines the core data model, introduction lifecycle, provenance model, matching principles, consent stages, and governance commitments. Machine-readable schemas, OpenAPI contracts, and the MCP tool definitions are included for review and will be hardened through reference implementation work.

The protocol moves to v1.0 once at least one trusted partner integration is live, the reference implementation has run end-to-end, and at least one external implementation has been built against the spec.

Breaking changes are possible until v1.0. The version negotiation and deprecation policy is defined in the spec.

**Note on maintenance.** ATAH is currently maintained by a small team with limited bandwidth. Published response times for security disclosures, governance objections, and review of contributions are best-effort targets and reflect that posture. The protocol's longer-term sustainability depends on transitioning to independent governance and on partners and implementers contributing to the work. Anyone interested in supporting the project — whether as a partner, implementer, reviewer, or volunteer maintainer — is encouraged to get in touch.

## Governance

ATAH is founded by [Grahame Cohen](https://github.com/grahamecohen). The protocol operates under the founding [CHARTER](CHARTER.md), which is intended to operate as a public governance covenant binding the founder to administer ATAH consistently with its commitments.

It is the founding commitment that governance of the protocol will transition to an independent structure — a not-for-profit, foundation, or equivalent — when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated. The trigger framework reflects that the protocol works with cross-industry organisations whose decision cycles are inherently slower than software-industry timelines, and that independent governance is appropriate when there is an ecosystem to govern rather than at an arbitrary calendar point.

An interim advisory group will be convened when suitable members are identified and available to serve. The status of governance transition will be reviewed and reported publicly when there is meaningful progress to share.

The Charter's permanent core commitments — regardless of corporate structure — include:

- The protocol specification is open and licensed under Apache 2.0
- The ATAH-operated reference MCP endpoint remains free to query (free meaning no per-query or licensing fee — querying still requires authentication); the protocol itself is royalty-free to implement
- Matching logic carries no commercial weighting at any point
- Provenance is always visible — every data point carries its source and verification status
- ATAH must not operate as a central repository of consumer personal data
- No professional category is surfaced in matching without an approved compliance annex
- The founder's role is advisory, carries no veto rights, is subject to conflict-of-interest disclosure and recusal rules, and confers no preferential commercial advantage — any entity the founder is involved in participates on terms equally available to any other participant

The full commitments are set out in [CHARTER.md](CHARTER.md).

## Acknowledgements

ATAH v0.8 was conceived and led by Grahame Cohen. The protocol architecture, the governance commitments, and the strategic framing all reflect his decisions throughout the design and review process.

Drafting and stress-testing of the specification and supporting documents was done in collaboration with Claude (Anthropic). Cross-AI editorial and strategic review was provided by ChatGPT (OpenAI). The mechanical work of producing the JSON Schemas, OpenAPI contract, MCP tools definitions, and supporting markdown files was carried out by Claude Code, working from a detailed handover document that captured the design decisions made during the spec process.

Architectural Decision Records ([docs/decisions/](docs/decisions/)) document the major design choices and the alternatives considered. Peer review contributions will be acknowledged in the CHANGELOG as they arrive.

## Licence

The ATAH specification is published under the [Apache License 2.0](LICENSE).

You may freely implement, build on, fork, or extend the protocol. No permission required, and no royalty. Standard Apache 2.0 attribution requirements apply when redistributing the specification. The ATAH-operated reference MCP endpoint is free to query subject to authentication, rate limits, and abuse controls.

## Get involved

This is a v0.8 release candidate. The work that comes next is hardening the spec through community review, talking to professional bodies and AI platforms, and building the first reference integrations.

If you build AI products, work in professional standards, run a professional body, or have eyes on related work in the standards space, your input is welcome. The most useful next steps:

- **Read** the [EXPLAINER](EXPLAINER.md) for the why, then the [spec](spec/v0.8/) for the how
- **Open an issue** with questions, challenges, or suggested changes
- **Open a discussion** for broader conversations about scope, governance, or strategy
- **Security disclosures** — see [SECURITY.md](SECURITY.md)
- **Get in touch directly** if you're representing a professional body, AI platform, or potential trusted partner: [grahame@atahprotocol.org](mailto:grahame@atahprotocol.org)
- **General questions and suggestions:** [hello@atahprotocol.org](mailto:hello@atahprotocol.org)

---

*ATAH — Agent to Authenticated Human Protocol · v0.8 · April 2026 · [atahprotocol.org](https://atahprotocol.org)*
