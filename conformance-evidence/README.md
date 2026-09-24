# Provenote — Validator Product conformance evidence

**Applicant:** Aethyia Inc. (Delaware) · **Product:** Provenote
**Asserted on the Program Intake Form, 23 September 2026:** specification **2.2**,
media types **`image/jpeg`** and **`application/pdf`**
**Software version:** `provenance_verify` 0.1.0 · **SDK:** `c2pa-python` 0.37.7

---

## What this is

The C2PA Conformance Program v0.2 states:

> Validator Product applicants must provide sample validation results of every asserted
> validate media type … along with their associated `.json` or `.crjson` files for analysis.

This pack is that, for both asserted media types, produced by running the official public
test vectors through the shipping product and recording what it said.

**Nine vectors. Nine matched the expected outcome.** See `RESULTS.md` for the table and
`INDEX.json` for the machine-readable record.

## Why the failing vectors are here

Four of the eight image vectors are **supposed to fail**, and they do. They are included
deliberately, because a validator that only ever says yes has demonstrated nothing. Each
record states which property the vector breaks:

| Vector | What is broken | Expected |
|---|---|---|
| `E-sig-CA` | signature does not verify | must not validate |
| `E-dat-CA` | hashed data does not match the claim | must not validate |
| `E-uri-CA` | a referenced URI does not resolve within the manifest | must not validate |
| `E-clm-CAICAI` | the claim itself does not verify | must not validate |

Provenote reports all four as `credential_missing` with c2pa state `Invalid`. We report a
broken manifest as *missing* rather than *valid*, because a manifest that fails validation is
not positive evidence. The distinction between "broken" and "absent" is preserved in the
caveat and surfaced to the reader.

## On `verify_trust`

The legacy Adobe vectors are signed with **test certificates**, which are not on the
production trust list. This pack therefore validates for **cryptographic correctness**
(`verify_trust: false`), which is the dimension the error vectors exercise. Validating them
against the production trust list would fail all nine for the same uninteresting reason and
would demonstrate nothing about the implementation.

In production we do the opposite, and we distinguish the two. A signature that verifies
cryptographically while its issuer is **not** on the trust list is reported as valid **with an
explicit caveat that we cannot say who signed it** — never as trusted. That behaviour is
visible on the public demo below.

## Layout

```
INDEX.json                    every result, plus environment and vector hashes
RESULTS.md                    the same as a table
image-jpeg/
  <vector>.json               manifest store JSON from the c2pa Reader
  <vector>.crjson             crjson
  <vector>.validation.json    our verdict vs expected, c2pa state, detail, caveats,
                              vector SHA-256, and the c2pa validation results
application-pdf/
  … same three files
```

Every record carries the **SHA-256 of the vector it was produced from**, so any file here can
be tied back to the exact bytes it describes.

## Ingesting your own samples

The programme document says:

> We recommend that Validator Product applicants provide the Conformance Program access to
> validation evidence so we can ingest our own samples into the Applicant's product.

Two ways, both live:

- **`https://provenote.us/try.html`** — no key, no account, no sign-up. Send a file, get a
  verdict. This is the fastest way to push your own samples through the product.
- **`https://api.provenote.us`** — `POST /documents/verify`, key-gated. Ask and we will issue
  a key for the Conformance Program; it is a per-partner key, revocable on its own without
  affecting anyone else.

Both run the same engine that produced this pack.

## Reproducing it

```
scripts/build_conformance_evidence.py
```

in the Provenote repository. It reads the expected outcomes from the committed test-suite
fixture rather than from a list inside itself, so this pack cannot quietly disagree with the
tests. It exits non-zero if any vector does not match, so a pack containing a mismatch is
never produced silently.

## What we have not asserted

**Text media types are not asserted**, and no text evidence appears here. We validate detached
text credentials and have six passing tests for it, but the Text Asset Conformance Rubric
checks `text:exclusions_defined`, which we read as assuming a manifest embedded in the asset —
and embedding into text is not something the current SDK supports. That reading is an
inference, and we will not assert a warranted media type on an inference. We have asked the
Conformance Program to clarify and will apply to extend the listing if the answer allows it.

The same discipline explains the narrow media-type list: `image/jpeg` and `application/pdf`
only. We ticked what a standing test proves and nothing else.

---

**Contact:** info@aethyia.com
