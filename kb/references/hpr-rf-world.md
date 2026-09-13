# HPR RF World — Source Register

This register is the publication evidence map for [`../streams/hpr-rf-world.md`](../streams/hpr-rf-world.md). Source quality follows [`../../meta/source-policy.md`](../../meta/source-policy.md).

## Coverage rule

The RF-world article is not considered publishable unless every technical section below has current authoritative coverage. Tier 1 standards/regulators are preferred. Protocol-owner or project documentation is used only where it is the correct authority for that claim.

| Article topic | Primary evidence | Authority | Coverage |
|---|---|---|---|
| ADS-B / 1090ES | FAA ADS-B pages and ADS-B In Strategy | PRIMARY-REGULATOR | Position, altitude, velocity/ground speed, identification, broadcast nature, 1090ES |
| UAT 978 MHz | FAA ADS-B Installation / FAQ | PRIMARY-REGULATOR | 978 MHz UAT, ADS-B, TIS-B/FIS-B capabilities |
| Mode-S / 1030–1090 MHz environment | EUROCONTROL Surveillance System and Sensors; SurRF monitoring | PRIMARY-REGULATOR | Mode S, ADS-B, ACAS, multilateration coexistence and surveillance role |
| MLAT / WAM | EUROCONTROL WAM guidance and surveillance material | PRIMARY-REGULATOR | MLAT as surveillance/positioning derived from multiple receiver observations; not a separate aircraft broadcast protocol |
| AIS | ITU-R M.1371-6 (02/2026) | PRIMARY-STANDARD | Current AIS technical characteristics; message families include position reports and static/voyage-related data |
| Remote ID | FAA Remote ID; EASA UAS rules | PRIMARY-REGULATOR | Local RF broadcast, identity/location/control-station or take-off information; open/documented direct broadcast in EU rules |
| FLARM FAMP | FLARM FAMP Public Protocol and license announcement | PROTOCOL-OWNER | Identity, position/motion data, 868/915 MHz regional use, receive-only/non-commercial license boundary |
| OGN ecosystem | Open Glider Network receiver documentation | PROJECT-AUTHORITY | What OGN receivers/ecosystem support; not used as regulatory authority |
| ACARS | ICAO GANP / ICAO material | PRIMARY-STANDARD / PRIMARY-REGULATOR | ACARS as aircraft-ground digital datalink; VHF/SATCOM and operational messaging context |
| VDL Mode 2 | ICAO Doc 9776 catalog/CNS material and GANP | PRIMARY-STANDARD | VDL2 as air/ground datalink and ATN-capable bearer |
| HFDL | ICAO CNS/GANP material | PRIMARY-STANDARD | HFDL as oceanic/remote datalink complement |
| Radiosonde | NOAA/NWS Radiosonde Observation | PRIMARY-REGULATOR | 400–405.9 MHz typical telemetry, pressure, temperature, humidity, GPS position, wind derivation |
| Weather satellite direct broadcast | NOAA/NESDIS OSPO HRIT/EMWIN | PRIMARY-REGULATOR | Direct-to-user L-band broadcast, open format, imagery/warnings/products |
| DSC | ITU-R M.493-16; USCG GMDSS FAQ | PRIMARY-STANDARD / PRIMARY-REGULATOR | Digital selective calling, distress/safety/calling role, VHF Ch 70 in US GMDSS guidance |
| NAVTEX | IMO MSC.1/Circ.1403/Rev.2 | PRIMARY-STANDARD | Maritime safety information, navigational/met warnings and urgent safety messages |
| VDES | ITU-R M.2092-2 (02/2026); IALA VDES overview | PRIMARY-STANDARD / DOMAIN-AUTHORITY | Current VDES technical characteristics; AIS/ASM/VDE terrestrial/satellite relationship |
| 406 MHz ELT/EPIRB/PLB | NOAA SARSAT; ITU maritime recommendations | PRIMARY-REGULATOR / PRIMARY-STANDARD | Distress-beacon classes, digital alerting and GNSS-location capability |
| APRS | TAPR APRS Protocol Reference | PROJECT-AUTHORITY | APRS protocol semantics; any regulatory use remains jurisdiction-specific |

## Authoritative references

### Aviation surveillance

**A1 — FAA, ADS-B "Ins and Outs"**  
https://www.faa.gov/air_traffic/technology/equipadsb/capabilities/ins_outs  
Supports the broadcast nature of ADS-B and aircraft GPS location, altitude and ground-speed information.

**A2 — FAA, ADS-B In Strategy**  
https://www.faa.gov/sites/faa.gov/files/air_traffic/technology/adsb/quicklinks/ADS-B_In_Strategy.pdf  
Supports ADS-B position, altitude, velocity, aircraft identification and data-quality semantics.

**A3 — FAA, ADS-B Installation**  
https://www.faa.gov/air_traffic/technology/equipadsb/installation  
Supports 1090ES and 978 MHz UAT as the two US ADS-B links.

**A4 — FAA, ADS-B FAQ**  
https://www.faa.gov/air_traffic/technology/equipadsb/resources/faq  
Supports UAT/FIS-B/TIS-B details and aircraft-address/flight-identification examples.

**A5 — EUROCONTROL, Surveillance System and Sensors**  
https://www.eurocontrol.int/service/surveillance-system-and-sensors  
Supports the role of Mode S, multilateration and ADS-B in surveillance infrastructure.

**A6 — EUROCONTROL, Surveillance RF Environment Monitoring 1030/1090 MHz**  
https://www.eurocontrol.int/service/surveillance-rf-environment-monitoring-10301090-mhz  
Supports use of 1030/1090 MHz by Mode A/C, Mode S, WAM/local multilateration, ADS-B and ACAS.

**A7 — EUROCONTROL, Wide Area Multilateration Guidelines**  
https://www.eurocontrol.int/publication/wide-area-multilateration-guidelines-achieving-operational-approval-wam-system  
Primary EUROCONTROL WAM guidance.

### UAV / electronic conspicuity

**U1 — FAA, Remote Identification of Drones**  
https://www.faa.gov/uas/getting_started/remote_id  
Supports RF broadcast of identification/location information and FAA examples using Wi-Fi/Bluetooth.

**U2 — EASA, Easy Access Rules for Unmanned Aircraft Systems**  
https://www.easa.europa.eu/en/document-library/easy-access-rules/online-publications/easy-access-rules-unmanned-aircraft-systems  
Supports direct periodic broadcast via an open/documented protocol and fields including identity, position, height, course, speed and remote-pilot/take-off location depending on UAS class/rule.

**U3 — FLARM, FAMP Public Protocol**  
https://www.flarm.com/en/integration/flarm-famp-public-protocol/  
Protocol-owner source for FAMP identity, position, heading and aircraft-type information.

**U4 — FLARM, FAMP publication/licensing notice**  
https://www.flarm.com/en/news/publishing-the-flarm-famp-public-protocol/  
Protocol-owner source for 868/915 MHz regional bands and the receive-only non-commercial licensing boundary.

**U5 — Open Glider Network, receiver documentation**  
https://wiki.glidernet.org/ogn-receiver-installation  
Project-authority source for the OGN receiver ecosystem only.

### Aviation datalink

**D1 — ICAO, Communications, Navigation and Surveillance section**  
https://www.icao.int/communications-navigation-and-surveillance-cns-section  
Lists ICAO manuals for HF Data Link (Doc 9741), VDL Mode 2 (Doc 9776), UAT and related CNS systems.

**D2 — ICAO, GANP ASBU Threads**  
https://ganpportal.icao.int/ASBU/Thread  
Supports continued use of ACARS, VDL Mode 2 and HFDL, including HFDL for oceanic airspace.

**D3 — ICAO Store, Manual on VHF Digital Link Mode 2 (Doc 9776)**  
https://store.icao.int/en/manual-on-vhf-digital-link-vdl-mode-2-doc-9776  
Authoritative description of VDL Mode 2 as an air/ground data link compatible with ATN.

### Maritime

**M1 — ITU-R M.1371-6 (02/2026), AIS**  
https://www.itu.int/rec/R-REC-M.1371-6-202602-I/en  
Current in-force AIS recommendation.

**M2 — ITU-R M.1371-6 contents**  
https://www.itu.int/dms_pubrec/itu-r/rec/m/R-REC-M.1371-6-202602-I%21%21TOC-HTM-E.htm  
Shows current AIS message families including position reports and ship static/voyage-related data.

**M3 — ITU-R M.493-16 (12/2023), DSC**  
https://www.itu.int/rec/R-REC-M.493/en  
Current in-force Digital Selective Calling recommendation.

**M4 — US Coast Guard, GMDSS FAQ**  
https://www.navcen.uscg.gov/gmdss-frequently-asked-questions  
Supports VHF Channel 70 use for DSC distress, safety and calling in the US/GMDSS context.

**M5 — IMO, NAVTEX Manual MSC.1/Circ.1403/Rev.2**  
https://wwwcdn.imo.org/localresources/en/OurWork/Safety/Documents/Documents%20relevant%20to%20GMDSS/MSC.1-Circ.1403-Rev.2.pdf  
Supports NAVTEX as automated maritime-safety-information service carrying navigational/met warnings, forecasts and urgent safety messages.

**M6 — ITU-R M.2092-2 (02/2026), VDES**  
https://www.itu.int/rec/R-REC-M.2092-2-202602-I/en  
Current in-force VDES recommendation.

**M7 — IALA, VDES overview**  
https://www.iala.int/technical/connectivity/vdes-vhf-data-exchange-system/  
Domain-authority explanation that VDES spans AIS, ASM and VDE ship/shore/satellite communications.

### Environmental / safety

**E1 — NOAA/NWS, Radiosonde Observation**  
https://www.weather.gov/upperair/factsheet  
Supports typical 400–405.9 MHz radiosonde transmission and pressure, temperature, humidity, GPS-position and wind observations.

**E2 — NOAA/NESDIS OSPO, HRIT/EMWIN**  
https://www.ospo.noaa.gov/operations/goes/hrit/  
Supports direct-to-user L-band weather-satellite broadcast, open format, imagery and warning/product content.

**E3 — NOAA SARSAT, 406 MHz Emergency Distress Beacons**  
https://www.sarsat.noaa.gov/emergency-406-beacons/  
Supports ELT/EPIRB/PLB classes and GNSS position inclusion in capable beacons.

**E4 — ITU-R maritime recommendations**  
https://www.itu.int/en/ITU-R/terrestrial/mars/Pages/References.aspx  
Lists current ITU-R recommendation families including 406 MHz COSPAS-SARSAT/EPIRB material.

### Amateur / experimental

**P1 — TAPR, APRS Protocol Reference v1.0.1**  
https://files.tapr.org/software_library/aprs/aprsspec/spec/aprs101m/APRS101m.pdf  
Project-authority protocol reference. Regulatory and redistribution questions require separate jurisdiction-specific sources.

## Publication notes

- The original article's FLARM licensing statement is supported by U4 and must retain the commercial/non-commercial distinction.
- OGN/FANET/ADS-L statements should be limited to what the OGN project itself documents unless a stronger standards source is added.
- ACARS/VDL2/HFDL are suitable as observation/correlation candidates, but the public article should avoid exposing or encouraging publication of operational message contents.
- 406 MHz distress-beacon data must be treated as a safety/SAR domain; technical receivability does not imply that public redistribution is appropriate.
- Any future claim about Vietnam-specific legality, permitted reception, storage or redistribution requires Vietnam-specific regulatory sources before publication.
