# Aplikasi Pengelolaan Keuangan UMKM

**Kelompok 5 | UAS Pemrograman Web**

| Role | Nama |
|------|------|
| Project Leader & System Analyst | Rifki |
| Backend (Model & Migration) | Fadzhil |
| Backend (API Testing) | Satria |
| Frontend Developer | Shidqi |
| Quality Assurance | Lita |

## Tech Stack

- **Backend:** Laravel 11 + MySQL (via XAMPP/Laragon)
- **Frontend:** Blade Templating + Tailwind CSS
- **Auth:** Laravel Breeze (extended dengan role & status aktif)
- **Chart:** Chart.js v4
- **Export:** Dompdf (PDF) + CSV built-in
- **Deployment:** Google Cloud Run (GCR)

## Lingkungan Development

Local development menggunakan **XAMPP** atau **Laragon** yang sudah include:
- **Apache** — web server
- **MySQL 8+** — database server (port 3306)
- **PHP 8.2+** — interpreter
- **phpMyAdmin** — database GUI di `http://localhost/phpmyadmin`

## Struktur Project

```
umkm-kelompok-05/
├── backend/                    # Laravel 11 application
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
│   │       ├── ReportService.php
│   │       └── ExportService.php
│   ├── database/
│   │   ├── migrations/          # 6 migration files (+indexes)
│   │   ├── factories/           # 6 model factories
│   │   └── seeders/             # ProductionSeeder + DevelopmentSeeder
│   ├── resources/views/
│   │   ├── layouts/
│   │   ├── admin/               # 12+ halaman admin
│   │   ├── nonadmin/            # 6 halaman non-admin
│   │   ├── shared/              # 6 komponen shared
│   │   └── errors/              # 403, 404
│   ├── routes/
│   │   └── web.php              # Semua route definition
│   └── tests/                   # PHPUnit feature tests (10 files)
├── docs/
│   ├── readme.md                # Ini — acuan utama project
│   ├── changelog.md             # Catatan perubahan
│   └── agent-guides/            # Panduan per anggota tim
│       ├── 01-rifki-leader.md
│       ├── 02-fadzhil-backend.md
│       ├── 03-satria-api-testing.md
│       ├── 04-shidqi-frontend.md
│       └── 05-lita-qa.md
├── prd.md                       # Product Requirements Document (Master)
└── README.md                    # Root README

## Panduan Memulai

1. Install **XAMPP 8.2+** (atau Laragon)
2. Start Apache & MySQL dari XAMPP Control Panel
3. Clone repo & masuk ke `backend/`
4. `composer install`
5. Copy `.env.example` ke `.env`, sesuaikan DB credentials
6. Buat database `uas-pw` via phpMyAdmin (`http://localhost/phpmyadmin`)
7. `php artisan key:generate`
8. `php artisan migrate --seed`
9. `npm install && npm run build`
10. `php artisan serve`
11. Buka `http://127.0.0.1:8000`

Login: `admin@example.com` / `password`

## Role & Hak Akses

| Modul | Admin | Non-Admin |
|-------|-------|-----------|
| Dashboard | Statistik global | Ringkasan harian sendiri |
| Transaksi | CRUD semua | Create & Read milik sendiri |
| Produk | CRUD penuh | Read-only |
| Kategori | CRUD penuh | Tidak bisa akses |
| Pengguna | CRUD + aktivasi | Tidak bisa akses |
| Laporan Laba/Rugi | Akses penuh | Tidak bisa |
| Laporan Arus Kas | Akses penuh | Tidak bisa |
| Laporan Piutang | Akses penuh | Tidak bisa |
| Laporan Harian | - | Transaksi sendiri |

## Aturan Penting

- Non-Admin yang akses `/admin/*` → **HTTP 403 Forbidden** (admin BOLEH akses `/nonadmin/*`)
- Stok otomatis: income = stok berkurang, expense (dengan produk) = stok bertambah
- Validasi stok: income ditolak jika stok < quantity
- Race condition: `DB::transaction()` + `lockForUpdate()` + retry 3x
- Piutang hanya untuk transaksi income (`is_credit=true` + `due_date`)
- Soft delete hanya untuk Transaction (audit trail)
- Kategori tidak bisa dihapus jika masih punya produk
- Produk tidak bisa dihapus jika punya riwayat transaksi
- Cash Flow basis akrual (semua transaksi termasuk unpaid)

## Deployment

Aplikasi dideploy ke **Google Cloud Run (GCR)** + **Cloud SQL MySQL**.
Repository: `github.com/24530008muhammad-lab/umkm-kelompok-05`.
