# HPRadarHQ

## Intent

Provide a dedicated GitHub organization boundary for HPRadar's aircraft/ship tracking, ingestion, edge and related product repositories, separate from PlantOps and the user's personal repository namespace.

## Current Canonical State

The GitHub organization is `hpradarhq`. It is accessible to the current Kiro GitHub credentials. At the latest check in this session it contained 0 repositories, so repository migration has not yet started.

HPRadar migration should follow the same proven checklist as PlantOps, but only after the Grand MATFLOW pilot is complete. A small repo such as `hpr-demod` was identified as a sensible first HPRadar pilot rather than starting with a larger operationally coupled repository.

## Confirmed Facts

- Organization: `hpradarhq`.
- Kiro can access the organization using the current `ngtrthanh` GitHub identity.
- Repo count was 0 at the time of access validation.
- The HPRadar product brand/domain remains distinct from the GitHub organization slug.

## Requirements

- Keep HPRadar source governance separate from PlantOps.
- Preserve CI/CD and package/deployment continuity during transfer.
- Start with a small pilot after PlantOps migration is proven.

## Constraints

- The organization currently has no migrated repositories in the session's latest verified state.
- Some HPRadar repositories are operationally coupled and therefore poor first migration pilots.

## Decisions

### Use `hpradarhq` as the organization boundary

- Status: CANONICAL
- Decision: HPRadar repositories should move under `hpradarhq` rather than remain permanently under `ngtrthanh`.
- Reason: Product source ownership should be separated from personal repositories and from PlantOps.
- Supersedes: Keeping the product family solely under the personal namespace.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### Defer HPRadar migration until the PlantOps pilot is complete

- Status: CANONICAL
- Decision: Do not migrate HPRadar in parallel with the first Grand MATFLOW transfer.
- Reason: Use the PlantOps pilot to establish the transfer/Actions/GHCR verification checklist first.
- Supersedes: Parallel first-time migration of both organizations.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

## Architecture / Model

```text
ngtrthanh/* HPRadar repos
        ↓
small pilot transfer
        ↓
hpradarhq/<repo>
        ↓
verify Actions / packages / deployments
        ↓
repeat for remaining HPRadar repos
```

## Business Rules / Invariants

- GitHub organization slug does not redefine the external HPRadar product brand.
- Migration must preserve operational continuity and traceability.

## Important Entities

- `hpradarhq`
- `ngtrthanh`
- HPRadar
- `hpr-demod`
- GitHub Actions
- GHCR

## Important Numbers / Parameters

| Parameter | Value | Context |
|---|---:|---|
| `hpradarhq` repo count at access test | 0 | Before migration |

## Rejected / Superseded Alternatives

- **Alternative:** Start migration with a large/coupled HPRadar repository.  
  **Why rejected/superseded:** Higher first-migration operational risk.  
  **Replaced by:** A small pilot such as `hpr-demod` after PlantOps validation.

## Open Questions

- Final migration order for HPRadar repositories after the pilot.

## Pending Actions

- **LATER:** Migrate the first HPRadar pilot after Grand MATFLOW migration is verified.

## Related Streams

- [GitHub Organization Migration](github-org-migration.md) — shared migration method and invariants.

## Evolution

Organization access was validated before any repository transfer. Migration remains intentionally deferred until the PlantOps pilot produces a proven checklist.

## Sources

- ChatGPT session `current-chat-2026-08-18`
  - source_status: context-derived
  - raw_available: false
