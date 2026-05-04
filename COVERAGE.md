# ATAH v0.8.2 Launch Coverage

This document shows which categories and jurisdictions are launch-enabled at v0.8.2, which are open for registration but not matchable, and which are deferred to later phases. The protocol is profession-agnostic and jurisdiction-agnostic at the architecture level; this matrix shows where verification pathways and compliance work are actually in place at launch.

The named regulators and professional bodies in the table are illustrative examples of the *types* of organisation the protocol works with for each category and jurisdiction (per the README "A note on named organisations" section). They are not committed partners at v0.8.2 publication.

## Coverage matrix

| Category | Jurisdiction | Launch status | Verification route (illustrative) | Partner / data source (representative example) | Matchable at v0.8.2? |
|---|---|---|---|---|---|
| Property and casualty insurance agent | US (all states and territories) | Phase 1 | Authoritative US licensing data (NIPR-style) | Subject to partner agreement | Self-declared at v0.8.2; partner-verified once relevant agreement is in place |
| Lawyer | US (jurisdictions with state bar partner agreements) | Phase 1 | State bar lookup (illustrative) | Subject to partner agreement | Self-declared at v0.8.2; partner-verified once relevant agreement is in place |
| Lawyer | US (jurisdictions without partner agreements) | Phase 1 | Self-declared | None at launch | Returned with self-declared label only |
| Financial advisor | US | Phase 1 | Authoritative US registers (FINRA-style) | Subject to redistribution terms and partner agreement | Self-declared at v0.8.2; partner-verified once agreement is in place |
| Tax planner / accountant | US | Phase 1 | Self-declared at launch; partner integration prospective | Accounting bodies (AICPA-style) and equivalents | Self-declared only at v0.8.2 |
| Healthcare professional | US | Registration only | HIPAA-compliant variant pending | TBD | No — compliance annex pending |
| Solicitor | UK | Phase 2 | Regulator verification (SRA-style) | Subject to partner agreement | No at v0.8.2 |
| FCA-regulated financial professional | UK | Phase 2 | Regulator register (FCA-style) | Subject to partner agreement | No at v0.8.2 |
| UK tax adviser | UK | Not launch-enabled at v0.8.2 | Pending | Tax-adviser professional bodies (representative candidates: CIOT, ATT, ACCA, AAT) | No at v0.8.2 |
| PR / communications professional | US, UK | Phase 2 | Membership data from communications and PR membership bodies | PRSA, CIPR, and equivalents (representative examples) | Self-declared only at v0.8.2 |
| Coach | All | Phase 2 | Coaching credential check | International Coaching Federation and equivalents (representative examples) | Self-declared only at v0.8.2 |
| Project manager | All | Phase 2 | Credential check from project management bodies | PMI and equivalents (representative examples) | Self-declared only at v0.8.2 |
| All other categories | All | Open for registration | Self-declared | None | Self-declared only at v0.8.2 |

Match responses always carry the `_provenance` map and `presentation_disclosure` block, so AI platforms and consumers see the verification status of every field for every professional, regardless of category coverage.

## What "matchable at v0.8.2" means

A category is matchable at v0.8.2 in a jurisdiction when:

1. A category compliance annex (where required by the regulatory landscape) has been approved and published.
2. A verification pathway exists — either an authoritative registry integration, a trusted partner integration, or a defined self-declared route with the appropriate provenance labelling.
3. The reference registry has the operational machinery to apply the relevant pre-handoff check (Stage 2) for that category.

Categories that are open for registration but not matchable will accept profile creation, partner data, and enhanced verification records, but will not be returned in match responses until they meet the conditions above.

## Roadmap context

This matrix reflects v0.8.2 launch state. Phase 2 (reference registry build and first partner conversations) brings additional jurisdictions and partner integrations as conversations advance. Phase 3 (reference registry operational and first integrations) and beyond extend to additional jurisdictions and categories. Phase progression depends on partner conversations advancing and on resources for reference registry build (whether by ATAH directly or by an external implementer); see PRD §14 for detail.

For the full roadmap, see [ROADMAP.md](ROADMAP.md). For the conformance model that governs how implementations declare what they support, see [CONFORMANCE.md](CONFORMANCE.md).
