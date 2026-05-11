# Testudo

**Quantum-resistant security, native to the inference stack.**

Testudo is the post-quantum cryptographic layer of the da Vinci AI system. It protects model weights at rest, inference channels in transit, and deployment artifacts end-to-end — built directly into the C++ inference engine, not bolted on after the fact.

All primitives are NIST 2024 finalized standards. No experimental algorithms. No external security dependencies.

---

## What It Protects

| Surface | Protection |
|---|---|
| Model weights at rest | AES-256-GCM encryption + Dilithium signature verification at load |
| Inference channel | Per-session Kyber key exchange + AES-256-GCM request/response encryption |
| Deployment artifacts | SPHINCS+ signed builds — tamper-evident, air-gap ready |

Inference without a valid weight signature fails hard. No graceful fallback, no partial load.

---

## Algorithms

| Role | Standard |
|---|---|
| Key encapsulation | ML-KEM (CRYSTALS-Kyber) — FIPS 203 |
| Digital signatures | ML-DSA (CRYSTALS-Dilithium) — FIPS 204 |
| Hash-based signatures | SLH-DSA (SPHINCS+) — FIPS 205 |
| Symmetric encryption | AES-256-GCM — CNSA 2.0 approved |

Hybrid classical + PQC mode (ECDH + Kyber) is supported for environments still transitioning away from classical cryptography.

---

## Compliance

Designed for **CNSA 2.0** compliance ahead of the January 2027 mandate for all National Security Systems. Targets **FIPS 140-3** certification readiness. Built on [liboqs](https://github.com/open-quantum-safe/liboqs) (Open Quantum Safe), Apache 2.0.

---



---

*Part of the [da Vinci Stack] — MAI Industries*
