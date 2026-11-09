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

## 2. Repository Layout

```text
/workspace/beta/
├── README.md                      # Primary project overview & binary specification
├── .gitignore                     # Git ignore rules
├── binary/                        # Production binary artifact & verified digests
│   ├── 15.0.0.3.156.apk           # Meta Muse target APK v15.0.0.3.156
│   └── checksums.txt              # SHA-256 and MD5 verification records
└── docs/                          # Comprehensive reverse-engineered architecture specifications
    └── ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW_V15.md
```

---

## 3. Architecture & Reverse Engineering Highlights (v15 Delta)

1. **Refresh Token Lifecycle Loop:**
   - Migration from static single-session access tokens to dynamic rotation:
     * `AuraSessionManager.migrateToRefreshToken()`
     * `POST /hatch/refresh_access_token` (`StefiHatchPath.RefreshAccessToken`)
     * `POST /hatch/logout` (`StefiHatchPath.Logout`)
2. **Post-Quantum Cryptography (PQ-KEM):**
   - 27 compiled native Rust libraries under `lib/arm64-v8a/`:
     * NIST FIPS 203 ML-KEM-768 (`libthird-party_rust_vendor_ml-kem_0.3.so`)
     * Hybrid X-Wing KEM (`libthird-party_rust_vendor_x-wing_0.1.so`)
     * OPAQUE Asymmetric PAKE RFC 9842 (`libthird-party_rust_vendor_opaque-ke_4.so`)
     * Vesta HSM v2 protocol (`libxplat_rust_vesta_client_v2_proto_vesta_proto-rustAndroid.so`)
3. **AYMH (Accounts You Might Have) Saved Account One-Tap OTP:**
   - Cross-app Meta identity authentication via `POST /hatch/auth/send_saved_account_otp`.
4. **Unified Plan Finishing Orchestrator:**
   - Consolidated activation pipeline via `AuraActivationPlanFinishViewModel` integrating ToS acceptance, subscription plans, and hardware enclave PIN enrollment (`CreatePin` / `EnterExistingPin`).
5. **Dual-Tier Referral & Pairing Engine:**
   - **Tier A (Server-Side Growth Referral):** Pre-flight handshake (`GET /hatch/invite`), self-referral collision prevention, and mutation (`POST /hatch/invite_code/redeem`).
   - **Tier B (Peer Invites P2P Subsystem):** 7 new endpoints under `com.facebook.aura.pairs.PeerInvitesPath` supporting media attachment uploads (`POST /peer_invite_photos/upload`) and encrypted envelope delivery (`POST /peer_invites/send`).

For detailed protocol specifications, wire schemas, and step-by-step lifecycle flows, refer to [`docs/ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW_V15.md`](docs/ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW_V15.md).
