# 6. Opaque ULID-style identifiers with typed prefixes

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

Identifiers across ATAH refer to professionals, partners, verifiers,
review platforms, consent receipts, handoffs, enhanced verifications,
queries, attestations, dispute records, concern flags, vault references,
referrals, and proposals. There is a spectrum from "fully structured"
(country code + profession code + sequence) to "fully opaque"
(uncorrelated random tokens).

A structured id like `lawyer-tx-12345` is human-readable and
self-describing. It is also a bug magnet: it breaks when the professional
changes jurisdiction, it leaks profession even when that is private, it
implies a sequence number that becomes meaningful and exploitable, and it
constrains the federation model.

## Decision

Identifiers are **opaque ULID-style** with **typed prefixes**:

- Pattern: `^<prefix>-<26-char-base32-suffix>$` (or compatible).
- Prefixes: `atah-` (professionals, reference registry), `tp-` (trusted
  partners), `iv-` (independent verifiers), `rp-` (review platforms),
  `cr-` (consent receipts), `hf-` (handoffs), `ev-` (enhanced
  verifications), `q-` (queries), `att-` (attestations), `dispute-`
  (dispute records), `flag-` (concern flags), `vault-` (transient vault
  references), `ref-` (Type 3 referrals), `proposal-` (Type 2
  proposals).

The suffix encodes nothing about the entity. Country, profession,
category, status, and sequence are stored as attributes on the record,
not as substrings of the id. Partners may be exempted from ULID-style
suffixes for human-meaningful slug ids (e.g. `tp-statebar-tx-001`); both
are accepted by the identifier pattern.

## Alternatives considered

- **Structured ids encoding country/profession/sequence.** Rejected.
  Structured ids break under any attribute change (jurisdictional moves,
  category reclassifications, business renaming) and disclose the
  attributes wherever the id appears. They also create a sequencing
  oracle.
- **UUIDv4 instead of ULID.** Considered. ULIDs are sortable by
  generation time, which is operationally useful for paging,
  observability, and audit log review. The privacy/security properties
  are equivalent.
- **Prefix-less opaque ids.** Rejected. The prefix is essential for
  audit log triage, log filtering, and human reading. The cost of a
  prefix is small; the operational benefit is large.

## Consequences

- The identifier regex pattern allows uppercase ULID suffixes and
  lowercase slug suffixes (`^[a-z]{2,8}-[A-Za-z0-9-]{6,64}$`),
  accommodating both ULID-generated entity ids and slug-style partner
  ids.
- Federation is unblocked: the namespace is the prefix; uniqueness is
  guaranteed within a registry by ULID generation; a future
  cross-registry resolver can disambiguate without re-encoding ids.
- Logs and error messages that include ids do not leak the entity's
  attributes.
- Reverse-engineering an id back to its entity requires registry
  access; an attacker scraping logs cannot derive jurisdiction or
  profession from the id alone.

## References

- Spec §4.1 (Schema Conventions), §4.6 (Professional Identity examples)
- Identifier pattern in every schema's `pattern` constraint
