# Meta Muse v15: Full Documentation & Implementation Plan Hub

Welcome to the centralized documentation hub for **Meta Muse** (`com.facebook.aura` v15.0.0.3.156, Version Code `1062001464`). This directory hosts the complete, production-ready master architecture specifications and the actionable step-by-step implementation plan for automating the direct API client.

---

## 1. Primary Master Implementation Plan

- 🚀 **[`docs/MASTER_REGISTRATION_AND_ONBOARDING_PLAN.md`](file:///home/ubuntu/projects/beta/docs/MASTER_REGISTRATION_AND_ONBOARDING_PLAN.md)**
  - **Status:** Complete, TDD-ready, and actionable.
  - **Scope:** 10 bite-sized, checkable tasks covering the complete lifecycle from transport setup, anonymous bootstrap, `#PWD_ENC:2` cryptographic packaging, session token rotation, 17-state gatekeeper resolution, CVM hardware enclave allocation, Meta Vesta HSM v2 PIN setup, workspace finalization, and the final referral module (Task 10).
  - **Execution Sub-Skills:** Fully compatible with `superpowers:subagent-driven-development` and `superpowers:executing-plans`.

---

## 2. Comprehensive Architectural Specifications

The following master specifications provide the theoretical foundation, sequence diagrams, and security models backing the implementation plan:

1. 📘 **[`docs/ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW_V15.md`](file:///home/ubuntu/projects/beta/docs/ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW_V15.md)**
   - End-to-end architectural flow specification in Technical English.
   - 24-field gatekeeper status models, multi-channel legal consent, hardware enclave provisioning, and P2P pairing models.
2. 📙 **[`docs/DIRECT_API_REGISTRATION_AND_INVITE_REDEEM_SYSTEM_V15.md`](file:///home/ubuntu/projects/beta/docs/DIRECT_API_REGISTRATION_AND_INVITE_REDEEM_SYSTEM_V15.md)**
   - Deep-dive technical reverse engineering and forensic analysis dossier.
   - Tigon transport telemetry, memory KeyStore sealing (`AuraMetaFRLSecureTokenStore`), multi-release comparative matrix (v13 vs v14 vs v15), and anti-abuse heuristics.

---

## 3. Modular Domain Dossiers Cross-Reference

For micro-level bytecode analysis, exact wire schemas, and native JNI disassembly, refer to the root domain directories:

- 📡 **[`api/`](file:///home/ubuntu/projects/beta/api/README.md)** — Transport specifications, HTTP headers, RFC 7807 error models, and [Complete Wire Schemas Catalogue](file:///home/ubuntu/projects/beta/api/ENDPOINTS_AND_SCHEMAS.md).
- 🔐 **[`auth/`](file:///home/ubuntu/projects/beta/auth/README.md)** — Registration coroutines, [#PWD_ENC:2 Binary Crypto](file:///home/ubuntu/projects/beta/auth/PWD_ENC_CRYPTO.md), and [Token Lifecycle & KeyStore](file:///home/ubuntu/projects/beta/auth/TOKEN_LIFECYCLE.md).
- 🛡️ **[`activation/`](file:///home/ubuntu/projects/beta/activation/README.md)** — Gatekeeper evaluation, [17-State Matrix](file:///home/ubuntu/projects/beta/activation/STATE_MACHINE_18.md), and [CVM Enclave & Vesta HSM](file:///home/ubuntu/projects/beta/activation/CVM_AND_VESTA_HSM.md).
- 🎟️ **[`referral/`](file:///home/ubuntu/projects/beta/referral/README.md)** — *Final Module (Redeem Placement Rule Compliant)*: [Tier A Growth Redemption](file:///home/ubuntu/projects/beta/referral/TIER_A_GROWTH_REDEMPTION.md) and [Tier B P2P Peer Invites](file:///home/ubuntu/projects/beta/referral/TIER_B_PEER_INVITES_P2P.md).
