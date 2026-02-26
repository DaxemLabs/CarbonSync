# CarbonProof™ — Hardware-Anchored Carbon Credits

**The first hardware-anchored dMRV platform for UK HGV fleets.**

A Sentinel Rig — combining a fuel flow sensor, OBD-II telemetry, and an ATECC608A cryptographic secure element — installs in any HGV and creates tamper-evident, Verra-auditable emissions data at source. That data flows through Dataloom™ AI validation to the Polygon blockchain, where verified reductions are minted as ERC-1155 carbon credit tokens and settled at market.

Fleet operators earn carbon credit revenue. Their customers get the verified Scope 3 transport data that CSRD and modern procurement frameworks now require. And the carbon market gets something it has never had: a credit it can actually trust.

---

## Why Now

**CSRD Scope 3 — the primary commercial driver**

The EU Corporate Sustainability Reporting Directive is in force. UK subsidiaries of EU companies, and any UK business supplying EU-connected buyers, face Scope 3 reporting requirements that must be traceable, auditable, and methodology-referenced. Transport sits at the core of Scope 3.

Fleet operators cannot currently provide their customers with verified, auditable transport emissions data. Those customers — large corporates, manufacturers, importers — are already requiring it in tender specifications and supplier onboarding. The fleet operator who can provide it wins and retains the contract. The one who cannot loses it.

**UK ETS — the fleet operator's financial exposure**

UK HGV operators are subject to the UK Emissions Trading Scheme. As the scheme expands and the carbon price rises — projected at £40–£100 per tonne through 2030 — fleets with no verified baseline face maximum ETS cost. CarbonProof creates the verified baseline that minimises ETS liability, while simultaneously generating carbon credit revenue from measured reductions.

**CBAM — indirect supply chain pressure**

Importers of steel, aluminium, cement, and other CBAM goods must account for supply chain emissions including transport in their CBAM and CSRD returns. Those importers are asking their logistics partners for verified transport emissions data. A CarbonProof-certified carrier can answer yes. An uncertified carrier cannot. This creates a procurement premium for CarbonProof-certified fleets.

---

## The Pipeline

```
Sentinel Rig → Dataloom™ AI → Polygon Blockchain → Carbon Market
```

| Stage | Component | Detail |
|-------|-----------|--------|
| 01 | **Sentinel Rig** | ESP32 + ATECC608A · Fuel flow sensor · OBD-II · GPS · 4G |
| 02 | **Dataloom™ AI** | LSTM per-vehicle models · Multi-modal cross-validation · <5% error rate |
| 03 | **Polygon Blockchain** | ERC-1155 tokens · IPFS content hash · Auto-minted on VVB approval |
| 04 | **Carbon Market** | ACX pricing · £72/tCO₂e · 70% to fleet operator · Auto-settled |

Every 30-second packet is SHA-256 signed and hash-chained at source by the ATECC608A secure element. A compromised ESP32 cannot produce a valid hash — the signing key is on independent hardware it cannot access.

→ See [ARCHITECTURE.md](./ARCHITECTURE.md) for full system design  
→ See [SECURITY.md](./SECURITY.md) for cryptographic schema

---

## The Hardware Moat

Software can be replicated in weeks. A certified automotive-grade secure element with a patented cryptographic architecture and an installed fleet base cannot.

**ATECC608A — Microchip CryptoAuthentication**

- **Hardware-accelerated SHA-256** — Hash computation in dedicated silicon. The private signing key never exists outside the ATECC608A at any point in its lifecycle.
- **Protected key storage** — 16 hardware-protected slots provisioned at factory. Physically unreadable by any software interface — extraction requires destroying the chip.
- **FIPS 140-2 compliant TRNG** — True hardware random number generator. Eliminates predictable-seed vulnerabilities present in every software-only implementation.
- **Active tamper detection** — Internal shield mesh + active pins. Physical interference triggers key zeroisation and hardware kill-switch. Events logged with timestamp.

*Patent Application Filed February 2026 — Preferred embodiment*

---

## Regulatory Alignment

| Framework | Status | Relevance |
|-----------|--------|-----------|
| **Verra VCS** | Methodology in engagement | Designed for Verra VCS alignment — active dialogue with Verra and SustainCERT |
| **DEFRA 2024** | Aligned | Fuel consumption and emissions calculation methodology |
| **UK ETS** | Active | Verified baseline data minimises operator ETS liability |
| **CSRD Scope 3** | Active | Provides the auditable transport emissions data corporate customers require |
| **CBAM** | Indirect | Creates procurement demand from importers for CarbonProof-certified carriers |

---

## Revenue Model

| Party | Share | Basis |
|-------|-------|-------|
| Fleet operator | 70% | Primary revenue — carbon credits generated |
| Daxem Labs (platform) | 15% | Platform and infrastructure fee |
| VVB / verification | 10% | Third-party audit and certification |
| Verra registry | 5% | Registry and methodology fee |

---

## Market

- **£2.1B** — UK HGV total addressable market
- **80% cheaper** than manual MRV approaches
- **30–60 days** time to first verified credit
- **CSRD Scope 3** requirements active now for large corporate supply chains
- **UK ETS expansion** — carbon price trajectory £40–£100/tonne through 2030

---

## Repository Structure

```
CarbonProof/
├── README.md              # This file
├── ARCHITECTURE.md        # Full system architecture
├── METHODOLOGY.md         # Emissions calculation and verification methodology
├── SECURITY.md            # Cryptographic schema and security model
└── CarbonProof-demo.html   # Live interactive demo
```

---

## Live Demo

[**→ View Live Demo**](https://daxemlabs.github.io/CarbonProof)

---

## Contact

| Enquiry | Address |
|---------|---------|
| Investor inquiries | investors@daxem.ai |
| Fleet partnerships & pilots | partnerships@daxem.ai |
| Technical & engineering | engineering@daxem.ai |

---

*CarbonProof™ is a trademark of Daxem Labs Ltd · Co. No. 16614429 · London, UK*  
*Patent Application Filed February 2026 · All rights reserved*  
*[daxem.ai](https://daxem.ai)*
