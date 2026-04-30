# 7. Three professional-record creation routes

- Status: Accepted
- Date: 2026-04 (v0.8)

## Context

How does a professional end up in the registry?

The simplest model is "they sign up": a professional registers, claims
their tier, provides credentials, and becomes matchable. That model
works for the small number of professionals who self-register early.
It does not work at scale, because the value of the registry depends on
broad coverage — a consumer asking "who are the lawyers in Travis County,
Texas?" expects coverage of the working bar, not just the few who happen
to have signed up.

The opposite extreme is "ATAH ingests partner data and that's the
registry" — every record is partner-created. That covers regulated
professions where authoritative bodies maintain rolls (e.g. state bar
associations) but excludes established-tier categories where partner
data is sparse and individual self-registration is the dominant path.

## Decision

The spec recognises **three record creation routes**, each with distinct
provenance and a defined merge process:

1. **Partner-created.** A trusted partner pushes the record (e.g. a state
   bar publishing all admitted lawyers). The record exists at ATAH
   without the professional having engaged. `registration_route:
   partner`. The professional may later **claim** the record.
2. **Claimed.** A professional with a partner-created record completes
   claim authentication (spec §15.6) and gains the ability to update
   self-managed fields. The record's provenance map distinguishes
   partner-managed fields from professional-self-declared fields.
3. **Direct self-registered.** A professional registers without a
   pre-existing partner record. They are matchable as
   `registration_route: individual`. If a partner later pushes a
   matching record, the **roll-up** lifecycle handles the merge with a
   pre-merge notification and objection window.

## Alternatives considered

- **Partner-only.** Rejected. Excludes the established tier
  (PR professionals, executive coaches, mediators — categories where
  partner data is optional or weak). Leaves coverage hostage to
  partner participation.
- **Direct-only.** Rejected. Loses partner provenance, which is the
  evidentiary backbone for regulated categories. Also fails the
  coverage problem: a register that requires every lawyer in Texas to
  individually opt in will have a few hundred records, not the working
  bar.
- **Two routes (partner + direct), no claim.** Considered. Without
  claim, a partner-created record can never be amended by the
  professional, which makes the registry useless for them and
  encourages them to sit out. Adding claim was necessary.

## Consequences

- The `registration_route` field on every profile distinguishes
  `partner` from `individual`. Claimed records remain `partner` (they
  originated at a partner) but acquire a self-declared overlay through
  the provenance map.
- The roll-up lifecycle (`rollup-match-record.schema.json`) handles
  partner-record-meets-individual-record collisions with a 7-day
  pre-merge notification window for regulated-tier matches.
- v0.8 specifies the **minimum** claim authentication (verified email
  match against partner records or equivalent partner-attested
  identifier) and defers the hardening (step-up, anomaly detection,
  partner-known-channel notification, rate limiting) to v0.9. The
  reference registry SHOULD apply the hardening operationally at
  launch.
- Hardship waivers and fee caps for individual self-registration
  prevent self-registration becoming a paid moat.

## References

- Spec §15 (Professional Records and Registration), §15.6 (Claim
  Authentication)
- `rollup-match-record.schema.json`
- Charter Part Two: professional participation principles
