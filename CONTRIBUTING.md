# Contributing

Thanks for your interest in contributing to ATAH. This document describes what
you can contribute, how to propose changes, and how those changes are reviewed.

## What you can contribute

- **Specification proposals** — additions, clarifications, or corrections to
  the spec, schemas, OpenAPI contract, or MCP tool definitions.
- **Category compliance annex proposals** — to bring new professional
  categories into the matchable set.
- **Bug reports** against the spec, schemas, or other published artefacts.
- **Documentation improvements** — README, EXPLAINER, GOVERNANCE,
  CONTRIBUTING (yes, this file too), CODE_OF_CONDUCT, SECURITY.
- **Reference implementation contributions** — once the reference
  implementation repository is open, contributions there follow this same
  process at a code level.

## Before you start

1. Read [README](README.md), [EXPLAINER](EXPLAINER.md), and [Charter](CHARTER.md).
2. Check open issues and discussions for related work — please don't open
   duplicate threads.
3. For substantive proposals, **open a discussion before opening a PR.** A
   discussion lets the proposal surface objections cheaply; a PR commits
   reviewer time to a specific implementation.

## How to propose a specification change

1. Open a GitHub issue with the `specification-change` label.
2. Describe the problem and the proposed change. Include:
   - which schemas, endpoints, or sections are affected
   - whether the change is breaking (requires v0.9+) or non-breaking (could
     land in v0.8.x)
   - any backwards-compatibility considerations
3. After discussion, submit a PR. Substantive changes go through governance
   review per [GOVERNANCE.md](GOVERNANCE.md).
4. Editorial changes (typos, clarifications without semantic impact) may be
   merged through standard PR review without governance consultation.

## How to propose a category compliance annex

Category compliance annexes are the mechanism by which new professional
categories become matchable. The process:

1. **Discussion thread** in GitHub Discussions describing the proposed
   category, the regulatory landscape, and the reason for inclusion.
2. **Draft PR** containing the annex content:
   - category metadata (matching weight profile, review weight cap, default
     pre-handoff check, stage 2 auth tier)
   - compliance and verification requirements relevant to the category
   - regulatory considerations (sectoral law, jurisdiction-specific
     constraints)
   - validator stack mapping (which trusted partners or verifiers cover the
     category at launch)
   - compliance gating: the category remains `compliance-pending` until the
     annex is signed off
3. **Governance review** per GOVERNANCE.md.
4. **Publication** as a category record in `profession-categories.json` plus a
   companion annex document under `docs/compliance-annexes/<category>.md`.

## How to report a bug

1. Open a GitHub issue with the `bug` label.
2. Include:
   - affected file or section
   - what you expected
   - what happened
   - evidence (a JSON sample, a curl command, a quoted spec passage)

## How to report a security issue

**Do NOT open a public issue for security disclosures.** See
[SECURITY.md](SECURITY.md) for the responsible-disclosure process.

## Code of Conduct

All contributors agree to the [Code of Conduct](CODE_OF_CONDUCT.md), including
ATAH-specific commitments around professional and consumer data, conflict of
interest disclosure, and antitrust expectations.

## Governance review process

Substantive changes go through governance review as described in
[GOVERNANCE.md](GOVERNANCE.md). The review includes:

- Charter compliance check
- Cross-document consistency
- Implementation impact assessment
- Public review window where required by GOVERNANCE.md

## Compatibility handling

- **Non-breaking additions** can land within a minor version (e.g. v0.8.1).
- **Breaking changes** require a version increment to the next minor or
  major.
- **Deprecation policy:** deprecated fields or endpoints are marked with a
  `deprecation_warning` and remain present for at least one minor version
  (per spec §7.1).

## Licensing

By contributing, you agree that your contribution is licensed under the
Apache License 2.0 (see [LICENSE](LICENSE)). Substantive contributors are
named in the CHANGELOG; ongoing contributors may be listed in a contributors
file once the project is established.

## Contact

- General: `hello@atahprotocol.org`
- Governance: `governance@atahprotocol.org`
- Security: `security@atahprotocol.org`
