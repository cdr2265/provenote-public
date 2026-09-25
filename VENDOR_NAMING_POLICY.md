# Vendor Naming Policy — Provenote

**Version 1.1 · Effective 2026-08-30 · Owner: Deepak Chandran (Security/Brand Owner)**

## The rule this serves

> **"We stay safe and neutral. We will never make any claim we cannot defend — including in
> any court, globally."** — DC, 2026-08-30

This extends the cardinal rule (*Provenote will not claim anything that cannot be verified*)
from **factual** defensibility to **legal** defensibility. A claim can be perfectly true and
still be actionable somewhere in the world. Comparative-advertising and trade-libel rules
vary sharply by jurisdiction.

## Two kinds of reference — the distinction that actually tracks the risk

Not all naming is equal. The legal exposure in naming a company lives almost entirely in
**disparagement** — placing their name beside a claim that their product is bad. Saying you
read someone's standard carries no such risk. So the policy turns on *which kind* of
reference it is, not on whether a name appears.

**A. ADVERSARIAL reference — never permitted on public pages.**
Naming a product alongside a criticism, a failure, an accuracy measurement, a
marketing-versus-reality comparison, or any figure we derived ourselves. This is where trade
libel and comparative-advertising rules bite, and those rules vary sharply by jurisdiction.
*Example of what we will not publish:* "Detector X claims 99% but independent testing measured
66%."

**B. INTEGRATION reference — permitted and encouraged.**
Naming a standard, watermark, credential, protocol, or provider **whose signal we read**.
This is factual, neutral-to-positive, and necessary: the product is not describable without
it. "We read the SynthID watermark and C2PA Content Credentials" *is* the value proposition;
"we read certain provenance signals" says nothing. Nobody is harmed by us saying we support
their standard.

**Currently named under (B), deliberately:** SynthID (Google DeepMind), C2PA / Content
Credentials, Anthropic's Claude watermark, and OpenAI where it concerns their own published
statement about their own product. These stay.

**The boundary case to watch:** a factual statement about another company's product *status*
— e.g. "Claude watermark — we have no access to detection." That is an integration reference
and is fine, but it goes stale when they ship. The `provenote-detector-watch` routine polls
for exactly that every three hours and emails the owner, so the dependency is monitored
rather than assumed.

**And this example went stale itself, which is the point.** It used to read *"detection API
not yet live"* — true when written, false from **2026-09-01**, when detection entered private
preview. **CORR-005.** The watch routine caught the change in 3.5 hours; what failed was
propagation, and this file — the file that predicted the staleness — was one of the places it
did not reach. **A sentence about an absence is still a claim about status, and goes stale
the same way.** Prefer a statement about *our own* position ("we have no access") over one
about the vendor's roadmap, because ours is one we control and can verify.

**It went stale on 2026-09-01, and monitoring was not enough.** The routine fired within 3.5
hours and the copy sweep that followed missed the homepage bullet carrying this exact
sentence, which then ran false for a week (see CLAIMS_REGISTER, "One surface was missed").
Detecting the change is the easy half; propagating it is the half that failed. When this
routine fires, grep the repo for the *old status wording* — not for the vendor's name.

## The policy

1. **No competitor or vendor is named on any public page.** Home, reliability, pricing,
   report, methodology, principles, security, FAQ, about, news, terms, privacy.
2. **No vendor is named in internal docs, code, comments, or tests either** (applied
   2026-08-30). Earlier revisions in Git history retain the original names if the mapping is
   ever needed.
3. **Names are permitted in blog and help content ONLY when both hold:**
   - the reference is either (a) the vendor's **own published statement about its own
     product**, or (b) a citation to a **named, published study**; and
   - the claim is stated **exactly as the source states it** and is logged in
     `CLAIMS_REGISTER.md`.
4. **Never place a vendor's name beside a measurement we derived ourselves.**
5. **Never attribute an example, sample, or illustrative score to a named vendor.**
6. **Adversarial references (type A) are never permitted on a public page.** No product name
   beside a criticism, failure, accuracy figure, or marketing-vs-reality comparison.
7. **Integration references (type B) are permitted and encouraged.** Name the standards,
   watermarks and credentials we read — it is factual, carries no disparagement, and the
   product cannot be described without it. Say plainly *that* we read them; never imply an
   endorsement by them of us.
8. **Pseudonyms are not a loophole.** "Detector C — marketed 99.12%, measured 66%" identifies
   a company as surely as naming it. Where a figure or anecdote uniquely identifies a vendor,
   either drop the identifying detail or drop the claim. Prefer **category-level** claims
   ("vendors market near-perfect accuracy; independent testing reports materially lower")
   which identify no one. **The example deliberately carries no figure.** It used to read
   "vendors market 99%+; independent testing lands at 66–92%" — **CORR-006 retracted that
   measured half**, which rested on a vendor-published comparison presented as independent.
   Anonymity and accuracy are separate tests, and this file only ever enforced the first:
   a category-level claim identifies no one **and** still has to be true.

## Why this is also better positioning

Our argument was never "these three companies lie." It is "**this whole category
over-promises, and the cost lands on innocent people.**" Category-level evidence states that
more accurately, is harder to attack, and is consistent with being the neutral layer. A
verification company that attacks named rivals is not visibly neutral.

## What changed on 2026-08-30

- `/reliability` — the four-vendor comparison table became a **marketed vs. independently
  measured** table by metric. All figures and sources retained; no vendor named.
- Home — "Works with" pills and the bring-your-own-detector copy de-named.
- `/report` (sample) — named rows became "Commercial detector A/B"; removed an **invented
  score attributed to a named vendor**, which was the single highest-risk item found.
- Internal — `docs/strategy.md`, `docs/strategy.html`, `README.md`, `docs/unit_economics.mjs`,
  `provenance_verify/signals/statistical.py`, `provenance_verify/benchmark/harness.py`,
  `tests/test_report.py`, `tests/test_http_detector.py`.
- Retained by exception under rule 3: **OpenAI's own 2023 statement retiring its own AI Text
  Classifier** (blog 3) and the **Liang et al., *Patterns* (2023)** citation. Both are a
  party's own admission or a named academic study, neither disparages a current commercial
  product, and both are logged in the register.

## Before publishing any new public copy

- [ ] Does it name a vendor? → remove it, unless rule 3 applies.
- [ ] Does a figure or anecdote **uniquely identify** a vendor? → generalise or drop.
- [ ] Does it assert admissibility, compliance, or legal effect? → reframe as
      "compliance-supporting… not legal advice."
- [ ] Does it render a verdict about a person ("convict", "proves cheating", "clears you")?
      → rewrite. We produce evidence and stated uncertainty, never findings about people.
- [ ] Does it imply a capability without its precondition? → state the precondition.
- [ ] Is every factual claim logged in `CLAIMS_REGISTER.md`?
