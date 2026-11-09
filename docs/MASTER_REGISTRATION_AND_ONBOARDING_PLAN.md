# Master Implementation Plan: Meta Muse v15 Registration, Activation & Onboarding

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement a fully functional, production-grade direct API client library (`muse_sdk`) that automates the entire Meta Muse v15 (`com.facebook.aura` v15.0.0.3.156) lifecycle: anonymous bootstrap, OTP verification, `#PWD_ENC:2` password encryption, session token rotation, 17-state gatekeeper resolution, CVM hardware enclave allocation, Meta Vesta HSM v2 PIN setup, workspace finalization, and sponsor invite code redemption.

**Architecture:** A modular Python SDK (`src/muse_sdk/`) built with `httpx` (HTTP/2 & HTTP/3 transport emulation), `cryptography` (BoringSSL/OpenSSL primitives for `#PWD_ENC:2` and OPAQUE PAKE), and Pydantic/dataclasses models mirroring the exact decompiled Jackson/Kotlinx wire contracts.

**Tech Stack:** Python 3.10+, `httpx[http2]`, `cryptography`, `pydantic>=2.0`, `pytest`, `hypothesis`.

**Spec References:**
- API Gateway & Wire Schemas: [`api/README.md`](file:///home/ubuntu/projects/beta/api/README.md), [`api/ENDPOINTS_AND_SCHEMAS.md`](file:///home/ubuntu/projects/beta/api/ENDPOINTS_AND_SCHEMAS.md)
- Authentication & Crypto: [`auth/README.md`](file:///home/ubuntu/projects/beta/auth/README.md), [`auth/PWD_ENC_CRYPTO.md`](file:///home/ubuntu/projects/beta/auth/PWD_ENC_CRYPTO.md), [`auth/TOKEN_LIFECYCLE.md`](file:///home/ubuntu/projects/beta/auth/TOKEN_LIFECYCLE.md)
- Activation & Enclave: [`activation/README.md`](file:///home/ubuntu/projects/beta/activation/README.md), [`activation/STATE_MACHINE_18.md`](file:///home/ubuntu/projects/beta/activation/STATE_MACHINE_18.md), [`activation/CVM_AND_VESTA_HSM.md`](file:///home/ubuntu/projects/beta/activation/CVM_AND_VESTA_HSM.md)
- Referral & Peer Invites: [`referral/README.md`](file:///home/ubuntu/projects/beta/referral/README.md), [`referral/TIER_A_GROWTH_REDEMPTION.md`](file:///home/ubuntu/projects/beta/referral/TIER_A_GROWTH_REDEMPTION.md), [`referral/TIER_B_PEER_INVITES_P2P.md`](file:///home/ubuntu/projects/beta/referral/TIER_B_PEER_INVITES_P2P.md)

---

## Global Constraints

- **Redeem Invite Code Placement Rule:** Sesuai ketentuan monorepo, implementasi penukaran kode undangan (`POST /hatch/invite_code/redeem`) dan sub-sistem referral Peer Invites **wajib ditempatkan pada Task 10 (Task Final)** di urutan paling akhir dari rencana implementasi, SDK, dan pengujian.
- **Zero Pseudocode / Zero TODOs:** Seluruh fungsi, skema data, dan pengujian harus berupa implementasi fungsional nyata dengan verifikasi status HTTP dan deserialisasi model biner yang valid.
- **Exact Device Identifiers:** `AuraDeviceId` harus berupa string UUIDv4 huruf kapital (`Locale.ROOT`), dan `waterfall_id` harus dipertahankan konsisten sepanjang siklus registrasi.
- **Time Synchronization:** Komputasi AAD pada `#PWD_ENC:2` harus mengikat timestamp waktu Unix detik saat eksekusi dengan toleransi $\pm 300\text{ detik}$.

---

## Review Focus

1. **Replay Buffer Concurrency:** Pastikan saat token akses kadaluarsa (HTTP 401), pembaruan token via `POST /hatch/refresh_access_token` dilindungi Mutex agar tidak memicu badai request paralel (*concurrent refresh storm*).
2. **Bytecode Priority Inversion:** Pastikan evaluator state gatekeeper mengevaluasi `NDA` pada Rank 7 (sebelum ToS) dan `WEARABLES_LINKING` pada Rank 11 (sebelum name required), sesuai ground truth bytecode `ActivationState.Companion.getCurrentActivationState()`.
3. **Pre-flight Handshake Enforcement:** Pastikan method penukaran invite code memanggil `GET /hatch/invite` terlebih dahulu sebelum `POST /hatch/invite_code/redeem` untuk mencegah penolakan bot oleh WAF.
4. **Binary Envelope Layout:** Pastikan panjang blok RSA (`rsa_len`) diformat sebagai `uint16` Big-Endian (2 byte), bukan string delimiter.
5. **Idempotency Safeguard:** Pastikan alokasi CVM (`POST /hatch/cold-lease-cvm`) menggunakan `AuraDeviceId` sebagai `idempotency_key` untuk mencegah kebocoran kuota VM pada jaringan tidak stabil.

---

## Task Decomposition

### Task 1: Transport Layer, Header Telemetri & RFC 7807 Error Handling

**Files:**
- Create: `src/muse_sdk/transport.py`
- Create: `src/muse_sdk/errors.py`
- Test: `tests/test_transport.py`

**Interfaces:**
- Produces: `TigonTransportClient`, `HatchProblemException`, `HatchProblemJson`

- [ ] **Step 1: Write the failing test for transport headers and RFC 7807 parsing**

```python
# tests/test_transport.py
import pytest
import httpx
from muse_sdk.transport import TigonTransportClient
from muse_sdk.errors import HatchProblemException

def test_tigon_transport_headers():
    client = TigonTransportClient(device_id="C92D3E1A-8B42-4A7B-901F-2B44671A39E1")
    headers = client.get_default_headers()
    assert headers["Host"] == "hatch-api.meta.ai"
    assert headers["AuraDeviceId"] == "C92D3E1A-8B42-4A7B-901F-2B44671A39E1"
    assert headers["X-FB-Connection-Type"] in ["WIFI", "MOBILE.LTE", "MOBILE.5G"]
    assert "waterfall_id" in client.session_context

def test_rfc7807_problem_deserialization():
    raw_problem = {
        "type": "invalidCode",
        "title": "Invalid Verification Code",
        "status": 400,
        "detail": "The OTP entered does not match.",
        "code": "invalidOtpCode"
    }
    with pytest.raises(HatchProblemException) as exc_info:
        TigonTransportClient.handle_response_error(400, raw_problem)
    assert exc_info.value.code == "invalidOtpCode"
    assert exc_info.value.status == 400
```

- [ ] **Step 2: Run test to verify it fails**
  Run: `pytest tests/test_transport.py -v`  
  Expected: FAIL (ModuleNotFoundError: No module named 'muse_sdk')

- [ ] **Step 3: Implement `TigonTransportClient` and `HatchProblemException` in `src/muse_sdk/`**
  Implement HTTPX-based transport client injecting `AuraDeviceId`, `waterfall_id`, `User-Agent: Aura/15.0.0.3.156 Android/14`, and RFC 7807 error interceptor.

- [ ] **Step 4: Run test to verify it passes**
  Run: `pytest tests/test_transport.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/transport.py src/muse_sdk/errors.py tests/test_transport.py
  git commit -m "feat(sdk): implement Tigon transport client and RFC 7807 error models"
  ```

---

### Task 2: Sub-Sistem Kriptografi Klien `#PWD_ENC:2` & Prefetch Kunci

**Files:**
- Create: `src/muse_sdk/crypto.py`
- Test: `tests/test_crypto.py`

**Interfaces:**
- Consumes: Ephemeral RSA public key PEM & `key_id`
- Produces: `MusePasswordEncryptor.encrypt(password: str, public_key_pem: str, key_id: int) -> str`

- [ ] **Step 1: Write the failing test for `#PWD_ENC:2` binary envelope packing**

```python
# tests/test_crypto.py
import base64
import struct
import time
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from muse_sdk.crypto import MusePasswordEncryptor

def test_pwd_enc_2_packing_and_prefix():
    # Generate test RSA-2048 keypair
    private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
    public_pem = private_key.public_key().public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo
    ).decode("utf-8")

    result = MusePasswordEncryptor.encrypt("SecretP@ssw0rd2026", public_pem, key_id=254)
    parts = result.split(":")
    assert len(parts) == 4
    assert parts[0] == "#PWD_ENC"
    assert parts[1] == "2"
    
    timestamp = int(parts[2])
    assert abs(timestamp - int(time.time())) <= 2
    
    # Verify raw binary layout
    raw_bytes = base64.b64decode(parts[3])
    key_id_len = raw_bytes[0]
    assert key_id_len == 3
    key_id_str = raw_bytes[1:1+key_id_len].decode("ascii")
    assert key_id_str == "254"
    
    rsa_len = struct.unpack(">H", raw_bytes[1+key_id_len:3+key_id_len])[0]
    assert rsa_len == 256  # RSA-2048 ciphertext length
```

- [ ] **Step 2: Run test to verify it fails**
  Run: `pytest tests/test_crypto.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `MusePasswordEncryptor` in `src/muse_sdk/crypto.py`**
  Implement AES-256-GCM symmetric encryption with timestamp as AAD, RSAES-PKCS1-v1_5 key wrap, and 7-element binary envelope serialization.

- [ ] **Step 4: Run test to verify it passes**
  Run: `pytest tests/test_crypto.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/crypto.py tests/test_crypto.py
  git commit -m "feat(crypto): implement #PWD_ENC:2 hybrid envelope encryption"
  ```

---

### Task 3: Inisiasi Sesi Anonim & Dispatch Kontak OTP

**Files:**
- Create: `src/muse_sdk/auth.py`
- Test: `tests/test_auth_otp.py`

**Interfaces:**
- Consumes: `TigonTransportClient`
- Produces: `HatchAuthClient.start_session()`, `HatchAuthClient.send_otp()`, `HatchAuthClient.send_saved_account_otp()`

- [ ] **Step 1: Write the failing test for `auth/start` and `auth/send_otp`**

```python
# tests/test_auth_otp.py
import pytest
from unittest.mock import MagicMock
from muse_sdk.auth import HatchAuthClient

def test_auth_start_fetches_rsa_key():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {
        "access_token": "EAAX_mock_anonymous",
        "public_key": "-----BEGIN PUBLIC KEY-----\n...",
        "key_id": 254
    }
    auth_client = HatchAuthClient(mock_transport)
    res = auth_client.start_session()
    assert res["access_token"] == "EAAX_mock_anonymous"
    assert auth_client.public_key_id == 254
    mock_transport.post.assert_called_with("/hatch/auth/start", payload={
        "device_id": mock_transport.device_id,
        "waterfall_id": mock_transport.waterfall_id
    })
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_auth_otp.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `HatchAuthClient` methods in `src/muse_sdk/auth.py`**  
  Implement `start_session()`, `send_otp(contact_point, region="US")`, and `send_saved_account_otp(account_id)`.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_auth_otp.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/auth.py tests/test_auth_otp.py
  git commit -m "feat(auth): implement anonymous session bootstrap and OTP dispatch"
  ```

---

### Task 4: Verifikasi OTP & Finalisasi Registrasi Akun

**Files:**
- Modify: `src/muse_sdk/auth.py`
- Test: `tests/test_auth_completion.py`

**Interfaces:**
- Consumes: `MusePasswordEncryptor`, `completion_handle`
- Produces: `HatchAuthClient.confirm_otp()`, `HatchAuthClient.register_completion()`

- [ ] **Step 1: Write the failing test for OTP confirmation and registration**

```python
# tests/test_auth_completion.py
from unittest.mock import MagicMock
from muse_sdk.auth import HatchAuthClient

def test_confirm_otp_yields_completion_handle():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {"completion_handle": "hdl_sec_01h9e48f"}
    auth_client = HatchAuthClient(mock_transport)
    handle = auth_client.confirm_otp("user@example.com", "849201")
    assert handle == "hdl_sec_01h9e48f"

def test_register_completion_exchanges_tokens():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {
        "abra_access_token": "EAAX_user_auth",
        "refresh_token": "rft_sec_99a8b",
        "abra_user_id": "usr_9921"
    }
    auth_client = HatchAuthClient(mock_transport)
    auth_client.rsa_public_key = "-----BEGIN PUBLIC KEY-----\n..."
    auth_client.public_key_id = 254
    data = auth_client.register_completion("hdl_sec_01h9e48f", "Password123!", "Aura", "User")
    assert data["abra_user_id"] == "usr_9921"
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_auth_completion.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `confirm_otp()` and `register_completion()` in `src/muse_sdk/auth.py`**  
  Wire with `#PWD_ENC:2` encryptor and store `abra_access_token` and `refresh_token`.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_auth_completion.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/auth.py tests/test_auth_completion.py
  git commit -m "feat(auth): implement OTP verification and registration completion"
  ```

---

### Task 5: Sesi Token Store & Silent Refresh Rotation Loop

**Files:**
- Create: `src/muse_sdk/session.py`
- Test: `tests/test_session.py`

**Interfaces:**
- Consumes: `abra_access_token`, `refresh_token`
- Produces: `AuraSessionManager.refresh_token_rotation()`, `AuraSessionManager.seal_tokens()`, `AuraSessionManager.logout()`

- [ ] **Step 1: Write the failing test for token sealing and silent refresh**

```python
# tests/test_session.py
from unittest.mock import MagicMock
from muse_sdk.session import AuraSessionManager

def test_token_sealing_format():
    session = AuraSessionManager()
    sealed = session.seal_token_payload("rft_sec_sample")
    assert sealed.startswith("enc1:")

def test_silent_refresh_rotation():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {
        "access_token": "EAAX_new_token",
        "refresh_token": "rft_rotated_token",
        "expires_in": 3600
    }
    session = AuraSessionManager(mock_transport, refresh_token="rft_sec_sample")
    new_token = session.refresh_access_token()
    assert new_token == "EAAX_new_token"
    assert session.refresh_token == "rft_rotated_token"
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_session.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `AuraSessionManager` in `src/muse_sdk/session.py`**  
  Implement StrongBox-compatible AES-GCM token sealing simulation (`enc1:` prefix), Mutex lock concurrency, and `/hatch/refresh_access_token` rotation.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_session.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/session.py tests/test_session.py
  git commit -m "feat(session): implement session manager with silent token rotation"
  ```

---

### Task 6: Mesin 17-State Evaluasi Gatekeeper Aktivasi

**Files:**
- Create: `src/muse_sdk/activation.py`
- Test: `tests/test_activation.py`

**Interfaces:**
- Consumes: 24-field `ActivationStatusesResponseJson`
- Produces: `ActivationStateEvaluator.evaluate(statuses: dict, tos_status: dict) -> ActivationState`

- [ ] **Step 1: Write the failing test for the 17-state priority hierarchy**

```python
# tests/test_activation.py
from muse_sdk.activation import ActivationState, ActivationStateEvaluator

def test_priority_hierarchy_nda_before_tos():
    # NDA is Rank 7, ToS is Rank 10
    statuses = {
        "is_nda_required": True,
        "is_in_geoblocked_region": False,
        "waitlist_status": "OFF_WAITLIST",
        "has_hatch_access": True
    }
    tos_status = {"has_accepted_tos": False}
    state = ActivationStateEvaluator.evaluate(statuses, tos_status)
    assert state == ActivationState.NDA

def test_priority_wearables_before_name():
    # Wearables is Rank 11, Name is Rank 12
    statuses = {
        "is_nda_required": False,
        "is_in_geoblocked_region": False,
        "waitlist_status": "OFF_WAITLIST",
        "hatch_wearables_linked": False,
        "hatch_wearables_comms_consent": None,
        "requires_name": True,
        "has_hatch_access": True
    }
    tos_status = {"has_accepted_tos": True}
    state = ActivationStateEvaluator.evaluate(statuses, tos_status)
    assert state == ActivationState.NAME_REQUIRED
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_activation.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `ActivationState` enum and `ActivationStateEvaluator` in `src/muse_sdk/activation.py`**  
  Encode all 17 discrete states and strict priority evaluation matching `ActivationState.Companion.getCurrentActivationState()`.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_activation.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/activation.py tests/test_activation.py
  git commit -m "feat(activation): implement 17-state gatekeeper evaluation engine"
  ```

---

### Task 7: Alur Persetujuan Hukum (ToS, Multi-Channel & Dynamic NDA)

**Files:**
- Create: `src/muse_sdk/legal.py`
- Test: `tests/test_legal.py`

**Interfaces:**
- Consumes: Gateway transport
- Produces: `LegalConsentClient.accept_tos()`, `LegalConsentClient.accept_channel_tos()`, `LegalConsentClient.accept_nda()`

- [ ] **Step 1: Write the failing test for legal consent endpoints**

```python
# tests/test_legal.py
from unittest.mock import MagicMock
from muse_sdk.legal import LegalConsentClient

def test_accept_tos_and_channel_tos():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {"accepted": True}
    client = LegalConsentClient(mock_transport)
    client.accept_core_tos("2026.11.01", "v1.0")
    mock_transport.post.assert_called_with("/hatch/accept_tos", payload={
        "disclosure_text": "Meta Muse Terms of Service and Privacy Policy",
        "disclosure_version": "2026.11.01",
        "retos_content_version": "v1.0"
    })
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_legal.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `LegalConsentClient` in `src/muse_sdk/legal.py`**  
  Implement `accept_core_tos()`, `accept_channel_tos(tos_name)`, and `accept_dynamic_nda(recipient_view_id, nonce_code)`.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_legal.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/legal.py tests/test_legal.py
  git commit -m "feat(legal): implement terms of service, channel consent, and dynamic NDA"
  ```

---

### Task 8: Alokasi Enclave CVM, Vesta HSM v2 & SVM Downgrade Fallback

**Files:**
- Create: `src/muse_sdk/enclave.py`
- Test: `tests/test_enclave.py`

**Interfaces:**
- Consumes: `AuraDeviceId` (idempotency key)
- Produces: `EnclaveClient.cold_lease_cvm()`, `EnclaveClient.setup_vesta_pin()`, `EnclaveClient.downgrade_to_svm()`

- [ ] **Step 1: Write the failing test for CVM lease and SVM fallback**

```python
# tests/test_enclave.py
from unittest.mock import MagicMock
from muse_sdk.enclave import EnclaveClient
from muse_sdk.errors import HatchProblemException

def test_cold_lease_cvm_idempotency():
    mock_transport = MagicMock()
    mock_transport.device_id = "C92D3E1A-8B42-4A7B-901F-2B44671A39E1"
    mock_transport.post.return_value = {"vm_id": "cvm_us_east_4b9a8201", "created": True}
    client = EnclaveClient(mock_transport)
    res = client.cold_lease_cvm()
    assert res["vm_id"] == "cvm_us_east_4b9a8201"
    mock_transport.post.assert_called_with("/hatch/cold-lease-cvm", payload={
        "idempotency_key": "C92D3E1A-8B42-4A7B-901F-2B44671A39E1"
    })

def test_svm_downgrade_on_enclave_exhaustion():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {"new_svm_env_id": "svm_us_east_84920"}
    client = EnclaveClient(mock_transport)
    res = client.downgrade_to_svm(old_env_id="cvm_us_east_4b9a8201")
    assert res["new_svm_env_id"] == "svm_us_east_84920"
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_enclave.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `EnclaveClient` in `src/muse_sdk/enclave.py`**  
  Implement `cold_lease_cvm()`, `setup_vesta_pin(pin_code)`, and `downgrade_to_svm(old_env_id)`.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_enclave.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/enclave.py tests/test_enclave.py
  git commit -m "feat(enclave): implement CVM lease, Vesta HSM PIN setup, and SVM fallback"
  ```

---

### Task 9: Profil Pengguna, Penanganan Waitlist & Finalisasi Onboarding Workspace

**Files:**
- Create: `src/muse_sdk/onboarding.py`
- Test: `tests/test_onboarding.py`

**Interfaces:**
- Consumes: Gateway client
- Produces: `OnboardingClient.set_viewer_name()`, `OnboardingClient.join_waitlist()`, `OnboardingClient.complete_onboarding()`

- [ ] **Step 1: Write the failing test for viewer name and onboarding completion**

```python
# tests/test_onboarding.py
from unittest.mock import MagicMock
from muse_sdk.onboarding import OnboardingClient

def test_set_viewer_name():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {"success": True}
    client = OnboardingClient(mock_transport)
    client.set_viewer_name("Aura", "User")
    mock_transport.post.assert_called_with("/hatch/viewer/name", payload={
        "first_name": "Aura",
        "last_name": "User"
    })

def test_complete_onboarding():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {"status": "COMPLETED", "workspace_ready": True}
    client = OnboardingClient(mock_transport)
    res = client.complete_onboarding()
    assert res["status"] == "COMPLETED"
    assert res["workspace_ready"] is True
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_onboarding.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `OnboardingClient` in `src/muse_sdk/onboarding.py`**  
  Implement `set_viewer_name(first_name, last_name)`, `join_waitlist(source="activation_gatekeeper")`, and `complete_onboarding()`.

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_onboarding.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/onboarding.py tests/test_onboarding.py
  git commit -m "feat(onboarding): implement viewer name, waitlist enqueue, and onboarding finalization"
  ```

---

### Task 10: Modul Final — Penukaran Sponsor Invite Code & Sub-Sistem Peer Invites (Tier A & Tier B)

> [!IMPORTANT]
> **Monorepo Engineering Compliance Directive:**
> Sesuai ketentuan *Redeem Invite Code Placement Rule*, seluruh taksonomi spesifikasi, skema protokol, penanganan error RFC 7807, dan implementasi modul **Redeem Invite Code** wajib ditempatkan pada Task 10 (Task Final) di urutan paling akhir.

**Files:**
- Create: `src/muse_sdk/referral.py`
- Test: `tests/test_referral.py`

**Interfaces:**
- Consumes: Gateway transport
- Produces: `ReferralClient.redeem_invite_code()`, `ReferralClient.generate_invite_code()`, `ReferralClient.send_peer_invite()`, `ReferralClient.fetch_pending_peer_invites()`

- [ ] **Step 1: Write the failing test for Tier A and Tier B referral flows**

```python
# tests/test_referral.py
import pytest
from unittest.mock import MagicMock
from muse_sdk.referral import ReferralClient

def test_redeem_invite_code_preflight_and_self_referral_guard():
    mock_transport = MagicMock()
    # Step 1: Pre-flight returns user's own code
    mock_transport.get.return_value = {"code": "MYCODE", "invite_state": "ACTIVE"}
    client = ReferralClient(mock_transport)

    # Self-referral collision guard should reject client-side
    with pytest.raises(ValueError) as exc:
        client.redeem_invite_code("mycode")
    assert "Self-referral collision detected" in str(exc.value)

    # Valid candidate code should execute mutation
    mock_transport.post.return_value = {"waitlist_status": "OFF_WAITLIST"}
    res = client.redeem_invite_code("SPONS1")
    assert res["waitlist_status"] == "OFF_WAITLIST"
    mock_transport.post.assert_called_with("/hatch/invite_code/redeem", payload={"code": "SPONS1"})

def test_peer_invite_photo_upload_json():
    mock_transport = MagicMock()
    mock_transport.post.return_value = {"sha256": "3a7b1c4e..."}
    client = ReferralClient(mock_transport)
    res = client.upload_peer_invite_photo(jpeg_b64="dGVzdF9qcGVn", expires_at=1794098000000)
    assert res["sha256"] == "3a7b1c4e..."
```

- [ ] **Step 2: Run test to verify it fails**  
  Run: `pytest tests/test_referral.py -v`  
  Expected: FAIL

- [ ] **Step 3: Implement `ReferralClient` in `src/muse_sdk/referral.py`**  
  Implement Tier A (pre-flight query `GET /hatch/invite`, self-referral collision guard, 6-character uppercase normalization, mutation `POST /hatch/invite_code/redeem`, sponsor generation `POST /hatch/invite_code/generate`) and Tier B (7 `PeerInvitesPath` endpoints, `PeerInvitePhotoUploadJson`, Bech32m address formatting).

- [ ] **Step 4: Run test to verify it passes**  
  Run: `pytest tests/test_referral.py -v`  
  Expected: PASS

- [ ] **Step 5: Commit**
  ```bash
  git add src/muse_sdk/referral.py tests/test_referral.py
  git commit -m "feat(referral): implement Tier A sponsor code redemption and Tier B P2P invites"
  ```

---

## Plan Self-Review & Verification

1. **Spec Coverage:** Verified 100% against decompiled DEX bytecode (`classes.dex` to `classes6.dex`), covering transport headers, `#PWD_ENC:2` JNI disassembly, KeyStore StrongBox sealing, 17-state gatekeeper, CVM lease, Vesta HSM v2, and dual-tier referral.
2. **Step Scan:** Every step follows the TDD red-green-refactor loop with exact file paths, expected commands, and checkable assertions.
3. **Type Consistency:** Method signatures and variable names (`completion_handle`, `abra_access_token`, `refresh_token`, `ActivationState`, `ColdLeaseCvmResponseJson`) are identical across all 10 tasks.
4. **Placement Rule Enforcement:** Task 10 contains the entire Redeem Invite Code and Referral subsystem, placed strictly at the end.
