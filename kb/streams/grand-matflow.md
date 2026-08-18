# Grand MATFLOW

## Intent

Provide a reusable operational engine that records and serves the truth of physical material movement across industrial operations, independent of any one WMS UI or customer deployment model.

## Current Canonical State

Grand MATFLOW (GM) is the material-movement system of record inside PlantOps. It owns actual movement execution, observations, quantities, timing, routes, resource assignments, lineage and audit evidence. It does not own employee lifecycle, maintenance work orders, laboratory/QC workflow, berth optimization, inventory policy or KPI scoring.

GM may be delivered as shared SaaS, dedicated SaaS, or customer-hosted software without changing its core domain model. CFC can run a dedicated deployment while retaining tenant-aware identifiers so the same code and migrations remain reusable.

The product should not depend on a thick CRUD/dashboard UI as its primary value. Its durable value is canonical operational data, integration, lineage, auditability and APIs/events that humans, portals, analytics and AI agents can consume.

## Confirmed Facts

- GM is being treated as broader than a truck-scale application.
- The operational lineage discussed includes mother ship / shipment → barges → unloading sessions → truck movements → weighbridge tickets → store/stockpile.
- A weighbridge ticket is measurement evidence associated with physical movement, not the entire business process.
- Operations can span multiple shifts and require shift-based projections.
- GM is intended to participate in a larger WMS/PlantOps composition rather than absorb every operational domain.

## Requirements

- Preserve material movement lineage across upstream and downstream steps.
- Record actual quantity, time, route and resource usage.
- Support explicit and inferred associations when source systems do not carry complete identifiers.
- Keep inference distinguishable from confirmed source evidence.
- Support shift slicing/projections without mutating the original continuous movement.
- Expose stable APIs/events for other engines, portals and agents.
- Keep tenant/site context explicit.
- Remain deployable as shared SaaS, dedicated SaaS or customer-hosted.

## Constraints

- Some source tickets may lack reliable barge/shipment identifiers.
- Historical operational data must not be rewritten merely to fit a new inferred lineage.
- GM must remain bounded and must not become the whole WMS.

## Decisions

### GM is the operational material-movement system of record

- Status: CANONICAL
- Decision: GM owns actual movement execution and measurement evidence, not every surrounding business workflow.
- Reason: Movement truth must remain stable while planning, HR, maintenance, QC and KPI domains evolve independently.
- Supersedes: Treating Matflow as only a ticket-centric truck-scale application or as the entire WMS.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### SaaS is a delivery model, not GM's domain identity

- Status: CANONICAL
- Decision: The same GM product can be delivered as shared SaaS, dedicated SaaS or customer-hosted.
- Reason: Tenant isolation and deployment topology are separate from the material-movement domain model.
- Supersedes: Equating SaaS with one mandatory shared database or one UI-centric product form.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### Avoid UI/CRUD as the primary moat

- Status: TENTATIVE
- Decision: Position GM around domain truth, integration, lineage, audit and execution data rather than the difficulty of writing screens and CRUD workflows.
- Reason: AI agents reduce the defensibility of generic application code and fixed UI workflows.
- Supersedes: None.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

## Architecture / Model

```text
Human UI / AI agents / WMS portal / planners / analytics
                         │
                         ▼
                  Grand MATFLOW API
                         │
                         ▼
       canonical movements + observations + lineage
                         │
                         ▼
                 audit / projections / events
```

Operational lineage example:

```text
Mother Ship / Shipment
        ↓
Multiple Barges
        ↓
Barge Unloading Session
        ↓
Multiple Truck Movements
        ↓
Weighbridge Tickets
        ↓
Store / Stockpile
```

## Business Rules / Invariants

- Measurement evidence must remain traceable to its source.
- Inferred links must not silently become confirmed links.
- A continuous movement may be projected into multiple shifts without altering the original movement interval.
- GM records actual execution; planning engines own intended plans.
- Cross-engine references use stable IDs/contracts rather than shared table ownership.

## Important Entities

- Grand MATFLOW
- Movement
- Observation
- Resource
- Location
- Assignment
- Relation
- Route
- Shipment
- Barge
- Weighbridge ticket
- Store / stockpile
- `tenant_id`
- `site_id`

## Important Numbers / Parameters

| Parameter | Value | Context |
|---|---:|---|
| CFC shift C | 00:00–08:00 | Existing operational shift model discussed for GM |
| CFC shift A | 08:00–16:00 | Existing operational shift model discussed for GM |
| CFC shift B | 16:00–24:00 | Existing operational shift model discussed for GM |
| CFC timezone | `Asia/Ho_Chi_Minh` | Canonical tenant timezone for future config |

## Rejected / Superseded Alternatives

- **Alternative:** GM as the entire WMS.  
  **Why rejected/superseded:** It would absorb planning, HR, CMMS, QC, inventory and KPI policy into one bounded context.  
  **Replaced by:** GM as one engine composed with other PlantOps engines.

- **Alternative:** Ticket-only lineage using a single `barge_id`.  
  **Why rejected/superseded:** It cannot robustly express mother ship → barge → truck execution or uncertain historical association.  
  **Replaced by:** Explicit movement relations plus association evidence/projections.

## Open Questions

- Exact confidence/evidence model for inferred movement associations.
- Exact first implementation of cross-engine read projections for the unified portal.

## Pending Actions

- **NOW:** Continue implementation of mother ship → barge → truck lineage and shift slicing inside the GM boundary.
- **NOW:** Complete GitHub migration pilot to `plantops/grand-matflow` before expanding migration scope.
- **LATER:** Provide shared-SaaS and dedicated deployment profiles using the same engine code/migrations.

## Related Streams

- [PlantOps Platform](plantops-platform.md) — GM is one bounded engine within PlantOps.
- [Legacy Port Modernization](legacy-port-modernization.md) — legacy port data can feed GM while legacy functions are progressively replaced.

## Evolution

Earlier Matflow work was ticket-centric and closely coupled to truck-scale ingestion. The architecture evolved toward Grand MATFLOW as a broader material-movement abstraction with explicit movement, observation, lineage and cross-engine boundaries. The current direction keeps GM narrow in domain ownership while making it reusable across deployments and consumption modes.

## Sources

- ChatGPT session `current-chat-2026-08-18`
  - source_status: context-derived
  - raw_available: false
