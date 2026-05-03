# SECURITY WHITEPAPER — BISNAPI

**Versi:** 1.0 | **Diterbitkan:** 1 Mei 2026
**Dokumen ini bersifat informatif, bukan kontraktual.**

---

## Pendahuluan

Dokumen ini menjelaskan arsitektur keamanan, postur perlindungan data, dan praktik operasional yang diterapkan oleh PT Snappy Angkasa Media dalam menjalankan platform Bisnapi.

Bisnapi dibangun di atas filosofi **Clean Infrastructure**: tidak ada server yang dikelola sendiri, tidak ada database shared antar tenant, tidak ada vendor AI pihak ketiga yang menyentuh data Pengguna. Setiap keputusan arsitektur dibuat dengan mempertimbangkan keamanan dan isolasi data sejak awal (_security by design_).

Dokumen ini ditujukan untuk tim teknis, tim keamanan, dan pengambil keputusan di organisasi Pengguna yang ingin memahami bagaimana data mereka dilindungi.

---

## 1. Infrastruktur Inti

### 1.1. Platform

Bisnapi berjalan sepenuhnya di atas **Cloudflare** — tidak ada server fisik atau virtual machine yang dikelola oleh PT Snappy Angkasa Media. Komponen utama:

| Komponen   | Teknologi                 | Fungsi                          |
| ---------- | ------------------------- | ------------------------------- |
| Compute    | Cloudflare Workers        | Business logic, API, routing    |
| Database   | Cloudflare D1 (SQLite)    | Data operasional per-tenant     |
| Storage    | Cloudflare R2             | File, dokumen, attachment       |
| AI Engine  | Cloudflare Workers AI     | BISNAY AI — inference on-edge   |
| CDN & Edge | Cloudflare Global Network | Delivery, SSL termination       |
| Cache      | Cloudflare KV             | Session, config, tenant routing |
| DNS & DDoS | Cloudflare                | Proteksi jaringan dan resolusi  |

### 1.2. Keunggulan Arsitektur Edge

Bisnapi tidak memiliki server origin tradisional. Seluruh request diproses di **edge node Cloudflare terdekat** dengan lokasi Pengguna — biasanya dalam radius kurang dari 100ms dari pengguna di Indonesia.

Implikasi keamanan:

- Tidak ada IP server yang dapat dijadikan target serangan langsung.
- Cloudflare Anycast routing otomatis menyerap serangan DDoS sebelum mencapai aplikasi.
- Tidak ada "single point of failure" berbasis server.

---

## 2. Isolasi Data Multi-Tenant

### 2.1. Model Isolasi

Bisnapi menggunakan model **fisik per-tenant** — bukan shared database dengan row-level security.

Setiap Organisasi Pengguna mendapatkan:

- **1 dedicated Cloudflare D1 database** — `org_{id}` — tidak ada organisasi lain yang dapat mengakses database ini.
- **1 dedicated Cloudflare R2 bucket** — untuk file dan dokumen organisasi.

```
Tenant A → D1: org_a1b2c3 (isolated)
Tenant B → D1: org_d4e5f6 (isolated)
Tenant C → D1: org_g7h8i9 (isolated)

Tidak ada shared DB. Zero cross-tenant access.
```

### 2.2. Mengapa Ini Penting

Model shared database dengan row-level security — yang umum digunakan kompetitor — memiliki risiko inherent:

- Misconfiguration RLS → kebocoran data antar tenant.
- Query optimization yang salah → data bocor ke tenant lain.
- Schema migration yang gagal → data corruption lintas tenant.

Dengan isolasi fisik, risiko-risiko di atas **secara arsitektural tidak mungkin terjadi**.

### 2.3. Tenant Routing

Routing dari request ke database yang tepat dikelola oleh **Cloudflare KV** yang menyimpan mapping `orgId → d1DatabaseId`. Key KV dilindungi dengan namespace isolation dan hanya dapat diakses oleh Workers yang memiliki binding yang relevan.

---

## 3. Enkripsi dan Keamanan Transport

### 3.1. Data In Transit

- Seluruh koneksi menggunakan **TLS 1.3** minimum.
- TLS 1.0 dan 1.1 dinonaktifkan secara global.
- Certificate management dikelola oleh Cloudflare (auto-renewal).
- **HTTP Strict Transport Security (HSTS)** diaktifkan dengan `max-age=31536000; includeSubDomains; preload`.
- **HSTS Preload List:** `bisnapi.id` sedang dalam proses pengajuan ke browser preload list ([hstspreload.org](https://hstspreload.org)). Setelah aktif, seluruh browser utama (Chrome, Firefox, Safari, Edge) akan memaksa HTTPS pada first-visit — mengeliminasi risiko SSL stripping.

### 3.2. DNS Security

**Certificate Authority Authorization (CAA):**
`bisnapi.id` dikonfigurasi dengan CAA records — hanya CA yang terdaftar secara eksplisit yang diizinkan menerbitkan sertifikat TLS untuk domain ini:

| Tag         | CA yang Diizinkan                                                        |
| ----------- | ------------------------------------------------------------------------ |
| `issue`     | `letsencrypt.org`, `digicert.com`, `pki.goog`, `comodoca.com`, `ssl.com` |
| `issuewild` | `letsencrypt.org`, `digicert.com`, `pki.goog`, `comodoca.com`, `ssl.com` |

CA yang tidak terdaftar di atas tidak dapat menerbitkan sertifikat untuk `bisnapi.id` — terlepas dari permintaan siapapun.
Dapat diverifikasi via: `dig CAA bisnapi.id @1.1.1.1 +short`

**DNSSEC:**
`bisnapi.id` telah aktif DNSSEC — diverifikasi dengan flag `ad` (Authenticated Data) dari resolver `1.1.1.1`. Seluruh respons DNS ditandatangani menggunakan **ECDSA P-256 (Algorithm 13)** dan dapat divalidasi secara kriptografis oleh resolver yang mendukung DNSSEC.

```
dig bisnapi.id A +dnssec @1.1.1.1 | grep "flags:"
# Output: flags: qr rd ra ad ← 'ad' = DNSSEC tervalidasi penuh
```

Mengeliminasi risiko DNS spoofing dan cache poisoning secara kriptografis.

### 3.3. Email Security

Email yang dikirim dari `@bisnapi.id` dilindungi dengan tiga lapisan standar industri:

| Mekanisme   | Status                        | Fungsi                                                                                                                                                                          |
| ----------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SPF**     | Aktif                         | Menentukan server mana yang berwenang mengirim email atas nama `bisnapi.id`.                                                                                                    |
| **DKIM**    | Aktif (`cf2024-1._domainkey`) | Setiap email ditandatangani secara kriptografis.                                                                                                                                |
| **DMARC**   | Aktif (`p=reject`, strict)    | Email yang gagal SPF/DKIM alignment otomatis ditolak. Laporan ke `abuse@bisnapi.id`.                                                                                            |
| **MTA-STS** | Aktif (`mode: enforce`)       | Server email pengirim **wajib** menggunakan TLS saat mengirim ke `@bisnapi.id`. Email tanpa TLS ditolak. File tersedia di `https://mta-sts.bisnapi.id/.well-known/mta-sts.txt`. |
| **BIMI**    | Aktif                         | Brand logo Bisnapi ditampilkan di inbox penerima (Gmail, Apple Mail) pada email yang terverifikasi. SVG tersedia di `https://bisnapi.id/assets/bimi-logo.svg`.                  |
| **TLS-RPT** | Aktif                         | Laporan kegagalan TLS dikirim ke `abuse@bisnapi.id`. Pair dengan MTA-STS untuk monitoring enforcement.                                                                          |

Email routing dikelola oleh **Cloudflare Email Routing** (`route1/2/3.mx.cloudflare.net`) — tidak ada mail server yang dikelola sendiri.

### 3.3. Data At Rest

- Data di **Cloudflare D1** dienkripsi at rest menggunakan AES-256 oleh Cloudflare secara default.
- Data di **Cloudflare R2** dienkripsi at rest menggunakan AES-256 oleh Cloudflare secara default.
- Enkripsi dikelola di level infrastruktur Cloudflare — PT Snappy Angkasa Media tidak mengelola encryption key secara manual.

### 3.4. Data Sensitif

Data sensitif (token, session, API key) di-hash menggunakan algoritma bcrypt atau Argon2 sebelum disimpan. Nilai plaintext tidak pernah tersimpan di database.

---

## 4. Keamanan AI (BISNAY AI)

### 4.1. Cloudflare Workers AI — Isolated Native Inference

BISNAY AI berjalan di atas **Cloudflare Workers AI**. Arsitektur ini memastikan:

- **On-Edge Inference**: Proses inferensi model AI terjadi sepenuhnya di infrastruktur global Cloudflare. Data Pengguna tidak pernah dikirimkan ke API penyedia AI eksternal (seperti OpenAI, Anthropic, atau Groq) untuk diproses.
- **Data Residency**: Seluruh prompt, metadata, dan output AI tetap berada di dalam ekosistem Cloudflare, selaras dengan postur data residency Snappy.
- **Model Transparency**: Snappy menggunakan model-model _open-weights_ ternama (seperti Meta Llama, Mistral, atau SEA-LION) yang di-host secara native oleh Cloudflare. Meskipun bobot model berasal dari pihak ketiga, eksekusinya terisolasi di dalam lingkungan komputasi Cloudflare.

### 4.2. Kebijakan Anti-Training & Privasi

Sesuai dengan kebijakan privasi infrastruktur kami:

- **No Training**: Cloudflare tidak menggunakan Konten Pelanggan (prompt atau output) untuk melatih model AI apapun (baik milik Cloudflare maupun pihak ketiga) tanpa persetujuan eksplisit dari Snappy.
- **On-Net Inference**: Seluruh pemrosesan AI dilakukan di dalam infrastruktur Cloudflare. Routing inferensi mengikuti lokasi GPU yang tersedia di jaringan global Cloudflare untuk memastikan ketersediaan layanan.
- **Privacy Enforcement**: Postur ini mengeliminasi kebutuhan untuk Data Processing Agreement (DPA) tambahan dengan vendor AI pihak ketiga, karena seluruh pemrosesan tercakup dalam DPA Cloudflare.

---

## 5. Autentikasi dan Otorisasi

### 5.1. Autentikasi Passwordless (Zero Password Architecture)

Bisnapi tidak menyimpan password. Login menggunakan **Magic Link** yang dikirim ke email terdaftar — link hanya berlaku satu kali dan kedaluwarsa dalam hitungan menit.

Implikasi keamanan:

- Tidak ada password hash di database yang dapat di-brute force.
- Tidak ada risiko credential stuffing dari breach di platform lain.
- Tidak ada risiko password reuse antar layanan.

Verifikasi tambahan:

- **Session token** berbasis JWT dengan expiry pendek — dirotasi setiap sesi baru.
- **WhatsApp OTP** untuk verifikasi nomor telepon (Gate 3) dan aksi sensitif.
- Session disimpan di Cloudflare KV dengan TTL yang dikonfigurasi ketat.

### 5.2. Role-Based Access Control (RBAC)

Setiap akun mendukung multi-user dengan role yang dapat dikonfigurasi oleh Admin:

- **Owner** — akses penuh termasuk billing dan pengaturan organisasi.
- **Admin** — manajemen user dan konfigurasi modul.
- **Staff** — akses operasional sesuai modul yang ditetapkan.
- Role kustom dapat dikonfigurasi pada Tier MADJOE ke atas.

### 5.3. API Security

- Seluruh endpoint API memerlukan autentikasi via Bearer token.
- Rate limiting diterapkan per-endpoint dan per-tenant.
- API key dapat di-revoke kapan saja dari dashboard.
- Webhook signature verification menggunakan HMAC-SHA256.

---

## 6. Keamanan Operasional

### 6.1. Zero Standing Privilege — Arsitektur Tanpa Server

Bisnapi berjalan di atas Cloudflare Workers — model **serverless edge compute**. Tidak ada server fisik atau virtual machine yang persisten.

Implikasi keamanan:

- Tidak ada SSH access ke server produksi — tidak ada server yang bisa di-SSH.
- Tidak ada persistent process yang dapat dieksploitasi.
- Setiap request adalah execution context baru yang terisolasi — tidak ada shared memory antar request.
- Attack surface berkurang drastis dibanding arsitektur server tradisional.

### 6.2. Prinsip Least Privilege

Staf PT Snappy Angkasa Media hanya mendapatkan akses ke sistem yang relevan dengan peran masing-masing:

- Developer tidak memiliki akses langsung ke database produksi Pengguna tanpa proses approval.
- Seluruh akses ke sistem produksi dicatat dalam audit log.
- Multi-factor authentication wajib untuk seluruh akses ke sistem internal.

### 6.3. Audit Log

- Seluruh akses ke data Pengguna oleh staf Bisnapi dicatat dan dapat diaudit.
- Log aktivitas Pengguna di dalam platform disimpan selama **90 hari**.
- Log keamanan terkait insiden disimpan selama penyelesaian investigasi.

### 6.4. Provisioning Rollback Protocol

Saat Organisasi baru dibuat, sistem provisioning edge menjalankan pembuatan D1 database dan konfigurasi KV secara atomik. Jika salah satu langkah gagal, rollback otomatis dijalankan:

```
1. D1 creation gagal → tidak ada aksi lanjut
2. KV mapping gagal → D1 yang sudah dibuat dihapus
3. Registry gagal → D1 + KV dihapus, Org diflag FAILED_PROVISIONING
```

Tidak ada "limbo resource" yang dapat menjadi attack surface atau menyebabkan data leakage antar tenant.

### 6.5. Pemisahan Lingkungan

- Lingkungan **production**, **staging**, dan **development** sepenuhnya terpisah.
- Data Pengguna tidak pernah digunakan di lingkungan staging atau development.
- Akses ke production memerlukan approval eksplisit dari lead engineering.

---

## 7. Perlindungan DDoS dan Ketersediaan

7.1. Cloudflare secara otomatis memfilter traffic berbahaya sebelum mencapai aplikasi, termasuk:

- **Layer 3/4 DDoS mitigation** — absorpsi serangan volumetrik.
- **Layer 7 WAF (Web Application Firewall)** — proteksi terhadap OWASP Top 10.
- **Bot Management** — deteksi dan pemblokiran bot berbahaya.

  7.2. Cloudflare memiliki kapasitas jaringan global yang mampu menyerap serangan DDoS berskala terabit per detik.

  7.3. Tidak ada IP origin server Bisnapi yang terpublikasi — seluruh traffic melalui Cloudflare proxy.

---

## 8. Security Headers — Verified

`app.bisnapi.id` diuji menggunakan **Mozilla HTTP Observatory** dan secara konsisten mendapatkan **A+ Rating** (Skor Tertinggi).

| Header / Test                       | Status | Keterangan                                                                  |
| ----------------------------------- | ------ | --------------------------------------------------------------------------- |
| Content Security Policy (CSP)       | Pass   | `default-src 'none'` — implementasi CSP paling strict. Zero unsafe sources. |
| Cross Origin Resource Policy (CORP) | Pass   | Mencegah data leak ke cross-origin context                                  |
| Referrer Policy                     | Pass   | `strict-origin-when-cross-origin`                                           |
| Strict Transport Security (HSTS)    | Pass   | `max-age=31536000`; preload sedang diajukan                                 |
| Subresource Integrity (SRI)         | Pass   | Seluruh script eksternal dilindungi hash integrity                          |
| X-Content-Type-Options              | Pass   | `nosniff` — mencegah MIME-type sniffing attack                              |
| X-Frame-Options                     | Pass   | Implementasi via `CSP frame-ancestors`                                      |
| CORS                                | Pass   | Konten tidak visible via cross-origin tanpa otorisasi                       |
| Redirection                         | Pass   | HTTP → HTTPS redirect aktif                                                 |

> Hasil scan dapat diverifikasi secara publik di [developer.mozilla.org/en-US/observatory](https://developer.mozilla.org/en-US/observatory).

---

## 9. Manajemen Kerentanan

### 9.1. Pembaruan Dependency

- Dependency library diperbarui secara berkala dan dipantau menggunakan automated tooling.
- Security patch kritis diterapkan dalam **24 jam** sejak dipublikasikan.

### 9.2. Vulnerability Disclosure

Bisnapi memiliki program Vulnerability Disclosure Policy (VDP) yang terbuka untuk komunitas keamanan. Detail prosedur pelaporan tersedia di `docs/legal/bisnapi/id/modules/VDP.md`.

### 9.3. Security Contact (RFC 9116)

Bisnapi mengimplementasikan standar `security.txt` (RFC 9116) — file terstandarisasi yang memberitahu peneliti keamanan cara melaporkan temuan:

```
https://bisnapi.id/.well-known/security.txt
```

File ini mencantumkan: contact email, kebijakan disclosure, dan bahasa yang didukung.
Dapat diverifikasi via: `curl https://bisnapi.id/.well-known/security.txt`

### 9.4. Certificate Transparency (CT Logs)

Seluruh sertifikat TLS yang diterbitkan untuk `bisnapi.id` secara otomatis dicatat dalam **Certificate Transparency Logs** — sistem log publik yang dapat diaudit oleh siapapun. Ini memastikan tidak ada sertifikat yang diterbitkan secara diam-diam untuk domain Bisnapi.

Sertifikat yang saat ini aktif:

| Issuer                          | Common Name                  | Valid Hingga |
| ------------------------------- | ---------------------------- | ------------ |
| Google Trust Services (WE1)     | `bisnapi.id`                 | 22 Jul 2026  |
| Cloudflare TLS Issuing ECC CA 4 | `*.bisnapi.id`, `bisnapi.id` | Aktif        |

Dapat diverifikasi via: [crt.sh/?q=bisnapi.id](https://crt.sh/?q=bisnapi.id)

---

## 10. Kepatuhan dan Regulasi

| Regulasi / Standar          | Status        | Catatan                                                              |
| --------------------------- | ------------- | -------------------------------------------------------------------- |
| UU PDP No. 27/2022          | **Aligned**   | Langkah kepatuhan diimplementasikan (DPA & Retensi)                  |
| UU ITE No. 11/2008          | Compliant     | PSE terdaftar                                                        |
| PP No. 71/2019 (PP PSTE)    | Compliant     | Sistem elektronik terdaftar                                          |
| PSE Registration            | Aktif         | No. 022966.01/DJAI.PSE/04/2026                                       |
| PCI DSS (pembayaran)        | (via Xendit)  | Bisnapi tidak menyimpan data kartu. Xendit PCI DSS Level 1 certified |
| D1 Jurisdiction Constraints | **Roadmap**   | Mendukung isolasi data di yurisdiksi tertentu (saat ini EU, FedRAMP) |
| Certificate Transparency    | Aktif         | Semua cert terpublikasi di CT logs — verifiable via crt.sh           |
| security.txt (RFC 9116)     | Aktif         | Tersedia di `/.well-known/security.txt`                              |
| Cloudflare SOC 2 Type II    | **Inherited** | Infrastruktur Cloudflare bersertifikat (via Cloudflare)              |
| Cloudflare ISO 27001        | **Inherited** | Infrastruktur Cloudflare bersertifikat (via Cloudflare)              |
| Mozilla HTTP Observatory    | **A+ Rating** | Verified Public — Best Practice Security Headers                     |

---

## 11. Backup dan Pemulihan

11.1. **D1 Time Travel**: Cloudflare menyediakan fitur _Time Travel_ secara native yang memungkinkan pemulihan data (_point-in-time recovery_) ke setiap menit dalam **30 hari** terakhir secara atomik.

11.2. **R2 Durability**: Penyimpanan objek di Cloudflare R2 dirancang untuk durabilitas tinggi (99.999999999% / 11 nines). Snappy mengimplementasikan strategi retensi jangka panjang melalui _object versioning_ dan _lifecycle policies_.

11.3. **Recovery Point Objective (RPO)**: Maksimum 24 jam untuk retensi jangka panjang, atau hitungan menit menggunakan _Time Travel_ untuk pemulihan operasional.

11.4. **Recovery Time Objective (RTO)**: Target pemulihan layanan dalam **4 jam** untuk insiden kritis.

11.5. Prosedur disaster recovery diuji secara berkala oleh tim engineering Bisnapi.

---

## 12. Incident Response

Jika terjadi insiden keamanan:

| Fase                | Tindakan                                        | Waktu Target              |
| ------------------- | ----------------------------------------------- | ------------------------- |
| Deteksi             | Monitoring otomatis + alert                     | Real-time                 |
| Triase              | Klasifikasi severity dan dampak                 | < 30 menit                |
| Containment         | Isolasi sistem terdampak                        | < 1 jam                   |
| Notifikasi Pengguna | Untuk insiden yang berdampak pada data Pengguna | < 72 jam                  |
| Remediasi           | Perbaikan dan patch                             | Sesuai severity           |
| Post-mortem         | Laporan analisis dan pencegahan                 | < 7 hari setelah resolusi |

Laporan insiden yang dipublikasikan tersedia di `status.bisnapi.id`.

---

## 13. Kontak Keamanan

| Keperluan                      | Kontak            |
| ------------------------------ | ----------------- |
| Laporan kerentanan             | abuse@bisnapi.id  |
| Pertanyaan keamanan enterprise | legal@bisnapi.id  |
| Status insiden                 | status.bisnapi.id |

---

> **Catatan:** Dokumen ini bersifat informatif dan tidak menimbulkan kewajiban kontraktual tambahan di luar yang telah diatur dalam dokumen legal Bisnapi (TOS, DPA, MSA). Komitmen kontraktual terkait data residency dan keamanan tercantum dalam `DPA.md`.

---

_© 2026 PT Snappy Angkasa Media._
_PSE: 022966.01/DJAI.PSE/04/2026_
_Versi dokumen ini dapat diperbarui tanpa pemberitahuan — selalu lihat versi terbaru di `docs.bisnapi.id`._
