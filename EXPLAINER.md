# ATAH Explainer

**Agent to Authenticated Human Protocol**

A plain-language introduction to what ATAH is, why it exists, and how it fits into the agent web.

*Companion to the technical [specification](spec/v0.8/). Read this first. Read the spec second.*

---

## The handoff problem

The agent web is moving fast. AI systems can now book flights, buy products, complete transactions, and run multi-step workflows on behalf of users. The protocols that make this possible — MCP, A2A, ACP, AP2 — have arrived rapidly, and adoption continues to grow.

Every one of those protocols solves part of the journey: tool access, agent-to-agent communication, payment flows, commerce. They handle the parts of the journey one machine can pass to another machine. Several of them — MCP and A2A in particular — also support patterns that include human-in-the-loop steps.

The part they do not standardise is the moment a verified human professional is needed: who that professional is, what has been verified about them, what consent has been captured for the introduction, what data may be shared, and how the handoff state is tracked across systems.

When an AI identifies that human expertise is genuinely required — a lawyer for a contract dispute, a tax planner for an unfamiliar liability, a PR consultant for a crisis, a structural engineer for a building concern — the journey breaks. The AI typically stops at a disclaimer ("you should speak to a licensed professional") and leaves the user with a search box.

That's not a completed journey. It's a dead end at the moment the user most needs help.

ATAH is the protocol layer that fills that gap.

## What ATAH is

ATAH is an open protocol layer that gives AI systems a structured, trusted way to connect users to verified human professionals at the point AI reaches the limit of what it can helpfully provide.

It is not a directory product. It is not a recommendation engine. It is a **trust and handoff contract** — a common machine-readable standard defining the data model, consent receipts, provenance rules, handoff lifecycle, and conformance requirements for AI-to-authenticated-human professional handoff. AI systems integrate with an ATAH-compatible endpoint and receive validated professional options together with transparent credential, provenance, and trust signals, so they can support a clean handoff without leaving users stranded.

If the analogy helps: MCP standardises how AI systems reach tools and data. ATAH standardises the trust and handoff payload for reaching verified human professionals. ATAH can itself be exposed through MCP — and is, in v0.8 — so it is not a competitor to MCP. It composes with MCP, A2A, OAuth 2.1, and W3C Verifiable Credentials, sitting in the layer above them.

## ATAH composes with what already exists

ATAH does not try to reinvent identity, authentication, or credential standards. It uses what already works and adds the layer above:

- **OAuth 2.1** — the standard authentication mechanism for ATAH's MCP and REST API. AI platforms authenticate the same way they authenticate to any modern API.
- **OpenID Connect (OIDC)** — supported for professionals to log into their portal using their existing identity provider where they have one.
- **W3C Verifiable Credentials** — accepted as a partner data submission format. A bar association that issues digital credentials to its members can push them straight to ATAH; the cryptographic signature is verified and the claim is recorded with full provenance.
- **Decentralized Identifiers (DIDs)** — not required by v0.8 but the protocol is designed to work with them as the ecosystem matures.

ATAH's specific contribution is the layer those standards don't cover: professional categorisation, the staged introduction lifecycle, the trusted-partner trust model, the matching engine, and the commercial-neutrality and provenance-visibility commitments. None of that exists in the underlying identity and credential standards.

## Three layers — specification, implementations, and the reference registry

There's a clean distinction worth making early:

1. **The specification.** The protocol itself — defined in this repository, published under Apache 2.0. Anyone can read, implement, fork, or extend it.
2. **Conforming implementations.** Software systems that implement the specification. Conformance means preserving ATAH's mandatory principles: provenance always visible, no commercial weighting, staged consent, category compliance gating, data minimisation.
3. **The initial reference registry.** ATAH operates an initial reference registry and MCP/REST endpoints that demonstrate the protocol and serve as the first place AI platforms can connect. The reference registry is *not* the protocol itself. Other implementations may exist if they meet the conformance principles.

The registry is still necessary. A protocol needs records to operate against: professionals, categories, verification sources, consent receipts, handoff states. The important distinction is that the registry is an *implementation* of ATAH, not ATAH itself. The first registry gives AI platforms something real to use; the protocol keeps the model open.

v0.8 is single-registry by design; the architecture is built so federation can be added in v0.9 or v1.0 without breaking changes.

## Goals

ATAH is designed to:

- Provide a common machine-readable contract, with an initial reference endpoint, that returns relevant validated professional options for any authorised AI system that asks
- Cover both formally licensed professionals (lawyers, doctors, engineers) and established practitioners whose standing is real but not formally licensed (PR specialists, consultants, coaches, project managers)
- Make the basis of trust visible to AI systems and end users — every data point tagged with how it was verified and by whom, through a parallel provenance map
- Manage the full introduction lifecycle from initial match through to professional engagement, with explicit consent at every stage captured as a structured consent receipt
- Support not just consumer-to-professional handoff but also professional-to-professional referral relationships and professional-initiated client referrals
- Operate on terms that prevent any single commercial actor from controlling discoverability or access
- Sit cleanly alongside MCP, A2A, ACP, and other agent-web protocols — composing with them rather than competing

## Non-goals

ATAH is explicitly *not* designed to:

- **Recommend specific professionals as the "best" choice for a specific outcome.** ATAH presents validated options and the evidence behind them. The AI system applies its own context. The user makes the choice. Every match response carries a `presentation_disclosure` block that AI platforms are expected to surface to users — making the non-recommendation framing machine-readable.
- **Replace professional regulators or membership bodies.** ATAH surfaces what regulators and bodies already know. It does not adjudicate professional conduct or set professional standards. Concerns reported through ATAH are routed to the relevant regulatory body, never investigated by ATAH itself.
- **Operate as a paid lead-generation marketplace.** Matching is never weighted by commercial relationships. Payment funds data integration and protocol participation, not ranking.
- **Function as a consumer-facing product.** Consumers reach ATAH only through their AI system. There is no ATAH website where users browse for professionals.
- **Solve every category of professional service from day one.** Some categories require specific compliance work — for example, healthcare requires HIPAA-compliant handling — before they can be surfaced in matching. Those categories can register, but are excluded from match results until the relevant compliance work is signed off.
- **Warrant outcomes.** ATAH commits to specific data freshness windows, dispute resolution timelines, and concern-routing protocols. It does not warrant the suitability, competence, or behaviour of any individual professional.

## A note on terminology — "consumer"

Throughout this explainer and the spec, "consumer" is used as a protocol-neutral term for the person seeking professional services. In real-world contexts the equivalent term is usually "client" (legal, accounting), "patient" (healthcare), or "policyholder" (insurance). ATAH never communicates with the consumer directly — interaction is always mediated by an AI platform — so "consumer" is the term the protocol uses internally.

## Three types of introduction

ATAH supports three distinct introduction types, all sharing the same underlying protocol infrastructure but differing in who initiates them and how data flows.

**Consumer introduction (Type 1).** The most common case. A consumer's AI system identifies that human expertise is needed and queries ATAH on the consumer's behalf. ATAH returns a shortlist of relevant validated professionals; the AI system presents the options to the consumer; the introduction proceeds through staged consent.

**Professional referral relationship (Type 2).** A professional's AI assistant identifies a potential complementary professional in a different field — a tax planner connecting with a corporate lawyer, for example, or a PR specialist with a crisis communications barrister. ATAH proposes the referral relationship to both. Nothing flows unless both professionals independently agree. Type 2 establishes a relationship between professionals; it does not transmit client personal data. Established referral relationships become a quality signal in consumer matching, because professionals tend to refer to people they trust.

**Professional-initiated client referral (Type 3).** A professional uses their AI assistant to introduce their own client to another professional in a different field. The referring professional has already obtained the client's consent — either through the client's own AI system (a structured consent receipt) or through professional practice such as a signed engagement letter (a structured professional attestation of client consent). ATAH delivers the introduction, records the referral event, and crypto-erases the client's contact details after the receiving professional has retrieved them.

These three types together let ATAH support not only the consumer handoff problem but also the professional referral network that has historically taken professionals years to build manually.

## How a consumer introduction works

A consumer is talking to an AI system about something — a contract dispute, a tax liability they don't understand, a corporate communications issue, a structural concern at home. The AI does what it can. At some point, it identifies that the user needs an actual human professional.

Here is the journey ATAH supports:

**1. Find.** The AI system queries the ATAH endpoint with structured information about the matter — category, type, location, urgency. No personal data at this point. ATAH returns a small ranked shortlist of relevant professionals, each with verification status, credential information, response time commitments, and trust signals from contributing partners. Every match response also includes a `presentation_disclosure` block making the non-recommendation framing explicit.

**2. Decide.** The AI system presents the options to the user, applying its own contextual understanding of who that user is and what they need. The user makes the choice.

**3. Stage 1 — appetite check.** ATAH sends a structured introduction to the chosen professional containing only the matter type and field. No personal data yet. The professional indicates whether they have appetite and capacity to take the matter on.

**4. Stage 2 — pre-handoff check.** If the professional accepts, ATAH passes a small, category-appropriate set of information for the professional to clear before the full handoff. For lawyers this is a conflict check (consumer name, opposing party where relevant). For tax planners it might be a scope confirmation. For other categories it might be capacity confirmation or fee-range alignment. The data goes into a transient encrypted vault — not into anyone's email or SMS — and the professional retrieves it through an authenticated link. After the check resolves, the data is crypto-erased.

**5. Stage 3 — handoff.** If the pre-handoff check is cleared, the consumer is asked for explicit consent (a fresh consent receipt) to share their full contact details with this specific professional. On consent, the contact information goes into the transient vault and the professional retrieves it through authenticated retrieval. After retrieval, the data is crypto-erased. From this point, the relationship is between the consumer and the professional. ATAH steps out.

The whole journey is designed so that consumer personal data passes through ATAH only when explicitly needed and is destroyed as soon as it has done its job. **No personal data is ever sent through SMS or email.** Only notifications and authenticated retrieval references. ATAH is a routing and trust layer, not a database of consumer information.

## Consent — receipts, not tickboxes

Earlier versions of ATAH treated consumer consent as a boolean flag — the AI platform asserts "consent: true" and ATAH proceeds. That's not enough for high-trust professional handoff.

In v0.8, every consent action is captured as a **structured consent receipt**. The receipt records: the scope of consent (what stage, what data), the data categories covered, the professional involved, the asserting platform, the consent text the user was shown, the timestamp, the expiry, and whether revocation is supported.

ATAH itself stores only a hash of the receipt and its metadata — no personal data. The asserting AI platform stores the full receipt. If a dispute ever arises, the platform produces the receipt and ATAH verifies the hash matches what was stored at consent time. This proves the receipt hasn't been altered after the fact, without ATAH having to hold consumer personal data.

This pattern is borrowed from how transparency logs work for software supply chains and certificate authorities. It is designed to support evidence of consent and align with consent-receipt models such as GDPR Article 7 and ISO/IEC 29184, without making ATAH a personal data store.

Cancelling and revoking consent are first-class operations. The MCP tools `cancel_introduction` and `revoke_consent` are equally available alongside the consent-submission tools. Privacy-first means revoke must be as easy as submit.

## How the handoff is secured — and why it matters

A subtle but important point: in earlier versions, holding a `handoff_id` was enough to read or change the introduction. That sounds harmless, but `handoff_id`s end up in logs, in screenshots, in support tickets, in URLs. Possession alone shouldn't grant power.

In v0.8, access is tiered:

- **Reading status** is available to the original AI platform that started the introduction, *and* to any authenticated AI platform that supplies a consumer reference matching the handoff. This means a user moving between AI platforms can still check on their introduction — cross-platform continuity is preserved.
- **Changing state** (advancing stages, submitting data, cancelling, revoking, reporting outcomes) requires a separate `handoff_access_token`, bound to the platform that started the introduction, scoped to the handoff, time-limited, and revocable.

The result: random possession of a `handoff_id` cannot expose data or move an introduction forward. The cross-platform user experience is preserved. The bearer-token vulnerability is closed.

## Two categories of professional

ATAH serves two distinct categories of professional, through identical infrastructure.

**Credentialled professionals** hold a formal licence, certification, or accreditation from a recognised regulatory or professional body. Their standing can be verified directly against an authoritative source. Lawyers are verifiable against bar association databases (state bar associations are the representative US example) where partner agreements exist. Property and casualty insurance agents are verifiable against authoritative licensing data (NIPR is a representative example for US licensing) where the relevant partner agreement is in place. Tax planners and accountants are verifiable against professional accounting bodies and national registries. Doctors are verifiable against medical board records (state medical boards are the representative US example) where partner agreements exist. If their licence lapses or a disciplinary action is recorded, that fact is reflected in their profile.

**Established professionals** are recognised practitioners in fields where formal licensing does not exist but professional standing is real, verifiable, and commercially meaningful. PR specialists with chartered or fellow status from communications and PR membership bodies (CIPR and PRSA are representative examples). Coaches credentialled by professional coaching bodies (the International Coaching Federation is a representative example). Project managers certified by project management bodies (PMI is a representative example). HR professionals through HR membership bodies (SHRM is a representative example). Their standing is verified through trusted partner data — the relevant body confirms membership, level, and good standing.

The trusted partner model handles both categories the same way. The evidence differs. The protocol is identical.

This dual scope is deliberate. The professional services world is wider than the regulated subset, and AI systems will increasingly need to hand off to both groups. Restricting ATAH to formally licensed professionals would solve only half the problem.

## How professionals connect to the protocol

Professionals do not register "with a protocol." They are represented in an ATAH-conformant registry. At launch, that means the ATAH reference registry. Later, other conforming registries may exist if federation and conformance rules are specified.

There are two routes for a professional to be present in the registry.

**Via a trusted partner.** When a professional body, regulator, or other authoritative organisation joins ATAH as a trusted partner, the professionals it covers are surfaced through that partner's data. They don't need to register individually — they're already represented through their body. The professional may later claim the record to add self-declared fields and manage handoff preferences. Over time this becomes the dominant route as more partners join.

**By individual self-registration.** A professional can register themselves directly. This route exists permanently for two cases: where there is no professional body for the field, and where there is a body but membership is optional and the professional has chosen not to belong. Examples include many PR consultants, independent coaches, and some categories of consulting where there is no obligatory membership requirement.

Each professional record carries field-level provenance. A self-declared field is always labelled as self-declared. Partner-managed fields identify the partner source. Verified credentials identify the issuer. Enhanced verification identifies the independent verifier. Professionals do not need to understand protocol mechanics — the protocol ensures every claim about them is labelled by source and verification status.

Individual self-registration is free during the initial period after launch. After that initial period closes, individual self-registration remains available — it is not removed — but a small annual fee applies to maintain the listing. Hardship waivers are available; fee caps published. **Crucially, paid individual listings and partner-route listings receive identical matching treatment.** Paying the fee does not buy ranking.

When a trusted partner joins ATAH for a category, professionals who are members of that partner have their individual listings absorbed into the partner's data through a roll-up process. For high-risk regulated categories, professionals are notified before merge with a 7-day window to object. This is set out in the registration terms accepted at sign-up — necessary to prevent duplicate records and conflicting data for the same person.

## How verification works

Every data point in a professional's profile carries its own verification status independently. There is no single trust score that collapses everything into one number. Instead, AI systems receive granular information about how each fact was confirmed, and can present that to users transparently.

This is delivered through a **parallel provenance map** — alongside the profile data, a `_provenance` block records, for every verifiable field, how it was verified, by whom, and when. The verification statuses include:

- **Registry-verified** — confirmed active against an authoritative source by ATAH directly. Bar association lookups for lawyers in covered jurisdictions (such as state bar association integrations), and licensing-data lookups for insurance agents (such as integrations with US licensing data sources like NIPR), are representative examples of the type of verification this status describes.
- **Partner-verified** — confirmed by a named trusted partner with active partner status. The partner is always identified.
- **VC-verified** — confirmed by cryptographically verifying a W3C Verifiable Credential issued by an approved partner. Treated equivalently to partner-verified for matching, with the issuer recorded.
- **Self-declared** — the professional has stated this and it has not yet been independently verified. Always surfaced as such.

ATAH is built to be honest about what it knows and what it doesn't. False confidence in trust signals is worse than transparency about uncertainty.

## What partners actually verify

"Partner-verified" is not a single uniform claim. Different partners verify different things, and the protocol is designed to surface that difference rather than collapse it.

A regulator like a state bar association or a medical board verifies regulatory standing directly. They confirm an active licence and surface any disciplinary record. If they strike a professional off, that fact propagates immediately.

A professional membership body with strong, strictly enforced membership standards — chartered or fellow status, continuing professional development requirements, active disciplinary processes — verifies more than just "paid the dues." Membership in a body of this kind is genuine evidence of standing.

A membership body with looser standards — where membership is essentially open to anyone who pays the joining fee — verifies less. The verification confirms membership, but membership alone is a weaker signal.

The protocol surfaces these differences in the partner record, so AI systems and end users can interpret partner-verified status accurately. A "partner-verified" status from a state bar association carries a different weight from a "partner-verified" status from an open membership body. Both are honest claims; they describe different things.

## Enhanced verification — going beyond the standard layer

Standard partner verification is enough for most cases. Sometimes professionals, employers, or professional bodies want a deeper, ongoing, independently-audited verification — covering identity, qualifications, professional history, insurance, references, client feedback, compliance obligations, and sanctions screening.

ATAH supports this through an **enhanced verification layer**, administered by approved independent verifiers. Verifiers are separate from professional bodies, separate from review platforms, and approved by the protocol governance body before they may operate. They can be commissioned by a professional body offering it as a member benefit, by an employer verifying its staff, or by an individual professional wanting a stronger trust signal in the registry.

Enhanced verification produces a structured record of what was checked and to what depth. ATAH itself computes the trust contribution from that structured data — verifiers cannot set their own scores. The result is a strong, transparent additional signal that AI systems can interpret alongside other verification.

The commercial model is designed to separate payment from ranking. The verifier sets the price for the verification work, paid by whoever commissions it. ATAH does not take a share of those fees. Verifiers themselves pay ATAH a non-refundable assessment fee at application (covering the suitability review whether or not approval is granted) and an annual audit fee once approved (covering yearly methodology adherence audit). Both fees are tiered by the verifier's declared scope — narrow scope, multi-scope, or global scope. Audit fees scale with audit work. The fees are cost-recovery for governance, transparently published, and never influence matching. Fee waivers are available for public-interest organisations.

**Enhanced verification is not a promoted-placement mechanism.** It is a paid route to a stronger trust signal scored under the same public rubric available equally to all approved verifiers. The party who paid for the verification carries no weight in matching.

Enhanced verification is particularly valuable for professionals who don't have a strong body behind them — independent consultants, sole practitioners in fields without a clear professional association — and for bodies that want to offer a premium tier of verification on top of basic membership.

## Review platforms — treated as a distinct signal class

Review platforms — such as general consumer review platforms, legal review platforms (Avvo and Martindale-Hubbell are representative examples), healthcare review platforms (Healthgrades is a representative example), and broad-spectrum review platforms (Google reviews and Trustpilot are representative examples) — contribute aggregated review and feedback data. This data is meaningful when sourced from a platform with strong anti-gaming controls. It is unreliable when not. The protocol does not pretend otherwise.

Review platforms are treated as a specialised partner class with specific requirements as a condition of integration:

- **Reviewer identity verification** — anonymous review counts don't contribute to ATAH signals
- **Engagement verification** — whether the platform requires proof of an actual transaction or service before accepting a review
- **Outlier and pattern detection** — statistical detection of suspicious patterns
- **Removal transparency** — the platform publishes how many reviews it removes and broadly why
- **Self-review prevention** — controls against reviewing through dummy accounts

Annual re-attestation. Audit rights. Suspension criteria. Public classification rationale.

Review platforms are sub-classified by verification rigour into three tiers — `verified_transaction` (proof of actual transaction required), `verified_account` (verified account but no transaction proof), and `open_review` (no verification at all, lowest signal weight). The classification is set by ATAH governance and is not adjustable by the platform unilaterally.

Review-derived signals contribute to matching only above a minimum volume threshold (default 10 reviews per professional) and are time-decayed. **Review weight is capped per category.** For high-stakes regulated categories the cap is ≤0.10 within the verification quality score. They sit in the matching engine as a separate sub-component, not blended with regulatory or membership data, and tagged distinctly so AI systems can communicate them with appropriate context. Review data complements but does not substitute for credential verification.

The point is not to penalise review platforms — well-run review platforms with strong anti-gaming controls produce genuinely useful data. The point is to be honest about which platforms produce reliable signals and which don't, and to surface that distinction transparently rather than treating all review data as equivalent.

## Where ATAH sits in the agent-web stack

The agent web has been built up layer by layer over the last two years, on top of established identity and credential standards:

| Layer | Purpose |
|---|---|
| **OAuth 2.1 / OIDC** | Authentication |
| **W3C Verifiable Credentials** | Cryptographically verifiable credentials |
| **MCP** (Model Context Protocol) | AI access to tools and data |
| **A2A** (Agent-to-Agent Protocol) | Agent-to-agent communication |
| **ACP / UCP / AP2** | Agentic commerce and payments |
| **ATAH** | **AI-to-authenticated-human professional handoff** |

Each layer solves a specific problem. None of them, individually or collectively, defines the trust, provenance, consent, and lifecycle payload required for a verified human-professional handoff. That layer was missing.

ATAH is a proposal for what that layer should look like. It does not replace any of the protocols above — it composes with them, adding a capability the agent web does not currently have. The v0.8 bindings expose ATAH operations through MCP and REST; future bindings (A2A-compatible, federation, push) are anticipated as the ecosystem evolves.

## Who this serves

**AI platforms** get a common ATAH-compatible handoff contract — available through the v0.8 MCP and REST bindings — that returns validated professional options when their users need human expertise. No bespoke integration per professional category or per jurisdiction. No directory deals to negotiate. Free to query (no per-query or licensing fee — querying still requires authentication for protocol integrity). Conforms to current MCP authorisation guidance. The initial reference registry provides the first live endpoint; future conforming implementations may expose the same protocol.

The proposition to AI platforms is direct: supporting ATAH addresses the broken handoff problem that every AI system currently hits. It turns a limitation into a capability without each platform having to solve it independently.

**Professional associations** get a practical answer to the question every member is asking: *how do I stay visible and trusted when AI becomes the first call for professional advice?* Associations participate as trusted partners — contributing what they already know about their members, and giving those members structured machine-readable presence in AI-mediated environments. The data already exists in their member registries. ATAH makes it useful at the moment AI systems need it.

**Credentialled and established professionals** get a way to be discoverable by AI systems without mastering Answer Engine Optimisation. Their standing is verified and kept current, then surfaced wherever AI systems reach the human-handoff moment. No platform lock-in. No marketing budget required. Concern flag protections — admin-only visibility, right of reply, pattern-based review — guard against defamation through bad-faith reports.

**Trusted partners** — regulatory databases, professional bodies, review platforms, independent verifiers — get a clean way to make their verified data useful at the exact point users are deciding whether to trust a professional. Provenance is always visible. The commercial model funds the work of maintaining the integration without compromising protocol neutrality.

**Consumers**, finally, get a continuation of journey rather than a dead end. When their AI system identifies that they need a professional, the AI can offer real options with real verification signals, rather than handing them back to a search engine.

## The trusted partner model

The trusted partner network is what makes the registry credible and what funds the protocol.

Trusted partners are organisations that hold reliable, maintained data about professionals and meet ATAH's partner standards. The types of organisation the protocol is designed to work with include: regulatory and licensing bodies (such as state bar associations, NIPR, NAIC, FINRA, and equivalents in other jurisdictions); professional bodies for credentialled fields (such as the American Bar Association, and bodies covering property and casualty insurance professionals like the Big "I" (IIABA), PIA, and the CPCU Society); professional bodies for established practitioner fields (such as PRSA, CIPR, the International Coaching Federation, SHRM, and PMI); and review platforms (such as Google reviews, Avvo, Healthgrades, and Martindale-Hubbell). Hundreds of similar organisations exist across hundreds of professional categories worldwide. None of the named bodies above are committed partners at v0.8 publication; each is a representative example of the type of organisation the protocol's trusted-partner framework is designed to work with.

The model is consistent across all of them: partners pay ATAH for integration access, with public fee schedules and waivers for regulators and public-interest bodies. Members gain structured machine-readable presence in AI-mediated environments — the answer to the AEO question. The commercial model exists to fund the work of building and maintaining the protocol. It is not the reason the protocol exists.

Three commitments are non-negotiable in this model:

- Partner payments do not influence matching ordering or recommendation status. Ever.
- Partner-provided data is always tagged with its source and verification basis. Provenance is never obscured.
- Where partners' data conflicts on something material — for example, two different sources reporting different licence statuses — the affected profile is suppressed from matching until the conflict is resolved.

Trust and commercial logic are structurally separate. Payment funds participation. It does not buy ranking.

**On named organisations.** Throughout this explainer, named bodies (state bar associations, NIPR, NAIC, FINRA, the American Bar Association, the Big "I", PIA, the CPCU Society, PRSA, CIPR, the International Coaching Federation, SHRM, PMI, Google reviews, Avvo, Healthgrades, Martindale-Hubbell, and others) appear as illustrative examples of the *types* of organisation the protocol is designed for. They are not committed partners. They appear because they are the most recognisable bodies in their respective categories and jurisdictions, helping readers see who the trusted-partner framework is for. Becoming a partner requires a partner agreement, data integration work, and governance approval — none of which is in place at v0.8 publication.

## Privacy and data — structural, not policy

ATAH is built around minimising central retention of consumer personal data. This isn't a policy that could be loosened later — it's a Charter Part One core commitment that requires supermajority governance to change.

The system distinguishes data categories and treats each separately:

- **Professional identity data** — names, credentials, contact preferences. Core registry data, retained for as long as the professional is registered. They control it.
- **Trust and verification metadata** — partner-contributed data points, verification records, validator provenance. Retained and tagged at all times.
- **Consent receipt metadata** — hash and metadata only. No personal data.
- **Introduction lifecycle data** — pseudonymous state transitions logged. No personal data.
- **Consumer personal data** — strictly transient. Held only in the encrypted vault during active handoffs. Crypto-erased on resolution. **Never written to the main registry database. Never transmitted through SMS or email.**
- **Audit log** — pseudonymous identifiers only. Tamper-evident. Retained 7 years for regulatory expectations.

Every introduction requires fresh explicit consent, captured as a structured consent receipt. ATAH never re-uses personal data from a previous introduction.

The transient encrypted vault and the authenticated retrieval portal mean personal data never sits in a notification provider's logs, an inbox, a phone notification history, or a device backup. The professional gets a notification with no personal data; they tap the link; they authenticate at the appropriate tier (light for low-sensitivity scope confirmations; step-up authentication for healthcare or sensitive legal matters); they see the data; the data is then crypto-erased.

## Alternatives considered

The "open protocol" approach is one of three plausible designs. The other two were considered and rejected.

**A platform-owned directory.** A single AI platform — Anthropic, OpenAI, Google — could build a proprietary professional directory tied to its own AI product. Some elements of this exist already in narrow forms. The reason this does not solve the actual problem is that it locks every professional into a specific platform's terms, and locks every consumer into whichever AI system their professional happens to be listed on. The AEO problem moves from "many platforms each with their own logic" to "a few platforms each acting as gatekeepers." That is not an improvement.

**Per-profession federations.** Each professional category could build its own discovery infrastructure — one for legal, one for medical, one for PR, and so on. Some categories have started doing this in fragmentary ways. The reason this does not solve the problem is that AI systems cannot realistically integrate with hundreds of bespoke per-category directories. The N×M integration problem applies here too: every AI platform would need bespoke integrations with every category-specific directory in every jurisdiction. The cost is prohibitive and the result is fragmented coverage with no consistent verification model.

**A single open protocol, run as neutral infrastructure.** This is what ATAH proposes. One endpoint. One verification model. One introduction lifecycle. Open governance. Free for any AI platform to query. The protocol is profession-agnostic at the architecture level, with category-specific compliance handled in annexes that gate when each category becomes available for matching. This approach addresses the integration cost problem for AI platforms, the visibility problem for professionals, and the AEO problem for professional bodies — through a single shared standard rather than three separate ones.

## Status and rollout

ATAH is at v0.8, released as a release-candidate specification. It is intended to be implementable and is ready for technical review and reference implementation. It defines the core data model, introduction lifecycle, provenance model, matching principles, consent stages, and governance commitments. Machine-readable schemas, OpenAPI contracts, and the MCP tool definitions are included for review and will be hardened through reference implementation work.

The protocol moves to v1.0 once at least one trusted partner integration is live, the reference implementation has run end-to-end, and at least one external implementation has been built against the spec.

Rollout is partner-driven. The protocol itself is profession-agnostic and jurisdiction-agnostic at the architecture level. Practical coverage in any given category or jurisdiction grows as trusted partners join — regulators, professional bodies, review platforms — and verification pathways are established.

Some categories will move quickly because partners are ready to integrate or because automated regulatory verification is straightforward. Others will move more gradually because verification pathways need more development, or because category-specific compliance work (such as HIPAA for healthcare) needs to be completed before profiles can be surfaced in matching. This is not a limitation of the protocol; it is how the network grows.

The shape of the rollout — partner-driven, category-by-category, with compliance gating where regulation requires it — is intentional. Better to do each category properly than to claim broad coverage that the verification model cannot actually support.

## What's deferred to v0.9 and beyond

Being honest about what's *not* in v0.8:

- **Federation mechanics.** v0.8 is single-registry by design. The architecture is federation-ready (atah_id namespacing, no single-registry assumptions in the data model), but cross-registry trust, ID resolution, and federated governance are deferred to v0.9 or v1.0.
- **Claim authentication hardening.** v0.8 specifies the minimum (verified email match against partner records, or equivalent partner-attested identifier). Step-up authentication, anomaly detection on claim attempts, notification to the partner-known channel with an objection window, and rate limiting are deferred to v0.9. The reference registry is expected to apply these operationally at launch.
- **Platform-light consent mode.** v0.8 puts consent capture on the asserting AI platform; ATAH stores only the receipt hash and metadata. v0.9 will additionally specify an optional mode where ATAH provides more of the consent ceremony (canonical consent text and flow orchestration), so platforms preferring a standard ATAH-driven flow can invoke it rather than constructing receipts themselves. The v0.8 model continues to be supported.
- **Sophisticated Sybil and referral-ring detection.** v0.8 has anti-gaming controls — reciprocal cap, time decay, referrer weighting, evidence requirements — but full Sybil/ring detection is a research-grade problem deferred.
- **Per-category matching weights populated with researched values for all launch categories.** v0.8 ships with sensible defaults plus exemplar overrides. Full per-category research is v0.9 work needing domain expertise.
- **Full STRIDE-level threat model.** SECURITY.md ships with threat categories and controls; full modelling is deferred.
- **Conformance test suite.** Capabilities discovery endpoints ship; the test suite itself is v0.9.
- **Multilingual support.** v0.8 is English-only in examples; localisation is later.

These are acknowledged as known limits, not pretended to be solved.

## Governance

ATAH is founded by Grahame Cohen. The protocol specification is published under Apache 2.0 and is permanently open. The MCP endpoint is free to query.

The Charter is intended to operate as a public governance covenant. The founder undertakes to administer ATAH consistently with its commitments. An interim advisory group will be convened when suitable members are identified and available to serve. Governance transitions to an independent structure when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated — recognising that the cross-industry organisations the protocol works with have inherently slower decision cycles than the software industry. Transition progress is reported publicly when there is meaningful progress to share.

The founder retains a permanent advisory seat with no veto rights, no preferential commercial advantage, and explicit conflict-of-interest, recusal, and disclosure rules. After transition the seat carries consultation rights but not approval rights for core amendments.

The full governance commitments are set out in [CHARTER.md](CHARTER.md).

## What comes next

This is a v0.8 release candidate. The work that follows falls into four streams:

- **Community review.** The specification needs eyes on it from people building AI products, working in professional standards, running professional bodies, and thinking about related problems in the standards space. Feedback is invited via GitHub issues and discussions.
- **Trusted partner conversations.** First conversations are open with regulatory databases, professional bodies, and review platforms. The natural first movers are the categories where the AEO question is sharpest right now — legal, property and casualty insurance, communications, coaching, project management.
- **Reference implementation.** A reference registry implementation is the Phase 2 build, commencing once resources allow. The implementation is separate from the protocol and does not change protocol governance. Anyone can build their own implementation against the same spec.
- **Independent governance structure.** Legal advice on the right structure (UK CIC, foundation model, US public benefit corporation, or equivalent) will be taken when transition is in prospect. The transition happens when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated, per the Charter.

ATAH is offered to the community to refine, adopt, fork, or improve.

## Get involved

The most useful next steps:

- **Read** the [specification](spec/v0.8/) for the technical detail
- **Open an issue** with questions, challenges, or suggested changes
- **Open a discussion** for broader conversations about scope, governance, or strategy
- **Security disclosures** — see [SECURITY.md](SECURITY.md)
- **Get in touch directly** if you're representing a professional body, AI platform, or potential trusted partner: [grahame@atahprotocol.org](mailto:grahame@atahprotocol.org)
- **General questions and suggestions:** [hello@atahprotocol.org](mailto:hello@atahprotocol.org)

---

*ATAH — Agent to Authenticated Human Protocol · v0.8 · April 2026 · [atahprotocol.org](https://atahprotocol.org)*
