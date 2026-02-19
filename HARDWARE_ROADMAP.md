# Sentinel Rig — Hardware Development Roadmap
**Daxem Labs · CarbonSync™ · Patent Pending GB2602946.2**

> From proof of concept to mass production — the engineering pathway for the Sentinel Rig, CarbonSync's hardware foundation.

---

## Overview

The Sentinel Rig is an edge-computing device installed on each HGV. It directly measures fuel consumption, reads engine telemetry via OBD-II, cryptographically signs all data at source, and transmits signed packets via MQTT over 4G every 30 seconds. It is the only component in the CarbonSync stack that cannot be replicated in software — and the only reason CarbonSync's carbon credits are auditable at Verra standards.

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
| Cryptographic co-processor testing (ATTiny85 unidirectional interface) | Weeks 4–8 |
| OBD-II / CAN bus integration testing | Weeks 5–8 |
| Power consumption profiling | Weeks 6–8 |
| **POC Completion Review** | **End of Week 8** |

**Success Criteria:** Core functionality demonstrated on dev kit. Fuel flow sensor achieving >98% accuracy. Cryptographic hashing confirmed tamper-evident. Power budget within spec for vehicle installation.

**Estimated Investment:** £8,000 – £40,000

---

### Phase 2 — EVT: Engineering Validation
**May 2026 – July 2026 · Weeks 9–20**

| Activity | Duration |
|---|---|
| Custom PCB design and fabrication | Weeks 9–14 |
| Firmware architecture and MQTT stack | Weeks 10–18 |
| Mechanical enclosure prototyping | Weeks 12–18 |
| 4G modem integration (Quectel BG96) | Weeks 13–16 |
| GPS module integration and clock validation | Weeks 14–17 |
| Lab integration testing — all subsystems | Weeks 17–20 |
| **EVT Completion Review** | **End of Week 20** |

**Success Criteria:** All features operational on custom PCB. Form factor fits within HGV engine bay. Firmware transmitting correctly signed packets. No data loss during simulated network outage.

**Estimated Investment:** £40,000 – £120,000

---

### Phase 3 — DVT: Design Validation
**August 2026 – November 2026 · Weeks 21–36**

| Activity | Duration |
|---|---|
| Design for Manufacture (DFM) optimisation | Weeks 21–24 |
| Environmental testing: -40°C to +85°C | Weeks 22–28 |
| Vibration and shock testing (5g RMS, automotive grade) | Weeks 24–28 |
| IP67 certification (dust-tight, temporary immersion) | Weeks 25–30 |
| CE / UK Conformity Assessed (UKCA) regulatory compliance | Weeks 26–34 |
| Beta fleet installation — 10 vehicles, diverse conditions | Weeks 29–36 |
| **DVT Completion Review** | **End of Week 36** |

**Success Criteria:** Design meets all environmental specifications consistently. Regulatory approvals granted. Beta fleet data flowing cleanly. Field failure rate below target. Installation process validated by non-specialist technicians.

**Estimated Investment:** £120,000 – £250,000

---

### Phase 4 — PVT: Production Validation
**December 2026 – February 2027 · Weeks 37–44**

| Activity | Duration |
|---|---|
| Pilot production run — 100 units | Weeks 37–42 |
| Yield optimisation (target: >95%) | Weeks 38–44 |
| Installation process documentation and contractor training | Weeks 40–44 |
| Supply chain stress testing — dual-source critical components | Weeks 40–44 |
| Cost target validation (£250/unit target) | Weeks 42–44 |
| **PVT Completion Review** | **End of Week 44** |

**Success Criteria:** >95% production yield. Unit cost at or below £250. Installation completable in under 2 hours by trained technician. Supply chain validated for volume.

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

**Funded by:** Carbon credit revenue and fleet operator hardware fees.

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
| Cryptographic co-processor | Independent ATTiny85 with unidirectional interface — per patent claim 1 | POC |
| Fuel flow accuracy | >98% — hall-effect ultrasonic sensor, inline fuel line | POC |
| Hardware kill-switch | Physical interrupt on tamper detection | EVT |
| Dual-persistent storage | Cloud (TLS) + SD card (tamper-evident, IP67 enclosure) | EVT |
| Environmental hardening | -40°C to +85°C operating range — automotive grade | DVT |
| Vibration tolerance | 5g RMS — HGV cabin and engine bay conditions | DVT |
| IP rating | IP67 — dust-tight, temporary immersion | DVT |
| Regulatory compliance | CE / UKCA — UK market | DVT |
| Unit cost | £250 per rig including installation | PVT |
| Production yield | >95% | PVT |

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Component supply chain disruption | Medium | High | Dual-source all critical components — Quectel BG96 alternatives qualified in EVT |
| Certification delays (CE/UKCA) | Medium | High | Engage test house at DVT start, not after completion |
| Field failure rates above target | Low | High | Beta fleet across diverse conditions (urban, motorway, waste, logistics) |
| Installation complexity | Medium | Medium | Standardised installation kit + certified contractor training programme |
| Cost overrun above £250/unit | Low | Medium | DFM review at DVT; volume pricing locked with CM at PVT |
| CBAM deadline (Jan 2027) pressure | Fixed | High | DVT completion by Nov 2026 gives 2-month buffer before CBAM live date |

---

## Why the CBAM Deadline Changes Everything

The UK Carbon Border Adjustment Mechanism goes live **1 January 2027**. Every unverified tonne of HGV emissions becomes a financial liability from that date. CarbonSync's PVT completes in February 2027 — meaning pilot fleet operators are covered by DVT beta units from October 2026, and volume deployment begins in March 2027, within the first CBAM quarter.

This is not a nice-to-have timeline. It is a market-entry window with a hard close.

---

*CarbonSync™ is a trademark of Daxem Labs. Patent Pending GB2602946.2. All rights reserved.*
