# Testudo

**Post-quantum cryptographic security layer for the da Vinci inference stack.**

Testudo is a standalone C++ static library (`libtestudo`) that provides native PQC protection across every sensitive surface of an AI inference pipeline — model weights at rest, inference channels in transit, and deployment artifacts. It is architecturally integrated, not a bolt-on wrapper.

---

## What It Secures

- **Model weights at rest** — AES-256-GCM encrypted storage with authenticated load
- **Inference channels** — per-session key exchange and request/response encryption
- **Weight integrity** — signature verification at load time; hard abort on failure
- **Deployment artifacts** — tamper-evident signed builds for air-gap chain-of-custody

## What It Does Not Cover (v1)

- Network security / VPN layer
- User authentication
- Training-time security

---

## Cryptographic Primitives

All algorithms are NIST 2024 finalized standards. No experimental primitives.

| Primitive | Algorithm | Purpose |
|---|---|---|
| Key encapsulation | ML-KEM (CRYSTALS-Kyber) | Session key exchange |
| Digital signatures | ML-DSA (CRYSTALS-Dilithium) | Weight and artifact signing |
| Hash-based signatures | SLH-DSA (SPHINCS+) | Long-lived, stateless weight authentication |
| Symmetric encryption | AES-256-GCM | Weight encryption at rest |

Hybrid classical + PQC mode (ECDH + Kyber in parallel) is supported as a configuration option for environments still in transition away from classical cryptography, consistent with CNSA 2.0 transition guidance.

---

## Architecture

```
testudo/
├── include/
│   └── testudo/
│       ├── kyber.hpp        # KEM interface
│       ├── dilithium.hpp    # signature interface
│       ├── sphincs.hpp      # hash-based sig interface
│       ├── weight_store.hpp # encrypted weight load/save
│       └── session.hpp      # inference session management
├── src/
│   ├── kyber.cpp
│   ├── dilithium.cpp
│   ├── sphincs.cpp
│   ├── weight_store.cpp
│   └── session.cpp
├── third_party/
│   └── liboqs/              # Open Quantum Safe reference implementations
├── tests/
│   ├── test_kyber.cpp
│   ├── test_dilithium.cpp
│   ├── test_weight_roundtrip.cpp
│   └── test_session.cpp
└── CMakeLists.txt
```

### Integration Points

**1. Weight load boundary**
Weights are stored encrypted on disk. At load time, a Dilithium signature is verified before decryption proceeds. Signature failure results in a hard abort — no partial weight load, no fallback.

**2. Inference channel**
Kyber KEM establishes a per-session symmetric key. All request/response traffic is encrypted under AES-256-GCM. Dilithium-signed responses are available as an optional mode for high-assurance deployments that require output authentication.

**3. Build pipeline**
SPHINCS+ signatures cover compiled binaries and weight files. Any post-build modification to weights or the engine binary is detectable before execution — relevant for air-gap deployments where chain of custody matters.

---

## Dependency

Testudo depends on [liboqs](https://github.com/open-quantum-safe/liboqs) (Open Quantum Safe project) for NIST-validated C implementations of Kyber, Dilithium, and SPHINCS+. Licensed Apache 2.0.

---

## Building

```bash
git clone --recurse-submodules https://github.com/mai-industries/testudo
cd testudo
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

### Running Tests

```bash
cd build && ctest --output-on-failure
```

---

## Compliance

Testudo is designed for CNSA 2.0 compliance (January 2027 mandate for National Security Systems) and targets FIPS 140-3 certification readiness. All selected algorithms are NIST-standardized (FIPS 203, 204, 205).

> **Note on FIPS 140-3:** liboqs is pursuing FIPS 140-3 validation but is not yet formally certified as of early 2026. Deployments requiring a validated cryptographic module should track the NIST CMVP queue and evaluate AWS-LC or BoringSSL with PQC extensions as alternatives once available.

---

## Security Notes

**Side-channel safety.** Kyber and Dilithium have known timing vulnerabilities in naive implementations. liboqs provides constant-time implementations; any custom wrapper code must not introduce timing leaks (e.g., conditional branches on secret data). This is the highest-priority implementation risk.

**Key management.** The cryptographic primitives are well-understood. Failures in practice happen at key generation, storage, and rotation. A key management policy covering storage, access control, and rotation procedures is required before production deployment.

---

## Part of the da Vinci Stack

Testudo is the security layer of the da Vinci AI system, developed by MAI Industries. It is distributed as a separately auditable static library so that the security boundary is explicit and reusable across projects.

---

*MAI Industries — da Vinci Stack*
