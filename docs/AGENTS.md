# Aturan AI (AGENTS.md) - Si Ujang

Dokumen ini aturan kerja AI. Wajib patuh setiap sesi agar hemat token dan tidak merusak kode.

## 1. Siklus Tiap Sesi

1. **Awal Sesi:**
   - Wajib baca `PROGRESS.md` dan `TASKS.md`.
   - Ambil SATU tugas kecil dari `TASKS.md` berstatus pending.
   - Ajukan rencana singkat (<5 baris). Tunggu persetujuan user sebelum tulis kode.

2. **Eksekusi:**
   - Ubah file seminimal mungkin (diff parsial, bukan tulis ulang satu file utuh).
   - Dilarang tambah dependensi baru jika fitur bawaan/stdlib cukup.
   - Jangan buat file dokumentasi/markdown baru kecuali diminta eksplisit.

3. **Akhir Sesi:**
   - Jalankan uji/linter dasar.
   - Update `PROGRESS.md` (status, selesai apa, blocker, langkah berikutnya).
   - Centang tugas di `TASKS.md`.

## 2. Aturan Hemat Token & Error Handling

- **Aturan 2 Kali Gagal:** Jika perbaikan bug gagal 2 kali, STOP. Catat error dan apa yang dicoba di `PROGRESS.md`. Minta user mulai sesi baru.
- **Tanpa Konteks Berulang:** Jangan minta user paste ulang kode. Cukup rujuk path file (`src/xxx.ts`).
- **Respon Singkat:** Tanpa basa-basi, langsung rencana, kode, atau status.

## 3. Keamanan & Logika Kritis

- **Secret:** API key, token, dan kredensial HANYA di `.env`. Dilarang commit atau tampilkan di log/chat.
- **Logika Kredit & Auth:** Wajib ada validasi input batas aman dan tes sederhana/assert (cegah saldo minus / bypass token).
- **Idempotensi:** Operasi potong saldo atau top-up harus aman dari request ganda.

## 4. Konvensi Kode & Lingkungan

- **Proyek:** Si Ujang (Asisten Dokumen Guru - Modul Ajar, Soal/Rubrik, Media Interaktif).
- **Dokumen Acuan:**
  - Konsep & Fitur: `docs/PRD (1).md` (v0.2)
  - Stack Teknologi: `docs/TECH_STACK.md`
  - Skema Data: `docs/DATABASE.md`
  - Pelacakan Tugas: `TASKS.md` & `PROGRESS.md`
