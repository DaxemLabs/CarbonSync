# Sentinel Rig — Hardware Development Roadmap
**Daxem Labs · CarbonSync™ · Patent Pending GB2602946.2**

> From proof of concept to mass production — the engineering pathway for the Sentinel Rig, CarbonSync's hardware foundation.

---

## Overview

The Sentinel Rig is an edge-computing device installed on each HGV. It directly measures fuel consumption, reads engine telemetry via OBD-II, cryptographically signs all data at source using a dedicated ATECC608A secure element, and transmits signed packets via MQTT over 4G every 30 seconds. It is the only component in the CarbonSync stack that cannot be replicated in software — and the only reason CarbonSync's carbon credits are auditable at Verra standards.

This roadmap follows the industry-standard hardware development framework: **POC → EVT → DVT → PVT → MP**.

---

## Phase Timeline

```
2026                                                    2027
Mar   Apr   May   Jun   Jul   Aug   Sep   Oct   Nov   Dec   Jan   Feb   Mar
|--POC--|
              |--------EVT--------|
                                    |----------DVT-----------|
                                                               |---PVT---|
                                                                          CBAM ↑
                                                                              |--MP-->
```

---

## Phase Details

### Phase 1 — POC: Proof of Concept
**March 2026 – April 2026 · Weeks 1–8**

| Activity | Duration |
|---|---|
| Component selection and dev kit prototyping | Weeks 1–4 |
| Core fuel flow sensor validation | Weeks 3–6 |
| ATECC608A secure element integration and unidirectional I2C interface testing | Weeks 4–8 |
| OBD-II / CAN bus integration testing | Weeks 5–8 |
| Power consumption profiling | Weeks 6–8 |
| **POC Completion Review** | **End of Week 8** |

**POC Component Selections:**

| Component | Part | Role |
|---|---|---|
| Primary MCU | ESP32-WROOM-32 | Data processing, sensor interfacing, cloud transmission |
| Cryptographic secure element | ATECC608A (Microchip) | Hardware SHA-256, protected key storage, TRNG, tamper detection |
| Fuel flow sensor | Hall-effect ultrasonic (inline, fuel return line) | Direct volumetric fuel measurement |
| Cellular modem | Quectel BG96 | 4G LTE connectivity, multi-band |
| GPS module | NEO-6M or NEO-M8N | Position tracking, route reconstruction |
| Local storage | Industrial micro-SD (32–128 GB, −40°C to +85°C) | Append-only audit black box |

**ATECC608A Integration Detail:**

The ATECC608A is the preferred embodiment of the independent cryptographic co-processor described in Patent GB2602946.2 Claims 1 and 2. It is a dedicated CryptoAuthentication IC — not a general-purpose microcontroller — providing hardware-enforced security primitives that no software-only implementation can replicate:

| ATECC608A Capability | Specification | Significance |
|---|---|---|
| SHA-256 | Hardware accelerated — not firmware emulated | Keys and hashes never exposed to software layer |
| Key storage | 16 hardware-protected slots — physically unreadable | Private signing keys cannot be extracted by compromised software |
| TRNG | True hardware random number generator | FIPS 140-2 compliant entropy — no predictable seed risk |
| Tamper detection | Active pins + internal shield mesh | Physical interference triggers hardware kill-switch |
| Interface | Unidirectional I2C (data in, signed hash out) | No command channel — co-processor cannot be instructed to produce false hashes |
| Operating temperature | −40°C to +85°C | Matches automotive/industrial operating envelope |
| Supply voltage | 2.0–5.5V | Compatible with ESP32 3.3V rail |

**Architecture rationale:** The ESP32 primary MCU and ATECC608A operate as a dual-processor pair. The ESP32 collects sensor data and transmits it to the ATECC608A. The ATECC608A computes the SHA-256 HMAC using its protected signing key and returns the hash. The ESP32 stores both the raw data and the hash in cloud and local storage. A compromised ESP32 cannot produce a valid hash without the ATECC608A's protected key — enforcing the unidirectional trust chain that Claim 1 of the patent describes.

**Success Criteria:** Core functionality demonstrated on dev kit. Fuel flow sensor achieving >98% accuracy. ATECC608A producing valid SHA-256 HMACs with tamper-evident architecture confirmed. Power budget within spec for vehicle installation.

**Estimated Investment:** £8,000 – £40,000

---

### Phase 2 — EVT: Engineering Validation
**May 2026 – July 2026 · Weeks 9–20**

| Activity | Duration |
|---|---|
| Custom PCB design and fabrication (ESP32 + ATECC608A integrated layout) | Weeks 9–14 |
| Firmware architecture and MQTT stack | Weeks 10–18 |
| Mechanical enclosure prototyping | Weeks 12–18 |
| 4G modem integration (Quectel BG96) | Weeks 13–16 |
| GPS module integration and clock validation | Weeks 14–17 |
| Lab integration testing — all subsystems | Weeks 17–20 |
| **EVT Completion Review** | **End of Week 20** |

**Success Criteria:** All features operational on custom PCB. Form factor fits within HGV engine bay. Firmware transmitting correctly ATECC608A-signed packets. No data loss during simulated network outage. Hardware kill-switch triggering correctly on hash mismatch.

**Estimated Investment:** £40,000 – £120,000

---

### Phase 3 — DVT: Design Validation
**August 2026 – November 2026 · Weeks 21–36**

| Activity | Duration |
|---|---|
| Design for Manufacture (DFM) optimisation | Weeks 21–24 |
| Environmental testing: −40°C to +85°C | Weeks 22–28 |
| Vibration and shock testing (5g RMS, automotive grade) | Weeks 24–28 |
| IP67 certification (dust-tight, temporary immersion) | Weeks 25–30 |
| CE / UK Conformity Assessed (UKCA) regulatory compliance | Weeks 26–34 |
| Beta fleet installation — 10 vehicles, diverse conditions | Weeks 29–36 |
| **DVT Completion Review** | **End of Week 36** |

**Success Criteria:** Design meets all environmental specifications consistently. ATECC608A tamper detection pins validated under vibration and temperature extremes. Regulatory approvals granted. Beta fleet data flowing cleanly with cryptographic chain of custody confirmed end-to-end. Field failure rate below target. Installation process validated by non-specialist technicians.

**Estimated Investment:** £120,000 – £250,000

---

### Phase 4 — PVT: Production Validation
**December 2026 – February 2027 · Weeks 37–44**

| Activity | Duration |
|---|---|
| Pilot production run — 100 units | Weeks 37–42 |
| Yield optimisation (target: >95%) | Weeks 38–44 |
| ATECC608A key provisioning process (secure factory programming) | Weeks 38–42 |
| Installation process documentation and contractor training | Weeks 40–44 |
| Supply chain stress testing — dual-source critical components | Weeks 40–44 |
| Cost target validation | Weeks 42–44 |
| **PVT Completion Review** | **End of Week 44** |

**Success Criteria:** >95% production yield. Unit CAC at target including installation. ATECC608A key provisioning process validated for secure factory programming. Installation completable in under 2 hours by trained technician. Supply chain validated for volume.

**Estimated Investment:** £160,000 – £400,000

---

### Phase 5 — MP: Mass Production
**March 2027 onwards**

| Activity |
|---|
| Volume manufacturing ramp |
| Fleet-wide deployments begin (UK HGV operators) |
| Field performance monitoring and continuous improvement |
| Second-generation hardware development initiated |

**Funded by:** Carbon credit revenue and fleet operator SaaS subscriptions.

---

## Investment Summary

| Phase | Timeline | Investment Range | Key Output |
|---|---|---|---|
| POC | Mar – Apr 2026 | £8K – £40K | Technical feasibility confirmed |
| EVT | May – Jul 2026 | £40K – £120K | Working custom hardware |
| DVT | Aug – Nov 2026 | £120K – £250K | Production-ready, certified design |
| PVT | Dec 2026 – Feb 2027 | £160K – £400K | Manufacturing process validated |
| MP | Mar 2027+ | Revenue-funded | Volume deployment |
| **Total to MP** | | **£328K – £810K** | **Fleet-ready product** |

---

## Critical Technical Milestones

| Milestone | Specification | Phase |
|---|---|---|
| Dual-processor architecture | ESP32 primary MCU + ATECC608A secure element with unidirectional I2C interface — per Patent GB2602946.2 Claim 1 | POC |
| ATECC608A hardware SHA-256 | Hardware-accelerated hash computation — keys physically unextractable | POC |
| ATECC608A tamper detection | Active pins + kill-switch triggering on hash mismatch | POC/EVT |
| Fuel flow accuracy | >98% — hall-effect ultrasonic sensor, inline fuel line | POC |
| Hardware kill-switch | Physical interrupt on tamper or hash mismatch | EVT |
| Dual-persistent storage | Cloud (TLS 1.3) + SD card (tamper-evident, IP67 enclosure) | EVT |
| Environmental hardening | −40°C to +85°C operating range — automotive grade | DVT |
| Vibration tolerance | 5g RMS — HGV cabin and engine bay conditions | DVT |
| IP rating | IP67 — dust-tight, temporary immersion | DVT |
| Regulatory compliance | CE / UKCA — UK market | DVT |
| Secure key provisioning | ATECC608A factory programming process validated | PVT |
| Unit CAC | Target cost per rig including installation | PVT |
| Production yield | >95% | PVT |

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Component supply chain disruption | Medium | High | Dual-source all critical components — Quectel BG96 and ATECC608A alternatives qualified in EVT |
| Certification delays (CE/UKCA) | Medium | High | Engage test house at DVT start, not after completion |
| Field failure rates above target | Low | High | Beta fleet across diverse conditions (urban, motorway, waste, logistics) |
| Installation complexity | Medium | Medium | Standardised installation kit + certified contractor training programme |
| Cost overrun above CAC target | Low | Medium | DFM review at DVT; volume pricing locked with CM at PVT |
| CBAM deadline (Jan 2027) pressure | Fixed | High | DVT completion by Nov 2026 gives 2-month buffer before CBAM live date |

---

## Why the CBAM Deadline Changes Everything

The UK Carbon Border Adjustment Mechanism goes live **1 January 2027**. Every unverified tonne of HGV emissions becomes a financial liability from that date. CarbonSync's PVT completes in February 2027 — meaning pilot fleet operators are covered by DVT beta units from October 2026, and volume deployment begins in March 2027, within the first CBAM quarter.

This is not a nice-to-have timeline. It is a market-entry window with a hard close.

---

## Related Documents

- [`README.md`](README.md) — Platform overview and live demo
- [`METHODOLOGY.md`](METHODOLOGY.md) — Cryptographic chain of custody and Verra methodology
- [`COMPETITIVE_LANDSCAPE.md`](COMPETITIVE_LANDSCAPE.md) — Market positioning
- [`PILOT_PROGRAM.md`](PILOT_PROGRAM.md) — Beta fleet programme

---

*CarbonSync™ is a trademark of Daxem Labs. Patent Pending GB2602946.2. All rights reserved.*
