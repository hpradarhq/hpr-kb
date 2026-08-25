# GitHub–Tailscale Container Deployment

## Intent

Provide a small, repeatable deployment procedure for containerized industrial/internal applications using GitHub Cloud CI, GHCR immutable images, ephemeral Tailscale access, Docker Compose, short-lived same-host staging, and manually approved production promotion.

The goal is to keep deployment simple enough for small teams while preserving industrial safeguards: no source builds on production, no public inbound SSH, isolated staging data, immutable artifacts, database backup before production migration, health verification, rollback, and deployment receipts.

## Current Canonical State

The preferred deployment pattern for similar small apps is:

```text
feature/*
   ↓
GitHub PR
   ↓
Cloud CI / tests
   ↓
main
   ↓
BUILD ONCE
   ↓
GHCR immutable SHA image
   ↓
GitHub-hosted runner
   ↓
ephemeral Tailscale node
   ↓
private SSH to application host
   ↓
STAGING on same host, different Compose project + port
   ↓
smoke / readiness checks
   ↓
manual production approval
   ↓
PRODUCTION using the SAME image
```

A self-hosted Gitea, local CI runner, Kubernetes cluster, public SSH endpoint, or separate permanent staging server is not required for this class of application unless scale or governance later justifies it.

## When To Use This Pattern

Use this pattern when most of the following are true:

- Application is containerized and deployed with Docker Compose.
- Team is small and operational simplicity matters.
- GitHub is the canonical source repository.
- CI can run in GitHub Cloud.
- Production host already runs Tailscale or can join a tailnet.
- Production should not expose SSH publicly.
- A separate permanent staging host would be wasteful.
- Staging can share the physical host but must not share DB/storage/session state.
- Production deployments should use prebuilt immutable images.

Do not use this pattern unchanged when the application requires hard physical isolation, regulated segregation, very high availability, multiple independent production nodes, or complex orchestration.

## Architecture / Model

```text
                    INTERNET
                       │
              ┌────────▼─────────┐
              │ GitHub           │
              │ source / PR / CI │
              └────────┬─────────┘
                       │
                build immutable image
                       │
              ┌────────▼─────────┐
              │ GHCR             │
              │ sha-<git-sha>    │
              └────────┬─────────┘
                       │
            GitHub-hosted deployment job
                       │
             ephemeral Tailscale identity
                       │
                       ▼
              private tailnet transport
                       │
              ┌────────▼─────────┐
              │ Application host │
              │                  │
              │ PROD : P         │
              │ STG  : S         │
              └──────────────────┘
```

The GitHub runner is temporary. It joins the tailnet only for the deployment job, receives a narrowly scoped deploy tag, reaches only the target host/port allowed by Tailscale policy, performs the deployment, then disappears.

## Core Deployment Invariants

### Build once, promote the exact artifact

The application image is built in CI and pushed to GHCR using an immutable Git SHA tag:

```text
ghcr.io/<org>/<app>:sha-<7-char-sha>
```

The same immutable image is used for staging and production.

Never rebuild the application separately for production.

### No source build on production

Production/staging hosts should not need Go, Node, npm, compilers, source checkout, or package installation to deploy an application release.

Deployment uses:

```bash
docker compose pull app
docker compose up -d --no-build db app
```

### Private network access only

GitHub reaches the target host over Tailscale. Public inbound SSH/NAT forwarding is unnecessary.

Recommended authorization layers:

```text
GitHub OIDC identity
      ↓
Tailscale workload identity
      ↓
tag:<app>-deploy
      ↓
Tailscale ACL/grant → target-host:22 only
      ↓
SSH key
      ↓
dedicated Linux deploy user
```

Using `root` is acceptable only as a bootstrap shortcut. Mature deployments should use a dedicated `deploy` user with only the permissions required for that application.

### Production and staging data must never be shared

Same host does not mean same runtime state.

Use a different Compose project name for staging:

```bash
docker compose \
  -p <app>-staging \
  --env-file .env.staging \
  up -d --no-build
```

This provides separate:

- containers
- Docker network
- PostgreSQL volume
- application storage volume
- environment file
- database credentials
- session secret
- session cookie name

### Different cookie name is mandatory on same host

Browser cookies are scoped primarily by host/domain, not TCP port.

If production and staging use the same hostname/IP but different ports, use different cookie names, for example:

```text
production: app_session
staging:    app_staging_session
```

Otherwise logging into staging may overwrite or interfere with the production browser session.

## Standard Environments

### Test

Ephemeral GitHub-hosted environment.

Typical checks:

- frontend/static asset build
- formatting
- static analysis / vet
- race tests where applicable
- unit/integration tests
- PostgreSQL migration tests
- application build
- Compose validation

### Staging

Same physical application host as production, but isolated through a separate Compose project and port.

Recommended lifecycle:

- create on demand
- start with clean isolated DB/storage
- deploy exact immutable image
- run readiness and smoke tests
- keep for no more than 48 hours
- automatically tear down containers, network and staging volumes

For an hourly cleanup scheduler, an expiry marker around 47 hours leaves margin to stay within the intended 48-hour maximum.

### Production

Persistent deployment using the same image tested in staging.

Production should require a GitHub Environment approval gate before deployment.

## Generic Parameters

| Parameter | Example | Rule |
|---|---|---|
| `TARGET_HOST` | `100.x.y.z` or MagicDNS name | Tailscale address only |
| `TARGET_USER` | `deploy` | Dedicated user preferred |
| `DEPLOY_DIR` | `/srv/apps/myapp` | Contains compose/runtime files |
| `PROD_PORT` | `8124` | Persistent production port |
| `STAGING_PORT` | `8123` | Must differ from production |
| `STAGING_PROJECT` | `myapp-staging` | Dedicated Compose project |
| production env | `.env` | Persistent, host-local secret config |
| staging env | `.env.staging` | Generated/isolated, disposable |
| image | `ghcr.io/org/app:sha-abcdef1` | Immutable SHA tag |

## GitHub Configuration

### Required permissions

Deployment jobs normally need:

```yaml
permissions:
  contents: read
  packages: read
  id-token: write
```

The image-building workflow may need `packages: write`.

### Tailscale secrets

Prefer workload identity federation / OIDC rather than long-lived Tailscale auth keys.

Typical GitHub secrets:

```text
TS_OAUTH_CLIENT_ID
TS_AUDIENCE
```

The federated Tailscale identity should be constrained to the intended GitHub repository/workflow/environment and should assign only the deploy tag required for this application.

### SSH secret

Store the SSH private key only in GitHub Secrets/Environment Secrets:

```text
PROD_SSH_KEY
```

Install the matching public key in the deploy account's `authorized_keys` on the target host.

Never paste the private key into chat, issues, logs, repositories, Compose files, or `.env` committed to Git.

### GHCR authentication

Prefer the workflow's short-lived `GITHUB_TOKEN` when the repository/package permissions allow it.

Use a separate scoped GHCR pull token only when package ownership/cross-repository permissions require one.

## Tailscale Policy

Use an application-specific deploy tag, for example:

```text
tag:myapp-deploy
```

Grant only the minimum required path:

```text
tag:myapp-deploy → target-app-host:22
```

Do not grant the deployment identity broad access to:

- the entire LAN
- PLC networks
- SCADA networks
- databases directly
- unrelated servers
- hypervisor management unless explicitly required

The application host performs local Docker/database operations after SSH login; the GitHub runner does not need direct PostgreSQL access.

## Staging Procedure

### 1. Join the tailnet

The GitHub-hosted runner authenticates to Tailscale using federated identity and becomes an ephemeral tagged node.

### 2. Verify target reachability

Check:

```bash
tailscale ping "$TARGET_HOST"
ssh "$TARGET_USER@$TARGET_HOST" 'docker version && docker compose version'
```

Pin/verify the SSH host key before privileged operations. In a hardened setup, pre-provision the expected host key instead of dynamically trusting an unverified first response.

### 3. Validate port separation

Read the actual production host port and refuse staging if it collides with the selected staging port.

### 4. Create isolated staging runtime configuration

Generate fresh:

- PostgreSQL password
- session secret
- staging DB name/user
- staging cookie name

Example runtime values:

```text
APP_ENV=staging
APP_ENV_FILE=.env.staging
BELTRISK_PORT=<STAGING_PORT>
POSTGRES_DB=<app>_staging
SESSION_COOKIE_NAME=<app>_staging_session
SESSION_ABSOLUTE_TTL=48h
APP_RESTART_POLICY=no
DB_RESTART_POLICY=no
```

Keep `.env.staging` mode `0600` and never commit it.

### 5. Pull and start exact image

```bash
export BELTRISK_IMAGE="$IMAGE"
export BELTRISK_PORT="$STAGING_PORT"

docker compose \
  -p "$STAGING_PROJECT" \
  --env-file .env.staging \
  pull app

docker compose \
  -p "$STAGING_PROJECT" \
  --env-file .env.staging \
  up -d --no-build db app
```

Environment variable names may differ by application; the invariant is the same: exact immutable image, separate project, no build.

### 6. Verify staging

Minimum automated checks:

```text
/health/live
/health/ready
/login or equivalent public route
critical authenticated pages if a safe smoke identity exists
RBAC negative checks where practical
migration version/current schema
running image equals expected immutable image
```

A deployment is not successful merely because containers are `Up`.

### 7. Record staging receipt

Write a receipt containing at least:

```text
environment
deployed_at_utc
git_sha
image
image_digest if available
host
port
compose_project
health_status
expiry
actor/workflow
```

### 8. Set expiry

Record an expiry epoch/date. For an hourly cleanup job and a requested 48-hour maximum, use approximately 47 hours.

## Automatic Staging Cleanup

Run an hourly scheduled workflow.

If staging is expired:

```bash
docker compose \
  -p "$STAGING_PROJECT" \
  --env-file .env.staging \
  down -v --remove-orphans

rm -f .env.staging deployments/staging-expires-epoch
```

The `-v` is intentional because staging data is disposable and must not silently become a second long-lived environment.

If the expiry marker is invalid, fail safe and refuse blind teardown rather than guessing.

## Production Procedure

Production deployment occurs only after the selected image has passed CI and staging checks and the production GitHub Environment is approved.

### 1. Verify prerequisites

- target reachable over Tailscale
- Docker and Compose healthy
- disk space acceptable
- PostgreSQL healthy
- expected production port active/free as designed
- immutable image exists

### 2. Record previous image

```bash
previous_image="$(docker inspect \
  --format='{{.Config.Image}}' \
  "$(docker compose ps -q app)")"
```

### 3. Back up PostgreSQL

Before migrations/startup:

```bash
docker compose exec -T db \
  sh -c 'pg_dump -Fc -U "$POSTGRES_USER" "$POSTGRES_DB"' \
  > "backup/app-prod-<timestamp>.dump"
```

Verify the dump is non-empty.

### 4. Pull exact image

```bash
export APP_IMAGE="ghcr.io/<org>/<app>:sha-<sha>"
docker compose pull app
```

### 5. Start without building

```bash
docker compose up -d --no-build db app
```

### 6. Readiness and smoke

Require `/health/ready`, `/health/live`, and application-specific smoke checks.

### 7. Verify running image

The image reported by Docker must equal the intended immutable image reference/digest.

### 8. Write production receipt

Recommended fields:

```text
deployment_id
timestamp
git_sha
image
image_digest
db_migration_version
host
previous_image
health_status
actor
```

## Rollback Rule

Application rollback and database rollback are separate decisions.

If the new application image fails readiness:

```text
new app fails
    ↓
restore previous application image
    ↓
restart app
    ↓
verify readiness
```

Do not automatically restore the database backup merely because the application container failed.

Database restore should be an explicit recovery action because migrations may be forward-compatible and restoring a DB can destroy valid post-migration data.

Prefer expand/contract, backward-compatible migrations so application rollback remains possible without DB rollback.

## Security Rules / Invariants

- Never build application source on production.
- Never use `latest` as the promoted production artifact.
- Never promote by branch name alone when an immutable SHA/digest is available.
- Never share the production database volume with staging.
- Never share the production application storage volume with staging.
- Never reuse the production session cookie name for same-host staging.
- Never expose production SSH publicly only to satisfy CI/CD.
- Never store private SSH keys or Tailscale secrets in the repository.
- Never let arbitrary repository workflows automatically gain the deploy Tailscale identity.
- Never let staging cleanup imply production-deploy authorization; use separate enable gates.
- Prefer a dedicated deployment user over root once bootstrap is complete.
- Prefer GitHub Environment approval for production.
- Prefer immutable image digest verification in addition to SHA tags when practical.

## Recommended Enable Gates

Keep production and cleanup/deploy switches separate, for example:

```text
APP_STAGING_CLEANUP_ENABLED=true
APP_PROD_DEPLOY_ENABLED=false
```

This permits staging lifecycle automation without accidentally enabling production deployment.

## Example: BeltRisk Reference Implementation

This pattern was concretely applied to BeltRisk with:

| Parameter | BeltRisk value |
|---|---|
| repository | `plantops/plantops-beltrisk` |
| target Tailscale IP | `100.100.40.89` |
| initial SSH user | `root` |
| deploy directory | `/srv/cfc/belt-risk` |
| production port | `8124` |
| staging port | `8123` |
| staging Compose project | `beltrisk-staging` |
| staging lifetime | approximately 47 h + hourly cleanup |
| frozen reference image | `ghcr.io/plantops/plantops-beltrisk:sha-29e275d` |

These values are examples only. The reusable procedure is the architecture/invariants, not the BeltRisk-specific addresses or ports.

## Rejected / Superseded Alternatives

### Public GitHub runner → public SSH

Rejected because it unnecessarily exposes a plant/internal host to the public Internet.

Replaced by ephemeral Tailscale connectivity.

### Dedicated permanent staging server for every small app

Rejected as unnecessary infrastructure for low-scale applications where same-host Compose project isolation is sufficient.

Replaced by disposable same-host staging with isolated volumes and port.

### Self-hosted deployment runner as the default

Still valid when outbound policy or governance requires it, but not necessary when GitHub-hosted runners can obtain narrowly scoped ephemeral Tailscale access.

### Gitea in the deployment path

Gitea remains useful as an internal forge/mirror if needed for offline operation or local governance, but should not be introduced solely to relay GitHub deployments when GitHub Actions can safely reach the application host directly over Tailscale.

### Bidirectional GitHub ↔ Gitea source-of-truth

Rejected for normal development because it creates synchronization/ownership ambiguity. If Gitea is used, GitHub should normally remain canonical and Gitea should be a one-way internal mirror.

### Build on production

Rejected because it makes production non-reproducible and adds compiler/package-manager attack surface.

### Shared staging/production database

Rejected because staging migrations, test data, sessions, and failures can contaminate production.

## Reusable Implementation Checklist

### Repository

- [ ] CI tests pass on PR/main.
- [ ] Container is built in cloud CI.
- [ ] Immutable SHA tag is pushed to GHCR.
- [ ] Compose validates in CI.
- [ ] Production job deploys only exact immutable image.

### GitHub

- [ ] `staging` Environment exists.
- [ ] `production` Environment exists.
- [ ] production has manual approval if appropriate.
- [ ] `TS_OAUTH_CLIENT_ID` stored securely.
- [ ] `TS_AUDIENCE` stored securely.
- [ ] SSH private key stored as environment/repository secret.
- [ ] production enable variable starts false.
- [ ] staging cleanup/deploy switches are independent.

### Tailscale

- [ ] Application-specific deploy tag exists.
- [ ] GitHub workload identity is restricted to the intended repo/workflow/environment.
- [ ] Deploy tag can reach only required target host/SSH port.
- [ ] Target host is online in tailnet.

### Host

- [ ] Docker installed.
- [ ] Docker Compose plugin installed.
- [ ] Tailscale installed and online.
- [ ] `/srv/.../<app>` deploy directory exists.
- [ ] production `.env` is host-local and protected.
- [ ] deploy SSH public key installed.
- [ ] staging port does not collide with production.
- [ ] backup directory exists or is created during deploy.
- [ ] production DB has enough storage for backup.

### Staging

- [ ] unique Compose project
- [ ] unique PostgreSQL volume
- [ ] unique app storage volume
- [ ] unique DB credentials
- [ ] unique session secret
- [ ] unique cookie name
- [ ] different host port
- [ ] readiness + smoke tests
- [ ] expiry marker
- [ ] automatic cleanup

### Production

- [ ] manual approval/gate
- [ ] current image recorded
- [ ] PostgreSQL backup created and validated
- [ ] exact same staging-tested image pulled
- [ ] `--no-build`
- [ ] readiness + smoke tests
- [ ] running image verified
- [ ] deployment receipt written
- [ ] application rollback path tested

## Related Streams

- [GitHub Organization Migration](github-org-migration.md) — repository governance and CI/CD continuity during organization moves.
- [PlantOps Platform](plantops-platform.md) — reusable deployment/control-plane concerns should remain separate from bounded domain logic.

## Evolution

The deployment design evolved through three increasingly simpler options:

```text
1. GitHub Cloud → GHCR → local self-hosted deploy runner → staging/prod
2. GitHub Cloud → GHCR → Gitea/local runner → staging/prod
3. GitHub Cloud → GHCR → ephemeral Tailscale GitHub runner → staging/prod
```

For small internal containerized applications, option 3 became canonical because it removes a permanent runner/Gitea dependency while retaining private network reachability and strong deployment boundaries.

The same-host staging model also evolved from a separate staging node to a disposable Compose project on the production host, with strict isolation of DB, storage, environment and cookies and a maximum lifetime of about two days.

## Sources

- ChatGPT deployment design session, 2026-08-25
  - source_status: session-derived
  - raw_available: false
- GitHub repository `plantops/plantops-beltrisk`
  - source_status: reference implementation
- Tailscale GitHub Actions / workload identity design discussed and validated during the session
  - source_status: external-design-reference
