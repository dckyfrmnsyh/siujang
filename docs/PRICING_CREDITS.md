# Model Harga & Kredit (PRICING_CREDITS.md) - Si Ujang

Dokumen acuan penetapan tarif kredit, paket top-up, estimasi margin, dan alur pembayaran manual via WhatsApp. Mengacu pada `PRD (1).md` (v0.2).

---

## 1. Filosofi & Aturan Dasar Kredit

1. **Prabayar Penuh:** Tidak ada langganan otomatis / tagihan berulang. Guru hanya bayar apa yang dipakai.
2. **Kredit Gratis Awal:** 10 kredit saat pertama kali login dengan Google.
3. **Download & Edit Manual Gratis:** Guru bebas mengedit, melihat riwayat, dan mengunduh ulang dokumen tanpa potong kredit.
4. **Garansi Generate Gagal:** Jika pemanggilan AI gagal karena sistem atau timeout API, kredit **wajib dikembalikan otomatis** (auto-refund).
5. **Mulai dari Tengah Hemat Kredit:** Guru yang menempel TP/ATP sendiri tanpa generate lewat AI tidak dikenai biaya untuk tahap yang dilewati.

---

## 2. Tarif Konsumsi Kredit per Fitur

| Fitur / Tahap | Kredit | Estimasi Token LLM | Keterangan |
|---|:---:|:---:|---|
| **Bonus Registrasi Baru** | **+10** | - | 1x per akun Google |
| **Modul Ajar - Tahap 1 (CP → TP)** | 1 | ~1.5k token | Terjemahkan CP ke daftar TP terukur |
| **Modul Ajar - Tahap 2 (TP → ATP)** | 1 | ~2.5k token | Susun alur waktu, materi pokok, asesmen |
| **Modul Ajar - Tahap 3 (ATP → Modul)** | 3 | ~5k - 8k token | RPP/Modul lengkap per pertemuan |
| **Soal + Kunci + Rubrik** | 2 | ~3k - 5k token | Paket soal lengkap kisi-kisi & rubrik |
| **Media Interaktif (Kuis HTML)** | 8 | ~8k - 12k token | Kode HTML + JS + CSS interaktif mandiri |
| **Revisi oleh AI** | 1 | ~2k - 4k token | Perintah revisi instruksional ke AI |
| **Edit Manual Guru** | **0** | 0 token | Bebas edit tanpa AI di aplikasi |
| **Unduh Ulang (DOCX / PDF / HTML)** | **0** | 0 token | Dari riwayat tersimpan |

---

## 3. Paket Pembelian Kredit (Top-Up)

Paket dibuat dalam nominal tetap untuk memudahkan verifikasi admin pada mutasi rekening/QRIS.

| Nama Paket | Jumlah Kredit | Harga (IDR) | Harga / Kredit | Cocok Untuk |
|---|:---:|:---:|:---:|---|
| **Paket Pemula** | 20 | Rp 15.000 | Rp 750 | Uji coba 3–4 modul lengkap |
| **Paket Hemat** | 50 | Rp 30.000 | Rp 600 | Kebutuhan 1 bulan ajar |
| **Paket Guru Aktif** *(Rekomendasi)* | 120 | Rp 60.000 | Rp 500 | Kebutuhan 1 semester (lengkap soal & kuis) |
| **Paket Komplit** | 300 | Rp 120.000 | Rp 400 | Persiapan 1 tahun ajaran penuh |

---

## 4. Analisis Biaya & Target Margin

### 4.1 Asumsi Biaya AI (OpenRouter / Google Gemini 2.0 Flash)
- Biaya per 1M input token: ~$0.10 (± Rp 1.600)
- Biaya per 1M output token: ~$0.40 (± Rp 6.400)
- Rata-rata 1 modul ajar lengkap (tahap 1–3 = 5 kredit):
  - Total token: ~12.000 token
  - Biaya LLM aktual: ± Rp 70 - Rp 100 per modul.
- Pendapatan dari 5 kredit (pada tarif termurah Rp 400/kredit) = **Rp 2.000**.
- **Margin Kotor:**
  $$\text{Margin} = \frac{\text{Rp } 2.000 - \text{Rp } 100}{\text{Rp } 2.000} \times 100\% \approx 95\%$$
- Sangat aman di atas target PRD (minimal 60%), bahkan setelah memperhitungkan biaya server, operasional admin WhatsApp, dan potensi revisi.

---

## 5. Alur Transaksi Manual WhatsApp

### 5.1 Langkah Pengguna
1. Pengguna membuka modal/halaman **Beli Kredit**.
2. Memilih salah satu paket (misal: "Paket Guru Aktif - 120 Kredit - Rp 60.000").
3. Memilih metode pembayaran: **QRIS** atau **Transfer Bank**.
4. Sistem membuat baris di tabel `topup_requests` dan menghasilkan **Kode Unik Invoice** (format: `UJG-YYMM-XXXX`, contoh: `UJG-2605-A782`). Masa berlaku: 24 jam.
5. Tombol **"Bayar & Konfirmasi via WhatsApp"** terbuka otomatis mengarahkan ke link WhatsApp Web/App:
   ```text
   https://wa.me/628xxxxxxxxxx?text=Halo%20Admin%20Si%20Ujang,%20saya%20ingin%20konfirmasi%20top-up%20kredit:%0A%0A- Kode Invoice: UJG-2605-A782%0A- Akun: guru.budi@gmail.com%0A- Paket: Paket Guru Aktif (120 Kredit)%0A- Total: Rp 60.000%0A%0ABerikut saya lampirkan bukti pembayarannya. Mohon diproses ya.
   ```

### 5.2 Langkah Admin
1. Admin menerima pesan WhatsApp dan memeriksa mutasi rekening / dashboard QRIS (bukan hanya percaya screenshot transfer).
2. Admin membuka Panel Admin Si Ujang (`/admin/topup`).
3. Cari `Kode Invoice` terkait.
4. Tekan tombol **"Konfirmasi & Tambah Kredit"**.
5. Sistem mengeksekusi DB Transaction:
   - Status invoice berubah menjadi `confirmed`.
   - `credit_balance` pengguna bertambah.
   - Tercatat di `credit_transactions` sebagai audit log.
6. Notifikasi saldo bertambah otomatis muncul di dashboard pengguna saat refresh / navigasi.
