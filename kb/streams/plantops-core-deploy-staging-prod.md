# PlantOps Core — STAGING / PROD Deploy Runbook

**Status:** CANONICAL for `plantops/plantops-terminal`  
**Scope:** KISS deployment only.  
**Parent pattern:** `kb/streams/github-tailscale-artifact-deployment.md`

## Decision

Reuse the existing GitHub → Tailscale OIDC configuration already proven by BeltRisk.

For PlantOps Core, do **not** create another Tailscale long-lived secret by default.

Reuse these existing GitHub Environment secret names:

```text
TS_OAUTH_CLIENT_ID
TS_AUDIENCE
```

Only provide/change the SSH deployment key for PlantOps Core:

```text
PROD_SSH_KEY
```

If the existing Tailscale OIDC identity/tag cannot reach the PlantOps target nodes, change only the Tailnet ACL/tag scope. Do not create another OAuth/OIDC credential unless isolation is actually required.

If a writable Tailscale connector becomes available to ChatGPT, prefer configuring/testing the Tailnet path directly instead of asking the operator to copy secrets manually.

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

Image:

```text
ghcr.io/plantops/plantops-terminal:sha-<commit12>
```

The exact same immutable image/digest must be used for STAGING and PROD.

## STAGING

Target:

```text
host: NUC
root: /srv/staging/cfc/plantops-core
port: 8481
database: plantops_core_stg
```

Purpose:

```text
apply schema + deterministic master seed
stage/replay real historical PLAN + QC + FMRP data
run health checks
inspect migration receipt
allow read-only MCP/data scouting
```

STAGING is where data/migration defects are found. Do not use toy fixtures as the main acceptance evidence.

## PROD

Target:

```text
host: inst4
root: /srv/prod/cfc/plantops-core
port: 8480
database: plantops_core_prod
```

PROD receives the exact artifact already verified on STAGING.

Do not automatically load historical migration staging evidence into PROD.

## GitHub Actions minimum

```yaml
permissions:
  contents: read
  packages: read
  id-token: write
```

Tailscale join follows the BeltRisk pattern:

```yaml
- name: Join plant tailnet
  uses: tailscale/github-action@v4
  with:
    oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
    audience: ${{ secrets.TS_AUDIENCE }}
    tags: <existing approved deploy tag>
```

SSH key follows the same existing secret name:

```yaml
env:
  PROD_SSH_KEY: ${{ secrets.PROD_SSH_KEY }}
```

No extra Tailscale auth key should be introduced unless OIDC cannot serve the target scope.

## Host deploy contract

Use the repository's existing bounded script:

```text
scripts/deploy-node.sh
```

It already separates:

```text
staging    → /srv/staging/cfc/plantops-core :8481
production → /srv/prod/cfc/plantops-core    :8480
```

Expected sequence:

```text
pull immutable image
→ apply migrations
→ seed master data
→ STAGING only: replay real history when provided
→ docker compose up --no-build
→ /healthz
→ /deployment.json
→ /migration-receipt
→ write deployment receipt
```

## KISS rules

1. Reuse proven Tailscale OIDC configuration.
2. Change only SSH key/target access when sufficient.
3. STAGING first; PROD only after STAGING passes.
4. Same artifact bytes on both nodes.
5. No build on nodes.
6. No public SSH for CI.
7. No Kubernetes/self-hosted runner/new control plane for this app.
8. Real PlantOps historical data is the primary migration test evidence.
9. If a Tailscale connector becomes writable, configure/test it directly rather than asking for manual secret handling.

## Recall phrase

When asked for **“PlantOps Core deploy”**, **“PlantOps staging/prod”**, or **“dùng lại TS OIDC như BeltRisk”**, use this runbook first.
