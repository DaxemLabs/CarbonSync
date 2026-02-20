# CarbonSync™ Security Overview
**Daxem Labs · CarbonSync™ · Patent Pending GB2602946.2**

**Document Version:** 1.1 (February 2026)
**Classification:** Public — Security Reference

> This document describes the threat model, cryptographic guarantees, chain of custody architecture, and key management practices for the CarbonSync™ platform. It is intended for fleet operators, investors, Verra/SustainCERT reviewers, and integration partners.

---

## 1. Threat Model

We assume the following attack vectors and design the Sentinel Rig and platform architecture to mitigate each:

| Attack Vector | Description | Mitigation |
| :--- | :--- | :--- |
| **Physical tampering** | Attacker accesses Sentinel Rig to modify data or extract cryptographic keys | ATECC608A secure element — keys provisioned at manufacture, physically unextractable; tamper detection pins trigger key zeroisation and hardware kill-switch |
| **Primary MCU compromise** | Attacker gains full control of the ESP32 — modifies sensor readings or firmware | ATECC608A operates independently via unidirectional I2C — a compromised ESP32 cannot produce valid signed hashes without the ATECC608A's protected key |
| **Network interception** | Attacker captures or modifies data in transit | MQTT over TLS 1.3 with mutual authentication; AES-256-GCM payload encryption (defence in depth) |
| **Cloud compromise** | Attacker gains access to backend servers | Hash chain enables independent verification without trusting the cloud; edge signing keys are separate from cloud keys |
| **Fake rig injection** | Attacker attempts to submit data from unauthorised devices | Mutual TLS with certificate revocation; hardware root of trust in ATECC608A — device identity is hardware-bound |
| **Replay attack** | Attacker resends previously captured valid packets | Timestamp + monotonic counter in each packet; backend rejects out-of-order or duplicate sequence numbers |
| **Hash collision attack** | Attacker attempts to generate a false data packet with a matching hash | SHA-256 HMAC using ATECC608A protected key — collision requires key extraction, which is physically prevented |
| **Supply chain attack** | Compromised hardware introduced during manufacturing | ATECC608A keys provisioned in a controlled factory environment; device attestation verified at first cloud registration |

---

## 2. Cryptographic Guarantees

### 2.1 At the Edge — Sentinel Rig

The hardware security architecture is a dual-processor design: ESP32 primary MCU paired with an ATECC608A dedicated cryptographic secure element. All security-critical operations occur exclusively on the ATECC608A.

| Component | Specification |
| :--- | :--- |
| **Secure Element** | ATECC608A (Microchip CryptoAuthentication) with hardware-accelerated SHA-256 |
| **Key Generation** | Private keys generated inside the ATECC608A at provisioning — never exist outside the chip |
| **Key Storage** | 16 hardware-protected key slots — physically unreadable by any software interface |
| **Signing** | Every 30-second data packet signed with SHA-256 HMAC using protected key |
| **Random Number Generation** | True hardware TRNG with FIPS 140-2 compliant entropy — no predictable seed risk |
| **Anti-Replay** | Timestamp + monotonic counter in each packet |
| **Tamper Detection** | Active tamper detection pins + internal shield mesh — physical interference triggers key zeroisation |
| **Tamper Response** | On tamper: keys zeroised, hardware kill-switch activated (cellular modem interrupt), event logged with timestamp |
| **Interface** | Unidirectional I2C — ESP32 sends data to ATECC608A, receives signed hash; no reverse command channel |

**Why the unidirectional interface matters:** A compromised ESP32 cannot instruct the ATECC608A to sign arbitrary data — the ATECC608A only signs data packets it receives via the one-way interface, computed against sensor inputs the ESP32 cannot forge without physical sensor access. This is the core security primitive described in Patent GB2602946.2, Claim 1.

### 2.2 In Transit

| Layer | Mechanism |
| :--- | :--- |
| **Transport Security** | MQTT over TLS 1.3 with mutual authentication |
| **Certificate Management** | Short-lived certificates (30 days) issued by private PKI — automatic rotation |
| **Payload Encryption** | AES-256-GCM on top of TLS (defence in depth — protects against TLS implementation vulnerabilities) |
| **Local SD Encryption** | AES-256, encryption key stored in ATECC608A protected slot — SD card is unreadable without the rig's ATECC608A |

### 2.3 At Rest

| Location | Mechanism |
| :--- | :--- |
| **Local Storage (SD card)** | AES-256 encrypted; key bound to ATECC608A — card cannot be read in another device |
| **Cloud Storage** | Server-side encryption with customer-managed keys; separate from edge signing keys |
| **Backup Integrity** | Cryptographic hash chain enables verification of backup integrity independent of storage provider |

---

## 3. Chain of Custody

Each data packet contains the following structure to create an immutable, linked hash chain from physical measurement to carbon credit:

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

**Hash chain mechanics:**
- Each packet's `previousHash` references the ATECC608A-computed hash of the immediately preceding packet
- A gap, modification, or insertion anywhere in the chain breaks hash continuity — detectable by any verifier with access to the public verification key
- The chain from the first packet to the most recently issued carbon credit token is continuously verifiable without trusting any intermediate party

**Verification path:**

```
Physical sensor reading
       ↓
ATECC608A SHA-256 HMAC (hardware key, unextractable)
       ↓
Packet stored: cloud (TLS 1.3) + SD card (AES-256)
       ↓
Dataloom™ hash chain verification
       ↓
Automated VVB verification package (IPFS content hash)
       ↓
ERC-1155 token minted on Polygon — IPFS hash in token metadata
       ↓
Carbon credit — cryptographically traceable to physical measurement
```

---

## 4. Key Management

| Key Type | Where Generated | Where Stored | Rotation |
| :--- | :--- | :--- | :--- |
| **Device signing key** | Inside ATECC608A at factory provisioning | ATECC608A protected slot — never leaves chip | Not rotated (hardware-bound identity) |
| **SD encryption key** | Inside ATECC608A at first boot | ATECC608A protected slot | On device replacement |
| **TLS client certificate** | Cloud PKI | ESP32 flash (non-sensitive — certificate only, not private key) | Every 30 days |
| **Cloud storage key** | Cloud KMS | Cloud KMS — customer-managed | Annual |

---

## 5. Responsible Disclosure

If you discover a security vulnerability in any CarbonSync™ component — hardware, firmware, platform, or this repository — please disclose it responsibly:

📧 **[security@daxem.ai](mailto:security@daxem.ai)**

Please include a clear description of the vulnerability, steps to reproduce, and your assessment of impact. We commit to acknowledging all reports within 48 hours and providing a remediation timeline within 14 days.

We do not operate a bug bounty programme at this stage, but we recognise and credit all good-faith disclosures.

---

## 6. Related Documents

- [`README.md`](README.md) — Platform overview and hardware security architecture summary
- [`HARDWARE_ROADMAP.md`](HARDWARE_ROADMAP.md) — ATECC608A integration detail and phase-gate milestones
- [`METHODOLOGY.md`](METHODOLOGY.md) — Cryptographic chain of custody in the Verra verification context
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — End-to-end system architecture

---

*CarbonSync™ is a trademark of Daxem Labs. Patent Pending GB2602946.2. All rights reserved.*
