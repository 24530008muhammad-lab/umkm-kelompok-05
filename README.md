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

## Tutorial Pengerjaan per Anggota

### Git Workflow

```
main (production, protected — hanya leader yang merge ke sini)
  └── develop (integration — leader merge dari feature branch ke sini)
       ├── feature/fadzhil-backend    ← Fadzhil
       ├── feature/satria-testing     ← Satria
       ├── feature/shidqi-frontend    ← Shidqi
       └── feature/lita-qa            ← Lita
```

**Aturan:**
1. Setiap anggota **WAJIB** buat branch sendiri dari `develop`: `git checkout -b feature/nama-anggota develop`
2. Kerja di branch masing-masing, **jangan push langsung ke develop/main**
3. Setelah selesai → `git push -u origin feature/nama-anggota`
4. Bikin **Pull Request** ke branch `develop` via GitHub
5. Leader (Rifki) yang review & merge PR
6. Setelah semua fitur selesai, leader merge `develop` → `main`

### Panduan per Anggota

**Cara pakai:** Salin block prompt di bawah, kirim ke AI agent kamu (atau jadikan sebagai task description). AI agent akan membaca file agent-guide terkait lalu mengerjakan bagiannya.

---

#### 👤 Rifki — Phase 1 & 9 (Foundation + Deployment)

```bash
git checkout -b feature/rifki-setup develop
# Kerjakan tugas di docs/agent-guides/01-rifki-leader.md
git add -A && git commit -m "Setup Laravel 11 + Breeze + Dockerfile"
git push -u origin feature/rifki-setup
# Bikin PR ke develop via GitHub
```

<details>
<summary>📋 Prompt untuk AI Agent Rifki</summary>

```
Kamu adalah AI Agent untuk Rifki (Project Leader). 
Repo: https://github.com/24530008muhammad-lab/umkm-kelompok-05
Branch: feature/rifki-setup (buat dari develop)

Baca panduan lengkap di docs/agent-guides/01-rifki-leader.md

Tugas:
1. Setup Laravel 11 di folder backend/ (composer create-project)
2. Install Breeze Blade stack
3. Konfigurasi .env untuk XAMPP (DB_HOST=127.0.0.1, DB_DATABASE=uas-pw, etc)
4. Buat AppServiceProvider (Schema::defaultStringLength)
5. Nonaktifkan route forgot-password & reset-password
6. Buat Dockerfile (multi-stage) & cloudbuild.yaml
7. Setup branch: main (protected), develop (protected), feature/*

Setelah selesai, commit & push. Beri tahu Rifki hasilnya.
```
</details>

---

#### 👤 Fadzhil — Phase 2, 3, 4, 5 (Backend Full)

```bash
git checkout -b feature/fadzhil-backend develop
# Kerjakan tugas di docs/agent-guides/02-fadzhil-backend.md
git add -A && git commit -m "Migration, model, controller, services"
git push -u origin feature/fadzhil-backend
# Bikin PR ke develop via GitHub
```

<details>
<summary>📋 Prompt untuk AI Agent Fadzhil</summary>

```
Kamu adalah AI Agent untuk Fadzhil (Backend Developer).
Repo: https://github.com/24530008muhammad-lab/umkm-kelompok-05
Branch: feature/fadzhil-backend (buat dari develop)

Baca panduan lengkap di docs/agent-guides/02-fadzhil-backend.md

PRASYARAT: Pastikan Phase 1 (Rifki) sudah selesai. Pull dari develop.

Tugas (kerjakan urut):
1. 6 Migration (+indexes, FK RESTRICT/CASCADE, SoftDeletes di transactions)
2. 6 Model (+casts, accessors, scopes, SoftDeletes di Transaction)
3. 8 FormRequest (+ custom validasi stok, is_credit)
4. Middleware CheckRole (admin boleh akses nonadmin routes)
5. Auth override (is_active check, redirect by role)
6. Service layer: StockService (lockForUpdate), TransactionService, ReportService, ExportService
7. Event/Listener untuk stock (opsional)
8. Admin Controllers: Dashboard, User, Category, Product, Transaction (+bulkDelete), Report (+export)
9. Non-Admin Controllers: Dashboard, Transaction, Product, Report
10. Seeder: ProductionSeeder (admin + settings), DevelopmentSeeder (sample data)
11. 6 Model Factories
12. Routes: semua termasuk bulk-delete & export

VERIFIKASI:
- php artisan migrate:fresh --seed berhasil
- php artisan route:list menunjukkan semua route
- Login admin redirect ke /admin/dashboard
- Login nonadmin redirect ke /nonadmin/dashboard
- Nonadmin akses /admin/* → 403
- Admin akses /nonadmin/* → OK (boleh)

Setelah selesai, commit & push. Beri tahu Fadzhil & Rifki hasilnya.
```
</details>

---

#### 👤 Shidqi — Phase 6 (Frontend)

```bash
git checkout -b feature/shidqi-frontend develop
# Kerjakan tugas di docs/agent-guides/04-shidqi-frontend.md
git add -A && git commit -m "Semua view Blade + Chart.js + responsive"
git push -u origin feature/shidqi-frontend
# Bikin PR ke develop via GitHub
```

<details>
<summary>📋 Prompt untuk AI Agent Shidqi</summary>

```
Kamu adalah AI Agent untuk Shidqi (Frontend Developer).
Repo: https://github.com/24530008muhammad-lab/umkm-kelompok-05
Branch: feature/shidqi-frontend (buat dari develop)

Baca panduan lengkap di docs/agent-guides/04-shidqi-frontend.md

PRASYARAT: Pastikan Phase 3 (Fadzhil) sudah selesai (controller & routes sudah ada). Pull dari develop.

Tugas (kerjakan urut):
1. Install Chart.js v4 + @tailwindcss/forms via npm
2. Layout: auth.blade.php (login/register), app.blade.php (sidebar dinamis by role, navbar, alert)
3. Shared Components: sidebar-admin, sidebar-nonadmin, alert, product-card, stock-badge, confirm-dialog
4. Admin Views:
   - dashboard (KPI cards, 2 chart, top 5 produk, transaksi terbaru)
   - transactions/index (search, filter, tabel, bulk delete checkbox, export buttons, pagination, summary)
   - transactions/form (date picker, radio type, dynamic product table + modal, is_credit checkbox, auto-calc amount)
   - products/index (search, filter kategori, stock badge, export CSV, pagination)
   - products/form, categories/index, categories/form
   - users/index (search, filter role, toggle active), users/form
   - reports/profit-loss (chart, export PDF), cash-flow (line chart, running balance), receivables (aging, mark as paid)
5. Non-Admin Views:
   - dashboard (greeting, today summary, recent transactions, low stock)
   - transactions/create (tanpa is_credit), transactions/history (tanpa edit/delete)
   - products/index (card grid read-only), products/show
   - reports/daily (date picker, summary, tabel, chart)
6. Error pages: 403.blade.php, 404.blade.php (Tailwind styled)
7. Responsive: sidebar collapsible mobile, overflow-x-auto tabel, grid responsive

VERIFIKASI:
- npm run build berhasil tanpa error
- Semua halaman bisa diakses via browser
- Stock badge muncul dengan warna sesuai (merah/kuning/hijau)
- Chart.js menampilkan grafik di dashboard & reports
- Modal pilih produk muncul & berfungsi
- Konfirmasi delete muncul saat klik tombol hapus
- Tampilan responsive di mobile (375px)

Setelah selesai, commit & push. Beri tahu Shidqi & Rifki hasilnya.
```
</details>

---

#### 👤 Satria — Phase 7 (API Testing)

```bash
git checkout -b feature/satria-testing develop
# Kerjakan tugas di docs/agent-guides/03-satria-api-testing.md
git add -A && git commit -m "PHPUnit tests + Postman collection"
git push -u origin feature/satria-testing
# Bikin PR ke develop via GitHub
```

<details>
<summary>📋 Prompt untuk AI Agent Satria</summary>

```
Kamu adalah AI Agent untuk Satria (API Testing).
Repo: https://github.com/24530008muhammad-lab/umkm-kelompok-05
Branch: feature/satria-testing (buat dari develop)

Baca panduan lengkap di docs/agent-guides/03-satria-api-testing.md

PRASYARAT: Pastikan Phase 5 (Fadzhil) sudah selesai. Pull dari develop.

Tugas (kerjakan urut):
1. Konfigurasi phpunit.xml (SQLITE in-memory, coverage 70% overall, 90% services)
2. Setup TestCase: RefreshDatabase + seed ProductionSeeder + buat admin & nonadmin user
3. 10 File Feature Test:
   - AuthTest (6 test + 2 edge case)
   - MiddlewareTest (4 test)
   - AdminUserTest (8 test + edge cases)
   - AdminCategoryTest (5 test + edge cases)
   - AdminProductTest (5 test + edge cases)
   - AdminTransactionTest (12 test + concurrent stock test + export test + edge cases)
   - AdminReportTest (8 test + export PDF test + edge cases)
   - NonAdminTransactionTest (5 test)
   - NonAdminProductTest (3 test)
   - NonAdminReportTest (3 test)
4. Concurrent stock update test (simulasi 3 request simultan, 2 sukses 1 gagal)
5. Postman Collection (4 folder, semua endpoint, environment variables)

VERIFIKASI:
- php artisan test → semua passing (minimal 50+ test)
- php artisan test --coverage → minimal 70%
- Postman collection bisa di-import & dijalankan

Setelah selesai, commit & push. Beri tahu Satria & Rifki hasilnya.
```
</details>

---

#### 👤 Lita — Phase 8 (QA)

```bash
git checkout -b feature/lita-qa develop
# Kerjakan tugas di docs/agent-guides/05-lita-qa.md
git add -A && git commit -m "Test plan + test case + bug report"
git push -u origin feature/lita-qa
# Bikin PR ke develop via GitHub
```

<details>
<summary>📋 Prompt untuk AI Agent Lita</summary>

```
Kamu adalah AI Agent untuk Lita (Quality Assurance).
Repo: https://github.com/24530008muhammad-lab/umkm-kelompok-05
Branch: feature/lita-qa (buat dari develop)

Baca panduan lengkap di docs/agent-guides/05-lita-qa.md

PRASYARAT: Pastikan semua fase development selesai (aplikasi sudah jalan). Pull dari develop.

Tugas:
1. Buat dokumen docs/qa-test-plan.md (scope, tools, prioritas)
2. Eksekusi 40+ test case manual:
   - TC-AUTH: 8 test (login admin/nonadmin, 403, is_active, register, guest, admin access nonadmin)
   - TC-TRX: 9 test (CRUD transaksi, stok, bulk delete, export CSV)
   - TC-CREDIT: 7 test (piutang, overdue, mark as paid, is_credit on expense)
   - TC-RPT: 8 test (laporan laba rugi, arus kas, piutang, export PDF, daily)
   - TC-MASTER: 7 test (hapus kategori/produk, nonadmin akses, toggle user, export produk)
   - TC-VAL: 9 test (validasi form, edge cases)
   - TC-UI: 9 test (responsive 3 ukuran, stock badge, alert, cross-browser, error page)
   - TC-EXPORT: 3 test (CSV transaksi, PDF laporan, PDF arus kas)
3. Catat hasil setiap test (Pass/Fail/Blocked) di spreadsheet
4. Jika ada bug: buat Bug Report (template di agent-guide) → assign ke anggota terkait
5. Regression testing setelah bug fix
6. Laporan QA final (ringkasan, rekomendasi Release/Delayed)

VERIFIKASI:
- Semua test case selesai dijalankan
- Bug report terisi untuk setiap issue yang ditemukan
- Laporan QA final siap

Setelah selesai, commit & push. Beri tahu Lita & Rifki hasilnya.
```
</details>

---

#### 👤 Rifki — Integrasi & Merge (setelah semua selesai)

```bash
git checkout develop
git pull origin develop
# Cek setiap PR, jika OK merge:
git merge --no-ff feature/fadzhil-backend
git merge --no-ff feature/shidqi-frontend
git merge --no-ff feature/satria-testing
git merge --no-ff feature/lita-qa
git push origin develop

# Uji coba:
php artisan migrate:fresh --seed
php artisan test
npm run build

# Jika semua OK, merge ke main:
git checkout main
git merge --no-ff develop
git push origin main
```

<details>
<summary>📋 Prompt untuk AI Agent Rifki (Integrasi)</summary>

```
Kamu adalah AI Agent untuk Rifki (Project Leader) — Fase Integrasi.

Repo: https://github.com/24530008muhammad-lab/umkm-kelompok-05

Tugas:
1. Pull branch develop (git pull origin develop)
2. Review & merge setiap PR dari anggota ke develop:
   - feature/fadzhil-backend → develop
   - feature/shidqi-frontend → develop
   - feature/satria-testing → develop
   - feature/lita-qa → develop
3. Setelah semua merge, jalankan:
   - php artisan migrate:fresh --seed (pastikan migrasi & seeder jalan)
   - php artisan test (pastikan semua test lulus)
   - npm run build (pastikan build frontend sukses)
4. Jika ada konflik: resolve dengan prioritas (backend > frontend > testing)
5. Jika semua OK, merge develop → main
6. Deploy ke GCR (ikuti docs/agent-guides/01-rifki-leader.md)

Beri tahu tim bahwa integrasi selesai & aplikasi siap.
```
</details>

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
