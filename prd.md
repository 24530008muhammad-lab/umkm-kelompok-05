# PRD (Product Requirements Document)
# Aplikasi Pengelolaan Keuangan UMKM

**Versi:** 1.0 (Master)
**Status:** Final untuk Development
**Stack:** Laravel 11, MySQL, Blade, Tailwind CSS, Laravel Breeze

---

## 1. Informasi Proyek

| Item | Detail |
|---|---|
| Nama Proyek | Aplikasi Pengelolaan Keuangan UMKM |
| Kelompok | Kelompok 5 |
| Backend Framework | Laravel 11 |
| Database | MySQL |
| Frontend | Blade Templating + Tailwind CSS |
| Autentikasi | Laravel Breeze (extended dengan role & status aktif) |
| Repository | github.com/24530008muhammad-lab/umkm-kelompok-05 |
| Deployment | Google Cloud Run (GCR) |

### 1.1 Tim & Tanggung Jawab

| Peran | Nama | Tanggung Jawab Utama |
|---|---|---|
| Project Leader & System Analyst | Rifki | Spesifikasi kebutuhan, rancangan database, dokumen API, struktur folder, pembagian tugas, integrasi modul |
| Backend Developer (Model & Migration) | Fadzhil | Migration, model & relasi, autentikasi, middleware role, controller, seeder |
| Backend Developer (API Testing) | Satria | Unit test endpoint, validasi, dokumentasi Postman |
| Frontend Developer | Shidqi | Layout, view per role, integrasi Tailwind, komponen alert |
| Quality Assurance (QA) | Lita | Test plan, pengujian manual, laporan bug |

---

## 2. Tujuan & Lingkup Proyek

### 2.1 Tujuan
Membangun aplikasi web yang membantu pelaku UMKM:
- Mencatat transaksi keuangan (pemasukan & pengeluaran).
- Mengelola data produk dan kategori produk beserta stoknya.
- Mengelola akun pengguna non-admin (karyawan/kasir).
- Menyediakan laporan keuangan: Laba/Rugi, Arus Kas, dan Piutang.
- Menyediakan dua level akses (Admin & Non-Admin) dengan pembatasan hak akses yang jelas.

### 2.2 Lingkup (In-Scope)
- CRUD penuh untuk: users (non-admin), categories, products, transactions (oleh Admin).
- Pencatatan transaksi terbatas (create & read milik sendiri) oleh Non-Admin.
- Manajemen stok otomatis berbasis transaksi.
- Laporan: Profit-Loss, Cash-Flow, Receivables (Piutang), Daily Report (non-admin).
- Autentikasi & otorisasi berbasis role (middleware).
- REST API endpoints terdokumentasi di Postman Collection.
- ERD, repository GitHub, demo aplikasi, deployment ke GCR.

### 2.3 Di Luar Lingkup (Out-of-Scope) — *untuk versi ini*
- Integrasi payment gateway (pembayaran online).
- Multi-cabang / multi-gudang.
- Manajemen supplier/vendor terpisah.
- Perhitungan pajak otomatis.
- Notifikasi email otomatis (tombol "Send reminder" pada Piutang bersifat opsional/stub, tidak wajib terhubung ke SMTP).
- Multi-currency.

---

## 3. Definisi & Istilah

| Istilah | Definisi |
|---|---|
| UMKM | Usaha Mikro, Kecil, dan Menengah — pengguna utama aplikasi |
| Admin | Pemilik usaha, akses penuh ke seluruh fitur |
| Non-Admin | Karyawan/kasir/staf keuangan, akses terbatas |
| Transaksi Income | Transaksi pemasukan (umumnya dari penjualan produk) |
| Transaksi Expense | Transaksi pengeluaran (operasional atau pembelian stok) |
| Piutang (Receivable) | Transaksi income yang dilakukan secara kredit (belum dibayar penuh) |
| Stok | Jumlah unit produk yang tersedia |
| CRUD | Create, Read, Update, Delete |

---

## 4. Role & Hak Akses (RBAC Matrix)

| Modul | Admin | Non-Admin |
|---|---|---|
| Dashboard | Statistik & ringkasan global (semua user) | Ringkasan harian milik sendiri |
| Transaksi | CRUD semua transaksi (semua user) | Create & Read transaksi milik sendiri saja (tidak bisa edit/hapus) |
| Produk | CRUD penuh | Read-only |
| Kategori | CRUD penuh | Tidak bisa akses |
| Pengguna (Users) | CRUD akun non-admin, aktivasi/deaktivasi | Tidak bisa akses |
| Laporan Laba/Rugi | Akses penuh, semua user/periode | Tidak bisa akses |
| Laporan Arus Kas | Akses penuh | Tidak bisa akses |
| Laporan Piutang | Akses penuh, tandai lunas | Tidak bisa akses |
| Laporan Harian | Tidak relevan (gunakan Laba/Rugi) | Hanya transaksi milik sendiri, hari tertentu |
| Halaman `/admin/*` | Akses penuh | **Dilarang** — **HTTP 403 Forbidden** |
| Halaman `/nonadmin/*` | Akses penuh (boleh, tidak dilarang) | Akses penuh |

> **Aturan tegas:** Non-Admin yang mencoba mengakses route dengan prefix `/admin/*` harus menerima **HTTP 403 Forbidden** menggunakan `abort(403)` di middleware `CheckRole`. Admin BOLEH mengakses route `/nonadmin/*`.

---

## 5. Skema Database (ERD)

### 5.1 Tabel `users`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | ID unik pengguna |
| name | VARCHAR(100) | NOT NULL | Nama lengkap |
| email | VARCHAR(100) | UNIQUE, NOT NULL | Email login |
| password | VARCHAR(255) | NOT NULL | Hash password |
| role | ENUM('admin','nonadmin') | NOT NULL, DEFAULT 'nonadmin' | Role akses |
| is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | Status aktif/nonaktif akun |
| email_verified_at | TIMESTAMP | NULLABLE | Bawaan Breeze |
| remember_token | VARCHAR(100) | NULLABLE | Bawaan Breeze |
| created_at, updated_at | TIMESTAMP | - | Default Laravel |

**Catatan penting:** Role **hanya** `admin` dan `nonadmin`. Opsi "supervisor/etc" pada spesifikasi awal **dihapus** untuk menjaga konsistensi dengan logika bisnis (lihat Bagian 9).

### 5.2 Tabel `categories`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | ID kategori |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Nama kategori |
| description | TEXT | NULLABLE | Deskripsi opsional |
| created_at, updated_at | TIMESTAMP | - | |

### 5.3 Tabel `products`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | ID produk |
| name | VARCHAR(100) | NOT NULL | Nama produk |
| price | DECIMAL(15,2) | NOT NULL, >= 0 | Harga jual saat ini |
| stock | INT | NOT NULL, DEFAULT 0, >= 0 | Stok tersedia |
| category_id | BIGINT UNSIGNED | FK → categories.id, NOT NULL, ON DELETE RESTRICT | Kategori produk |
| created_at, updated_at | TIMESTAMP | - | |
| | | INDEX on category_id | |

### 5.4 Tabel `transactions`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | ID unik transaksi |
| transaction_date | DATE | NOT NULL | Tanggal transaksi |
| type | ENUM('income','expense') | NOT NULL | Jenis transaksi |
| description | VARCHAR(255) | NULLABLE | Keterangan transaksi |
| amount | DECIMAL(15,2) | NOT NULL, >= 0 | Total nominal transaksi |
| user_id | BIGINT UNSIGNED | FK → users.id, NOT NULL | User yang mencatat |
| | | INDEX on transaction_date, type, user_id, is_credit, payment_status | |
| is_credit | BOOLEAN | NOT NULL, DEFAULT FALSE | Apakah transaksi ini piutang (hanya valid untuk type='income') |
| due_date | DATE | NULLABLE | Wajib diisi jika `is_credit = true` |
| payment_status | ENUM('unpaid','paid') | NULLABLE, DEFAULT NULL | Status pembayaran (hanya relevan jika `is_credit = true`); default `'unpaid'` saat dibuat |
| created_at, updated_at | TIMESTAMP | - | |

> **Field tambahan dari PRD awal:** `is_credit`, `due_date`, `payment_status` ditambahkan karena fitur Laporan Piutang (Receivables) membutuhkan data ini, namun tidak tercantum di skema awal.

### 5.5 Tabel `transaction_details`

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | ID detail |
| transaction_id | BIGINT UNSIGNED | FK → transactions.id, NOT NULL, ON DELETE CASCADE | Induk transaksi |
| product_id | BIGINT UNSIGNED | FK → products.id, NOT NULL, ON DELETE RESTRICT | Produk terkait |
| quantity | INT | NOT NULL, > 0 | Jumlah unit |
| price_per_unit | DECIMAL(15,2) | NOT NULL, >= 0 | Harga per unit saat transaksi (bisa beda dari harga produk saat ini) |
| subtotal | DECIMAL(15,2) | NOT NULL, = quantity * price_per_unit (computed/ diisi manual) | Subtotal baris |
| created_at, updated_at | TIMESTAMP | - | |
| | | INDEX on transaction_id, product_id | |

> **Catatan:** Satu `transaction` boleh memiliki **0 (nol)** baris `transaction_details`. Ini digunakan untuk transaksi non-produk seperti biaya sewa, listrik, gaji, dll (lihat Bagian 9.1).

### 5.6 Tabel `settings` *(tabel baru)*

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT | |
| key | VARCHAR(100) | UNIQUE, NOT NULL | Kunci konfigurasi |
| value | TEXT | NOT NULL | Nilai konfigurasi (TEXT untuk fleksibilitas) |
| created_at, updated_at | TIMESTAMP | - | |

**Seed data wajib:**

| key | value | Keterangan |
|---|---|---|
| `cash_opening_balance` | `0` | Saldo kas awal sistem |
| `cash_opening_date` | tanggal hari sistem pertama dijalankan (YYYY-MM-DD) | Tanggal acuan saldo awal |
| `low_stock_threshold` | `10` | Ambang batas stok rendah untuk indikator UI |

> **Catatan:** Tabel ini ditambahkan untuk menyimpan **saldo awal kas** yang dibutuhkan oleh Laporan Arus Kas, yang tidak ada di skema awal.

### 5.7 Relasi Antar Tabel (ERD Sederhana)

```
users (1) ----< (N) transactions
categories (1) ----< (N) products
products (1) ----< (N) transaction_details
transactions (1) ----< (N) transaction_details
```

- `transactions.user_id` → `users.id`
- `products.category_id` → `categories.id`
- `transaction_details.transaction_id` → `transactions.id` (CASCADE DELETE)
- `transaction_details.product_id` → `products.id` (RESTRICT DELETE — lihat 9.5)

---

## 6. Model & Relasi (app/Models/)

| Model | Relasi | Casts & SoftDeletes | Scopes & Accessors |
|---|---|---|---|
| `User.php` | `hasMany(Transaction::class)` | `is_active → boolean` | `scopeActive()` — filter is_active=true |
| `Category.php` | `hasMany(Product::class)` | — | `getProductCountAttribute()` |
| `Product.php` | `belongsTo(Category)`, `hasMany(TransactionDetail)` | `price → decimal:2`, `stock → integer` | `getStockStatusAttribute()` (out of stock / low stock / normal), `scopeLowStock($threshold)`, `scopeInStock()` |
| `Transaction.php` | `belongsTo(User)`, `hasMany(TransactionDetail)` | **SoftDeletes**, `amount → decimal:2`, `transaction_date → date`, `due_date → date`, `is_credit → boolean` | `getPaymentStatusLabelAttribute()` (unpaid/paid/overdue), `getIsOverdueAttribute()`, `scopeType($type)`, `scopeDateRange($from, $to)`, `scopeCredit()` |
| `TransactionDetail.php` | `belongsTo(Transaction)`, `belongsTo(Product)` | `price_per_unit → decimal:2`, `subtotal → decimal:2` | — |
| `Setting.php` | — | static accessor `Setting::get('key')`, `Setting::set('key', 'value')` | — |

---

## 7. Logika Bisnis (Business Rules)

### 7.1 Tipe Transaksi & Detail Produk

Sebuah transaksi (`transactions`) dapat berupa salah satu dari 3 kombinasi berikut:

| Kombinasi | type | Memiliki `transaction_details`? | Efek terhadap Stok | Contoh |
|---|---|---|---|---|
| A | `income` | Ya | **Stok berkurang** sebesar `quantity` per produk | Penjualan produk ke pelanggan |
| B | `expense` | Ya | **Stok bertambah** sebesar `quantity` per produk | Pembelian/restock barang dagangan |
| C | `expense` | Tidak (0 baris detail) | Tidak ada efek ke stok | Bayar sewa, listrik, gaji, dll |
| D | `income` | Tidak (0 baris detail) | Tidak ada efek ke stok | Pemasukan non-produk (jasa, dll) |

> Kombinasi A & B disebut "transaksi berbasis produk". Pada kombinasi ini, field `amount` pada `transactions` **dihitung otomatis** sebagai `SUM(transaction_details.subtotal)` dan **read-only** di form. Pada kombinasi C & D, `amount` **diinput manual** oleh user.

### 7.2 Aturan Manajemen Stok

1. **Validasi saat input transaksi `income` dengan detail produk:**
   - Untuk setiap baris detail, sistem memvalidasi `quantity <= product.stock` sebelum transaksi disimpan.
   - Jika tidak cukup → tolak dengan pesan error: *"Stok [nama produk] tidak cukup. Tersedia: X"*.
2. **Saat transaksi `income` + detail disimpan:** `product.stock -= quantity` untuk setiap baris.
3. **Saat transaksi `expense` + detail disimpan:** `product.stock += quantity` untuk setiap baris (dianggap restock).
4. **Saat transaksi diedit:** stok harus di-rollback dahulu ke kondisi sebelum perubahan, lalu diterapkan ulang sesuai data baru.
5. **Saat transaksi dihapus:** stok di-rollback (kebalikan dari poin 2/3).
6. **Race condition handling:** semua operasi yang mengubah stok (store/update/destroy transaksi dengan detail produk) **WAJIB** dibungkus dalam `DB::transaction()`. Sebelum membaca stok untuk update, gunakan **pessimistic locking** `Product::whereIn('id', $ids)->lockForUpdate()` untuk mencegah race condition. Gunakan retry 3x pada transaksi database: `DB::transaction(function(){...}, 3)`.
7. **Stok tidak boleh negatif** — validasi ini berlaku di level form request (FormRequest validation) dan di level model (defensive check).

### 7.3 Aturan Piutang (Receivables)

1. Field `is_credit`, `due_date`, `payment_status` **hanya berlaku jika `type = 'income'`**. Jika `type = 'expense'`, ketiga field ini dipaksa `false`/`null` di level controller/model.
2. Jika `is_credit = true` saat create/update transaksi income:
   - `due_date` **wajib diisi** dan harus `>= transaction_date`.
   - `payment_status` otomatis diset `'unpaid'` saat pertama dibuat.
3. Status "**overdue**" bukan nilai tersimpan di database, melainkan **dihitung saat ditampilkan** (computed/accessor):
   ```
   if (payment_status == 'unpaid' AND due_date < hari_ini) → tampil sebagai "Overdue"
   else → tampil sesuai payment_status ("Paid" / "Unpaid")
   ```
4. Admin dapat menandai transaksi sebagai "Lunas" melalui tombol **"Mark as Paid"** → mengubah `payment_status` menjadi `'paid'`. Aksi ini **tidak mengubah stok atau amount**.
5. Tombol "Send Reminder" (opsional) — jika diimplementasikan, hanya menampilkan notifikasi sukses (UI-only / stub), tidak wajib terintegrasi email pada versi ini.

### 7.4 Laporan Laba/Rugi (Profit & Loss)

Untuk rentang tanggal `[from, to]` (dan filter opsional: `category`, `user_id`):

```
Total Income  = SUM(transactions.amount) WHERE type='income' AND transaction_date BETWEEN [from, to] (+ filter)
Total Expense = SUM(transactions.amount) WHERE type='expense' AND transaction_date BETWEEN [from, to] (+ filter)
Net Profit/Loss = Total Income - Total Expense
Profit Margin % = (Net Profit / Total Income) * 100   -- jika Total Income = 0, maka Profit Margin = 0%
```

- Filter `by category`: berlaku pada transaksi yang memiliki `transaction_details` dengan produk pada kategori tersebut (transaksi non-produk dikecualikan dari filter kategori).
- Filter `by user`: berdasarkan `transactions.user_id`.

### 7.5 Laporan Arus Kas (Cash Flow)

Untuk rentang tanggal `[from, to]`:

```
Opening Balance = nilai `cash_opening_balance` dari tabel settings
                  + SUM(income.amount) - SUM(expense.amount) untuk SEMUA transaksi
                    dengan transaction_date < `from` (akumulasi sejak cash_opening_date)

Untuk setiap baris transaksi (urut by transaction_date ASC, lalu id ASC) dalam rentang [from, to]:
    Inflow  = amount jika type='income'
    Outflow = amount jika type='expense'
    Running Balance = Running Balance (baris sebelumnya, atau Opening Balance jika baris pertama)
                       + Inflow - Outflow

Closing Balance = Opening Balance + Total Inflow - Total Outflow (dalam rentang [from,to])
```

> Transaksi `is_credit=true` dengan `payment_status='unpaid'` **tetap dihitung sebagai inflow** pada Cash Flow (pencatatan berbasis **akrual sederhana**). Keputusan final: akrual — semua transaksi dicatat saat terjadi, bukan saat dibayar.

### 7.6 Laporan Harian (Non-Admin)

Untuk tanggal tertentu (default: hari ini) dan `user_id = auth user`:

```
Today Income  = SUM(amount) WHERE type='income' AND user_id=X AND transaction_date=tanggal
Today Expense = SUM(amount) WHERE type='expense' AND user_id=X AND transaction_date=tanggal
Today Net     = Today Income - Today Expense
```

### 7.7 Aturan Penghapusan Kategori

- Kategori **tidak dapat dihapus** jika `products.where('category_id', $id)->count() > 0`.
- Jika user mencoba hapus → tampilkan pesan error: *"Kategori tidak dapat dihapus karena masih memiliki N produk."* (HTTP 422 jika via API).

### 7.8 Aturan Penghapusan Produk

- Produk **tidak dapat dihapus** jika memiliki riwayat di `transaction_details` (RESTRICT). Tampilkan pesan: *"Produk tidak dapat dihapus karena memiliki riwayat transaksi. Nonaktifkan produk sebagai gantinya."*
- *(Opsional, future enhancement)*: tambahkan kolom `is_active` pada `products` untuk soft-disable tanpa hapus. Tidak wajib di versi ini, dicatat sebagai rekomendasi.

### 7.9 Status & Indikator Stok (UI)

| Kondisi | Label |
|---|---|
| `stock = 0` | "Out of Stock" (merah) |
| `0 < stock <= low_stock_threshold` (default 10, dari `settings`) | "Low Stock" (kuning) |
| `stock > low_stock_threshold` | "Normal" (hijau) |

---

## 8. Autentikasi & Middleware

### 8.1 Pendekatan Autentikasi

- Gunakan **Laravel Breeze** (Blade stack) sebagai basis autentikasi: register, login, logout, forgot password, reset password.
- **Tidak membuat ulang** controller Auth custom dari nol — extend controller bawaan Breeze bila perlu menambah logika (misal cek `is_active` saat login, redirect berdasarkan `role`).
- Tambahan logika di `LoginController`/`AuthenticatedSessionController`:
  1. Setelah autentikasi berhasil, cek `is_active`. Jika `false` → logout otomatis & tampilkan pesan: *"Akun Anda tidak aktif. Hubungi admin."*
  2. Redirect berdasarkan `role`:
     - `admin` → `route('admin.dashboard')`
     - `nonadmin` → `route('nonadmin.dashboard')`

### 8.2 Middleware `CheckRole`

File: `app/Http/Middleware/CheckRole.php`

```php
public function handle($request, Closure $next, $role)
{
    if (!auth()->check()) {
        return redirect()->route('login');
    }
    // Admin boleh akses route mana pun (termasuk /nonadmin/*)
    if (auth()->user()->role === 'admin') {
        return $next($request);
    }
    // Non-admin hanya boleh akses route dengan role:nonadmin
    if (auth()->user()->role !== $role) {
        abort(403);
    }
    return $next($request);
}
```

Registrasi alias di `bootstrap/app.php` (Laravel 11):
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'role' => \App\Http\Middleware\CheckRole::class,
    ]);
})
```

---

## 9. Struktur Folder Project (Laravel)

### 9.1 Migrations (`database/migrations/`)
```
2026_01_01_000001_create_users_table.php          (tambah kolom role, is_active)
2026_01_01_000002_create_categories_table.php     (+ index on name)
2026_01_01_000003_create_products_table.php       (+ FK ON DELETE RESTRICT, index on category_id)
2026_01_01_000004_create_transactions_table.php   (+ is_credit, due_date, payment_status, indexes on transaction_date, type, user_id, is_credit, payment_status)
2026_01_01_000005_create_transaction_details_table.php (+ FK ON DELETE RESTRICT on product_id, indexes on transaction_id, product_id)
2026_01_01_000006_create_settings_table.php       (BARU)
```

### 9.2 Models (`app/Models/`)
```
User.php
Category.php
Product.php
Transaction.php
TransactionDetail.php
Setting.php   (BARU)
```

### 9.3 Controllers (`app/Http/Controllers/`)

**Auth** (extend Breeze):
```
Auth/AuthenticatedSessionController.php   (override redirect logic)
Auth/RegisteredUserController.php
Auth/PasswordResetLinkController.php
Auth/NewPasswordController.php
```

**Admin**:
```
Admin/DashboardController.php        (statistik global, chart data, top produk)
Admin/ProductController.php          (CRUD + export)
Admin/CategoryController.php         (CRUD + RESTRICT delete)
Admin/UserController.php             (CRUD + toggleStatus)
Admin/TransactionController.php      (CRUD + bulkDelete + export)
Admin/ReportController.php           (profitLoss, cashFlow, receivables, markAsPaid, export)
```

**NonAdmin**:
```
NonAdmin/DashboardController.php
NonAdmin/TransactionController.php   (create, store, history)
NonAdmin/ProductController.php       (index, show — read only)
NonAdmin/ReportController.php        (daily)
```

### 9.4 Services (`app/Services/`) — *lapisan logika bisnis*

```
StockService.php           (handle update/rollback stock, validasi stok, pessimistic locking)
TransactionService.php     (orchestrasi create/update/delete transaksi + delegasi ke StockService)
ReportService.php          (query laporan: profitLoss, cashFlow, receivables, daily)
ExportService.php          (generate CSV & PDF)
```

> **Aturan:** Controller hanya menerima request, memvalidasi via FormRequest, lalu memanggil Service. Service mengandung logika bisnis murni. Repository pattern tidak digunakan — cukup Eloquent langsung di Service.

### 9.5 FormRequest (`app/Http/Requests/`)

```
StoreUserRequest.php
UpdateUserRequest.php
StoreCategoryRequest.php
UpdateCategoryRequest.php
StoreProductRequest.php
UpdateProductRequest.php
StoreTransactionRequest.php
UpdateTransactionRequest.php
```

### 9.6 Middleware (`app/Http/Middleware/`)
```
CheckRole.php
```

### 9.7 Views (`resources/views/`)
```
layouts/
├── app.blade.php
└── auth.blade.php

admin/
├── dashboard.blade.php
├── transactions/{index, form}.blade.php
├── reports/{profit-loss, cash-flow, receivables}.blade.php
├── users/{index, form}.blade.php
├── products/{index, form}.blade.php
└── categories/{index, form}.blade.php

nonadmin/
├── dashboard.blade.php
├── transactions/{create, history}.blade.php
├── products/{index, show}.blade.php
└── reports/daily.blade.php

shared/
├── sidebar-admin.blade.php
├── sidebar-nonadmin.blade.php
├── alert.blade.php
└── product-card.blade.php

auth/
├── login.blade.php
└── register.blade.php
```

---

## 10. Routes Specification (`routes/web.php`)

| Group | Prefix | Middleware | Contoh Route |
|---|---|---|---|
| Guest | - | `guest` | `/login`, `/register` |
| Authenticated (umum) | - | `auth` | `/logout`, `/profile` |
| Admin | `/admin` | `['auth','role:admin']` | `/admin/dashboard`, `/admin/products`, `/admin/categories`, `/admin/users`, `/admin/transactions`, `/admin/reports/*` |
| Non-Admin | `/nonadmin` | `['auth','role:nonadmin']` | `/nonadmin/dashboard`, `/nonadmin/transactions/*`, `/nonadmin/products`, `/nonadmin/reports/daily` |
| Root `/` | - | `auth` | Redirect ke dashboard sesuai role |

### Contoh Definisi Route

```php
Route::middleware(['auth'])->group(function () {
    Route::get('/', fn() => auth()->user()->role === 'admin'
        ? redirect()->route('admin.dashboard')
        : redirect()->route('nonadmin.dashboard'));

    // ADMIN
    Route::middleware(['role:admin'])->prefix('admin')->name('admin.')->group(function () {
        Route::get('/dashboard', [Admin\DashboardController::class, 'index'])->name('dashboard');
        Route::resource('products', Admin\ProductController::class);
        Route::resource('categories', Admin\CategoryController::class);
        Route::resource('users', Admin\UserController::class);
        Route::patch('users/{user}/toggle-status', [Admin\UserController::class, 'toggleStatus'])->name('users.toggle');
        Route::resource('transactions', Admin\TransactionController::class);
        Route::delete('transactions/bulk', [Admin\TransactionController::class, 'bulkDelete'])->name('transactions.bulk-delete');
        Route::get('transactions/export/csv', [Admin\TransactionController::class, 'exportCsv'])->name('transactions.export-csv');
        Route::get('transactions/export/pdf', [Admin\TransactionController::class, 'exportPdf'])->name('transactions.export-pdf');
        Route::get('reports/profit-loss', [Admin\ReportController::class, 'profitLoss'])->name('reports.profit-loss');
        Route::get('reports/profit-loss/export/pdf', [Admin\ReportController::class, 'profitLossExportPdf'])->name('reports.profit-loss.export-pdf');
        Route::get('reports/cash-flow', [Admin\ReportController::class, 'cashFlow'])->name('reports.cash-flow');
        Route::get('reports/cash-flow/export/pdf', [Admin\ReportController::class, 'cashFlowExportPdf'])->name('reports.cash-flow.export-pdf');
        Route::get('reports/receivables', [Admin\ReportController::class, 'receivables'])->name('reports.receivables');
        Route::patch('reports/receivables/{transaction}/mark-paid', [Admin\ReportController::class, 'markAsPaid'])->name('reports.receivables.mark-paid');
    });

    // NON-ADMIN
    Route::middleware(['role:nonadmin'])->prefix('nonadmin')->name('nonadmin.')->group(function () {
        Route::get('/dashboard', [NonAdmin\DashboardController::class, 'index'])->name('dashboard');
        Route::get('/transactions/create', [NonAdmin\TransactionController::class, 'create'])->name('transactions.create');
        Route::post('/transactions', [NonAdmin\TransactionController::class, 'store'])->name('transactions.store');
        Route::get('/transactions/history', [NonAdmin\TransactionController::class, 'history'])->name('transactions.history');
        Route::get('/products', [NonAdmin\ProductController::class, 'index'])->name('products.index');
        Route::get('/products/{product}', [NonAdmin\ProductController::class, 'show'])->name('products.show');
        Route::get('/reports/daily', [NonAdmin\ReportController::class, 'daily'])->name('reports.daily');
    });
});
```

---

## 11. API Endpoints (untuk Postman Collection)

> Semua endpoint berada di bawah middleware `auth`. Endpoint dengan prefix `/admin` membutuhkan `role:admin`; prefix `/nonadmin` membutuhkan `role:nonadmin`.

### 11.1 Auth
| Method | Endpoint | Deskripsi |
|---|---|---|
| POST | `/register` | Registrasi user baru (default role: nonadmin) |
| POST | `/login` | Login, redirect sesuai role |
| POST | `/logout` | Logout |
| POST | `/forgot-password` | Kirim link reset password |
| POST | `/reset-password` | Reset password |

### 11.2 Admin — Users
| Method | Endpoint | Deskripsi |
|---|---|---|
| GET | `/admin/users` | List user non-admin (search, filter role) |
| GET | `/admin/users/create` | Form tambah user |
| POST | `/admin/users` | Simpan user baru |
| GET | `/admin/users/{id}/edit` | Form edit user |
| PUT | `/admin/users/{id}` | Update user |
| DELETE | `/admin/users/{id}` | Hapus user |
| PATCH | `/admin/users/{id}/toggle-status` | Aktifkan/nonaktifkan user |

### 11.3 Admin — Categories
| Method | Endpoint | Deskripsi |
|---|---|---|
| GET | `/admin/categories` | List kategori |
| GET | `/admin/categories/create` | Form tambah |
| POST | `/admin/categories` | Simpan kategori |
| GET | `/admin/categories/{id}/edit` | Form edit |
| PUT | `/admin/categories/{id}` | Update kategori |
| DELETE | `/admin/categories/{id}` | Hapus kategori (RESTRICT jika ada produk) |

### 11.4 Admin — Products
| Method | Endpoint | Deskripsi |
|---|---|---|
| GET | `/admin/products` | List produk (search, filter kategori) |
| GET | `/admin/products/create` | Form tambah produk |
| POST | `/admin/products` | Simpan produk |
| GET | `/admin/products/{id}/edit` | Form edit produk |
| PUT | `/admin/products/{id}` | Update produk |
| DELETE | `/admin/products/{id}` | Hapus produk (RESTRICT jika ada di transaction_details) |

### 11.5 Admin — Transactions
| Method | Endpoint | Deskripsi |
|---|---|---|
| GET | `/admin/transactions` | List semua transaksi (search, filter type/date/user) |
| GET | `/admin/transactions/create` | Form tambah transaksi |
| POST | `/admin/transactions` | Simpan transaksi (+ details, update stok) |
| GET | `/admin/transactions/{id}/edit` | Form edit |
| PUT | `/admin/transactions/{id}` | Update transaksi (rollback & re-apply stok) |
| DELETE | `/admin/transactions/{id}` | Hapus transaksi (rollback stok) |

### 11.6 Admin — Reports
| Method | Endpoint | Query Params | Deskripsi |
|---|---|---|---|
| GET | `/admin/reports/profit-loss` | `from, to, category_id?, user_id?` | Laporan laba/rugi |
| GET | `/admin/reports/cash-flow` | `from, to` | Laporan arus kas + running balance |
| GET | `/admin/reports/receivables` | `status? (unpaid/paid/overdue), user_id?` | Daftar piutang |
| PATCH | `/admin/reports/receivables/{id}/mark-paid` | - | Tandai piutang lunas |

### 11.7 Non-Admin — Transactions
| Method | Endpoint | Deskripsi |
|---|---|---|
| GET | `/nonadmin/transactions/create` | Form input transaksi harian |
| POST | `/nonadmin/transactions` | Simpan transaksi (user_id = auth user, is_credit dipaksa false) |
| GET | `/nonadmin/transactions/history` | Riwayat transaksi milik sendiri (search, filter) |

### 11.8 Non-Admin — Products & Reports
| Method | Endpoint | Deskripsi |
|---|---|---|
| GET | `/nonadmin/products` | List produk (read-only, search & filter kategori) |
| GET | `/nonadmin/products/{id}` | Detail produk |
| GET | `/nonadmin/reports/daily` | Laporan harian milik sendiri |

---

## 12. Spesifikasi Halaman (View)

### 12.1 Layouts
**`layouts/app.blade.php`**: navbar + sidebar dinamis (berdasarkan role) + `@yield('content')` + footer + komponen alert + CSRF token + toggle menu mobile.

**`layouts/auth.blade.php`**: layout terpusat tanpa sidebar, untuk halaman login/register.

### 12.2 Auth
- **`login.blade.php`**: input email, password, "remember me", link ke register & forgot password, pesan error.
- **`register.blade.php`**: input name, email, password, confirm password, link ke login.

### 12.3 Admin Pages

| Halaman | Komponen Kunci |
|---|---|
| `admin/dashboard.blade.php` | KPI cards (Total Revenue, Total Expense, Net Profit, Outstanding Receivables — periode bulan berjalan); chart pendapatan (line/bar); chart breakdown pengeluaran (pie); tabel 5-10 transaksi terbaru; top 5 produk terjual (berdasarkan `SUM(quantity)` pada transaksi income); quick-access buttons |
| `admin/transactions/index.blade.php` | Search, filter (type, date range, user), tabel (ID, Date, Type, Description, Amount, User, Actions), pagination, sort, bulk delete, export, summary total income/expense/net |
| `admin/transactions/form.blade.php` | Date picker, radio type (income/expense), textarea description, tabel detail produk (dinamis tambah/hapus baris via JS, modal pilih produk), checkbox "Transaksi Kredit (Piutang)" + due_date (muncul jika type=income & checkbox dicentang), amount auto-calc/readonly jika ada detail, manual input jika tidak ada detail |
| `admin/products/index.blade.php` | Search, filter kategori, tabel (ID, Name, Category, Price, Stock, Status indikator, Actions), pagination, export |
| `admin/products/form.blade.php` | Input name, price (decimal), stock (integer, readonly saat create karena stok awal = 0 atau bisa diisi sebagai stok awal), select category |
| `admin/categories/index.blade.php` | Search, tabel (ID, Name, Description, Product Count, Actions), delete dengan validasi RESTRICT |
| `admin/categories/form.blade.php` | Input name, textarea description |
| `admin/users/index.blade.php` | Search (name/email), filter role, tabel (ID, Name, Email, Role, Status, Actions), toggle active/inactive |
| `admin/users/form.blade.php` | Input name, email, password (required saat create, optional saat edit), select role (**admin / nonadmin saja**), checkbox is_active |
| `admin/reports/profit-loss.blade.php` | Date range picker, filter (category, user), Total Income/Expense/Net cards, profit margin %, tabel breakdown, chart income vs expense, export |
| `admin/reports/cash-flow.blade.php` | Date range picker, opening balance (dari `settings`), tabel (Date, Description, Inflow, Outflow, Running Balance), total inflow/outflow, closing balance, line chart |
| `admin/reports/receivables.blade.php` | Date range, filter status (unpaid/paid/overdue) & user, tabel (Transaction ID, Date, User, Amount, Due Date, Status badge), total piutang (paid vs unpaid), highlight overdue, tombol "Mark as Paid", aging report (0-30/30-60/60+ hari) |

### 12.4 Non-Admin Pages

| Halaman | Komponen Kunci |
|---|---|
| `nonadmin/dashboard.blade.php` | Greeting, ringkasan hari ini (income/expense/net), jumlah transaksi hari ini, transaksi terbaru milik sendiri, quick view produk, quick action buttons |
| `nonadmin/transactions/create.blade.php` | Sama seperti form admin, **tanpa** opsi `is_credit`/checkbox piutang (selalu `is_credit=false` untuk non-admin), `user_id` otomatis = auth user |
| `nonadmin/transactions/history.blade.php` | Search, filter (type, date range), tabel (ID, Date, Type, Description, Amount), pagination, view detail, grouping per tanggal |
| `nonadmin/products/index.blade.php` | Search, filter kategori, card/table read-only, indikator stok |
| `nonadmin/products/show.blade.php` | Detail produk (read-only) |
| `nonadmin/reports/daily.blade.php` | Date picker (default hari ini), summary income/expense/net, tabel transaksi hari itu, chart sederhana |

### 12.5 Shared Components
- **`sidebar-admin.blade.php`**: menu Dashboard, Products, Categories, Users, Transactions, Reports (sub-menu Profit-Loss/Cash-Flow/Receivables), Logout.
- **`sidebar-nonadmin.blade.php`**: menu Dashboard, Create Transaction, Transaction History, Products, Daily Report, Logout.
- **`alert.blade.php`**: alert success/error/warning/info dari `session('success')`, `session('error')`, dst, dengan tombol close.
- **`product-card.blade.php`**: gambar (opsional), nama, badge kategori, harga, indikator stok, tombol view/select.

---

## 13. Validasi Input (Form Request Rules)

| Form | Field | Rule |
|---|---|---|
| Register/User | `name` | `required|string|max:100` |
| | `email` | `required|email|max:100|unique:users,email` |
| | `password` | `required|min:8|confirmed` |
| | `role` | `required|in:admin,nonadmin` |
| Category | `name` | `required|string|max:100|unique:categories,name` |
| | `description` | `nullable|string` |
| Product | `name` | `required|string|max:100` |
| | `price` | `required|numeric|min:0` |
| | `stock` | `required|integer|min:0` |
| | `category_id` | `required|exists:categories,id` |
| Transaction | `transaction_date` | `required|date` |
| | `type` | `required|in:income,expense` |
| | `description` | `nullable|string|max:255` |
| | `amount` | `required_without:details|numeric|min:0` |
| | `details.*.product_id` | `required|exists:products,id` |
| | `details.*.quantity` | `required|integer|min:1` |
| | `details.*.price_per_unit` | `required|numeric|min:0` |
| | `is_credit` | `boolean` (hanya jika type=income; non-admin selalu false) |
| | `due_date` | `required_if:is_credit,true|date|after_or_equal:transaction_date` |

**Validasi tambahan (custom, di FormRequest/Controller):**
- Jika `type=income` dan ada `details`: setiap `details.*.quantity <= Product::find(product_id)->stock`.
- Jika `is_credit=true` tapi `type=expense` → tolak (422), pesan: *"Piutang hanya berlaku untuk transaksi pemasukan."*

---

## 14. Skenario Pengujian (QA Checklist)

| No | Skenario | Expected Result |
|---|---|---|
| 1 | Login sebagai admin | Redirect ke `/admin/dashboard`, akses semua menu admin |
| 2 | Login sebagai non-admin | Redirect ke `/nonadmin/dashboard`, sidebar hanya menu non-admin |
| 3 | Non-admin akses `/admin/dashboard` via URL langsung | HTTP 403 |
| 4 | Login dengan akun `is_active=false` | Login gagal, pesan "Akun tidak aktif" |
| 5 | Buat transaksi income dengan detail produk, stok cukup | Stok produk berkurang sesuai quantity, `amount` = total subtotal |
| 6 | Buat transaksi income dengan detail produk, stok tidak cukup | Ditolak, pesan error stok tidak cukup |
| 7 | Buat transaksi expense dengan detail produk (restock) | Stok produk bertambah |
| 8 | Buat transaksi expense tanpa detail (misal: bayar listrik) | Tidak ada perubahan stok, `amount` diinput manual |
| 9 | Edit transaksi (ubah quantity produk) | Stok di-rollback dan dihitung ulang dengan benar |
| 10 | Hapus transaksi | Stok di-rollback sesuai jenis transaksi |
| 11 | Buat transaksi income dengan `is_credit=true` tanpa `due_date` | Ditolak, validasi `due_date required` |
| 12 | Tandai piutang sebagai "Paid" | `payment_status` berubah jadi `paid`, tidak mengubah stok/amount |
| 13 | Piutang dengan `due_date < hari ini` & `payment_status=unpaid` | Tampil sebagai "Overdue" di laporan |
| 14 | Hitung laporan laba/rugi untuk periode tertentu | Total income/expense/net sesuai SUM data transaksi |
| 15 | Hitung laporan arus kas | Opening balance sesuai `settings`, running balance per baris benar, closing balance benar |
| 16 | Hapus kategori yang masih punya produk | Ditolak dengan pesan error |
| 17 | Hapus produk yang punya riwayat transaksi | Ditolak dengan pesan error |
| 18 | Non-admin coba edit/hapus transaksi (termasuk milik sendiri) | Tombol edit/hapus tidak tampil & endpoint menolak (403/405) |
| 19 | Non-admin coba akses transaksi history milik user lain | Hanya menampilkan transaksi `user_id = auth()->id()` |
| 20 | Validasi form: input kosong/format salah (price huruf, stock negatif) | Pesan validasi sesuai field |
| 21 | Tampilan responsif (mobile, tablet, desktop) | Layout menyesuaikan, sidebar collapsible |

---

## 15. Deliverables (Output Proyek)

1. **Repository GitHub**: source code lengkap (`github.com/RCS15/uas-pw`).
2. **File Rancangan ERD**: diagram skema database (Bagian 5).
3. **Postman Collection**: seluruh endpoint pada Bagian 11, lengkap dengan contoh request/response.
4. **Demo Aplikasi**: walkthrough fitur admin & non-admin.
5. **Submit ke GCR**: aplikasi terdeploy di Google Cloud Run.

---

## 16. Setup & Instalasi

### 16.1 Prasyarat

| Software | Keterangan |
|----------|------------|
| **XAMPP 8.2+** (atau **Laragon**) | Include PHP 8.2+, MySQL 8+, Apache, phpMyAdmin |
| **Composer 2.x** | Dependency manager PHP |
| **Node.js 20+ & npm** | Build frontend assets |
| **Git** | Version control |

> **Catatan:** Pastikan PHP dan MySQL dari XAMPP/Laragon terdaftar di `PATH` system agar bisa diakses dari terminal/cmd.

### 16.2 Langkah Instalasi

```bash
# ============== STEP 0: JALANKAN XAMPP ==============
# Buka XAMPP Control Panel
# Klik Start pada Apache (port 80)
# Klik Start pada MySQL (port 3306)
#
# (Jika pakai Laragon: buka Laragon → Start All)

# ============== STEP 1: CLONE REPOSITORY ==============
git clone https://github.com/24530008muhammad-lab/umkm-kelompok-05.git
cd umkm-kelompok-05/backend

# ============== STEP 2: INSTALL DEPENDENCY ==============
composer install

# ============== STEP 3: KONFIGURASI ENVIRONMENT ==============
cp .env.example .env
# Edit .env:
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=uas-pw
# DB_USERNAME=root
# DB_PASSWORD=               # kosongkan untuk XAMPP default
# SESSION_DRIVER=database
# MAIL_MAILER=log            # email ke file log

# ============== STEP 3b: BUAT DATABASE ==============
# Opsi A — via phpMyAdmin:
#    Buka http://localhost/phpmyadmin
#    Klik tab "Database"
#    Isi nama database: uas-pw
#    Pilih charset: utf8mb4_unicode_ci
#    Klik "Create"
#
# Opsi B — via terminal:
# mysql -u root -e "CREATE DATABASE IF NOT EXISTS uas-pw CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci"

# ============== STEP 4: GENERATE APP KEY ==============
php artisan key:generate

# ============== STEP 5: MIGRASI & SEEDER ==============
php artisan migrate --seed
# Seeder membuat:
# - 1 admin default:     admin@example.com / password
# - 2-3 non-admin:       kasir@example.com / password
# - 5-10 kategori
# - 10-20 produk
# - Settings: cash_opening_balance=0, cash_opening_date=today, low_stock_threshold=10
# - 15-20 transaksi contoh (income & expense, beberapa piutang)

# ============== STEP 6: BUILD FRONTEND ==============
npm install
npm run build

# ============== STEP 7: JALANKAN SERVER ==============
php artisan serve
# Buka http://127.0.0.1:8000
```

### 16.3 Verifikasi

Setelah instalasi, verifikasi dengan:
1. Buka `http://127.0.0.1:8000` → redirect ke login
2. Login sebagai admin: `admin@example.com` / `password`
3. Login sebagai non-admin: `kasir@example.com` / `password`
4. Cek dashboard admin → KPI cards, charts, transaksi terbaru
5. Cek akses non-admin ke `/admin/dashboard` → **403 Forbidden**

### 16.4 Catatan Penting

- **phpMyAdmin** tersedia di `http://localhost/phpmyadmin` (user: `root`, password: kosong)
- **php.ini** XAMPP ada di `C:\xampp\php\php.ini` — pastikan `extension=mysqli` dan `extension=pdo_mysql` aktif
- **MySQL** XAMPP menggunakan port 3306 (default). Jika ada aplikasi lain (seperti Laragon) yang juga pakai port 3306, matikan salah satu
- **Session driver** menggunakan database agar kompatibel dengan Cloud Run yang stateless
- **Email** untuk password reset dinonaktifkan — admin reset password via menu Users

---

## 17. Asumsi, Koreksi & Rekomendasi (Catatan Perubahan dari PRD Awal)

Dokumen ini melakukan beberapa **koreksi & penambahan** terhadap PRD awal agar logika sistem konsisten dan dapat diimplementasikan. Berikut ringkasannya:

1. **Tabel `transactions`** ditambahkan kolom `is_credit`, `due_date`, `payment_status` — wajib untuk mendukung fitur Laporan Piutang yang sudah direncanakan di halaman view tapi belum ada di skema awal.
2. **Tabel `settings` (baru)** ditambahkan untuk menyimpan saldo awal kas (`cash_opening_balance`) yang dibutuhkan Laporan Arus Kas, serta `low_stock_threshold` untuk indikator stok.
3. **Logika stok** diperjelas dengan 4 kombinasi (Bagian 7.1–7.2): income+detail = stok keluar, expense+detail = stok masuk (restock), transaksi tanpa detail = tidak memengaruhi stok. `transaction_details` bersifat **opsional** (boleh kosong) — ini mengubah skema relasi dari "wajib" menjadi "opsional".
4. **Role user** dibatasi tegas menjadi 2 (`admin`, `nonadmin`) — opsi "supervisor/etc" pada form user di PRD awal dihapus karena tidak konsisten dengan ENUM di database.
5. **Strategi autentikasi** disatukan: gunakan **Laravel Breeze** sebagai basis, controller Auth custom pada PRD awal **tidak dibuat dari nol** melainkan extend dari Breeze (menghindari duplikasi kode).
6. **Status "Overdue"** pada Piutang ditetapkan sebagai **nilai terhitung (computed)**, bukan disimpan di database, untuk menghindari kebutuhan job/scheduler tambahan.
7. **Aturan penghapusan** kategori & produk ditetapkan sebagai **RESTRICT** (block delete jika masih ada relasi), dengan pesan error yang jelas ke user.
8. **Kolom `is_active`** ditambahkan ke tabel `users` untuk mendukung fitur Activate/Deactivate pada `admin/users/index.blade.php` yang sudah direncanakan tapi belum ada di skema awal.
9. **Basis perhitungan Arus Kas** menggunakan pendekatan **akrual sederhana** (semua transaksi termasuk unpaid dihitung sebagai inflow/outflow). Keputusan final, tidak ada opsi kas murni di versi ini.
10. **Fitur "Send Reminder"** pada Laporan Piutang bersifat **opsional/UI-stub** — tidak wajib terhubung ke layanan email pada versi ini, untuk menjaga lingkup proyek tetap realistis sesuai timeline UAS.
11. **Race condition stok** ditangani dengan `DB::transaction()` + `lockForUpdate()` pessimistic locking + retry 3x.
12. **Export CSV & PDF** menggunakan library `barryvdh/laravel-dompdf` (PDF) dan streaming CSV built-in Laravel.
13. **Soft Delete** hanya pada model `Transaction` (audit trail). Model lain hard delete.
14. **Service layer** memisahkan logika bisnis dari controller: `StockService`, `TransactionService`, `ReportService`, `ExportService`.
15. **Password reset via email dinonaktifkan** — admin dapat reset password user via menu Users.

---

*Akhir dokumen — PRD Master v1.0*
