# 3. Parallel `_provenance` map rather than per-field wrappers

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

ATAH's headline trust commitment is per-field provenance: every claim about
a professional should be traceable to a verification source with a
freshness timestamp. The schema needs to expose this without obscuring the
data.

Two structural patterns were considered:

1. **Per-field wrapping.** Every value becomes
   `{ value, verification_status, source, as_of }`. The shape of every
   claim field is `{...}` instead of a primitive.
2. **Parallel `_provenance` map.** The data fields stay primitive; a
   sibling `_provenance` object holds an entry per field path, keyed by
   dot-notation paths.

## Decision

We adopted (2). Schemas and example payloads carry the data as natural
primitives. A separate `_provenance` map at the top level of any schema
containing verifiable claims holds the verification status, source, and
freshness for each field that has been verified.

Field paths use dot notation for nested fields (e.g.
`licence.disciplinary_record`) and bracket notation for arrays (e.g.
`licensed_jurisdictions[0]`). The map is defined as
`provenance-map.schema.json` and referenced by `$ref` from any schema that
allows it.

## Alternatives considered

- **Per-field wrapping.** Rejected. It bloats every payload, hurts
  read-ability for downstream consumers (every field access becomes
  `obj.foo.value`), and makes typed bindings in client libraries
  awkward. It also implies that *every* field has provenance, which is
  not true — provenance only applies to verifiable claims, not to
  internal identifiers or version strings.
- **Provenance only at the trusted-partner-data level.** Rejected. This
  was the v0.7 approach. It fails to express that some fields on a
  profile may be `partner-verified` while others are `self-declared`,
  even when both originate at the partner level. The headline trust
  commitment cannot be honoured without per-field granularity.

## Consequences

- Schemas remain readable; example payloads look like normal JSON.
- The `_provenance` map is optional on objects that include it — absent
  means "no per-field verification recorded", consistent with
  `verification-deferred` semantics.
- Every entry in a `_provenance` map conforms to the
  `ProvenanceEntry` schema in `provenance-map.schema.json`:
  `verification_status` (closed enum), `source` (verifier identifier or
  literal), `as_of` (RFC 3339 UTC).
- Adding new fields to a schema does not require modifying the provenance
  schema; the map is dynamic.

## References

- Spec §4.2 (Provenance Model), §4.3 (Verification Status Codes)
- `provenance-map.schema.json`
- README "What you get back" section
