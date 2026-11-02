# PRD: Asisten Dokumen Guru

> Versi: 0.2 (draf) · Status: perlu ditinjau pemilik produk
> Dokumen ini menjawab **apa** dan **mengapa**. Pilihan teknologi ada di `TECH_STACK.md`, skema data di `DATABASE.md`.

## Riwayat revisi

| Versi | Perubahan |
|---|---|
| 0.2 | Alur modul ajar bertahap (CP → TP → ATP → Modul); pembayaran manual via admin WhatsApp; login Google; riwayat dan hasil tersimpan per akun; panel admin minimal |
| 0.1 | Draf awal |

## 1. Ringkasan

Aplikasi web yang membantu guru membuat dokumen administrasi dan media pembelajaran (modul ajar, soal, rubrik, hingga media ajar interaktif) dengan bantuan AI. Guru login dengan Google, mengisi input, meninjau hasil di setiap tahap, lalu mengunduh. Semua hasil tersimpan di riwayat akun masing-masing.

Model bisnis: **kredit prabayar**. Pada tahap awal, **top-up dilakukan manual lewat admin WhatsApp**; gateway otomatis ditunda sampai ada cukup pengguna.

## 2. Masalah

- Guru menghabiskan banyak waktu menyusun modul ajar, soal, dan rubrik yang formatnya berulang.
- Penyusunan modul ajar yang benar membutuhkan urutan berpikir (CP → TP → ATP → modul), sementara AI umum biasanya langsung menulis modul tanpa runtutan itu.
- Hasil dari AI umum tidak terstruktur sesuai format dokumen guru, sehingga perlu diformat ulang.
- Membuat media ajar interaktif membutuhkan keterampilan teknis yang tidak dimiliki kebanyakan guru.

## 3. Target pengguna

| Pengguna | Kebutuhan utama |
|---|---|
| **Guru SD/SMP/SMA/SMK** (utama) | Dokumen cepat, runtut sesuai kurikulum, mudah diedit, harga terjangkau |
| **Guru honorer / pengajar privat** | Harga per pemakaian, tanpa komitmen bulanan |
| **Admin (pemilik produk)** | Memproses top-up kredit dan memantau pemakaian dengan mudah |
| **Sekolah / KKG / MGMP** (tahap lanjut) | Kuota bersama, invoice |

Asumsi: pengguna lebih sering memakai ponsel; punya akun Google; terbiasa memakai WhatsApp.

## 4. Tujuan dan metrik keberhasilan

| Tujuan | Metrik | Target awal (perlu dikalibrasi) |
|---|---|---|
| Dokumen berguna tanpa banyak revisi | Persentase modul ajar yang diunduh setelah generate | ≥ 60% |
| Guru kembali memakai | Pengguna aktif yang membuat dokumen ≥ 2 kali dalam 30 hari | ≥ 30% |
| Bisnis sehat | Margin kotor per dokumen (setelah biaya AI) | ≥ 60% |
| Top-up manual lancar | Waktu median konfirmasi top-up oleh admin | ≤ 1 jam pada jam kerja |
| Alur bertahap dipakai | Persentase proyek modul ajar yang sampai tahap modul | ≥ 50% |

## 5. Lingkup MVP

### Termasuk

- [ ] **Login dengan Google** (satu-satunya metode login)
- [ ] **Modul Ajar bertahap**: CP → daftar TP → ATP → Modul (lihat 7.1)
- [ ] Generate **Soal + Kunci Jawaban + Rubrik Penilaian**
- [ ] Generate **Media Ajar Interaktif** (satu jenis dulu, misalnya kuis interaktif)
- [ ] Edit hasil di setiap tahap sebelum lanjut atau ekspor
- [ ] **Riwayat dan hasil tersimpan per akun** (lihat 8)
- [ ] Ekspor `.docx` dan `.pdf` (media interaktif sebagai file HTML)
- [ ] Sistem **kredit**: saldo, pemotongan per tahap/generate, riwayat kredit
- [ ] **Top-up manual via WhatsApp admin** (lihat 10)
- [ ] **Panel admin minimal**: proses top-up, tambah/koreksi kredit, lihat pemakaian
- [ ] Batas keamanan: rate limit, batas harian akun gratis

### Sengaja TIDAK dibuat dulu

- Payment gateway otomatis (QRIS/e-wallet/VA otomatis)
- Login email/password atau metode login lain
- Basis data CP resmi per mapel dan fase (MVP: guru menempel CP sendiri)
- Aplikasi mobile native
- Kolaborasi multi-guru / manajemen kelas
- Langganan bulanan dan paket sekolah
- Integrasi ke sistem sekolah (Dapodik, e-rapor, dll.)
- Penyimpanan data siswa
- Berbagi publik / marketplace dokumen
- Pembuatan gambar oleh AI

## 6. Alur utama pengguna

1. Guru menekan **Masuk dengan Google**; akun baru mendapat **kredit gratis** sekali.
2. Guru memilih jenis dokumen (modul ajar, soal, media interaktif).
3. Guru mengisi input; sistem menampilkan **estimasi kredit** sebelum proses.
4. Hasil tampil, **tersimpan otomatis di riwayat**, dan dapat diedit.
5. Untuk modul ajar, guru melewati beberapa tahap (7.1) dan menyetujui hasil tiap tahap.
6. Guru mengunduh dokumen kapan saja, termasuk dari riwayat.
7. Kredit habis: guru membuka halaman **Beli Kredit**, memilih paket, dan dihubungkan ke admin lewat WhatsApp (bagian 10).

## 7. Spesifikasi fitur

### 7.1 Modul Ajar (alur bertahap)

Modul ajar **tidak boleh dibuat langsung**. Prosesnya berurutan, dan setiap tahap menjadi dasar tahap berikutnya.

```
CP  ──►  daftar TP  ──►  ATP  ──►  Modul Ajar
 (input)  (AI + edit)   (AI + edit)  (AI + edit)
```

**Satu proyek modul ajar** = satu rangkaian tahap di bawah ini, tersimpan sebagai satu entri riwayat.

| Tahap | Input | Proses | Output | Aksi guru |
|---|---|---|---|---|
| **0. Identitas** | Jenjang, fase/kelas, mapel, topik/elemen, alokasi waktu, jumlah pertemuan | Disimpan sebagai konteks proyek | Identitas proyek | Isi form |
| **1. CP → TP** | Teks Capaian Pembelajaran (ditempel guru) | AI menerjemahkan CP menjadi **daftar Tujuan Pembelajaran** (rumusan operasional) | Daftar TP | Edit, hapus, tambah, urutkan, **setujui** |
| **2. TP → ATP** | Daftar TP yang disetujui | AI menyusun **Alur Tujuan Pembelajaran**: urutan TP, alokasi waktu/JP, materi pokok, asesmen ringkas | Tabel ATP | Edit, **setujui** |
| **3. ATP → Modul** | ATP yang disetujui + pilihan TP/pertemuan yang ingin dibuatkan modul | AI menyusun modul ajar lengkap | Dokumen modul | Edit, unduh |

**Aturan alur:**
- Tahap berikutnya hanya aktif setelah tahap sebelumnya **disetujui** guru.
- Guru boleh **mulai dari tengah** bila sudah punya bahan: menempel TP sendiri (lewati tahap 1) atau menempel ATP sendiri (lewati tahap 1–2). Tahap yang dilewati tidak memotong kredit.
- Tahap 3 memakai TP dan ATP yang **sudah tersimpan** sebagai konteks, bukan menghasilkannya ulang (menghemat token).
- Mengubah tahap sebelumnya setelah tahap berikutnya dibuat memberi peringatan bahwa hasil di bawahnya mungkin tidak sinkron, dan guru dapat menjalankan ulang tahap tersebut.
- Satu ATP dapat dipakai untuk membuat beberapa modul (misalnya satu modul per pertemuan/TP) tanpa mengulang tahap 1–2.

**Struktur output modul ajar (template tetap, AI mengisi konten):**
- Identitas modul
- Capaian Pembelajaran dan Tujuan Pembelajaran (dari tahap 1–2)
- Dimensi/profil lulusan yang relevan (mengikuti kurikulum yang berlaku)
- Kegiatan pembelajaran (pembuka, inti, penutup)
- Asesmen (diagnostik, formatif, sumatif)
- Media dan sumber belajar
- Refleksi guru dan siswa

**Aturan umum:** format dokumen berasal dari template tetap; AI hanya mengisi konten dalam output JSON terstruktur. Nama atau data siswa tidak diminta.

### 7.2 Soal + Kunci + Rubrik

**Input:** mata pelajaran, topik, jenjang, bentuk soal (pilihan ganda / uraian / campuran), jumlah soal, tingkat kesulitan, (opsional) TP dari riwayat proyek modul ajar sebagai acuan.

**Output:** daftar soal, kunci jawaban, pembahasan singkat, rubrik penilaian, kisi-kisi sederhana.

### 7.3 Media Ajar Interaktif

**Input:** topik, jenjang, jenis media (MVP: kuis interaktif), jumlah item, gaya tampilan sederhana; (opsional) mengambil TP atau materi dari riwayat.

**Output:** satu file HTML mandiri yang bisa dibuka di browser/ponsel tanpa instalasi, dengan skor dan umpan balik.

**Catatan:** paling mahal di sisi token, jadi memakai harga kredit tertinggi dan batas ukuran output.

## 8. Riwayat dan penyimpanan hasil (per akun)

**Yang tersimpan untuk setiap proyek/dokumen:**
- Jenis dokumen, judul otomatis (dapat diubah), tanggal dibuat dan diubah
- Input guru (identitas, CP yang ditempel, catatan)
- **Hasil setiap tahap** (TP, ATP, modul) beserta status (draf / disetujui)
- Hasil terbaru setelah diedit guru (versi asli dari AI tetap disimpan sebagai pembanding)
- Kredit yang terpakai per tahap

**Kemampuan pengguna:**
- [ ] Melihat daftar riwayat (urut terbaru, pencarian berdasarkan judul/mapel)
- [ ] Membuka kembali dan **melanjutkan** proyek yang belum selesai
- [ ] Mengunduh ulang tanpa memotong kredit
- [ ] **Menduplikasi** proyek sebagai dasar proyek baru
- [ ] Menghapus proyek (hapus permanen oleh pengguna)
- [ ] Melihat **riwayat kredit** (top-up, pemakaian, pengembalian)

**Aturan data:**
- Data tiap akun **terisolasi**: pengguna hanya bisa melihat miliknya sendiri; admin hanya mengakses data pemakaian seperlunya.
- Hasil yang tersimpan tidak hangus karena saldo kredit habis.
- Kebijakan retensi (berapa lama data disimpan bila akun tidak aktif) ditentukan sebelum rilis dan dicantumkan di kebijakan privasi.

## 9. Model kredit (angka awal, wajib dikalibrasi)

| Item | Kredit (awal) |
|---|---|
| Modul ajar: tahap 1 (CP → TP) | 1 |
| Modul ajar: tahap 2 (TP → ATP) | 1 |
| Modul ajar: tahap 3 (ATP → Modul), per modul | 3 |
| Soal + kunci + rubrik | 2 |
| Media ajar interaktif | 8 |
| Revisi oleh AI (setelah jatah gratis) | 1 |

Aturan:
- Saldo dipotong saat sebuah tahap/generate dimulai; jika gagal karena kesalahan sistem, kredit **dikembalikan otomatis**.
- Mengedit manual dan mengunduh dari riwayat **tidak** memakai kredit.
- Kredit gratis awal hanya sekali per akun Google.
- Tidak ada paket unlimited.
- Harga per kredit ditentukan di `PRICING_CREDITS.md` dengan rumus `(biaya AI + pajak) ÷ (1 − margin target)`. Pada tahap manual belum ada biaya gateway, tetapi tetap sisihkan biaya waktu admin dan potensi selisih transfer.
- Paket top-up berupa pilihan tetap (misalnya beberapa nominal), bukan jumlah bebas, agar admin mudah memverifikasi.

## 10. Pembayaran: manual via admin WhatsApp

**Alur:**
1. Pengguna membuka **Beli Kredit** dan memilih paket.
2. Sistem membuat **permintaan top-up** dengan **kode unik** (status: menunggu) dan mencatat akun, paket, nominal.
3. Tombol **"Hubungi Admin via WhatsApp"** membuka WhatsApp dengan pesan terisi otomatis (kode permintaan, paket, nominal, email akun).
4. Admin memberikan instruksi pembayaran (transfer/QRIS statis) dan pengguna membayar.
5. Admin memverifikasi pembayaran **pada mutasi rekening/e-wallet** (bukan hanya dari tangkapan layar).
6. Di **panel admin**, admin memasukkan/memilih kode permintaan lalu menekan **Konfirmasi** → kredit bertambah.
7. Pengguna melihat saldo dan riwayat kredit bertambah.

**Aturan:**
- Satu kode permintaan hanya bisa dikonfirmasi **sekali** (tidak ada kredit ganda).
- Setiap penambahan/koreksi kredit oleh admin tercatat dalam **log audit** (siapa, kapan, berapa, alasan).
- Permintaan yang tidak dibayar dalam batas waktu tertentu berstatus **kedaluwarsa**.
- Hanya akun dengan peran **admin** yang dapat membuka panel admin (ditentukan oleh daftar email Google admin).
- Pengguna dapat melihat status permintaan top-up miliknya (menunggu / dikonfirmasi / kedaluwarsa / ditolak).
- Pembayaran otomatis lewat gateway menjadi **fase berikutnya**, ketika volume top-up sudah merepotkan admin. Detail di `PAYMENT_FLOW.md`.

## 11. Login dan akun

- Login hanya melalui **Google (OAuth)**; tidak ada kata sandi yang dikelola aplikasi.
- Identitas akun = akun Google (ID unik), bukan hanya email.
- Data profil yang disimpan seminimal mungkin (nama, email, foto opsional).
- Peran: `pengguna` dan `admin`.
- Logout dan penghapusan akun (beserta riwayatnya) disediakan; kebijakan detail di kebijakan privasi.

## 12. Kebutuhan non-fungsional

| Aspek | Kebutuhan |
|---|---|
| Kinerja | Tiap tahap modul ajar tampil dalam kisaran 15–60 detik; ada indikator progres |
| Perangkat | Responsif, nyaman dipakai di ponsel |
| Keamanan | Kunci API AI hanya di server; rate limit; validasi input; isolasi data antar akun; akses admin dibatasi |
| Biaya | Batas pengeluaran harian pada penyedia AI; batas token per permintaan |
| Privasi | Data pribadi seminimal mungkin; tidak mengirim data identitas siswa ke AI; kebijakan privasi tersedia sebelum rilis |
| Keandalan | Generate gagal tidak memotong kredit; hasil tahap tersimpan sebelum ditampilkan; log pemakaian disimpan |
| Bahasa | Antarmuka dan output dalam Bahasa Indonesia |

## 13. Risiko dan mitigasi

| Risiko | Mitigasi |
|---|---|
| Biaya AI melebihi pemasukan | Kredit prabayar, harga per tahap sesuai bobot token, batas output, anggaran harian |
| Alur bertahap terasa panjang bagi guru | Boleh mulai dari tengah (TP/ATP sendiri), simpan otomatis, tombol lanjut yang jelas |
| Hasil CP → TP tidak konsisten | Template prompt tetap, contoh few-shot, guru wajib menyetujui sebelum lanjut |
| Admin kewalahan memproses top-up | Paket tetap, kode unik, pesan WhatsApp otomatis, target waktu konfirmasi, rencana gateway otomatis |
| Bukti transfer palsu | Verifikasi lewat mutasi, bukan tangkapan layar |
| Salah input kredit oleh admin | Log audit, konfirmasi dua langkah, kemampuan koreksi tercatat |
| Penyalahgunaan kredit gratis dengan banyak akun Google | Batas per perangkat/IP, kredit gratis kecil, pantau pola |
| Kebocoran data antar akun | Isolasi data di level database, tes akses |
| Ketidaksesuaian dengan kurikulum terbaru | Rujukan kurikulum di prompt ditinjau berkala; disclaimer bahwa guru wajib meninjau |
| Pelanggaran hak cipta materi | Larangan menyalin materi buku secara utuh; syarat layanan jelas |

## 14. Pertanyaan terbuka (isi sebelum lanjut)

- [ ] Nama aplikasi?
- [ ] Kurikulum acuan MVP dan istilah resminya (CP, TP, ATP, profil/dimensi lulusan) yang dipakai di template?
- [ ] Format kolom ATP yang diinginkan (misalnya: urutan, TP, materi, alokasi JP, asesmen)?
- [ ] Apakah guru cukup menempel teks CP, atau perlu pilihan CP resmi per mapel/fase sejak MVP?
- [ ] Nomor WhatsApp admin, metode pembayaran yang dilayani (rekening, QRIS statis), dan jam layanan admin?
- [ ] Daftar paket top-up (nominal dan jumlah kredit)?
- [ ] Jumlah kredit gratis awal dan batas harian akun gratis?
- [ ] Kebijakan retensi riwayat dan penghapusan akun?
- [ ] Jenjang awal: semua atau satu dulu?
- [ ] Jenis media interaktif pertama selain kuis?
- [ ] Jumlah guru untuk beta tertutup?

## 15. Kriteria "MVP selesai"

- [ ] Guru baru dapat masuk dengan Google, menempel CP, melewati tahap CP → TP → ATP → Modul, dan mengunduh modul tanpa bantuan.
- [ ] Guru dapat menutup aplikasi, kembali, lalu melanjutkan proyek dari riwayat tanpa kehilangan hasil.
- [ ] Akun A tidak dapat melihat data akun B.
- [ ] Alur top-up via WhatsApp berjalan: permintaan → konfirmasi admin → kredit bertambah tepat sekali, dengan log audit.
- [ ] Tahap yang gagal tidak memotong kredit.
- [ ] Margin per modul ajar lengkap (tahap 1–3) terbukti positif dari data uji nyata.
- [ ] 20–50 guru beta menggunakan aplikasi dan memberi umpan balik.

## 16. Dokumen terkait

`TECH_STACK.md` · `ARCHITECTURE.md` · `DATABASE.md` · `PRICING_CREDITS.md` · `PAYMENT_FLOW.md` · `PROMPTS.md` · `TASKS.md` · `PROGRESS.md`
