---
title: "Beyond ADS-B and AIS: The Free-to-Air RF World"
slug: hpr-rf-world
status: EXPLORATORY
maturity: MEDIUM
content_type: blog-candidate
created: 2026-09-13
updated: 2026-09-13
related:
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

HPRadar began from two obvious public broadcast worlds: aircraft on ADS-B and vessels on AIS. But an SDR receiver can observe a much larger set of legitimate, openly broadcast radio systems used for identification, navigation, safety, telemetry and environmental sensing.

The useful architectural insight is not simply “add more decoders.” It is to treat each receiver as an **observation node** and each protocol as one possible source of evidence about real-world objects and events.

That changes the product idea from an ADS-B/AIS tracker into a **multi-domain RF observation network**.

One correction matters from the beginning: **MLAT is not another RF signal.** Multilateration is a positioning method. Multiple receivers timestamp the same Mode-S transmission and a solver estimates the transmitter position from time-difference-of-arrival measurements.

```text
RF broadcast
    ↓
receiver observation
    ↓
decoder / solver
    ↓
canonical object or event
    ↓
HPR Atlas
```

This stream surveys the most interesting free-to-air families beyond the current ADS-B/AIS core. It is exploratory, not yet a commitment to implement every protocol.

---

## 1. Mode-S: the aviation world is larger than ADS-B

1090 MHz contains much more than ADS-B Extended Squitter. Mode-S surveillance traffic can expose aircraft identity and surveillance state even when an aircraft is not broadcasting an ADS-B position.

Useful observations may include:

- ICAO 24-bit address
- Mode-S replies
- altitude and squawk when available
- Comm-B information for suitably interrogated aircraft
- receiver timestamp and signal level
- source/validity metadata

The important consequence for HPRadar is that **“no ADS-B position” does not mean “no observable aircraft.”**

With synchronized receivers, the same Mode-S observations can feed MLAT:

```text
Aircraft Mode-S transmission
        │
        ├── Receiver A: t1
        ├── Receiver B: t2
        ├── Receiver C: t3
        └── Receiver D: t4
                    ↓
                MLAT solver
                    ↓
                 position
```

Mode-S therefore belongs in the HPR aviation core, while MLAT belongs in the derived/solved layer.

---

## 2. UAT 978 MHz: another ADS-B ecosystem

The United States also uses 978 MHz Universal Access Transceiver (UAT) alongside 1090ES. UAT can carry aircraft surveillance traffic and also supports services such as TIS-B and FIS-B.

For a Vietnam-focused HPRadar deployment this is low priority. For a future global receiver architecture, however, it is useful evidence that the aviation ingestion layer should not be hard-coded around one physical bearer.

The HPR abstraction should be closer to:

```text
aviation observation
    ├── 1090ES ADS-B
    ├── Mode-S
    ├── UAT
    ├── MLAT-derived state
    └── other aviation datalinks
```

rather than “aircraft = 1090 MHz JSON.”

---

## 3. Drone Remote ID: probably the highest-value new domain

Broadcast Remote ID is unusually well aligned with HPRadar. Standard implementations broadcast identification and location data over local radio technologies such as Wi-Fi and Bluetooth. Depending on the implementation and jurisdiction, observations can include drone identity, drone position, altitude, velocity and a control-station or take-off location.

Conceptually this fills a large gap in low-altitude awareness:

```text
ADS-B       → conventional aviation
Mode-S/MLAT → aircraft without direct ADS-B position
Remote ID   → drones / UAS
```

For HPR Atlas this should not be forced into the ADS-B decoder. It is better represented as another observation family feeding the same canonical spatial model.

A future HPR Edge station could therefore support:

```text
1090 MHz receiver
AIS receiver
Remote ID receiver
        ↓
common station identity
        ↓
HPR object/event model
```

Remote ID is the strongest candidate for opening a genuine UAV domain rather than trying to infer UAVs only from ADS-B aircraft categories.

---

## 4. FLARM, OGN, FANET and related low-altitude aviation networks

Gliders, sailplanes, paragliders, ultralights and other light aircraft can be poorly represented in a pure ADS-B view.

The Open Glider Network ecosystem already demonstrates multi-protocol reception around FLARM, OGN/OGNTP, FANET, ADS-L and ADS-B. FLARM's public FAMP specification exposes traffic information including identification, position, heading and aircraft type, using license-free spectrum such as 868/915 MHz depending on region.

This makes the family technically attractive for HPRadar because it can improve low-altitude situational awareness where ADS-B coverage is incomplete.

However, protocol availability and commercial rights are not the same thing. FLARM's public-protocol terms permit defined receive-only non-commercial uses, while commercial use requires separate licensing. HPRadar must therefore treat licensing as part of the ingestion contract, not as an afterthought.

Potential HPR value:

```text
ADS-B / Mode-S
       +
FLARM / OGN / FANET
       +
Remote ID
       ↓
much stronger low-altitude aviation picture
```

---

## 5. ACARS: operational observations from aircraft

ACARS is not primarily a tracker. It is an airline/aircraft operational messaging system carried over several bearers, including VHF and other long-range links.

Depending on message content and network, observable metadata may help correlate:

- flight identity
- aircraft identity
- departure/arrival context
- weather reports
- maintenance/operational events
- airline operational traffic

For HPRadar, the useful role is **secondary observation and correlation**, not publishing raw message traffic.

A strong default policy would be:

> Extract only the minimum metadata needed for legitimate tracking, correlation and system research; do not build a public archive of operational/private message content.

That keeps ACARS useful without turning HPRadar into a message-sniffing product.

---

## 6. VDL Mode 2: modern aviation datalink observation

VDL Mode 2 is a major VHF air-ground datalink technology and continues to support operational datalink services including CPDLC environments.

It can expose a different class of aviation observation than ADS-B:

- aircraft/network identity
- datalink logon and network events
- operational communications metadata
- AOC/ATC datalink context where legally and appropriately processed

Again, this is not a high-rate tracking source. Its value is in **correlation, network research and operational context**.

For HPR architecture, VDL2 belongs beside ACARS as an observation source, not inside the hot aircraft-position wire.

---

## 7. HFDL: long-range aviation observations beyond VHF line of sight

High Frequency Data Link extends aircraft datalink communications into regions where VHF coverage is absent or impractical. HF propagation can allow a ground station to receive aircraft observations over very long distances.

This creates an interesting complement:

```text
ADS-B / Mode-S → precise local/regional line-of-sight surveillance
HFDL           → sparse long-range operational observations
```

HFDL is not a replacement for ADS-B and should not be normalized as if it were a continuous track. It is evidence about an aircraft at a point in time and should retain that provenance.

---

## 8. Radiosondes: moving objects plus atmospheric telemetry

Weather balloons are exceptionally compatible with the Atlas object model because they are both moving spatial objects and environmental sensors.

A radiosonde observation can contain or enable:

- sonde identity
- latitude/longitude
- altitude
- temperature
- pressure
- humidity
- movement-derived wind information

This produces a very natural HPR object:

```text
Radiosonde
├── identity
├── position / altitude / velocity
├── receiver observation
└── atmospheric payload
    ├── temperature
    ├── pressure
    └── humidity
```

Unlike a decorative weather overlay, this is a physically observed sensor moving through the atmosphere.

Radiosondes therefore offer one of the cleanest ways to connect HPR tracking with environmental sensing.

---

## 9. Direct weather-satellite reception

Weather satellites are a different category. They are not primarily another moving-object feed for Atlas; their value is direct environmental observation.

Several meteorological satellite systems provide direct-broadcast services intended for receiving stations. These can deliver imagery or instrument products without relying on a commercial tracking API.

The hardware and bandwidth requirements are often far beyond a simple RTL-SDR edge node, so this should be treated as a future sensor class rather than bundled into the current Pi appliance.

Conceptually:

```text
moving-object RF     → ADS-B, AIS, Remote ID, FLARM
atmospheric telemetry → radiosondes
earth observation     → meteorological satellites
```

That is a much broader view of what Atlas can eventually represent.

---

## 10. Maritime DSC: event data rather than continuous tracks

Digital Selective Calling (DSC) is part of maritime safety and calling infrastructure. Depending on message type it can carry information such as station identity, call category and distress/urgency/safety events, with position information available in relevant cases.

This should not be modeled like AIS position streaming.

Its HPR value is an **event layer**:

```text
AIS vessel state
       +
DSC safety/distress event
       ↓
richer maritime situational context
```

The distinction between object state and safety event is important for both data semantics and the user interface.

---

## 11. NAVTEX: maritime safety context

NAVTEX broadcasts maritime safety information such as navigational warnings, meteorological warnings and related safety information.

NAVTEX does not identify or track vessels. Its natural role in HPR is contextual:

```text
vessel tracks
shipping context
weather
NAVTEX warning
```

This is an example of why Atlas should not reduce every RF source to “another marker on the map.” Some RF observations are objects; others are events, warnings, measurements or areas of relevance.

---

## 12. VDES: the maritime future beyond AIS

VDES — the VHF Data Exchange System — is particularly important for a long-lived HPR Marine architecture.

VDES encompasses AIS, Application Specific Messages (ASM) and higher-capacity VHF Data Exchange links, including terrestrial and satellite components. AIS is therefore better understood as one component of a broader maritime digital communication system rather than the final form of maritime RF data.

For HPR, that suggests the durable abstraction:

```text
Maritime RF
├── AIS
├── ASM
├── VDE terrestrial
├── VDE satellite
├── DSC
└── maritime safety broadcasts
```

not simply:

```text
AIS decoder → ship marker
```

ITU-R Recommendation M.2092-2, approved in February 2026, is a strong signal that VDES should be treated as a real architecture horizon rather than a speculative side topic.

---

## 13. 406 MHz ELT, EPIRB and PLB distress beacons

The 406 MHz Cospas-Sarsat ecosystem includes:

- ELT — Emergency Locator Transmitter for aviation
- EPIRB — Emergency Position-Indicating Radio Beacon for maritime use
- PLB — Personal Locator Beacon

These beacons carry a unique digital identity, and GNSS-enabled devices can include position information in the distress message.

They are not trackers. They are **distress events with identity and possibly position**.

If HPR ever ingests this class, it must be treated as a specialized SAR/emergency layer with appropriate legal, ethical and operational safeguards. Public visualization by default would be the wrong design assumption.

---

## 14. Amateur APRS and experimental telemetry

APRS can broadcast callsign, position, speed/course, telemetry, weather and messages. It is technically attractive because it resembles a generic low-bandwidth object/telemetry network and is widely used for amateur stations, vehicles, balloons and experimental payloads.

It is a useful architectural reference and potential research source, but any production use must respect amateur-radio rules, local regulation and community norms.

---

## What HPR should *not* become

An SDR can receive many other transmissions: proprietary ISM telemetry, LoRa/LoRaWAN signals, Bluetooth devices, Wi-Fi management traffic and many other radio systems.

The fact that a signal is physically receivable does **not** automatically make it appropriate to collect, persist or publish.

A useful HPR policy boundary is:

> Prefer broadcasts whose intended purpose is public identification, navigation, safety, meteorology or situational awareness, or whose protocol/license explicitly supports the intended receive use.

This keeps HPRadar on the side of legitimate observation infrastructure rather than indiscriminate RF surveillance.

---

## Priority for HPRadar

If only a small number of new domains are pursued, the best candidates are:

1. **Drone Remote ID** — highest strategic value; opens the UAV domain directly.
2. **Mode-S non-ADS-B + MLAT** — not optional long-term; this belongs in the aviation core.
3. **FLARM / OGN / FANET / ADS-L** — fills important low-altitude aviation gaps, subject to licensing.
4. **Radiosonde** — unusually clean fit between tracking and environmental telemetry.
5. **ACARS / VDL2 / HFDL** — useful secondary aviation observations and correlation, not primary position feeds.
6. **VDES + DSC + NAVTEX** — evolves HPR Marine beyond an AIS-only worldview.

This order is based on architectural value to HPR, not merely on how easy each decoder is to run.

---

## Emerging HPR RF architecture

```text
                             HPR RF WORLD

        AVIATION                MARITIME              ENVIRONMENT
        ────────                ────────              ───────────
        ADS-B                   AIS                   Radiosonde
        Mode-S                  VDES                  Weather satellite
        UAT                     DSC
        FLARM / OGN             NAVTEX
        FANET                   EPIRB
        Remote ID
        ACARS
        VDL2
        HFDL
        ELT
             \                    |                    /
              \                   |                   /
                       receiver observations
                               ↓
                    decoder / solver / normalizer
                               ↓
                     HPR canonical object model
                               ↓
                           HPR Atlas
```

`MLAT` deliberately does not appear as an RF bearer in this diagram. It is a solver that turns synchronized Mode-S observations from multiple receivers into a derived aircraft position.

---

## Architectural consequence: normalize semantics, not protocols

The long-term mistake would be to create a separate end-to-end product stack for every signal:

```text
ADS-B app
AIS app
Remote-ID app
FLARM app
Radiosonde app
...
```

A stronger HPR model is:

```text
physical signal
      ↓
protocol-specific decoder
      ↓
observation with provenance
      ↓
canonical object / event / measurement
      ↓
shared HPR wire + APIs
      ↓
Atlas surfaces
```

The canonical model should preserve the distinction between:

- what the transmitter explicitly broadcast;
- what the local decoder derived;
- what a network solver such as MLAT calculated;
- which receiver observed it;
- what static enrichment came from an external database;
- what contextual intelligence came from another system.

That provenance becomes more important, not less, as HPR adds sources.

The product should be able to say not only **“where is this object?”** but also **“how do we know?”**

---

## Blog angle

A public article should avoid presenting this as a promise to ingest every radio system. The stronger narrative is:

> ADS-B and AIS are only two examples of a much larger class of legitimate free-to-air digital observations. HPRadar's opportunity is not to accumulate decoders, but to build one coherent model for objects, events, measurements and provenance across domains.

That is the point where HPRadar stops being merely an aircraft-and-ship tracker and starts becoming a **multi-domain observation platform**.

---

## References

- FAA — Remote Identification of Drones: https://www.faa.gov/uas/getting_started/remote_id
- FLARM — FAMP Public Protocol: https://www.flarm.com/en/integration/flarm-famp-public-protocol/
- FLARM — Publishing the FAMP Public Protocol and licensing notes: https://www.flarm.com/en/news/publishing-the-flarm-famp-public-protocol/
- Open Glider Network — Receiver ecosystem: https://wiki.glidernet.org/ogn-receiver-installation
- EUROCONTROL — Datalink / CPDLC material: https://www.eurocontrol.int/publication/controller-pilot-datalink-communications-cpdlc-recommended-practices
- EUROCONTROL — Datalink Performance and Capacity Analysis: https://www.eurocontrol.int/publication/eurocontrol-datalink-performance-and-capacity-analysis-2024-edition
- IALA — VHF Data Exchange System (VDES): https://www.iala.int/technical/connectivity/vdes-vhf-data-exchange-system/
- ITU-R — Recommendation M.2092-2 (02/2026): https://www.itu.int/rec/R-REC-M.2092
- NOAA SARSAT — 406 MHz emergency distress beacons: https://www.sarsat.noaa.gov/emergency-406-beacons/

## Status / Next Questions

- **EXPLORATORY:** this is a technology/domain map, not yet an implementation roadmap.
- Define a formal HPR taxonomy for `object`, `event`, `measurement`, `observation` and `context` before adding many new protocols.
- Extend the ADS-B / MLAT / AIS data-layer matrix into the candidate RF families only after the taxonomy is stable.
- Scout Remote ID receiver implementations first.
- Evaluate FLARM commercial licensing before any public HPR service integration.
- Decide which sources belong on Edge, which belong only at aggregator level, and which should remain research-only.
- Preserve provenance in every future HPR wire contract.