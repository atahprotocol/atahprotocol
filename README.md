# ATAH

**Agent to Authenticated Human Protocol**

An open protocol layer for AI-to-authenticated-human professional handoff. ATAH defines the trust, provenance, consent, and lifecycle contract that conforming registries and AI platforms can implement — exposed through MCP and REST bindings, with the ATAH reference registry as the first operational implementation.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Spec Version](https://img.shields.io/badge/spec-v0.8.2-orange.svg)](spec/v0.8/)
[![Status: Release Candidate](https://img.shields.io/badge/status-release_candidate-yellow.svg)](#status)

---

**AI is becoming the first call for advice on legal, tax, insurance, and other professional matters. But when human expertise is genuinely needed, the journey usually breaks — leaving the user with a disclaimer and the responsibility to find one themselves. ATAH defines the trust, consent and handoff layer between AI systems and verified human professionals.**

## The problem

AI systems can now book flights, buy products, complete transactions, and run multi-step workflows on behalf of users. But when an AI identifies that human professional expertise is genuinely needed — a lawyer for a contract, an insurance agent for a complex risk, a financial advisor for a regulated decision, a PR consultant for a crisis, a structural engineer for a building concern — the journey breaks. The AI typically stops at a disclaimer and leaves the user with a search box.

That broken handoff is the gap ATAH is designed to fill.

ATAH is not primarily about AI giving up. It is about the point at which AI reaches a boundary where verified human authority, accountability, consent, and provenance matter.

## What ATAH is

ATAH is an open protocol layer that gives AI systems a structured, trusted way to connect users to credentialled and established human professionals when AI reaches the limit of what it can helpfully provide.

It is a **trust and handoff contract** — a common machine-readable standard defining the data model, consent receipts, provenance rules, handoff lifecycle, and conformance requirements for AI-to-authenticated-human professional handoff. The protocol is transport-compatible: it can be exposed through MCP tools, REST APIs, A2A-compatible agents, or future bindings. The v0.8 package defines MCP and REST bindings and an initial reference registry.

ATAH separates the standard from the service that first implements it. AI systems integrate with an ATAH-compatible endpoint and receive verified professional options together with transparent credential, provenance, and trust signals — supporting a clean handoff without leaving users stranded.

The protocol works for two categories of professional, through identical infrastructure:

- **Credentialled professionals** — those whose standing can be verified against formal licences, certifications, or regulatory records. Lawyers, property and casualty insurance agents, financial advisors, doctors, engineers, tax planners, accountants, architects.
- **Established professionals** — those whose standing is verifiable through professional body membership, peer recognition, and trusted partner data, even where formal licensing does not apply. PR specialists, management consultants, executive coaches, HR professionals, project managers, and many others.

Every data point in a profile carries its own verification status — registry-verified, partner-verified, VC-verified, or self-declared — returned transparently in every response via a parallel provenance map. AI systems receive granular trust signals and can communicate them clearly to users. ATAH is designed to make the basis of trust visible, not to collapse it into an opaque score.

Two routes exist for professionals to be represented in an ATAH-conformant registry: through their professional body when that body has joined as a trusted partner, or by individual self-registration where no partnered body covers them. Both routes coexist permanently, and matching treatment is identical between them. At launch, those records live in the ATAH reference registry; future conforming registries may exist once federation mechanics are specified.

The protocol is structured as two components. **Component 1 (Discovery)** is the foundation — a query for candidate professionals with a required `request_intent` parameter (`self`, `on_behalf_of_client`) that gates whether Component 2 is available next, and a required `limit` (1–100) for candidate-set size. **Component 2 (Consumer-self handoff)** is the staged consent / vault / authenticated-retrieval flow, available only when Discovery was called with `request_intent: 'self'`; the AI agent chooses one of three flow variants (off-protocol, contact-share, full Stage 1 / Stage 2 / Stage 3) based on user preference and category requirements.

The professional-on-behalf-of-client referral case is served by Component 1 (Discovery) alone, with `request_intent: 'on_behalf_of_client'`. The professional gets the candidate set; they handle the introduction off-platform under their own consent capture with the client. ATAH does not deliver client contact details based on professional attestation, ever.

## What ATAH is *not*

- **Not a recommendation engine — provides defensible verifiable grounds.** ATAH does not express preference among eligible candidates. It returns a provenance-visible candidate set based on declared need, verification evidence, category rules, and availability. v0.8.2 makes this structural rather than declarative: hard filters → bands → randomisation within bands. There is no global match score in the schema, no hidden ranking, and the conformance test verifies that within-band ordering is observably non-deterministic. When the user has no explicit ordering preference, candidates are ordered by stratified randomisation; when they do (nearest, soonest available, specific language, highest verification confidence, etc.), ATAH applies the named mode and records it on the response. AI platforms rank everything they surface, by their nature, and ATAH does not prevent that — AI platforms reordering downstream MUST preserve the disclosure that ATAH applied non-preferential ordering. ATAH's contribution is providing the verifiable basis for the candidate pool; the ranking AI platforms apply downstream is the AI platform's domain. The user or calling AI platform remains responsible for selection.
- **Transparent — every meaningful decision is explainable.** Provenance and transparency are ATAH's two pillars. Every relevant response carries a machine-readable `decision_explanation` at both response level and per candidate, plus an aggregate `exclusion_summary` for excluded candidates. Professionals can see why they appeared or did not appear in candidate searches via a rules-derived view (no query traffic exposed). Governance and auditors can retrieve named-excluded-candidate detail through a dedicated endpoint. Transparency is a top-level conformance requirement in v0.8.2 — the new fifth class alongside Core Object, Binding, Registry, and Governance.
- **Verified contact details, kept fresh.** ATAH says "verified candidates" and means it including reachability. Channel verification runs on a per-category cadence (quarterly for high-stakes regulated categories; annual for established practitioners), escalates when challenges go unanswered, and runs a just-in-time check before high-stakes introductions. Stale channels flip the professional's `matching_status` to `contact_unverified` and they stop appearing in Discovery results until re-verified. Engagement liveness — whether a professional with verified contact details actually responds — is a separate problem honestly deferred to v0.8.3.
- **Non-recommendation framing is machine-readable.** Every match response carries a `presentation_disclosure` block that AI platforms are expected to surface to users. The non-recommendation stance is not a policy claim; it is a structural commitment built into every response payload.
- **Not a regulator, enforcement body, or complaints adjudicator.** Professional conduct concerns are routed to the relevant regulatory or professional body.
- **Not a payment processor or marketplace.** No transaction or commerce capability.
- **Not a directory product or AEO replacement.** Professionals are not "promoted" or "ranked for visibility" in any commercial sense.
- **Not a personal data repository.** Consumer personal data passes through ATAH only as transient handoff data, held in a transient encrypted vault, and crypto-erased per protocol.
- **Not a consumer interface.** Consumers interact with AI systems, which use ATAH on their behalf.
- **Does not warrant outcomes.** ATAH commits to specific data freshness windows and dispute timelines, but does not warrant the suitability, competence, or behaviour of any individual professional.
- **Allocation of responsibilities is documented.** AI platforms are the consumer-facing entity and carry consent capture, presentation, and consumer-relationship responsibilities. ATAH commits to provenance visibility, freshness windows, conflict suppression, dispute SLAs, concern flag escalation, and audit integrity. ATAH does not warrant outcomes or real-time accuracy beyond freshness windows. The full allocation is in PRD §11.

## Three layers

Clarity about which layer is being discussed matters:

1. **The open specification.** The protocol itself, defined in this repository and published under Apache 2.0. Anyone may implement, fork, or extend it.
2. **Conforming implementations.** Software systems implementing the specification. Conformance requires preserving ATAH's mandatory principles: provenance visibility, no commercial weighting, staged consent, category compliance gating, and data minimisation.
3. **The initial reference registry.** ATAH operates an initial reference registry and MCP/REST endpoints that demonstrate and operate the protocol. The reference registry is *not* the protocol itself. As federation develops (deferred to v0.9 or v1.0), other conforming registries may exist, provided they preserve the conformance principles.

The initial registry exists because launching a protocol with no operational instance produces no real-world adoption. v0.8 is single-registry by design; the architecture is federation-ready.

## Forks, derivatives, and official ATAH status

The ATAH specification is published under Apache 2.0 and may be implemented, forked, or extended in accordance with that licence. The open licence is intentional — it allows anyone to build on the work and protects the protocol from being captured by any single party.

A fork or derivative work may not represent itself as the official ATAH protocol, the ATAH-operated reference registry, or an ATAH-governed implementation unless it is operating under the applicable ATAH governance process. Use of the ATAH name, logo, conformance badge, official conformance recognition, or official project identity must not imply endorsement, certification, governance continuity, or founder participation where none exists. Forks and derivative protocols should clearly disclose their relationship to ATAH and identify any material changes from the published ATAH specification.

Conforming implementations — third-party registries or endpoints that operate the ATAH protocol as published, meeting the conformance criteria — are different from forks. Conforming implementations remain part of the ATAH ecosystem; their operators are bound by the conformance principles set out in the specification. Forks are independent works unless and until they operate the ATAH protocol as published and meet the applicable conformance criteria.

## Implementation model

ATAH separates the standard from the service that first implements it.

The **ATAH Core Protocol** defines the data structures, consent receipts, provenance rules, handoff states, presentation disclosures, and conformance requirements.

The **ATAH bindings** define how those objects are exchanged. v0.8 defines MCP and REST bindings. Future versions may define A2A-compatible or federation bindings.

The **ATAH reference registry** is the first operational implementation. It stores professional records, ingests trusted partner data, runs matching, manages handoff lifecycle state, and exposes MCP/REST endpoints. It demonstrates and operates the protocol — but it is not a requirement that all ATAH records live in one database forever.

Future conforming registries may exist once federation and cross-registry trust mechanics are specified. Until then, v0.8 is single-registry by design and federation-ready by architecture.

### A note on named organisations

Throughout this package, named professional bodies, regulators, and review platforms (state bar associations, NIPR, FINRA, PRSA, CIPR, the International Coaching Federation, and others) appear as **illustrative examples** of the *types* of organisations the protocol is designed to work with. They are not partners with whom agreements exist. They appear because they are the most recognisable bodies in their respective categories and jurisdictions, and they help readers understand who the protocol's trusted-partner framework is for.

Becoming a trusted partner requires a partner agreement, data integration work, and governance approval. None of those are in place at v0.8 publication. The protocol's value is demonstrated by its design — not by claiming relationships that do not yet exist.

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

**AI platforms.** A common ATAH-compatible handoff contract, available through the v0.8 MCP and REST bindings, that returns verified professional options when your users need human expertise. No bespoke integration per category or jurisdiction. A production integration involves OAuth 2.1 with audience validation, the handoff access token lifecycle, idempotency on mutating endpoints, and the cross-platform status check model — engineering effort comparable to integrating a payments processor rather than a search API. Free to query (no per-query or licensing fee — querying still requires authentication for protocol integrity). Conforms to current MCP authorisation guidance (OAuth 2.1-compatible). The reference registry will provide the first conforming endpoint when operational; future conforming implementations may expose the same protocol.

**Professional associations.** A practical answer to the question every member is asking: *how do I stay visible and trusted when AI becomes the first call for professional advice?* Associations participate as trusted partners — contributing what they already know about their members, and giving those members structured machine-readable presence in AI-mediated environments. Public partner fee schedules; waivers available for public-interest bodies.

**Credentialled and established professionals.** A way to be discoverable by AI systems as part of their AI-discoverability infrastructure. No platform lock-in. Standing verified and kept current. Concern flag protections (admin-only visibility, right of reply).

ATAH is not a replacement for a professional's existing marketing or visibility work. It is infrastructure for being represented to AI systems consistently and credibly. Professionals continue to need their own marketing for direct consumer reach; ATAH addresses the specific moment an AI system needs to identify and hand off to a verified human professional. There is no guarantee of introduction volume, ranking, or work; the protocol is designed to become a trusted handoff rail as platforms integrate.

**Trusted partners.** Authoritative regulatory sources (state bar associations, NIPR-style licensing data, FINRA-style registers, medical boards, and equivalents) and professional membership bodies are the credential-verification authority layer — they confirm what professionals are licensed or qualified to do. Review platforms participate as a distinct partner class — signal providers, not authority providers — contributing aggregated review data subject to anti-gaming attestations and per-category review weight caps (≤0.10 within verification quality for high-stakes regulated categories). Independent verifiers add a separate enhanced-verification layer. Provenance is always visible across all partner types, and review-derived signals complement but do not substitute for credential verification.

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
- **[COVERAGE.md](COVERAGE.md)** — launch coverage matrix by category and jurisdiction
- **[docs/consent-storage-rationale.md](docs/consent-storage-rationale.md)** — design rationale for the consent receipt split between AI platform and ATAH
- **[spec/v0.8/](spec/v0.8/)** — full technical specification, OpenAPI definition, MCP tool schemas, JSON Schema files
- **[press/](press/)** — launch announcements for media, professional associations, and AI platforms

## Status

**v0.8.2 — Release candidate.**

The v0.8.2 specification is intended to be implementable and is ready for technical review and reference implementation. It defines the core data model, introduction lifecycle, provenance model, matching principles, consent stages, transparency-as-conformance, and governance commitments. Machine-readable schemas, OpenAPI contracts, and the MCP tool definitions are included for review and will be hardened through reference implementation work.

The protocol moves to v1.0 once at least one trusted partner integration is live, the reference implementation has run end-to-end, and at least one external implementation has been built against the spec.

Breaking changes are possible until v1.0. The version negotiation and deprecation policy is defined in the spec.

**Note on maintenance.** ATAH is currently maintained by a small founding team. Published response times for security disclosures, governance objections, and review of contributions are best-effort at release-candidate stage. The project is actively seeking reviewers, partners, implementers, and advisors — anyone interested in supporting the protocol's development is encouraged to get in touch via the channels in this README.

## Governance

ATAH is founded by [Grahame Cohen](https://github.com/grahamecohen). The protocol operates under the founding [CHARTER](CHARTER.md), which is intended to operate as a public governance covenant binding the founder to administer ATAH consistently with its commitments.

It is the founding commitment that governance of the protocol will transition to an independent structure — a not-for-profit, foundation, or equivalent — when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated. The trigger framework reflects that the protocol works with cross-industry organisations whose decision cycles are inherently slower than software-industry timelines, and that independent governance is appropriate when there is an ecosystem to govern rather than at an arbitrary calendar point.

An interim advisory group will be convened when suitable members are identified and available to serve. The status of governance transition will be reviewed and reported publicly when there is meaningful progress to share.

The Charter's entrenched core commitments — regardless of corporate structure — include:

- The protocol specification is open and licensed under Apache 2.0
- The ATAH-operated reference MCP endpoint remains free to query (free meaning no per-query or licensing fee — querying still requires authentication); the protocol itself is royalty-free to implement
- Matching logic carries no commercial weighting at any point
- Provenance is always visible — every data point carries its source and verification status
- ATAH must not operate as a central repository of consumer personal data
- No professional category is surfaced in matching without an approved compliance annex
- The founder's role is advisory, carries no veto rights, is subject to conflict-of-interest disclosure and recusal rules, and confers no preferential commercial advantage — any entity the founder is involved in participates on terms equally available to any other participant

The full commitments are set out in [CHARTER.md](CHARTER.md).

## Acknowledgements

ATAH v0.8.1 was conceived, architected, and led by Grahame Cohen. The protocol architecture, governance commitments, commercial-neutrality model, privacy architecture, and strategic framing reflect his decisions throughout the design and review process. AI tools were used as drafting, stress-testing, editorial, and schema-generation assistants — Claude (Anthropic) for specification drafting and stress-testing, ChatGPT (OpenAI) for cross-AI editorial and strategic review, and Claude Code for mechanical production of the JSON Schemas, OpenAPI contract, MCP tool definitions, and supporting markdown. Final structural decisions, trade-offs, and publication responsibility remain human-led.

Architectural Decision Records ([docs/decisions/](docs/decisions/)) document the major design choices and the alternatives considered.

v0.8.2 incorporates substantial peer review from Paolo Piponi, whose findings shaped the principal/delegation model, the three-concept separation of payload erasure / audit retention / withdrawal-as-state-transition, three consent types with engagement consent excluded by design, stratified randomisation as the default ordering policy, transparency-as-conformance, and the stress-test matrix discipline.

Thanks also to Matthew Kogan for review and feedback on v0.8.1.

The protocol benefits from their work; remaining errors are mine.

## Licence

The ATAH specification is published under the [Apache License 2.0](LICENSE).

You may freely implement, build on, fork, or extend the protocol. No permission required, and no royalty. Standard Apache 2.0 attribution requirements apply when redistributing the specification. The ATAH-operated reference MCP endpoint is free to query subject to authentication, rate limits, and abuse controls.

## Get involved

This is a v0.8.2 release candidate. The work that comes next is hardening the spec through community review, talking to professional bodies and AI platforms, and building the first reference integrations.

If you build AI products, work in professional standards, run a professional body, or have eyes on related work in the standards space, your input is welcome. The most useful next steps:

- **Read** the [EXPLAINER](EXPLAINER.md) for the why, then the [spec](spec/v0.8/) for the how
- **Open an issue** with questions, challenges, or suggested changes
- **Open a discussion** for broader conversations about scope, governance, or strategy
- **Security disclosures** — see [SECURITY.md](SECURITY.md)
- **Get in touch directly** if you're representing a professional body, AI platform, or potential trusted partner: [grahame@atahprotocol.org](mailto:grahame@atahprotocol.org)
- **General questions and suggestions:** [hello@atahprotocol.org](mailto:hello@atahprotocol.org)

---

*ATAH — Agent to Authenticated Human Protocol · v0.8.2 · May 2026 · [atahprotocol.org](https://atahprotocol.org)*
