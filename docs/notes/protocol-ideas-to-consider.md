# Notes — protocol ideas to consider

**Status:** discussion notes only. Not committed to any version. Not part of the spec. Captured here so the ideas don't get lost and so others can pick them up and develop them properly.

**Context.** Came out of a conversation reviewing ATAH v0.8.x against three external protocols — ACP (OpenAI/Stripe, commerce), UCP (Google/Shopify, commerce), and A2A (Google, agent-to-agent). The point of looking outward was to check whether ATAH's current shape — notification + transient vault + authenticated retrieval — is the right modern design, or whether other protocols are doing something cleaner.

The headline conclusion: ATAH's core data-handling model is right for the problem (handing off sensitive consumer information to a verified professional), and should not be replaced with a merchant-style push model. But several ideas from the other protocols are genuinely worth adopting, and one use case from the original conversation isn't covered today.

Five ideas below. None are commitments. Each notes what it is, why it's worth doing, and roughly how much work it implies.

---

## 1. Standing-check endpoint for organisations with curated panels

**The use case.** An organisation already has a curated list of professionals — a corporate panel, an insurer's preferred-firm list, an association's referral network. Before they put one of those professionals forward, they want to check the professional is still in good standing. If the answer is no (disbarred, suspended, retired, contact unverified, disciplinary action), they want to put a different one forward. If they don't have anyone suitable on their panel, they want to fall back to a normal ATAH Discovery query.

**Why it isn't covered today.** ATAH today has two intents — `request_intent: self` (consumer handoff) and `request_intent: on_behalf_of_client` (professional gets a candidate set for their client). Neither lets an organisation say "I already know who I want, just tell me if they're still standing." Without this, an organisation either has to call Discovery (wasteful, and they can't direct the result) or maintain their own check process (which defeats the point of having a verified registry).

**Rough shape.**
- A new endpoint, something like `GET /v1/professionals/{atah_id}/standing` or `POST /v1/standing-check` for batch lookups.
- Returns: `matching_status`, `verification_confidence`, disciplinary state, contact-freshness state, conflict-suppression state. Does **not** return contact details. Does **not** initiate a handoff. Does **not** appear in public Discovery flow.
- Gated behind authentication so it can't be used for scraping or phishing.
- Pairs naturally with a follow-up `request_intent: on_behalf_of_client` Discovery call as the fallback path.

**Why this fits cleanly.** It's exposing standing data the registry already maintains. No new data shapes, no new trust model. Small, additive, and fills a real gap.

**Effort.** Small. One endpoint, one schema, one ADR explaining what the endpoint does and does not return, a note in EXPLAINER, an entry in the conformance matrix.

---

## 2. Capability negotiation (from UCP)

**What UCP does.** Both sides — the merchant (business) and the agent (platform) — publish a profile listing the capabilities they support, each with its own version. When they talk, the active session is the *intersection* of what both sides support. Every response carries back the active capability set so both sides agree about what's in play.

**Why it's worth borrowing.** ATAH already has `/.well-known/atah` and `/v1/capabilities`. The bones are there. What's missing is the negotiation semantics. Today, ATAH implicitly assumes a fairly fixed set of features every conforming registry and AI platform supports. If the protocol grows optional features over time — a webhook delivery path for large firms, a standing-check endpoint, future federation, partner-specific extensions — there needs to be a clean way for both sides to declare what they support and agree on what's active without forking the protocol.

**Rough shape.**
- AI platforms publish a profile declaring which ATAH capabilities they consume (Component 1, Component 2 variants, optional extensions).
- Registries publish their existing `.well-known/atah` with the same shape on their side.
- The active session is the intersection. If an AI platform doesn't support Component 2 `full_lifecycle`, the registry doesn't offer it.
- Every match/handoff response carries an `atah` block declaring the active capability set, the protocol version, and the active flow variant. (Some of this is already present in `presentation_disclosure` and `ordering_policy`, but not consistently structured.)

**Why this fits cleanly.** It doesn't change what ATAH does. It changes how ATAH grows. Without it, every optional feature has to be a guess-and-check or a hardcoded assumption. With it, optional features compose cleanly.

**Effort.** Medium. ADR on the negotiation model, schema for the platform-side profile, update to `.well-known/atah`, and a normative section in the spec on how active capabilities are computed and represented in responses. Doesn't break anything that exists today.

---

## 3. Reverse-domain namespaced extensions (from UCP)

**What UCP does.** Extensions to the protocol are named using reverse-DNS notation (e.g. `com.loyaltyprovider.points`). Any organisation can extend the protocol within its own namespace by publishing a schema under a URI in that namespace. The protocol validates that the spec URI origin matches the namespace authority. No central approval committee for every extension.

**Why it's worth borrowing.** ATAH will accumulate extension pressure over time from three directions:
- **Partners** wanting to contribute data shapes the generic `verification_data_point` doesn't model well — jurisdiction-specific status codes, specialist accreditations with sub-types, CPD records.
- **Category annexes** wanting to declare category-specific Stage 2 check types or category-specific bands without forcing a core spec change.
- **Firms and platforms** wanting to attach internal routing or metadata to handoffs that only their own systems consume.

Without a namespacing mechanism, every one of these is either rejected (bad for adoption) or absorbed into the core schema (bloats the protocol and creates maintenance burden).

**Rough shape.**
- Extensions named under reverse-DNS notation tied to the publishing organisation's domain.
- Extensions are opt-in per capability — the receiving side decides whether it understands and implements a given extension.
- Namespace ownership is validated through standard means (DNS verification or signed manifest).
- Extensions that aren't recognised are passed through but not acted on, never silently dropped without a flag.

**Why this fits cleanly.** It's not a free-for-all. There's a governance constraint (namespace ownership validation) and a behaviour constraint (extensions are opt-in, ignored extensions are flagged). It lets ATAH grow at the edges without ever forcing the core to absorb edge cases.

**Effort.** Medium-to-large. Bigger than capability negotiation because it touches governance, not just protocol semantics. Needs an ADR on extension governance, namespace validation rules, what happens to extensions when the publishing organisation goes away, and how extensions interact with the conformance classes. Best done after capability negotiation lands, because extensions naturally live inside the negotiated capability set.

---

## 4. Date-based per-capability versioning (from UCP)

**What UCP does.** Versions are `YYYY-MM-DD` strings. Each capability is versioned independently — the checkout capability and the catalogue capability don't have to move together.

**Why it's worth considering.** ATAH today versions as a whole (`v0.8.1`, `v0.8.2`, `v0.8.3`). A small refinement to one component drags the entire protocol's version forward. If Discovery, Component 2, and the partner data push schema all evolve at different cadences in future, today's model forces them to ship together.

**Why it's not urgent.** Right now ATAH is small enough that bundling versions makes sense. The bookkeeping cost of per-capability versioning would outweigh the flexibility benefit. This is worth filing as a "consider for v1.0" idea rather than something to do soon.

**Effort.** Small if done early. Larger if done late.

---

## 5. Agent Card / capability advertisement (from A2A)

**What A2A does.** Every agent publishes an Agent Card at `/.well-known/agent-card.json` describing its capabilities, the skills it offers, the authentication schemes it supports, and how to talk to it. It's the same idea as UCP's manifest, but built specifically for agents.

**Why it's worth a small look.** ATAH already has `/.well-known/atah`. The A2A Agent Card is more structured — it includes things like declared authentication schemes, skill descriptions with input/output modes, and explicit task lifecycle states. A few of these are worth borrowing into ATAH's discovery manifest:
- Declared authentication schemes per endpoint (rather than implicit from the spec).
- Explicit declaration of which task lifecycle states the implementation supports (the Component 2 state machine has many states; not every implementation needs all of them).
- Stateless vs stateful interaction declaration — A2A's v0.2 added a stateless mode for implementations that don't need session management. ATAH's Tier 1 stateless caller support already does something similar, but isn't declared on the manifest.

**Why this fits cleanly.** It's not a separate idea from capability negotiation — it's the manifest format that capability negotiation reads. If we adopt UCP-style negotiation, A2A's manifest patterns are worth folding in at the same time.

**Effort.** Small. Folds into the capability-negotiation work.

---

## What was looked at and not borrowed

For the record, so reviewers can see the reasoning rather than re-tread it:

- **ACP merchant-push model.** Pushing consumer data into a professional-controlled web endpoint sounds modern but breaks the privacy floor. Most professionals are not Stripe-grade merchants with secure receiving endpoints. ATAH's notification + transient vault + authenticated retrieval is the deliberate answer to this, not a legacy design. Worth borrowing only as an *optional* capability-negotiated delivery path for firms and platforms with real infrastructure (sits alongside the vault, not replacing it).
- **UCP merchant-manifest-as-source-of-truth.** Works for merchants because they're the authority on their own catalogue. Does not work for professionals because the whole point of ATAH is that the professional is not the authority on their own credentials — the regulator or professional body is. Self-published manifests work for operational facts (availability, declared specialisms, response-time commitments), not for credentials.
- **UCP/ACP payment rails.** Not applicable. PRD §9 already disallows per-referral fees and outcome-contingent fees. Bringing payment infrastructure into ATAH would create exactly the regulatory exposure that exclusion was designed to avoid.
- **UCP "full lifecycle to returns and post-purchase."** ATAH stops at outcome reporting plus the §12A contact-freshness mechanism, deliberately. Extending into ongoing relationship state would conflict with the privacy floor and with the v0.8.3 engagement-liveness gap that the EXPLAINER is already honest about.
- **A2A task lifecycle states.** ATAH already has a fuller state machine for Component 2 than A2A's generic lifecycle. Nothing to borrow on the lifecycle itself — A2A's value is in the manifest format, not the state model.

---

## Suggested ordering if any of this is taken forward

1. **Standing-check endpoint** (idea 1) is small, additive, fills a real gap, doesn't depend on anything else. Could be done independently.
2. **Capability negotiation** (idea 2) is the foundation for ideas 3 and 5. Worth doing first if the protocol expects to grow.
3. **A2A manifest patterns** (idea 5) fold into idea 2.
4. **Reverse-domain extensions** (idea 3) sit on top of idea 2. Don't attempt before idea 2 lands.
5. **Per-capability versioning** (idea 4) is a v1.0 consideration. Park for now.

---

## What this document is not

- Not an ADR. ADRs commit to a decision; this is a list of candidate ideas.
- Not a roadmap entry. Roadmap entries commit to a version; nothing here is committed.
- Not a spec change. None of these are normative.

If any of these progress, each one wants its own ADR and roadmap entry written properly before any spec work starts.
