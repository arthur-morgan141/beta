# Project Beta: Meta Muse (`com.facebook.aura`) v15.0.0.3.156

This repository hosts the reverse engineering audit, technical specifications, and production binary artifacts for **Meta Muse** version `15.0.0.3.156` (Version Code `1062001464`), the multimodal agentic AI companion for Meta smart glasses, wearables, and isolated confidential compute.

---

## 1. Target Binary Artifact Identity

- **Filename:** `binary/15.0.0.3.156.apk`
- **Package Name:** `com.facebook.aura`
- **Application Label:** `Muse`
- **Release Version:** `15.0.0.3.156`
- **Version Code:** `1062001464`
- **Build ID:** `1109388401`
- **VCS Branch / Revision:** `master` / `MASTER`
- **Min SDK / Target SDK / Compile SDK:** `29` (Android 10) / `36` (Android 16) / `37` (Android 17 preview)
- **Native ABI:** `arm64-v8a` (64-bit ARM)
- **Binary Size:** 45,820,517 bytes (~43.7 MB)
- **SHA-256 Checksum:** `a099b8ec4aeda5bd9c4f2abfcb191388c3d300720691b1ed4b55b23c4a708cb5`
- **MD5 Checksum:** `aa992c2717f9c86dadc9bb7c51400145`
- **Checksum Verification File:** `binary/checksums.txt`

---

## 2. Structured Domain Architecture & Navigation Map

The technical reverse-engineering analysis is partitioned into four focused domain modules at the repository root, ensuring rigorous isolation of concerns:

```text
/workspace/beta/
├── README.md                          # Master technical architecture & navigation map
├── binary/                            # Target APK artifact & cryptographic checksums
│   ├── 15.0.0.3.156.apk
│   └── checksums.txt
├── api/                               # 1. Gateway & Formal Wire Schemas Specification
│   ├── README.md                      # Transport layer, Tigon stack, headers, and RFC 7807 errors
│   └── ENDPOINTS_AND_SCHEMAS.md       # Complete request/response JSON wire catalogue
├── auth/                              # 2. Authentication, Registration & Credential Sealing
│   ├── README.md                      # Registration lifecycle, coroutine state machines, and sequence flow
│   ├── PWD_ENC_CRYPTO.md              # Mathematical specification of #PWD_ENC:2 binary envelope
│   └── TOKEN_LIFECYCLE.md             # AuraSessionManager, silent token renewal, and KeyStore StrongBox
├── activation/                        # 3. Activation Gatekeeper, Onboarding & Hardware Enclave
│   ├── README.md                      # Lifecycle gatekeepers, ToS consent, and workspace finalization
│   ├── STATE_MACHINE_18.md            # The 18-state evaluation matrix and priority hierarchy
│   └── CVM_AND_VESTA_HSM.md           # Confidential VM provisioning, Meta Vesta HSM v2 (OPAQUE PAKE)
├── referral/                          # 4. Referral Subsystem & Invite Redemption (FINAL MODULE)
│   ├── README.md                      # Dual-Tier referral architecture & sequence flow
│   ├── TIER_A_GROWTH_REDEMPTION.md    # Pre-flight handshake, self-referral guard, and mutation
│   └── TIER_B_PEER_INVITES_P2P.md     # P2P decentralized invite envelopes (7 endpoints)
└── docs/                              # Full Documentation Hub & Master Implementation Plan
    ├── README.md                      # Centralized documentation index
    ├── MASTER_REGISTRATION_AND_ONBOARDING_PLAN.md # Ready-to-execute Master Implementation Plan
    ├── ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW_V15.md # End-to-end architecture flow spec
    └── DIRECT_API_REGISTRATION_AND_INVITE_REDEEM_SYSTEM_V15.md # Reverse-engineering analysis dossier
```

> [!IMPORTANT]
> **Monorepo Engineering Compliance Directive:**
> Sesuai ketentuan *Redeem Invite Code Placement Rule*, seluruh taksonomi spesifikasi, skema protokol, penanganan error RFC 7807, dan implementasi modul **Redeem Invite Code** (`referral/`) ditempatkan pada urutan paling akhir navigasi monorepo dan arsitektur dokumen.

---

## 3. Architecture & Reverse Engineering Highlights (v15 Delta)

1. **Decoupled OTP & Ephemeral Completion Handles:**
   - Migration from single-shot registration to handle-bound credentials (`HatchAuthApi.confirmOtp` $\to$ `completion_handle`).
2. **Post-Quantum Cryptography & Hardware Security:**
   - NIST FIPS 203 ML-KEM-768 (`libthird-party_rust_vendor_ml-kem_0.3.so`)
   - OPAQUE Asymmetric PAKE RFC 9842 (`libthird-party_rust_vendor_opaque-ke_4.so`)
   - Meta Vesta HSM v2 Protocol (`libxplat_rust_vesta_client_v2_proto_vesta_proto-rustAndroid.so`)
3. **17-State Gatekeeper Engine:**
   - Evaluated in strict priority order via `ActivationState.Companion.getCurrentActivationState()` over 24 boolean/entity fields in `ActivationStatusesResponseJson`.
4. **Hardware Enclave Isolation (CVM):**
   - Confidential Virtual Machines running with AMD SEV-SNP/Intel TDX hardware memory encryption, with automated SVM fallback downgrade upon cluster saturation.
5. **Dual-Tier Referral & Pairing Subsystem:**
   - **Tier A (Server-Side Growth Referral):** Pre-flight handshake (`GET /hatch/invite`), self-referral collision prevention, and mutation (`POST /hatch/invite_code/redeem`).
   - **Tier B (Peer Invites P2P Subsystem):** 7 new endpoints under `com.facebook.aura.pairs.PeerInvitesPath` supporting media attachment uploads (`POST /peer_invite_photos/upload`) and encrypted envelope delivery (`POST /peer_invites/send`).

---

## 4. Master Documentation & Domain Links

- 🚀 **[Master Implementation Plan (TDD-Ready)](docs/MASTER_REGISTRATION_AND_ONBOARDING_PLAN.md)** | **[Full Docs Hub](docs/README.md)**
- 📡 **[API Gateway & Wire Schemas](api/README.md)** | **[Catalogue](api/ENDPOINTS_AND_SCHEMAS.md)**
- 🔐 **[Authentication & Registration](auth/README.md)** | **[#PWD_ENC:2 Crypto](auth/PWD_ENC_CRYPTO.md)** | **[Token Lifecycle](auth/TOKEN_LIFECYCLE.md)**
- 🛡️ **[Activation & Gatekeeper Engine](activation/README.md)** | **[17-State Matrix](activation/STATE_MACHINE_18.md)** | **[CVM & Vesta HSM](activation/CVM_AND_VESTA_HSM.md)**
- 🎟️ **[Referral & Redeem Code (Final Module)](referral/README.md)** | **[Tier A Growth](referral/TIER_A_GROWTH_REDEMPTION.md)** | **[Tier B P2P Invites](referral/TIER_B_PEER_INVITES_P2P.md)**
