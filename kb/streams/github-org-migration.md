# GitHub Organization Migration

## Intent

Move product repositories from personal ownership under `ngtrthanh` into product organizations while preserving source history, CI/CD, GHCR packages, deployment continuity and local developer workflows.

## Current Canonical State

Two new GitHub organizations are accessible with the current Kiro/GitHub credentials:

- `plantops`
- `hpradarhq`

Kiro CLI is the preferred execution tool for repository transfer and local/workflow updates because it has an authenticated `gh` session and organization/repository permissions. ChatGPT's GitHub connector is used as a verification/review control plane after the transfer.

Migration is intentionally incremental. The first pilot is `ngtrthanh/plantops-grand-matflow` → `plantops/grand-matflow`. HPRadar migration should wait until this first loop is completed and its checklist is proven.

## Confirmed Facts

Kiro reported in this session:

- Authenticated GitHub user: `ngtrthanh`.
- Authentication source: `GH_TOKEN`.
- Scopes include `admin:org`, `repo`, `delete_repo`, `workflow`, and other broad permissions.
- `plantops`: accessible, 0 repositories at the time of test.
- `hpradarhq`: accessible, 0 repositories at the time of test.
- No access errors were reported.

## Requirements

- Preserve repository history and default-branch state.
- Record pre-transfer main SHA, open PRs, workflows and package/image references.
- Update local `origin` after transfer.
- Search and update old repository URLs/badges/workflow references that are materially affected.
- Build new GHCR namespace artifacts before removing old package references.
- Do not deploy production merely to test the migration.
- Complete one pilot end-to-end before bulk migration.

## Constraints

- Repository transfer is not exposed as a direct action by the current ChatGPT GitHub connector.
- Existing deployments may still pull `ghcr.io/ngtrthanh/...` images.
- GitHub redirects after repository transfer are useful compatibility aids but should not be treated as the permanent configuration.

## Decisions

### Kiro executes repository transfers

- Status: CANONICAL
- Decision: Use Kiro/`gh` to perform organization migrations and local/workflow adjustments.
- Reason: It has the required org/repo permissions and can operate on both GitHub and the local checkout.
- Supersedes: Attempting to perform the transfer solely through the ChatGPT connector.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### Connector verifies and reviews

- Status: CANONICAL
- Decision: Use the ChatGPT GitHub connector to verify the new repository, review diffs/PRs and inspect CI after Kiro's transfer.
- Reason: Separates execution from independent inspection.
- Supersedes: None.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### Migrate one pilot before HPRadar

- Status: CANONICAL
- Decision: Complete the PlantOps Grand MATFLOW pilot before migrating an HPRadar repo.
- Reason: Establish a repeatable transfer/CI/GHCR checklist with lower change surface.
- Supersedes: Migrating both organizations in parallel immediately.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

## Architecture / Model

```text
Kiro / gh
  ↓
record current repo state
  ↓
transfer + rename
  ↓
update local remote and affected references
  ↓
test/build
  ↓
push migration branch / draft PR
  ↓
ChatGPT connector verifies repo + diff + CI
  ↓
new GHCR/deployment references only after validation
```

## Business Rules / Invariants

- Do not delete old GHCR packages until new namespace artifacts are built and consumers are migrated.
- Do not combine unrelated repository migrations in the pilot.
- Do not use `latest`-style ambiguity as migration evidence; preserve traceable release identity where deployment is involved.

## Important Entities

- `ngtrthanh`
- `plantops`
- `hpradarhq`
- Kiro CLI
- `gh`
- ChatGPT GitHub connector
- GitHub Actions
- GHCR
- `ngtrthanh/plantops-grand-matflow`
- `plantops/grand-matflow`

## Important Numbers / Parameters

| Parameter | Value | Context |
|---|---:|---|
| `plantops` repo count at access test | 0 | Before migration |
| `hpradarhq` repo count at access test | 0 | Before migration |

## Rejected / Superseded Alternatives

- **Alternative:** Bulk transfer many repositories at once.  
  **Why rejected/superseded:** CI, package and deployment breakage would be harder to isolate.  
  **Replaced by:** One complete pilot followed by repeatable migration.

## Open Questions

- Has the Grand MATFLOW transfer actually completed yet? No completion result was present before this distillation.
- Exact package/deployment adjustments needed will depend on the migrated repository's current workflow references.

## Pending Actions

- **NOW:** Kiro migrates `ngtrthanh/plantops-grand-matflow` → `plantops/grand-matflow` and opens the migration PR after tests.
- **NOW:** Verify the migrated repo and CI through the connector once Kiro reports completion.
- **LATER:** Migrate a small HPRadar repo after the PlantOps checklist is proven.

## Related Streams

- [PlantOps Platform](plantops-platform.md) — destination organization for PlantOps engine repositories.
- [HPRadarHQ](hpradarhq.md) — separate product organization following the same migration discipline.

## Evolution

The organization connectors/access were first validated without write actions. After confirming both orgs were visible with adequate Kiro permissions, responsibility was split: Kiro executes transfer/local changes; ChatGPT connector verifies and reviews.

## Sources

- ChatGPT session `current-chat-2026-08-18`
  - source_status: context-derived
  - raw_available: false
