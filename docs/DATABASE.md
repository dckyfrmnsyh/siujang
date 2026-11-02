# Skema Database (DATABASE.md) - Si Ujang

Dokumen spesifikasi skema database MySQL untuk aplikasi Si Ujang (Asisten Dokumen Guru).
Mengacu pada `PRD (1).md` (v0.2).

---

## 1. Relasi Antar Tabel

```text
users (1) ──────< (N) credit_transactions
  │   │
  │   └─────────< (N) topup_requests >────── (1) credit_packages
  │
  └─────────────< (N) projects
                        │ (1)
                        ├───< (N) project_steps (riwayat CP -> TP -> ATP -> Modul)
                        │
                        └───< (N) documents (hasil akhir siap unduh / ekspor)
```

---

## 2. Rincian Tabel & Kolom

### 2.1 `users`
Menyimpan akun pengguna. Autentikasi utama menggunakan Google OAuth.

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `name` | VARCHAR(255) | No | - | Nama lengkap |
| `email` | VARCHAR(255) | No | - | Unique Index |
| `google_id` | VARCHAR(255) | Yes | NULL | Unique Index Google OAuth |
| `avatar` | VARCHAR(255) | Yes | NULL | URL foto profil dari Google |
| `role` | ENUM('guru', 'admin') | No | 'guru' | Peran akun |
| `credit_balance` | INT UNSIGNED | No | 10 | Saldo kredit aktif (bonus daftar = 10) |
| `email_verified_at` | TIMESTAMP | Yes | NULL | Waktu verifikasi email |
| `remember_token` | VARCHAR(100) | Yes | NULL | Token login session |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu registrasi |
| `updated_at` | TIMESTAMP | Yes | NULL | Waktu modifikasi profil |
| `deleted_at` | TIMESTAMP | Yes | NULL | Soft delete (kebijakan retensi akun) |

**Constraints & Indexes:**
- `PRIMARY KEY (id)`
- `UNIQUE KEY users_email_unique (email)`
- `UNIQUE KEY users_google_id_unique (google_id)`
- `INDEX users_role_index (role)`
- `CONSTRAINT chk_credit_balance_non_negative CHECK (credit_balance >= 0)`

---

### 2.2 `credit_packages`
Daftar paket pembelian kredit tetap (sesuai PRD Bagian 9 & 10).

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `name` | VARCHAR(100) | No | - | Contoh: "Paket Pemula", "Paket Semester" |
| `credits` | INT UNSIGNED | No | - | Jumlah kredit yang didapat (misal 20, 50, 120) |
| `price_idr` | INT UNSIGNED | No | - | Harga rupiah (misal 15000, 35000, 75000) |
| `is_active` | BOOLEAN | No | 1 | Status aktif/tampil di katalog top-up |
| `sort_order` | TINYINT UNSIGNED | No | 0 | Urutan tampilan di UI |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu buat |
| `updated_at` | TIMESTAMP | Yes | NULL | Waktu ubah |

---

### 2.3 `topup_requests`
Permohonan top-up manual via WhatsApp admin dengan kode unik.

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `request_code` | VARCHAR(32) | No | - | Kode unik invoice (misal: `UJG-2605-8821`), Unique Index |
| `user_id` | BIGINT UNSIGNED | No | - | FK -> `users.id` |
| `package_id` | BIGINT UNSIGNED | Yes | NULL | FK -> `credit_packages.id` (ON DELETE SET NULL) |
| `credits_amount` | INT UNSIGNED | No | - | Jumlah kredit yang dibeli |
| `price_idr` | INT UNSIGNED | No | - | Nominal transfer rupiah |
| `payment_method` | ENUM('qris', 'bank_transfer') | No | 'qris' | Metode transfer |
| `status` | ENUM('pending', 'confirmed', 'expired', 'rejected') | No | 'pending' | Status approval admin |
| `confirmed_by` | BIGINT UNSIGNED | Yes | NULL | FK -> `users.id` (Admin yang approve) |
| `confirmed_at` | TIMESTAMP | Yes | NULL | Waktu konfirmasi admin |
| `expires_at` | TIMESTAMP | No | - | Batas waktu bayar (misal +24 jam) |
| `admin_notes` | VARCHAR(255) | Yes | NULL | Alasan tolak / catatan mutasi bank |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu pengajuan |
| `updated_at` | TIMESTAMP | Yes | NULL | Waktu perubahan |

**Constraints & Indexes:**
- `PRIMARY KEY (id)`
- `UNIQUE KEY topup_requests_request_code_unique (request_code)`
- `INDEX topup_requests_user_id_index (user_id)`
- `INDEX topup_requests_status_index (status)`
- `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE`
- `FOREIGN KEY (confirmed_by) REFERENCES users(id) ON DELETE SET NULL`
- `FOREIGN KEY (package_id) REFERENCES credit_packages(id) ON DELETE SET NULL`

---

### 2.4 `credit_transactions`
Buku kas / audit log mutasi kredit. Tidak boleh dihapus sembarangan.

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `user_id` | BIGINT UNSIGNED | No | - | FK -> `users.id` |
| `type` | ENUM('signup_bonus', 'topup', 'usage', 'refund', 'admin_adjustment') | No | - | Jenis transaksi |
| `amount` | INT | No | - | Nominal mutasi (+10, -3, -1, dll) |
| `balance_after` | INT UNSIGNED | No | - | Snapshot saldo setelah transaksi |
| `description` | VARCHAR(255) | No | - | Keterangan transaksi |
| `reference_type` | VARCHAR(100) | Yes | NULL | Polimorfik (`App\Models\ProjectStep`, `App\Models\TopupRequest`) |
| `reference_id` | BIGINT UNSIGNED | Yes | NULL | ID entitas terkait |
| `created_by` | BIGINT UNSIGNED | Yes | NULL | FK -> `users.id` (bisa user atau admin) |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu transaksi terjadi |

**Constraints & Indexes:**
- `PRIMARY KEY (id)`
- `INDEX credit_transactions_user_id_index (user_id)`
- `INDEX credit_transactions_type_index (type)`
- `INDEX credit_transactions_ref_index (reference_type, reference_id)`
- `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT`

---

### 2.5 `projects`
Induk dokumen / ruang kerja guru per dokumen administrasi.

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `user_id` | BIGINT UNSIGNED | No | - | FK -> `users.id` |
| `type` | ENUM('modul_ajar', 'soal_rubrik', 'media_interaktif') | No | - | Jenis dokumen |
| `title` | VARCHAR(255) | No | - | Judul proyek (otomatis / diubah guru) |
| `subject` | VARCHAR(100) | No | - | Mata pelajaran (e.g. "Matematika", "IPAS") |
| `grade_level` | VARCHAR(50) | No | - | Jenjang / Fase (e.g. "Fase B - Kelas 4") |
| `topic` | VARCHAR(255) | Yes | NULL | Topik / Elemen materi |
| `time_allocation`| VARCHAR(100) | Yes | NULL | Alokasi waktu (e.g. "2 x 35 Menit") |
| `meetings_count` | TINYINT UNSIGNED | Yes | 1 | Jumlah pertemuan |
| `raw_input` | JSON | Yes | NULL | Input form mentah (teks CP guru, catatan khusus) |
| `current_step` | VARCHAR(30) | No | 'init' | Posisi progres ('init', 'tp', 'atp', 'modul') |
| `status` | ENUM('draft', 'in_progress', 'completed') | No | 'draft' | Status proyek |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu dibuat |
| `updated_at` | TIMESTAMP | Yes | NULL | Waktu terakhir diedit |

**Constraints & Indexes:**
- `PRIMARY KEY (id)`
- `INDEX projects_user_id_index (user_id)`
- `INDEX projects_type_index (type)`
- `INDEX projects_status_index (status)`
- `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE`

---

### 2.6 `project_steps`
Menyimpan riwayat bertahap (khusus modul ajar: Tahap 1 CP->TP, Tahap 2 TP->ATP, Tahap 3 ATP->Modul).

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `project_id` | BIGINT UNSIGNED | No | - | FK -> `projects.id` |
| `step_code` | ENUM('tp', 'atp', 'modul') | No | - | Kode tahap |
| `step_order` | TINYINT UNSIGNED | No | 1 | Urutan tahap (1, 2, 3) |
| `input_context` | JSON | Yes | NULL | Konteks data yang dikirim ke prompt AI |
| `original_ai_output` | JSON | Yes | NULL | Output mentah asli dari AI (acuan pembanding) |
| `edited_output` | JSON | Yes | NULL | Output setelah diedit & disetujui guru |
| `credits_used` | TINYINT UNSIGNED | No | 0 | Jumlah kredit yang dipotong untuk step ini |
| `status` | ENUM('pending', 'processing', 'draft', 'approved', 'failed') | No | 'pending' | Status approval step |
| `error_message` | TEXT | Yes | NULL | Log error bila generate gagal |
| `approved_at` | TIMESTAMP | Yes | NULL | Waktu guru menekan "Setujui & Lanjut" |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu step dibuat |
| `updated_at` | TIMESTAMP | Yes | NULL | Waktu modifikasi step |

**Constraints & Indexes:**
- `PRIMARY KEY (id)`
- `INDEX project_steps_project_id_index (project_id)`
- `INDEX project_steps_step_code_index (step_code)`
- `FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE`

---

### 2.7 `documents`
Hasil akhir dokumen yang siap unduh / ekspor (`.docx`, `.pdf`, atau file `.html` interaktif mandiri).

| Kolom | Tipe Data | Null | Default | Keterangan |
|---|---|---|---|---|
| `id` | BIGINT UNSIGNED | No | AUTO_INCREMENT | Primary Key |
| `project_id` | BIGINT UNSIGNED | No | - | FK -> `projects.id` |
| `user_id` | BIGINT UNSIGNED | No | - | FK -> `users.id` |
| `title` | VARCHAR(255) | No | - | Nama file/dokumen |
| `type` | ENUM('modul_ajar', 'soal_rubrik', 'media_interaktif') | No | - | Jenis dokumen |
| `content_json` | JSON | No | - | Data terstruktur final untuk render |
| `html_output` | LONGTEXT | Yes | NULL | File HTML mandiri (khusus media interaktif) |
| `docx_path` | VARCHAR(255) | Yes | NULL | Path cache file .docx siap unduh |
| `pdf_path` | VARCHAR(255) | Yes | NULL | Path cache file .pdf siap unduh |
| `download_count`| INT UNSIGNED | No | 0 | Statistik unduh |
| `created_at` | TIMESTAMP | Yes | NULL | Waktu generate final |
| `updated_at` | TIMESTAMP | Yes | NULL | Waktu update |

**Constraints & Indexes:**
- `PRIMARY KEY (id)`
- `INDEX documents_project_id_index (project_id)`
- `INDEX documents_user_id_index (user_id)`
- `FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE`
- `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE`

---

## 3. Logika Transaksi & Keamanan Saldo

### 3.1 Pemotongan Kredit Aman (Anti Race-Condition)
```php
DB::transaction(function () use ($user, $cost, $projectStep) {
    // Kunci baris user agar tidak double-spend
    $lockedUser = User::where('id', $user->id)->lockForUpdate()->first();

    if ($lockedUser->credit_balance < $cost) {
        throw new InsufficientCreditsException("Saldo kredit tidak cukup.");
    }

    $lockedUser->decrement('credit_balance', $cost);

    CreditTransaction::create([
        'user_id' => $lockedUser->id,
        'type' => 'usage',
        'amount' => -$cost,
        'balance_after' => $lockedUser->credit_balance,
        'description' => "Generate tahap {$projectStep->step_code}",
        'reference_type' => ProjectStep::class,
        'reference_id' => $projectStep->id,
        'created_by' => $lockedUser->id,
    ]);
});
```

### 3.2 Idempotensi Konfirmasi Top-Up Admin
```php
DB::transaction(function () use ($requestCode, $adminUser) {
    $topup = TopupRequest::where('request_code', $requestCode)->lockForUpdate()->firstOrFail();

    if ($topup->status !== 'pending') {
        throw new \Exception("Permintaan sudah diproses atau kedaluwarsa.");
    }

    $topup->update([
        'status' => 'confirmed',
        'confirmed_by' => $adminUser->id,
        'confirmed_at' => now(),
    ]);

    $user = User::where('id', $topup->user_id)->lockForUpdate()->firstOrFail();
    $user->increment('credit_balance', $topup->credits_amount);

    CreditTransaction::create([
        'user_id' => $user->id,
        'type' => 'topup',
        'amount' => $topup->credits_amount,
        'balance_after' => $user->credit_balance,
        'description' => "Top-up manual paket {$topup->package_name} ({$topup->request_code})",
        'reference_type' => TopupRequest::class,
        'reference_id' => $topup->id,
        'created_by' => $adminUser->id,
    ]);
});
```
