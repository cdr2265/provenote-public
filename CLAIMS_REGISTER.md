# Provenote — Claims Register (substantiation file)

## ★ Cardinal rule
> **Provenote will not claim anything that cannot be verified.**
> This governs the product, the marketing site, and the blog alike. Every factual or
> performance statement must trace to evidence a reader could check. If it can't be
> verified, it doesn't ship — no exceptions, no "probably," no borrowed numbers.

> Every public claim → its narrow, testable form → the evidence that backs it → status.
> This is both our credibility asset and our defense against unsubstantiated-AI-claim
> scrutiny (FTC/ASA). **Rule: don't ship a claim that isn't in this table with evidence.**
> Status: `substantiated` / `partial` / `aspirational` / `todo`.

| ID | Public claim (as worded) | Narrow, testable form | Evidence / test | Status |
|----|--------------------------|-----------------------|-----------------|--------|
| C1 | "Neutral verification layer over every AI watermark and detector" | Aggregates provider watermark detectors (SynthID today; others as their APIs ship) + BYO statistical detectors, on one calibrated report | Architecture: `signals/` adapters; SynthID working; Anthropic adapter stubbed pending API | **partial / aspirational** — say "watermarks and detectors," reserve "every" for vision |
| C2 | "We prove how content was made — we never hide it" | We report authoritative provenance when present + calibrated secondary signals; we never claim to prove *human* authorship | `report.py` combination rules; "absence ≠ human" on every report | **substantiated** |
| C3 | "Provenance-first: a watermark drives the verdict; a detector can never reach 'confirmed'" | Tier-A present ⇒ confirmed; Tier-C alone ⇒ at most "lead for review" | `report.py` + `tests/test_report.py::test_statistical_alone_cannot_confirm` | **substantiated** |
| C4 | "Absence ≠ human" | "No signal detected" is never rendered as "human-written" | schema `const` disclaimer; `test_absence_never_means_human` | **substantiated** |
| C5 | "We measure detectors' real false-positive rate and weight by it" | Each Tier-C detector's weight = measured FPR from the harness, FP-penalized | `benchmark/harness.py`; `test_harness_penalizes_false_positives` | **substantiated** (own harness; expand corpus) |
| C6 | "Every check returns a tamper-evident, re-checkable certificate" | Certificate is **Ed25519-signed over canonical JSON**; the signer and the public verifier are now proven to agree, so anyone holding the published public key can re-check a certificate offline | `ledger.py` (Ed25519) + `web/api/verify.js`; `tests/test_cert_signing.py` runs the **shipped JavaScript** against Python-signed certificates and asserts tampering is rejected — 2026-08-30 | **substantiated for the scheme. ⚠️ Key custody still pending: the private key must move into a KMS/HSM before any real certificate is issued. Do not describe production certificates as issued until then.** |
| C7 | ~~"Detectors claiming 99% deliver 66–92% real-world"~~ | **WITHDRAWN 2026-08-30 — see CORR-006.** The 12-tool comparison it rested on is published by a company that **sells its own AI detector and ranks itself first**, so it is not independent testing; and it reports roughly **60–80%**, not 66–92%. Replaced by C7a. | — | **🔴 WITHDRAWN — do not reuse** |
| C7a | "Detector accuracy varies sharply by tool; one detector reached 99.8–100% while other commercial tools performed materially worse" | Named-study claim about specific tested tools, not a claim about the category | Jabarian & Imas, "Artificial Writing and Automated Detection," University of Chicago Booth (2025), SSRN 5407424 — verified 2026-08-30 | **substantiated (named academic study; revalidate 2027-02)** |
| C8 | ~~"≈1 in 20 honest writers wrongly flagged by leading detectors"~~ | **WITHDRAWN 2026-08-30 — see CORR-006.** Rested on "independent 2026 detector reports", which is a gesture and not a citation. The one comparison behind it tested false positives on **five human texts**. Replaced by C8a. | — | **🔴 WITHDRAWN — do not reuse** |
| C8a | "Only one tool tested held false positives at or below 0.5% without sacrificing accuracy; others did not" | Named-study claim; states the tested set, not all detectors | Jabarian & Imas, UChicago Booth (2025) — verified 2026-08-30 | **substantiated (named academic study; revalidate 2027-02)** |
| C8b | "At a **claimed** 1% false-positive rate, a university calculated ~750 of ~75,000 papers could be wrongly flagged in a year" | The institution's **own** arithmetic using the **vendor's advertised** rate — not a measurement we made | Vanderbilt University published guidance, 16 Aug 2023 — fetched and verified 2026-08-30 | **substantiated (primary, first-party)** |
| C8c | "OpenAI's own AI Text Classifier flagged 26% of AI text and mislabelled 9% of human text; retired for low accuracy" | A company's published figures about its **own** discontinued product | openai.com — verified 2026-08-30 | **substantiated (primary, first-party)** |
| C9 | "SynthID watermark — TPR 1.0 / FPR 0.0" | On an n=12 own-key labeled corpus at threshold 0.52; NOT a claim about any production watermark | `scripts/run_benchmark.py`; caveat shown on `/reliability` | **substantiated (narrow; small-n, disclosed)** |
| C10 | "EU AI Act Art. 50 transparency duties in force 2 Aug 2026; Art. 50(2) marking grace period to Dec 2026 for pre-existing systems" | Duties apply from 2 Aug 2026. The Digital Omnibus on AI grace period for the machine-readable marking duty covers ONLY generative systems placed on the EEA market before 2 Aug 2026. | artificialintelligenceact.eu/article/50 (Art. 113 date); EC "Quick facts: transparency rules for AI systems" (grace period) — both re-fetched 2026-08-30 | **substantiated (factual, cited)** — CORRECTED 2026-08-30, see note below |
| C11 | "We do not strip, weaken, or evade watermarks or detectors" | Product has no removal/evasion capability | codebase (no such feature); Principles page | **substantiated** |
| C22 | "Four kinds of signal, and an honest column for where each one stands today" (`/what-we-check.html`, home teaser) | The published status of each is exactly: C2PA **working**; SynthID **partly**, own keys only; Claude watermark **not yet**, no access; statistical detectors **supported but capped**. No signal is described as working beyond what `BETA_CAPABILITY.md` records | `tests/test_c2pa_public_vectors.py`, `tests/test_c2pa_detached_text.py`, `tests/test_c2pa_pdf_manuscript.py`; `scripts/synthid_roundtrip.py`; `signals/providers.py` AnthropicWatermarkAdapter returns SKIPPED; `signals/statistical.py` | **substantiated** |
| C23 | "We cannot read Google's or Anthropic's production watermarks" (`/what-we-check.html`, `/methodology.html`) | SynthID detection works only on watermarks generated with keys we hold; Anthropic detection is private preview and we have not been granted access | `synthid_adapter.py` docstring + `BETA_CAPABILITY.md`; Anthropic access re-verified absent 2026-09-21 | **substantiated** |
| C24 | "We are not a plagiarism checker, we produce no similarity report, we are not an AI-writing detector" (`/what-we-check.html`, home) | A statement of absence: we hold no document corpus, perform no document-to-document matching, and emit no similarity percentage | No corpus or matching code exists in `provenance_verify/`; the only style-based input is the capped BYO adapter in `signals/statistical.py` | **substantiated** |
| C25 | "An authorship record only helps if it exists before the question is asked" (`/methodology.html`) | We state that a signed draft chain cannot be reconstructed after the fact, and we do not offer to do so | `ledger.py` chains drafts forward only; matches the standing rule in `CLAUDE.md` | **substantiated (a guard)** |
| C26 | "No tool can tell you which essays were written by AI, including ours" (`/universities.html`) | An explicit refusal, paired on the same page with the contested state of the evidence (C19 both ways) and Vanderbilt's own arithmetic (C8b) | Cites Liang et al. *Patterns* 2023 **and** the Feb 2026 Czech preprint that found no systematic bias, so the disconfirming result ships with the claim | **substantiated (a guard)** |
| C27 | "Our engine can run inside your own environment" (`/universities.html`) | Stated as architecture, not availability: no database, no state, nothing written to disk. The page says outright it is **not yet packaged as an installable product** and there is no hosted service | `provenance_verify/api.py` is stdlib-only and stateless. **Updated 2026-09-23:** the engine is now deployed at `api.provenote.us`, so `BETA_CAPABILITY.md` no longer says "not deployed anywhere". The claim is unaffected — it is about running inside *your* environment, and the page still states outright that there is no installable package | **partial — capability real, packaging explicitly disclaimed on the page** |
| C20 | "We do not claim that a watermark survives any given attack, or that it identifies a specific asset" | No Provenote surface asserts watermark robustness, payload capacity, or asset-level identification from a soft binding. If we ever do, the **error-corrected** bit budget and the collision bound are the disconfirming fields and must be stated with the claim | No such claim exists in `web/build.mjs`, the report copy or `BETA_CAPABILITY.md`; the constraints and their numbers are held internally | **substantiated (a guard, not a capability)** |
| C21 | "A watermark can always be removed. Provenance is opt-in, not tamper-proof" | We never describe a watermark, a soft binding or a Content Credential as indelible, permanent, unremovable or tamper-proof — only as **tamper-evident**, which is a different and weaker claim we can actually support | **Checkable on our own surfaces, which is the point:** "tamper-proof", "tamperproof", "indelible", "unremovable", "permanent record" and "cannot be removed" return **zero occurrences across ten live pages of provenote.us**, verified 2026-09-23. C6 is deliberately worded *tamper-evident* | **substantiated (a guard)** |

## Standing rules
1. **Performance claims carry their conditions** (corpus, length, threshold) inline or one click away.
2. **Third-party claims are cited** at point of use.
3. **New marketing copy** gets a row here *before* it ships, or it doesn't ship.
4. **Re-verify on change** — when the harness corpus or a detector's measured FPR changes, update C5/C7/C8/C9 and the site numbers together.

## Blog fact-check protocol (per the cardinal rule)

Every blog post logs its verifiable factual claims here before it is scheduled. A
scheduled check runs **3 hours before each post's release**, re-verifying every claim
against its live source. **If any claim has changed, been retracted, or can no longer be
verified, open a GitHub issue in `cdr2265/provenote` (primary channel) and email
`cdrtaj@gmail.com`** with title/subject **`Provenote Blog changes - blog <N>`**,
listing the claim, the old value, and what changed — so a human decides before it goes
live. No change → no email (silent pass). A post that cannot be re-verified does not
auto-publish.

### Blog 3 — "Grading detectors honestly" (scheduled)
| Claim in post | Source (verify at T-3h) | Value as written |
|---|---|---|
| OpenAI retired its AI Text Classifier for "low rate of accuracy" (2023) | openai.com/index/new-ai-classifier-for-indicating-ai-written-text/ | discontinued Jul 20 2023 |
| That classifier: 26% of AI text caught; 9% of human text mislabeled | OpenAI's own reported figures (as above) | 26% TP / 9% FP |
| >50% of non-native TOEFL essays misclassified as AI-generated | Liang et al., *Patterns* (2023), sciencedirect.com/science/article/pii/S2666389923001307 | "more than half" |
| EU AI Act Art. 50 transparency duties in force since 2 Aug 2026; Dec 2026 = marking grace period for pre-existing systems | artificialintelligenceact.eu/article/50 ; EC quick-facts page (see C10) | in force 2 Aug 2026 |

### Blog 4 — "Article 50, in plain English" (scheduled 2026-09-13)
| Claim in post | Source (verify at T-3h) | Value as written |
|---|---|---|
| Art. 50 transparency duties have applied since 2 Aug 2026 (NOT moved to Dec); Dec 2026 = Art. 50(2) marking grace period, pre-existing systems only | artificialintelligenceact.eu/article/50 ; digital-strategy.ec.europa.eu quick-facts page | in force 2 Aug 2026 |
| Four duties: interactive-system disclosure; machine-readable marking of AI output; emotion/biometric notice; deepfake + public-interest-text disclosure | EC quick-facts page / Art. 50 text | as listed |
| A hidden machine-readable mark alone does NOT satisfy the deepfake disclosure duty (deployer must add human-perceptible label) | Art. 50(4) text (the "disclose" duty) **+ Art. 50(5)**, which is the actual anchor for perceptibility: information under paras 1-4 "shall be provided to the natural persons concerned in a clear and distinguishable manner at the latest at the time of the first interaction or exposure" | as written |
| Provenote published an incorrect version of the Art. 50 date and corrected it (stated in-post) | This register + GitHub issues #5/#6 (2026-08-30 hold) | as written |

### Blog 5 — "Content Credentials for text" (scheduled 2026-09-27)
| Claim in post | Source (verify at T-3h) | Value as written |
|---|---|---|
| C2PA text support arrived in stages: **2.3 (December 2025)** added embedding into unstructured text files; **2.4 (April 2026)** added HTML and structured text formats, alongside the document containers (PDF, Office/OOXML, ODF, EPUB/ZIP) already covered | spec.c2pa.org 2.4 specification changelog | 2.3 / 2.4, **not 2.2** (CORR-003) |
| **2.4** introduced the AI Disclosure Assertion `c2pa.ai-disclosure` | Same source | as written |
| A C2PA manifest binds to the file/container, not the raw text; extracted/pasted text loses the credential | C2PA spec (soft/hard binding) ; c2pa.org explainer | as written |
| Provenote validates real C2PA manifests against the official trust list (internal) | C6/C2PAAdapter + `scripts/c2pa_conformance.py` — 13/13 incl. the 9 official public test files, run with the vendored official trust list loaded; `tests/test_c2pa_conformance.py::test_trust_list_is_load_bearing` proves an off-list signer is reported differently from an on-list one | as written, **with its qualifier**: the legacy Adobe test certs are not themselves on the official list, so those vectors validate as `Valid` (sound signature, untrusted signer), not `Trusted`. The list is loaded and enforced; our own on-list vector is the `Trusted` case. **Evidence corrected 2026-09-16 (FIX-003)** — this row previously cited `c2pa_public_vectors.py`, which runs with `verify_trust=false` and therefore could not support a trust-list claim. |

### Blog 6 — "The asymmetric cost of a false accusation" (scheduled 2026-10-11)
| Claim in post | Source (verify at T-3h) | Value as written |
|---|---|---|
| Vanderbilt disabled its AI detector in Aug 2023, stating it did not believe AI detection software is an effective tool that should be used | vanderbilt.edu/brightspace 2023-08-16 (their own published guidance) — fetched + verified 2026-08-30 | as written |
| At a **claimed** 1% false-positive rate against ~75,000 papers submitted in 2022, ~750 could have been incorrectly flagged | Same Vanderbilt post — this is **their** arithmetic using the **vendor's own claimed** rate, not a figure we derived | ~750 of ~75,000 |
| Measured false-positive rates for leading detectors run ~2%–5% | See C8/C18 — category-level, no vendor named | ~2%–5% |
| >50% of TOEFL essays by non-native writers misclassified as AI-generated | Liang et al., *Patterns* (2023) — see C19 | "more than half" |
| OpenAI retired its own AI Text Classifier in 2023 for low accuracy; 26% of AI text caught, 9% of human text mislabelled | OpenAI's own published statement about its own product — the one named reference permitted under VENDOR_NAMING_POLICY §3 | 26% TP / 9% FP |
| **The 10,000-essay worked illustration (5% prevalence, 80% sensitivity, 4% FPR → 380 of 780 flags are innocent)** | **NOT a measurement.** Pure arithmetic on **stated assumptions**, labelled in-post as "assumptions, chosen to be generous to the detector, not measurements". Anyone can re-run it. | ~49% of flags wrong |
| A verifiable authorship record only helps if it exists BEFORE the accusation | Stated in-post as an explicit limitation; follows from C12/C13 | as written |

**Naming check (VENDOR_NAMING_POLICY):** no detector vendor is named in the prose. The
Vanderbilt citation is a third party's published statement about its **own** decision, and the
OpenAI reference is that company's published statement about its **own** discontinued product —
both permitted under §3. The 750-paper figure is Vanderbilt's own arithmetic using the vendor's
**advertised** rate, so it is not a measurement we placed beside a company's name.

### Blog 7 — "Disclosure isn't verification" (scheduled 2026-10-25)
| Claim in post | Source (verify at T-3h) | Value as written |
|---|---|---|
| EU AI Act Art. 50 transparency duties have been in force since 2 August 2026 | artificialintelligenceact.eu/article/50 (Art. 113); EC quick-facts page — see **C10** | in force 2 Aug 2026 |
| Art. 50 contains **two distinct duties**: machine-readable marking (provider) and human-perceptible disclosure (deployer) | Art. 50(2) and Art. 50(4), with Art. 50(5) supplying the perceptibility standard | as listed |
| A hidden machine-readable mark alone does NOT satisfy the deepfake / public-interest-text disclosure duty; a person must be able to perceive it at first exposure without a detection tool | Art. 50(4) **+ Art. 50(5)** ("clear and distinguishable manner at the latest at the time of the first interaction or exposure") — see **Blog 4** row | as written |
| A declaration cannot be checked, has no granularity, binds no one, and does not travel with copied text | Reasoning from the nature of a self-declaration, not an empirical claim. Stated as argument, not measurement. | as written |
| A record only helps if it exists **before** the question is asked | Explicit limitation; follows from **C12/C13** | as written |

**Naming check:** no vendor named anywhere in the post. All regulatory claims are
category/legal-text level. The post asserts no compliance outcome and repeats
"compliance-supporting, not legal advice".

### Blog 8 — "The case we cannot solve" (scheduled 2026-11-08)
| Claim in post | Source (verify at T-3h) | Value as written |
|---|---|---|
| OpenAI described its text-watermarking method as "highly accurate and even effective against localized tampering, such as paraphrasing" | openai.com/index/understanding-the-source-of-what-we-see-and-hear-online/ — **quoted verbatim**, so it must match word for word | direct quote |
| …but "less robust against globalized tampering; like using translation systems, rewording with another generative model" | Same source — **quoted verbatim** | direct quote |
| Among its reasons for not releasing it, OpenAI cited that watermarking could "stigmatize use of AI as a useful writing tool for non-native English speakers" | Same source | direct quote |
| OpenAI has not shipped a public text watermark for ChatGPT | Absence of any such announcement. **If this changes it is a material failure** — the post's premise is that most AI text carries no mark | as written |
| Anthropic's text watermark covers **future** Claude models, with older ones transitioning | anthropic.com/news/claude-text-watermark | as written |
| That watermark says nothing about ownership or authorship, and cannot distinguish "Claude wrote this" from "Claude heavily edited this" | Same source + **C12/C13** | as written |
| Provenote has **no access** to the detection API (private preview from 2026-09-01) | `docs/BETA_CAPABILITY.md` — stated as a limitation, never as a capability | as written |
| A credential cannot be added after the fact, and style comparison cannot close the gap | **NOT a measurement.** Reasoning from what a provenance record is; presented as argument, never as data | as written |

**Naming check:** OpenAI and Anthropic each appear only as **that party's own published
statement about its own product** — permitted under VENDOR_NAMING_POLICY §3, and the same
basis as the Blog 6 rows. No other vendor is named, and no vendor name sits beside a figure
Provenote derived. Both quotations are verbatim: a paraphrase that drifts is a claim we could
not defend.

## Open tightening (site edits)
- C1: keep "every" only as vision; ensure the substantive layer copy says "watermarks and detectors."
- C6: once KMS signing is live, promote from *partial* → *substantiated* and add a public "verify a certificate" page.
- Add a "Sources ›" link under the homepage "Why now" stats → `/reliability` for C7/C8 traceability.

## Audience & FAQ copy (added 2026-08-30)

Every claim in the new "Who it's for" homepage section and the new FAQ entries, with the
limit that keeps each one defensible. Governed by `VENDOR_NAMING_POLICY.md`.

| ID | Claim | Narrow/testable form | Evidence | Status |
|----|-------|----------------------|----------|--------|
| C12 | "A tamper-evident record of how your draft came together, built as you write" | Chained draft hashes + signed certificate; **only covers drafts recorded while writing** — it cannot reconstruct a trail after the fact | `ledger.py`; `/api/verify` round-trip + tamper tested | **substantiated (design; KMS signing pending — see C6)** |
| C13 | "We cannot clear a student who has no record" | Stated explicitly in the FAQ as a limitation | Follows from C12 | **substantiated (stated limitation)** |
| C14 | "A detector score, on its own, does not establish that someone used AI" | Statistical detectors are Tier-C, capped, and can never alone reach "confirmed" | `report.py` combination rules; `/reliability` | **substantiated** |
| C15 | "We do not render verdicts about people" | No output asserts a person's conduct; reports state evidence + uncertainty | Product design; Principles page | **substantiated** |
| C16 | "A certificate is examinable — signed, tamper-evident, re-checkable against a published public key" | Deliberately **does not** claim admissibility, which is stated to be for the court | `web/api/verify.js`; wording in FAQ + Legal card | **substantiated (scope-limited)** |
| C17 | "Compliance-supporting, not legal advice" (EU AI Act Art. 50) | We read Art. 50 marks and produce evidence; we do not assert that a customer is compliant | See C10 | **substantiated (scope-limited)** |
| C18 | ~~"Vendors market 99%+; independent testing lands at 66–92%"~~ | **WITHDRAWN 2026-08-30 — see CORR-006.** De-naming made it read as a category claim, which the evidence does not support: the best available academic study found one tool essentially meeting its claims. Use **C7a** instead. | — | **🔴 WITHDRAWN — do not reuse** |
| C19 | "More than half of TOEFL essays by non-native writers were wrongly flagged **by the detectors tested in 2023**" | Must carry the date and tested population. **Do NOT state as a current or general property of detectors** | Liang et al., *Patterns* (2023) — re-verified 2026-08-30. **CONTESTED** by Al Ali/Helcl/Libovický, arXiv:2602.05769 (Feb 2026 preprint, Czech, three detector families, no systematic bias found) | **substantiated as a 2023 finding; contested as a present-tense claim — see CORR-002. Revalidate before any reuse.** |

**Standing limits restated in the copy itself:** we never claim to prove a human wrote
something (see FAQ); we never name a vendor beside a figure we derived; we never assert
admissibility or compliance.

### Status change — Claude watermark detection API, 2026-09-01

Anthropic's detection API entered **private preview** on 1 September 2026 (page updated that
day). Verified at the primary source before any copy was changed.

**What is now true:** detection exists, gated to eligibility categories Anthropic lists —
regulators, law enforcement, media, fact-checkers, researchers, educational organisations, EU
civil society groups, and enterprises with EU AI Act compliance duties. Access is by request
form.

**What is NOT true and must not be implied:** that Provenote can detect Claude watermarks.
We have not requested access, have not been granted it, and have performed no detection. The
`anthropic_watermark` adapter still reports **skipped**, and its reason now names the private
preview rather than saying the API does not exist.

**Copy updated the same day:** blog 2, both sample reports, the coverage remedy text, the
adapter docstring and skip reason, `/news`, and `BETA_CAPABILITY`.

**One surface was missed and ran stale for a week (found 2026-09-08).** The homepage
"What we do not claim" panel still read "some (Claude) are not yet live". After 1 September
that was false: the detection API is live, we simply cannot reach it. The sweep above was
assembled by listing the places that discuss the watermark, and this bullet discusses the
*absence* of a detection API, so it did not read as a hit. Corrected to name the private
preview and our lack of access. **Lesson: a status change invalidates the claims that assert
the old status is still true, not only the claims that describe the subject** — grep for the
old status, not just the topic.

**Promote to a capability claim only when:** access is granted AND a real Claude-watermarked
sample has been detected end to end through our own pipeline. Not on access alone.

**Worth recording:** the watch routine we built on 2026-08-30 caught this within 3.5 hours,
alerted by email and push, and correctly distinguished private preview from general
availability without being told to. The stale-claim risk this session has repeatedly exposed
was, this time, caught by machinery rather than by an outside review.

### Status change — C2PA contributor membership, 2026-09-09

**C2PA confirmed CONTRIBUTOR MEMBERSHIP for Aethyia Inc.** (owner-reported). Contributor
Member is the broadest of C2PA's three tiers — Steering Committee, General, Contributor —
verified against c2pa.org/membership on 2026-09-09.

**What is now true:** Aethyia Inc. participates in the C2PA. We may say, exactly:
*"Aethyia Inc. is a Contributor Member of the C2PA."*

Capitalised as a proper tier name, matching c2pa.org's own heading exactly (owner-confirmed
2026-09-09). The point of this claim is that a reader can find the same words on C2PA's page;
paraphrasing it — "contributing member", "C2PA partner", "member of the C2PA coalition" —
breaks that match and makes a checkable claim slightly less checkable for no gain.

**What is NOT true and must never be implied.** Membership is participation. It is not
certification, not conformance, and puts nothing on the trust list — that is the separate
**Conformance Program**, which requires evaluation and legal onboarding and ends in a public
listing. C2PA's own FAQ is explicit that a product "must undergo evaluation and legal
onboarding through the C2PA Conformance Program". We have a drafted validator application
(`docs/C2PA_VALIDATOR_APPLICATION.md`) and are **not listed**. The banned wordings are
enforced by `tests/test_forbidden_claims.py`, not by anyone remembering this paragraph.

**Membership is NOT a prerequisite for conformance.** Checked, because a search result claimed
the opposite and it was wrong: per the C2PA FAQ, *"You can implement the C2PA specification
without joining the organization."* So this news does not unblock the validator application —
that has only ever been blocked on owner sign-off.

**🔴 The public claim is HELD, and this is the important part.** As of 2026-09-09 **Aethyia does
not appear on c2pa.org's public member list.** Confirmed by fetching the raw page and grepping
it, with Adobe / Truepic / DigiCert / Bloomberg as controls to prove the list was actually
searchable — the names are there, ours is not. Public listing normally lags confirmation.

Until we appear, "we are a C2PA contributor member" is a claim a reader **cannot check**. On
this site of all sites, that is the wrong kind of claim to make: a sceptic goes to c2pa.org,
does not find us, and concludes we inflated it — which costs more than the credential earns.
Publishing an unverifiable claim about verification is self-refuting.

**Onboarding began 2026-09-10** — C2PA confirmed the application is processed and started
Contributor onboarding (Linux Foundation account, then Slack / Groups.io / GitHub, up to two
representatives). **Re-checked the member page that same day: Aethyia still absent** (controls
Adobe / Truepic / DigiCert all present, so the page was searchable and the absence real).
**The claim therefore stays held.** Onboarding is not listing.

**Publish when:** Aethyia Inc. appears on https://c2pa.org/membership/ — then state the
membership and link the listing as its own proof. The detector-watch routine checks for this
on every run (see `routines/DETECTOR_BASELINE.md`), so nobody has to remember to look.

**Promote to a conformance claim only when:** the Administrator has actually listed Provenote
on the Conforming Products List. Not on application, not on membership, not on passing our own
tests.

### External constraints on what we may claim about watermarks

Some of what limits our watermark claims comes from C2PA task force participation. **Those
notes are internal and are not reproduced here**: C2PA discussions and materials are subject
to C2PA's IP and confidentiality policies and are not ours to publish. They live in
`docs/C2PA_TF_NOTES_PRIVATE.md`, which is deliberately excluded from the public repository.

What that leaves in this public register is the part that is genuinely ours: the claims we
make, the words we refuse to use, and evidence anyone can check on our own surfaces. C20 and
C21 below are guards of that kind, and they stand on our own conduct rather than on anyone
else's remarks.

## Corrections log

The cardinal rule is only real if corrections are visible. Every claim we shipped and
later found wrong gets a row here, permanently.

### CORR-009 — Blog 5 said a C2PA credential is "gone the moment the text is extracted" (2026-09-24) — ✅ FIXED before publication

**Caught three days before the post was due out** (scheduled 2026-09-27), while researching the
landscape after CORR-008. Not caught by a routine, a test or a reviewer.

**What we claimed.** Blog 5 `c2pa-for-writing` built its central contrast on this: *"A C2PA
manifest binds to the file, not to the words… it cannot follow a paragraph once it leaves that
document"*, and *"A C2PA credential rides with the file: robust while the file is intact, gone
the moment the text is extracted."* The pull quote made it a slogan: *"A watermark travels with
the words… a content credential travels with the file."*

**Why it is wrong.** Specification **2.4 Annex A.8** defines embedding a manifest into
*unstructured text itself*, using **Unicode variation selectors** interleaved with the
characters; **§9.2.4** carries the normative reference, and **A.9** does the same for structured
text. A credential built that way lives in the characters, not in a container around them. The
neat opposition the post rested on is not a property of C2PA; it is a property of the
**PDF** case we had generalised from. Verified at spec.c2pa.org on 2026-09-24, independently of
the survey that flagged it.

**What replaced it.** The PDF statement is kept, because it is true and concrete. The general
claim is gone, A.8 and A.9 are named, and the new sentence carries its own limit: whether such
a credential survives a given copy, paste or normalisation depends on whether the intervening
tools preserve those characters, **which we have not tested and do not assert**.

**Lesson, and it is the same one as CORR-008 two days earlier.** We took a true, checkable
statement about one format and let it carry an untested general claim about a standard. The
countable part was right both times. **Watch the sentence that generalises — it is doing work
the evidence has not paid for.**

**Guarded so it cannot come back:** `routines/manifest.json` now lists the three retracted
phrases under `must_not_contain` and requires "A.8" under `post_must_contain`, so
`tests/test_routine_sync.py` fails if the old framing returns or the correction is quietly
removed.

### CORR-008 — "Text watermarking is provider gated and not generally accessible" (2026-09-23) — ✅ WITHDRAWN 2026-09-24

**Corrected by a third party in private correspondence**, within minutes of reading it. We had
invited the correction and received it. **The correspondent is not named, at their request, and
that request was right**: a private exchange is not ours to publish or to attribute. The
correction is recorded because our conduct is ours to record; the other person's is not.

**What we claimed**, in correspondence on 2026-09-23 arguing that of the three provenance
pillars only metadata is available for written work: *"Text watermarking exists but is provider
gated and not generally accessible."*

**Why it is wrong.** Meta's **TextSeal** is open source — arXiv 2605.12456, code at
`github.com/facebookresearch/textseal` — and it does **post-hoc** watermarking through LLM
rephrasing, not only generation-time watermarking by the model provider. Watermarking by
inserting invisible Unicode variation selectors was also raised, and C2PA specification 2.4
Annex A.8 defines exactly that technique for unstructured text. Post-hoc text watermarking
does not require a provider's cooperation at all, which is precisely what "provider gated"
denied.

**The fingerprinting half of the same argument also failed.** We said a perceptual hash
has no analogue for text because paraphrase destroys it. The answer put to us: compute a gist of the
text, store it in the manifest or a link to it, and compare the asset to the gist **to
validate the lookup** rather than to identify the work. Paraphrase resistance is then not the
requirement. The objection was to a job we had assumed the pillar must do.

**Where it appeared, and where it did not.** Correspondence only. It is **not** on any
Provenote surface and **not** in this register's live rows — checked across the site and the
published docs on 2026-09-24. The unsent letter carrying it was corrected. No blog, page or
public claim has to be retracted.

**What survives.** The narrower claim still stands, and is now on firmer evidence than when we
made it: **no conformant product declares any text media type.** Re-counted 2026-09-24 by
parsing `containers.generate` / `containers.validate` across all 219 products — zero non-empty
text arrays. Yesterday's count searched for the string `text/plain` and found none, which was
the right answer reached by the wrong method: the schema uses `textHtml`, `textUnstructured`
and `textStructured`. **A search that matches nothing is not evidence of absence.** Encypher
Corporation, named to us as the leading text implementer, is itself on the list at spec 2.2
declaring image, video, audio and `application/pdf` — and no text.

**Lesson.** Our argument bundled two very different statements: a checkable fact about a
published list, and a general assertion about what technology is available. The first was
sound. The second was an impression, and it reached a domain expert inside a week. **State the
countable thing; do not let it carry an uncountable one alongside it.**

### CORR-007 — Blog 5 claimed a content credential "survives editing" (2026-09-16) — ✅ FIXED 2026-09-17

**Found while substantiating C20/C21**, by grepping our own surfaces for robustness language
rather than trusting the assumption that we carry none. We do carry one.

**What we claim.** Blog 5 `c2pa-for-writing` (scheduled **2026-09-27**, 11 days out) carries a
pull quote: *"A watermark travels with the words and survives a copy-paste. A content
credential travels with the file and **survives editing**. They fail in opposite directions —
which is why you read both."*

**Why the second half does not hold.** A C2PA manifest survives editing **only by a
C2PA-aware tool that re-signs and records the edit as an action or ingredient**. Edited by a
tool that is not C2PA-aware, the manifest is stripped or its hard binding breaks — which is
the entire reason soft bindings exist, and exactly what Sony and TVU demonstrated as the
"sad pass" case noted internally (manifest stripped during conversion, then
recovered from a watermark). The post's own body already states the accurate version four
lines later — *"robust while the file is intact, gone the moment the text is extracted"* —
so the quote contradicts the paragraph it sits beside.

**The other two robustness statements in that post are sound** and stay: "a watermark
survives a copy-paste" is true by construction, since a text watermark *is* the token
sequence; and "fragile to heavy rewriting" is the correctly hedged direction, independently
supported by `c2pa-org/softbinding-algorithm-list` **PR #60**, where a text fingerprint
measured **0.99 balanced accuracy under length-preserving synonym substitution and 0.52 —
chance — under substantive lexical paraphrase**.

**Applied 2026-09-17** (owner-approved). The quote now reads: *"A watermark travels with the
words and survives a copy-paste. A content credential travels with the file — and only as far
as the next tool that knows to carry it. They fail in opposite directions, which is why you
read both."* Site rebuilt, `qa.mjs` clean across 9 pages.

**Note the shape.** This is the same failure as PR #60 — a prose robustness claim that
overstates what is measured — found in our own copy in the same week we recorded that failure
happening to someone else. Nothing about writing #61 into this register protected us from it;
the grep did. **And it is now a test, not a paragraph.** `docs/retracted_claims.json` gained a second rule,
`WATERMARK-ROBUSTNESS-2026-09-16`, guarding three overclaims in public copy: *tamper-proof*
(C6 says tamper-**evident**, which is weaker and true), *indelible / unremovable / cannot be
removed* (C21), and *survives editing / transcoding / conversion* unqualified (this
correction). Each carries positive controls, and the negative controls prove the guard leaves
the true statements alone — "survives a copy-paste", "survives editing **by a C2PA-aware
tool**", "tamper-evident", and our standing refusal to build a remover all still pass.
**Exercised by mutation:** the retracted sentence was re-injected into `web/build.mjs` and
watched failing on the exact match before being removed again. Suite 144 → **157 passed, 0
skipped**.

Blog 5's routine anchors (`post_must_contain`,
`must_not_contain` in `routines/manifest.json`) do not touch this sentence, so the reword does
not desynchronise the routines — but blog 5 is the post that drifted from its routine once
before (FIX-002), so re-run `scripts/check_routines_live.py` after any edit to it.

### FIX-003 — The conformance harness was running none of the official vectors (2026-09-16)

**Not a correction to a public claim — a repair of the evidence under one, found before the
claim shipped.** Blog 5 is scheduled 2026-09-27; its row asserting that Provenote validates
real C2PA manifests *against the official trust list* cited `scripts/c2pa_public_vectors.py`,
which sets `verify_trust=false` by design. The cited evidence could not support the claim as
worded.

Looking for better evidence found worse news. `scripts/c2pa_conformance.py` — the one place
that loads the official trust list — read `expected.json` from `tests/c2pa_vectors/`, while
the official vectors have always lived in `tests/c2pa_vectors/legacy-1.4/`. So `_dir_vectors()`
returned `[]` on every run since it was written, the harness ran its 4 self-generated vectors,
printed "4/4 passed" and exited 0. The docs read "4/4 generated vectors" and nobody asked why
it was never 13.

Three failures, one shape — **a step that reported a success it never verified**:

1. The harness could not distinguish "the official vectors all matched" from "no official
   vectors were loaded": both exit 0. `tests/test_c2pa_conformance.py` asserted only
   `main() == 0`, so it could not tell either.
2. Every assertion was on `outcome`, and `outcome` is the one field that does not depend on
   trust — the adapter returns `CREDENTIAL_VALID` for both `Trusted` and `Valid`, separating
   them by weight/confidence/caveat. **Deleting the official trust list would not have failed
   a single test.** Verified by running the official vectors with the list removed: identical
   results.
3. A declared-but-unfetched vector was silently dropped, turning a partial download into a
   smaller green run.

Fixed: recursive `expected.json` discovery (13/13, 9/9 declared file vectors); the harness
returns counts and exits non-zero if no file vectors loaded at all; the test asserts the
counts, fails on a partial fetch, and skips only on a clean "nothing fetched"; and
`test_trust_list_is_load_bearing` is the missing negative control. Both mutations that would
hollow it out — equalised weights, `verify_trust` defaulted to false — were confirmed to fail
it. Suite 143 → 144, still 0 skipped.

**This was live in the field at the same time:**
issue #61 on `softbinding-algorithm-list` exists because PR #60 measured a contributor's own
robustness claim and it did not hold. Ours was one directory level away from the same
outcome.

### FIX-001 — The signer and the verifier now actually agree (2026-08-30)

**Not a correction to a public claim — a repair of the thing underneath one.**

`web/api/verify.js` has always verified **Ed25519** signatures over canonical JSON.
`ledger.py`, which actually signs certificates, used **HMAC-SHA256** — a symmetric scheme that
cannot be verified with a public key at all. The two halves of "anyone can re-check this"
were different algorithms. **No certificate we could have issued was ever re-checkable**,
which is exactly what the site promised.

**What changed.**
- `ledger.py` now signs with **Ed25519** over the canonical JSON of the payload.
- New `provenance_verify/canonical.py` mirrors the JavaScript `canonical()` exactly —
  including the trap that JavaScript has one number type, so `1.0` must serialise as `"1"`
  and not `"1.0"`, or every signature fails.
- `api.py` takes `PROVENOTE_CERT_SIGNING_KEY_PEM`; with no key it generates an **ephemeral**
  one and warns that the resulting certificates are unverifiable by anyone — the correct
  failure for a dev default. The old `signing_secret()` now raises rather than silently
  signing something nobody can check.
- The dev keypair lives in `.keys/` (**gitignored**); `verify.js`'s baked dev public key was
  updated to its public half, so the round trip works end to end in development.

**Proof, not assertion.** `tests/test_cert_signing.py` extracts `canonical()` **from the
shipped `verify.js`** and runs it under Node, asserting that Python and JavaScript produce
byte-identical output across unicode, escapes, nesting and number-format edge cases. It then
signs a certificate in Python and verifies it with `crypto.verify` using the same code path
the endpoint uses — and asserts that altering one field breaks it.

**What is still not done.** Key custody. The private key currently exists as a file or an
environment variable; it must live in a KMS or HSM before real certificates are issued, so
the private half never exists outside it. That is the remaining step, and it is why C6 is
"substantiated for the scheme" rather than fully substantiated.

Full suite: **41 passed, 0 failed** — including a stale `test_c2pa` assertion, unrelated to
this work, which had been failing all day because it assumed the C2PA SDK was *not* installed.

### CORR-006 — The two headline detector numbers were not traceable (2026-08-30)

**The audit.** Every number on the public site was traced back to a named source, a tested
population and a date. Two of the most prominent did not survive.

**"Detectors claiming 99% deliver 66–92%"** — carried on the homepage as a stat tile, in the
`/reliability` page heading, and as a table row. Two problems:
- The 12-tool comparison behind it is **published by a company that sells its own AI
  detector, and which ranks itself first in that comparison.** That is a vendor's competitive
  content, not independent testing, and we were describing it as independent.
- It reports roughly **60–80%**. The figure "66–92%" does not match what the source says.

**"≈1 in 20 honest writers wrongly flagged"** — homepage stat tile and `/reliability` row.
Sourced in the register to *"independent 2026 detector reports"*, which is a gesture, not a
citation. The one comparison behind it tested false positives on **five human texts**.

**The uncomfortable part.** The best academic evidence we have — Jabarian & Imas, University
of Chicago Booth (2025) — points the *other* way: one detector achieved **99.8–100% accuracy
with false positives at or below 0.5%**, the only tool meeting a strict cap without losing
accuracy. We were citing that study on the same page as "the category delivers 66–92%", while
it shows the category claim is wrong. Some tools really are close to their claims.

**What replaced them.** Both homepage tiles now carry **first-party** numbers — a university's
own arithmetic (~750 of ~75,000 papers, using the vendor's *advertised* 1% rate) and OpenAI's
own reported 9% mislabelling on its own retired classifier. `/reliability` now says accuracy
and false-positive rates **vary sharply by tool**, names the study, and tells the reader to
ask for the rate measured on text like theirs. Blog 3, blog 6 and the FAQ were corrected to
match.

**Why the new position is stronger.** "All detectors are bad" was never true and we could not
support it. "Performance is a property of the specific tool, and almost nobody measures the
one they use" is true, defensible, and is *precisely* the argument for what we sell.

**Root cause, and it is the same one twice.** CORR-002 was over-generalising a single study
across time. This is over-generalising a vendor's comparison across a category. Both took a
narrow finding and made it sound like a law.

**Process change.** No number ships without: the **named** source, whether that source is
**independent of the products it evaluates**, the **population and sample size** tested, the
**date**, and a **revalidation date**. Vendor-published comparisons are not independent
evidence and may not be cited as such.

### FIX-002 — Blog 5's routines were still checking the retracted claim (2026-09-05)

**Found by an audit, not by the pipeline.** CORR-003 corrected blog 5 on 2026-08-31 — C2PA
text support is 2.3/2.4, not 2.2. **The post was fixed; its two routines were not.** Both
still instructed the checker to verify *"C2PA 2.2 (2025) added manifest embedding for
text/document formats"* — a claim the post no longer makes.

**Why this would not have been caught on the night.** The prompts carried an escape hatch —
*"a newer spec version is fine as long as text/document embedding is still supported"* — so on
27 September the publisher would most likely have **passed**, verified the wrong claim, and
published without ever checking what the post actually says. A silent false pass is worse than
a hold: a hold is visible.

**The rule that already existed and was not followed.** `CLAUDE.md`: *"When a post's copy
changes, its routine prompt must change too."* It is there because blog 4's routines had the
identical defect on 2026-08-30. **The same mistake, twice, by the author of the rule.**

**Fixed 2026-09-05.** Both blog 5 routines now verify the corrected 2.3/2.4 staging and the
`c2pa.ai-disclosure` assertion, name CORR-003 so the retracted framing cannot be reintroduced,
and additionally check that the post has not drifted into implying we can *embed* a credential
into text — which our SDK cannot do (`tests/test_c2pa_text_support.py`).

**Process change:** correcting a scheduled post is not complete until its routines are
re-read. Not updated — **re-read**, because the failure mode is a prompt that still passes
while checking something else. Blog 4, 6 and 7 routines were re-read on 2026-09-05 and match
their posts.

### CORR-002 — Detector bias against non-native writers, stated as current fact (2026-08-30)

**What we claimed.** That statistical detectors are biased against non-native English writers,
in the **present tense and as a general property**. Live in blog 3 (*"A detector that punishes
people for writing in their second language is not 99% anything"*), in the then-scheduled blog
6 (*"It does not fall randomly"*), and as a `/reliability` row reading ">50% misclassified"
with no date qualifier.

**What is actually supportable.** Liang et al., *Patterns* (2023) found >50% of TOEFL essays by
non-native writers misclassified — **by the detectors tested at that time**. A February 2026
preprint, *"Different Time, Different Language: Revisiting the Bias Against Non-Native Speakers
in GPT Detectors"* (Al Ali, Helcl, Libovický, arXiv:2602.05769), **retested three detector
families and found no systematic bias**, noting contemporary detectors no longer rely on
perplexity as earlier ones did.

**Why it qualifies rather than overturns.** The 2026 work tested **Czech**, not English/TOEFL,
and is a **preprint**, not peer-reviewed. Both facts are stated wherever we now cite it.

**How it was caught.** An external marketing review flagged it; we then verified the preprint
directly. This is the **second** time in one day that an outside check found us presenting a
stale claim as current reality (see CORR-001).

**Fixes applied 2026-08-30.** Blog 3, blog 6 and the `/reliability` row now present bias as
**a per-tool, per-population, per-moment empirical question** rather than a settled fact in
either direction — which is a more useful claim and the one we can actually defend. The 2026
preprint is cited in both posts.

**Process change.** Any claim resting on a single study now carries the study's **date and
tested population** in the copy itself, and gets a **revalidation date**. A finding about
"detectors" in 2023 is not a finding about detectors in 2026.

### CORR-003 — C2PA text-support version attribution wrong (2026-08-30)

**What we claimed.** Blog 5 (scheduled 27 Sep) said *"As of the 2.2 specification (2025)"* a
C2PA manifest can be embedded into PDF, Office/OOXML, ODF, EPUB, **HTML and structured or
unstructured text**. The agency brief also carried *"text is the gap"* in C2PA positioning.

**What is true**, verified against the specification's own changelog:
- **2.3 (December 2025)** — "comprehensive support for embedding C2PA manifests in
  **unstructured text files**".
- **2.4 (April 2026)** — "embedding C2PA Manifests into **HTML documents**" and into
  "**structured text formats** (source code, YAML, Markdown, AsciiDoc, etc.)", plus a new
  **AI Disclosure Assertion** (`c2pa.ai-disclosure`).

So the capability was attributed to the wrong version, and "text is the gap" was **factually
wrong** by the time we wrote it — text is covered; what remains emerging is adoption and
interoperability.

**Fixes applied 2026-08-30.** Blog 5 corrected to the 2.3/2.4 staging, source updated to the
2.4 spec, and the `c2pa.ai-disclosure` assertion added as a substantive point. Agency brief
corrected.

### CORR-004 — Three indefensible public claims (2026-08-30)

Removed from the live site, all flagged by an external review against the owner's rule that we
never make a claim we could not defend in any court, globally:

1. **`/pricing`** — a feature bullet reading **"Prove you wrote it"**, which directly
   contradicted our own FAQ (*"Can you prove a human wrote something? No"*). An internal
   contradiction on our own site. Now: *"Document how the work developed — a signed chain of
   drafts."*
2. **Homepage** — *"we're **the only ones** who tell you how much to trust the score you're
   paying for"*: an unqualified worldwide uniqueness claim requiring us to continuously prove
   a negative about every competitor. Now states what we do, without the superlative.
3. **Homepage** — *"No provider can verify a rival's watermark. **We can.**"*: an absolute
   market claim combined with an unverified live capability. Now: *"Built to read provenance
   signals across providers, not to favour one — subject to each provider's detection
   access."*

**Note:** claim 3 had also been repeated as a headline strength in the agency brief. Corrected
there too.

### CORR-005 — Anthropic watermark scope understated (2026-08-30)

Blog 2 reported the Claude watermark and the forthcoming detection API correctly but omitted
three limits **Anthropic states itself**: it applies to **future** Claude models with older
models covered over following months; it "doesn't say anything about ownership or authorship";
and it "cannot distinguish 'Claude wrote this' from 'Claude heavily edited this'". All three
now appear in the post. A provider being that precise about what its own signal does not prove
is the standard we should be citing, not softening.

### CORR-001 — EU AI Act Article 50 effective date (2026-08-30)

**What we claimed.** That the Article 50 "marking & detection duties" / "transparency
duties" *take effect on 2 December 2026*. This appeared on the homepage "Why now" stat
tile, on `/security.html`, in the body and source list of blog 2
(`what-the-claude-watermark-is`, published 2026-08-16), in blog 3's closing teaser, and
as the entire premise of the then-unpublished blog 4 (`eu-ai-act-article-50`), which
additionally claimed the date had been "pushed" from August to December.

**What is actually true.** Article 50's transparency duties have applied since
**2 August 2026** (AI Act Art. 113). The **December 2026** date is a narrower grace
period — introduced by the EU's Digital Omnibus on AI — for the machine-readable
*marking* duty under Art. 50(2), and only for generative AI systems already placed on
the EEA market **before** 2 August 2026. Systems placed on the market on or after
2 August 2026 have had no grace period at all.

**Sources (re-fetched 2026-08-30).** `artificialintelligenceact.eu/article/50/`
("Article 50 comes into force 2 August 2026", per Art. 113); European Commission,
*Quick facts: transparency rules for AI systems*, `digital-strategy.ec.europa.eu`
("These transparency rules apply from 2 August 2026" · "Grace period for marking
obligation until December 2026 for generative AI systems placed on the market before
2 August 2026").

**How it was caught.** The scheduled T-3h pre-release fact-check and the T-0 publisher
for blog 3 both re-verified the claim, both failed it, and the publisher **held the
post** — no commit, no push, exactly as designed. Because the Gmail connector token had
expired, the alerts fell back to GitHub issues #5 and #6 instead of email. The protocol
worked; the notification channel did not.

**Root cause.** Note that `docs/strategy.md` and register row C10 had always scoped this
*correctly* and narrowly ("Art. 50(2) transitional deadline for existing providers").
The error was introduced when that nuance was flattened into marketing and blog copy.
The register was right and the prose drifted from it.

**Fixes applied 2026-08-30.** All six surfaces corrected; blog 4 rewritten on the
correct premise and now states the correction in-post; blog 3 published with a corrected
teaser; C10 rewritten; `docs/MARKETING.md` wedge #2 and the `providers.py` docstring
corrected.

**Process change.** Blog and marketing copy must restate the register's *scope*, not a
shortened version of it. Where a claim has a qualifier ("for existing providers", "under
Art. 50(2)"), the qualifier ships with the claim or the claim does not ship.
