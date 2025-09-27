# context.md — SIM-Klinik (Multi-Tenant, SaaS)

> **Purpose**
> File konteks ini membantu AI (generator kode, dokumentasi, dan penguji) memahami domain, tujuan produk, batasan, istilah, serta artefak inti proyek **SIM-Klinik**. Gunakan bersama **brief.md** dan **data-model.md**.

---

## 1) Produk dalam Satu Kalimat

Platform EMR & operasional klinik multi-tenant (IGD, Rawat Jalan, **Rawat Inap**, Farmasi, Lab, Radiologi, Billing) dengan bridging **BPJS** & **SATUSEHAT (FHIR)**, dioptimalkan untuk Klinik & Puskesmas.

---

## 2) Tujuan Utama

-   Otomasi end-to-end layanan klinis & administrasi.
-   Data akurat untuk klaim BPJS, pelaporan Puskesmas (SP2TP/LPLPO), & pengambilan keputusan.
-   Standarisasi proses antar fasilitas (multi-tenant, konfigurabel).
-   Kepatuhan keamanan & privasi (UU PDP), audit & keterlacakan.

---

## 3) Persona & Kebutuhan

-   **Pimpinan**: KPI operasional/keuangan, BOR/ALOS, tren diagnosis/obat, SPM.
-   **Dokter**: EMR cepat (SOAP, ICD-10, order set), e-Resep, hasil penunjang terintegrasi.
-   **Perawat/Bidan**: triage, vital/EWS, MAR, intake-output, bedboard.
-   **Apoteker**: verifikasi terapi, dispensing FEFO, batch/ED, LPLPO.
-   **Analis Lab**: worklist, panel/subtest, nilai rujukan, TAT, validasi/riwayat koreksi.
-   **Radiografer**: jadwal modalitas, kontras, template expertise, unggah hasil.
-   **Kasir/Keu**: tagihan otomatis, split penjamin/pribadi, metode bayar (QRIS).
-   **Petugas Program**: form dinamis KIA/KB, Imunisasi, Gizi, P2P, ekspor SP2TP.
-   **Admin Sistem (Tenant)**: tenancy lokal, RBAC granular, penomoran, template dokumen, audit.
-   **Owner/Operator SaaS**: provisioning tenant, penagihan & paket langganan, pemantauan platform, SLA, keamanan & kepatuhan.

---

## 4) Lingkup Fitur (High-Level)

-   **Registrasi & Antrean**: pasien baru/eksisting, SEP BPJS (RJ/RI/IGD), nomor antrean (JKN).
-   **EMR**: SOAP/Progress/Discharge, Diagnosa ICD-10, Prosedur ICD-9-CM, Vitals/EWS, Consent.
-   **CPOE**: order layanan/obat/lab/imaging/diet/konsultasi, template order set.
-   **Rawat Inap**: admission, bedboard, perintah harian, **MAR**, diet, intake-output, discharge.
-   **Lab**: panel/subtest, referensi by usia/JK, flag kritis, TAT, verifikasi, histori koreksi.
-   **Radiologi**: modalitas, kontras, template expertise, file DICOM/JPG/PDF.
-   **Farmasi**: e-Resep/racikan, clinical check (alergi/interaksi), dispensing batch/ED, stok.
-   **Billing**: tarif multi penjamin, paket, invoice otomatis, pembayaran, jurnal, piutang.
-   **Program Puskesmas**: form dinamis (schema JSON), SP2TP, LPLPO, Posyandu.
-   **Integrasi**: BPJS VClaim/P-Care/Antrean JKN, SATUSEHAT FHIR, PACS/LIS, QRIS.
-   **Admin**: Tenancy, RBAC, COA, penomoran, template dokumen, settings, audit.

---

## 5) Glosarium Singkat

-   **Encounter**: episode kunjungan pasien (IGD/Rawat Jalan/Rawat Inap).
-   **Registration**: pendaftaran layanan yang memicu encounter.
-   **SEP**: Surat Eligibilitas Peserta (BPJS).
-   **EMR**: rekam medis elektronik (SOAP, diagnosis, prosedur, vital, consent, hasil).
-   **CPOE**: Computerized Provider Order Entry (order layanan/medikasi/penunjang).
-   **MAR**: Medication Administration Record (catatan pemberian obat).
-   **BOR/ALOS**: Bed Occupancy Rate / Average Length of Stay.
-   **SP2TP/LPLPO**: pelaporan Puskesmas & logistik farmasi program.

---

## 6) Batasan & Asumsi

-   **Arsitektur**: Multi-tenant single DB (shared schema) dengan `tenant_id`.
-   **PK**: UUID; **TZ** default `Asia/Jakarta`.
-   **Soft delete** untuk tenant tables (`deleted_at`).
-   **Keamanan**: TLS, enkripsi at-rest opsional, masking NIK, audit trail penuh.
-   **Kinerja**: job queue untuk bridging/ekspor/cetak, pagination, cache dashboard.

---

## 7) Model Data (Orientasi)

Lihat `/docs/data-model.md` (landlord/tenant).
Poin kunci:

-   Tenant tables **WAJIB**: `tenant_id`, timestamps, soft delete.
-   **Unique isolation** contoh: `unique(tenant_id, code|number|email)`.
-   **Referensi global**: `icd10_codes`, `icd9cm_codes`, (opsional `loinc_codes`, `atc_codes`).
-   Domain inti: Patients, Encounters, EMR, CPOE, Pharmacy, Lab, Imaging, Billing, Programs, Integrations, Admin.

---

## 8) Aturan Bisnis Penting

-   **Duplikasi Pasien**: cek NIK/No. BPJS/No. RM saat registrasi.
-   **Tarif per Penjamin**: fallback ke tarif umum jika spesifik tak tersedia.
-   **Resep**: wajib cek alergi & interaksi; racikan menyimpan komposisi & takaran.
-   **Lab**: nilai kritis → alert + konfirmasi diterima.
-   **Radiologi**: kontras perlu consent & checklist.
-   **RI**: 1 bed hanya 1 pasien aktif; MAR tidak boleh backdate tanpa otorisasi.
-   **Billing**: perubahan pasca bayar → nota kredit/retur (log & otorisasi).
-   **Pelaporan Puskesmas**: konsistensi stok vs pemakaian (LPLPO) & capaian (SP2TP).
-   **Audit**: semua perubahan kritis (harga, diagnosis, hasil, resume, transaksi) terekam.

---

## 9) Penomoran & Template

-   **Penomoran** (`number_sequences`): `MRN`, `REG`, `SEP`, `RX`, `ORDER`, `INV`, `GRN`, `PO`
    Pola: `INV/{YYYY}/{MM}/####`, reset: monthly.
-   **Template Dokumen** (`document_templates`): Resep, Resume Medis, Surat Sakit, Rujukan, Kwitansi, LPLPO, SP2TP.

---

## 10) Integrasi Eksternal (Ringkas)

-   **BPJS VClaim**: pembuatan/cek SEP RJ/RI/IGD, monitoring, rujukan.
-   **BPJS P-Care**: klinik primer/Puskesmas (kunjungan primer, obat program).
-   **Antrean JKN**: sinkron nomor antrean.
-   **SATUSEHAT FHIR**: Patient, Practitioner, Organization, Encounter, Condition, Procedure, Observation (vitals/lab), Medication/Request/Admin, DiagnosticReport, ImagingStudy.
-   **PACS/LIS**: adaptor sederhana; unggah file bila non-realtime.
-   **Pembayaran**: **QRIS** (static/dynamic), kartu/ewallet (ops).

---

## 11) Non-Fungsional

-   **Security**: RBAC granular, field-level privacy untuk data sensitif (HIV/kehamilan/alergi).
-   **Reliability**: backup harian, uji restore; job retry + idempotency key.
-   **Observability**: health endpoints, job dashboard, akses log, audit log, metrik TAT.
-   **Compliance**: UU PDP, kebijakan fasyankes; consent terarsip (PDF/HTML signed).

---

## 12) Konvensi Kode (Kilo Code / Generator)

-   **Blueprint**: entities, fields, relations, policies, forms, lists, filters, actions.
-   **Penamaan**: tabel plural snake_case (PK uuid), semua tenant tables wajib `tenant_id` + soft delete, detail pakai `_items`/`_lines`.
-   **UI default**: List (filter tenant, tanggal, unit, status), Form (tab Info/Tarif/Advanced).
-   **Validasi**: harga > 0, ED ≥ today, unik by `(tenant_id, …)`.

---

## 13) Acceptance Criteria (Contoh End-to-End)

1. **RJ Flow**: Registrasi → Encounter → SOAP + ICD10 → CPOE (lab/obat) → Hasil lab ke EMR → Verifikasi resep → Dispense → Invoice → Bayar QRIS → Jurnal → Audit OK.
2. **RI Flow**: IGD → Admission → Bed → CPOE harian → MAR/intake-output → Diet → Discharge summary → Final bill → Pembayaran.
3. **BPJS**: SEP RJ/RI/IGD berhasil.
4. **SATUSEHAT**: kirim Patient + Encounter + Observation (vitals) + Condition + DiagnosticReport sukses (2xx).
5. **Puskesmas**: SP2TP & LPLPO ekspor sesuai format.

---

## 14) Data Uji Minimal (Seeding)

-   1 tenant demo + 1 cabang; 3 roles (Admin, Dokter, Perawat) + 5 permissions inti.
-   20 pasien dummy; 5 poli; 2 bangsal; 6 bed; jadwal dokter.
-   10 layanan; 20 item obat (dengan batch/ED); 5 panel lab + subtest; 6 imaging catalog.
-   Tarif umum & 1 penjamin (BPJS) untuk 10 layanan.
-   Sequence aktif: MRN/REG/RX/ORDER/INV.

---

## 15) Risiko & Mitigasi

-   **Perubahan API eksternal** → adaptor ber-versi + feature flags.
-   **Data master buruk** → wizard import/validasi + audit + notifikasi.
-   **Adopsi pengguna** → order set, auto-suggest, hotkeys, training mode.

---

## 16) Peta Berkas Dokumentasi

-   `/docs/brief.md` — ringkasan proyek & target
-   `/docs/context.md` — **file ini**
-   `/docs/data-model.md` — skema landlord/tenant
-   `/docs/rbac-matrix.md` — matriks peran vs izin
-   `/docs/integration/` — BPJS/SATUSEHAT/PACS/LIS/QRIS
-   `/kilo/blueprints/` — sumber generator (entities, forms, lists, policies)
-   `/tests/` — skenario E2E

---

## 17) **Owner/Operator SaaS** (Menyewakan Aplikasi)

> Bagian ini mendeskripsikan peran & kebutuhan pemilik platform yang menyewakan SIM-Klinik sebagai layanan.

### 17.1 Tujuan & Ruang Lingkup Owner

-   Menyediakan platform **multi-tenant** yang aman, stabil, dan patuh regulasi.
-   Mengelola **provisioning** tenant, **paket langganan**, **penagihan**, **SLA**, **dukungan**, dan **kompatibilitas integrasi** (BPJS/SATUSEHAT).
-   Menjaga **ketersediaan**, **backup/DR**, **pemantauan**, dan **pembaruan** fitur.

### 17.2 Portal Landlord (Fitur Admin Owner)

-   **Tenant Management**: buat/aktif/nonaktif/suspend, domain/subdomain, logo & branding, kuota (users/branch/storage).
-   **Subscription Plans**: CRUD paket, fitur per plan (feature flags), batas kuota, harga (bulanan/tahunan), promo/kode voucher.
-   **Billing & Invoicing**: integrasi **payment gateway** (mis. QRIS/VA/kartu), auto-invoice, pajak (PPN), status pembayaran, denda keterlambatan, **prorata**.
-   **Metering & Quota**: hitung penggunaan (active users, encounters/bulan, storage, API calls). Rate-limit per plan.
-   **Monitoring**: health checks, error rate, latensi, job queue, kapasitas DB, storage; notifikasi insiden.
-   **Security Center**: audit global, kebijakan kata sandi/2FA, rotasi kunci integrasi, daftar IP allow/deny (opsional).
-   **Templates & Catalogs**: distribusi global katalog (ICD import, order set, template dokumen).
-   **Release & Feature Flags**: rilis bertahap (canary), toggle fitur per plan/tenant.
-   **Support Desk**: tiket dukungan, prioritas SLA, basis pengetahuan.
-   **Data Lifecycle**: offboarding/export data, retensi, penghapusan aman (soft/hard), legal hold.

### 17.3 Model Harga & Penagihan

-   **Model**: per-tenant (flat), per user aktif, per encounter, atau hybrid.
-   **Add-ons**: integrasi SATUSEHAT premium, PACS/LIS adaptor, extra storage, HA/DR plus.
-   **Siklus**: bulanan/tahunan; diskon komitmen tahunan.
-   **Kewajiban Pajak**: PPN & faktur pajak (bila relevan).
-   **Kebijakan Keterlambatan**: grace period, penguncian akses read-only, suspend.

### 17.4 SLA & Dukungan

-   **SLA Uptime**: contoh 99.5% (plan standard), 99.9% (enterprise).
-   **Dukungan**: Standard (jam kerja) vs Premium (24/7); kanal (email/chat/telepon).
-   **RTO/RPO**: contoh RTO ≤ 4 jam, RPO ≤ 24 jam (standard).
-   **Incident Management**: status page publik, RCA pasca insiden material.

### 17.5 Kepemilikan Data & Legal

-   **Data Ownership**: data klinik & pasien milik tenant; owner pengelola teknis.
-   **Perjanjian**: ToS, DPA (Data Processing Addendum), SLA, NDA (opsional).
-   **Privasi**: kepatuhan UU PDP; pemrosesan data lintas wilayah (jika ada).
-   **Permintaan Data**: ekspor massal saat terminasi; format terbuka (CSV/JSON/PDF).

### 17.6 Keamanan & Kepatuhan (RACI ringkas)

-   **Infra (Owner)**: patch OS/DB, firewall, enkripsi at-rest (opsional), TLS, backup/DR, monitoring.
-   **Aplikasi (Owner)**: hardening, penanganan kerentanan, SAST/DAST, audit aplikasi.
-   **Operasional (Tenant)**: manajemen user/role, praktik kata sandi, input data valid.
-   **Regulasi**: mapping klaim & pelaporan sesuai standar (owner sediakan alat, tenant bertanggung jawab isi & akurasi).

### 17.7 Onboarding & Offboarding

-   **Onboarding**: pembuatan tenant, konfigurasi profil & numbering, impor master (tarif, item farmasi, ICD subset), pelatihan dasar, aktivasi integrasi (BPJS/SATUSEHAT) dengan kredensial tenant.
-   **Offboarding**: freeze penagihan, ekspor data, penutupan akses, purge terjadwal sesuai retensi & DPA.

### 17.8 White-Label & Branding

-   Custom logo/warna, domain kustom + **SSL otomatis** (ACME), teks footer & dokumen.
-   Halaman landing tenant (opsional) & email template transactional yang dapat disesuaikan.

### 17.9 Governance & Roadmap

-   **Cadence rilis**: minor 2–4 minggu, patch keamanan as needed.
-   **Dewan Produk (opsional)**: perwakilan tenant kunci untuk prioritas backlog.
-   **Telemetry**: anon usage metrics (opt-in/opt-out) untuk peningkatan UX.

### 17.10 Acceptance Criteria (Owner Flows)

-   Buat paket → provision tenant → terbit invoice → bayar sukses → plan & fitur aktif.
-   Suspend otomatis jika melewati grace period → reaktifasi setelah pembayaran.
-   Status page & notifikasi insiden terkirim ke tenant sesuai SLA.
-   Ekspor data penuh (EMR, billing, farmasi) tersedia saat terminasi.

---
