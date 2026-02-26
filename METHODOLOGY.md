# CarbonProof™ — Emissions Measurement & Verification Methodology

*Daxem Labs Ltd · Co. No. 16614429 · Patent Application Filed February 2026*

---

## Overview

CarbonProof measures, verifies, and tokenises emissions reductions from UK HGV fleets using a hardware-anchored digital MRV (dMRV) approach. The methodology is designed for alignment with Verra VCS standards and is in active engagement with Verra and SustainCERT.

The core principle: every emissions data point is cryptographically signed at the point of physical measurement by dedicated hardware whose state software cannot observe or influence. No manual data entry. No human trust assumptions. Every carbon credit is traceable to a physical measurement event.

---

## Regulatory & Standards Framework

### Verra VCS — Methodology in Engagement
CarbonProof is designed for alignment with the Verified Carbon Standard (VCS). The platform architecture is built to produce the tamper-evident, auditable measurement data that VCS methodology approval requires. Active engagement with Verra and SustainCERT is ongoing.

> **Note:** CarbonProof is in methodology engagement with Verra. VCS methodology approval is in progress and has not yet been formally granted.

### DEFRA 2024 Emission Factors
Fuel consumption and emissions calculations use DEFRA's 2024 greenhouse gas conversion factors for company reporting. These are the UK government's official methodology for fleet emissions quantification and are updated annually.

### UK Emissions Trading Scheme (UK ETS)
UK HGV operators are subject to the UK ETS. CarbonProof creates the verified baseline data that minimises operator ETS liability by providing auditable, methodology-referenced measurement rather than estimated default values. As the UK carbon price rises toward £40–£100 per tonne through 2030, the difference between verified and unverified baseline data becomes material.

### CSRD Scope 3
The EU Corporate Sustainability Reporting Directive requires large companies — and their supply chains — to report Scope 3 emissions in a traceable, auditable, methodology-referenced format. Transport is a primary Scope 3 category. CarbonProof provides fleet operators with the verified emissions data their corporate customers now require for CSRD compliance.

---

## Measurement Architecture

### Physical Measurement Layer

**Fuel Flow Sensor**
- Positive displacement flow meter installed in fuel line
- Measures actual fuel consumed per trip and per vehicle
- Resolution: 0.1 litre per pulse
- Calibrated against OBD-II fuel consumption data at commissioning

**OBD-II Telemetry**
- Real-time vehicle diagnostics via standardised OBD-II port
- Parameters captured: engine load, RPM, vehicle speed, fuel trim, intake air temperature, mass air flow
- Cross-validates fuel sensor readings
- Detects anomalous engine states that would affect emissions calculation accuracy

**GPS**
- Route distance verification
- Journey classification (motorway, A-road, urban)
- Enables route-adjusted emissions factors where applicable
- Geofenced zone detection for future low-emission zone compliance

**4G Connectivity**
- Encrypted transmission to cloud validation layer
- Offline buffering with signed packet queuing during connectivity loss
- Packets transmitted with sequence numbers and timestamps for completeness verification

### Cryptographic Integrity Layer

**ATECC608A Secure Element**
All data packets are SHA-256 signed at source by the ATECC608A CryptoAuthentication secure element before transmission. Key properties:

- Private signing key provisioned at factory and never accessible to software
- Hardware-accelerated SHA-256 — computation occurs in dedicated silicon
- FIPS 140-2 compliant true random number generator
- Active tamper detection with automatic key zeroisation on physical interference
- Every 30-second data packet is signed and hash-chained to the previous packet

This means: a valid data packet can only have been produced by the specific Sentinel Rig from which it originated. Packet substitution, replay attacks, and data fabrication are cryptographically impossible without physical access to and destruction of the secure element.

---

## Emissions Calculation

### Baseline Establishment

A verified baseline is established for each vehicle over a minimum 90-day measurement period prior to any reduction claim. The baseline captures:

- Fuel consumption per kilometre by route type (motorway, A-road, urban, mixed)
- Seasonal variation factors
- Load variation factors (where OBD-II data permits inference)
- Idling time and associated consumption

Baseline data is submitted to the VVB for approval before any credits are issued against reductions.

### Reduction Quantification

Emissions reductions are calculated against the verified baseline using DEFRA 2024 conversion factors:

```
Reduction (tCO₂e) = (Baseline_Consumption_L - Actual_Consumption_L) × DEFRA_Factor_kgCO₂e/L × 0.001
```

Where:
- `Baseline_Consumption_L` = verified baseline litres per km × distance km
- `Actual_Consumption_L` = measured actual litres consumed (fuel flow sensor, OBD-II cross-validated)
- `DEFRA_Factor_kgCO₂e/L` = appropriate DEFRA 2024 factor for fuel type

For HVO and alternative fuels, the relevant DEFRA lifecycle emissions factor is applied, capturing the full well-to-wheel reduction relative to diesel baseline.

### Anomaly Detection — Dataloom™ AI

Per-vehicle LSTM (Long Short-Term Memory) neural network models are trained on each vehicle's individual baseline data. The models flag anomalous readings that require manual review before inclusion in a credit claim:

- Sudden consumption changes inconsistent with route or load profiles
- Sensor drift or calibration deviation indicators
- Data gaps or packet sequence breaks
- Cross-validation failures between fuel sensor and OBD-II readings

Anomaly-flagged data is quarantined and excluded from credit calculations pending VVB review. The target false-positive rate is <5% of all packets.

---

## Verification & Credit Issuance

### VVB (Validated and Verified Body) Process

1. **Registration** — Sentinel Rig commissioned, vehicle enrolled, baseline period begins
2. **Baseline approval** — 90-day baseline data submitted to VVB for methodology review
3. **Monitoring** — Continuous measurement with real-time anomaly detection
4. **Periodic verification** — VVB reviews signed data packets, spot-checks cryptographic integrity, approves reduction claims
5. **Credit issuance** — On VVB approval, ERC-1155 tokens are minted on Polygon blockchain

### Blockchain Settlement

- **Token standard:** ERC-1155 on Polygon PoS
- **Metadata:** IPFS content hash linking to full measurement dataset, VVB approval reference, vehicle ID, measurement period
- **Immutability:** Once minted, the credit record cannot be altered — the on-chain token permanently references the off-chain measurement data via content-addressed storage
- **Settlement:** Credits settled at ACX market rate; revenue distributed automatically via smart contract (70% fleet operator / 15% platform / 10% VVB / 5% registry)

---

## Data Retention & Audit Trail

All measurement data is retained for a minimum of 7 years in accordance with Verra VCS requirements:

- Raw signed packets (immutable, stored in IPFS)
- Processed emissions calculations with full audit trail
- Anomaly detection logs
- VVB correspondence and approval records
- Blockchain transaction records

The complete chain from physical sensor pulse to minted token is reconstructable and auditable at any point.

---

## Limitations & Scope

**Current scope:**
- UK HGV fleets (7.5t+) operating on diesel or HVO
- Road transport emissions (tank-to-wheel)
- Fuel consumption reduction credits

**Not currently in scope:**
- Electric HGV (upstream electricity emissions — future roadmap)
- Refrigeration unit emissions (future roadmap)
- Sea or air freight

**Methodology status:**
- Designed for Verra VCS alignment — methodology in active engagement
- DEFRA 2024 emission factors: current as of publication
- UK ETS applicability: subject to scheme expansion schedule

---

## Further Documentation

→ [ARCHITECTURE.md](./ARCHITECTURE.md) — Full hardware and software system design  
→ [SECURITY.md](./SECURITY.md) — Cryptographic schema and security model  
→ [Live Demo](https://daxemlabs.github.io/CarbonProof/CarbonProof-demo.html) — Interactive platform demonstration

---

*CarbonProof™ is a trademark of Daxem Labs Ltd · Co. No. 16614429 · London, UK*  
*Patent Application Filed February 2026 · All rights reserved*  
*[daxem.ai](https://daxem.ai)*
