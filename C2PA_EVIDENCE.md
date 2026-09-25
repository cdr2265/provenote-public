# Provenote — C2PA Validator Conformance Evidence Packet

Submitted with the C2PA Conformance Program application. Role: **Validator Product**.

**No assurance level.** Assurance levels apply only to Generator Products — corrected
2026-09-09, having previously and wrongly claimed "Level 1" here.

## 1. Product identity
| Field | Value |
|---|---|
| Product | **Provenote** — AI-content verification & provenance service |
| Role | **Validator** (reads + validates Content Credentials) |
| Version | provenance_verify 0.1.0 |
| C2PA SDK | c2pa-python 0.37.7 (c2pa-rs core 0.90.x) |
| Spec conformed to | C2PA Content Credentials 2.x |
| Owner / contact | DC — hello@provenote.us |
| Repository | github.com/cdr2265/provenote (private) |

## 2. What the product does
Provenote aggregates provenance signals over a document and issues a calibrated,
provenance-first report. The **C2PA validator** is a Tier-A signal: it reads any embedded
or detached Content Credential, cryptographically validates it, checks the signer against
the **official C2PA trust list**, and surfaces the validation state — it never treats mere
presence of a manifest as trust.

## 3. Data-flow diagram

```mermaid
flowchart TD
  A["Asset\n(PDF / JPEG / TIFF / GIF / AVIF / JXL / MP4 / MOV / AVI / WAV /\ndetached manifest store)"] --> B{"c2pa Reader\n(read manifest store)"}
  B -->|no manifest| M["credential_missing\n(absence ≠ human)"]
  B -->|manifest present| C["Validate: asset hashes\n+ claim signature"]
  C -->|hash / signature fail| I2["state = Invalid"]
  C -->|hashes + signature OK| D{"Signer chains to the\nofficial C2PA trust list?\n(verify_trust = true)"}
  D -->|no| I1["state = Invalid\n(untrusted signer)"]
  D -->|yes| V["state = Trusted / Valid"]
  V --> S1["Tier-A signal: credential_valid\nweight 0.9, issuer + generator surfaced"]
  I1 --> S2["credential_missing\n(possibly tampered / untrusted)"]
  I2 --> S2
  S1 --> R["Provenance Report\n(provenance-first combination + certificate)"]
  S2 --> R
  M --> R
```

ASCII fallback:
```
asset --> [c2pa Reader] --no manifest--> credential_missing
                |
             manifest
                v
        [validate hashes + claim signature]
                |                     \--fail--> state=Invalid --\
             pass                                                 v
                v                                        credential_missing
   [signer on official C2PA trust list?] --no--> Invalid --------/
                | yes
                v
        state=Trusted/Valid --> credential_valid (Tier A, weight 0.9)
                                          |
                                          v
                         Provenance Report (provenance-first)
```

## 4. How validation works (step by step)
1. **Read** — `c2pa.Reader` parses the manifest store from the asset (or a detached
   store). No manifest → `ManifestNotFound` → outcome `credential_missing`.
2. **Cryptographic validation** — the SDK verifies the asset hard-binding hashes and the
   claim signature (`get_validation_results()` returns per-assertion status codes).
3. **Trust** — with `verify_trust=true` and the official C2PA trust list loaded at startup
   (`configure_c2pa_trust`), the signer certificate must chain to a trust anchor. An
   untrusted signer yields `state=Invalid`.
4. **State → signal** — `get_validation_state()` / `is_valid` are mapped to a Provenote
   Tier-A `SignalResult`:
   - `Trusted` → `credential_valid`, weight **0.9**, confidence 0.99, issuer + generator surfaced.
   - `Valid` (valid but not trust-listed) → `credential_valid`, weight 0.7, with an "issuer unverified" caveat.
   - `Invalid` (hash/sig failure or untrusted) → `credential_missing`, caveat "possibly tampered".
5. **Fail-safe** — any SDK/read error is caught and returned as `error`/`skipped`; a C2PA
   failure never crashes the verification pipeline. If the SDK is absent the adapter self-skips.

Implementation: `provenance_verify/signals/providers.py::C2PAAdapter`,
`provenance_verify/c2pa_trust.py`.

## 5. Trust establishment
- Official trust list **vendored** at `provenance_verify/trust/C2PA-TRUST-LIST.pem`
  (30 anchors) and `C2PA-TSA-TRUST-LIST.pem` (22 TSA anchors), sourced from
  `github.com/c2pa-org/conformance-public/trust-list`.
- Loaded at startup via `configure_c2pa_trust()` (defaults to the vendored list; refresh
  with `scripts/update_c2pa_trust.sh`). `verify_trust=true`.

## 6. Supported asset formats
PDF, JPEG, TIFF, GIF, AVIF, JXL, MP4, MOV, AVI, WAV, and detached C2PA manifest stores
(`application/x-c2pa-manifest-store`). Text-only inputs carry C2PA via a containing asset
(e.g., a signed PDF) or a detached store.

## 7. Test evidence (reproducible)
- **Round-trip proof** — `scripts/c2pa_roundtrip.py`: signs a real asset, validates via the
  adapter → `state=Trusted, weight 0.9`; a single tampered byte → `state=Invalid`.
- **Conformance harness** — `scripts/c2pa_conformance.py`: **13/13** with the official trust
  list loaded — 4 self-generated (valid-trusted / tampered / no-manifest / no-input) plus the
  **9 official** c2pa-org/public-testfiles vectors. This is the only run that exercises the
  official vectors with trust verification **on**; the dedicated public-vectors script below
  deliberately turns it off to isolate cryptographic correctness.
- **The trust list is load-bearing** — `tests/test_c2pa_conformance.py::test_trust_list_is_load_bearing`:
  an asset signed by an off-list cert reads `state=Valid`, weight 0.7, with an explicit
  "signer is not on a trust list" caveat; add that cert as an anchor and the same asset reads
  `state=Trusted`, weight 0.9, no caveat. Both mutations that would hollow this out (equalised
  weights, `verify_trust` silently false) were shown to fail the test on 2026-09-16.
- **Official C2PA public test vectors** — `scripts/c2pa_public_vectors.py`: **8/8** from
  `c2pa-org/public-testfiles` (legacy 1.4). 4 valid provenance chains → `credential_valid`
  (state=Valid); 4 deliberately-broken error cases (E-sig / E-dat / E-uri / E-clm) →
  `credential_missing` (state=Invalid). Validated for cryptographic correctness
  (`verify_trust=false`, legacy test certs). `tests/test_c2pa_public_vectors.py` runs it in CI.
- Reproduce:
  ```
  python -m venv .venv && .venv/bin/pip install c2pa-python
  .venv/bin/python scripts/c2pa_roundtrip.py
  .venv/bin/python scripts/c2pa_conformance.py
  ```

## 8. Security & integrity
- Trust anchors are the **official C2PA list**, pinned/vendored (not fetched live in prod).
- The validator is **read-only**; it holds no signing keys.
- Provenote does not strip, weaken, or evade watermarks or credentials (see Principles).
- Generator role (future): signing uses an X.509 cert from an approved CA with the private
  key in KMS (`c2pa_signer.sign_c2pa(..., sign_callback=...)`) — out of scope for the
  validator application.

## 9. References
- C2PA Conformance: https://c2pa.org/conformance/
- Trust list: https://github.com/c2pa-org/conformance-public/tree/main/trust-list
- Open-source tools: https://opensource.contentauthenticity.org/docs/conformance/

## Conformance Program submission — record 01a0d0e4-b6c7-73e8-84ff-7442cf014805

| | |
|---|---|
| Intake Form submitted | **2026-09-23** |
| Accepted, record ID issued | **2026-09-24** by the Conformance Program Administrator |
| **Record ID** | **`01a0d0e4-b6c7-73e8-84ff-7442cf014805`** — this becomes our entry on the Conforming Products List if the submission is deemed conformant |
| Asserted | specification **2.2** · `image/jpeg` · `application/pdf` |
| Sample evidence sent | **2026-09-24**, four files, in the Administrator's naming convention |
| Status | **in the assessment queue** |

**What was sent**, built by `scripts/build_conformance_evidence.py` into
`for-c2pa-submission/`:

```
a-sample.jpeg          a-sample.crjson.json      image/jpeg
b-sample.pdf           b-sample.crjson.json      application/pdf
```

**Both samples are vectors the 2.2 tree itself lists**, which matters because we asserted
2.2 and our own harness holds them under a `legacy-1.4` directory name: the 2.2 image README
names `adobe-20220124-C`, and the 2.2 PDF README names
`adobe-20240110-single_manifest_store`. Checked 2026-09-24.

**Verified before sending**, from the submission folder rather than the originals:

- `a-sample.jpeg` → `credential_valid`, state Valid, issuer *C2PA Test Signing Cert*
- `b-sample.pdf` → `credential_valid`, state Valid, issuer *Adobe Inc.*
- **Identical with `verify_trust` on and off**, re-tested in **separate processes** because
  `load_settings` is deprecated and could cache within one. So nothing in the evidence
  depends on how the assessor configures trust.
- Both `.crjson.json` files are valid JSON carrying `@context` / `jsonGenerator` / `manifests`.

**Not raised in that thread, deliberately:** the text-scope question of 21 September is still
open. Mixing an unresolved scope question into an evidence submission gives an assessor a
reason to pause the queue. The threads stay apart until one of them closes.

**Open:** the reply's closing line offers evidence for additional media types. If we intend to
widen beyond `image/jpeg` and `application/pdf`, doing it before assessment completes is
cheaper than amending a listed record.

## Sample evidence, round 2 — TRUSTED, 2026-09-24

**The first submission was rejected on its samples, correctly.** The Administrator's note:
samples must be signed with certificates chaining to the official CA and TSA trust lists and
**must produce a verdict of TRUSTED**. Ours were signed against an interim trust list that has
been discontinued.

**Measured, against the official list with `verify_trust` enabled:**

| Sample | State | Issuer |
|---|---|---|
| our old JPEG, legacy 1.4 | `Valid` | C2PA Test Signing Cert |
| our old PDF, adobe-20240110 | `Valid` | Adobe Inc. |
| **their library's JPEG** | **`Trusted`** | **Google LLC** |
| their library's PNG / MP4 / MP3 | `Trusted` | Google LLC |

**The capability was always there; we were not asking for it.** `provenance_verify/c2pa_trust.py`
loads the official anchors, and `scripts/build_conformance_evidence.py` hardcoded
`verify_trust: False` — right for testing deliberately-broken vectors, wrong for a conformance
submission. `scripts/build_submission_pack.py` now exists for the submission case and **deletes
its own output** if any sample falls short of Trusted.

Our vendored `C2PA-TRUST-LIST.pem` (30 anchors) and `C2PA-TSA-TRUST-LIST.pem` (22) were verified
**byte-identical to the live copies** on 2026-09-24.

### ✅ RESOLVED — both media types evidenced as Trusted, 2026-09-24

**Sent 2026-09-24**, superseding the JPEG-only pack sent earlier the same day:

| | Media type | State | Issuer | Provenance of the sample |
|---|---|---|---|---|
| `a-sample.jpeg` | `image/jpeg` | **Trusted** | Google LLC | the library the Programme supplied, unmodified |
| `b-sample.pdf` | `application/pdf` | **Trusted** | OpenAI OpCo, LLC | **ours** — generated via OpenAI Media Service / ChatGPT |

Both re-validated **from inside the zip** rather than from the source files. Zero caveats.

**How the PDF gap was closed, since it was not obvious.** No publicly available PDF reaches
Trusted: the supplied library holds none, and in `public-testfiles` both `2.2/pdf/good` and
`2.2/pdf/bad` contain only a zero-byte README. Rather than wait, we found that **OpenAI Media
Service is on the Conforming Products List as a generator product declaring
`application/pdf`** — and generated our own sample through it. Found by scanning local PDFs for
embedded manifests and validating all 72 candidates against the official list; exactly one
reached Trusted, which proved the route before we used it.

**The letter states the PDF is ours and how it was made.** An assessor who discovers that has
reason to doubt everything else in the pack; an assessor who is told has reason to trust it.
The offer to re-evidence from a different source is left open.

**Still true and worth raising later, not in a submission thread:** Google is on the trust list
and declares `application/pdf` on two conformant products, Google Media Processing Services and
NotebookLM. A PDF from either would close this gap in the library for every future applicant.

### ~~🔴 OPEN: there is no publicly available PDF that reaches Trusted~~ (RESOLVED above, kept for the record)

`application/pdf` is half our assertion and the product's substance, and we cannot evidence it:

- The library supplied to us holds six files — jpg, png, mp3, m4a, two mp4. **No PDF.**
- `public-testfiles` **`2.2/pdf/good` and `2.2/pdf/bad` each contain only a zero-byte README.**
- The only PDF that tree references is the one just rejected.

**Asked of the Administrator 2026-09-24**, with a concrete suggestion rather than a bare request:
**Google is on the trust list and declares `application/pdf` on two conformant products —
Google Media Processing Services and NotebookLM.** A PDF from either, added to the same library,
would settle it for us and for any other applicant asserting PDF.

**Do not quietly drop the PDF assertion while this is open.** Amending an assertion because
evidence is inconvenient is the opposite of the discipline the rest of this document records.

