# GitHub–Tailscale Immutable Artifact Deployment

## Intent

Provide a reusable deployment procedure for small industrial/internal applications where GitHub is the canonical source and deployment must remain simple, private, reproducible, and auditable.

The procedure is **artifact-agnostic**. The artifact may be:

- OCI/Docker image
- Go/Rust/native binary
- `.deb` / `.rpm`
- static web bundle
- signed firmware/package

The canonical invariant is:

```text
BUILD ONCE
   ↓
IMMUTABLE ARTIFACT
   ↓
STAGING
   ↓
VERIFY
   ↓
APPROVE
   ↓
PROMOTE THE EXACT SAME ARTIFACT
   ↓
PRODUCTION
```

Never rebuild separately for production.

## Canonical Architecture

```text
feature/*
   ↓
GitHub PR
   ↓
GitHub Cloud CI
   ↓
main
   ↓
build + test once
   ↓
immutable artifact + checksum/digest
   ↓
GitHub-hosted deployment runner
   ↓
ephemeral Tailscale identity
   ↓
private SSH to target host
   ↓
short-lived staging
   ↓
health/smoke/RBAC checks
   ↓
GitHub Environment approval
   ↓
production using exact same artifact
   ↓
deployment receipt
```

Public inbound SSH, a permanent self-hosted CI runner, Gitea, Kubernetes, or a dedicated staging server are not required by default for this class of app.

## Core Invariants

1. **GitHub is the source of truth.**
2. **Build once.** Production never recompiles source.
3. **Artifact is immutable and identifiable** by image digest, Git SHA, checksum, release version, or equivalent.
4. **Staging and production consume the same artifact bytes.**
5. **GitHub reaches the host privately over Tailscale.** No Internet-facing SSH is required.
6. **Tailscale deploy identity is narrowly scoped** to the intended repo/workflow/environment and target host:SSH port.
7. **Production requires explicit approval.**
8. **Health/readiness checks determine deployment success**, not merely process/container startup.
9. **Write a deployment receipt** for every successful promotion.
10. **Application rollback and database rollback are separate decisions.**
11. **Use a dedicated `deploy` OS user** once bootstrap is complete; root is a temporary shortcut only.

## Artifact Type A — Container / OCI Image

### Build

CI builds and publishes an immutable image, for example:

```text
ghcr.io/<org>/<app>:sha-abcdef1
```

Prefer also recording the image digest:

```text
sha256:<digest>
```

### Deploy

Production/staging hosts do not need source code, Go, Node, npm, or compilers.

```bash
export APP_IMAGE="ghcr.io/<org>/<app>:sha-abcdef1"
docker compose pull app
docker compose up -d --no-build app
```

For DB-backed apps, start the DB/runtime according to the application's Compose contract.

### Staging on the same host

Use a separate Compose project and port:

```bash
docker compose \
  -p <app>-staging \
  --env-file .env.staging \
  up -d --no-build
```

This must isolate:

- containers
- network
- DB volume
- application storage
- runtime secrets
- session secret
- session cookie name
- host port

If staging and production share a hostname/IP but use different ports, the staging cookie name **must differ** because browser cookies are scoped by host/domain, not TCP port.

### Disposable staging

Recommended for small apps:

- create on demand
- clean isolated DB/storage
- separate port
- 47-hour expiry marker
- hourly cleanup
- `docker compose ... down -v --remove-orphans`

This keeps staging under the intended 48-hour maximum and prevents it silently becoming a second production environment.

## Artifact Type B — Native Binary + systemd

Use this for small edge agents, collectors, feeders, daemons, heartbeat services, NUC/Pi utilities, and other software where a container adds little value.

### Build

CI produces a deterministic target binary and checksum, for example:

```text
myapp-linux-amd64
myapp-linux-amd64.sha256
```

Publish through a GitHub Release, package registry, or another immutable artifact store.

Required metadata:

```text
git_sha
version
artifact_name
sha256
build_target
```

### Host layout

Recommended:

```text
/opt/myapp/
├── releases/
│   ├── 1.2.3/myapp
│   └── 1.2.4/myapp
├── current -> releases/1.2.4/myapp
├── .env
└── data/
```

The systemd service executes `/opt/myapp/current`.

### Binary deployment

```bash
mkdir -p /opt/myapp/releases/1.2.4
# download exact immutable artifact
sha256sum -c myapp-linux-amd64.sha256
install -m 0755 myapp-linux-amd64 /opt/myapp/releases/1.2.4/myapp
ln -sfn /opt/myapp/releases/1.2.4/myapp /opt/myapp/current
systemctl restart myapp
curl -fsS http://127.0.0.1:<port>/health/ready
```

Do not compile on the target host.

### Binary rollback

```bash
ln -sfn /opt/myapp/releases/1.2.3/myapp /opt/myapp/current
systemctl restart myapp
curl -fsS http://127.0.0.1:<port>/health/ready
```

Retain a small number of known-good releases so rollback is a pointer switch rather than a rebuild.

### Binary staging

Options, in order of preference:

1. same binary artifact, separate systemd unit and port (`myapp-staging.service`)
2. same host, isolated staging data directory and env file
3. disposable VM/container only when process-level isolation is insufficient

Example:

```text
PROD: myapp.service         :8124   /opt/myapp/data
STG : myapp-staging.service :8123   /opt/myapp/staging-data
```

The staging service must not share writable production state.

## GitHub → Tailscale → Target Host

Use workload identity federation/OIDC rather than a reusable Tailscale auth key where possible.

Typical GitHub job permissions:

```yaml
permissions:
  contents: read
  packages: read
  id-token: write
```

Typical secrets:

```text
TS_OAUTH_CLIENT_ID
TS_AUDIENCE
PROD_SSH_KEY
```

The workflow joins Tailscale with an application-specific tag:

```text
tag:<app>-deploy
```

Tailnet policy should allow only the minimum path:

```text
tag:<app>-deploy → <target-host>:22
```

Do not grant broad LAN, PLC, SCADA, hypervisor, database, or unrelated-server access merely for deployment.

## Staging Verification Contract

A staging deployment is successful only when appropriate checks pass.

Minimum web/service checks:

```text
/health/live
/health/ready
/login or equivalent public route
critical authenticated page(s), where safe
RBAC negative checks, where practical
expected schema/migration version
running artifact identity == intended artifact identity
```

For non-HTTP daemons, replace HTTP checks with deterministic service-specific health probes, for example:

- systemd active state
- local control socket
- heartbeat timestamp
- metrics endpoint
- expected output file/event
- upstream/downstream connectivity test

## Production Promotion Contract

Before promotion:

- CI green
- exact immutable artifact exists
- staging passed
- production approval granted
- target reachable over Tailscale
- disk/resource checks acceptable
- previous artifact recorded
- DB backup completed if the release can migrate persistent DB state

Then deploy the **same artifact** tested in staging.

## Database Rule

Before a production migration, take and verify a backup.

Container/PostgreSQL example:

```bash
docker compose exec -T db \
  sh -c 'pg_dump -Fc -U "$POSTGRES_USER" "$POSTGRES_DB"' \
  > backup/app-prod-<timestamp>.dump
```

Application rollback does not automatically imply DB restore.

Prefer backward-compatible expand/contract migrations so a previous application artifact can run against the migrated schema when possible.

## Deployment Receipt

Every successful staging/production deploy should record at least:

```text
deployment_id
timestamp
environment
git_sha
artifact_type
artifact_reference
artifact_digest_or_sha256
host
port_or_service
previous_artifact
db_migration_version if applicable
health_status
actor/workflow
```

Examples:

```text
artifact_type      = OCI
artifact_reference = ghcr.io/org/app:sha-abcdef1
artifact_digest    = sha256:...
```

or:

```text
artifact_type      = BINARY
artifact_reference = myapp-linux-amd64@1.2.4
artifact_sha256    = ...
```

## Choosing Container vs Binary

Use **container + Compose** when:

- application has DB/runtime dependencies
- multiple services must move together
- reproducible runtime packaging is valuable
- same-host isolated staging is useful

Use **binary + systemd** when:

- service is small and self-contained
- edge hardware is resource constrained
- operational dependency surface should be minimal
- target already has a stable OS/runtime contract

Do not convert a naturally simple binary into a multi-container stack just for architectural uniformity.

## Reference Pattern — BeltRisk

BeltRisk is the first concrete container reference implementation:

| Parameter | Value |
|---|---|
| repository | `plantops/plantops-beltrisk` |
| target Tailscale IP | `100.100.40.89` |
| initial SSH user | `root` (bootstrap; dedicated deploy user preferred) |
| deploy directory | `/srv/cfc/belt-risk` |
| production port | `8124` |
| staging port | `8123` |
| staging Compose project | `beltrisk-staging` |
| staging lifetime | ~47h + hourly cleanup |
| frozen V1 image | `ghcr.io/plantops/plantops-beltrisk:sha-29e275d` |

These values are reference-instance facts, not defaults for other apps.

## Rejected Defaults

- **Build on production** — rejected; non-reproducible and increases attack surface.
- **`latest` as production artifact** — rejected; not an immutable release identity.
- **Public SSH solely for CI/CD** — rejected; use Tailscale.
- **Shared staging/production writable state** — rejected.
- **Permanent staging for every small app** — unnecessary by default.
- **Kubernetes for a single small app** — unnecessary unless scale/availability requires it.
- **Gitea solely as a deployment relay** — unnecessary if GitHub Actions can reach the target privately via Tailscale.
- **Two-way GitHub/Gitea source-of-truth** — rejected due ownership/synchronization ambiguity.

## Reusable Checklist

### Repository / CI

- [ ] GitHub canonical source
- [ ] tests before artifact publication
- [ ] immutable artifact reference
- [ ] digest/SHA256 recorded
- [ ] no production builds

### Tailscale / SSH

- [ ] OIDC workload identity
- [ ] repo/workflow/environment constrained
- [ ] application-specific deploy tag
- [ ] only target SSH allowed
- [ ] SSH private key stored only as secret
- [ ] dedicated `deploy` user planned/used

### Staging

- [ ] exact production candidate artifact
- [ ] separate port/service instance
- [ ] isolated writable state
- [ ] separate cookie/session state for same-host web apps
- [ ] readiness/smoke checks
- [ ] artifact identity verified
- [ ] automatic expiry/cleanup where staging is ephemeral

### Production

- [ ] manual approval
- [ ] previous artifact recorded
- [ ] DB backup if applicable
- [ ] exact staged artifact promoted
- [ ] readiness + smoke checks
- [ ] receipt written
- [ ] application rollback path proven

## Decision

**Status: CANONICAL**

For similar small PlantOps/HPR/internal apps, the default deployment model is now:

> **GitHub Cloud CI → immutable artifact → ephemeral Tailscale deployment access → short-lived isolated staging → approval → exact-artifact production promotion.**

Containers and native binaries are two implementation profiles of the same deployment contract.

## Sources

- ChatGPT deployment design session, 2026-08-25 — session-derived.
- `plantops/plantops-beltrisk` — first container reference implementation.
