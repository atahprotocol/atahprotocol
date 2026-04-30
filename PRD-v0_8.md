# ATAH

**Agent to Authenticated Human Protocol**

**Product Requirements Document**

| Version | 0.8 |
| --- | --- |
| Status | Release Candidate |
| Date | April 2026 |
| Founded by | Grahame Cohen |

---

## 1. Purpose of This Document

This PRD defines what ATAH is, who it serves, what it must do, how it is structured, and what success looks like. It is the business-facing source of truth for the ATAH build. It should be read after the [README](README.md) and [EXPLAINER](EXPLAINER.md), and before the technical specification at [spec/v0.8/](spec/v0.8/).

It is written for developers, contributors, association partners, trusted partners, peer reviewers, and anyone making decisions about what to build and in what order.

The governance commitments that constrain how ATAH operates and evolves are in [CHARTER.md](CHARTER.md). This PRD reflects those commitments throughout.

## 2. Problem Statement

AI systems are becoming the first place many users turn when they have legal, medical, financial, technical, or other professional questions. In lower-stakes cases, AI may be enough. In higher-stakes cases, AI can often identify that human expertise is required, but it cannot reliably complete the transition to trusted human help. The user is left with a warning, a disclaimer, or an unstructured search problem.

This creates a structural problem for three groups.

**For consumers.** When an AI reaches the limit of what it can handle alone — a legal matter requiring licensed advice, a property and casualty insurance question requiring a licensed agent, a tax matter requiring a qualified planner, a PR crisis requiring experienced judgment, a structural concern requiring a certified engineer — there is currently no common, trusted, machine-readable way for it to move a user from AI interaction to validated human help.

**For professionals.** Credentialled and established professionals across every field are largely invisible to AI systems. There is no machine-readable standard for representing who they are, what they handle, where they practise, and how they want to be engaged. Answer Engine Optimisation is a specialism most professionals will never master. They need infrastructure that supports this consistently, at scale, through a structure they can join without becoming AI experts. At the same time, professionals have always relied on referral networks. Building those manually takes years.

**For AI platforms.** Without a trusted, structured, and machine-readable source of professional identity and trust signals, AI systems cannot safely and consistently move users from general AI interaction to validated human help. Building bespoke integrations with every professional directory in every jurisdiction is not feasible. A shared open standard addresses this for every platform simultaneously.

There is also a structural commercial opportunity for professional bodies. Every association's members are asking the same question: how do I stay visible and relevant when AI becomes the first port of call? ATAH gives associations a concrete, practical answer — and a new revenue stream built on what they already know about their members.

ATAH — Agent to Authenticated Human Protocol — addresses these problems with a single open standard.

## 3. Vision

ATAH aims to become the common trust, provenance, consent, and handoff protocol layer through which AI systems can move from AI-only interaction to verified human professional engagement — across multiple sectors and jurisdictions worldwide — while also supporting verified professional-to-professional referral pathways.

The initial ATAH reference registry is the first implementation of that layer, not the limit of the protocol. Where MCP standardises AI access to tools and data, ATAH standardises the trust and handoff payload for reaching verified human professionals. ATAH composes with MCP, A2A, OAuth 2.1, and W3C Verifiable Credentials rather than competing with them — and can itself be exposed through MCP, REST, or future bindings.

## 4. Guiding Principles

- **Protocol before implementation.** ATAH defines the handoff contract first: professional records, provenance, consent receipts, lifecycle states, presentation disclosures, and conformance rules. The reference registry is the first implementation of that contract — not the protocol itself. Product, commercial, and operational choices must not collapse the protocol into a proprietary referral network.
- **Open by default.** The protocol specification is public and royalty-free to implement. The MCP endpoint is free to query (free meaning no per-query or licensing fee — querying still requires authentication for protocol integrity). ATAH does not impose commercial weighting on matching.
- **Composes with existing standards.** ATAH builds on OAuth 2.1, OpenID Connect, and W3C Verifiable Credentials. ATAH's contribution is the layer above — professional identity, the introduction lifecycle, the matching engine, the trust model. Where existing standards already do part of the job, ATAH uses them rather than reinventing.
- **Honest about what it knows.** Every data point carries explicit verification status and source via a parallel provenance map. Self-declared, partner-verified, registry-verified, and VC-verified are tagged distinctly. Never presented with false confidence.
- **Neutral by design.** Matching is based on relevance, verification quality, availability, response time, profile completeness, and inbound referral signal. No commercial weighting. No promoted positions.
- **Non-endorsement by design.** ATAH is not a recommendation engine. It returns provenance-visible shortlists. The user or calling AI platform remains responsible for selection. Every match response carries a `presentation_disclosure` block making this machine-readable.
- **Structural separation of trust and commercial logic.** Commercial participation, integration fees, verifier governance fees, and partner status do not directly determine user-facing ordering or matching outcomes. Where paid services generate verification evidence, that evidence is scored under the same public rubric available to all approved sources.
- **Bidirectional by design.** ATAH manages the full lifecycle of every introduction in both directions, with explicit consent at each stage captured as a structured consent receipt.
- **AI-assistant first.** ATAH actively encourages professionals to connect their AI assistant as the primary channel for receiving and managing introductions.
- **Privacy as a structural commitment, not a policy.** Consumer personal data is never written to the main registry database. It is held only in a transient encrypted vault during active handoffs and crypto-erased on resolution. Notification channels carry no personal data — only authenticated retrieval references. This is built into the protocol, not added on top.
- **Independent governance on trigger-based transition.** Governance is committed to transitioning to an independent structure when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated. Transition progress is reported publicly when there is meaningful progress to share. The trigger framework reflects that cross-industry standards work runs on multi-month-to-multi-year decision cycles, not software-industry calendars.
- **Built for federation, single-registry at v0.8.** The protocol architecture supports federation. v0.8 launches with a single reference registry. Federation mechanics deferred to v0.9 / v1.0.

## 5. Professional Scope

ATAH is a universal open protocol for any professional in any field where expertise, standing, or credentials can be verified or attested.

ATAH's protocol is profession-agnostic at the architecture level, but category-specific launch timing, validation sources, and compliance rules will differ. A profession being within protocol scope does not mean it is enabled for matching at launch. Some categories may register from day one but be surfaced in matching results only once the relevant validation pathway and compliance checks are confirmed.

### Two categories of professional

**Credentialled professionals** hold a formal licence, certification, or accreditation from a recognised regulatory or professional body. Their standing can be verified against an authoritative source. Examples: lawyers, property and casualty insurance agents, financial advisors, doctors, engineers, tax planners, accountants, pharmacists, architects.

**Established professionals** are recognised practitioners in fields where formal licensing does not exist but professional standing is real, verifiable, and meaningful. Their standing is verified through membership of professional bodies, peer recognition, client reviews, and trusted partner data. Examples: PR professionals, management consultants, marketing specialists, executive coaches, HR professionals, project managers, mediators, certified translators.

The trusted partner model handles both categories through the same infrastructure. A state bar association contributes licence and disciplinary data for lawyers. NIPR contributes licence and appointment data for insurance agents across all 50 states. The CIPR or PRSA contributes membership and fellowship data for PR professionals. A review platform contributes verified client scores. The evidence differs. The protocol is identical.

### Two registration routes

ATAH supports two routes for a professional to be present in the registry. Both routes coexist permanently.

**The partner route** surfaces professionals through their professional body, regulator, or other authoritative organisation when that organisation has joined ATAH as a trusted partner. The professionals it covers are surfaced in the registry through that partner's data. They do not register individually. Over time this becomes the dominant route as more partners join.

**The individual self-registration route** lets a professional register themselves directly. This route exists permanently for: where there is no professional body for the field; where there is a body but membership is optional and the professional has chosen not to belong; or where the relevant body has not yet joined ATAH as a trusted partner. Individual self-registration is free during the protocol's initial period after launch and continues thereafter for a small annual fee, with hardship waivers and fee caps published in operational documentation. **Paid individual registrations and partner-route registrations receive identical matching treatment** — there is no ranking difference between them.

When a trusted partner joins ATAH for a category, professionals who are members of that partner have their existing individual registrations absorbed into the partner's data through the roll-up process. For high-risk categories (regulated tier), professionals are notified before merge with a 7-day objection window.

### The Validator Stack

ATAH uses a structured, multi-class validator model:

- **Authoritative registry validation** — licensing boards, statutory registries, regulator databases. NIPR, FINRA, state bar databases, state medical boards.
- **Professional-body validation** — membership, fellowship, certification, confirmed by named bodies with active partner status.
- **Verifiable Credential (VC) validation** — W3C Verifiable Credentials issued by approved partners. Cryptographically verified.
- **Firm or institutional validation** — employer, panel membership, accredited network, or insurer panel confirmation.
- **Independent verifier validation** — enhanced verification records produced by approved independent verifiers, scored under a public rubric for breadth, depth, and freshness.
- **Experience and trust signals** — verified reviews from approved review platforms with anti-gaming attestations, weighted by review platform class and time-decayed.
- **Self-declared information** — always labelled as such.

### How verification works

Every data point in a professional's profile carries its own provenance via the parallel `_provenance` map. The map records `verification_status`, `source`, and `as_of` for every verifiable field.

- **Registry-verified** — confirmed active against an authoritative source by ATAH directly.
- **Partner-verified** — confirmed by a named trusted partner with active partner status.
- **VC-verified** — confirmed via cryptographic verification of a W3C Verifiable Credential. Treated equivalently to partner-verified for matching purposes.
- **Self-declared** — provided by the professional, not yet independently verified.

The registry gets richer as trusted partners connect and contribute verified data. Better-validated profiles carry stronger trust signals within matching calculations.

### What partners actually verify

"Partner-verified" is not a single uniform claim. Different partners verify different things. The protocol records this with a `verification_scope` field and a `vetting_strength` classification (`regulatory`, `strong_membership`, `open_membership`). This is set by ATAH governance at partner approval and may only be revised through governance review, never by the partner unilaterally.

### Compliance-pending categories

Some professional categories — notably healthcare — require specific regulatory compliance work before profiles can be surfaced in matching. Professionals in compliance-pending categories may join and create profiles from day one. Their profiles are enriched and maintained normally. They are not returned in match results until the relevant compliance annex is complete.

This is a Charter Part One core commitment: no professional category may be surfaced in matching unless category-specific minimum verification, compliance, and disclosure requirements have been approved and published.

## 6. Target Users

### 6.1 Credentialled and Established Professionals

Any professional in any field where expertise and standing can be verified or attested. Their needs:

- Being discoverable by AI systems without mastering Answer Engine Optimisation
- Receiving qualified, well-prepared introductions
- Building verified referral relationships with complementary professionals
- Responding through familiar channels
- Maintaining an accurate, current profile with minimal ongoing effort
- Protection from defamation through bad-faith concern flags (admin-only visibility, right of reply, pattern-based review)

Where their professional body has joined ATAH as a trusted partner, they appear in the registry through their body's data — no individual registration required. Where they are not represented by a partnered body, individual self-registration is available, free during the protocol's initial period after launch and continuing thereafter for a small annual fee with hardship waivers.

### 6.2 AI Platforms

Any AI system that needs to route users to human professionals. The core role of ATAH in AI platform integration is to support the transition from AI-only interaction to human professional engagement when the need for human expertise becomes clear. Their needs:

- A common ATAH-compatible handoff contract, available through the v0.8 MCP and REST bindings, so AI platforms can integrate once and understand any conforming implementation that preserves the same schemas and conformance rules
- A reliable first endpoint provided by the ATAH reference registry, with full provenance metadata
- Conformance to current MCP authorisation guidance (OAuth 2.1-compatible)
- Idempotency on mutating endpoints for safe retry behaviour
- Cross-platform status check support so users moving between AI platforms can continue tracking
- A clear consent receipt model that integrates with the platform's own consent capture flow
- Non-recommendation framing the platform can surface to users (`presentation_disclosure` block on every match response)
- No bespoke integration work per professional category or jurisdiction

Supporting ATAH creates a path for professionals to reach AI platforms without per-platform integrations — every professional in the registry has an incentive to connect their AI assistant, and every platform that supports ATAH is where those introductions arrive.

### 6.3 Trusted Partners

Organisations that hold reliable, maintained data about professionals and meet ATAH's partner standards. Both data quality mechanism and primary commercial model.

**Regulatory and licence databases.** State bar associations (priority states at launch, expanding as verification pathways are established); NIPR for insurance agent licensing across all 50 states; NAIC for state-level insurance regulator coordination; FINRA for financial advisors; state medical and nursing boards.

**Professional bodies — credentialled fields.** American Bar Association; American Medical Association; American Institute of CPAs (AICPA); American Institute of Architects (AIA); The Big "I" (IIABA) and PIA for property and casualty insurance agents; CPCU Society for chartered property and casualty underwriters.

**Professional bodies — established practitioner fields.** PRSA / CIPR for public relations; American Marketing Association / CIM for marketing; Institute of Management Consultants (IMC); International Coaching Federation (ICF); Society for Human Resource Management (SHRM); Project Management Institute (PMI).

**Review platforms.** Google reviews, Avvo, Healthgrades, Martindale-Hubbell, Trustpilot, and similar. Specialised partner class with anti-gaming attestation requirements as a condition of partner status, classified by verification rigour, with review weight capped per category.

**Independent verifiers.** Approved by ATAH governance, audited annually, providing enhanced verification commissioned by professional bodies, employers, or individual professionals.

The commercial model is consistent: partners pay ATAH for integration access (with published fee schedules and waivers for public-interest organisations), they may charge their members for accreditation or data services, members benefit from structured machine-readable presence in AI-mediated environments.

### 6.4 Professional Associations

For professional associations, ATAH provides a practical way to translate member trust, status, and standing into machine-readable form at the point AI systems need human expertise. Commercial models exist around data integration and member services, but the primary value is maintaining member relevance and trusted visibility in an AI-mediated world. Associations participate as trusted partners — contributing what they know about their members, benefiting commercially from the integration, and giving their members something genuinely new.

### 6.5 Consumers

People whose AI systems query an ATAH-conformant registry on their behalf. Their needs:

- Confidence that professionals returned are properly credentialled or established and the right fit for their matter
- Continuity of experience across AI platforms (cross-platform status check support)
- Control over consent at every stage, with revocation supported
- Minimum personal data retention — held only in the transient vault during active handoffs
- Clear non-recommendation framing so they understand ATAH presents options rather than endorses individuals

ATAH never communicates with consumers directly. All consumer interaction is through the AI platform.

## 7. Introduction Types

ATAH supports three distinct introduction types. All share the same underlying protocol infrastructure but differ in who initiates them, how personal data flows, and how consent is captured.

**Type 1 — Consumer Introduction.** A consumer's AI system queries ATAH to find a professional for a consumer matter. Cold algorithmic match enriched by the AI system's contextual knowledge of its user. Full three-stage handoff applies, with Stage 2 being a category-appropriate pre-handoff check. Consent captured as a structured consent receipt issued by the asserting AI platform.

**Type 2 — Referral Partner Establishment.** A professional's AI assistant identifies a potential referral partner in the registry — a tax planner connecting with a corporate lawyer, a property and casualty insurance agent connecting with a financial advisor, a PR specialist with a crisis communications barrister. Mutual blind opt-in — nothing flows unless both professionals independently say yes. Type 2 establishes a relationship between professionals; it does not transmit client personal data. Referral relationship recorded and becomes a quality signal in consumer matching.

**Type 3 — Professional-Initiated Client Referral.** A professional uses their AI assistant to introduce their client to another professional. Two consent paths supported: a structured client consent receipt (preferred where the client has used ATAH or an AI to consent) or a structured professional attestation of client consent (where consent has been obtained through professional practice such as a signed engagement letter authorising referrals). Client contact details are stored in the transient encrypted vault and crypto-erased after retrieval.

## 8. Core Capabilities

### 8.1 Professional Registry

A structured, machine-readable registry of credentialled and established professionals, supporting both registration routes (partner and individual). Seeded at launch from publicly available sources and partner data. Professionals who are members of a partnered body are surfaced through their partner's data. Professionals not represented by a partnered body register individually. Data is enriched over time through self-declaration, document upload, trusted partner data contributions, Verifiable Credentials where supported, enhanced verification (where commissioned), and AI assistant connection.

### 8.2 Profile Data Model

Each profile contains the following data categories. Fields returned only where data exists. Each data point carries its own verification status via the parallel `_provenance` map.

- Identity and credentials — name, firm or practice, jurisdictions, licence or membership number where applicable, issuing body, status, admission or join date
- Registration route — partner or individual
- Location fields split: `licensed_jurisdictions`, `service_locations`, `physical_locations`, `remote_service_available`. Category-specific matching rules govern how each is used.
- Experience — years in practice, years in specialism, typical matter or engagement size and complexity
- Specialisms and matter types — professional category, specialism tags, types of work handled
- Certifications and accreditations — formal qualifications and designations beyond the base credential
- Professional indemnity insurance — whether held, coverage level, insurer where applicable
- Response time commitment — declared response time, with operating timezone for clarity
- Engagement preferences — fee model, consultation terms, virtual or in-person, languages, geographic reach, availability status
- Referral network — established mutual referral relationships contributing to consumer matching quality signal, subject to anti-gaming controls
- Trusted partner data — extensible array of data points contributed by trusted partners, each tagged with source, validator class, vetting strength, trust tier, and last updated timestamp
- Enhanced verification records — optional array of independently audited verification records (Section 8.4)
- `_provenance` map — parallel record of verification status, source, and last-updated timestamp for every verifiable field

The atah_id is opaque (ULID-style with a typed prefix). Country, profession, and category are stored as attributes, not encoded in the identifier.

### 8.3 Verification Engine

Automated verification against authoritative sources for credentialled professionals where accessible — including NIPR for insurance agents (all 50 states), state bar databases for lawyers in priority states, and FINRA for financial advisors. NIPR verifies licensing and appointment data where available; category fit (lines of authority, carrier appointments, P&C focus, farm/commercial/personal specialisation) still depends on declared and partner-verified specialisms.

For established professionals, verification is via trusted partner data. W3C Verifiable Credentials issued by professional bodies are accepted where supported. Verification frequency: on joining, every ninety days thereafter, and triggered on any match query where last verification was more than thirty days ago.

### 8.4 Trusted Partner Model

Trusted partners contribute verified, maintained data to the ATAH registry. Both core data quality mechanism and primary commercial model. The model is particularly important for established professionals, where trusted partner data is the primary source of verifiable standing.

Partners pay ATAH for integration access. Public partner fee schedules with cost-recovery rationale. Fee waivers available for regulators and public-interest organisations. Public partner registry showing partner status, scope, and approval date. No exclusive arrangements within any category.

A core condition of trusted partner status is keeping data current. Partners are expected to push updates within their declared SLA when a professional's status changes. Partner data quality is scored continuously. Stale data is flagged and deprioritised. Trusted partner status means something operationally, not just commercially.

It is fundamental to ATAH's integrity that partner payments do not influence user-facing ordering or matching outcomes. Where partner-provided data influences trust presentation, the source and verification basis remain transparent.

#### Independent verifiers and enhanced verification

ATAH supports an optional enhanced verification layer. This is a deeper, ongoing, independently-audited verification commissioned by professional bodies (for their members), employers (for their employees), or individual professionals (for themselves). It is administered by approved independent verifiers — separate from professional bodies, separate from review platforms, and approved by the protocol governance body before they may operate.

Enhanced verification covers dimensions like identity, qualifications, professional standing, professional history, insurance, references, client feedback, compliance obligations, and sanctions screening. The verifier confirms what was checked and to what depth. ATAH computes the trust contribution from the structured data — verifiers cannot set their own scores. The result is a strong, transparent additional trust signal that AI systems can interpret alongside other verification.

This is particularly valuable for individual self-registered professionals without a strong body behind them, and for professional bodies wanting to offer their members a premium tier of verification.

The commercial model for verifiers:

- The verifier sets the fee for verification work, paid by the commissioning party. ATAH does not receive a share of verification fees and does not set verifier prices.
- Verifiers pay ATAH a non-refundable assessment fee at application and an annual audit fee thereafter, both tiered by declared scope (`narrow_scope`, `multi_scope`, `global_scope`). These cover governance and audit work.
- Public fee schedule. Fee waivers for public-interest organisations.

**Enhanced verification is not a promoted-placement mechanism.** It is a paid route to a stronger trust signal scored under the same public rubric available equally to all approved verifiers. The party who paid for the verification carries no weight in matching. Where the structured verification depth, breadth, and freshness are higher, the matching contribution is higher — regardless of who paid.

#### Review platforms

Review platforms — Google reviews, Avvo, Trustpilot, Healthgrades, Martindale-Hubbell, and similar — are a specialised partner class. The protocol treats review platform data as a distinct signal class with specific requirements.

Review platform partner status requires the platform to attest, as a condition of integration, to controls covering reviewer identity verification, engagement verification, outlier and pattern detection, removal transparency, and self-review prevention. Annual re-attestation. Audit rights. Suspension criteria. Public classification rationale.

Review platforms are sub-classified by verification rigour: `verified_transaction` (proof of actual transaction required), `verified_account` (verified account but no transaction proof), or `open_review` (no verification). The classification is set by ATAH governance and is not amendable by the platform unilaterally.

Review-derived signals contribute only above a minimum volume threshold (default 10 reviews per professional) and are time-decayed (full weight under 24 months, reduced weight 24–60 months, no contribution beyond 60 months). **Review weight is capped per category** in `profession-categories.json` — for high-stakes regulated categories the cap is ≤0.10 within verification quality. Reviews are tagged as a separate sub-component, never blended into a single opaque trust score.

### 8.5 Matching Engine

AI systems query an ATAH-conformant registry via the MCP or REST binding. The registry runs the matching algorithm and returns a set of relevant validated professional options with the full trusted partner data payload, the `presentation_disclosure` block, and per-sub-component contribution metadata. The AI system applies its own contextual layer to refine and present options.

Matching should be understood primarily as eligibility filtering plus transparent trust-signal presentation. Ordering reflects declared relevance and trust inputs. ATAH does not make a substantive recommendation that a particular professional is the correct or best choice for a particular outcome. Ordering must not be affected by commercial payments.

**Per-category matching weight profiles.** Categories may define `matching_weight_profile` in `profession-categories.json`. Regulated categories (lawyer, physician, financial advisor, insurance agent) weight verification quality higher than availability. Lower-regulatory categories may weight availability and relevance more heavily. Category-specific overrides require governance approval. v0.8 ships with defaults plus exemplar overrides for representative regulated and non-regulated categories; researched per-category weights for all launch categories are a v0.9 task.

**Scoring components:** relevance, verification quality, availability and response time, profile completeness, and inbound referral signal. The verification quality component has four sub-components: credential and standing verification (weighted by partner vetting strength), enhanced verification (where present), review platform signals (above the volume threshold and time-decayed, weighted by review platform class, capped per category), and scope completeness (split into declared and verified — only verified materially contributes). The contribution of each sub-component is preserved in match response metadata so the basis of the verification quality score is transparent.

**Anti-gaming controls on inbound referral signal:** reciprocal cap (closed pairs and triangles discounted), time decay, referrer verification quality weighting, evidence of successful referral outcomes for full weight, dense-cluster discounting. Sophisticated Sybil/ring detection deferred to v0.9.

No commercial weighting at any point. No weight assigned to which review platform contributed data beyond the platform's published verification rigour classification. The `presentation_disclosure` block on every match response makes ATAH's non-recommendation stance machine-readable; AI platforms are expected to surface its content to end users.

### 8.6 Bidirectional Protocol and Introduction Lifecycle

ATAH manages the full lifecycle of every introduction as a bidirectional protocol. State changes trigger non-PII notifications to the relevant professional. Consumer AI agents track status by polling `check_introduction_status`. Cross-platform status check supported for authenticated AI platforms with matching consumer reference. Webhook push delivery to consumer AI agents is a Phase 2 capability.

**Stage 1** — matter type, field, urgency only. No personal data. Appetite and capacity check.

**Stage 2** — category-flexible pre-handoff check. The check varies by category: conflict check (lawyers), scope confirmation (PR, tax planning), capacity confirmation (P&C insurance), fee alignment, or none (where no check needed). Personal data submitted at Stage 2 is held in the transient encrypted vault, retrieved by the professional through authenticated retrieval, and crypto-erased on resolution.

**Stage 3** — consumer contact details released after consumer Stage 3 consent receipt. Stored in the transient vault, retrieved through authenticated retrieval, crypto-erased after retrieval.

Cancel and revoke are first-class operations: `cancel_introduction` and `revoke_consent` MCP tools and corresponding API endpoints, both triggering immediate vault crypto-erasure.

### 8.7 Consent Receipts

Every consent action in ATAH is captured as a structured consent receipt rather than a boolean flag. Receipts include scope, data categories, professional/handoff reference, asserting platform, captured timestamp, expiry, revocation support, and revocation status. ATAH stores a hash of the receipt and metadata; the asserting platform stores the full receipt. At dispute time the platform produces the receipt and ATAH verifies the hash matches what was stored at consent time.

This provides demonstrable consent suitable for GDPR Article 7, ISO/IEC 29184, and similar frameworks, while keeping ATAH from becoming a personal data store.

### 8.8 Tiered Handoff Access

`handoff_id` identifies an introduction. It does not on its own grant access. **Read-like operations** (status checks) are available to the original asserter platform or to authenticated AI platforms presenting a matching consumer reference (preserving cross-platform user experience). **State-changing operations** require a `handoff_access_token` bound to platform/client/scope/expiry, issued at introduction creation, refreshed on use, and revocable on cancellation.

This model preserves cross-platform user experience for status checks while preventing possession of `handoff_id` alone from granting state-changing power.

### 8.9 Professional Notification Model

Notifications carry no consumer personal data — only metadata and authenticated retrieval references. Primary (Phase 1): SMS as a notification with a time-limited authenticated link. Parallel: email with same content. Best-effort: MCP notification to professional's AI assistant if connected. Phase 2 primary: MCP push notifications once push notification standards are stable.

Authenticated retrieval is tiered per category sensitivity:

- `tier_1` — notification metadata only (used only for low-sensitivity scope confirmation)
- `tier_2` — single-use authenticated retrieval token + device-bound session (most categories)
- `tier_3` — token + step-up authentication (SMS code or email confirmation in addition to original notification — required for healthcare, sensitive legal matters)

No username or password anywhere.

### 8.10 Dispute Resolution

Professionals may dispute data contributed by trusted partners (including enhanced verification records and review platform data) at any time via the professional portal. Disputes are logged synchronously, escalated to the contributing partner, and tracked through to resolution. The partner has a defined SLA to respond. ATAH admin reviews disputes where the partner fails to respond or where the professional provides contradicting documentation. During an active meaningful dispute, the affected data point is flagged in the profile. Suppression from matching is applied only where the disputed data creates a meaningful conflict with another source.

### 8.11 Concern Flag Mechanism

The `consumer-reported-concern` outcome code creates a concern flag against the professional's record. Concern flags are admin-only visibility — never published, never used in matching weighting. The professional named is notified of any flag against them and has a 30-day right of reply. Patterns of flags trigger admin review and may result in escalation to the relevant regulatory or professional body. ATAH does not adjudicate professional conduct. Bad-faith flag detection (rapid succession from same consumer, contradictory content) applies — flagged for review and may be discarded.

This protects against both consumer harm (bad actors gaming the flag system) and professional harm (defamation through unverified flags).

### 8.12 Registry Integrity and Fraud Prevention

Both registration routes are subject to integrity checks. For partner-route professionals, the partner's data quality and freshness obligations apply. For individual self-registered professionals, basic credibility checks run at registration: email verification, name and firm findable in at least one publicly accessible source, flagging of profiles where multiple data points appear very recently created or where review histories show anomalous patterns. Flagged profiles are queued for review before becoming active in matching.

### 8.13 Consumer AI System Integration

The core role of ATAH in AI system integration is to support the transition from AI-only interaction to human professional engagement when the need for human expertise becomes clear.

- MCP tool calls — eight core tools: `find_professional`, `initiate_introduction`, `check_introduction_status`, `submit_stage2_data`, `submit_stage3_data`, `cancel_introduction`, `revoke_consent`, `get_consent_requirements`, `report_introduction_outcome`
- REST API — full API for non-MCP clients
- Webhooks — status updates pushed to consumer AI agents on state changes (Phase 2, once major AI platforms support persistent callback endpoints)
- Cross-platform status check — authenticated AI platforms with matching consumer reference can read introduction status

### 8.14 Profile Maintenance

- Web portal — magic link, SMS one-time code, or OIDC. No password.
- AI assistant connection via MCP — read-only, scoped connection
- Trusted partner contributions — partners push updates automatically when professional data changes in their systems
- Folder sync — a future phase feature, not available at MVP

## 9. Privacy and Data Governance

ATAH handles consumer personal data as a transient conduit, never a repository. The system distinguishes clearly between data categories:

- **Professional identity data** — name, credentials, contact preferences, and profile information. Core registry data, maintained as long as the professional is registered.
- **Trust and verification metadata** — partner-contributed data points, enhanced verification records, review platform data, verification status records, and validator provenance. Retained and tagged with source at all times.
- **Consent receipt metadata** — hash and metadata only. No PII. Retained per receipt expiry plus 90 days.
- **Introduction lifecycle data** — state transitions logged with pseudonymous identifiers. No PII.
- **Consumer personal data** — strictly transient. Held only in the encrypted vault during active handoffs. Crypto-erased on resolution. Never written to the main registry database. Never transmitted through SMS or email.
- **Audit log** — pseudonymous identifiers only. Tamper-evident hash-chained. Retained 7 years.
- **Anonymised post-introduction data** — non-PII outcome data only. Retained two years then deleted.

### Notification policy

SMS and email carry no consumer personal data. They carry only non-sensitive notification metadata and authenticated retrieval references. This is a Charter Part One commitment, not an operational policy.

### Authenticated retrieval

Personal data is delivered to professionals through authenticated retrieval portals, not through SMS or email. Tiered authentication per category sensitivity (`tier_1`, `tier_2`, `tier_3`).

### Right to erasure

Because ATAH holds consumer personal data only transiently, the standard right to erasure is satisfied by the protocol's design. Audit records contain no PII. Detailed DSAR machinery for federation scenarios is deferred to v0.9.

Every introduction requires fresh explicit consent captured as a structured consent receipt. ATAH never re-uses personal data from a previous introduction.

### Regulatory considerations by category

- **Lawyers** — attorney-client privilege. Stage 1 contains no personal data. Stage 2 minimum for category-appropriate check (typically conflict check). Full matter content never flows through ATAH.
- **Property and casualty insurance agents** — licence and appointment verification enforced at matching layer. NIPR provides automated verification across all 50 states. Stage 2 is typically a capacity confirmation or scope alignment. NIPR verifies licensing and appointment data where available; category fit (lines of authority, carrier appointments, farm/commercial/personal focus) still depends on declared and partner-verified specialisms, product lines, and engagement scope.
- **Tax planners and accountants** — Stage 2 is typically a scope confirmation. Verification depends on the relevant accounting body (AICPA, equivalents) and any regulatory licensing where applicable.
- **Financial advisors** — FINRA registration, RIA status, and disclosed disciplinary history in profile where available.
- **Healthcare professionals** — registration open from day one. Not surfaced in matching until HIPAA-compliant variant is implemented and the category compliance annex is complete.
- **All other categories** — compliance annex specified before each new category opens for matching.
## 10. The AI Platform Flywheel

Each additional AI platform integration increases the practical usefulness of ATAH because the same structured trust layer can support repeated high-stakes handoff moments across different user environments.

ATAH creates a structural incentive for AI platforms to support and promote the protocol. Every professional represented in an ATAH-conformant registry has an incentive to connect their AI assistant. Introductions arrive in their AI environment. Referral relationships are managed there. Profile stays current automatically.

This creates a shared route to AI platforms at no per-platform integration cost — across every professional category, credentialled and established alike. The more platforms support ATAH, the richer the registry. The richer the registry, the better the consumer matches. The better the matches, the more AI systems integrate with conforming registries.

The proposition to AI platforms is direct: supporting ATAH addresses the broken handoff problem that every AI system currently hits. It turns a limitation into a capability.

## 11. The Commercial Model

The protocol is open and royalty-free to implement. The MCP endpoint is free to query (no per-query or licensing fee — querying still requires authentication for protocol integrity). The commercial model is built around the trusted partner network.

### Protocol versus reference registry / operational layer

The commercial model belongs to the **reference registry and associated governance/operations layer**, not the protocol itself. The open protocol is royalty-free to implement. Commercial participation funds the work of running the reference registry, governing the partner and verifier networks, maintaining integrations, and supporting the introduction lifecycle. Commercial participation must not be confused with protocol conformance or matching weight.

A future conforming registry operator may set its own operational fees, subject to the conformance principles (Section 14 of the spec) — including no commercial weighting, provenance visibility, and category compliance gating.

### Failure modes and accountability

ATAH is best-effort within the published rubric and verification freshness windows. AI platforms and consumers retain responsibility for selection and use. ATAH commits to specific data freshness windows, dispute resolution timelines, and concern-routing protocols, but does not warrant the suitability, competence, or behaviour of any individual professional.

Where matching surfaces a professional whose status has changed (recent suspension, disciplinary action) faster than partner data freshness can reflect, ATAH's commitment is the published verification-deferred handling and the meaningful conflict suppression mechanism — not a guarantee of real-time accuracy.

### Capture resistance

The protocol is designed against capture by large early partners. Specific structural protections:

- **No exclusivity** in any category — multiple regulatory bodies, professional bodies, verifiers, and review platforms may operate in any category
- **Public partner fee schedules** with cost-recovery rationale
- **Public verifier acceptance criteria** and scoring rubric
- **Public review platform acceptance criteria**, including anti-gaming controls
- **Equal-terms participation** for any entity in which the founder has an interest, with annual conflict disclosure and recusal rules
- **The individual self-registration route** exists permanently and receives identical matching treatment to partner-route listings
- **Hardship waivers** for individuals; **fee waivers** for public-interest partners
- **No commercial weighting** in matching — ever, in any form
- **Royalty-free protocol implementation** — no licence fee or permission requirement applies to implementing ATAH outside the reference registry

### Revenue streams (reference registry)

Trusted partner integrations are the primary revenue stream for the reference registry. Designed to be cost-recovery for integration and maintenance work, not pay-for-ranking. Every professional body, regulator, review platform, and data provider that connects pays for integration access. With hundreds of professional bodies in the US alone, and the same again internationally, the addressable partner universe is large.

Independent verifier governance fees (non-refundable assessment fee + annual audit fee, both tiered by verifier scope) cover the governance and audit work that keeps the verifier model credible. ATAH does not receive a share of the verification fees the verifier charges its commissioning parties.

Three explicit commercial commitments:

- ATAH is designed to separate payment from ranking. Payment funds data integration, verifier governance, and partner participation. It does not purchase ranking, recommendation status, or preferential matching treatment.
- Where partner-provided data influences trust presentation, the source and verification basis remain transparent.
- Where paid services generate verification evidence, that evidence is scored under the same public rubric available to all approved sources.

The proposition to professional bodies is direct: contribute data on your members, give them structured machine-readable presence at the point AI systems need validated human sources, and earn from providing it.

Beyond the protocol: a referral analytics platform, association white-label programmes, managed onboarding, and premium analytics may exist as adjacent commercial products. None of these touch the protocol's neutrality, and ATAH governance audits the boundary between the open protocol and any adjacent commercial activity.

## 12. Governance and Corporate Structure

ATAH is founded by Grahame Cohen. The protocol operates under [CHARTER.md](CHARTER.md). The Charter is intended to operate as a public governance covenant, with the founder undertaking to administer ATAH consistently with its commitments until governance transfers to an independent entity.

The choice of legal structure for the protocol's interim and long-term governance is under active consideration with legal advice. The Charter applies to all versions of the protocol regardless of corporate structure.

### Governance transition covenant

This is set out in detail in the Charter and summarised here:

- An interim advisory group will be convened when suitable members are identified and available to serve.
- Governance transitions to an independent entity when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated. The trigger framework reflects the realistic adoption timelines of cross-industry standards work — established standards bodies (W3C, OASIS, OCDS, IEEE) operate on implementation evidence rather than calendar deadlines, and ATAH follows the same pattern.
- Transition progress is reported publicly when there is meaningful progress to share. The report covers progress toward the trigger conditions, the state of partner integrations, and any factors affecting the path to independent governance.

### Interim advisory structure

Before the interim governance body is formed, the founder will convene an interim advisory group. The interim advisory group must be consulted before any of the following decisions:

- Approval of the first independent verifier
- Approval of the first review platform
- Approval of the first commercial partner agreement
- Approval of the first category compliance annex
- Any change to the Charter

### Key governance commitments

Set out in the CHARTER and reflected throughout this PRD:

- The protocol specification is open and licensed under Apache 2.0 permanently.
- Matching logic carries no commercial weighting at any point.
- Trusted partner data is always tagged with its source and validator class. Provenance is never obscured.
- The MCP endpoint is free for any AI platform to query.
- **Privacy floor (core commitment):** ATAH must not operate as a central repository of consumer personal data. Notification channels carry no PII.
- **Compliance gating floor (core commitment):** No professional category may be surfaced in matching without an approved compliance annex.
- The founder retains a permanent advisory seat with no veto rights, no preferential commercial advantage, conflict-of-interest disclosure, recusal from decisions affecting founder-affiliated entities, and a public register of related-party transactions.
- Reasonable compensation for the founder's time spent on protocol work is permitted, set transparently and reviewed by the governance body once established. The founder does not receive a personal share of partner integration fees, professional registration fees, enhanced verification fees, or any commercial benefit conditional on the registry's growth.
- After transition to independent governance, the founder advisory seat carries consultation rights but not approval rights for core amendments. No nominee may hold approval rights over core amendments.

The full Charter is the authoritative source for governance commitments. Where this PRD and the CHARTER differ, the CHARTER governs.

## 13. Open Source and Licensing

Protocol specification published under Apache 2.0 licence. Anyone can implement, query, or build on ATAH. No permission required. The reference registry implementation is also open source.

ATAH composes with established standards rather than replacing them: OAuth 2.1 (used for MCP and REST API authentication), OpenID Connect (supported for professional portal authentication), W3C Verifiable Credentials (accepted as a partner data submission format), Decentralized Identifiers (atah_id format compatible with future DID resolution).

## 14. Roadmap

The roadmap below describes the direction of work for the **ATAH-operated reference registry**, not protocol-level conformance requirements. The phases describe sequence — what comes before what — not a delivery schedule. Calendar dates have been removed because the work depends on the people clock: regulator conversations, professional body decisions, partner agreement negotiations, and AI platform partnership cycles all run on multi-month-to-multi-year timelines that no software roadmap can compress. Phase progression reflects what those conversations produce, not what was hoped for.

Conforming registries are not bound to this roadmap; they are bound to the conformance principles in the specification (Section 14). Where protocol-level commitments evolve (e.g. new schema versions, new bindings), they are tracked in the spec CHANGELOG and the top-level [ROADMAP.md](ROADMAP.md).

### Phase 1 — Publication and community review

Protocol specification v0.8.1 published on GitHub with all schemas, OpenAPI contract, MCP tool definitions, and governance machinery (CHARTER, GOVERNANCE, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, CONFORMANCE, ROADMAP, CHANGELOG, CHANGES-FROM-V0_7-TO-V0_8). Repository accepts issues, discussions, and contributions. Community review begins — feedback from standards reviewers, AI platform technical staff, professional bodies, and prospective trusted partners may be incorporated in subsequent patch versions as time and resources allow. Publication announcement, press cycle, and outreach to associations and AI platforms begin. An interim advisory group will be convened when suitable members are identified and available to serve.

The reference registry is in design; build commences as resources allow. The reference registry's eventual operational scope (which categories, which jurisdictions, which verification routes are live) depends on partner conversations advancing and on the resources available to the build. Phase 1 has no committed end date — it runs until the work of Phase 2 begins to overlap with it, at which point Phase 1 activity (community review, governance maturation) continues alongside reference registry build and first partner conversations.

### Phase 2 — Reference registry build and first partner conversations

Reference registry implementation may begin if resources allow. Alternatively, an external implementer may build a conforming registry against the spec — the protocol is open and any conforming implementation moves the protocol forward. First partner conversations may advance with regulators, professional bodies, and review platforms across the categories where the AI-discoverability question is sharpest. AI assistant MCP connection for profile maintenance, first W3C Verifiable Credential submission paths, first independent verifier conversations, healthcare compliance annex work with specialist legal input, conduct handling framework, webhook push delivery, and MCP push notifications all sit in this phase as design and development work that proceeds as conversations and resources allow.

This phase is dependent on partner conversations advancing and on resources for reference registry build (whether by ATAH directly or by an external implementer). Decision cycles in cross-industry standards work are inherently slower than software-industry timelines; phase progression reflects what those conversations produce, not a calendar.

### Phase 3 — Reference registry operational and first integrations

Reference registry deployed and operational. First trusted partner integrations live with verified data flow. First AI platform integration conversations advance toward production integration. First approved independent verifier operational; first enhanced verification records visible in match responses. First review platform integrated with anti-gaming attestations confirmed. A2A protocol binding work begins. Additional category verification pathways come online as partner conversations conclude.

The transition to independent governance is not a Phase 3 deliverable; it is triggered when two trusted partner integrations are live and at least one external conforming implementation has been demonstrated, per the Charter. The transition may occur during, before, or after Phase 3 depending on when the trigger conditions are met.

### Phase 4 — Federation and broader adoption

Federation mechanics specified in v0.9 / v1.0: cross-registry trust, atah_id resolution across registries, handoff routing between registries, federated governance. Broader trusted partner network. Additional jurisdictions opened as partner agreements and compliance annexes mature. Native support for digital twin identity and data sharing as personal AI agent standards emerge. Sybil/referral-ring detection beyond v0.8 baseline.

### Phase 5 — Maturity

Trusted partner network across multiple jurisdictions and a broad range of professional categories. Full healthcare category with confirmed compliance variant. Independent governance transitioned per Charter triggers. Conformance test suite published. Multilingual support. The protocol matures from release-candidate through stable releases as adoption broadens and the independent governance body forms its operating cadence.

## 15. Success Metrics

Success metrics are split into three layers — protocol, reference registry, and ecosystem — to keep clear which kind of success is being measured. The protocol can succeed even where the reference registry has not yet hit operational scale, and vice versa.

### Protocol success (Phase 1)

- Protocol specification v0.8 published on GitHub with all governance and supporting documents
- Schemas validate against JSON Schema 2020-12
- OpenAPI definition validates and lints clean
- MCP tool definitions usable by an MCP-capable client without bespoke adaptation
- Conformance criteria (Section 14 of the spec) reviewed externally and confirmed implementable without relying on unpublished reference-registry assumptions
- At least one external implementation prototype or proof of concept built against the spec
- Protocol specification forked or downloaded by at least ten independent organisations

### Reference registry success (Phase 1)

- Interim advisory group convened (no fixed deadline; convened when suitable members are identified and available to serve, per Charter)
- Reference registry MCP endpoint live and queryable by at least two different AI platforms
- All three introduction types demonstrated successfully end to end through the reference registry
- End-to-end handoff demonstrated in at least one high-stakes category
- Both registration routes (partner and individual) live and tested in the reference registry, including roll-up logic with pre-merge notification
- Stage 2 pre-handoff check working for at least two categories with different check types (e.g. lawyer conflict check, tax planner scope confirmation)
- Tiered notification + authenticated retrieval confirmed working across all three auth tiers; no PII transmitted through notification channels
- Consent receipt generation, hash storage, and revocation working end-to-end
- Tiered handoff access control validated (cross-platform read; write-token write)
- At least two trusted partner integrations live and contributing data to the reference registry
- Verification routes live for at least one regulated category in at least one jurisdiction, with appropriate labelling for jurisdictions where verification is not yet in place
- Privacy and deletion controls (transient vault, crypto-erasure, deletion scope) tested under load
- Reference registry conformance statement published

### Ecosystem success (Phase 1)

- Conversations initiated with at least five professional associations or trusted partner organisations
- At least one professional body in active conversation about issuing W3C Verifiable Credentials to its members
- Legal entity incorporated or founding charter signed
- First conversations begun with potential successor independent governance structures (foundation, CIC, public benefit corporation, or equivalent)

### Longer Term

- ATAH referenced by at least one major AI platform as a recommended professional handoff standard
- At least one professional association operating as both white-label partner and trusted partner
- Protocol adopted in at least one jurisdiction outside the United States
- Independent governance body established and operational
- Ten or more professional categories represented in the registry, spanning both credentialled and established practitioner fields
- Trusted partner network covering the majority of verified professionals in launch categories
- At least one approved independent verifier operational with active enhanced verification records
- At least one review platform integrated with anti-gaming attestations confirmed and review-derived signals contributing to matching within category caps
- W3C Verifiable Credential submissions in production from at least one partner
- At least one external conforming implementation built against the spec (a transition trigger)

## 16. Open Questions

- Which professional body should be the first trusted partner for an established practitioner category? PRSA or CIPR would be natural first movers for communications. The Big "I" or PIA would be natural first movers for insurance agents.
- How should ordering be disclosed to consuming AI systems beyond the `presentation_disclosure` block? What additional UX guidance should accompany the disclosure?
- What trust signals are required before a professional can be surfaced in a high-stakes category?
- Which validator classes are mandatory versus optional by category?
- What disclosures should an AI system make when presenting ATAH results to users beyond the `presentation_disclosure` block?
- Under what conditions should professional associations, regulators, or review platforms be treated as authoritative for different fields?
- How should the entity be funded beyond trusted partner integration fees, verifier fees, and the initial founder contribution?
- Composition of the interim advisory group: lawyer, property and casualty insurance professional, AI platform representative, regulator, consumer advocate, established practitioner representative, trusted partner representative? The founder seat structure is set out in the CHARTER.
- What is the right partner SLA threshold for different partner types? Regulatory databases may update daily. Review platforms near real time. Professional bodies monthly.
- Healthcare compliance annex: when should specialist legal advice be commissioned? Recommended before end of Phase 1.
- At what threshold of consumer-reported concerns should ATAH proactively notify a professional body? To be defined in the conduct handling framework in Phase 2.
- Per-category matching weight profiles populated with researched values: which categories should be researched first? Regulated tier (lawyer, physician, financial advisor, insurance agent) is the priority.
- Federation: which jurisdictions should be the first regional registries when federation is specified in v0.9 / v1.0? EU is a likely candidate given GDPR-native operation.

## 17. Summary

ATAH is open infrastructure for a problem that is real and currently unsolved across every professional field — regulated and established alike.

AI platforms are increasingly the interface for everything. They can handle products, transactions, and services. The one thing they cannot reliably do is complete the journey from AI interaction to the right human professional in a structured, trusted, validated way. ATAH addresses that gap. For AI platforms, it turns a broken handoff into a capability. For professionals, it provides a structural way to be discoverable to AI systems without each one solving Answer Engine Optimisation alone. For professional bodies, it is a new membership benefit and a new revenue stream built on data they already hold.

The trusted partner model is what makes the registry credible and the operational model coherent. Sustainability rests on that network — partner integrations, verifier governance fees, and adjacent operational services — and the network grows every time a new category opens or a new jurisdiction is added.

The protocol composes with existing identity and credential standards rather than competing with them. Its specific contribution is the layer above: professional categorisation, the staged introduction lifecycle, the matching engine, the trusted-partner trust model, and the commercial-neutrality and provenance-visibility commitments.

The protocol is designed to be implementable. The MCP endpoint is free to query. The governance commitments are set out in the CHARTER, with a trigger-based transition framework rather than a calendar deadline. The trusted partner model is commercially coherent. The privacy architecture is structural, not policy.

This is a v0.8 release candidate intended for technical review and reference implementation work. Hardening continues through community review, partner integration feedback, and reference implementation experience.
