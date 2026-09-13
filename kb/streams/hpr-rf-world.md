---
title: "Beyond ADS-B and AIS: The Free-to-Air RF World"
slug: hpr-rf-world
status: EXPLORATORY
maturity: MEDIUM
content_type: blog-candidate
created: 2026-09-13
updated: 2026-09-13
related:
  - hpr-observation-core
  - hpr-atlas
  - hpradarhq
tags:
  - hpradar
  - rf
  - sdr
  - ads-b
  - ais
  - mlat
  - remote-id
  - flarm
  - vdes
  - radiosonde
---

# Beyond ADS-B and AIS: The Free-to-Air RF World

## Thesis

HPRadar started from two obvious public-broadcast worlds: aircraft on ADS-B and vessels on AIS. But those are only two members of a much larger family of radio systems used for surveillance, identification, navigation, safety, telemetry and environmental sensing.

The architectural opportunity is not to accumulate decoders. It is to treat each receiver as an **observation node**, each protocol as an adapter, and each decoded or solved value as evidence with provenance.

That turns the product idea from an ADS-B/AIS tracker into a **multi-domain observation network**.

One distinction matters immediately: **MLAT is not another RF signal.** Multilateration is a surveillance/positioning technique that consumes synchronized observations of transponder transmissions from multiple receivers and solves a position. EUROCONTROL treats Mode S, multilateration and ADS-B as distinct but complementary surveillance technologies. [A5][A7]

```text
RF / network input
      ↓
receiver observation
      ↓
protocol adapter / solver
      ↓
HPR Observation Core
      ↓
canonical entity / event / measurement
      ↓
Atlas / APIs / wire projections
```

The canonical data model is defined separately in [HPR Observation Core](hpr-observation-core.md). This article is the exploratory domain map; the core is what prevents every new signal from becoming a new architecture.

A section-by-section evidence audit is maintained in [`../references/hpr-rf-world.md`](../references/hpr-rf-world.md), under the KB-wide [`../../meta/source-policy.md`](../../meta/source-policy.md).

---

## 1. Mode-S: aviation is larger than ADS-B

The 1030/1090 MHz surveillance environment carries more than ADS-B Extended Squitter. EUROCONTROL explicitly identifies Mode A/C, Mode S, multilateration, ADS-B and ACAS as users of the 1030/1090 MHz surveillance RF environment. [A5][A6]

For HPRadar, this means that an aircraft can still leave useful surveillance observations even when it is not directly broadcasting an ADS-B position. Mode-S observations may contribute identity or surveillance fields depending on message/reply type, and synchronized observations can feed a multilateration system. [A5][A7]

```text
Mode-S transmission / reply
        │
        ├── Receiver A
        ├── Receiver B
        ├── Receiver C
        └── Receiver D
                 ↓
              MLAT
                 ↓
          solved position
```

Therefore Mode-S belongs in the aviation ingestion family, while MLAT belongs in the `network_solved` provenance class of the HPR Observation Core.

---

## 2. UAT 978 MHz: ADS-B is not tied to one bearer

In the United States, FAA guidance recognizes both 1090 MHz Extended Squitter and 978 MHz Universal Access Transceiver as ADS-B links. UAT also supports reception of FAA services including FIS-B, while traffic services are available through the ADS-B ecosystem. [A3][A4]

UAT is low priority for a Vietnam-first deployment, but architecturally it proves an important point:

```text
aircraft surveillance != one frequency != one decoder != one object model
```

HPR should model the aircraft observation independently from the physical bearer.

---

## 3. Drone Remote ID: a direct path into the UAV domain

FAA Remote ID rules describe drones broadcasting identification and location information by radio frequency, with Wi-Fi and Bluetooth given as examples. FAA distinguishes Standard Remote ID drones from add-on broadcast modules. [U1]

EASA's direct-remote-identification rules likewise require periodic local broadcast using an open and documented transmission protocol, with data that can include operator/aircraft identity, timestamp, aircraft position and height, course, ground speed, and remote-pilot or take-off location depending on the applicable class/rule. [U2]

This makes Remote ID a much cleaner UAV source than trying to infer drones solely from ADS-B aircraft categories.

```text
ADS-B / Mode-S → conventional aviation surveillance
MLAT            → solved position from transponder observations
Remote ID       → direct local UAS identification / state broadcast
```

In HPR terms, Remote ID should be another source adapter feeding the same canonical entity/fact model.

---

## 4. FLARM, OGN and related low-altitude ecosystems

FLARM has published the FAMP Public Protocol. The protocol owner states that FAMP traffic information includes identification, position, heading and aircraft type, and that regional implementations use license-free bands including 868 MHz and 915 MHz. [U3][U4]

The Open Glider Network documents a receiver ecosystem supporting multiple low-altitude aviation protocols and sources. OGN documentation is authoritative for what the OGN project supports, but it is not a regulator or standards body; HPR must preserve that distinction. [U5]

The licensing boundary is particularly important. FLARM's own publication states that defined receive-only, non-commercial use is permitted under the public terms, while commercial or transmitting uses require separate arrangements. [U4]

Therefore FLARM is technically attractive but cannot be promoted into a commercial HPR service merely because the packets can be received.

---

## 5. ACARS: operational datalink, not a primary tracker

ICAO material describes ACARS as an aircraft-ground digital datalink used for message exchange and airline operational control, with use across VHF and satellite-linked environments. ICAO's broader communications roadmap treats ACARS as part of the air-ground communications ecosystem rather than as surveillance equivalent to ADS-B. [D1][D2]

For HPRadar, ACARS is most useful as a **secondary observation/correlation source**. It should not be normalized into a fake continuous track.

A conservative product rule is preferable:

> Extract only the minimum metadata required for legitimate correlation, engineering or research. Do not build a public archive of operational message contents by default.

That policy is architectural as well as privacy-conscious: the HPR core stores what a source *is*, not what would be convenient to display.

---

## 6. VDL Mode 2: modern air-ground datalink observation

ICAO Doc 9776 defines VDL Mode 2 as an air/ground data link compatible with the Aeronautical Telecommunication Network, and ICAO's GANP continues to include VDL Mode 2 in the operational communications roadmap. [D3][D2]

VDL2 therefore belongs beside ACARS as a datalink observation source. It may expose network/aircraft communication events useful for engineering or correlation, but it should not be mixed into the hot aircraft-position path unless a specific application justifies it.

---

## 7. HFDL: long-range datalink observations

ICAO's GANP explicitly identifies High Frequency Data Link as a datalink supporting oceanic airspace and complementing voice communications. [D2]

That gives HFDL a very different observation profile from line-of-sight ADS-B:

```text
ADS-B / Mode-S → dense local/regional surveillance
HFDL           → sparse long-range datalink observations
```

The HPR core can represent both without pretending they provide the same kind of evidence.

---

## 8. Radiosondes: moving objects plus environmental telemetry

NOAA/NWS describes radiosondes as balloon-borne instrument packages whose sensors transmit pressure, temperature, relative humidity and GPS position, typically in the 400–405.9 MHz range; winds aloft are derived by tracking the radiosonde's motion. [E1]

This is an unusually clean fit for Atlas because one source simultaneously produces:

```text
entity      → radiosonde
position    → lat/lon/altitude
motion      → movement / derived wind
telemetry   → pressure / temperature / humidity
observation → receiver + timestamp + RF evidence
```

It is one of the strongest examples of why HPR should model `Entity`, `Measurement` and provenance independently of transport protocol.

---

## 9. Direct weather-satellite reception

NOAA/NESDIS operates HRIT/EMWIN as a direct-to-user L-band broadcast from the GOES-R series. NOAA documents weather warnings, environmental charts, satellite imagery and data-collection-system products, and states that the broadcast is open-format with no NOAA fee or license requirement to receive it. [E2]

This is not another aircraft/vessel tracker feed. It is an environmental-observation source.

Its hardware profile is also different from a small 1090 MHz RTL-SDR node, so HPR should treat direct weather-satellite reception as a distinct receiver capability rather than forcing it into the current Pi appliance.

---

## 10. Maritime DSC: events rather than tracks

ITU-R M.493-16 is the current in-force recommendation for Digital Selective Calling in the maritime mobile service. US Coast Guard GMDSS guidance identifies VHF Channel 70 as a DSC channel for distress, safety and calling purposes. [M3][M4]

DSC should therefore enter HPR primarily as an **event source**, not as an AIS-like continuous track.

```text
vessel / station context
        +
DSC call / distress / safety event
        ↓
maritime event model
```

The distinction between object state and event state should remain explicit all the way to the UI.

---

## 11. NAVTEX: maritime safety context

The IMO NAVTEX Manual defines NAVTEX as an automated service for maritime safety information including navigational and meteorological warnings, forecasts and other urgent safety-related messages. [M5]

NAVTEX does not become a vessel marker. Its natural representation is an advisory/event/context object associated with time, geography or maritime operations.

This is another reason the HPR core must support more than `object + position`.

---

## 12. VDES: the maritime architecture horizon beyond AIS

ITU-R M.2092-2, approved in February 2026 and currently in force, defines the technical characteristics of the VHF Data Exchange System. Its structure includes terrestrial and satellite VDE components as well as Application Specific Message channels. [M6]

IALA describes VDES as a system spanning ships, shore stations and satellites and explains that AIS is one component alongside ASM and VDE. [M7]

For a long-lived HPR Marine architecture, the abstraction should therefore be:

```text
Maritime RF / data exchange
├── AIS
├── ASM
├── VDE terrestrial
├── VDE satellite
├── DSC
└── maritime safety information sources
```

rather than permanently equating `marine = AIS`.

---

## 13. AIS itself already proves why provenance matters

The current AIS standard is ITU-R M.1371-6 (02/2026). The recommendation's message structure includes position reports as well as ship static and voyage-related data. [M1][M2]

That creates an important semantic distinction:

```text
ship name / MMSI received in AIS message → transmitted identity
AIS destination / ETA                    → transmitted declaration
owner / corporate operator from registry → registry enrichment
normalized destination port              → external context
```

The values may look similar on screen, but their authority and freshness are not the same.

This is exactly why provenance must accompany every fact rather than being stored in a single generic `source` field at object level.

---

## 14. 406 MHz ELT, EPIRB and PLB distress beacons

NOAA SARSAT documents the 406 MHz distress-beacon ecosystem including aviation ELTs, maritime EPIRBs and personal PLBs. GNSS-capable models can include position information in the digital distress message. [E3]

ITU-R also maintains recommendation families covering COSPAS-SARSAT/EPIRB systems. [E4]

These should be modeled as **distress/safety events with identity and possibly position**, not as normal public trackers.

Technical receivability does not imply that public redistribution is appropriate. Any production use needs explicit legal, safety and operational review.

---

## 15. Amateur APRS and experimental telemetry

TAPR's APRS Protocol Reference defines the protocol used in the amateur ecosystem. APRS is useful to HPR mainly as an architectural reference for low-bandwidth identity, position, telemetry and message-bearing systems. [P1]

Any actual deployment must separately respect amateur-radio regulation and local rules; a project protocol specification is not a regulatory authorization.

---

## What HPR should not become

An SDR can physically receive many more signals: proprietary ISM telemetry, LoRa/LoRaWAN, Bluetooth activity, Wi-Fi management traffic and other systems.

**Receivable is not the same as appropriate to collect, retain or publish.**

The default HPR boundary should be:

> Prefer signals whose intended function is public/cooperative surveillance, identification, navigation, safety, meteorology or situational awareness, or whose governing specification/license clearly supports the intended receive use.

Anything outside that boundary requires a separate legal, privacy and product justification.

---

## Priority for HPRadar

From an architectural-value perspective, the strongest next candidates remain:

1. **Remote ID** — direct entry into the UAV domain. [U1][U2]
2. **Mode-S non-ADS-B + MLAT** — core aviation surveillance rather than an optional side feature. [A5][A7]
3. **FLARM / OGN family** — useful low-altitude coverage, subject to explicit licensing review. [U3][U4][U5]
4. **Radiosonde** — excellent fit for entity + position + telemetry. [E1]
5. **ACARS / VDL2 / HFDL** — secondary aviation observations and correlation. [D1][D2][D3]
6. **VDES + DSC + NAVTEX** — the natural path beyond an AIS-only marine worldview. [M3][M5][M6][M7]

This is a product/architecture priority, not a claim about decoder difficulty.

---

## Emerging HPR RF architecture

```text
                             HPR RF WORLD

        AVIATION                MARITIME              ENVIRONMENT
        ────────                ────────              ───────────
        ADS-B                   AIS                   Radiosonde
        Mode-S                  VDES                  Weather satellite
        UAT                     DSC
        Remote ID               NAVTEX
        FLARM / OGN             EPIRB
        ACARS
        VDL2
        HFDL
        ELT
             \                    |                    /
              \                   |                   /
                       receiver observations
                               ↓
                       source adapters
                               ↓
                     HPR Observation Core
                               ↓
                 canonical state + provenance
                               ↓
                   HPR wire / APIs / Atlas
```

`MLAT` deliberately does not appear as an RF bearer. It consumes observations and emits solved facts.

---

## Core consequence: normalize semantics, not protocols

The wrong long-term architecture is:

```text
ADS-B model
AIS model
Remote-ID model
FLARM model
Radiosonde model
VDES model
...
```

The durable model is:

```text
source adapter
      ↓
Observation
      ↓
Fact / Event / Measurement
      ↓
provenance-aware resolution
      ↓
Entity State
      ↓
Enrichment + Context
      ↓
projection to wire / API / UI
```

The detailed matrix and adapter contract live in [HPR Observation Core](hpr-observation-core.md). Its key design rule is simple:

> **Rows are source adapters. Columns are fixed processing stages. Adding a signal appends a row; it does not redesign the matrix.**

That is the difference between building a collection of receivers and building a core platform.

---

## Blog angle

The public article should not promise that HPRadar will ingest every receivable RF system. The stronger narrative is:

> ADS-B and AIS are two examples of a broader class of legitimate digital observations. HPRadar's opportunity is not to accumulate decoders, but to build one coherent model for objects, events, measurements and provenance across domains.

The platform should be able to answer not only **“where is this object?”** but also **“how do we know?”**

---

## Source authority

All external factual claims in this article follow the HPR KB [Source Policy](../../meta/source-policy.md). The full section-by-section audit is maintained in the [HPR RF World Source Register](../references/hpr-rf-world.md).

### Aviation surveillance

[A1]: https://www.faa.gov/air_traffic/technology/equipadsb/capabilities/ins_outs
[A2]: https://www.faa.gov/sites/faa.gov/files/air_traffic/technology/adsb/quicklinks/ADS-B_In_Strategy.pdf
[A3]: https://www.faa.gov/air_traffic/technology/equipadsb/installation
[A4]: https://www.faa.gov/air_traffic/technology/equipadsb/resources/faq
[A5]: https://www.eurocontrol.int/service/surveillance-system-and-sensors
[A6]: https://www.eurocontrol.int/service/surveillance-rf-environment-monitoring-10301090-mhz
[A7]: https://www.eurocontrol.int/publication/wide-area-multilateration-guidelines-achieving-operational-approval-wam-system

### UAV / low-altitude

[U1]: https://www.faa.gov/uas/getting_started/remote_id
[U2]: https://www.easa.europa.eu/en/document-library/easy-access-rules/online-publications/easy-access-rules-unmanned-aircraft-systems
[U3]: https://www.flarm.com/en/integration/flarm-famp-public-protocol/
[U4]: https://www.flarm.com/en/news/publishing-the-flarm-famp-public-protocol/
[U5]: https://wiki.glidernet.org/ogn-receiver-installation

### Aviation datalink

[D1]: https://www.icao.int/communications-navigation-and-surveillance-cns-section
[D2]: https://ganpportal.icao.int/ASBU/Thread
[D3]: https://store.icao.int/en/manual-on-vhf-digital-link-vdl-mode-2-doc-9776

### Maritime

[M1]: https://www.itu.int/rec/R-REC-M.1371-6-202602-I/en
[M2]: https://www.itu.int/dms_pubrec/itu-r/rec/m/R-REC-M.1371-6-202602-I%21%21TOC-HTM-E.htm
[M3]: https://www.itu.int/rec/R-REC-M.493/en
[M4]: https://www.navcen.uscg.gov/gmdss-frequently-asked-questions
[M5]: https://wwwcdn.imo.org/localresources/en/OurWork/Safety/Documents/Documents%20relevant%20to%20GMDSS/MSC.1-Circ.1403-Rev.2.pdf
[M6]: https://www.itu.int/rec/R-REC-M.2092-2-202602-I/en
[M7]: https://www.iala.int/technical/connectivity/vdes-vhf-data-exchange-system/

### Environmental / distress

[E1]: https://www.weather.gov/upperair/factsheet
[E2]: https://www.ospo.noaa.gov/operations/goes/hrit/
[E3]: https://www.sarsat.noaa.gov/emergency-406-beacons/
[E4]: https://www.itu.int/en/ITU-R/terrestrial/mars/Pages/References.aspx

### Amateur / experimental

[P1]: https://files.tapr.org/software_library/aprs/aprsspec/spec/aprs101m/APRS101m.pdf

## Status / next questions

- **EXPLORATORY:** this is a sourced technology/domain map, not an implementation commitment.
- **CANONICAL CORE:** protocol-independent semantics now live in `hpr-observation-core.md`; this article must not invent a parallel taxonomy.
- Vietnam-specific receive/store/redistribute legality still requires Vietnam-specific authoritative legal sources before any relevant public service is launched.
- Remote ID is the first new source worth a receiver/adapter scout.
- FLARM commercial licensing must be settled before public/commercial integration.
- Future signal families should pass the Observation Core adapter-conformance gate before entering production.
