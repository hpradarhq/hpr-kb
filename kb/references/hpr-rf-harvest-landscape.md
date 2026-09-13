# Source Register — HPR Free-to-Harvest RF Landscape

This register follows [`../../meta/source-policy.md`](../../meta/source-policy.md). Protocol semantics, safety systems and spectrum claims prefer standards/regulators. Open-source repositories are cited only for implementation availability/capability.

## Aviation / surveillance

### [A1] EUROCONTROL — Surveillance RF Environment Monitoring 1030/1090 MHz
- Authority: `PRIMARY-REGULATOR / ATM AUTHORITY`
- URL: https://www.eurocontrol.int/service/surveillance-rf-environment-monitoring-10301090-mhz
- Supports: 1030/1090 MHz as shared critical environment for Mode A/C, Mode S, MLAT/WAM, ADS-B and ACAS; value of RF-environment monitoring.

### [A2] EUROCONTROL — Surveillance Systems and Sensors
- Authority: `PRIMARY-REGULATOR / ATM AUTHORITY`
- URL: https://www.eurocontrol.int/service/surveillance-system-and-sensors
- Supports: complementary role of Mode S, multilateration and ADS-B.

### [A3] FAA — ADS-B Installation
- Authority: `PRIMARY-REGULATOR`
- URL: https://www.faa.gov/air_traffic/technology/equipadsb/installation
- Supports: 1090ES and 978 MHz UAT; UAT weather-service capability.

### [A4] EUROCONTROL WikiLink — VDL Mode 2 deployment/frequencies
- Authority: `PRIMARY-OPERATIONAL-AUTHORITY`
- URL: https://ext.eurocontrol.int/WikiLink/index.php/Deployment
- Supports: 136.975 MHz common signalling channel and current European alternate VDL2 frequencies.

### [D1] ICAO — CNS documentation index
- Authority: `PRIMARY-STANDARD`
- URL: https://www.icao.int/communications-navigation-and-surveillance-cns-section
- Supports: ICAO manuals for HFDL, VDL2, UAT and related aeronautical communication systems.

### [D2] EUROCONTROL — Datalink
- Authority: `PRIMARY-OPERATIONAL-AUTHORITY`
- URL: https://www.eurocontrol.int/function/datalink
- Supports: VDL2 as standardized VHF air/ground datalink and separation of communications from surveillance.

## Maritime

### [M1] ITU-R M.1371-6 (02/2026)
- Authority: `PRIMARY-STANDARD`
- Status: in force
- URL: https://www.itu.int/rec/R-REC-M.1371-6-202602-I/en
- Supports: AIS technical characteristics and current recommendation version.

### [M2] USCG NAVCEN — VHF channel information
- Authority: `PRIMARY-NATIONAL-AUTHORITY`
- URL: https://navcen.uscg.gov/us-vhf-channel-information
- Supports: AIS 1 = 161.975 MHz; AIS 2 = 162.025 MHz; DSC Ch 70 frequency.

### [M3] USCG NAVCEN — GMDSS FAQ
- Authority: `PRIMARY-NATIONAL-AUTHORITY`
- URL: https://navcen.uscg.gov/gmdss-frequently-asked-questions
- Supports: VHF Channel 70 = 156.525 MHz and DSC distress/safety/calling role.

### [M4] IMO — Revised NAVTEX Manual, MSC.1/Circ.1403/Rev.1; current manual family updated by Rev.2/2023 edition
- Authority: `PRIMARY-STANDARD / IMO`
- Technical circular URL: https://wwwcdn.imo.org/localresources/en/OurWork/Safety/Documents/MSC.1-Circ.1403-Rev.1%20-%20REVISED%20NAVTEX%20MANUAL%20%28Secretariat%29.pdf
- Current GMDSS circular index: https://www.imo.org/en/ourwork/safety/pages/imo-circulars-related-to-the-gmdss.aspx
- Supports: 518 kHz international NAVTEX; 490 kHz and 4209.5 kHz national services; current IMO NAVTEX manual lineage.

### [M5] ITU-R M.2092-2 (02/2026)
- Authority: `PRIMARY-STANDARD`
- Status: in force
- URL: https://www.itu.int/rec/R-REC-M.2092-2-202602-I/en
- Supports: VDES including terrestrial/satellite VDE, ASM and AIS integration.

## UAS / low-altitude

### [U1] FAA — Remote Identification of Drones
- Authority: `PRIMARY-REGULATOR`
- URL: https://www.faa.gov/uas/getting_started/remote_id
- Supports: Standard Remote ID / broadcast module transmit identification and location via RF; Wi-Fi/Bluetooth examples.

### [U2] EASA — Direct Remote Identification / UAS rules
- Authority: `PRIMARY-REGULATOR`
- Current rules: https://www.easa.europa.eu/en/document-library/easy-access-rules/online-publications/easy-access-rules-unmanned-aircraft-systems
- Privacy/direct-ID background: https://www.easa.europa.eu/en/domains/civil-drones/privacy/privacy-by-design
- Supports: direct periodic broadcast, open/documented protocol, identity/location/motion/pilot or take-off-point data; 2.4/5 GHz implementation context.

### [F1] FLARM — FAMP Public Protocol
- Authority: `PROTOCOL-OWNER`
- URL: https://www.flarm.com/en/news/publishing-the-flarm-famp-public-protocol/
- Supports: FAMP exchanges identity, position/motion data; 868/915 MHz license-free regional operation; receive-only/non-commercial vs commercial licensing boundary.

## Environmental / satellite

### [E1] NOAA/NWS — Radiosonde Observation
- Authority: `PRIMARY-PUBLIC-AUTHORITY`
- URL: https://www.weather.gov/upperair/factsheet
- Supports: 400–405.9 MHz typical radiosonde transmissions; pressure, temperature, humidity, GPS position; winds derived from track.

### [E2] NOAA/NESDIS — HRIT/EMWIN
- Authority: `PRIMARY-PUBLIC-AUTHORITY`
- URL: https://ospo.noaa.gov/operations/goes/hrit/
- Supports: direct-to-user 1694.1 MHz L-band broadcast, open format, no NOAA fee/license to receive, antenna/receiver requirements. This is cited as an example of a genuinely open direct-broadcast service, not as a Vietnam-local source.

## Vietnam receive-only / licensing boundary

### [VN1] Vietnam Radio Frequency Department — receive-only equipment FAQ
- Authority: `PRIMARY-NATIONAL-AUTHORITY`
- URL: https://www.rfd.gov.vn/hoidap/pages/cap-phep.aspx?ItemID=890
- Supports: RFD statement that receive-only radio equipment can be used without a frequency/equipment-use licence.
- Caveat: the cited answer is specifically framed around receive-only satellite equipment; data/content rights require separate analysis.

### [VN2] Government of Vietnam — consolidated licence-exempt radio-equipment rules
- Authority: `PRIMARY-NATIONAL-LAW/REGULATION`
- URL: https://vanban.chinhphu.vn/?docid=213856&pageid=27160
- Document: 01/VBHN-BKHCN, 4 June 2025.
- Supports: current consolidated framework for categories of radio equipment exempt from frequency-use licensing and associated technical/operational conditions.

## Passive radar / opportunistic sensing

### [P1] Sensors 2021 — passive-radar inter-channel calibration
- Authority: `PEER-REVIEWED-RESEARCH`
- URL: https://www.mdpi.com/1424-8220/21/1/69
- Supports: passive radar as receive-only use of illuminators of opportunity; FM, DVB-T and GSM examples; low transmitter cost; coherent receiver considerations.

### [P2] Sensors 2023 — Beamforming Techniques for Passive Radar: An Overview
- Authority: `PEER-REVIEWED-RESEARCH`
- URL: https://www.mdpi.com/1424-8220/23/7/3435
- Supports: FM, DVB-T, DAB, GSM, LTE as opportunistic waveforms; need for multiple coherent channels and beamforming/DoA methods.

### [P3] Remote Sensing / Sensors passive-radar literature
- Authority: `PEER-REVIEWED-RESEARCH`
- URLs:
  - https://www.mdpi.com/2072-4292/8/11/929
  - https://www.mdpi.com/1424-8220/16/10/1594
- Supports: limitations of narrowband illuminators, DVB-T passive radar, multi-frequency processing and relationship between waveform bandwidth and achievable range resolution.

### [P4] KrakenSDR DoA / passive-radar project
- Authority: `PROJECT-AUTHORITY`
- URLs:
  - https://github.com/krakenrf/krakensdr_doa
  - https://github.com/krakenrf/krakensdr_pr
- Supports: practical coherent multi-channel SDR/DoA/passive-radar implementation patterns.

### [P5] jmfriedt/passive_radar
- Authority: `PROJECT-AUTHORITY / RESEARCH IMPLEMENTATION`
- URL: https://github.com/jmfriedt/passive_radar
- Supports: synchronized RTL-SDR DVB-T passive-radar measurements using separate reference/surveillance antennas.

### [P6] 30hours/blah2
- Authority: `PROJECT-AUTHORITY / EXPERIMENTAL IMPLEMENTATION`
- URL: https://github.com/30hours/blah2
- Supports: real-time passive-radar pipeline supporting RSPduo, USRP, synchronized HackRF/RTL-SDR and KrakenSDR.

## Open-source receiver/decoder projects

These sources establish implementation availability, not the normative meaning or legal status of the protocol.

### [O1] wiedehopf/readsb
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/wiedehopf/readsb
- Role: ADS-B / Mode-S decoder and tracker.

### [O2] FlightAware/dump978
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/flightaware/dump978
- Role: 978 MHz UAT demodulator/decoder.

### [O3] AIS-catcher
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/jvde-github/AIS-catcher
- Role: AIS receiver supporting RTL-SDR, Airspy, HackRF, SDRplay/Soapy and network outputs; GPLv3.

### [O4] OpenDroneID Core C
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/opendroneid/opendroneid-core-c
- Role: ASTM F3411 / ASD-STAN Direct Remote ID message packing/unpacking; Apache-2.0.

### [O5] radiosonde_auto_rx
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/projecthorus/radiosonde_auto_rx
- Role: continuous radiosonde scanning/detection/decoding; Docker deployment supported.

### [O6] f00b4r0/acarsdec
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/f00b4r0/acarsdec
- Role: active multichannel ACARS decoder with SDR inputs and JSON/MQTT outputs.

### [O7] TLeconte/vdlm2dec
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/TLeconte/vdlm2dec
- Role: VDL Mode 2 SDR decoder.

### [O8] szpajder/dumphfdl
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/szpajder/dumphfdl
- Role: multichannel HFDL decoder.

### [O9] jontio/JAERO
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/jontio/JAERO
- Role: Aero/SatCom ACARS demodulator/decoder.

### [O10] SatDump
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/SatDump/SatDump
- Role: general satellite data-processing/decoding framework.

### [O11] GNU Radio / GNU Radio 4
- Label: `PROJECT-AUTHORITY`
- URLs:
  - https://www.gnuradio.org/
  - https://github.com/gnuradio/gnuradio4
- Role: general SDR/DSP framework; GNU Radio 4 super-repo announced August 2026.

### [O12] SDRangel
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/f4exb/sdrangel
- Role: multi-hardware open-source SDR receiver/transmitter/signal-analysis platform.

### [O13] OpenWebRX
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/jketterl/openwebrx
- Role: multi-user web SDR receiver and decoder frontend.

### [S1] rtl_433
- Label: `PROJECT-AUTHORITY`
- URL: https://github.com/merbanan/rtl_433
- Role: generic decoder for 315/345/433.92/868/915 MHz and other low-power device signals.
- Governance note: technical decodability does not imply HPR should collect private/security-device traffic.

## Evidence rules for this stream

1. Frequency/channel/protocol semantics require Tier 1/2 sources where available.
2. Open-source availability is separately evidenced by project repositories.
3. “Receive-only” and “commercial redistribution” are treated as different claims.
4. Passive-radar performance claims must remain bounded by experimental geometry/hardware; no generic detection-range promise is canonical.
5. Vietnam deployment decisions require current Vietnamese legal review before commercial publication of decoded operational/private data.
