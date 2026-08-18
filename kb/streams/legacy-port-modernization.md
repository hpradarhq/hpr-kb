# Legacy Port Modernization

## Intent

Manage the company's decision to restore an older port-management application without allowing short-term continuity work to become the long-term PlantOps architecture.

## Current Canonical State

The legacy port application should be treated as a temporary operational-continuity layer with a deliberately limited scope, not as the target WMS architecture. The preferred migration pattern is strangler/integration: restore only the functions required for continuity, ensure CFC retains control of data and integration access, feed useful operational events/data into new canonical engines, then replace legacy capabilities incrementally with PortOps/Grand MATFLOW/other PlantOps modules.

The organizational strategy is to avoid a political argument framed as "old software is stupid". The stronger case is risk containment: continuity now, ownership/API/export guarantees, explicit scope boundaries and a defined exit path.

## Confirmed Facts

- The company is returning to/restoring a port-management application written roughly four years earlier.
- The user considers the application technically poor compared with the new architecture being developed.
- New PlantOps/Grand MATFLOW architecture is being developed in parallel.

## Requirements

For any restored legacy deployment, aim to secure:

- Company control of the operational database/data.
- Company-controlled administrative access.
- Source availability or another credible continuity mechanism where contractually possible.
- Read API and/or reliable CSV/JSON export.
- Auditability appropriate to operational changes.
- Clear ownership for maintenance/support.
- Explicit limit on new feature expansion in the legacy codebase.

## Constraints

- Management may value familiarity and immediate continuity more than architecture quality.
- A big-bang replacement may be politically and operationally unrealistic.
- The actual source/build/database/security condition of the legacy application has not yet been audited in this session.

## Decisions

### Treat restoration as temporary continuity

- Status: CANONICAL
- Decision: Do not treat the revived legacy port application as the target architecture.
- Reason: It can satisfy immediate operational continuity while new bounded engines mature, without committing future capabilities to the old codebase.
- Supersedes: Either immediately replacing everything or extending the legacy application indefinitely.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

### Use a strangler migration pattern

- Status: CANONICAL
- Decision: Integrate and progressively replace legacy functions instead of a big-bang rewrite.
- Reason: Reduces operational/political risk while allowing canonical operational data to move toward PlantOps engines.
- Supersedes: Full cutover as the first modernization step.
- Source: ChatGPT session `current-chat-2026-08-18`, context-derived.

## Architecture / Model

```text
Legacy Port App
├── registration
├── queue
└── berth/status functions
        │
        ▼
 integration / export / API
        │
        ├──► PortOps       # registry, queue, berth planning progressively
        └──► Grand MATFLOW # actual movement / quantity / lineage
                         
New modules replace legacy capabilities one by one
        ↓
legacy becomes read-only
        ↓
retire
```

## Business Rules / Invariants

- Restoring continuity must not implicitly authorize unlimited new development in the legacy codebase.
- Operational data must remain extractable and under company control.
- Legacy evidence can be consumed by new engines, but domain ownership should move toward the PlantOps boundaries.
- GM owns actual material movement; PortOps owns voyage/barge/queue/berth planning context.

## Important Entities

- Legacy Port App
- CFC
- PlantOps
- PortOps
- Grand MATFLOW
- operational database
- API/export

## Important Numbers / Parameters

| Parameter | Value | Context |
|---|---:|---|
| Legacy application age | ~4 years | User description in this session |

## Rejected / Superseded Alternatives

- **Alternative:** Confront management by arguing the old software is simply bad and should not be restored.  
  **Why rejected/superseded:** Does not address management's continuity/risk motivation and creates unnecessary political resistance.  
  **Replaced by:** Conditional restoration with scope, data-access and exit controls.

- **Alternative:** Keep adding new WMS functionality to the restored app.  
  **Why rejected/superseded:** Recreates long-term dependency on the legacy architecture.  
  **Replaced by:** Progressive replacement with bounded PlantOps engines.

## Open Questions

- What stack/database does the legacy application actually use?
- Is the complete source and build chain available?
- Does it expose an API or only direct database/manual export?
- What security/RBAC/audit capability currently exists?
- Is restoration already formally approved or still under evaluation?

## Pending Actions

- **NOW:** Audit source/build chain, database, security, API/export and ownership/support arrangements before substantial new investment.
- **NOW:** Define the minimum restoration scope and prevent unrelated feature expansion.
- **LATER:** Feed legacy events/data into PortOps/GM and replace capabilities incrementally.

## Related Streams

- [Grand MATFLOW](grand-matflow.md) — target system of record for actual material movement.
- [PlantOps Platform](plantops-platform.md) — target bounded-engine architecture.

## Evolution

The immediate reaction was frustration that the company was reviving an old application while modern architecture work was underway. The durable strategy is not direct replacement-by-argument, but controlled continuity plus integration and progressive strangling of the legacy surface.

## Sources

- ChatGPT session `current-chat-2026-08-18`
  - source_status: context-derived
  - raw_available: false
