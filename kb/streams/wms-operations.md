# WMS Operations

## Intent

Run the real warehouse and import-port workflow correctly while keeping planning, execution truth and broader PlantOps domain ownership distinguishable.

## Current Canonical State

WMS is the practical operational application/composition for the current warehouse and port workflow. Its immediate job is to preserve the proven workflow, consolidate fragmented Excel-era state into a small web application, retain Excel import/export compatibility, integrate the deterministic Port Unloading Planner, and consume Grand MATFLOW as the source of actual movement execution.

Canonical operational separation:

```text
Planner = what SHOULD move
GM / MatFlow = what ACTUALLY moved
REPORT = how it actually performed

PLAN → EXECUTE → REPORT → LEARNED TPH → PLAN
```

For V1, implementation should remain deliberately small: Go + PostgreSQL + server-rendered HTML/htmx, with Python/openpyxl for Excel interchange. Excel is an import/export/fallback channel, not the production database. The V1 application may temporarily own practical workflow state such as shipments, barge registration, QC state and store/inventory views; long-term PlantOps architecture may move these responsibilities into bounded engines such as PortOps, Quality and Inventory without changing GM's execution boundary.

## Confirmed Facts

- The operational workflow existed and had been running across multiple Excel workbooks before the web migration effort.
- The WMS V1 direction is to preserve the proven business workflow rather than redesign it from scratch.
- GM / MatFlow remains an external execution engine to the WMS application.
- Excel interoperability remains required during migration and operation.

## Requirements

- Preserve the proven operational workflow.
- Consolidate operational state into PostgreSQL.
- Integrate the existing Port Unloading Planner.
- Consume actual material movement from GM / MatFlow through an idempotent adapter.
- Retain Excel import/export and reconciliation.
- Provide a dense, fast operational UI with clear action/status visibility.
- Support user/workstation RBAC and audit logging.
- Reconcile historical/operational results rather than changing formulas merely to force a match.

## Constraints

- V1 must avoid speculative expansion.
- Do not introduce microservices or Kubernetes for the V1 problem.
- Do not add AI, generic optimization engines, digital-twin redesign or new MatFlow variants before the core workflow is working.
- Existing Excel semantics and real historical data must be treated as migration evidence, not casually discarded.

## Decisions

### Keep WMS V1 deliberately small

- Status: CANONICAL
- Decision: Use a small web stack centered on Go, PostgreSQL, server-rendered HTML/htmx and Python/openpyxl for Excel interchange.
- Reason: The business workflow already exists; the immediate objective is a reliable production application, not architectural novelty.
- Supersedes: Continuing fragmented Excel files as the primary production state, and over-engineered microservice/platform rewrites for V1.
- Source: ChatGPT context `chess-to-grand-matflow-context-2026-08`, corroborated by the existing WMS V1 handoff artifact.

### Separate plan, execution and performance evidence

- Status: CANONICAL
- Decision: Planner owns intended movement, GM owns actual execution truth, and reporting measures actual performance and feeds learned TPH back into planning.
- Reason: Separating intent from execution facts prevents planning state from masquerading as operational truth.
- Supersedes: Treating WMS planning tables or manual entries as the sole execution source.
- Source: ChatGPT context `chess-to-grand-matflow-context-2026-08`.

### Treat WMS as composition, not the universal domain core

- Status: CANONICAL
- Decision: The operational WMS may present one workflow/UI while bounded PlantOps engines retain their own long-term domain ownership.
- Reason: UX unity does not require Grand MATFLOW or one WMS database to own HR, maintenance, QC, berth planning, inventory policy and KPI logic.
- Supersedes: Expanding GM into the entire WMS or treating one monolithic application boundary as the permanent PlantOps domain model.
- Source: ChatGPT context `chess-to-grand-matflow-context-2026-08`.

## Architecture / Model

```text
Browser
   │
   ▼
WMS operational app
   │
   ├── PostgreSQL
   ├── Port Unloading Planner
   ├── Grand MATFLOW API
   └── Excel import/export worker
```

Operational loop:

```text
PLAN
  ↓
EXECUTE
  ↓
REPORT
  ↓
LEARNED TPH
  ↓
PLAN
```

## Business Rules / Invariants

- Planner state expresses intended work, not confirmed physical execution.
- GM / MatFlow movement IDs must be handled idempotently when imported into WMS events.
- Production source of truth should not remain fragmented across editable Excel workbooks.
- Excel remains useful for import/export/reconciliation but is not the canonical production database.
- Working real port workflow takes priority over speculative architectural elegance in V1.

## Important Entities

- WMS
- Port Unloading Planner
- Grand MATFLOW / MatFlow
- PostgreSQL
- Go
- htmx
- Python
- openpyxl
- Excel
- Shipment
- Barge
- Crane
- QC
- Store / Inventory
- REPORT / TPH

## Important Numbers / Parameters

No new numerical parameter from the historical cross-topic session is sufficiently certain to add here beyond values already retained in other streams.

## Rejected / Superseded Alternatives

- **Alternative:** Keep the production workflow primarily in fragmented Excel files.  
  **Why rejected/superseded:** Multi-user operational state and integration require a durable shared application/database.  
  **Replaced by:** PostgreSQL-backed WMS with Excel interoperability.

- **Alternative:** Build generic ERP/TOS, microservices, Kubernetes, AI or broad optimization before V1.  
  **Why rejected/superseded:** These distract from the committed real workflow and add unnecessary complexity.  
  **Replaced by:** Small V1 focused on the existing port/warehouse workflow.

## Open Questions

- What is the safest migration path from pragmatic WMS V1 ownership into long-term PortOps, Quality and Inventory engine boundaries without disrupting production?
- Whether the first unified portal should compose directly from engine APIs or use a dedicated operations read model remains an architectural question shared with PlantOps.

## Pending Actions

- **NOW:** Keep WMS implementation aligned with the real operational workflow and GM execution contract.
- **NOW:** Preserve Excel reconciliation during migration to PostgreSQL.
- **LATER:** Evolve responsibilities toward bounded PlantOps engines only when production value and boundaries justify it.

## Related Streams

- [Grand MATFLOW](grand-matflow.md) — provides actual material-movement execution truth.
- [PlantOps Platform](plantops-platform.md) — provides the long-term bounded-engine architecture into which the WMS experience composes.

## Evolution

The WMS began from a proven multi-workbook operational process. The near-term solution remains a deliberately small web application. As Grand MATFLOW and PlantOps boundaries became clearer, WMS shifted conceptually from being the entire architecture to being the practical workflow/composition layer over planning, execution and other bounded domains.

## Sources

- ChatGPT context `chess-to-grand-matflow-context-2026-08`
  - source_status: context-derived
  - raw_available: false
- Supporting artifact available in File Library: `WMS_V1_Technical_Implementation_Handoff.md`
