# Stack Teknologi (TECH_STACK.md) - Si Ujang

Dokumen acuan teknologi untuk Si Ujang.

## 1. Ringkasan Stack

| Komponen | Pilihan | Alasan |
|---|---|---|
| **Backend Framework** | Laravel 11 / 10 | Ekosistem matang, auth kuat, ORM Eloquent, queuing bawaan |
| **Frontend Adapter** | Inertia.js (Vue 3) | SPA tanpa ribet bikin REST API terpisah, routing langsung dari controller |
| **Styling** | Tailwind CSS | Cepat, responsif, konsisten tanpa tulis CSS manual banyak |
| **Database** | MySQL (XAMPP / Production MySQL) | Relasional, andal untuk transaksi kredit, ACID compliant |
| **Autentikasi** | Laravel Socialite (Google OAuth) + Session | Sesuai PRD v0.2: guru login sekali klik dengan akun Google |
| **AI Integration** | Laravel HTTP Client (`Http::post()`) | Langsung ke OpenRouter / OpenAI / Gemini API tanpa library AI berat |
| **Ekspor Dokumen** | `phpoffice/phpword` (DOCX), `barryvdh/laravel-dompdf` (PDF) | Standar industri untuk dokumen perkantoran/guru |
| **Media Interaktif** | Single HTML File Template | Output mandiri (vanilla HTML+JS+CSS) bisa dibuka offline di browser/HP |
| **Pembayaran** | Manual via Admin WhatsApp | MVP tahap awal: kirim bukti transfer, admin input saldo manual |

## 2. Struktur Arsitektur

- **Monolith Modern:** Laravel + Inertia. Satu repositori, satu deployment.
- **State Management:** Vue 3 Composition API / Pinia jika perlu, shared data via Inertia Props.
- **Background Jobs:** Laravel Queue (database driver) untuk proses generate AI agar tidak timeout.

## 3. Komponen Pihak Ketiga & Library Inti

- `laravel/socialite`: Login Google.
- `phpoffice/phpword`: Generate `.docx`.
- `barryvdh/laravel-dompdf`: Generate `.pdf`.
- `openai-php/client` atau raw `Illuminate\Support\Facades\Http`: Panggilan LLM dengan retry & timeout handling.
- `@inertiajs/vue3`, `vue`, `@vitejs/plugin-vue`: Frontend bundler via Vite.
- `lucide-vue-next`: Icon pack ringan dan konsisten.

## 4. Konfigurasi Lingkungan (.env)

```env
APP_NAME="Si Ujang"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=siujang
DB_USERNAME=root
DB_PASSWORD=

# Google OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI="${APP_URL}/auth/google/callback"

# LLM API
AI_PROVIDER=openrouter # openrouter | openai | gemini
AI_API_KEY=
AI_MODEL=google/gemini-2.0-flash-001

# Admin WA
ADMIN_WHATSAPP_NUMBER=628xxxxxxxxxx
```

## 5. Batasan & Aturan Khusus

- Jangan tambah library JavaScript berat di frontend jika vanilla / utility Tailwind cukup.
- Semua pemotongan kredit dieksekusi di database transaction (`DB::transaction()`) dengan `lockForUpdate()`.
- Respon AI wajib divalidasi ke format JSON sebelum disimpan ke tabel `documents`.
