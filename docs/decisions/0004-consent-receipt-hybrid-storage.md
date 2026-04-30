# 4. Consent receipt hybrid storage — hash at ATAH, full receipt at the platform

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

ATAH replaced boolean `consumer_consent: true` with structured consent
receipts (W3C Consent Receipt / Kantara CR 1.1 patterns). A receipt
records who asserted consent, what scope, what data categories, what text
the consumer was shown, and when it was captured.

The question: where does the receipt live?

If ATAH stores the full receipt, ATAH becomes a personal-data store for
every consenter who ever interacts with any platform — which directly
contradicts the privacy floor.

If only the platform stores the receipt, there is no tamper-evident
anchor: a platform could rewrite history without detection, and disputes
become a he-said-she-said.

## Decision

We adopted a hybrid:

- **The asserting platform retains the full receipt.** The platform
  collected the consumer's consent; the platform's own data-protection
  obligations cover the receipt.
- **ATAH stores only the receipt hash plus reference metadata** (receipt
  id, asserter platform id, scope, captured/expires timestamps,
  revocation status, optional handoff/professional bindings). The stored
  form has its own schema (`consent-receipt-stored.schema.json`),
  structurally distinct from the full receipt
  (`consent-receipt.schema.json`).

At dispute time, the platform produces the full receipt; ATAH verifies
the hash matches the value stored at consent time, confirming the
receipt has not been altered.

## Alternatives considered

- **ATAH stores the full receipt.** Rejected. Makes ATAH a personal-data
  store for every consumer, contradicts Charter Part One's privacy
  commitment, and creates a single point of breach for every consenter
  in the ecosystem.
- **Platform stores everything; ATAH stores nothing.** Rejected. There
  is no tamper-evident anchor. A malicious or compromised platform
  could rewrite a receipt to claim broader scope or different data
  categories than the consumer agreed to, and the consumer has no
  ability to dispute what was actually captured.
- **Verifiable Credentials as the substrate.** Considered. v0.8 accepts
  VCs alongside the hash-and-metadata model (`vc-verified` is a
  verification status), but does not require them. The hybrid model
  works without requiring the entire ecosystem to operate VC issuance
  on day one.

## Consequences

- The privacy floor is structurally enforceable: a Stage 8 build-time
  test reads `consent-receipt-stored.schema.json` and rejects the
  literal field names that the privacy floor excludes.
- Dispute resolution requires the asserting platform to produce the
  receipt. If they cannot, the receipt is treated as not asserted.
- The pattern is analogous to Sigstore / RFC 6962 transparency logs:
  verifiable presence-of-record, contents-on-demand.

## References

- Spec §4.10 (Consent Receipt Object), §11 (Privacy and Data Lifecycle)
- `consent-receipt.schema.json`, `consent-receipt-stored.schema.json`
- Charter Part One: privacy floor
