# Aplikasi Pengelolaan Keuangan UMKM

**v1.0** — UAS Pemrograman Web | Kelompok 5

Aplikasi web berbasis Laravel untuk membantu pelaku UMKM mencatat transaksi keuangan, mengelola produk & stok, serta menyediakan laporan keuangan (Laba/Rugi, Arus Kas, Piutang) dengan dua level akses (Admin & Non-Admin).

---

## Tech Stack

| Layer | Teknologi | Versi |
|-------|-----------|-------|
| Backend Framework | Laravel | 11 |
| Database | MySQL | 8+ |
| Frontend | Blade + Tailwind CSS | 3.x |
| Autentikasi | Laravel Breeze (Blade stack) | — |
| Chart | Chart.js | 4.x |
| Export PDF | Dompdf (barryvdh/laravel-dompdf) | — |
| Deployment | Google Cloud Run | — |

## Tim & Tanggung Jawab

| Role | Nama | Tanggung Jawab |
|------|------|----------------|
| **Project Leader & System Analyst** | Rifki | Setup project, koordinasi, dokumentasi, Dockerfile, deployment GCR, merge modul |
| **Backend (Model & Migration)** | Fadzhil | Migration, model & relasi, middleware, controller (Admin & Non-Admin), seeder, routes, stock logic, export |
| **Backend (API Testing)** | Satria | PHPUnit feature tests (10 kategori), Postman Collection, validasi form |
| **Frontend Developer** | Shidqi | Layout Blade, seluruh view per role, shared components, Chart.js, responsive design |
| **Quality Assurance** | Lita | Test plan, 40+ test case manual, bug reporting, regression testing |

## Fitur Utama

- **Manajemen Transaksi** — Pencatatan pemasukan & pengeluaran dengan dukungan detail produk
- **Manajemen Produk & Kategori** — CRUD produk, stok, kategori produk
- **Manajemen Pengguna** — CRUD akun non-admin, aktivasi/deaktivasi akun
- **Manajemen Stok Otomatis** — Stok berubah otomatis berdasarkan transaksi (income = stok keluar, expense = stok masuk)
- **Piutang (Receivables)** — Dukungan transaksi kredit dengan status unpaid/paid/overdue
- **Laporan Keuangan** — Laba/Rugi, Arus Kas (running balance), Piutang (aging report), Laporan Harian
- **Export CSV & PDF** — Semua laporan & tabel dapat diexport
- **Two Role Access** — Admin (full akses) dan Non-Admin (akses terbatas)
- **Bulk Delete** — Hapus transaksi massal dengan rollback stok otomatis

## Prerequisites

- **XAMPP 8.2+** (PHP + MySQL + Apache) — [download](https://www.apachefriends.org/)
- **Atau** **Laragon** (PHP + MySQL + Nginx/Apache) — [download](https://laragon.org/)
- Composer 2.x — [download](https://getcomposer.org/)
- Node.js 20+ & npm — [download](https://nodejs.org/)
- Git — [download](https://git-scm.com/)

> **Catatan:** XAMPP / Laragon sudah include PHP 8.2+ dan MySQL 8+. Pastikan PHP dan MySQL dari XAMPP/Laragon terdaftar di `PATH` system.

## Cara Install & Jalankan

```bash
# 0. Jalankan XAMPP
#    Buka XAMPP Control Panel → Start Apache → Start MySQL
#    (Atau jika pakai Laragon, cukup Start All)

# 1. Clone repository
git clone https://github.com/24530008muhammad-lab/umkm-kelompok-05.git
cd umkm-kelompok-05/backend

# 2. Install dependency PHP
composer install

# 3. Copy & konfigurasi environment
cp .env.example .env
# Edit .env:
#   DB_HOST=127.0.0.1
#   DB_PORT=3306
#   DB_DATABASE=uas-pw
#   DB_USERNAME=root
#   DB_PASSWORD=        # kosongkan untuk XAMPP default
#   SESSION_DRIVER=database
#   MAIL_MAILER=log

# 3b. Buat database (via phpMyAdmin atau terminal):
#     Opsi A — Buka http://localhost/phpmyadmin → Database → isi "uas-pw" → Create
#     Opsi B — mysql -u root -e "CREATE DATABASE IF NOT EXISTS uas-pw CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci"

# 4. Generate application key
php artisan key:generate

# 5. Jalankan migrasi & seeder
php artisan migrate --seed

# 6. Install & build frontend assets
npm install
npm run build

# 7. Jalankan development server
php artisan serve
# Buka http://127.0.0.1:8000
```

**Login default:**
- Admin: `admin@example.com` / `password`
- Non-Admin: `kasir@example.com` / `password`

## Struktur Project

```
umkm-kelompok-05/
├── backend/                     # Laravel 11 application
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   │   ├── Admin/       # Admin controllers (7)
│   │   │   │   ├── NonAdmin/    # Non-Admin controllers (4)
│   │   │   │   └── Auth/        # Extend Breeze
│   │   │   ├── Middleware/
│   │   │   │   └── CheckRole.php
│   │   │   └── Requests/        # FormRequest (8 files)
│   │   ├── Models/              # Eloquent models (6)
│   │   └── Services/            # Business logic services
│   │       ├── StockService.php
│   │       ├── TransactionService.php
│   │       └── ReportService.php
│   ├── database/
│   │   ├── migrations/          # 6 migration files
│   │   ├── factories/           # 6 model factories
│   │   └── seeders/             # DatabaseSeeder + seeders
│   ├── resources/views/
│   │   ├── layouts/
│   │   ├── admin/               # 10+ halaman admin
│   │   ├── nonadmin/            # 6 halaman non-admin
│   │   ├── shared/              # 4 komponen shared
│   │   └── auth/                # Login, register
│   ├── routes/
│   │   └── web.php              # Semua route definition
│   └── tests/                   # PHPUnit tests
├── docs/
│   ├── readme.md                # Acuan project (detail)
│   ├── changelog.md             # Catatan perubahan
│   └── agent-guides/            # Panduan per anggota
│       ├── 01-rifki-leader.md
│       ├── 02-fadzhil-backend.md
│       ├── 03-satria-api-testing.md
│       ├── 04-shidqi-frontend.md
│       └── 05-lita-qa.md
├── prd.md                       # Product Requirements Document
└── README.md                    # File ini
```

## Urutan Pengerjaan (Development Phases)

| Phase | Isi | Pengerja | Target Selesai |
|-------|-----|----------|----------------|
| **1. Foundation** | Setup Laravel 11 + Breeze, konfigurasi MySQL, struktur folder, Git branching | Rifki | — |
| **2. Database & Auth** | 6 migration, 6 model + relasi + casts, middleware CheckRole, auth override (is_active, redirect by role), seeder | Fadzhil | — |
| **3. Admin Backend** | CRUD controllers (users, categories, products, transactions), stock management logic, FormRequest, routes | Fadzhil | — |
| **4. Reports Backend** | ReportController (profit-loss, cash-flow, receivables, markAsPaid, bulk delete), export CSV & PDF | Fadzhil | — |
| **5. Non-Admin Backend** | DashboardController, TransactionController (create/history), ProductController (read-only), ReportController (daily) | Fadzhil | — |
| **6. Frontend** | Layout Blade, semua view admin & non-admin, shared components, Chart.js, responsive, error pages | Shidqi | — |
| **7. Testing** | PHPUnit feature tests (10 kategori), konfigurasi phpunit.xml, model factories | Satria | — |
| **8. QA** | Test plan, eksekusi 40+ test case, laporan bug, regression | Lita | — |
| **9. Deployment** | Dockerfile, cloudbuild.yaml, GCR deploy | Rifki | — |

> **Catatan:** Phase 1-5 bersifat sekuensial. Phase 6 (Frontend) bisa dimulai paralel setelah Phase 3. Phase 7-8 dimulai setelah Phase 5 selesai.

## Role & Hak Akses (RBAC)

| Modul | Admin | Non-Admin |
|-------|-------|-----------|
| Dashboard | Statistik global (semua user) | Ringkasan harian milik sendiri |
| Transaksi | CRUD semua transaksi | Create & Read milik sendiri (no edit/hapus) |
| Produk | CRUD penuh | Read-only |
| Kategori | CRUD penuh | Tidak bisa akses |
| Pengguna | CRUD + aktivasi/deaktivasi | Tidak bisa akses |
| Laporan Laba/Rugi | Akses penuh | Tidak bisa akses |
| Laporan Arus Kas | Akses penuh | Tidak bisa akses |
| Laporan Piutang | Akses penuh | Tidak bisa akses |
| Laporan Harian | — | Transaksi sendiri, tanggal tertentu |

## Aturan Bisnis Penting

- **403 Forbidden** — Non-Admin yang akses `/admin/*` mendapat HTTP 403
- **Stok Otomatis** — Transaksi income berkurang stok, expense (dengan produk) bertambah stok
- **Validasi Stok** — Transaksi income ditolak jika stok < quantity (pesan error: "Stok [nama] tidak cukup")
- **Race Condition** — Update stok menggunakan `DB::transaction()` + `lockForUpdate()` + retry 3x
- **Piutang** — Hanya untuk transaksi income, wajib `due_date`, status `unpaid/paid/overdue` (overdue = computed)
- **Soft Delete** — Transaksi menggunakan soft delete (audit trail), model lain hard delete
- **Restrict Delete** — Kategori tidak bisa dihapus jika masih punya produk; produk tidak bisa dihapus jika punya riwayat transaksi
- **Cash Flow** — Basis akrual (semua transaksi termasuk unpaid dihitung sebagai inflow/outflow)

## Dokumentasi Terkait

| Dokumen | Deskripsi |
|---------|-----------|
| [PRD Master](prd.md) | Product Requirements Document lengkap (742 baris) |
| [Changelog](docs/changelog.md) | Catatan perubahan & riwayat versi |
| [Acuan Project](docs/readme.md) | Panduan detail struktur & setup |
| [Panduan Rifki](docs/agent-guides/01-rifki-leader.md) | Setup, integrasi, deployment |
| [Panduan Fadzhil](docs/agent-guides/02-fadzhil-backend.md) | Backend model, migration, controller |
| [Panduan Satria](docs/agent-guides/03-satria-api-testing.md) | PHPUnit testing, Postman |
| [Panduan Shidqi](docs/agent-guides/04-shidqi-frontend.md) | Frontend Blade, Tailwind, Chart.js |
| [Panduan Lita](docs/agent-guides/05-lita-qa.md) | QA test plan, test case, bug report |

## Deployment

Aplikasi dideploy ke **Google Cloud Run** dengan **Cloud SQL (MySQL)** sebagai database production.

Arsitektur deployment:
```
Cloud Run (stateless) → Cloud SQL (MySQL)
     ├── Session: database driver
     ├── Storage: local (public disk)
     └── Env vars: via Secret Manager
```

---

**© 2026 Kelompok 5 — UAS Pemrograman Web**
