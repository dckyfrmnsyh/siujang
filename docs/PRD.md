# PRD: Asisten Dokumen Guru

> Versi: 0.1 (draf) · Status: perlu ditinjau pemilik produk
> Dokumen ini menjawab **apa** dan **mengapa**. Pilihan teknologi ada di `TECH_STACK.md`, skema data di `DATABASE.md`.

## 1. Ringkasan

Aplikasi web yang membantu guru membuat dokumen administrasi dan media pembelajaran (modul ajar, soal, rubrik, hingga media ajar interaktif) dengan bantuan AI. Guru mengisi form singkat, AI menghasilkan dokumen terstruktur, guru meninjau, mengedit, lalu mengunduh.

Model bisnis: **kredit prabayar** (top-up), bukan langganan unlimited, supaya biaya AI tidak pernah melebihi pemasukan.

## 2. Masalah

- Guru menghabiskan banyak waktu menyusun modul ajar, soal, dan rubrik yang formatnya berulang.
- Hasil dari AI umum (chat biasa) tidak terstruktur sesuai format dokumen guru, sehingga perlu diformat ulang manual.
- Membuat media ajar interaktif (kuis, simulasi) butuh keterampilan teknis yang tidak dimiliki kebanyakan guru.

## 3. Target pengguna

| Pengguna | Kebutuhan utama |
|---|---|
| **Guru SD/SMP/SMA/SMK** (utama) | Dokumen cepat, sesuai format, mudah diedit, harga terjangkau |
| **Guru honorer / pengajar privat** | Harga per pemakaian, tanpa komitmen bulanan |
| **Sekolah / KKG / MGMP** (tahap lanjut) | Kuota bersama, invoice |

Asumsi: pengguna lebih sering memakai ponsel; sebagian besar tidak familier dengan istilah teknis AI.

## 4. Tujuan dan metrik keberhasilan

| Tujuan | Metrik | Target awal (perlu dikalibrasi) |
|---|---|---|
| Dokumen berguna tanpa banyak revisi | Persentase dokumen diunduh setelah generate | ≥ 60% |
| Guru kembali memakai | Pengguna aktif yang generate ≥ 2 kali dalam 30 hari | ≥ 30% |
| Bisnis sehat | Margin kotor per generate (setelah biaya AI dan gateway) | ≥ 60% |
| Pembayaran lancar | Top-up berhasil / top-up dimulai | ≥ 80% |

## 5. Lingkup MVP

### Termasuk

- [ ] Registrasi dan login (email; verifikasi untuk mencegah akun massal)
- [ ] Generate **Modul Ajar**
- [ ] Generate **Soal + Kunci Jawaban + Rubrik Penilaian**
- [ ] Generate **Media Ajar Interaktif** (satu jenis dulu, misalnya kuis interaktif)
- [ ] Edit hasil sebelum ekspor
- [ ] Ekspor `.docx` dan `.pdf` (media interaktif diekspor sebagai file HTML)
- [ ] Riwayat dokumen
- [ ] Sistem **kredit**: saldo, pemotongan per generate, riwayat pemakaian
- [ ] Top-up kredit (QRIS dan e-wallet diprioritaskan)
- [ ] Batas keamanan: rate limit, batas harian akun gratis

### Sengaja TIDAK dibuat dulu (di luar lingkup MVP)

- Aplikasi mobile native
- Kolaborasi multi-guru / manajemen kelas
- Langganan bulanan dan paket sekolah (menyusul setelah ada data pemakaian)
- Integrasi ke sistem sekolah (Dapodik, e-rapor, dll.)
- Penyimpanan data siswa
- Fitur berbagi publik / marketplace dokumen
- Pembuatan gambar oleh AI

## 6. Alur utama pengguna

1. Guru daftar dan mendapat sedikit **kredit gratis** (sekali).
2. Guru memilih jenis dokumen.
3. Guru mengisi form (lihat bagian 7).
4. Sistem menampilkan **estimasi kredit** sebelum generate.
5. Guru menekan Generate; kredit dipotong; hasil tampil.
6. Guru mengedit atau meminta revisi (jumlah revisi gratis terbatas, selebihnya memakai kredit).
7. Guru mengunduh dokumen.
8. Kredit habis: guru diarahkan ke halaman top-up.

## 7. Spesifikasi fitur

### 7.1 Modul Ajar

**Input:** jenjang, kelas/fase, mata pelajaran, topik/materi, alokasi waktu, jumlah pertemuan, (opsional) catatan khusus dan capaian pembelajaran yang ingin dipakai.

**Output (struktur tetap, diisi AI):**
- Identitas modul
- Capaian Pembelajaran, Tujuan Pembelajaran
- Profil pelajar Pancasila / dimensi yang relevan (mengikuti kurikulum yang berlaku)
- Kegiatan pembelajaran (pembuka, inti, penutup)
- Asesmen (diagnostik, formatif, sumatif)
- Media dan sumber belajar
- Refleksi guru dan siswa

**Aturan:** format dokumen berasal dari **template tetap**; AI hanya mengisi konten (output JSON terstruktur). Nama atau data siswa tidak diminta.

### 7.2 Soal + Kunci + Rubrik

**Input:** mata pelajaran, topik, jenjang, bentuk soal (pilihan ganda / uraian / campuran), jumlah soal, tingkat kesulitan, (opsional) level kognitif.

**Output:** daftar soal, kunci jawaban, pembahasan singkat, rubrik penilaian, kisi-kisi sederhana.

### 7.3 Media Ajar Interaktif

**Input:** topik, jenjang, jenis media (MVP: kuis interaktif), jumlah item, gaya tampilan sederhana.

**Output:** satu file HTML mandiri yang bisa dibuka di browser/ponsel tanpa instalasi, dengan skor dan umpan balik.

**Catatan:** fitur ini paling mahal di sisi token, jadi memakai harga kredit tertinggi dan batas ukuran output.

## 8. Model kredit dan harga (angka awal, wajib dikalibrasi)

| Item | Kredit (awal) |
|---|---|
| Modul ajar | 3 |
| Soal + kunci + rubrik | 2 |
| Media ajar interaktif | 8 |
| Revisi (setelah jatah gratis) | 1 |

Aturan:
- Saldo dipotong saat generate dimulai; jika generate **gagal karena kesalahan sistem**, kredit dikembalikan otomatis.
- Kredit gratis awal hanya sekali per akun terverifikasi.
- Tidak ada paket unlimited.
- Harga per kredit ditentukan di `PRICING_CREDITS.md` dengan rumus: `(biaya AI + biaya gateway + pajak) ÷ (1 − margin target)`.
- Nominal top-up minimum ditetapkan agar biaya tetap metode pembayaran tidak menggerus margin.

## 9. Pembayaran (tingkat kebutuhan)

- Metode prioritas: QRIS dan e-wallet; virtual account sebagai pilihan dengan minimum top-up lebih tinggi.
- Kredit **bertambah hanya setelah konfirmasi server dari penyedia pembayaran** (bukan dari browser).
- Setiap transaksi hanya boleh menambah kredit **sekali** (tidak ada kredit ganda).
- Riwayat transaksi dapat dilihat pengguna.
- Detail teknis: `PAYMENT_FLOW.md`.

## 10. Kebutuhan non-fungsional

| Aspek | Kebutuhan |
|---|---|
| Kinerja | Modul ajar tampil dalam kisaran 30–60 detik; ada indikator progres |
| Perangkat | Responsif, nyaman dipakai di ponsel |
| Keamanan | Kunci API AI hanya di server; rate limit; validasi input |
| Biaya | Batas pengeluaran harian pada penyedia AI; batas token per permintaan |
| Privasi | Data pribadi seminimal mungkin; tidak mengirim data identitas siswa ke AI; kebijakan privasi tersedia sebelum rilis |
| Keandalan | Generate gagal tidak boleh memotong kredit; log pemakaian disimpan |
| Bahasa | Antarmuka dan output dalam Bahasa Indonesia |

## 11. Risiko dan mitigasi

| Risiko | Mitigasi |
|---|---|
| Biaya AI melebihi pemasukan | Kredit prabayar, harga per fitur sesuai bobot token, batas output, anggaran harian |
| Kualitas hasil AI tidak konsisten | Template tetap, output JSON terstruktur, uji dengan data nyata sebelum rilis |
| Penyalahgunaan kredit gratis | Verifikasi email/nomor HP, batas per perangkat/IP |
| Kesalahan saldo / webhook ganda | Idempotency, tes otomatis untuk logika kredit |
| Ketidaksesuaian dengan kurikulum terbaru | Rujukan kurikulum dipasang di prompt dan ditinjau berkala; disclaimer bahwa guru wajib meninjau |
| Pelanggaran hak cipta materi | Larangan menyalin materi buku secara utuh; syarat layanan jelas |

## 12. Pertanyaan terbuka (isi sebelum lanjut)

- [ ] Nama aplikasi?
- [ ] Kurikulum acuan yang dipakai pada MVP (Kurikulum Merdeka atau yang berlaku saat rilis)?
- [ ] Jenjang awal: semua, atau mulai dari SD/SMP/SMA tertentu?
- [ ] Jenis media interaktif pertama selain kuis?
- [ ] Badan usaha atau status penjual untuk verifikasi gateway?
- [ ] Jumlah kredit gratis awal?
- [ ] Target jumlah guru untuk beta tertutup?

## 12b. Kriteria "MVP selesai"

- [ ] Guru baru dapat daftar, generate satu modul ajar, dan mengunduhnya tanpa bantuan.
- [ ] Top-up lewat QRIS/e-wallet menambah kredit tepat sekali.
- [ ] Generate yang gagal tidak memotong kredit.
- [ ] Margin per generate terbukti positif dari data uji nyata.
- [ ] 20–50 guru beta menggunakan aplikasi dan memberi umpan balik.

## 13. Dokumen terkait

`TECH_STACK.md` · `ARCHITECTURE.md` · `DATABASE.md` · `PRICING_CREDITS.md` · `PAYMENT_FLOW.md` · `PROMPTS.md` · `TASKS.md` · `PROGRESS.md`
