# Si Ujang - Asisten Dokumen Guru

Aplikasi web cerdas untuk membantu guru menyusun administrasi pembelajaran terstruktur (Modul Ajar bertahap, Soal + Kunci + Rubrik, dan Media Ajar Interaktif) menggunakan model AI prabayar berbasis kredit.

---

## 🚀 Fitur Utama MVP

1. **Modul Ajar Runtut Bertahap:**
   - Tahap 1: Capaian Pembelajaran (CP) → Tujuan Pembelajaran (TP)
   - Tahap 2: Tujuan Pembelajaran (TP) → Alur Tujuan Pembelajaran (ATP)
   - Tahap 3: Alur Tujuan Pembelajaran (ATP) → Modul Ajar Lengkap
   - Fleksibel: Guru dapat melewati tahap jika sudah memiliki bahan sendiri.
2. **Paket Asesmen Otomatis:**
   - Pembuat Soal (Pilihan Ganda / Uraian) + Kunci Jawaban + Pembahasan + Rubrik Penilaian & Kisi-kisi.
3. **Media Ajar Interaktif:**
   - Generator kuis interaktif siap pakai dalam 1 file HTML mandiri (bisa dibuka offline di ponsel tanpa instalasi).
4. **Sistem Kredit Prabayar & Top-up WhatsApp:**
   - Transparan, tanpa tagihan langganan otomatis.
   - Bonus 10 kredit akun baru.
   - Top-up praktis transfer / QRIS via konfirmasi admin WhatsApp.
5. **Ekspor & Riwayat:**
   - Unduh dokumen format `.docx`, `.pdf`, atau `.html`.
   - Riwayat tersimpan per akun Google; unduh ulang dan edit manual gratis.

---

## 🛠️ Stack Teknologi

- **Backend:** Laravel 11 / 10 (PHP 8.2+)
- **Frontend:** Inertia.js + Vue 3
- **Styling:** Tailwind CSS
- **Database:** MySQL
- **Autentikasi:** Laravel Socialite (Google OAuth)
- **AI Engine:** OpenRouter / Google Gemini 2.0 Flash / OpenAI API via HTTP Client
- **Dokumen Generator:** `phpoffice/phpword` & `barryvdh/laravel-dompdf`

---

## 📋 Struktur Dokumentasi (`/docs`)

Sebelum melakukan pengembangan atau vibe coding bersama AI, baca dokumen terkait di folder `docs/`:

| Dokumen | Isi & Panduan |
|---|---|
| [`docs/AGENTS.md`](docs/AGENTS.md) | **Aturan kerja AI**: alur per sesi, hemat token, batas 2x gagal |
| [`docs/PRD (1).md`](docs/PRD%20(1).md) | Product Requirement Document (v0.2) lengkap |
| [`docs/TECH_STACK.md`](docs/TECH_STACK.md) | Pilihan pustaka, arsitektur, dan konfigurasi `.env` |
| [`docs/DATABASE.md`](docs/DATABASE.md) | Skema tabel MySQL, relasi, audit saldo, anti double-spend |
| [`docs/PRICING_CREDITS.md`](docs/PRICING_CREDITS.md) | Penetapan harga kredit per fitur, paket top-up, estimasi margin |
| [`docs/TASKS.md`](docs/TASKS.md) | Daftar checklist tugas terpecah per sesi (S01 - S30) |
| [`docs/PROGRESS.md`](docs/PROGRESS.md) | Log serah-terima status pekerjaan terkini antar sesi |

---

## 💻 Panduan Instalasi Lokal

### Prasyarat
- PHP >= 8.2 (ekstensi: `pdo_mysql`, `curl`, `mbstring`, `fileinfo`, `gd`, `zip`)
- Composer
- Node.js (v18+) & NPM
- MySQL Database (misal via XAMPP)

### Langkah Setup
1. Clone repositori dan buka folder proyek:
   ```bash
   cd "D:\WEB\Si Ujang"
   ```
2. Salin environment file:
   ```bash
   cp .env.example .env
   ```
3. Konfigurasikan koneksi database dan kredensial API di `.env`:
   - `DB_DATABASE=siujang`
   - `GOOGLE_CLIENT_ID` & `GOOGLE_CLIENT_SECRET`
   - `AI_API_KEY`
4. Jalankan migrasi dan seeder awal:
   ```bash
   php artisan migrate --seed
   ```
5. Install dependensi frontend dan backend:
   ```bash
   composer install
   npm install
   ```
6. Jalankan server lokal:
   ```bash
   # Terminal 1 (Vite HMR)
   npm run dev

   # Terminal 2 (Laravel Server)
   php artisan serve
   ```
7. Buka browser di `http://127.0.0.1:8000`.

---

## 📜 Lisensi
Proprietary - Hak Cipta dilindungi undang-undang.
