# CarbonSync™ System Architecture
**Daxem Labs · CarbonSync™ · Patent Pending GB2602946.2**

**Document Version:** 1.0 (February 2026)
**Classification:** Public — Technical Reference

> This document describes the end-to-end architecture from Sentinel Rig hardware through to blockchain minting and carbon market settlement. Intended for engineers, integration partners, Verra/SustainCERT reviewers, and technical investors.

---

## Overview

CarbonSync is a distributed system for cryptographically-secured measurement, validation, and tokenisation of industrial emissions reductions. The architecture is divided into four layers: Edge (hardware), Validation (AI and data integrity), Ledger (chain of custody and verification), and Settlement (blockchain and carbon market).

The defining architectural property is that **trust is established at the point of physical measurement** — not at the cloud layer, not at the registry, and not by policy. Every carbon credit is cryptographically traceable to a physical sensor reading signed by hardware whose keys are physically unextractable.

**Patent Reference:** GB2602946.2 — Cryptographically-Secured Hardware-Software System for Real-Time Measurement, Validation, and Automated Verification of Industrial Emissions Reductions with Distributed Ledger Integration

---

## High-Level Data Flow

```
Sentinel Rig          Dataloom™              CarbonSync™           Settlement
(Vehicle Edge)        (Validation)           (Ledger + Pipeline)

┌─────────────┐       ┌─────────────┐        ┌─────────────┐       ┌──────────────┐
│ ESP32 MCU   │       │ Multi-modal │        │ Automated   │       │ Polygon      │
│ + ATECC608A │──────▶│ validation  │───────▶│ VVB package │──────▶│ ERC-1155     │
│ (signed pkt)│ MQTT/ │ LSTM anomaly│        │ generation  │       │ token minting│
│             │ TLS   │ detection   │        │             │       └──────┬───────┘
└─────────────┘       │ Hash verify │        │ IPFS upload │              │
                      └─────────────┘        │ Verra API   │       ┌──────▼───────┐
                                             └─────────────┘       │ Carbon Market│
                                                                    │ ACX / buyers │
                                                                    └──────────────┘
```

---

## Layer 1 — Edge: Sentinel Rig Hardware

The Sentinel Rig is an IP67-rated edge computing device installed on each HGV. It is the only component in the stack that cannot be replicated in software. Full specification in [`HARDWARE_ROADMAP.md`](HARDWARE_ROADMAP.md).

### 1.1 Dual-Processor Architecture

| Processor | Component | Role |
| :--- | :--- | :--- |
| **Primary MCU** | ESP32-WROOM-32 (240 MHz dual-core, WiFi/BT) | Sensor interfacing, data aggregation, cloud transmission, OBD-II communication |
| **Cryptographic secure element** | ATECC608A (Microchip CryptoAuthentication) | Hardware SHA-256, protected key storage, TRNG, tamper detection — all crypto operations |

The ESP32 and ATECC608A communicate via a **unidirectional I2C interface**:
- ESP32 transmits sensor data packet → ATECC608A
- ATECC608A computes SHA-256 HMAC → returns signed hash to ESP32
- No reverse command channel exists — the ATECC608A cannot be instructed to produce arbitrary signatures

### 1.2 Sensor Suite

| Sensor | Component | Data Captured | Accuracy |
| :--- | :--- | :--- | :--- |
| **Fuel flow** | Hall-effect ultrasonic (inline, fuel return line) | Volumetric fuel consumption (L) at pulse level | >98% |
| **Engine telemetry** | OBD-II (ISO 15765 / CAN) | RPM, speed, engine load, coolant temp, throttle | Hardware standard |
| **Position** | NEO-6M / NEO-M8N GPS | Lat/lng, altitude, speed, route reconstruction | ~2.5m CEP |
| **Communication** | Quectel BG96 4G LTE | Multi-band, embedded SIM, kill-switch enable pin | — |

### 1.3 Data Packet (30-second interval)

Every 30 seconds, the Sentinel Rig produces a signed packet containing:

```json
{
  "rigId": "SENT-001",
  "timestamp": 1700000000000,
  "sequenceNumber": 47291,
  "fuelPulse_L": 1.234,
  "obdRpm": 1450,
  "obdSpeed_kmh": 62,
  "obdLoad_pct": 74,
  "gpsLat": 51.5074,
  "gpsLng": -0.1278,
  "gpsAlt_m": 24,
  "previousHash": "sha256:3a7f...",
  "signature": "sha256-hmac:9c2b..."
}
```

The `signature` is computed by the ATECC608A using its hardware-protected private key. The `previousHash` links each packet to its predecessor, forming a tamper-evident chain.

### 1.4 Dual-Persistent Storage

Every packet is written to two independent storage layers simultaneously:

| Layer | Mechanism | Purpose |
| :--- | :--- | :--- |
| **Cloud** | MQTT over TLS 1.3 → AWS/Azure/GCP | Real-time processing, Dataloom™ validation, dashboard |
| **Local SD card** | AES-256 encrypted, append-only, IP67 enclosure | Audit black box — survives network outages, cloud failures, disputes |

During network outages, packets accumulate on the SD card and transmit as a verified catch-up batch on reconnection. No data is lost.

### 1.5 Hardware Kill-Switch

If the ATECC608A detects a hash mismatch (stored hash ≠ recomputed hash from stored data), or if its tamper detection pins are triggered:

1. Physical interrupt signal sent to Quectel BG96 cellular modem enable pin
2. Data transmission ceases immediately
3. Tamper event logged with timestamp in append-only local storage
4. Cloud backend notified of kill-switch event on reconnection

---

## Layer 2 — Validation: Dataloom™ AI Engine

Dataloom™ is the cloud-based AI validation layer. It performs two functions: cryptographic verification of the hash chain, and AI-based multi-modal anomaly detection.

### 2.1 Hash Chain Verification

On receipt of each packet batch:
1. Recompute SHA-256 hash of each packet's data fields
2. Verify signature using device's public verification key
3. Confirm `previousHash` continuity — any gap or mismatch flags the affected packets
4. Assign hash verification score (component of data quality score — see [`METHODOLOGY.md`](METHODOLOGY.md) Section 5.3)

### 2.2 Multi-Modal Cross-Validation

Three independent data streams are cross-checked on every packet:

| Stream | Source | Cross-Validation Rule |
| :--- | :--- | :--- |
| **Fuel flow** | Hall-effect sensor | Fuel consumption consistent with engine RPM and vehicle speed |
| **Engine telemetry** | OBD-II | RPM > idle threshold when fuel consumption > idle rate; no fuel when stationary > threshold |
| **GPS** | GPS module | Distance travelled consistent with OBD-II speed; route topology physically plausible |

Packets failing cross-validation are quarantined and flagged for VVB review. False positive rate: <1% (vs. 3–5% for software-only systems — a direct result of hardware-signed source data).

### 2.3 LSTM Baseline Modelling

For each vehicle, Dataloom™ trains a vehicle-specific Long Short-Term Memory (LSTM) neural network on validated historical data:

| LSTM Property | Specification |
| :--- | :--- |
| **Input features** | Fuel flow, RPM, speed, engine load, GPS elevation delta, route topology |
| **Architecture** | Input layer → 3 LSTM layers (128, 64, 32 neurons) + dropout → dense output |
| **Training data** | Minimum 60-day baseline period per vehicle |
| **Prediction target** | Fuel consumption for each 30-second interval |
| **Accuracy** | <5% mean absolute prediction error (vs. 12–18% for fleet-average models) |

The LSTM generates the baseline against which project-period fuel consumption is compared, producing the verified emissions reduction quantity.

---

## Layer 3 — Ledger: CarbonSync™ Verification Pipeline

### 3.1 Monitoring Period Close

At the close of each monitoring period (typically monthly), Dataloom™ automatically generates a structured verification package:

| Package Component | Content |
| :--- | :--- |
| **Complete sensor data** | All packets with cryptographic hash chain intact |
| **Quality scores** | Per-packet and period-level data quality scores |
| **Anomaly log** | All flagged packets with quarantine reasons |
| **Baseline comparison** | LSTM baseline vs. actual — verified emissions reduction in tCO₂e |
| **Attribution breakdown** | LSTM attribution by reduction source (route, behaviour, idle, etc.) |
| **IPFS content hash** | Content identifier for the complete package — used in token metadata |

### 3.2 VVB Submission

The verification package is submitted to the Validation and Verification Body (VVB) via REST API:

```
POST https://api.vvb-partner.com/v1/submissions
Authorization: Bearer {vvb_api_key}
Content-Type: application/json

{
  "projectId": "CARBONSYNC-001",
  "monitoringPeriod": "2026-Q2",
  "packageIpfsHash": "QmXf3...",
  "rigIds": ["SENT-001", "SENT-002", ...],
  "claimedReduction_tCO2e": 12.4,
  "dataQualityScore": 96.2
}
```

The VVB computationally verifies the hash chain (seconds) rather than conducting manual document review (days to weeks). Physical site visit frequency is reduced from every crediting period to annual.

### 3.3 Cost and Time Comparison

| Metric | Manual MRV | CarbonSync™ Automated |
| :--- | :--- | :--- |
| Verification cost | $3–5 per tonne | $0.50–0.75 per tonne |
| Time to credit | 6–12 months | 30–60 days |
| VVB site visits | Every crediting period | Annual |
| Audit method | Manual document review | Computational hash verification |

---

## Layer 4 — Settlement: Blockchain and Carbon Market

### 4.1 Token Minting

Upon VVB approval, Verified Carbon Units (VCUs) are minted as ERC-1155 tokens on Polygon Mainnet:

```
Minting trigger:  VVB approval certificate received via API
Token standard:   ERC-1155 (semi-fungible — supports batch operations, lower gas)
Network:          Polygon Mainnet (proof-of-stake — >99% lower energy than PoW)
```

Each token's metadata contains:

```json
{
  "name": "CarbonSync VCU",
  "description": "1 tonne CO₂e verified reduction — UK HGV fleet",
  "projectId": "CARBONSYNC-001",
  "rigId": "SENT-001",
  "monitoringPeriod": "2026-Q2",
  "tCO2e": 1,
  "vvbCertificate": "CERT-2026-0042",
  "dataPackageIpfs": "QmXf3...",
  "mintTimestamp": 1700000000000
}
```

The `dataPackageIpfs` field links every token to its underlying verified data package — any buyer can retrieve and independently verify the full hash chain.

### 4.2 Revenue Distribution

Smart contracts distribute revenue automatically on token sale:

| Recipient | Share | Notes |
| :--- | :--- | :--- |
| Fleet operator | 70% | Primary beneficiary |
| Daxem Labs (platform fee) | 15% | Platform fee |
| Driver incentives | 10% | Operator discretion |
| Reserve fund | 5% | Verra risk buffer |

### 4.3 Carbon Market Integration

| Market | Type | Access Method |
| :--- | :--- | :--- |
| ACX (AirCarbon Exchange) | Voluntary carbon market | API integration — listed at prevailing spot price |
| Corporate direct buyers | OTC / bilateral | Direct sale via CarbonSync™ marketplace |
| Verra registry | VCS | API submission for formal VCU issuance |

---

## Repository and API Reference

### WebSocket Live Data

```
wss://api.daxem.ai/rigs/live
```

Delivers live signed packets from connected Sentinel Rigs. Press `Ctrl+D` inside the [live dashboard](https://daxemlabs.github.io/CarbonSync/carbonsync-demo.html) to inspect the packet schema and connection log in developer mode.

### Key Endpoints

| Endpoint | Method | Purpose |
| :--- | :--- | :--- |
| `/rigs/live` | WebSocket | Live packet stream |
| `/rigs/{rigId}/packets` | GET | Historical packet retrieval |
| `/rigs/{rigId}/quality` | GET | Data quality score history |
| `/projects/{projectId}/credits` | GET | Issued credit history |
| `/verification/submit` | POST | VVB verification package submission |

Full integration specification: `CarbonSync_Rig_Brief.docx`

---

## Related Documents

- [`README.md`](README.md) — Platform overview and live demo
- [`HARDWARE_ROADMAP.md`](HARDWARE_ROADMAP.md) — ATECC608A integration detail and phase-gate milestones
- [`METHODOLOGY.md`](METHODOLOGY.md) — Verra methodology, baseline construction, verification architecture
- [`SECURITY.md`](SECURITY.md) — Threat model, cryptographic guarantees, key management
- [`COMPETITIVE_LANDSCAPE.md`](COMPETITIVE_LANDSCAPE.md) — Market positioning

---

*CarbonSync™ is a trademark of Daxem Labs. Patent Pending GB2602946.2. All rights reserved.*
