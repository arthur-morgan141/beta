# Meta Muse v15: Dual-Tier Referral Architecture & Invite Code System

> [!IMPORTANT]
> **Monorepo Engineering Compliance Directive:**
> Sesuai ketentuan *Redeem Invite Code Placement Rule*, seluruh taksonomi spesifikasi, skema protokol, penanganan error RFC 7807, dan implementasi modul **Redeem Invite Code** wajib ditempatkan pada urutan paling akhir seluruh dokumentasi, repositori, dan sistem antarmuka.

---

## 1. Executive Summary & Dual-Tier Architecture

In Meta Muse v15 (`com.facebook.aura` v15.0.0.3.156, Version Code `1062001464`), the referral subsystem has evolved from a single centralized invite string into an advanced **Dual-Tier Referral Engine**:

```
+-----------------------------------------------------------------------------------+
|                        Meta Muse v15 Referral Engine                              |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  +-------------------------------------+   +------------------------------------+ |
|  |  Tier A: Server-Side Growth Referral|   |  Tier B: Peer Invites P2P Subsystem| |
|  |  (https://hatch-api.meta.ai/hatch/) |   |  (com.facebook.aura.pairs)         | |
|  |                                     |   |                                    | |
|  |  - Gated Account Admission          |   |  - Decentralized User Pairing      | |
|  |  - Pre-flight Handshake Protection  |   |  - Base64 Encrypted Envelopes      | |
|  |  - Regional Waitlist Bypass         |   |  - Bech32m Address Formats (muse1) | |
|  |  - 6-Char Alphanumeric Code Engine  |   |  - Media (JPEG) SHA-256 Binding    | |
|  |  - CVM Compute Credit Rewards       |   |  - 7 P2P Path Endpoints + Accept   | |
|  |  - Deferred Install Referrer Job    |   |  - Wire Format Protocol Buffers    | |
|  +-------------------------------------+   +------------------------------------+ |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

1. **Tier A (Server-Side Growth & Waitlist Redemption):** The central gatekeeper mutation that unlocks accounts held in `ActivationState.REQUEST_INVITE_CODE` or `ALREADY_WAITLISTED`. Enforces a mandatory pre-flight handshake query, 6-character alphanumeric normalization, client-side self-referral collision protection, background deferred installation redemption, and compute credit allocation.
2. **Tier B (Peer Invites P2P Subsystem):** A decentralized pairing protocol located under `com.facebook.aura.pairs.PeerInvitesPath` and `PairsInviteMailbox`, enabling authenticated users to exchange rich, media-backed encrypted invitation envelopes directly with recipient devices using Bech32m-addressed IDs and wire-format envelopes.

---

## 2. End-to-End Referral Lifecycle Flow

```mermaid
sequenceDiagram
    autonumber
    participant Invitee as New User / Invitee Client
    participant API as Hatch API Gateway
    participant Sponsor as Active User / Sponsor Client

    Note over Sponsor: 1. Sponsor Generates Referral Assets
    Sponsor->>API: POST /hatch/invite_code/generate
    API-->>Sponsor: GenerateInviteCodeResponseJson (code: "ALP842", max_uses: 5)

    Note over Invitee: 2. Invitee Pre-Flight Handshake (Mandatory)
    Invitee->>API: GET /hatch/invite
    API-->>Invitee: InvitePageResponseJson (code: "OWN991", show_redeem_button: true)

    Note over Invitee: 3. Self-Referral Guard & String Normalization
    Invitee->>Invitee: Assert ("ALP842" != "OWN991")
    Invitee->>Invitee: code = normalizeInviteCodeInput(" alp-842 ") -> "ALP842" (length == 6)

    Note over Invitee: 4. Redemption Mutation
    Invitee->>API: POST /hatch/invite_code/redeem (code: "ALP842")
    API-->>Invitee: {"waitlist_status": "OFF_WAITLIST"}
    Note over Invitee: Account unblocked; transitions out of REQUEST_INVITE_CODE

    Note over Sponsor,Invitee: 5. (Tier B) Rich P2P Media Invite Flow
    Sponsor->>API: POST /peer_invite_photos/upload (jpeg_b64: "...", expires_at: 1794098000000)
    API-->>Sponsor: {"sha256": "3a7b1c4e9f8a2b5d..."}
    Sponsor->>Sponsor: Encode wire envelope into Base64Url (invite_b64)
    Sponsor->>API: POST /peer_invites/send (recipient, invite_b64, expires_at_ms)
    API-->>Sponsor: {"status": "sent", "delivery_id": "deliv_01h9e48f..."}
    Invitee->>API: POST /peer_invites/pending
    API-->>Invitee: PeerInvitePendingResponseJson (items: [delivery_id, invite_b64, ...])
    Invitee->>API: POST /peer_invites/mark_delivered (delivery_id)
    Invitee->>API: POST /peer/accept (fragment: "...")
    API-->>Invitee: PeerInviteAcceptResponseJson (muse_id: "muse1...", relationship_generation_id: "...")
```

---

## 3. Specialized Deep-Dive Sub-Dossiers

For complete step-by-step specifications, verified decompiled bytecode evidence, and wire schemas for each tier:
- 🎟️ [`TIER_A_GROWTH_REDEMPTION.md`](file:///home/ubuntu/projects/beta/referral/TIER_A_GROWTH_REDEMPTION.md) — Pre-flight handshake query `GET /hatch/invite`, client-side self-referral collision guard, 6-character alphanumeric normalization, mutation `POST /hatch/invite_code/redeem`, `HatchInviteCodeRedeemStartupJob` with Google Play Install Referrer, sponsor code generation `POST /hatch/invite_code/generate`, and RFC 7807 error taxonomy.
- 🤝 [`TIER_B_PEER_INVITES_P2P.md`](file:///home/ubuntu/projects/beta/referral/TIER_B_PEER_INVITES_P2P.md) — Subsystem `com.facebook.aura.pairs`, all 7 `PeerInvitesPath` endpoints + `/peer/accept`, BIP-0350 Bech32m address encoding (`muse1...`), Base64Url wire envelope format (`invite_b64`), and SHA-256 media thumbnail upload (`POST /peer_invite_photos/upload`).
