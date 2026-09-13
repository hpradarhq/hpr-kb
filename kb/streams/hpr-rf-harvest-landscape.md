---
title: "HPR Free-to-Harvest RF Landscape"
slug: hpr-rf-harvest-landscape
status: EXPLORATORY
maturity: MEDIUM
content_type: research-landscape
created: 2026-09-13
updated: 2026-09-13
related:
  - hpr-observation-core
  - hpr-rf-world
  - hpr-atlas
  - hpradarhq
tags:
  - rf
  - sdr
  - spectrum
  - passive-radar
  - antenna
  - receiver
  - economics
  - hpradar
---

# HPR Free-to-Harvest RF Landscape

## Intent

Build a durable big picture of radio-frequency information that HPRadar can **passively receive, decode, solve or infer without owning the transmitter**.

The useful unit is not “a frequency”. It is a harvest chain:

```text
RF source
  ↓
propagation + antenna
  ↓
RF front-end / receiver
  ↓
IQ / packet / timing observation
  ↓
decoder / solver / detector
  ↓
HPR Observation Core
  ↓
canonical state + provenance
  ↓
Atlas / API / analytics / commercial product
```

A signal is attractive to HPR only when the whole chain is viable.

## What “free to harvest” means

“Free” has four independent meanings and HPR must never collapse them:

1. **Free to receive** — a receiver can legally listen without HPR transmitting.
2. **Free/open protocol** — semantics are publicly documented or reliably implementable.
3. **Free/open software** — usable decoder/DSP implementation exists under acceptable license.
4. **Free to redistribute/commercialize** — received or derived data may be republished under applicable law, license and privacy rules.

A source can score well on the first three and fail the fourth. FLARM is a clear example: the protocol owner publishes FAMP and permits defined receive-only non-commercial uses, while commercial use requires separate arrangement. [F1]

In Vietnam, the Radio Frequency Department has stated that receive-only radio equipment can be used without a frequency-use licence, and current Vietnamese rules contain a licence-exemption framework for radio equipment. That does **not** by itself grant rights to decrypt, retain, republish or sell every received payload. Those are separate legal/data-governance questions. [VN1][VN2]

## RF harvestability model

Every candidate source should be scored on the same dimensions:

| Dimension | Question |
|---|---|
| `public_intent` | Was the transmission intended for broad identification, safety, navigation, telemetry or public reception? |
| `legal_receive` | Is receive-only operation lawful at the deployment location? |
| `redistribution` | Can decoded/derived data be stored, shared and commercialized? |
| `protocol_openness` | Is the protocol standardized, public or documented by its owner? |
| `payload_value` | Does it produce identity, position, motion, event or environmental data? |
| `coverage_value` | Does it fill a geographic/altitude/domain gap not already covered by HPR? |
| `receiver_cost` | Can it be harvested with commodity RF hardware? |
| `antenna_complexity` | Can an unattended practical antenna cover it? |
| `decoder_maturity` | Is mature open-source decoding/DSP available? |
| `network_effect` | Does adding more HPR stations improve value non-linearly? |
| `hpr_uniqueness` | Does HPR gain something more defensible than a commodity public feed? |

This score belongs beside each future adapter manifest, not in frontend code.

## Spectrum big picture

Approximate bands below describe practical receive targets; exact allocations, channel plans and legal conditions vary by country and service.

| RF region | Attractive harvest families | Typical HPR value | Typical antenna/front-end |
|---|---|---|---|
| HF, roughly 2–30 MHz | HFDL, NAVTEX at MF/HF edges, amateur/APRS-like experiments, propagation monitoring | long-range sparse observations, maritime safety context | active loop / long wire, HF preselector, HF-capable SDR |
| VHF air, roughly 118–137 MHz | ACARS, VDL2 | aircraft datalink observation/correlation | VHF vertical/discone or tuned antenna; RTL/Airspy/SDRplay |
| VHF marine, roughly 156–163 MHz | AIS, DSC, VDES/ASM | vessels, AtoN, maritime events | marine VHF vertical, band-pass filter, RTL/Airspy |
| VHF broadcast, 88–108 MHz | FM broadcast as illuminator of opportunity | passive bistatic radar / propagation research | directional reference + surveillance antenna; coherent dual-channel receiver |
| UHF meteo, ~400–406 MHz | radiosonde; 406 MHz distress-beacon ecosystem | environmental tracks; SAR event research | quarter-wave/turnstile/Yagi, LNA/filter, RTL/Airspy |
| ISM/SRD, ~433/868/915 MHz | weather/environment sensors, FLARM/FANET/OGN family by region | low-altitude aircraft and environmental telemetry | band-specific vertical/Yagi, RTL/Soapy SDR |
| aviation L-band, 978/1030/1090 MHz | UAT, SSR/Mode-S/ADS-B/ACAS observations | aircraft identity/state, MLAT input | tuned 978/1090 antenna, LNA + SAW/BPF, RTL-class receiver |
| satellite/mobile L-band, ~1.5–1.7 GHz | satcom research, GOES HRIT/EMWIN example, GNSS signals | environmental products, future passive sensing | patch/dish/helix + low-noise front-end |
| 2.4 / 5 GHz | Direct Remote ID over Wi-Fi/Bluetooth; Wi-Fi as possible passive-radar illuminator | UAV identity/state; experimental passive sensing | commodity Wi-Fi/BLE chipset or wideband SDR; directional arrays for radar |

### Important known frequencies

- **ADS-B / Mode-S**: 1090 MHz downlink environment; Mode-S interrogation uses 1030 MHz. EUROCONTROL treats 1030/1090 MHz as a shared surveillance resource for Mode A/C, Mode S, MLAT/WAM, ADS-B and ACAS. [A1][A2]
- **UAT**: 978 MHz in the United States. [A3]
- **VDL Mode 2**: VHF aeronautical band; 136.975 MHz is the common signalling channel in current European deployment, with additional channels in the 136.675–136.900 MHz range. [A4]
- **AIS**: AIS 1 = 161.975 MHz and AIS 2 = 162.025 MHz are standard operational channels; protocol semantics are defined by ITU-R M.1371-6 (02/2026). [M1][M2]
- **DSC**: marine VHF channel 70 = 156.525 MHz. [M3]
- **NAVTEX**: 518 kHz international; 490 kHz and 4209.5 kHz for national NAVTEX services under IMO guidance. [M4]
- **Radiosonde**: NOAA/NWS documents typical operation from 400 to 405.9 MHz. [E1]
- **FLARM FAMP**: protocol owner documents license-free 868 MHz / 915 MHz regional bands. [F1]
- **Remote ID**: FAA describes local RF broadcast using Wi-Fi/Bluetooth examples; EASA describes open/direct periodic broadcast and references 2.4/5 GHz use. [U1][U2]
- **GOES HRIT/EMWIN**: 1694.1 MHz L-band, open format, no NOAA fee/licence to receive within coverage. This is a useful free-to-receive model, though it is not a Vietnam-local service. [E2]

## Harvest classes

### H0 — Public cooperative surveillance/identification

Best class for HPR.

Examples:

- ADS-B / Mode-S
- AIS / AtoN AIS
- Remote ID
- FLARM/FAMP where licence permits
- radiosonde telemetry

Properties:

```text
transmitter intends to expose useful identity/state
+ low receiver cost
+ mature decoders
+ strong map value
+ simple provenance
```

This is HPR’s natural foundation.

### H1 — Public safety and advisory broadcasts

Examples:

- DSC safety/distress calls
- NAVTEX maritime safety information
- 406 MHz distress-beacon ecosystem
- weather satellite products

These should normally become `Event`, `Measurement` or `Context`, not another moving-object track.

### H2 — Operational datalinks observable over RF

Examples:

- ACARS
- VDL Mode 2
- HFDL
- some satellite aeronautical datalink

Technically harvestable and supported by strong open-source tooling, but content may include operational or private communications. HPR should default to **metadata minimization**: identity/correlation/network statistics where legitimate, rather than public mirroring of raw message content.

ICAO and EUROCONTROL treat ACARS, VDL2 and HFDL as communications/datalink technologies, not surveillance equivalents to ADS-B. [D1][D2]

### H3 — Opportunistic sensing / passive radar

Passive radar does not decode a cooperative target message. It uses an existing transmitter as an **illuminator of opportunity** and observes reflections from targets.

Typical illuminators studied in the literature include:

- FM broadcast
- DAB
- DVB-T
- GSM/LTE and other communication emitters
- sometimes satellite downlinks

A conventional passive bistatic setup has two coherent signal paths:

```text
known transmitter
   ├── direct path → reference channel
   └── target reflection → surveillance channel
                         ↓
                  cross-correlation / CAF
                         ↓
                  bistatic delay + Doppler
                         ↓
                 detection / tracking
```

Academic literature confirms FM, DVB-T, DAB and cellular signals as common illuminators of opportunity and emphasizes coherent multi-channel reception, clutter/direct-path cancellation and delay-Doppler processing. [P1][P2][P3]

This is strategically important because it can observe **non-cooperative objects** that transmit nothing useful themselves.

### H4 — ISM/SRD environmental and device telemetry

`rtl_433` demonstrates how much structured telemetry exists around 315/345/433/868/915 MHz: weather sensors, thermometers, rain gauges, air-quality devices and many other low-power devices. [S1]

This is technically easy but strategically dangerous if HPR indiscriminately harvests private household/device traffic. The HPR rule should be:

> Environmental/public infrastructure telemetry may be considered; private consumer/security/device traffic is excluded by default even if technically decodable.

### H5 — Natural and propagation RF sensing

Not every useful RF observation contains a protocol. Future research can include:

- lightning/sferics detection
- meteor forward scatter using broadcast transmitters
- ionospheric/propagation measurements
- GNSS reflectometry / opportunistic environmental sensing

These produce measurements, not communications content, and fit the Observation Core well. They are research-grade for HPR today, not product commitments.

## Antenna architecture

A single “wideband antenna” is useful for scouting but is not the right production architecture.

### Discovery antenna

Use a discone/log-periodic/wideband vertical for:

- spectrum reconnaissance
- identifying local emitters
- testing new decoders

Advantage: broad coverage.

Penalty: lower gain and poorer rejection than a tuned production antenna.

### Production antenna

For valuable continuous feeds, prefer dedicated RF chains:

```text
antenna tuned to service
  ↓
lightning/static protection
  ↓
band-pass / SAW filtering
  ↓
LNA near antenna when link budget benefits
  ↓
low-loss coax / bias tee
  ↓
SDR / receiver
```

Examples:

- 1090 MHz vertical/collinear + 1090 BPF/LNA for ADS-B.
- 162 MHz marine VHF vertical for AIS.
- 400 MHz vertical/Yagi for radiosonde.
- active loop or long-wire/preselector for HF/HFDL/NAVTEX.
- Wi-Fi/BLE interfaces for Remote ID rather than forcing everything through SDR.

### Coherent arrays

Passive radar and direction finding change the hardware requirement:

```text
ordinary decoder: frequency/time accuracy is enough
MLAT network:      accurate common/disciplined time across stations
DoA array:         phase coherence across local channels
passive radar:     coherent reference + surveillance channels
```

Open-source KrakenSDR tooling demonstrates coherent multi-channel direction finding and a passive-radar path; other research projects use synchronized RTL-SDR, HackRF, SDRplay or USRP hardware. [P4][P5]

## Receiver architecture

HPR should not standardize on one SDR. Standardize on a receiver capability interface.

### R0 — protocol-native commodity radios

Examples: Wi-Fi/BLE chipset for Remote ID.

Best when commodity hardware already demodulates the PHY cheaply and reliably.

### R1 — low-cost narrow/wide SDR

Examples: RTL-SDR-class receivers.

Best for:

- ADS-B/Mode-S
- AIS
- radiosonde
- ACARS/VDL2
- many ISM/SRD signals

This tier creates the strongest feeder economics.

### R2 — higher-dynamic-range SDR

Airspy/SDRplay-class hardware can improve crowded-band performance, instantaneous bandwidth and weak-signal reception.

### R3 — coherent multi-channel SDR

KrakenSDR/USRP/synchronized receivers for:

- DoA
- passive radar
- local interferometry
- research-grade multichannel observation

### R4 — dedicated satellite/HF front-end

Dish/helix/patch, LNB/LNA/downconverter or HF preselection as required. The receiver is only one part of the RF system.

## Open-source decoder/DSP landscape

| Domain | Open-source project | Role | HPR note |
|---|---|---|---|
| ADS-B / Mode-S | `wiedehopf/readsb` | high-performance decoder/tracker | already central to HPR Edge; strong C hot path [O1] |
| UAT 978 | `flightaware/dump978` | UAT demodulator/decoder | mature US-specific adapter [O2] |
| AIS | `jvde-github/AIS-catcher` | dual-channel AIS receiver, NMEA/network output | active project, GPLv3, broad SDR support [O3] |
| Remote ID | `opendroneid/opendroneid-core-c` | ASTM/ASD-STAN message encode/decode | Apache-2.0; good decoder primitive, PHY capture still separate [O4] |
| radiosonde | `projecthorus/radiosonde_auto_rx` | scanning, detection, decode, upload | mature continuous receiver pattern; Docker supported [O5] |
| ACARS | `f00b4r0/acarsdec` | multichannel ACARS decoder | active 2026, JSON/MQTT outputs [O6] |
| VDL2 | `TLeconte/vdlm2dec` | VDL Mode 2 decoder | useful adapter/reference [O7] |
| HFDL | `szpajder/dumphfdl` | multichannel HFDL decoder | strong HF aviation adapter [O8] |
| satcom ACARS | `jontio/JAERO` | Aero/SatCom ACARS demod/decoder | useful research source around 1.5 GHz [O9] |
| generic ISM/SRD | `merbanan/rtl_433` | hundreds of low-power device decoders | use selectively; privacy boundary required [S1] |
| satellite | `SatDump/SatDump` | general satellite DSP/decoding pipelines | strong research/edge building block [O10] |
| general SDR | GNU Radio / GNU Radio 4, SDRangel, OpenWebRX | DSP framework, receiver UI, experimentation | scout/prototype layer; not necessarily production hot path [O11][O12][O13] |
| passive radar | `gr-radar`, KrakenSDR passive radar, `30hours/blah2` | radar DSP/research implementations | excellent prototypes/donors; HPR needs its own validated tracker contract [P4][P5][P6] |

Open source dramatically lowers decoder R&D cost. It does **not** eliminate the product work around calibration, RF site engineering, feeder identity, timestamps, observability, licensing and data quality.

## Passive radar as an HPR capability

Passive radar deserves its own branch of the architecture because it changes HPR from “decode what objects tell us” to “sense objects that may tell us nothing.”

### What it can add

- independent aircraft detection to compare with ADS-B/MLAT;
- possible detection of non-cooperative aircraft/UAVs depending on geometry, illuminator and RCS;
- sea-surface/ship experiments near strong coastal illuminators;
- RF-coverage and propagation research.

### What it does not magically solve

Passive radar is not “two SDRs and a map”. Performance depends on:

- illuminator geometry and EIRP;
- waveform bandwidth/ambiguity function;
- coherent receiver phase/time stability;
- direct-path and multipath suppression;
- clutter cancellation;
- antenna isolation/patterns;
- target RCS and bistatic geometry;
- calibration and ground truth;
- tracker quality after delay-Doppler detection.

DVB-T is attractive in the literature because digital bandwidth can provide much better range resolution than narrowband FM; FM can provide wide coverage but comparatively poor range resolution. [P2][P3]

For HPR, passive radar should begin as a **cross-validation sensor** alongside ADS-B/AIS ground truth, not as a promise of military-grade primary radar.

## Economics

### The transmitter is free; the sensor network is not

HPR pays nothing to operate ADS-B aircraft transmitters, AIS ship transmitters, FM broadcast towers or radiosondes. The economic cost moves to:

```text
site access
+ antenna / mast / lightning protection
+ SDR + filters + LNA
+ edge compute
+ power
+ timing/synchronization
+ internet/backhaul
+ maintenance
+ calibration
+ data governance
```

The cheap SDR is often the smallest part of lifetime cost.

### Approximate investment tiers

These are order-of-magnitude architecture bands, not vendor quotes:

| Tier | Typical node | Economic character |
|---|---|---|
| `E0` | one RTL-class SDR + tuned antenna + existing Pi | tens to low hundreds USD incremental hardware; excellent community feeder economics |
| `E1` | multi-radio Edge: ADS-B + AIS + radiosonde/other | low hundreds USD; high information density per site |
| `E2` | higher-grade filtered SDR + GPS timing + rugged outdoor RF | hundreds to low thousands USD; semi-professional station |
| `E3` | coherent multichannel/DoA/passive radar | typically higher hundreds to several thousands USD plus antennas/site engineering |
| `E4` | satellite dish / large coherent arrays / research instrumentation | site-specific; economics dominated by RF mechanics and engineering rather than decoder software |

### Where value compounds

A single feeder gives local coverage. A network gives:

- deduplication and best-source selection;
- MLAT/TDOA;
- receiver-quality comparison;
- propagation and coverage models;
- anomaly detection;
- resilient multi-source tracks;
- passive-radar calibration against cooperative tracks;
- historical RF intelligence.

Therefore HPR’s defensible asset is not a decoder binary. It is the **network + calibrated provenance + historical observation corpus**.

## HPR niche

HPR should not try to beat FlightRadar24, FlightAware or MarineTraffic by displaying the same commodity aircraft/ship dots.

A more defensible niche is:

> **A Southeast-Asia-first, RF-native, multi-domain observation network that owns the path from antenna to canonical evidence.**

### Niche 1 — Edge appliance as a universal observation node

One HPR Edge identity, but pluggable RF modules:

```text
HPR Edge
├── ADS-B / Mode-S
├── AIS / VDES-ready maritime
├── Remote ID
├── radiosonde
├── optional ACARS / VDL2 / HFDL metadata
├── spectrum health
└── later coherent sensing / passive radar
```

The modules emit the HPR Observation Core, not protocol-specific product silos.

### Niche 2 — Low-altitude and coastal gaps

Vietnam and Southeast Asia are naturally interesting for:

- dense coastal shipping;
- ports and anchorages;
- airports close to dense urban/coastal terrain;
- UAV growth;
- mountains/islands with RF shadowing;
- weather and tropical propagation;
- feeder sites on rooftops, industrial locations and islands.

Remote ID + FLARM-like low-altitude sources + ADS-B/MLAT + AIS is more differentiated than another ADS-B aggregator.

### Niche 3 — Evidence provenance

Atlas should be able to answer:

```text
Where is it?
What is it?
How fresh is the state?
Who/what observed it?
Was this transmitted, decoded, solved or externally enriched?
How confident are we?
What other sensors agree?
```

That is far more valuable to serious users than a visually attractive marker alone.

### Niche 4 — Passive validation, not passive-radar marketing

Use passive radar first to create an independent evidence layer:

```text
ADS-B says aircraft here
MLAT solves aircraft here
passive radar sees delay-Doppler target here
multiple HPR receivers agree
```

The commercial value is **confidence and anomaly detection**, not a premature claim that HPR has built primary radar from cheap dongles.

### Niche 5 — RF health as a product

A distributed receiver network inherently measures:

- packet/message density;
- RF noise floor;
- congestion;
- receiver gain/health;
- interference events;
- coverage degradation;
- propagation anomalies.

EUROCONTROL itself treats monitoring of the 1030/1090 MHz environment as critical to surveillance performance. HPR can apply the same idea at community/industrial scale without pretending to be an ANSP. [A1]

## Recommended HPR priority

### P0 — keep strengthening

1. ADS-B / Mode-S Edge core.
2. AIS Edge core.
3. common station identity, timing and provenance.
4. RF health metrics.

### P1 — high-value additions

1. Remote ID receiver adapter.
2. radiosonde adapter.
3. low-altitude FLARM/FAMP/OGN research with licensing gate.
4. VDL2/ACARS/HFDL metadata-only research profile.

### P2 — strategic research

1. coherent DoA.
2. two-channel FM/DVB-T passive radar with ADS-B as ground truth.
3. multi-station TDOA beyond current MLAT use.
4. selected satellite/environmental direct-broadcast reception.

### P3 — avoid by default

- private Wi-Fi/Bluetooth payload collection unrelated to Remote ID;
- cellular subscriber/user traffic;
- encrypted/private communications;
- consumer security-device tracking;
- indiscriminate `rtl_433` harvesting;
- publishing raw ACARS/VDL/HFDL operational communications without a clear lawful purpose and minimization policy.

## Architectural conclusion

The HPR opportunity is not “all RF that an SDR can decode.”

It is narrower and stronger:

```text
legitimate free-to-receive RF
        +
open/controlled decoder adapters
        +
cheap distributed receivers
        +
strong provenance/timing
        +
network-derived sensing
        +
canonical HPR Observation Core
        =
independent multi-domain situational evidence
```

The more signals HPR adds, the less the product should care about protocol names. The durable product is the observation network.

## Sources

Detailed source register: [`../references/hpr-rf-harvest-landscape.md`](../references/hpr-rf-harvest-landscape.md).

Authority policy: [`../../meta/source-policy.md`](../../meta/source-policy.md).
