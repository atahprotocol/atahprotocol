# Consent storage rationale

This document explains how consent is captured, stored, and disputed in
ATAH v0.8.2. It is design rationale, not legal advice. Each AI platform
considering integration should review with their own counsel.

## What ATAH stores

For each consent action, ATAH stores:

- A SHA-256 hash of the canonical consent receipt
- Receipt metadata: scope (which stage of which handoff type), data categories
  covered (declared, not the data itself), professional reference, asserting
  platform identifier, captured timestamp, expiry, revocation support flag,
  current revocation status
- The mapping between hash and handoff record

ATAH does NOT store:

- The full consent text shown to the consumer
- The consumer's name, email, phone, or any identifier
- The consumer's IP address, device, or session details
- The full receipt payload

## What the asserting AI platform stores

The platform asserting consent on behalf of a consumer stores:

- The full structured consent receipt, including the consent text shown to
  the consumer
- Whatever consumer identity binding the platform's own consent capture
  process produces (typically already part of the platform's existing user
  account and audit infrastructure)
- The receipt hash that was submitted to ATAH

## Why this split

ATAH's architecture treats consumer personal data as transient. The
registry is not a personal data store. Consent receipts that included
consumer-identifying data and full consent text would make ATAH a
long-tail repository of personal data — exactly the model the v0.8
privacy floor (Charter Part One core commitment) is designed to prevent.

The hash-and-metadata pattern follows transparency log models used in
software supply chain (Sigstore) and certificate authority (RFC 6962)
systems. Tampering with the platform-side full receipt is detectable by
hash comparison without ATAH holding the receipt content.

### Scope of the privacy claim

The hash-and-metadata pattern is what ATAH does to avoid becoming
another persistent consumer-PII store. What it does not do, and should
not be read as doing, is control the full downstream data lifecycle.
The honest framing:

> ATAH reduces central honeypot risk by avoiding another persistent
> store of consumer personal data. It does not control the full
> downstream data lifecycle inside AI platforms, professional systems,
> CRMs, or assistants.

The asserting AI platform retains the full consent receipt for its own
records (per its developer terms). The professional, after retrieving
the consumer's data from the transient encrypted vault, may write that
data into a CRM, an engagement record, or other systems under their
own data-protection framework. ATAH's structural mechanisms — payload
erasure of vault contents (spec §11.6), audit retention without
consumer PII in plaintext (spec §11.8 HMAC-not-plain-hash), and
withdrawal-as-state-transition (spec §11.9) — are the protocol-layer
guarantees ATAH provides. They are what ATAH does, and they are what
the consent-storage split is designed to support; they are not a
guarantee about systems ATAH cannot reach. The continuity-binding model
and the three-concept separation of erasure / audit retention /
withdrawal are what ATAH delivers; the privacy-scope framing names what
they do not promise.

## What happens at dispute

If a consumer or regulator disputes whether consent was validly
obtained:

1. The asserting platform produces the full structured consent receipt
   from their records.
2. ATAH verifies that the SHA-256 hash of the produced receipt matches
   the hash stored at consent time.
3. If hashes match: the receipt has not been altered post-hoc and is
   evidence of what the consumer was shown at consent time.
4. If hashes do not match: the receipt has been altered post-hoc;
   the platform's claim of consent is not supported by ATAH's stored
   metadata.

This pattern provides demonstrable consent suitable for GDPR Article 7
and ISO/IEC 29184-style frameworks while keeping the personal-data
storage burden on the platform that captured the consent — which is
the entity with the consumer relationship and the obligation to manage
that data anyway.

## What the platform takes on

A platform asserting consent on behalf of a consumer to ATAH is taking
on:

- The obligation to capture consent in a form that meets applicable
  legal standards (which the platform already manages for its own
  product)
- The obligation to retain the structured consent receipt for the
  receipt's expiry plus a reasonable buffer
- The ability to produce the receipt on demand at dispute

A platform is NOT taking on:

- An obligation to indemnify ATAH or any partner against consent
  disputes
- An obligation to retain consent receipts beyond the receipt's
  declared expiry plus normal evidence-retention practice
- An obligation to allow ATAH to access platform-side receipts other
  than via the dispute process

## What v0.9 will change

v0.9 will specify an optional platform-light consent mode:

- ATAH provides canonical consent text for each stage of each handoff
  type
- ATAH provides a consent flow orchestration helper
- The platform invokes the standard ATAH consent flow rather than
  constructing receipts itself
- The legal responsibility for capturing consent remains with the
  platform; ATAH provides standard machinery

The v0.8.1 self-asserted-receipt model continues to be supported in
v0.9 alongside the platform-light mode. Platforms that prefer their
own consent UX retain the ability to use it.

## Where this is normative

The hash-and-metadata storage rule is normative (§4.10 of the
specification and the privacy floor in Charter Part One). The split
of obligations between platform and ATAH is described here as
design rationale; the obligations themselves are governed by the
platform's own legal compliance and any agreement between the
platform and the consumer.
