# Daftar Tugas (TASKS.md) - Si Ujang

Daftar tugas terstruktur untuk alur Vibe Coding. **1 Sesi = 1 Tugas Kecil**.
Tandai `[x]` jika tugas sudah diverifikasi selesai.

---

## Fase 0: Fondasi Dokumen (Perencanaan Selesai)
- [x] S01: Buat `docs/AGENTS.md` (Aturan AI hemat token & sesi)
- [x] S02: Buat `docs/TECH_STACK.md` (Laravel + Inertia Vue + Tailwind + MySQL)
- [x] S03: Buat `docs/DATABASE.md` (Skema data lengkap & audit kredit)
- [x] S04: Buat `docs/PRICING_CREDITS.md` (Tarif kredit, margin, flow WhatsApp)
- [x] S05: Buat `docs/PROGRESS.md` (File log serah-terima tiap sesi)
- [x] S06: Buat `README.md` (Ringkasan proyek & panduan setup lokal)
- [x] S06B: Buat `docs/SKILLS.md` (Keahlian teknis & instruksi spesifik AI)

---

## Fase 1: Setup Proyek & Fondasi Sistem
- [ ] S07: Setup skeleton Laravel + Inertia Vue 3 + Tailwind CSS + DaisyUI/shadcn-vue
- [ ] S08: Migrasi database (`users`, `credit_packages`, `topup_requests`, `credit_transactions`, `projects`, `project_steps`, `documents`)
- [ ] S09: Seeder awal (Role admin/guru, default `credit_packages`)
- [ ] S10: Autentikasi Google OAuth via Laravel Socialite + bonus 10 kredit akun baru
- [ ] S11: Layout antarmuka utama (Sidebar/Navbar responsif, indikator saldo kredit, menu navigasi)

---

## Fase 2: Sistem Kredit & Transaksi Manual WA
- [ ] S12: Service pemotongan kredit aman (`CreditService`: atomic lock & logging audit)
- [ ] S13: Halaman "Beli Kredit" (Daftar paket, pembuat kode invoice, tombol direct WhatsApp)
- [ ] S14: Panel Admin verifikasi top-up (`/admin/topup`: cari invoice, konfirmasi saldo, audit)
- [ ] S15: Halaman Riwayat Transaksi Kredit untuk guru

---

## Fase 3: Alur Bertahap Modul Ajar (Core MVP)
- [ ] S16: Service integrasi AI LLM (OpenRouter / Gemini API dengan format output JSON ketat)
- [ ] S17: Modul Ajar - Langkah 0: Form Identitas & Input CP guru
- [ ] S18: Modul Ajar - Langkah 1: Generate CP → TP (Tabel TP, edit/tambah/hapus, tombol setujui, potong 1 kredit)
- [ ] S19: Modul Ajar - Langkah 2: Generate TP → ATP (Alur waktu, materi pokok, asesmen, potong 1 kredit)
- [ ] S20: Modul Ajar - Langkah 3: Generate ATP → Dokumen Modul Ajar lengkap (potong 3 kredit)
- [ ] S21: Fitur lewati tahap (Mulai dari tengah: tempel TP/ATP sendiri tanpa potong kredit)

---

## Fase 4: Dokumen Tambahan & Media Interaktif
- [ ] S22: Fitur Soal + Kunci Jawaban + Rubrik Penilaian (Form parameter, generate JSON, preview & edit)
- [ ] S23: Fitur Media Ajar Interaktif (Kuis mandiri: generator satu file HTML responsif + preview iframe)

---

## Fase 5: Ekspor Dokumen & Manajemen Riwayat
- [ ] S24: Generator Ekspor `.docx` (Modul Ajar & Soal sesuai template format guru)
- [ ] S25: Generator Ekspor `.pdf` (Modul Ajar & Soal)
- [ ] S26: Generator Ekspor `.html` (Download langsung kuis interaktif mandiri)
- [ ] S27: Halaman Riwayat Proyek (Daftar proyek guru, filter jenis, lanjut edit draf, hapus, duplikasi)

---

## Fase 6: Keamanan, Polish & Peluncuran
- [ ] S28: Middleware isolasi data (Memastikan User A dilarang akses dokumen User B)
- [ ] S29: Rate limiting & proteksi request ganda (idempotency token pada generate AI)
- [ ] S30: Uji coba menyeluruh alur pengguna baru (Login -> Top-up -> Generate -> Ekspor)
