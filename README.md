# CarbonSync™ — Live Intelligence Platform
**by Daxem Labs · Patent Filed GB2602946.2**

> The first platform to directly measure, verify, and tokenise emissions reductions from UK HGV fleets in real time.

---

## Live Demo

**[Launch CarbonSync™ Dashboard →](https://daxemlabs.github.io/CarbonSync/carbonsync-demo.html)**

### Pre-Configured Client Links

| Client | Direct Link |
|--------|------------|
| Pickfords (700 vehicles) | [Open Pickfords Dashboard](https://daxemlabs.github.io/CarbonSync/carbonsync-demo.html?company=Pickfords&fleet=700&sector=removals&autolaunch=true) |
| O'Donovan Waste (85 vehicles) | [Open O'Donovan Dashboard](https://daxemlabs.github.io/CarbonSync/carbonsync-demo.html?company=O%27Donovan+Waste&fleet=85&sector=waste&autolaunch=true) |
| Generic (any fleet) | [Open with Login Screen](https://daxemlabs.github.io/CarbonSync/carbonsync-demo.html) |

---

## What Is This?

This is a fully interactive sandbox demonstration of the CarbonSync™ platform — built for fleet operators, investors, and pilot partners to experience the product before a Sentinel Rig is installed.

It runs on simulated data that mirrors exactly what the live platform produces when a real rig is connected. Every number, calculation, and credit figure uses real methodology — DEFRA 2024 emission factors, Verra VCS standards, and live ACX market pricing.

**When a real Sentinel Rig connects, nothing in the UI changes. The simulation data is simply replaced with live packets.**

---

## Platform Overview

| Tab | What It Shows |
|-----|--------------|
| **Overview** | Live credits, fuel savings, revenue, and the full verification pipeline from sensor to blockchain |
| **Live Telemetry** | Per-vehicle data feed — fuel flow, CO₂, OBD-II signals, cryptographic signatures |
| **Carbon Credits** | Credit breakdown by source category, Verra VCS methodology, CBAM alignment |
| **Blockchain Ledger** | Real-time ERC-1155 minting events on Polygon — transaction hashes, block numbers |
| **Revenue** | 3-year financial projection, 70/15/10/5 revenue split, hardware payback period |
| **⚠ CBAM Exposure** | UK Carbon Border Adjustment Mechanism liability calculator — live from January 2027 |

---

## How It Works

```
Sentinel Rig  →  MQTT / 4G  →  Dataloom™ AI  →  CarbonSync™ Ledger  →  Polygon Blockchain  →  Carbon Market
   (HGV)          (signed)       (validated)        (cryptographic)          (ERC-1155)           (revenue)
```

---

## Repository Contents

| File | Purpose |
|------|---------|
| `carbonsync-demo.html` | Full interactive platform — single file, no dependencies |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | End-to-end system architecture — edge hardware through to blockchain settlement |
| [`METHODOLOGY.md`](METHODOLOGY.md) | Scientific and regulatory methodology — DEFRA 2024, Verra VCS, ACX pricing |
| [`HARDWARE_ROADMAP.md`](HARDWARE_ROADMAP.md) | Sentinel Rig phase-gate development roadmap — POC through to Mass Production |
| [`SECURITY.md`](SECURITY.md) | Threat model, cryptographic guarantees, chain of custody, key management |
| [`COMPETITIVE_LANDSCAPE.md`](COMPETITIVE_LANDSCAPE.md) | Competitor matrix and market positioning analysis |
| [`PILOT_PROGRAM.md`](PILOT_PROGRAM.md) | Beta fleet pilot programme — objectives, partner criteria, timeline |
| `CarbonSync_Rig_Brief.docx` | Technical specification for embedded and backend engineers |
| `NOTICE` | IP and trademark notice |

---

## Connecting a Real Sentinel Rig

The platform is production-ready for live rig integration. Full technical specification in `CarbonSync_Rig_Brief.docx`.

**WebSocket endpoint:**
```
wss://api.daxem.ai/rigs/live
```

**Developer Mode:** Press `Ctrl+D` inside the dashboard — WebSocket status, live connection log, packet schema, and computation constants. Simulation mode is on by default.

---

## Financial Model

| Parameter | Value | Source |
|-----------|-------|--------|
| CO₂ emission factor | 2.68 kg/litre | DEFRA 2024 |
| Efficiency baseline | 18% improvement | Verified methodology |
| Carbon credit price | £72/tCO₂e | ACX voluntary market |
| Fleet revenue share | 70% | CarbonSync™ standard |
| Platform fee | 15% | Daxem Labs |
| Driver incentives | 10% | Fleet operator distributed |
| Reserve fund | 5% | Buffer |
| Hardware CAC | £473/vehicle | Sentinel Rig inc. installation |

---

## CBAM Compliance

The **⚠ CBAM Exposure** tab calculates each fleet's liability under the UK Carbon Border Adjustment Mechanism, effective **1 January 2027**.

- Without verified data: HMRC default penalty rates applied to all emissions
- With CarbonSync™: Verified actual data = lowest possible CBAM charge + carbon credit income offsets liability
- UK ETS rate: £40–£100/tonne (adjustable in the calculator)

---

## Technology

| Layer | Technology |
|-------|-----------|
| Hardware | Sentinel Rig — ESP32 primary MCU + ATECC608A secure element, fuel flow sensor, OBD-II, GPS, 4G (Quectel BG96) |
| Cryptography | ATECC608A hardware-accelerated SHA-256, protected key storage, FIPS-compliant TRNG |
| Data capture | MQTT over TLS, SHA-256 signed at source, SD card local backup |
| AI validation | Dataloom™ — LSTM per-vehicle anomaly detection, efficiency scoring |
| Blockchain | Polygon Mainnet — ERC-1155 tokens, gas-efficient minting |
| Verification | Verra VCS methodology, DEFRA 2024 emission factors |
| Frontend | Vanilla JS, no dependencies, WebSocket live data layer |

### Hardware Security Architecture

The Sentinel Rig uses a dual-processor architecture for hardware-enforced cryptographic integrity — the core of what makes CarbonSync™ data Verra-auditable and tamper-evident.

**Primary MCU: ESP32-WROOM-32**
- 32-bit dual-core processor, 240 MHz, WiFi/Bluetooth
- Manages sensor interfacing, data aggregation, OBD-II telemetry, and cloud transmission
- Coordinates all data flows with the ATECC608A secure element

**Cryptographic Secure Element: ATECC608A (Microchip CryptoAuthentication)**
- Hardware-accelerated SHA-256 — hash computation occurs in dedicated silicon, never exposed to software
- Protected key storage — private keys are provisioned at manufacture and physically cannot be extracted by any software attack
- True hardware random number generator (TRNG) with FIPS 140-2 compliant entropy
- Tamper detection pins with active internal shield layer
- Communicates with the ESP32 via unidirectional I2C interface — receives sensor data, returns signed hashes, accepts no command inputs

**Why this matters:** A compromised ESP32 cannot generate false sensor data and a matching cryptographic hash simultaneously. Hash computation and key signing occur on the independent ATECC608A, whose private keys are physically unextractable. This hardware-enforced separation is what creates Verra-credible chain of custody from the physical measurement event to the issued carbon credit.

> *Patent Pending GB2602946.2 — ATECC608A secure element preferred embodiment. US Provisional filed simultaneously. 12-month PCT window active. See [METHODOLOGY.md](METHODOLOGY.md) for full cryptographic chain of custody specification and [SECURITY.md](SECURITY.md) for threat model and key management.*

---

## About Daxem Labs

Daxem Labs is a UK-based climate technology company building the infrastructure for hardware-anchored carbon credit verification.

- **Stage:** Pre-pilot · Q2 2026 launch
- **Pilot partner:** Strategic validation partner (to be announced)
- **Target market:** UK HGV fleet operators — waste, logistics, construction, removals
- **Patent:** GB2602946.2 (filed February 2026)
- **Contact:** [daxem.ai](https://daxem.ai) · [investors@daxem.ai](mailto:investors@daxem.ai) · [partnerships@daxem.ai](mailto:partnerships@daxem.ai)

---

*CarbonSync™ is a trademark of Daxem Labs. All rights reserved.*
