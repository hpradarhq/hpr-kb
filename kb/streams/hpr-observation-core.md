---
title: "HPR Observation Core"
slug: hpr-observation-core
status: ACTIVE
maturity: HIGH
content_type: architecture-core
created: 2026-09-13
updated: 2026-09-13
related:
  - hpr-rf-world
  - hpr-atlas
  - hpradarhq
tags:
  - hpradar
  - observation
  - provenance
  - core-model
  - ads-b
  - mode-s
  - mlat
  - ais
---

# HPR Observation Core

> **Reconciliation notice — 2026-09-20:** The [draft Architecture Constitution](hpr-architecture-constitution.md) records newer session direction and proposes how to reconcile this stream. Its supersession table covers visual baseline and core extensibility/retention. Historical content below is preserved; the draft does not become approved policy until review. This stream's maturity label is not a runtime certification.

## Intent

Define one protocol-independent observation model for HPRadar so that ADS-B, Mode-S, MLAT, AIS and future sources such as Remote ID, FLARM, VDES, radiosondes or other legitimate broadcast systems can be added without redesigning the data model.

The core must answer two questions independently:

1. **What does this fact mean?** — identity, position, motion, status, declaration, telemetry, event, relationship.
2. **How do we know it?** — transmitted by the object, measured by a receiver, decoded locally, solved by a network, enriched from a registry, or supplied by external context.

The second question is provenance. Provenance is not a UI label and not a single top-level layer. It accompanies every fact.

## Current Canonical State

### Canonical rule

**Protocols are adapters. The observation model is the core.**

Adding a new signal family must normally require:

```text
new receiver / input
      ↓
new protocol adapter
      ↓
existing Observation Core
```

and must not require:

```text
new protocol
      ↓
new object model
      ↓
new database shape
      ↓
new wire semantics
      ↓
new frontend architecture
```

### Core pipeline

```text
C0 ACQUIRE
physical/network bearer
        ↓
C1 OBSERVE
immutable receiver evidence
        ↓
C2 DECODE
typed claims explicitly carried by the transmission
        ↓
C3 DERIVE / SOLVE
CPR decode, MLAT, reassembly, normalization, calculated values
        ↓
C4 RESOLVE
entity association + best-current canonical state
        ↓
C5 ENRICH
registry / relatively static external metadata
        ↓
C6 CONTEXT
external dynamic operational/environmental context
        ↓
C7 PROJECT
wire / API / storage view / Atlas UI
```

These are processing stages, not separate product databases.

## Core invariants

1. **Raw observations are immutable evidence.**
2. **Decoded claims do not become truth merely because they were transmitted.** AIS destination, for example, is a transmitted declaration and may be stale or manually entered.
3. **Derived/solved facts must reference their inputs.** MLAT position must remain traceable to the observations/receivers used by the solver.
4. **Enrichment never overwrites observed state without preserving provenance.** A registry aircraft registration and a transmitted flight identity are different facts even when they appear related.
5. **Context never rewrites core observation history.** Flight route, port intelligence, weather or airspace context are overlays on evidence, not replacements for it.
6. **Canonical state is a materialized best-current view, not the historical evidence store.**
7. **Every projected field can expose its provenance when needed.** The hot FE wire may omit verbose provenance for efficiency, but the backend model cannot lose it.
8. **A new protocol adds an adapter row, not a new taxonomy.**

## Orthogonal semantic facets

The semantic facet answers *what kind of information is this?* It is independent from source protocol and processing stage.

| Facet | Meaning | Examples |
|---|---|---|
| `identity` | identifiers and human-readable identity | ICAO24, MMSI, callsign, ship name, drone serial |
| `position` | spatial location | lat/lon, altitude, reference position |
| `motion` | movement state | ground speed, SOG, track/COG, heading, vertical rate, ROT |
| `status` | operational/status flags | squawk, emergency, air/ground, navigation status |
| `declaration` | transmitter/operator-declared intent or voyage data | AIS destination, ETA, draught |
| `telemetry` | sensed quantitative payload | temperature, humidity, pressure, signal metrics |
| `event` | discrete occurrence | distress alert, safety call, warning, network logon |
| `capability` | supported equipment/protocol capability | ADS-B version/capability, station capabilities |
| `relationship` | links among entities or observations | received-by, solved-from, belongs-to, source-station |

A future signal should map its fields into these facets. It should not create new top-level categories merely because its packet layout is different.

## Provenance envelope

Every canonical fact should be representable with an envelope equivalent to:

```json
{
  "entity_id": "aircraft:icao24:888123",
  "facet": "position",
  "key": "lat_lon",
  "value": [106.68, 20.84],
  "units": "deg",

  "origin": "network_solved",
  "protocol": "mode-s",
  "method": "mlat",

  "receiver_id": null,
  "solver_id": "hpr-mlat-01",
  "input_observation_refs": ["obs:a", "obs:b", "obs:c", "obs:d"],

  "observed_at": "2026-09-13T01:02:03.123Z",
  "valid_at": "2026-09-13T01:02:03.100Z",
  "quality": {
    "confidence": 0.94
  },

  "source_ref": "mlat-solution:xyz"
}
```

Not every storage or wire representation needs every field, but the semantic contract must be able to express them.

### Stable `origin` classes

Use a small fixed vocabulary:

- `transmitted` — explicitly carried by the emitting object/station.
- `receiver_measured` — measured locally by the receiving node, e.g. RSSI or receive timestamp.
- `decoder_derived` — calculated from one or more encoded fields at a decoder, e.g. CPR-decoded coordinates.
- `network_solved` — calculated from multiple observations/stations, e.g. MLAT.
- `registry_enriched` — external registry/database metadata.
- `context_external` — dynamic outside context such as schedule, port, weather or airspace.
- `operator_asserted` — HPR/user/operator-entered information when such workflow exists.

`origin` is deliberately independent of `facet`.

Examples:

```text
position + transmitted       → AIS lat/lon
position + decoder_derived   → ADS-B CPR decoded lat/lon
position + network_solved    → MLAT lat/lon
identity + transmitted       → ADS-B flight ID / AIS MMSI
identity + registry_enriched → registration / owner / manufacturer
```

## Canonical record types

### `Observation`

Immutable evidence that one receiver/input observed one transmission or input event.

Minimum conceptual fields:

```text
observation_id
receiver_id
received_at
bearer
protocol_hint
raw_ref / raw_hash
signal_metrics
channel / frequency when relevant
```

### `Fact`

A typed normalized claim or derived result associated with an entity/event/measurement.

### `Entity`

A durable resolved real-world object or station, for example aircraft, vessel, drone, AtoN, receiver station or radiosonde.

### `Event`

A discrete occurrence, for example a DSC distress event, 406 MHz beacon alert, network logon or safety warning.

### `Measurement`

A time/location-bound quantitative value such as temperature, pressure, humidity or RF signal level.

### `Context`

External information useful for interpretation but not produced by the observed signal itself, e.g. weather field, airport/port metadata, normalized route or airspace.

### `State`

A materialized best-current view assembled from facts according to source priority, validity, freshness and quality policy. State is a projection, not the evidence record.

## Stable source-adapter contract

Every source adapter should declare a manifest equivalent to:

```yaml
id: adsb-1090es
domain: aviation
kind: broadcast
bearer:
  medium: rf
  band: 1090MHz
protocol: ads-b-1090es
standard_refs:
  - FAA-ADS-B
produces:
  - observation
  - fact
entity_types:
  - aircraft
facets:
  - identity
  - position
  - motion
  - status
origins:
  - transmitted
  - receiver_measured
  - decoder_derived
legal_profile: public-surveillance-broadcast
```

Adding Remote ID, FLARM, VDES or radiosonde later means adding another manifest plus decoder/adapter. The core schema stays unchanged unless the new source reveals a genuinely new semantic primitive that cannot be represented by `Entity`, `Observation`, `Fact`, `Event`, `Measurement`, `Context` and the fixed facets.

## Core matrix — stage-oriented and append-only

**Rows are sources/adapters. Columns are fixed HPR processing stages.** A new signal appends a row. It does not add or reorder columns.

| Source adapter | C0 Acquire | C1 Observe | C2 Decode — transmitted claims | C3 Derive / Solve | C4 Resolve / canonical state | C5 Enrich | C6 Context | Primary authority |
|---|---|---|---|---|---|---|---|---|
| `adsb-1090es` | 1090 MHz ADS-B/Mode-S ES RF | raw message + receiver time/RF metrics | aircraft address/flight ID and broadcast surveillance data such as position/altitude/velocity/status when present | CPR position decode, validity/quality handling, normalized values | canonical aircraft identity + current kinematic/status state | registration, type, manufacturer, operator, photo | route/schedule, airport, weather, airspace | FAA ADS-B; applicable ICAO/RTCA/EUROCAE standards |
| `mode-s-1090` | 1090 MHz replies/squitters in 1030/1090 surveillance environment | raw Mode-S message + receiver time/RF metrics | address, altitude/squawk/BDS or other surveillance fields when carried/available | track correlation; provides synchronized observations usable by MLAT | aircraft entity/state even when direct ADS-B position is absent | aircraft registry/type/operator | flight/airspace/weather context | EUROCONTROL surveillance/1030–1090 material; ICAO standards |
| `mlat-mode-s` | **no new RF bearer**; consumes synchronized Mode-S observations | references multiple C1 observations | no new transmitter claim introduced by MLAT itself | TDOA/multilateration position solution + solver quality/residual/geometry metadata | candidate aircraft position with `origin=network_solved` | same aircraft enrichment as other aviation sources | same aviation context | EUROCONTROL WAM/surveillance guidance |
| `ais-vhf` | maritime VHF AIS channels | raw AIS message + receiver time/RF metrics | MMSI; dynamic reports; static/voyage-related data; AtoN/base-station fields by message type | message assembly/normalization, validity, derived movement only when explicitly calculated | vessel/AtoN/base-station canonical state | vessel registry, owner/operator, build/type details, photo | port, berth, route, weather, lane/advisory context | ITU-R M.1371-6 (02/2026) |

### Why this matrix scales

The previous temptation is to make columns such as `ICAO`, `MMSI`, `altitude`, `draught`, `weather`, `MLAT geometry`, etc. That matrix must be redesigned every time a new domain appears.

The core matrix instead fixes **processing semantics**. Protocol-specific fields live in adapter mappings under stable facets.

Therefore:

```text
add Remote ID → append one row
add FLARM     → append one row
add radiosonde→ append one row
add VDES      → append one row
```

No core-column surgery is required.

## ADS-B / MLAT / AIS semantic comparison under the core

| Semantic question | ADS-B / Mode-S | MLAT | AIS |
|---|---|---|---|
| What is directly emitted? | ADS-B/Mode-S surveillance messages | No special MLAT transmission; solver consumes existing transponder emissions | AIS VHF messages |
| Direct identity available? | ICAO24; flight ID/callsign when broadcast; other surveillance identity/status depending on message | Inherited from the Mode-S observation being solved/correlated | MMSI and message-dependent static identity fields |
| Direct position? | ADS-B carries encoded position information when equipped/transmitting | No; position is solved from multiple receiver observations | Yes, in position-report messages |
| Position origin in HPR | usually `decoder_derived` after ADS-B CPR decoding | `network_solved` | `transmitted` after message decoding |
| Receiver metadata | receive time, receiver UUID, RSSI/message metrics | participating receivers + timing/solver metadata | receive time, station UUID, RF/channel metrics where available |
| Static external enrichment | registration, type/model, manufacturer/operator, photo | same aircraft enrichment | owner/operator/registry/build data/photo |
| Dynamic external context | flight/airport/weather/airspace | same | port/route/weather/maritime advisories |

The important point is that **the same semantic field can have a different origin**. `position` is the clearest example.

## Source arbitration

Canonical state should choose among candidate facts using explicit policy, not source-name assumptions hidden in frontend code.

Conceptually:

```text
candidate facts
    ↓
validity
freshness
quality / confidence
source policy
consistency checks
    ↓
canonical state
```

For example, an aircraft can simultaneously have:

```text
ADS-B position
MLAT position
last known position
external context position estimate
```

HPR should retain all valid evidence and select one canonical display position with a traceable decision.

The canonical selector must never destroy the losing candidates.

## Wire/API consequence

AirWire or any future HPR binary protocol is a **projection of the core**, not the core itself.

For example:

```text
Core
├── rich evidence + provenance
├── canonical current state
└── projections
    ├── compact hot aircraft wire
    ├── compact vessel wire
    ├── station/status API
    ├── history API
    └── Atlas detail API
```

Therefore a compact `0x02` aircraft-position frame may contain only position/motion values while the backend still retains whether each value came from ADS-B or MLAT and which receiver/solver produced it.

## Business rules / invariants

- A source protocol must never define HPR's global object schema.
- `transmitted` means "the transmitter claimed/sent this", not "verified truth".
- `registry_enriched` and `context_external` must be distinguishable from RF evidence.
- A solver is a data producer but not necessarily a bearer. MLAT is the canonical example.
- Receiver/source quality metadata belongs to provenance, not static enrichment.
- Callsign/flight ID received over ADS-B is transmitted identity, not enrichment.
- AIS static/voyage data transmitted by a vessel is transmitted/declaration data, not registry enrichment.
- A field may change origin without changing its semantic key.
- Wire compression/serialization must not leak upward into the canonical model.
- Adding a new signal should normally be testable by an adapter contract test plus core conformance tests.

## Adapter conformance gate

Before a new source enters production it should answer:

1. What is the bearer and protocol/standard?
2. Is it `broadcast`, `reply`, `solver`, `registry`, `context`, or another explicit source kind?
3. What exact entity/event/measurement types can it produce?
4. Which stable semantic facets does it populate?
5. Which values are transmitted versus receiver-measured versus derived/solved?
6. What timestamps exist and what do they mean?
7. What quality/validity fields exist?
8. What authoritative specification defines the semantics?
9. What receive/store/redistribute license or regulatory constraints apply?
10. What canonical-state arbitration policy applies if it conflicts with another source?

If those ten questions cannot be answered, the source is not ready for the HPR canonical core.

## Source authority policy

All external protocol claims in this stream and its adapters follow [`../../meta/source-policy.md`](../../meta/source-policy.md).

For the broader RF candidate survey, see [`../references/hpr-rf-world.md`](../references/hpr-rf-world.md).

## Sources

### ADS-B / UAT

- FAA — ADS-B Ins and Outs: https://www.faa.gov/air_traffic/technology/equipadsb/capabilities/ins_outs
- FAA — ADS-B In Strategy: https://www.faa.gov/sites/faa.gov/files/air_traffic/technology/adsb/quicklinks/ADS-B_In_Strategy.pdf
- FAA — ADS-B Installation / 1090ES and UAT: https://www.faa.gov/air_traffic/technology/equipadsb/installation
- FAA — ADS-B FAQ: https://www.faa.gov/air_traffic/technology/equipadsb/resources/faq

### Mode-S / MLAT

- EUROCONTROL — Surveillance System and Sensors: https://www.eurocontrol.int/service/surveillance-system-and-sensors
- EUROCONTROL — Surveillance RF Environment Monitoring 1030/1090 MHz: https://www.eurocontrol.int/service/surveillance-rf-environment-monitoring-10301090-mhz
- EUROCONTROL — Wide Area Multilateration Guidelines: https://www.eurocontrol.int/publication/wide-area-multilateration-guidelines-achieving-operational-approval-wam-system

### AIS

- ITU-R M.1371-6 (02/2026), current in-force AIS recommendation: https://www.itu.int/rec/R-REC-M.1371-6-202602-I/en
- ITU-R M.1371-6 contents/message families: https://www.itu.int/dms_pubrec/itu-r/rec/m/R-REC-M.1371-6-202602-I%21%21TOC-HTM-E.htm

## Related Streams

- [HPR RF World](hpr-rf-world.md) — candidate free-to-air signal families and blog candidate.
- [HPR Atlas](hpr-atlas.md) — consumer of canonical objects and state projections.
- [HPRadarHQ](hpradarhq.md) — organization/control-plane context.

## Evolution

The initial ADS-B/MLAT/AIS comparison used source-specific columns and treated provenance as one layer among others. That worked for three technologies but would become unstable as Remote ID, FLARM, radiosonde, VDES and other sources were added.

Observation Core v1 replaces that approach with two independent dimensions:

```text
PROCESSING STAGE: acquire → observe → decode → derive/solve → resolve → enrich → context → project
SEMANTIC FACET:   identity | position | motion | status | declaration | telemetry | event | capability | relationship
```

Every fact carries provenance. Every protocol becomes an adapter. This is the canonical direction for future HPR multi-domain ingestion.
