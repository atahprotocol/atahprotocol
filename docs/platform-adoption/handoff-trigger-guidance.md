# Handoff trigger guidance

**Status:** platform-side guidance, not normative protocol text. The protocol defines what a handoff is and how it works; this document describes when an AI platform should invoke a handoff. Platforms remain responsible for their own user-facing decision logic.

**Audience:** AI platform integration teams designing the trigger logic that decides when to surface ATAH-mediated handoff to a verified human professional.

**Companion to:** [PRD §9 Privacy and Data Governance — Regulatory considerations by category](../../PRD-v0_8.md), [spec §6 Introduction Lifecycle](../../spec/v0.8/full-spec.md), and [`profession-categories.json`](../../spec/v0.8/schemas/profession-categories.json).

---

## Purpose

ATAH supplies the trust, provenance, consent, and lifecycle layer for handoff. It does not decide *whether* a given user message warrants handoff — that decision sits with the AI platform. This document gathers the safety-integration considerations a platform should weigh, by category, when designing its own trigger logic.

The guidance is descriptive. It does not bind platforms to specific trigger conditions; platforms apply their own product judgement, their own legal posture, and their own user-experience standards. ATAH's role is to make the handoff provenance-visible and consent-respecting once the platform decides to invoke it.

## General principles

Three principles cut across categories:

**Defer rather than guess.** When the user's situation is plausibly within a regulated profession's remit — legal liability, financial advice, medical decisions, insurance coverage — and the AI is operating at the edge of its training, deferring to a verified human professional is the safer move. ATAH gives the AI a structured way to do that, rather than leaving the user with a disclaimer and the responsibility to find one themselves.

**Don't trigger on category alone.** "User mentioned a contract" or "user mentioned tax" is not by itself a trigger condition. People discuss legal and tax concepts informationally all the time. The trigger is when the AI assesses that the user genuinely needs human professional input — meaning the question carries consequences (a decision, a transaction, a deadline, a regulated action) that the AI cannot responsibly answer.

**Continue helping where it's safe to.** ATAH is for the moment of handoff, not a wholesale replacement for AI assistance. Many adjacent questions remain in scope for the AI: explaining what a clause means, summarising a regulator's published guidance, walking through how an insurance claim process generally works, helping a user prepare a list of questions for a professional. The handoff invocation is for the specific moment the AI determines the user needs the human.

## Category-specific guidance

The following category-specific guidance describes signals indicating handoff is appropriate, signals indicating handoff is premature, the shape of an effective Stage 1 query, and special considerations — for each launch category covered by ATAH at v0.8.2.

The categories below are the launch-supported set with the sharpest AI-discoverability questions. Refer to `profession-categories.json` for the canonical category metadata (matching weight profile, review weight cap, default Stage 2 check, auth tier).

### Legal services

**When handoff is appropriate.**
- The user describes a specific legal liability exposure (contract dispute, tort claim, regulatory action) where the AI's general explanation cannot substitute for jurisdiction-specific legal advice.
- The user is approaching a deadline tied to a legal process (statute of limitations, response window in litigation, deadlines on a contractual cure period).
- The user describes a transaction with material legal risk (acquisition, large lease, employment dispute, divorce, immigration filing).
- The user has expressed an intent to act on their own legal interpretation in a way the AI assesses carries real risk.

**When handoff is premature.**
- The user is asking informationally about how a legal concept generally works ("what is consideration in contract law?").
- The user is researching for a class, a journalistic piece, or general curiosity.
- The user is preparing for a meeting with a lawyer they already have engaged.
- The matter is plausibly small-claims-court territory and the user has not asked to be connected to a lawyer.

**Stage 1 query shape.** Category set to the appropriate legal sub-category (corporate, family, property, immigration, etc.). `matter_type` set to a structured descriptor of the matter without identifying detail. `location` set to the user's jurisdiction (or, where cross-jurisdictional, the relevant licensing jurisdiction — see PRD §8.5 cross-jurisdiction default). `urgency` set per the user's stated timeframe. No personal data at this stage.

**Special considerations.** Legal services trigger jurisdiction-specific advertising, referral, and lead-generation rules; ATAH's matching engine does not permit per-referral fees, outcome-contingent fees, or ranking based on payment (per PRD §9 lawyers entry). Stage 2 conflict screening is preliminary and does not replace the lawyer's own conflict-checking obligations.

### Property and casualty insurance

**When handoff is appropriate.**
- The user describes a coverage decision tied to a specific risk (homeowners coverage for a new home, business interruption coverage, professional indemnity, motor coverage with non-standard exposures).
- The user is filing a claim with material complexity (subrogation, disputed coverage, denial of coverage, multi-line claim).
- The user is comparing policies in a category with non-trivial exclusions (flood, professional liability, cyber).
- The user has expressed dissatisfaction with a current carrier and is considering switching.

**When handoff is premature.**
- The user is asking informationally about how a coverage type works.
- The user is working through a routine claim within their existing agent's purview.
- The user has not described a specific coverage decision.

**Stage 1 query shape.** Category set to property-casualty insurance. `matter_type` set to descriptor (homeowners-claim, commercial-coverage, etc.). `location` set to the user's licensing jurisdiction. Lines of authority and product-line specialism captured in filters where the user has indicated a specific coverage area.

**Special considerations.** US licensing data sources (NIPR-style) cover licensing and appointments where partner agreements are in place; category fit (lines of authority, carrier appointments, farm/commercial/personal focus) depends on declared and partner-verified specialisms.

### Financial advice

**When handoff is appropriate.**
- The user describes a specific financial decision with regulatory implications (retirement-plan rollover, large investment decision, tax-advantaged account strategy, regulated investment product).
- The user has assets or income at a level where a registered investment adviser or financial planner adds meaningful value over self-directed information.
- The user is asking about a regulated product (annuities, structured products, alternative investments) where unsuitability risk is real.
- The user has expressed a fiduciary-relevant question (is this advice in my best interest? am I being properly served by my current adviser?).

**When handoff is premature.**
- The user is asking informational questions about financial concepts (compound interest, asset allocation, tax brackets).
- The user is comparing routine consumer financial products (savings accounts, credit cards) where regulated advice is overkill.
- The user is working through self-directed investing within their existing knowledge.

**Stage 1 query shape.** Category set to financial advice (with sub-category where the protocol distinguishes by product line or advice scope). `matter_type` set to descriptor. `location` set to the user's regulatory jurisdiction. Disclosed disciplinary history surfaced from authoritative US registers (FINRA-style sources are representative examples), subject to redistribution terms and partner agreements (per PRD §9 financial advisors entry).

### Healthcare (deferred)

**Status at v0.8.2.** Healthcare is registration-only at v0.8.2 — open for professional registration but not surfaced in matching pending HIPAA-compliant variant implementation and category compliance annex completion (per PRD §9 healthcare entry).

**Platform-side guidance for v0.8.2.** AI platforms should not rely on ATAH for healthcare professional handoff at v0.8.2. Where a user requires healthcare professional input, platforms should continue to use whatever fallback they currently use (generic referral text, integration with a category-specific service, or escalation to direct user search). The AI may still help the user prepare for a healthcare conversation (questions to ask, information to bring, how to describe symptoms).

**When the healthcare annex is published.** Once the HIPAA-compliant variant lands and the category compliance annex is approved, this section will be expanded with category-specific trigger guidance covering urgent care vs. routine, mental health considerations, pediatric and minor-consent edge cases, and special handling for substance use, reproductive health, and other categories with elevated privacy considerations.

### Categories without a specific section

Other launch-supported categories (PR/communications, coaching, project management, accountancy, architecture, engineering) follow the general principles. The category metadata in `profession-categories.json` carries the matching weight profile, review weight cap, default Stage 2 check, and auth tier. Platform-side trigger logic for these categories is generally simpler than for the regulated four above — the consequences are real but lower-stakes than legal/insurance/financial/medical decisions.

## Cross-category considerations

**Urgency.** The `urgency` field in the Stage 1 query (none, routine, urgent, immediate) shapes which professionals are returned. Platforms designing the trigger should treat user-described urgency as the primary input rather than guessing. If the user has not stated urgency, ask before invoking the handoff.

**Jurisdiction.** ATAH does not return cross-jurisdiction matches by default for regulated categories (per PRD §8.5 cross-jurisdiction default). Cross-jurisdiction results require explicit query parameters indicating the consumer's intent to engage cross-border, plus a `presentation_disclosure` note that the professional's licensing or authorisation may not cover the consumer's jurisdiction. Platforms should default to the user's jurisdiction and only opt into cross-jurisdiction when the user has stated they want it.

**Capacity.** ATAH assumes the asserting AI platform has determined the consumer has capacity to consent (per PRD §9 consumer capacity assumption). Categories involving services for minors, patients, or vulnerable adults may specify additional platform-side verification requirements in their compliance annex. Platforms are responsible for capacity assessment; ATAH does not independently verify.

**Reading the verification confidence signal.** Match responses for regulated categories carry a `verification_confidence` field separate from the verification-quality score (per spec §4.12, schema implementation in v0.8.2). Platforms should surface confidence — single-source vs multi-source corroboration — to users alongside the verification-quality information. Single-source verification is common at protocol launch and reflects partner-network maturity, not weakness.

## Reference

For the protocol-level definition of the handoff lifecycle, see [spec §6 Introduction Lifecycle](../../spec/v0.8/full-spec.md). For matching engine behaviour and the `presentation_disclosure` requirement, see [spec §9](../../spec/v0.8/full-spec.md) and [PRD §8.5](../../PRD-v0_8.md). For category-specific compliance handling, see [PRD §9](../../PRD-v0_8.md). For the platform liability allocation and what platforms are expected to surface to users, see [PRD §11 Liability allocation and platform/ATAH responsibilities](../../PRD-v0_8.md).
