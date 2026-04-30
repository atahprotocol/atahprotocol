# 5. Transient encrypted vault — no PII through notification channels

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

Stages 2 and 3 of the introduction lifecycle move actual personal data:
Stage 2 carries pre-handoff check data (consumer name, optionally an
opposing party's name, scope summary, etc.); Stage 3 releases consumer
contact details to the selected professional.

The naïve implementation is an SMS or email to the professional with the
data inline: "Your new client is Jordan Public, contact +1-512-555-0123,
matter is contract review for Acme."

That implementation moves consumer PII through SMS/email providers,
through inboxes, through device backups, through carrier logs. Each of
those is a residency the consumer never consented to and the professional
cannot honour deletion against.

## Decision

Personal data lands only in a **transient encrypted vault** at the
registry, with per-record encryption keys, and is retrieved by the
professional through **authenticated retrieval**.

- Notifications carry only metadata and a retrieval reference (no PII).
- The professional authenticates and pulls the record from the vault.
- The vault entry has a defined retention window per category sensitivity
  (`tier_1`, `tier_2`, `tier_3`).
- Deletion is by **crypto-erasure**: destroy the per-record DEK and the
  ciphertext is unreadable. This satisfies right-to-erasure obligations
  even when underlying storage hasn't been physically rewritten.
- Cancellation, consent revocation, or expiry trigger crypto-erasure.

## Alternatives considered

- **Notifications include the matter summary.** Rejected. Even
  "summary" is PII — matter summaries routinely include consumer names,
  case context, and party identifiers. Once it is in an email or SMS
  provider, the consumer has lost control.
- **Encrypt the notification payload to the professional's public key
  and let the professional decrypt locally.** Considered. Operationally
  costly: professionals would need to manage keys, and key recovery
  becomes a vector. The vault model gives the same property
  (notification content reveals nothing) without imposing key
  management on professionals at v0.8 scale.

## Consequences

- `submit_stage2_data` and `submit_stage3_data` MCP tools, and the
  corresponding REST endpoints, store data in the vault and return a
  `vault_reference` opaque to the caller.
- The professional retrieves through an authenticated endpoint — the
  spec defines `tier_1`, `tier_2`, `tier_3` authentication strength
  thresholds per category, with healthcare and legal at the higher
  tiers.
- Notifications are SMS/email/push but contain only "you have a new
  introduction; sign in to retrieve" — never the data itself.
- Vault deletion is auditable: the DEK destruction is logged with a
  tamper-evident chain.

## References

- Spec §11.6 (Transient encrypted vault and authenticated retrieval),
  §11.2 (Deletion scope)
- MCP tool descriptions: `submit_stage2_data`, `submit_stage3_data`
- OpenAPI: `vault_reference` response field on stage-2 and stage-3
  endpoints
