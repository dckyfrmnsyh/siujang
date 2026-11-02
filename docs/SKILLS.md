# Kumpulan Keahlian AI (SKILLS.md) - Si Ujang

Katalog aturan eksekusi teknis untuk AI agent saat menulis kode proyek Si Ujang.

---

## 1. Skill: Laravel 11 + Inertia Vue 3
**Pemicu:** Setiap pembuatan controller, route, dan halaman Vue.
**Aturan:**
- Controller tipis: hanya validasi request, delegasi ke Service/Action, return `Inertia::render()`.
- Hindari REST API terpisah jika rute internal aplikasi. Gunakan Inertia props bawaan.
- Vue 3 pakai `<script setup>` standar, tanpa Options API.
- Form pakai `useForm()` dari `@inertiajs/vue3` untuk handle submit, loading state, dan error bag otomatis.
- Mobile-first Tailwind: komponen wajib muat di viewport 360px (layar HP guru).

---

## 2. Skill: Prompting Kurikulum & Validasi JSON
**Pemicu:** Integrasi AI LLM untuk CP, TP, ATP, Modul Ajar, Soal, Media.
**Aturan:**
- Prompt wajib memaksa output format JSON mentah tanpa markdown wrap (tanpa \`\`\`json).
- Format prompt terstruktur:
  1. Identitas (Mapel, Fase/Kelas, Topik).
  2. Input Guru (CP/TP/ATP).
  3. Skema Output JSON (kunci, tipe data, batas panjang).
  4. Instruksi pedagogis (taksonomi Bloom revisi, bahasa operasional, kontekstual).
- Validator: parsing JSON wajib dibungkus `try-catch`. Bila JSON rusak, lempar exception untuk auto-refund kredit.

---

## 3. Skill: Atomic Credits & Anti Race-Condition
**Pemicu:** Logika tambah/potong kredit, top-up manual, auto-refund.
**Aturan:**
- Semua mutasi saldo wajib di dalam `DB::transaction()`.
- Wajib pakai `User::where('id', $id)->lockForUpdate()->first()`.
- Cek saldo sebelum potong. Jika kurang: throw `InsufficientCreditsException`.
- Mutasi berhasil: wajib buat baris baru di `credit_transactions` sebagai audit log.
- Generate AI gagal di tengah jalan: wajib trigger fungsi `refundCredit()` dengan tipe `refund`.

---

## 4. Skill: Generator Dokumen (DOCX, PDF, HTML)
**Pemicu:** Ekspor hasil akhir ke file unduhan.
**Aturan:**
- DOCX: pakai template PHPWord sederhana, font standar (Arial / Times New Roman ukuran 11-12pt), tabel bergaris rapi.
- PDF: generate via Dompdf dari view Blade minimalis, hindari flexbox kompleks CSS3 yang sering rusak di Dompdf.
- Media Interaktif: kuis wajib single-file HTML (inline CSS + inline JS), zero-dependency, auto-score, responsif HP.
