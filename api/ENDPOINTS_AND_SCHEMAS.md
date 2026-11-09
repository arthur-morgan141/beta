# Meta Muse v15: Complete Endpoints & Formal Wire Schemas Catalogue

This document provides the formal wire schemas (JSON request/response models) decompiled directly from `classes.dex` through `classes6.dex` of `binary/15.0.0.3.156.apk` (`com.facebook.aura` v15.0.0.3.156, Version Code `1062001464`).

---

## 1. Authentication & Registration Endpoints

### 1.1 Inisiasi Sesi & Prefetch Kunci (`POST /hatch/auth/start`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthStart`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.start`
- **Request Schema (`AuthStartRequestJson`):**
```json
{
  "id_for_aymh": "usr_01h9e48f7a99b2401bcdae8901",
  "device_id": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01"
}
```
- **Response Schema (`AuthStartResponseJson`):**
```json
{
  "access_token": "EAAX...anonymous_preauth_token...",
  "id_for_aymh": "usr_01h9e48f7a99b2401bcdae8901"
}
```

### 1.2 Dispatch Kontak OTP (`POST /hatch/auth/send_otp`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthSendOtp`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.sendOtp`
- **Request Schema (`AuthSendOtpRequestJson`):**
```json
{
  "contact_point": "developer@example.com",
  "region": "US",
  "auth_state": "st_auth_01h9e48f7a99b2401bcdae8901",
  "registration_source": "native_onboarding",
  "supports_xapp_two_factor_ntp_upsell": false,
  "supports_ntp_upsell_account_choice": false,
  "supported_landing_presentations": [
    "bottom_sheet",
    "full_screen"
  ],
  "supports_meta_two_factor": true,
  "device_id": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01"
}
```
- **Response Schema (`AuthSendOtpResponseJson`):**
```json
{
  "registration_state": "EXISTING_USER",
  "auth_state": "st_auth_849201bcdae8901",
  "password_login_available": true,
  "flow": "login",
  "ntp_source_app": null,
  "has_muse_account": true,
  "password_copy": {
    "title": "Welcome Back",
    "body": "Enter your password to sign into your Meta Muse account.",
    "primary_button_text": "Continue",
    "secondary_button_text": "Reset Password"
  }
}
```

### 1.3 AYMH Saved Account One-Tap OTP (`POST /hatch/auth/send_saved_account_otp`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthSendSavedAccountOtp`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.sendSavedAccountOtp`
- **Request Schema (`SendSavedAccountOtpRequestJson`):**
```json
{
  "account_id": "usr_01h9e48f7a99b2401bcdae8901",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01"
}
```
- **Response Schema:**
```json
{
  "status": "OTP_SENT_TO_ASSOCIATED_DEVICE"
}
```

### 1.4 Pilihan Autentikasi / Passkey Options (`POST /hatch/auth/login_options`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthLoginOptions`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.loginOptions`
- **Request Schema (`AuthLoginOptionsRequestJson`):**
```json
{
  "contact_point": "developer@example.com",
  "region": "US",
  "device_id": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01"
}
```
- **Response Schema (`AuthLoginOptionsResponseJson`):**
```json
{
  "registration_state": "EXISTING_USER",
  "auth_state": "st_auth_849201bcdae8901",
  "password_login_available": true,
  "flow": "passkey_or_password",
  "ntp_source_app": null,
  "passkey_request_options": {
    "challenge": "dGVzdF9jaGFsbGVuZ2Vfc3RyaW5n",
    "rp_id": "meta.ai",
    "user_verification": "required"
  }
}
```

### 1.5 Verifikasi Kode OTP (`POST /hatch/auth/confirm_otp`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthConfirmOtp`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.confirmOtp`
- **Request Schema (`AuthConfirmOtpRequestJson`):**
```json
{
  "auth_state": "st_auth_849201bcdae8901",
  "otp_code": "849201",
  "supports_account_selection": true,
  "supports_muse_link_transparency": true,
  "supports_reactivation": true,
  "supports_inactive_account_labels": true,
  "supported_disclosure_presentations": [
    "bottom_sheet",
    "full_screen"
  ],
  "supported_ntp_upsell_disclosure_presentations": [
    "bottom_sheet"
  ],
  "device_id": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01"
}
```
- **Response Schema (`AuthRegisterCompletionResponseJson` atau Handle Baru):**
```json
{
  "abra_access_token": "EAAX...user_authenticated_token...",
  "abra_user_id": "usr_9921849102",
  "frl_access_token": "EAAX...frl_token...",
  "frl_account_id": "frl_acc_01h9e48f7a99b2401bcdae8901",
  "refresh_token": "rft_sec_99a8b7c6d5e4f3a2b1"
}
```

### 1.6 Finalisasi Registrasi Akun Standalone (`POST /hatch/auth/register_completion`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthRegisterCompletion`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.registerCompletion`
- **Request Schema (`AuthRegisterCompletionRequestJson`):**
```json
{
  "completion_handle": "hdl_sec_01h9e48f7a99b2401bcdae8901",
  "contact_point": "developer@example.com",
  "region": "US",
  "date_of_birth": "1995-06-15",
  "first_name": "Aura",
  "last_name": "User",
  "password": "#PWD_ENC:2:1794011607:AbCdEf123456...",
  "tos_content_version": "v1.0",
  "device_id": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01",
  "attestation": null
}
```
- **Response Schema (`AuthRegisterCompletionResponseJson`):**
```json
{
  "abra_access_token": "EAAX...user_authenticated_token...",
  "abra_user_id": "usr_9921849102",
  "frl_access_token": "EAAX...frl_token...",
  "frl_account_id": "frl_acc_01h9e48f7a99b2401bcdae8901",
  "refresh_token": "rft_sec_99a8b7c6d5e4f3a2b1"
}
```

### 1.7 Finalisasi Registrasi Akun Terhubung / Linked (`POST /hatch/auth/register_completion_linked`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AuthRegisterCompletionLinked`
- **Controller Implementation:** `com.facebook.aura.login.nativeflow.HatchAuthApi.registerCompletionLinked`
- **Request Schema (`AuthRegisterCompletionLinkedRequestJson`):**
```json
{
  "link_handle": "lnk_hdl_01h9e48f7a99b2401bcdae8901",
  "supports_registration_recovery": true,
  "contact_point": "developer@example.com",
  "region": "US",
  "tos_content_version": "v1.0",
  "device_id": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1",
  "waterfall_id": "e4b1a82f-7c91-4901-bf12-984011607a01",
  "supported_disclosure_presentations": [
    "bottom_sheet"
  ],
  "linking_disclosure_version": "2026.11",
  "attestation": null
}
```
- **Response Schema (`AuthRegisterCompletionResponseJson`):**
```json
{
  "abra_access_token": "EAAX...user_authenticated_token...",
  "abra_user_id": "usr_9921849102",
  "frl_access_token": "EAAX...frl_token...",
  "frl_account_id": "frl_acc_01h9e48f7a99b2401bcdae8901",
  "refresh_token": "rft_sec_99a8b7c6d5e4f3a2b1"
}
```

### 1.8 Modul Autentikasi Tambahan
- `POST /hatch/auth/confirm_passkey` — Verifikasi asersi WebAuthn/Passkey.
- `POST /hatch/auth/confirm_password` — Verifikasi kredensial sandi `#PWD_ENC:2`.
- `POST /hatch/auth/two_factor` — Verifikasi tantangan OTP/TOTP Two-Factor.
- `POST /hatch/auth/two_factor/send_code` — Dispatch ulang kode challenge 2FA.
- `POST /hatch/auth/cancel_deletion` — Pembatalan jadwal penghapusan akun.
- `POST /hatch/auth/dbl_login` — Device-based account quick login.
- `POST /hatch/auth/dbl_provision` — Registrasi token biometrik perangkat DBL.
- `POST /hatch/auth/sso_exchange` — Pertukaran token SSO cross-app Meta.

---

## 2. Token Lifecycle & Management Endpoints

### 2.1 Silent Token Renewal (`POST /hatch/refresh_access_token`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.RefreshAccessToken`
- **Request Schema (`RefreshAccessTokenRequestJson`):**
```json
{
  "refresh_token": "rft_sec_99a8b7c6d5e4f3a2b1"
}
```
- **Response Schema (`RefreshAccessTokenResponseJson`):**
```json
{
  "access_token": "EAAX...new_access_token...",
  "refresh_token": "rft_sec_rotated_new_token...",
  "expires_in": 3600
}
```

### 2.2 Sesi Logout & Pencabutan Kunci (`POST /hatch/logout`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.Logout`
- **Request Schema:** `{}`
- **Response Schema:** `{"status": "LOGGED_OUT"}`

---

## 3. Activation Gatekeeper, Legal & Terms of Service Endpoints

### 3.1 Evaluasi Gerbang Aktivasi 24-Field (`POST /hatch/activation_statuses`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.ActivationStatuses`
- **Query Parameter:** `?supported_retos_presentations[0]=bottom_sheet` (mendukung indeks `supported_retos_presentations[i]=...`)
- **Request Schema:** `{}`
- **Response Schema (`ActivationStatusesResponseJson` - 24 Verified Fields):**
```json
{
  "has_hatch_access": true,
  "has_cleared_acquisition": true,
  "invite_code_entry_available": true,
  "waitlist_available": false,
  "has_provisioned_vm": false,
  "waitlist_status": "OFF_WAITLIST",
  "has_redeemed_invite_code": false,
  "ccv_status": "not_required",
  "has_completed_hatch_activation": false,
  "is_in_geoblocked_region": false,
  "hatch_wearables_linked": false,
  "hatch_wearables_comms_consent": null,
  "is_nda_required": false,
  "is_vm_type_selection_required": true,
  "requires_name": false,
  "is_vm_type_plan_required": false,
  "name_prefill": "Aura User",
  "email_address_prefill": "developer@example.com",
  "waitlist_choice_screen": null,
  "enter_invite_code_screen": null,
  "waitlisted_screen": null,
  "waitlisted_limited_screen": null,
  "retos": null,
  "name_dialog": null
}
```

### 3.2 Status Persetujuan & Tampilan Ketentuan Layanan (`GET /hatch/tos_status`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.TosStatus`
- **Controller Implementation:** `com.facebook.aura.activation.repo.AuraActivationRepository.refreshTosStatus`
- **HTTP Method:** `GET`
- **Request Schema:** `{}`
- **Response Schema (`TosStatusResponseJson`):**
```json
{
  "has_accepted_tos": false,
  "disclosure_text": "By tapping Continue, you agree to the Meta Muse Terms of Service.",
  "disclosure_version": "2026.11.01",
  "disclosure_entities": [
    {
      "entity_type": "terms",
      "url": "https://meta.ai/terms"
    },
    {
      "entity_type": "privacy_policy",
      "url": "https://meta.ai/privacy"
    }
  ],
  "disclosure_title": "Terms of Service",
  "disclosure_button_text": "I Agree",
  "disclosure_sections": [],
  "cvm_upsell_cell": {
    "title": "Secure Enclave Compute",
    "subtitle": "Dedicated hardware confidential VM included"
  },
  "subscription_upsell_cell": null,
  "subscription_disclosure_text": "",
  "subscription_disclosure_entities": []
}
```

### 3.3 Persetujuan Terms of Service (`POST /hatch/accept_tos`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AcceptTos`
- **Request Schema (`AcceptTosRequestJson`):**
```json
{
  "disclosure_text": "Meta Muse Terms of Service and Privacy Policy",
  "disclosure_version": "2026.11.01",
  "retos_content_version": "v1.0"
}
```
- **Response Schema:** `{"accepted": true}`

### 3.4 Channel-Bound Consent (`POST /hatch/accept-channel-tos`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.AcceptChannelTos`
- **Request Schema:** `{"tos_name": "channel_agent_tos_v1"}`
- **Response Schema:** `{"accepted": true}`

### 3.5 Dynamic NDA Agreement (`POST /hatch/agreements/{agreement_id}/accept`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.OneClickAgreementAccept`
- **Request Schema (`NdaAcceptRequestJson`):**
```json
{
  "disclosure_text": "Meta Confidential Non-Disclosure Agreement...",
  "disclosure_version": "2026.11"
}
```
- **Response Schema:** `{"accepted": true}`

---

## 4. Hardware Enclave (CVM) & HSM Vesta Endpoints

### 4.1 Alokasi Enclave CVM Cold Lease (`POST /hatch/cold-lease-cvm`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.ColdLeaseCvm`
- **Controller Implementation:** `com.facebook.aura.gateway.HatchCvmColdLeaseClient`
- **Request Schema (`ColdLeaseCvmRequestJson`):**
```json
{
  "idempotency_key": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1"
}
```
- **Response Schema (`ColdLeaseCvmResponseJson`):**
```json
{
  "vm_id": "cvm_us_east_4b9a8201",
  "created": true
}
```

### 4.2 Standard VM (SVM) Downgrade Pipeline (`POST /hatch/start_svm_downgrade`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.StartSvmDowngrade`
- **Controller Implementation:** `com.facebook.aura.confidentialvm.vmswitch.ConfidentialVmDowngradeOperation`
- **Request Schema:** `{}`
- **Response Schema (`StartSvmDowngradeResponseJson`):**
```json
{
  "new_svm_env_id": "svm_us_east_7c8b1209"
}
```

### 4.3 Migrasi Enclave CVM Antar-Environment (`POST /hatch/start_cvm_migration`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.StartCvmMigration`
- **Request Schema (`StartCvmMigrationRequestJson`):**
```json
{
  "source_env_id": "cvm_us_east_4b9a8201",
  "target_env_id": "cvm_us_west_99a8b7c6"
}
```
- **Response Schema (`StartCvmMigrationResponseJson`):**
```json
{
  "migration_id": "mgr_01h9e48f7a99b2401bcdae8901",
  "status": "INITIALIZED"
}
```

### 4.4 Polling Status Migrasi CVM (`GET /hatch/cvm_migration_status`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.CvmMigrationStatus`
- **Query Parameter:** `?old_env_id={env_id}`
- **Response Schema (`CvmMigrationStatusJson`):**
```json
{
  "migration_status": "COMPLETED",
  "new_env_id": "cvm_us_west_99a8b7c6",
  "progress_pct": 100
}
```

### 4.5 Pemeriksaan Status Penggantian State CVM (`GET /hatch/cvm-replace-state`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.CvmReplaceState`
- **Query Parameter:** `?env_id={env_id}`
- **Response Schema (`CvmReplaceStateResponseJson`):**
```json
{
  "state": "ACTIVE",
  "replacement_env_id": null
}
```

### 4.6 Vesta HSM v2 User Info (`GET /hatch/vesta-auth/user-info`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.VestaUserInfo`
- **Query Parameter:** `?vesta_client_api_version=2`
- **Response Schema (`ConfidentialVmVestaUserInfoResponseJson`):**
```json
{
  "has_vesta_account": true,
  "vesta_user_id": "vusr_01h9e48f7a99b2401bcdae8901",
  "enrolled_pin_revision": 1
}
```

---

## 5. Wearables Integration Subsystem (`com.facebook.aura.partners.wearables`)

### 5.1 Tautan Perangkat Wearables (`POST /hatch/wearables/link` & `DELETE /hatch/wearables/link`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.WearablesLink`
- **Controller Implementation:** `com.facebook.aura.partners.wearables.repo.WearablesLinkingRepository`
- **POST Method:** Mengikat perangkat pintar (Meta Ray-Ban / Oakley smart glasses) ke profil akun.
- **DELETE Method:** Memutuskan tautan perangkat yang sebelumnya terikat.
- **Request Schema:** `{}`
- **Response Schema:** HTTP 200/204 OK (`Unit`)

### 5.2 Penawaran Komplementer Wearables (`POST /hatch/wearables/complimentary_offer`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.WearablesComplimentaryOffer`
- **Request Schema:** `{}`
- **Response Schema (`WearablesComplimentaryOfferJson`):**
```json
{
  "holds_offer": true,
  "offer_status": "AVAILABLE"
}
```

### 5.3 Persetujuan Komunikasi Wearables (`POST /hatch/wearables/comms_consent`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.WearablesCommsConsent`
- **Request Schema (`WearablesCommsConsentJson`):**
```json
{
  "comms_consent_granted": true
}
```
- **Response Schema:** HTTP 200 OK (`Unit`)

---

## 6. Profile, Workspace & Catalog Discovery Endpoints

### 6.1 Pengaturan Profil Nama Pengguna (`POST /hatch/viewer/name`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.SetViewerName`
- **Request Schema (`ViewerNameRequestJson`):**
```json
{
  "first_name": "Aura",
  "last_name": "User"
}
```
- **Response Schema:** `{"success": true}`

### 6.2 Profil Akun Pengguna (`GET /hatch/viewer/profile`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.ViewerProfile`
- **HTTP Method:** `GET`
- **Response Schema:**
```json
{
  "user_id": "usr_9921849102",
  "first_name": "Aura",
  "last_name": "User",
  "email": "developer@example.com",
  "profile_picture_url": "https://scontent.meta.ai/..."
}
```

### 6.3 Katalog Langganan Komputasi (`GET /hatch/subscription/catalog`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.ViewerCatalog`
- **Query Parameters:** `?store_country={country}&is_cvm={true|false}`
- **Response Schema:**
```json
{
  "tier": "tier_pro_cvm",
  "price": "$19.99",
  "currency": "USD",
  "features": [
    "Dedicated Hardware CVM",
    "Unlimited Tool Execution",
    "Priority Tigon Routing"
  ]
}
```

### 6.4 Detail Langganan Pengguna (`GET /hatch/subscription`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.ViewerSubscription`
- **Query Parameters:** `?include=agreement,offers&payment_type=...`
- **Response Schema:**
```json
{
  "status": "ACTIVE",
  "expires_at": 1794098000000,
  "tier": "tier_pro_cvm"
}
```

### 6.5 Pendaftaran Antrean Regional Waitlist (`POST /hatch/waitlist/join`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.WaitlistJoin`
- **Request Schema (`JoinWaitlistRequestJson`):**
```json
{
  "source": "activation_gatekeeper"
}
```
- **Response Schema (`JoinWaitlistResponseJson`):**
```json
{
  "status": "WAITLISTED",
  "eligibility_error": null
}
```

### 6.6 Finalisasi Workspace Onboarding (`POST /api/onboarding/complete`)
- **Request Schema:**
```json
{
  "completed_screens": [
    "name_reveal",
    "avatar_select",
    "disclosure_tos"
  ],
  "selected_preset": "assistant_general"
}
```
- **Response Schema:**
```json
{
  "status": "COMPLETED",
  "workspace_ready": true
}
```

---

## 7. Referral & Invite Code Endpoints (Final Module)

> [!IMPORTANT]
> **Monorepo Engineering Compliance Directive:**
> Sesuai ketentuan *Redeem Invite Code Placement Rule*, seluruh endpoint penukaran kode undangan (`POST /hatch/invite_code/redeem`), mutasi undangan sponsor, serta sub-sistem referral Peer Invites P2P ditempatkan secara mutlak di bagian paling akhir dokumen spesifikasi ini.

### 7.1 Tier A: Server-Side Growth & Waitlist Inspection

#### 1. Handshake Pre-Flight (`GET /hatch/invite`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.Invite`
- **Response Schema (`InvitePageResponseJson`):**
```json
{
  "code": "MUSE-ALPHA-9821",
  "has_redeemed_invite_code": false,
  "invite_link": "https://agent.meta.ai/invite?code=MUSE-ALPHA-9821",
  "invite_state": "ACTIVE",
  "uses_remaining": 5,
  "show_redeem_button": true,
  "shows_gift": true,
  "title": "Claim Compute Credits",
  "subtitle": "Redeem an invite code to activate access.",
  "reward": {
    "reward_type": "CVM_HOURS",
    "amount": 50
  }
}
```

#### 2. Sponsor Invite Code Generation (`POST /hatch/invite_code/generate`)
- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.InviteCodeGenerate`
- **Request Schema:** `{}`
- **Response Schema (`GenerateInviteCodeResponseJson`):**
```json
{
  "code": "MUSE-ALPHA-9821",
  "invite_link": "https://agent.meta.ai/invite?code=MUSE-ALPHA-9821",
  "max_uses": 5,
  "use_count": 0,
  "share_text": "Join me on Meta Muse using my VIP invite code: MUSE-ALPHA-9821"
}
```

#### 3. Penukaran Undangan Rekan Peer (`POST /hatch/peer/accept`)
- **DEX Identifier:** `classes.dex` (`/hatch/peer/accept`)
- **Request Schema (`PeerInviteAcceptRequestJson`):**
```json
{
  "fragment": "frg_sec_01h9e48f7a99b2401bcdae8901"
}
```
- **Response Schema (`PeerInviteAcceptResponseJson`):**
```json
{
  "muse_id": "usr_9921849102",
  "relationship_generation_id": "rel_gen_01h9e48f7a99b2401bcdae8901"
}
```

#### 4. Debug Reset Endpoints (Dogfooding Builds)
- `POST /hatch/debug/reset_invite_code_redemption` -> `{"success": true}`
- `POST /hatch/debug/reset_activation` -> `{"success": true}`

---

### 7.2 Tier B: Sub-Sistem Peer Invites P2P (`com.facebook.aura.pairs`)

#### 1. Pembuatan Undangan Peer Baru (`POST /peer_invites/create`)
- **Request Schema (`PeerInviteCreateRequestJson`):**
```json
{
  "create_request_id": "req_01h9e48f7a99b2401bcdae8901",
  "label": "My Collaborator Invite",
  "offered_preset": "assistant_general",
  "recipient_name": "Bob"
}
```
- **Response Schema (`PeerInviteCreateResponseJson`):**
```json
{
  "expires_at_ms": 1794098000000,
  "fragment": "frg_sec_01h9e48f7a99b2401bcdae8901",
  "invite_b64": "ZXhhbXBsZV9lbmNyeXB0ZWRfZW52ZWxvcGVfcGF5bG9hZA==",
  "invite_id_b64": "aW52aXRlX2lkX2V4YW1wbGU=",
  "owner_photo_jpeg_b64": "/9j/4AAQSkZJRg..."
}
```

#### 2. Dispatch Undangan Enkripsi P2P (`POST /peer_invites/send`)
- **DEX Identifier:** `com.facebook.aura.pairs.PeerInvitesPath.Send`
- **Request Schema (`PeerInviteSendRequestJson`):**
```json
{
  "recipient": {
    "app_target": "aura",
    "recipient_type": "user",
    "recipient_id": "usr_9921849102",
    "sender_account_id": "usr_01h9e48f7a99b2401bcdae8901"
  },
  "invite_b64": "ZXhhbXBsZV9lbmNyeXB0ZWRfZW52ZWxvcGVfcGF5bG9hZA==",
  "expires_at_ms": 1794098000000,
  "offered_preset": "assistant_general"
}
```
- **Response Schema (`PeerInviteSendResponseJson`):**
```json
{
  "status": "SENT",
  "delivery_id": "deliv_01h9e48f7a99b2401bcdae8901"
}
```

#### 3. Polling Undangan Masuk Tertunda (`POST /peer_invites/pending`)
- **DEX Identifier:** `com.facebook.aura.pairs.PeerInvitesPath.Pending`
- **Request Schema:** `{}`
- **Response Schema (`PeerInvitePendingResponseJson`):**
```json
{
  "items": [
    {
      "delivery_id": "deliv_01h9e48f7a99b2401bcdae8901",
      "invite_b64": "ZXhhbXBsZV9lbmNyeXB0ZWRfZW52ZWxvcGVfcGF5bG9hZA==",
      "sender": {
        "display_name": "Alice"
      },
      "created_at": 1794011600000,
      "expires_at": 1794098000000
    }
  ]
}
```

#### 4. Pemeriksaan Status Undangan (`POST /peer_invites/status`)
- **DEX Identifier:** `com.facebook.aura.pairs.PeerInvitesPath.Status`
- **Request Schema (`PeerInviteStatusRequestJson`):**
```json
{
  "delivery_ids": [
    "deliv_01h9e48f7a99b2401bcdae8901"
  ]
}
```
- **Response Schema (`PeerInviteStatusResponseJson`):**
```json
{
  "items": [
    {
      "delivery_id": "deliv_01h9e48f7a99b2401bcdae8901",
      "state": "PENDING"
    }
  ]
}
```

#### 5. Manajemen Status Lifecycle Undangan
- `POST /peer_invites/cancel` (`PeerInviteDeliveryIdJson`) -> `{"success": true}`
- `POST /peer_invites/mark_delivered` (`PeerInviteDeliveryIdJson`) -> `{"success": true}`
- `POST /peer_invites/decline` (`PeerInviteDeliveryIdJson`) -> `{"success": true}`

#### 6. Unggah Foto Media Pendukung Undangan (`POST /peer_invite_photos/upload`)
- **DEX Identifier:** `com.facebook.aura.pairs.PeerInvitesPath.PhotoUpload`
- **Request Stream:** Binary JPEG stream (`Content-Type: image/jpeg`).
- **Response Schema (`PeerInvitePhotoUploadResponseJson`):**
```json
{
  "sha256": "3a7b1c4e9f8a2b5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d"
}
```

---

### 7.3 Modul Final: Mutasi Penukaran Sponsor Invite Code (`POST /hatch/invite_code/redeem`)

> [!IMPORTANT]
> **Engineering Placement Requirement:**
> Modul ini ditempatkan di posisi paling akhir dokumen sesuai *Redeem Invite Code Placement Rule*.

- **DEX Identifier:** `com.facebook.aura.network.api.StefiHatchPath.InviteCodeRedeem`
- **Controller Implementation:** `com.facebook.aura.activation.repo.AuraActivationRepository.redeemInviteCode`
- **HTTP Method:** `POST`
- **Endpoint Path:** `/hatch/invite_code/redeem`
- **Full Wire URL:** `https://hatch-api.meta.ai/hatch/invite_code/redeem`
- **Request Headers:**
  - `Host: hatch-api.meta.ai`
  - `Content-Type: application/json`
  - `X-API-Version: 1.0.0`
  - `Authorization: Bearer <abra_access_token>`
  - `AuraDeviceId: <UPPERCASE_UUID>`
  - `waterfall_id: <LOWERCASE_UUID>`
- **Request Schema (`RedeemInviteCodeRequestJson`):**
```json
{
  "code": "MUSE-ALPHA-8492"
}
```
- **Response Schema (HTTP 200 OK):**
```json
{
  "waitlist_status": "OFF_WAITLIST"
}
```
- **RFC 7807 Error Responses (`InviteCodeServerErrorKt`):**
  - `400 Bad Request` + `{"type": "inviteCodeInvalid", "error_code": "invalid_code", ...}`
  - `400 Bad Request` + `{"type": "inviteCodeRevoked", "error_code": "code_revoked", ...}`
  - `400 Bad Request` + `{"type": "inviteCodeUsedUp", "error_code": "code_used_up", ...}`
  - `400 Bad Request` + `{"type": "inviteCodeWindowExpired", "error_code": "redemption_window_expired", ...}`

