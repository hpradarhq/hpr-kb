# PlantOps Core — STAGING / PROD Deploy Runbook

**Status:** CANONICAL for `plantops/plantops-terminal`  
**Scope:** KISS deployment only.  
**Parent pattern:** `kb/streams/github-tailscale-artifact-deployment.md`

## Decision

Use GitHub Actions + Tailscale workload identity federation/OIDC. Do not add a reusable Tailscale auth key by default.

For PlantOps Core STAGING, GitHub Environment must be named exactly:

```text
staging
```

Environment secrets:

```text
TS_OAUTH_CLIENT_ID
TS_AUDIENCE
STAGING_SSH_KEY
```

The first two may reuse the BeltRisk federated credential **only if that Tailscale trust credential also accepts the PlantOps Core GitHub OIDC identity**.

For this repository, Tailscale reported and successfully verified the actual GitHub OIDC subject as the ID-qualified form:

```text
repo:plantops@312055198/plantops-terminal@1351582048:environment:staging
```

Use the **exact subject Tailscale reports in the token-exchange error**, not a hand-written name-only approximation such as `repo:plantops/plantops-terminal:environment:staging`.

If the Tailscale action receives Client ID/Audience but token exchange returns HTTP 403 Unauthorized, the GitHub secrets are already working. Fix the Tailscale Trust credential claim/repository/environment restriction to accept the exact reported subject. Do not rotate SSH keys or recreate GitHub secrets for that error.

If a writable Tailscale connector becomes available to ChatGPT, prefer configuring/testing this trust path directly instead of asking the operator to copy values manually.

## App topology

```text
GitHub main
   ↓
build/test once
   ↓
immutable multi-arch OCI image
   ↓
STAGING — NUC / amd64
   ↓
health + migration receipt + real-data scout
   ↓
manual approval
   ↓
PROD — inst4 / arm64
```

Repository:

```text
plantops/plantops-terminal
```

The exact same immutable image/digest must be used for STAGING and PROD.

## STAGING

```text
host: 100.100.40.89
Tailscale node name observed: proxmox
user: deploy
root: /srv/staging/cfc/plantops-core
port: 8481
database: plantops_core_stg
compose project: plantops-core-stg
```

Purpose:

```text
isolated PostgreSQL STG
apply schema + deterministic seed
stage/replay real historical PLAN + QC + FMRP data
run health checks
write deployment receipt
allow read-only scouting
```

STAGING is where data/migration defects are found. Real PlantOps history, not toy fixtures, is the acceptance evidence.

Current workflow:

```text
.github/workflows/deploy-staging.yml
```

Credential preflight must show only presence, never values:

```text
TS_OAUTH_CLIENT_ID=PRESENT
TS_AUDIENCE=PRESENT
STAGING_SSH_KEY=PRESENT
```

Then:

```text
GitHub OIDC
→ Tailscale token exchange
→ Tailscale reachability to node
→ SSH deploy@100.100.40.89
→ isolated Postgres STG
→ migrations + seed
→ PlantOps Core :8481
→ /healthz + /deployment.json + /migration-receipt
→ receipt
```

### Verified deployment — 2026-09-01

GitHub Actions run:

```text
33534790430 attempt 4
```

Result:

```text
OIDC token exchange PASS
Tailscale connectivity PASS
SSH deploy user PASS
compose install PASS
GHCR login PASS
isolated STG deployment PASS
workflow conclusion SUCCESS
```

Verified artifact:

```text
ghcr.io/plantops/plantops-terminal:sha-7c6469af4f8c
commit 7c6469af4f8c0783b9a00ddc9f58c8bc132f69b2
```

The workflow is fail-closed on `/healthz`, `/deployment.json`, `/migration-receipt`, and commit matching before it can report success. Historical workbook replay remains a separate next step; the successful infrastructure deployment does not claim historical rows have been loaded yet.

## PROD

```text
host: inst4
root: /srv/prod/cfc/plantops-core
port: 8480
database: plantops_core_prod
```

Do not focus on PROD until STAGING data/migration validation is green. PROD later receives the exact artifact already verified on STAGING. Do not automatically load historical migration staging evidence into PROD.

## KISS rules

1. STAGING first; ignore PROD until STAGING passes.
2. Build once; no source build on nodes.
3. Same immutable artifact for later PROD promotion.
4. OIDC over Tailscale; no public SSH for CI.
5. Dedicated STAGING SSH key: `STAGING_SSH_KEY`.
6. Reuse BeltRisk Tailscale credential only when its trust claims explicitly allow PlantOps Core.
7. HTTP 403 during JWT exchange means Tailscale trust mismatch, not SSH failure.
8. Trust the exact ID-qualified subject shown by Tailscale; do not infer a name-only subject.
9. No Kubernetes, permanent self-hosted runner, or extra deployment control plane for this app.
10. Use real PlantOps historical data as the migration acceptance evidence.

## Recall phrase

When asked for **“PlantOps Core deploy”**, **“PlantOps staging/prod”**, or **“dùng lại TS OIDC như BeltRisk”**, use this runbook first.
