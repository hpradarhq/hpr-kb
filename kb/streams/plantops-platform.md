# PlantOps Platform

## Intent

Provide a modular industrial operations platform in which bounded domain engines can be composed per tenant without turning the whole system into one monolithic WMS or ERP.

## Current Canonical State

PlantOps is organized as a family of reusable domain engines behind a unified operational experience. Grand MATFLOW owns material movement truth; PortOps owns voyage/barge/queue/berth planning context; Workforce owns people/roster; CMMS owns asset maintenance; Quality owns QC workflow; Polaris owns KPI policy; other engines can own inventory and coal intelligence/planning.

A tenant enables capabilities through a module matrix. Platform/control-plane concerns such as tenant registry, module subscription, identity/RBAC, deployment profile, service discovery, usage/health, schema version and feature flags should stay outside GM.

A unified portal may compose data from multiple engines, but UX unity does not imply shared business-table ownership.

## Confirmed Facts

- GitHub organization `plantops` has been created and is accessible to the current GitHub credentials.
- At the latest access check in this session, `plantops` contained 0 repositories.
- The intended first repository migration is `ngtrthanh/plantops-grand-matflow` → `plantops/grand-matflow`.
- The WMS/operations solution needs domains beyond material movement, including workforce, maintenance, voyage/barge operations, QC, inventory/dispatch and KPI.

## Requirements

- Keep engine ownership explicit.
- Allow one tenant to enable only the modules it needs.
- Support a unified portal without direct cross-engine business-table writes.
- Use stable IDs, APIs/events and rebuildable projections for integration.
- Keep tenant-specific configuration separate from reusable engine code/migrations.

## Constraints

- Existing code is still distributed across personal repositories and different maturity levels.
- Migration must preserve current CI/CD and package/deployment continuity.
- Do not create customer-specific source forks merely to support deployment isolation.

## Decisions

### Organize PlantOps around bounded engines

- Status: CANONICAL
- Decision: PlantOps is a product family of independent engines, not one giant WMS codebase/database.
- Reason: Real operations span domains with independent ownership and lifecycle.
- Supersedes: Expanding Grand MATFLOW to own all WMS, HR, CMMS, QC, planning and KPI concerns.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### Keep platform control-plane concerns outside GM

- Status: CANONICAL
- Decision: Tenant registry, subscriptions/modules, identity/RBAC, deployment profile, health/schema version and feature flags belong to the PlantOps platform/control plane.
- Reason: These are platform concerns shared across engines rather than material-movement domain state.
- Supersedes: None.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

## Architecture / Model

```text
PlantOps Platform
├── Grand MATFLOW        # material movement execution truth
├── PortOps              # voyage/barge/queue/berth planning context
├── Workforce            # people/team/roster/attendance
├── CMMS / Nano-CMMS     # asset/work-order/maintenance
├── Quality              # samples/tests/specs
├── Inventory            # stock/placement optimization
├── Coal Intelligence    # coal-quality enrichment/relations
├── Polaris              # KPI policy/scoring
└── Portal / Control Plane
```

Composition rule:

```text
Tenant module matrix
        ↓
resolve required engines
        ↓
provision engine deployments/config
        ↓
compose through APIs/events/read models
```

## Business Rules / Invariants

- Engine != module: an engine is a bounded domain service; a module is a tenant-selectable capability.
- One engine may provide multiple modules.
- Each engine owns writes to its own domain data.
- No cross-engine foreign-key dependency is required for business ownership.
- A portal can aggregate views without becoming the source of truth for every domain.

## Important Entities

- `plantops`
- Grand MATFLOW
- PortOps
- Workforce
- CMMS / Nano-CMMS
- Quality
- Inventory
- Coal Intelligence
- Polaris
- Portal
- Control Plane
- Tenant module matrix

## Important Numbers / Parameters

| Parameter | Value | Context |
|---|---:|---|
| `plantops` repositories at access test | 0 | New organization before migration pilot |

## Rejected / Superseded Alternatives

- **Alternative:** One monolithic WMS engine/database.  
  **Why rejected/superseded:** Blurs ownership and makes independent evolution/integration difficult.  
  **Replaced by:** Bounded engines composed at platform/portal level.

## Open Questions

- Exact initial boundary between direct API composition and a dedicated operations read model.
- Final repo creation/migration order after the Grand MATFLOW pilot.

## Pending Actions

- **NOW:** Complete Grand MATFLOW repository migration pilot.
- **LATER:** Establish shared foundation repos such as `.github`, contracts and deployment configuration as needed.
- **LATER:** Migrate or create additional engines only after boundaries are stable.

## Related Streams

- [Grand MATFLOW](grand-matflow.md) — one bounded PlantOps engine.
- [GitHub Organization Migration](github-org-migration.md) — governs movement of existing repos into `plantops`.
- [Legacy Port Modernization](legacy-port-modernization.md) — legacy port capability should transition into bounded PlantOps engines rather than a replacement monolith.

## Evolution

The operating concept evolved from treating WMS/Matflow as a large application toward a platform of bounded engines with a unified portal and shared control-plane concerns. This keeps the UI composable while preserving domain ownership.

## Sources

- ChatGPT session `current-chat-2026-08-18`
  - source_status: context-derived
  - raw_available: false
