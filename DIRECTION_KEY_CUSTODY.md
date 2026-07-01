# Direction: eCW FHIR key custody (Workstream B1)

Companion to the central roadmap
(`rapidmed-denial-orchestrator/ROADMAP_CENTRAL_2026-07.md`, same branch).

This repo holds the **public** JWKS (`jwks.json`, RSA-2048/RS384, kid
`115fec2d-b4eb-430e-9c12-99f073417e0a`) that eCW uses to verify our SMART
Backend Services JWTs. The **private** key is today distributed by USB to
every clinic machine at `~/Code/jwks/SECRET_DO_NOT_COMMIT/private_key.pem`.

## Decided 2026-07-01 (with the coding service's move to EC2)

1. **Private key moves to AWS Secrets Manager**
   (`rapidmed/ecw-fhir-private-key`); the EC2-hosted coding service reads it
   at boot via the instance role. After cutover + burn-in, the key is
   **deleted from every clinic machine** — custody shrinks to AWS + the eCW
   developer portal.
2. **Rotation gets a story** (none exists today): generate a new keypair,
   add the new public key to `jwks.json` **alongside** the old (a JWKS is a
   set — eCW selects by `kid`), update the registered JWKS with eCW, flip
   the signing key in Secrets Manager, then retire the old entry. Document
   this as a runbook in this repo; target an annual cadence or on any
   suspected exposure.
3. **Scope registration**: while in the eCW developer portal for rotation
   setup, register `system/ServiceRequest.read` (unblocks the ORDER
   VERIFICATION feature flag — see
   `rapidmed-v3-coding-service/DIRECTION_EC2_MIGRATION.md`). Remember eCW's
   token endpoint is all-or-nothing on scopes: verify a token mint with the
   expanded scope set before flipping any flags.

## Invariants
- The private key never enters git (the `SECRET_DO_NOT_COMMIT/` gitignore
  stands) and never enters the deploy tarball.
- One eCW client app, one active signing key at a time; overlap only during
  rotation.
