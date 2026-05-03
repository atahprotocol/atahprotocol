# ATAH Explainer

**Agent to Authenticated Human Protocol**

A plain-language introduction to what ATAH is, why it exists, and how it fits into the agent web.

*Companion to the technical [specification](spec/v0.8/). Read this first. Read the spec second.*

---

**AI is becoming the first call for advice on legal, tax, insurance, and other professional matters. But when human expertise is genuinely needed, the journey usually breaks — leaving the user with a disclaimer and the responsibility to find one themselves. ATAH defines the trust, consent and handoff layer between AI systems and verified human professionals.**

## The handoff problem

The agent web is moving fast. AI systems can now book flights, buy products, complete transactions, and run multi-step workflows on behalf of users. The protocols that make this possible — MCP, A2A, ACP, AP2 — have arrived rapidly, and adoption continues to grow.

Every one of those protocols solves part of the journey: tool access, agent-to-agent communication, payment flows, commerce. They handle the parts of the journey one machine can pass to another machine. Several of them — MCP and A2A in particular — also support patterns that include human-in-the-loop steps.

The part they do not standardise is the moment a verified human professional is needed: who that professional is, what has been verified about them, what consent has been captured for the introduction, what data may be shared, and how the handoff state is tracked across systems.

When an AI identifies that human expertise is genuinely required — a lawyer for a contract dispute, a tax planner for an unfamiliar liability, a PR consultant for a crisis, a structural engineer for a building concern — the journey breaks. The AI typically stops at a disclaimer ("you should speak to a licensed professional") and leaves the user with a search box.

That's not a completed journey. It's a dead end at the moment the user most needs help.

ATAH is the protocol layer that fills that gap.

## What ATAH is

ATAH is an open protocol layer that gives AI systems a structured, trusted way to connect users to verified human professionals at the point AI reaches the limit of what it can helpfully provide.

It is not a directory product. It is not a recommendation engine. It is a **trust and handoff contract** — a common machine-readable standard defining the data model, consent receipts, provenance rules, handoff lifecycle, and conformance requirements for AI-to-authenticated-human professional handoff. AI systems integrate with an ATAH-compatible endpoint and receive verified professional options together with transparent credential, provenance, and trust signals, so they can support a clean handoff without leaving users stranded.

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

- Provide a common machine-readable contract, with an initial reference endpoint, that returns verified candidate professional options for any authorised AI system that asks
- Cover both formally licensed professionals (lawyers, doctors, engineers) and established practitioners whose standing is real but not formally licensed (PR specialists, consultants, coaches, project managers)
- Make the basis of trust visible to AI systems and end users — every data point tagged with how it was verified and by whom, through a parallel provenance map
- Manage the full introduction lifecycle from initial match through to professional engagement, with explicit consent at every stage captured as a structured consent receipt
- Support not just consumer-to-professional handoff but also professional-to-professional referral relationships and professional-initiated client referrals
- Operate on terms that prevent any single commercial actor from controlling discoverability or access
- Sit cleanly alongside MCP, A2A, ACP, and other agent-web protocols — composing with them rather than competing

## Non-goals

ATAH is explicitly *not* designed to:

- **Recommend specific professionals as the "best" choice for a specific outcome.** ATAH presents verified options and the evidence behind them. The AI system applies its own context. The user makes the choice. Every match response carries a `presentation_disclosure` block that AI platforms are expected to surface to users — making the non-recommendation framing machine-readable.
- **Replace professional regulators or membership bodies.** ATAH surfaces what regulators and bodies already know. It does not adjudicate professional conduct or set professional standards. Concerns reported through ATAH are routed to the relevant regulatory body, never investigated by ATAH itself.
- **Operate as a paid lead-generation marketplace.** Matching is never weighted by commercial relationships. Payment funds data integration and protocol participation, not ranking.
- **Function as a consumer-facing product.** Consumers reach ATAH only through their AI system. There is no ATAH website where users browse for professionals.
- **Solve every category of professional service from day one.** Some categories require specific compliance work — for example, healthcare requires HIPAA-compliant handling — before they can be surfaced in matching. Those categories can register, but are excluded from match results until the relevant compliance work is signed off.
- **Warrant outcomes.** ATAH commits to specific data freshness windows, dispute resolution timelines, and concern-routing protocols. It does not warrant the suitability, competence, or behaviour of any individual professional.

ATAH presents verified *options* against the query — it does not recommend, endorse, or vouch for any specific professional. AI platforms integrating with ATAH are expected to preserve that framing in how they present results to users, and not describe ATAH-sourced professionals as "recommended," "approved," "best," or "endorsed" by ATAH. The `presentation_disclosure` block in every match response captures this requirement in machine-readable form.

**On responsibilities.** ATAH is infrastructure. AI platforms are the consumer-facing entity — they have the user relationship, the consent capture obligation, and the user-facing presentation responsibility. ATAH commits to provenance visibility, freshness windows, conflict suppression, dispute resolution timelines, concern flag escalation, audit integrity, and the verification confidence signal. ATAH does not warrant outcomes, professional competence, or real-time accuracy beyond published freshness windows. The PRD §11 sets out the allocation in detail.

## A note on terminology — "consumer"

Throughout this explainer and the spec, "consumer" is used as a protocol-neutral term for the person seeking professional services. In real-world contexts the equivalent term is usually "client" (legal, accounting), "patient" (healthcare), or "policyholder" (insurance). ATAH never communicates with the consumer directly — interaction is always mediated by an AI platform — so "consumer" is the term the protocol uses internally.

## Three protocol components: Discovery, Consumer handoff, Referral connection-making

ATAH is structured as three components. Component 1 is the foundation; Components 2 and 3 are optional layers on top, invoked by AI agents based on what the user wants to do next.

**Component 1 — Discovery.** A query to ATAH for candidate professionals. Required: a `request_intent` parameter declaring why the request is being made — `self` (acting for a consumer seeking a professional for themselves), `on_behalf_of_client` (a professional's AI assistant identifying candidates for a client referral), or `referral_partner_search` (a professional's AI assistant searching for new referral partners). `request_intent` gates which next-step components are available. Discovery returns a working candidate set with provenance, verification status, and a `presentation_disclosure` block that AI platforms must surface. No personal data flows in Component 1; rate-limit posture is the strongest of any operation class.

**Component 2 — Consumer-self handoff.** Available only when Discovery was called with `request_intent: 'self'`. The AI agent chooses one of three flow variants based on user preference and category requirements:

- *Off-protocol* — the AI takes the Discovery result and acts independently; ATAH is not invoked again. Suited to low-stakes introductions where the consumer just wants to know who to call.
- *Contact-share* — a single ATAH-mediated step. Consumer consents (consent receipt captured by the AI, hash stored at ATAH); contact details flow through the transient encrypted vault; professional retrieves through authenticated retrieval; vault crypto-erases on retrieval. No Stage 1 appetite check, no Stage 2 pre-handoff check.
- *Full lifecycle* — the Stage 1 / Stage 2 / Stage 3 flow described below. Suited to regulated categories where the pre-handoff check is operationally meaningful (lawyers running conflict checks, healthcare professionals confirming scope, financial advisors confirming RIA fit).

The professional-on-behalf-of-client case (formerly v0.8.1's "Type 3") is **not** part of Component 2. It is served by Discovery alone — the professional gets the candidate set and handles the introduction through their own off-platform channels using the consent they captured directly with the client. ATAH does not deliver client contact details based on professional attestation, ever.

**Component 3 — Referral connection-making.** AI-mediated mutual matching for professionals who want to find new referral partners. Standalone — not connected to Components 1 or 2 in a lifecycle sense, though it uses Discovery (with `request_intent: 'referral_partner_search'`) as its candidate-identification step. A professional opts in by setting `looking_for_referral_partners: true` (default off, toggleable). When professional A proposes a connection, ATAH records a persistent proposal with a default six-month expiry. Professional B may accept (creating a mutual connection record), decline (proposal deleted), or ignore (proposal silently lapses and is deleted at expiry, no notification). Connection records are kept for one operational purpose only — de-duplication of future referral-partner Discovery results — and **do not feed matching as a competence or trust signal**. Component 3 actions are governed by professional delegated authority; consumer consent receipts are not accepted on Component 3 endpoints.

These three components together let ATAH support the consumer handoff problem (Components 1 and 2), the professional-on-behalf-of-client referral case (Component 1 with the `on_behalf_of_client` intent), and the professional referral network historically built by hand (Component 3).

## "Isn't this just X?" — common misreads

Four readings ATAH gets that miss what it actually is:

**Isn't this just LinkedIn for AI?** No. LinkedIn is a consumer-facing professional directory people browse and message through. ATAH has no consumer-facing surface — consumers reach it only through their AI system. ATAH does not host professional profiles for users to browse, does not run a messaging product, and does not weight matching by commercial relationships. The point is provenance-visible handoff, not professional networking.

**Isn't this just Avvo / Healthgrades / Martindale-Hubbell with a different label?** No. Those are review-driven professional directories with their own ranking logic and commercial models. ATAH treats review platforms as one specialised partner class among several, with capped weight per category and an anti-gaming attestation requirement. Review data feeds in alongside regulatory data, professional body data, and enhanced verification — never as the dominant signal, never opaquely blended into a single trust score.

**Isn't this just an MCP server for some bar association data?** No. ATAH is a transport-neutral protocol with MCP and REST as its v0.8 bindings. The protocol defines the trust, provenance, consent receipt, and handoff lifecycle objects independently of how they're transported. The MCP binding is one way to access ATAH; the protocol itself sits a layer above. A bar association connector is one possible trusted partner integration; the protocol's value is the standardisation across categories, jurisdictions, and partner types.

**Isn't this a regulator-blessed directory?** No. ATAH is not a regulator and does not adjudicate professional conduct. Regulators may participate as trusted partners (state bar associations, NIPR-style licensing data sources, FINRA-style registers, medical boards are representative examples of the type), but ATAH surfaces what regulators already know rather than replacing or supplementing their authority. Concerns reported through ATAH are routed to the relevant regulatory body, not investigated by ATAH itself.

The framing that fits is: ATAH is the trust and handoff layer for agentic AI. It composes with MCP, A2A, OAuth 2.1, and W3C Verifiable Credentials. It exposes professional records through provenance-visible match responses. The user's AI makes the call; ATAH supplies the structured evidence.

## How a consumer introduction works

A consumer is talking to an AI system about something — a contract dispute, a tax liability they don't understand, a corporate communications issue, a structural concern at home. The AI does what it can. At some point, it identifies that the user needs an actual human professional.

Here is the journey ATAH supports:

**1. Find.** The AI system queries the ATAH endpoint with structured information about the matter — category, type, location, urgency. No personal data at this point. ATAH returns a small ranked shortlist of relevant professionals, each with verification status, credential information, response time commitments, and trust signals from contributing partners. Every match response also includes a `presentation_disclosure` block making the non-recommendation framing explicit.

**2. Decide.** The AI system presents the options to the user, applying its own contextual understanding of who that user is and what they need. The user makes the choice.

**3. Stage 1 — appetite check.** ATAH sends a structured introduction to the chosen professional containing only the matter type and field. No personal data yet. The professional indicates whether they have appetite and capacity to take the matter on.

**4. Stage 2 — pre-handoff check.** If the professional accepts, ATAH passes a small, category-appropriate set of information for the professional to clear before the full handoff. For lawyers this is a conflict check (consumer name, opposing party where relevant). For tax planners it might be a scope confirmation. For other categories it might be capacity confirmation or fee-range alignment. The data goes into a transient encrypted vault — not into anyone's email or SMS — and the professional retrieves it through an authenticated link. After the check resolves, the data is crypto-erased.

**5. Stage 3 — handoff.** If the pre-handoff check is cleared, the consumer is asked for explicit consent (a fresh consent receipt) to share their full contact details with this specific professional. On consent, the contact information goes into the transient vault and the professional retrieves it through authenticated retrieval. After retrieval, the data is crypto-erased. From this point, the relationship is between the consumer and the professional. ATAH steps out.

The whole journey is designed so that consumer personal data passes through ATAH only when explicitly needed and is destroyed as soon as it has done its job. **No personal data is ever sent through SMS or email.** Only notifications and authenticated retrieval references. ATAH is a routing and trust layer, not a database of consumer information.

## Who's acting on whose authority

Every ATAH action answers four questions: **who is authenticated**, **who they represent**, **on whose authority**, and **what they're allowed to do**. v0.8.2 makes that structure explicit in a single shared shape used wherever actor information is recorded — query, handoff, consent receipt, audit event.

Three things get recorded:

- The **authenticated actor** — the directly authenticated party (an AI platform, professional, partner, verifier, admin, or system role). Consumers don't authenticate to ATAH directly; in consumer flows the AI platform is the authenticated actor and the consumer is identified separately as the represented principal.
- The **client application** — for AI-platform calls, which platform and client it is. Required for AI-platform actors, optional otherwise.
- The **authority context** — on whose behalf the action is being taken (consumer, professional, firm admin, partner, verifier, governance admin, or no delegation), the credential class establishing that authority (user session, professional delegated token, partner credential, handoff access token, etc.), the scopes permitted, expiry where applicable, and any object-level constraints ("may act only on this handoff", "may manage this professional profile").

A consumer-mediated discovery call looks like: `ai_platform` is the authenticated actor; the platform's `client_application` is recorded; the authority context says `represented_principal_type: consumer`, `authority_basis: user_session`. A partner data push has the partner as the authenticated actor, no client application, and a `partner_credential` authority basis representing the partner itself. A scheduled vault-erasure event is a `system` actor with `authority_basis: system_role` and no represented principal.

The model lets every audit entry, consent receipt, and protocol decision be traced back not just to "who did this" but to "under what authority". ATAH verifies the integrity of the authority context it is presented with; the asserting platform, professional, partner, or verifier remains responsible for the validity of the underlying credentials and any consent ceremonies they imply.

## Consent — three types, one excluded by design

ATAH supports two consent types and explicitly excludes a third:

- **Query authorisation** — the user permits their AI to request candidate professionals. No personal data flows; the receipt covers the Discovery query.
- **Disclosure consent** — the user permits specific data to be shared with a specific professional for a specific introduction stage. Personal data may flow through the transient encrypted vault per the staged-handoff design.
- **Engagement consent** — the consent that creates a professional-client relationship: any professional-client agreement, fee agreement, treatment consent, regulated advice consent, conflict waiver, scope-of-work acceptance, or instruction to act. **Engagement consent is outside ATAH by design.** ATAH consent receipts MUST NOT be represented as engagement consent; the schema's enum has no value for it. Engagement consent is the professional's responsibility, captured through their own onboarding, engagement, regulatory, or contractual process.

This three-way distinction matters because the same word "consent" carries very different legal weight across the three concepts. ATAH is not a substitute for a professional's intake process; it is the trust-and-handoff layer that gets the consumer to the professional with verified identity, structured matter context, and provenance-visible verification. What happens after the handoff — fee letters, conflict waivers, treatment plans, instructions to act — is the professional's domain, captured through their own process under their own regulatory framework.

## Consent receipts — what they prove, and what they don't

Earlier versions of ATAH treated consumer consent as a boolean flag — the AI platform asserts "consent: true" and ATAH proceeds. That's not enough for high-trust professional handoff.

In v0.8, every consent action is captured as a **structured consent receipt**. The receipt records: the consent type (query authorisation or disclosure consent), the scope (which stage, which data), the data categories covered, the professional involved, the asserting platform, the consent text the user was shown, the timestamp, the expiry, and whether revocation is supported.

The asserting AI platform constructs the receipt and submits it to ATAH via `POST /v1/consent-receipts`. ATAH computes a SHA-256 hash, computes a small non-identifying continuity-binding metadata block (the asserting platform's client identifier, an HMAC of the consumer reference, and a hash of the data categories), stores the hash and metadata, and returns a `consent_receipt_id`. Subsequent consenting endpoints reference the receipt by id only — the full receipt body is never re-submitted. The asserting platform retains the full receipt for dispute resolution. ATAH never stores the consumer-identifying fields.

The continuity binding closes two abuse vectors that v0.8.1 left ambiguous: a platform submitting a receipt for one user then attempting to consume it for an action concerning a different user (cross-user confusion); and receipt replay across sessions (the same receipt consumed multiple times for unrelated actions). Both produce continuity-binding mismatches at consumption time and are rejected.

This pattern is borrowed from how transparency logs work for software supply chains and certificate authorities. It is designed to support evidence of consent and align with consent-receipt models such as GDPR Article 7 and ISO/IEC 29184, without making ATAH a personal data store.

What the receipt model proves and what it does not: the receipt-and-hash pattern proves that the consent receipt has not been altered between the moment the asserting platform captured consent and the moment a dispute arises. It does not, on its own, prove that the user actually saw or understood the consent text. Responsibility for capturing consent properly — the consent ceremony, the wording, evidence that the user actively consented — sits with the asserting AI platform under its developer terms. ATAH verifies receipt integrity; the platform is responsible for the consent itself. The v0.9 platform-light consent mode (see ROADMAP) will offer an alternative model where ATAH provides more of the consent ceremony directly, for platforms that prefer to invoke a standard ATAH-driven flow.

Cancelling and revoking consent are first-class operations. The MCP tools `cancel_introduction` and `revoke_consent` are equally available alongside the consent-submission tools. Privacy-first means revoke must be as easy as submit.

What revocation actually achieves depends on the stage of the introduction.
Before Stage 2, revocation is clean — no professional has seen any data.
After a Stage 2 pre-handoff check has been completed, the professional has
seen the scope-confirmation data; that cannot be unseen, though no further
data flows. After Stage 3 retrieval, the professional has the consumer's
contact details; the transient vault is crypto-erased on revocation, but
the professional's own systems hold the data and ATAH cannot compel its
deletion. Revocation in all cases prevents further ATAH-mediated use of the
introduction.

## Three different operations on data

When data is removed from ATAH, three different things might be happening, and v0.8.2 keeps them deliberately distinct.

**Payload erasure** — removing or crypto-erasing the sensitive content itself. Stage 2 pre-handoff data and Stage 3 contact details flow through the transient encrypted vault and are crypto-erased on retrieval, expiry, cancellation, or revocation. Crypto-erasure means the data-encryption key is destroyed; the encrypted ciphertext may remain in storage or backups until ordinary retention expiry, but it is treated as unrecoverable once the key is gone. ATAH's vault implementation requires unique keys per object, keys stored separately from ciphertext, destroyed keys excluded from recoverable backups, no plaintext in logs or queues or analytics or notification providers, auditable key-destruction events, and short single-use retrieval tokens.

**Audit retention** — preserving tamper-evident metadata about what happened. Audit records carry pseudonymous identifiers, integrity references (consent receipt id and hash, request_intent, stage, payload type), timestamps, the principal-delegation context (who acted, on whose authority), abuse flags, and HMAC-protected references to contact identifiers and devices where lawful. Plain hashes of email or phone numbers are guessable across the population they identify and are explicitly excluded; HMACs with protected audit keys are required. Audit retention applies to metadata, not to payload content; the two operations are deliberately separate so that erasing sensitive content does not erase the evidence of what happened.

**Withdrawal** — a state transition stopping future protocol processing. Withdrawal does NOT erase audit records, consent receipt metadata, state history, abuse signals, dispute records, or legally required retention records. ATAH distinguishes six withdrawal scenarios (consumer before data sharing, consumer after Stage 2, consumer after Stage 3 retrieval, professional withdrawal from an introduction, professional withdrawal from matching, Component 3 proposal or connection withdrawal), each with its own semantics. Two of these — professional withdrawal from matching, and revocation of a professional delegated-agent token — are high-impact and require step-up authentication at the point of action.

The reason for keeping these three concepts separate is concrete: if payload erasure also removed audit metadata, ATAH would become harder to investigate when abuse, spam, phishing, bad consent assertions, weak authentication, or disputes occur; if withdrawal meant deletion of lifecycle history, withdrawal itself would become an evidence-deletion abuse primitive. Separating the three keeps each operation honest about what it does.

## How the handoff is secured — and why it matters

A subtle but important point: in earlier versions, holding a `handoff_id` was enough to read or change the introduction. That sounds harmless, but `handoff_id`s end up in logs, in screenshots, in support tickets, in URLs. Possession alone shouldn't grant power.

In v0.8, access is tiered:

- **Reading status** is available to the original AI platform that started the introduction, *and* to any authenticated AI platform that supplies a consumer reference matching the handoff. This means a user moving between AI platforms can still check on their introduction — cross-platform continuity is preserved.
- **Changing state** (advancing stages, submitting data, cancelling, revoking, reporting outcomes) requires a separate `handoff_access_token`, bound to the platform that started the introduction, scoped to the handoff, time-limited, and revocable.

The result: random possession of a `handoff_id` cannot expose data or move an introduction forward. The cross-platform user experience is preserved. The bearer-token vulnerability is closed.

## Matching, randomisation, and not being a recommendation engine

ATAH does not rank candidates by preference. The matching engine applies hard filters (category, jurisdiction, matching status, compliance status, contact freshness, availability, category-required verification thresholds), groups the remaining eligible candidates into transparent bands of similar verification confidence and fit, and randomises or rotates within each band under a documented fairness policy. There is no global score; there is no hidden ranking. The structural commitment in v0.8.2 is that ATAH may determine eligibility and exclusion, but does not express preference among eligible candidates unless the user has asked for an explicit ordering.

When a user has an explicit ordering preference — nearest, soonest available, remote available, a specific language, highest verification confidence, a specific qualification, professional type, or jurisdiction — ATAH applies that mode and records the choice on the response. When the user has no preference, the response is ordered by stratified randomisation; the same query repeated produces different orderings within bands, and the randomisation seed is not disclosed (disclosing the seed would create a gaming surface).

Review platform signals — where present — supplement the transparency view but do not move a candidate into a higher band in regulated categories unless corroborated by an authoritative source like a regulator or professional body. This preserves the original v0.8.1 safeguard (review-derived signals can't dominate matching in high-stakes categories) under the band-based model.

AI platforms reordering ATAH's response downstream MUST preserve the disclosure that ATAH applied non-preferential ordering. The protocol's "we are not a recommendation engine" commitment is structural, not just declarative — there is no global score in the schema, the band assignment is exposed for inspection, and the conformance test verifies that within-band ordering is observably non-deterministic across repeated queries.

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

The protocol records this distinction in a structured field, `vetting_strength`,
with three values: `regulatory` (a regulator with statutory or constitutional
authority over the profession), `strong_membership` (a membership body with
chartered/fellow status, CPD requirements, and active disciplinary processes),
and `open_membership` (a membership body where membership is essentially open
to anyone who pays the joining fee). The classification is set by ATAH
governance at partner approval and may only be revised through governance
review, never by the partner unilaterally. Match responses surface the
classification so AI platforms can present partner-verified status with
appropriate context.

In public-facing terms, ATAH classifies trusted partners into six types: **authoritative regulatory sources** (regulators with statutory authority — state bar associations, medical boards, and equivalents); **professional membership bodies** (chartered or fellow-status bodies with CPD and disciplinary processes — CIPR, the International Coaching Federation, PMI, and equivalents); **open membership bodies** (where membership is essentially open); **review signal providers** (review platforms contributing aggregated feedback subject to anti-gaming attestations); **independent verifiers** (approved enhanced-verification providers); and **public registry sources** (partners surfacing public-record data such as companies-house-style data or registries of accredited training providers). The first three map onto the three `vetting_strength` values; the others are complementary classes that contribute different kinds of signal.

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

**AI platforms** get a common ATAH-compatible handoff contract — available through the v0.8 MCP and REST bindings — that returns verified professional options when their users need human expertise. No bespoke integration per professional category or per jurisdiction. No directory deals to negotiate. Free to query (no per-query or licensing fee — querying still requires authentication for protocol integrity). Conforms to current MCP authorisation guidance. The reference registry will provide the first conforming endpoint when operational; future conforming implementations may expose the same protocol.

The proposition to AI platforms is direct: supporting ATAH addresses the broken handoff problem that every AI system currently hits. It turns a limitation into a capability without each platform having to solve it independently.

**Professional associations** get a practical answer to the question every member is asking: *how do I stay visible and trusted when AI becomes the first call for professional advice?* Associations participate as trusted partners — contributing what they already know about their members, and giving those members structured machine-readable presence in AI-mediated environments. The data already exists in their member registries. ATAH makes it useful at the moment AI systems need it.

**Credentialled and established professionals** get a way to be discoverable by AI systems as part of their AI-discoverability infrastructure. Their standing is verified and kept current, then surfaced wherever AI systems reach the human-handoff moment. No platform lock-in. Concern flag protections — admin-only visibility, right of reply, pattern-based review — guard against defamation through bad-faith reports.

ATAH is not a replacement for a professional's existing marketing or visibility work. It is infrastructure for being represented to AI systems consistently and credibly. Professionals continue to need their own marketing for direct consumer reach; ATAH addresses the specific moment an AI system needs to identify and hand off to a verified human professional. There is no guarantee of introduction volume, ranking, or work; the protocol is designed to become a trusted handoff rail as platforms integrate.

**Trusted partners** sit in distinct classes. Authoritative regulatory sources and professional membership bodies are the credential-verification authority layer — what professionals are licensed or qualified to do, who has revoked or sanctioned them, what membership level they hold. Review platforms are a separate class — signal providers, not authority providers — contributing aggregated client feedback subject to anti-gaming attestations and per-category review weight caps (≤0.10 within verification quality for high-stakes regulated categories). Independent verifiers add a separate enhanced-verification layer. Each partner type gets a clean way to make verified data useful at the exact point users are deciding whether to trust a professional. Provenance is always visible. Review-derived signals complement but do not substitute for credential verification. The commercial model funds the work of maintaining the integration without compromising protocol neutrality.

**Consumers**, finally, get a continuation of journey rather than a dead end. When their AI system identifies that they need a professional, the AI can offer real options with real verification signals, rather than handing them back to a search engine.

## The trusted partner model

The trusted partner network is what makes the registry credible and what funds the protocol.

Trusted partners are organisations that hold reliable, maintained data about professionals and meet ATAH's partner standards. The types of organisation the protocol is designed to work with include: regulatory and licensing bodies (such as state bar associations, NIPR, NAIC, FINRA, and equivalents in other jurisdictions); professional bodies for credentialled fields (such as the American Bar Association, and bodies covering property and casualty insurance professionals like the Big "I" (IIABA), PIA, and the CPCU Society); professional bodies for established practitioner fields (such as PRSA, CIPR, the International Coaching Federation, SHRM, and PMI); and review platforms (such as Google reviews, Avvo, Healthgrades, and Martindale-Hubbell). Hundreds of similar organisations exist across hundreds of professional categories worldwide. None of the named bodies above are committed partners at v0.8 publication; each is a representative example of the type of organisation the protocol's trusted-partner framework is designed to work with.

The model is consistent across all of them: partners pay ATAH for integration access, with public fee schedules and waivers for regulators and public-interest bodies. Members gain structured machine-readable presence in AI-mediated environments — a structured answer to the AI-discoverability question. The commercial model exists to fund the work of building and maintaining the protocol. It is not the reason the protocol exists.

Three commitments are non-negotiable in this model:

- Partner payments do not influence matching ordering or recommendation status. Ever.
- Partner-provided data is always tagged with its source and verification basis. Provenance is never obscured.
- Where partners' data conflicts on something material — for example, two different sources reporting different licence statuses — the affected profile is suppressed from matching until the conflict is resolved.

Trust and commercial logic are structurally separate. Payment funds participation. It does not buy ranking.

**On named organisations.** Throughout this explainer, named bodies (state bar associations, NIPR, NAIC, FINRA, the American Bar Association, the Big "I", PIA, the CPCU Society, PRSA, CIPR, the International Coaching Federation, SHRM, PMI, Google reviews, Avvo, Healthgrades, Martindale-Hubbell, and others) appear as illustrative examples of the *types* of organisation the protocol is designed for. They are not committed partners. They appear because they are the most recognisable bodies in their respective categories and jurisdictions, helping readers see who the trusted-partner framework is for. Becoming a partner requires a partner agreement, data integration work, and governance approval — none of which is in place at v0.8 publication.

## Privacy and data — structural, not policy

ATAH is built around minimising central retention of consumer personal data. This isn't a policy that could be loosened later — it's a Charter Part One entrenched commitment that requires supermajority governance to change.

The system distinguishes data categories and treats each separately:

- **Professional identity data** — names, credentials, contact preferences. Core registry data, retained for as long as the professional is registered. They control it.
- **Trust and verification metadata** — partner-contributed data points, verification records, validator provenance. Retained and tagged at all times.
- **Consent receipt metadata** — hash and metadata only. No personal data.
- **Introduction lifecycle data** — pseudonymous state transitions logged. No personal data.
- **Consumer personal data** — strictly transient. Held only in the encrypted vault during active handoffs. Crypto-erased on resolution. **Never written to the main registry database. Never transmitted through SMS or email.**
- **Audit log** — pseudonymous identifiers only. Tamper-evident. Retained 7 years for regulatory expectations.

Every introduction requires fresh explicit consent, captured as a structured consent receipt. ATAH never re-uses personal data from a previous introduction.

The transient encrypted vault and the authenticated retrieval portal mean personal data never sits in a notification provider's logs, an inbox, a phone notification history, or a device backup. The professional gets a notification with no personal data; they tap the link; they authenticate at the appropriate tier (light for low-sensitivity scope confirmations; step-up authentication for healthcare or sensitive legal matters); they see the data; the data is then crypto-erased.

A note on professional data. The privacy commitments above describe how ATAH
handles consumer personal data — minimised, transient, never stored centrally.
Professional data is treated separately. ATAH holds and maintains professional
records, and professionals have the standard data protection rights —
access, rectification, objection, erasure (subject to public-interest
carve-outs for regulator-sourced data), restriction, and portability.
Professionals appearing through a trusted partner are notified by their
partner; self-registered professionals are notified at registration.
Detailed treatment is in the PRD and specification.

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

The founder retains a continuing advisory seat with no veto rights, no preferential commercial advantage, and explicit conflict-of-interest, recusal, and disclosure rules. After transition the seat carries consultation rights but not approval rights for core amendments.

The full governance commitments are set out in [CHARTER.md](CHARTER.md).

## What comes next

This is a v0.8 release candidate. The work that follows falls into four streams:

- **Community review.** The specification needs eyes on it from people building AI products, working in professional standards, running professional bodies, and thinking about related problems in the standards space. Feedback is invited via GitHub issues and discussions.
- **Trusted partner conversations.** First conversations are open with authoritative regulatory sources and professional membership bodies as the credential-verification authority layer, and separately with review platforms as signal providers. The natural first movers are the categories where the AI-discoverability question is sharpest right now — legal, property and casualty insurance, communications, coaching, project management.
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
