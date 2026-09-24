# Certificate signing key — how to store it properly

**Version 1.0 · 2026-08-30 · Owner: Deepak Chandran**

## What the key is, in one paragraph

Every Provenote certificate is signed with an **Ed25519 private key**. The matching **public**
key is published, so anyone can re-check a certificate without trusting us. That only works if
**exactly one** private key exists and nobody can copy it. If the private key leaks, anyone can
forge a Provenote certificate, and every certificate ever issued becomes worthless — including
ones already relied on. This is the single most sensitive secret the company will hold.

## The rule

> **The private key must be created inside a KMS and must never leave it.**
> We do not generate it on a laptop and upload it. We ask AWS to generate it, and from then on
> we can only ask AWS to *sign things* — we can never read the key back. Not even you can.

That property is what makes a leak structurally impossible rather than merely unlikely.

## Where it will live

**AWS KMS**, in the **Aethyia** organisation, in a **dedicated Provenote account** (not shared
with another product — see the divestibility rule in `PAYMENTS.md` §4). KMS supports Ed25519
(`ECC_ED25519`) with `KeyUsage=SIGN_VERIFY`.

Cost is trivial: about **$1/month** per key plus a fraction of a cent per thousand signatures.

## Steps — when the AWS account exists

Do these in order. Steps 1–5 are one sitting.

1. **Create the key, inside KMS.** Note `Origin = AWS_KMS`: the private half is generated in
   the HSM and is not exportable, by construction.
   ```
   aws kms create-key \
     --key-spec ECC_ED25519 \
     --key-usage SIGN_VERIFY \
     --origin AWS_KMS \
     --description "Provenote certificate signing key (Ed25519)"
   ```
2. **Give it a stable alias**, so the key can be rotated later without changing code:
   ```
   aws kms create-alias --alias-name alias/provenote-cert-signing --target-key-id <KEY_ID>
   ```
3. **Lock the key policy down.** Only the signing role may `kms:Sign`. Nobody gets
   `kms:ScheduleKeyDeletion` except a break-glass admin. **Enable key rotation is NOT applicable
   to asymmetric keys** — rotation is manual and deliberate (see below).
4. **Export the PUBLIC half** (this one is safe to publish):
   ```
   aws kms get-public-key --key-id alias/provenote-cert-signing --output text --query PublicKey \
     | base64 -d | openssl pkey -pubin -inform DER -out provenote-cert-public.pem
   ```
5. **Publish the public key**: set `PROVENOTE_CERT_PUBKEY_PEM` in Vercel production, redeploy,
   and confirm `GET /api/verify` now reports `key_source: "configured"` instead of
   `"dev-placeholder"`. Also pin it in `docs/VERIFIABLE_CERTIFICATES.md` so there is a written
   record of which key was live from which date.
6. **Point the signer at KMS.** `ledger.py` signs over canonical JSON; the only change is that
   the signing call becomes a `kms:Sign` request instead of a local key operation. The payload
   and the canonicalisation do not change, which is why the cross-language tests keep working.
7. **Prove it end to end** before issuing anything real: sign a test certificate via KMS and
   verify it through `/api/verify`. Do not skip this — a signature that verifies locally but
   not through the public endpoint is the exact failure this whole design exists to prevent.

## Until then

The current dev key lives in `.keys/` and is **gitignored**. It is fine for development and
tests. **No certificate signed with it may be given to anyone as evidence**, and the site must
keep saying `dev-placeholder`.

If no key is configured, `api.py` generates an **ephemeral** one and warns loudly that the
resulting certificates cannot be verified by anyone. That is deliberate: a loud useless
signature is safer than a quiet one that looks real.

## Rotation and the thing people forget

You will eventually need a second key. **Certificates signed with the old key must still
verify after rotation** — otherwise you have retroactively invalidated evidence that people may
be relying on, which is the worst possible failure for this product.

So: publish keys as a **set with validity dates**, and record the key ID inside each
certificate. Retire a key from *signing* without retiring it from *verification*. This needs a
small design change before the first rotation, not after — it is on the roadmap in
`VERIFIABLE_CERTIFICATES.md`.

## If the key is ever suspected compromised

1. Disable `kms:Sign` on the key immediately — stop new forgeries first.
2. Do **not** delete the key: you still need it to verify legitimately-issued certificates.
3. Publish a dated notice of which key is affected and from when.
4. Issue a new key, publish it, and re-sign anything still in active use.
5. Follow `INCIDENT_RESPONSE.md` for the rest.
