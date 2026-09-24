# Provenote — public record

Provenote is a verification service. You send it a document; it tells you what can and cannot
be proven about how that document was made. It reads Content Credentials and provider
watermarks, and it says plainly when a file carries nothing at all.

**Try it, without an account or a key: [provenote.us/try.html](https://provenote.us/try.html)**

This repository is not the product. It is the part of our working record that is only worth
anything if a stranger can read it.

---

## Why these documents are public

Our cardinal rule is that we will not claim anything we cannot verify. A register of our
claims that nobody outside the company can read is itself an unverifiable claim about our
discipline. So here it is.

| | |
|---|---|
| [`CLAIMS_REGISTER.md`](CLAIMS_REGISTER.md) | Every public claim we make, its source, and whether it is substantiated. Including the ones we have retracted. |
| [`C2PA_EVIDENCE.md`](C2PA_EVIDENCE.md) | What our C2PA validation actually does, tested against the official public vectors. |
| [`VENDOR_NAMING_POLICY.md`](VENDOR_NAMING_POLICY.md) | How we talk about other companies. Short version: never beside a criticism or a number we derived. |
| [`KEY_CUSTODY.md`](KEY_CUSTODY.md) | Why we do not issue certificates yet. The scheme works; key custody does not exist. |
| [`conformance-evidence/`](conformance-evidence/) | Validation results for every media type we asserted to the C2PA Conformance Program, with the manifest JSON behind each one. |

## The corrections are the point

The claims register carries a corrections log. We publish corrections rather than quietly
fixing them, so the log will grow, and entries will be struck through and replaced. That is
the system working, not a sign of trouble. A register that never changed would mean nobody was
checking it.

The most recent one is a good example. **CORR-008** records a claim we made in correspondence
and got wrong, the expert who corrected us, and what replaced it. It is here because we said
it would be.

## What we do not claim

- That we can prove a human wrote something. **Nobody can.**
- That we are C2PA certified, conformant or compliant. Aethyia Inc. holds **contributor
  membership**, executed the Validator Product Agreement on 22 September 2026, and submitted
  the conformance intake form on 23 September 2026. **We are not listed yet.**
- That we detect Anthropic's Claude watermark. Detection entered private preview and we have
  no access. It is recorded as a check that did not run, never as a check that found nothing.
- That a missing record is evidence of anything. Most documents carry no record of origin, and
  that is normal.

## What is deliberately not here

Everything operational. The staging runbook, the security exceptions register, cloud posture
reports, internal strategy and working logs stay private. A public list of unremediated
findings beside an account number is a shopping list, not transparency.

The published set is assembled by an allowlist and then scanned before it is written, so a new
file cannot be published by accident. If the scan finds an account number, an address, an
infrastructure identifier, a contracted vendor name or key material, the build refuses.

## Contact

`hello@provenote.us` · security disclosures to `security@provenote.us`

Provenote is a product of **Aethyia Inc.**, a Delaware corporation.
