# Technical Specification & Guidelines: Meta Muse Monorepo (muse-rea)

## 1. Project Identity & Target
- **Project Name:** Meta Muse (Project Aura / Hatch) Reverse Engineering & Automation Suite
- **Target Binaries:**
  - `binary/base-14.0.0.4.149.apk` — Rilis target produksi terbaru (`v14.0.0.4.149`, Version Code `1061901406`, `com.facebook.aura`)
  - `binary/base-13.0.0.4.159.apk` — Target biner v13 (`v13.0.0.4.159`, Version Code `1061801514`, `com.facebook.aura`)
  - `binary/base-11.0.0.18.164.apk` — Target biner baseline (`v11.0.0.18.164`, Version Code `1061601472`)
- **Struktur Unified Modern Monorepo:**
  - `src/muse_sdk/` — Paket Python SDK resmi (PEP 621 / Modern Python layout, crypto `#PWD_ENC:2`, client sync & async, scheduler, proxy fleet, CLI).
  - `tests/` — Suite pengujian unit & property-based testing (125 test cases dengan `pytest` & `hypothesis`).
  - `scripts/` — Skrip eksekusi dan referensi cURL terverifikasi (`curl_reference.sh`).
  - `api/` — Taksonomi lengkap 96 endpoint, spesifikasi formal OpenAPI 3.1, dan skema JSON RFC 7807.
  - `audits/` — 11 Dossier audit arsitektur sistem mendalam (v11 & v13).
  - `docs/` — Dokumentasi arsitektur onboarding (`ACCOUNT_REGISTRATION_AND_ONBOARDING_FLOW.md`) dan bukti native crypto bridge (`02_CRYPTO_AND_JNI_BRIDGES.md`).
  - `plans/` — 7 Dokumen perencanaan teknis reverse engineering dan otomatisasi.
  - `binary/` — Artefak biner target APK dan verifikasi `checksums.txt`.
  - `rea-output/` — Bukti dekompilasi bytecode DEX, session bundle REA, dan analisis biner native.
  - `.agents/skills/` — Harness 6 Agent Skills publik kredibel untuk analisis biner, kriptografi, dan kepatuhan kode.
  - `pyproject.toml`, `Makefile`, `uv.lock`, `.python-version` — Konfigurasi build, lint, dan test terpadu di root.

---

## 2. Machine Environment & Installed Tooling
Mesin ini sudah dilengkapi dengan seluruh perkakas analisis biner dan reverse engineering. Selalu gunakan perkakas lokal berikut tanpa perlu menginstal ulang:

- **REA (Reverse Engineer Anything):**
  - **Biner CLI:** `/home/ubuntu/.local/bin/rea` & `/home/ubuntu/.local/bin/rea-agents` (versi `6.8.0`)
  - **MCP Integration:** Terdaftar di Antigravity (`~/.gemini/config/mcp_config.json`) dan Hermes.
  - **Skill:** `reverse-engineer-anything` aktif di `~/.gemini/config/skills/`.
- **Apktool:**
  - **Biner:** `/home/ubuntu/.local/bin/apktool` (versi `3.0.3` upstream, mendukung Android 14–16 / API 36).
  - **Lokasi JAR:** `/home/ubuntu/.local/share/apktool/apktool.jar`.
- **Ghidra Suite:**
  - **Home:** `/home/ubuntu/.local/share/ghidra/ghidra_12.1.4_PUBLIC` (versi `12.1.4`).
  - **Headless Decompiler:** `/home/ubuntu/.local/share/ghidra/ghidra_12.1.4_PUBLIC/support/analyzeHeadless`.
- **Hopper Disassembler:**
  - **Biner:** `/opt/hopper/bin/Hopper` (versi `6.6.0`, terverifikasi PGP).
- **JADX Headless MCP Bridge:**
  - **JAR:** `/home/ubuntu/.local/share/rea/android/jadx-headless-mcp-0.7.1-all.jar`.
- **Platform & Debugger Tools:**
  - `adb` di `/usr/bin/adb`
  - `gdb` di `/usr/bin/gdb`
- **Runtimes & Compiler:**
  - **Node.js & npm:** `v24.22.1` / npm `11.19.1` (path `/home/ubuntu/.local/bin/node`, dikelola via NVM).
  - **Java:** OpenJDK 21 64-bit (`/usr/lib/jvm/java-21-openjdk-amd64`).
  - **Python:** `/opt/manus/python/bin/python3` (virtualenv mandiri dengan `uv`).
  - **Rust:** `rustc` dan `cargo` dikelola via `rustup` di `/usr/local/rustup`.
- **Docker:**
  - User `ubuntu` telah memiliki akses grup `docker` (dapat menjalankan `docker` tanpa `sudo`).

---

## 3. Reverse Engineering Guidelines & Directives
1. **Prioritas Tooling:** Saat diminta menganalisis, membongkar, atau membedah berkas APK (`.apk`), pustaka native (`.so`), atau DEX:
   - Gunakan **REA CLI** atau **REA MCP Tools** sebagai entry point utama untuk ekstraksi manifest, taksonomi API, inspeksi memori, dan dekompilasi.
   - Gunakan **Apktool v3.0.3** (`apktool d <file.apk>`) jika memerlukan dekompresi resource Android raw atau smali.
   - Gunakan **Ghidra Headless** untuk dekompilasi pustaka native C++/Rust yang berada di `lib/arm64-v8a/`.
2. **Integritas Workspace:**
   - Jangan menulis file temporer di luar direktori kerja proyek (`/workspace/muse-rea/`).
   - Simpan bukti atau artefak ekstraksi baru di folder `rea-output/`.
   - Pertahankan konsistensi dokumentasi di berkas `audits/` dan `CHANGELOG.md`.

---

## 4. Harnessed Project Skills (`.agents/skills/`)
Direktori `.agents/skills/` di root workspace telah dilengkapi (*harnessed*) dengan 6 Agent Skills publik kredibel:

1. **`android-reverse-engineering`** ([`SKILL.md`](file:///workspace/muse-rea/.agents/skills/android-reverse-engineering/SKILL.md) — *SimoneAvogadro/android-reverse-engineering-skill*, 8.3k+ ★):
   - Dekompilasi APK/DEX, ekstraksi pola HTTP API (Tigon/Retrofit/OkHttp), pelacakan call-flow UI ke jaringan, dan pemulihan nama simbol Kotlin yang terobfuskasi (`scripts/recover-kotlin-names.sh`, `scripts/find-api-calls.sh`).
2. **`spec-to-code-compliance`** ([`SKILL.md`](file:///workspace/muse-rea/.agents/skills/spec-to-code-compliance/SKILL.md) — *trailofbits/skills*, 7.8k+ ★):
   - Audit kepatuhan 1:1 antara dokumen spesifikasi arsitektur/protokol dan implementasi kode/biner (`resources/DIVERGENCE_RUBRIC.md`).
3. **`sharp-edges`** ([`SKILL.md`](file:///workspace/muse-rea/.agents/skills/sharp-edges/SKILL.md) — *trailofbits/skills*, 7.8k+ ★):
   - Evaluasi keamanan primitif kriptografi (`references/crypto-apis.md`), pola autentikasi token (`references/auth-patterns.md`), dan konfigurasi jaringan/proxy (`references/config-patterns.md`).
4. **`modern-python`** ([`SKILL.md`](file:///workspace/muse-rea/.agents/skills/modern-python/SKILL.md) — *trailofbits/skills*, 7.8k+ ★):
   - Standar penulisan kode Python modern, struktur SDK & CLI, serta manajemen lingkungan menggunakan `uv` dan type annotations.
5. **`property-based-testing`** ([`SKILL.md`](file:///workspace/muse-rea/.agents/skills/property-based-testing/SKILL.md) — *trailofbits/skills*, 7.8k+ ★):
   - Metodologi pengujian properti dan invariant untuk serializer biner (`#PWD_ENC:2`), parser konfigurasi (`ProxyConfig`), dan state machine (`ActivationState`).
6. **`audit-context-building`** ([`SKILL.md`](file:///workspace/muse-rea/.agents/skills/audit-context-building/SKILL.md) — *trailofbits/skills*, 7.8k+ ★):
   - Analisis mikro baris-demi-baris pada fungsi kritis sebelum melakukan rekonstruksi atau pembaruan sistem.

---

## 5. Python Architecture & Engineering Guidelines
1. **Redeem Invite Code Placement Rule:**
   - Pertahankan penempatan modul **Redeem Invite Code (`POST /hatch/invite_code/redeem`)** di bagian paling akhir seluruh dokumen, skema, SDK, dan CLI.
2. **Modern Python Standards:**
   - Gunakan format type hint modern (PEP 585, PEP 604) dan linting dengan `ruff` (`make lint`).
   - Eksekusi pengujian deterministik menggunakan `uv run pytest` atau `make test`.
3. **Workspace Hygiene:**
   - Eksekusi `make clean` untuk membersihkan cache bytecode, cache pytest, dan cache hypothesis.

---

## 6. Operational Directives & Engineering Standards
1. **Deliverables Nyata & Fungsional:**
   - Seluruh kode, skrip, dan modul harus berupa implementasi fungsional nyata (socket aktual, arsitektur utuh, penanganan error). Dilarang menyajikan pseudocode, stubs, atau komentar placeholder "TODO".
2. **Preservasi Dokumentasi & Konteks:**
   - Pertahankan integritas komentar, docstring, dan histori audit yang telah tervalidasi.
   - Perhatikan radius dampak (*blast radius*) dan sifat reversibilitas sebelum memodifikasi atau menghapus file biner dan artefak analisis.
