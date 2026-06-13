# Agent Guide: Fadzhil — Backend Developer (Model & Migration)

## Tanggung Jawab
- Migration (6 tabel)
- Model & relasi
- Autentikasi (extend Breeze)
- Middleware CheckRole
- Controller (Admin & NonAdmin)
- Seeder

## Source of Truth
- **Database Schema:** `prd.md` Bagian 5 (Skema Database)
- **Model & Relasi:** `prd.md` Bagian 6
- **Controllers:** `prd.md` Bagian 9.3
- **Business Rules:** `prd.md` Bagian 7
- **Middleware:** `prd.md` Bagian 8.2
- **Validation:** `prd.md` Bagian 13
- **Routes:** `prd.md` Bagian 10

## Tugas Detail

### Phase 1: Migration (`database/migrations/`)
Buat 6 file migration dengan urutan berikut:

1. **`create_users_table.php`** — tambah kolom `role` (ENUM: admin, nonadmin, default: nonadmin) dan `is_active` (BOOLEAN, default: true) dari Breeze default
2. **`create_categories_table.php`** — id, name (VARCHAR 100, UNIQUE), description (TEXT, nullable)
3. **`create_products_table.php`** — id, name, price (DECIMAL 15,2), stock (INT, default 0), category_id (FK → categories, RESTRICT)
4. **`create_transactions_table.php`** — id, transaction_date (DATE), type (ENUM: income, expense), description (nullable), amount (DECIMAL 15,2), user_id (FK → users), is_credit (BOOLEAN, default false), due_date (nullable), payment_status (ENUM: unpaid, paid, nullable)
5. **`create_transaction_details_table.php`** — id, transaction_id (FK → transactions, CASCADE DELETE), product_id (FK → products, RESTRICT), quantity (INT > 0), price_per_unit (DECIMAL 15,2), subtotal (DECIMAL 15,2)
6. **`create_settings_table.php`** — id, key (VARCHAR 100, UNIQUE), value (VARCHAR 255)

### Phase 2: Models (`app/Models/`)
Buat 6 model dengan relasi:

| Model | Relasi |
|-------|--------|
| `User.php` | `hasMany(Transaction::class)` |
| `Category.php` | `hasMany(Product::class)` |
| `Product.php` | `belongsTo(Category)`, `hasMany(TransactionDetail)` |
| `Transaction.php` | `belongsTo(User)`, `hasMany(TransactionDetail)` |
| `TransactionDetail.php` | `belongsTo(Transaction)`, `belongsTo(Product)` |
| `Setting.php` | Key-value helper: `Setting::get('key')` static accessor |

**Transaction Model — Accessor tambahan:**
- `payment_status_label` — computed: jika `payment_status == 'unpaid' && due_date < hari_ini` → "Overdue", else tampilkan status asli
- `is_overdue` — boolean: `payment_status == 'unpaid' && due_date < today()`

### Phase 3: Middleware (`app/Http/Middleware/CheckRole.php`)
```php
public function handle($request, Closure $next, $role)
{
    if (!auth()->check()) {
        return redirect()->route('login');
    }
    if (auth()->user()->role !== $role) {
        abort(403);
    }
    return $next($request);
}
```
Registrasi alias di `Kernel.php`: `'role' => \App\Http\Middleware\CheckRole::class`

### Phase 4: Auth — Extend Breeze
Override `AuthenticatedSessionController`:
1. Cek `is_active` setelah login sukses → jika false, logout & flash error
2. Redirect by role: admin → `/admin/dashboard`, nonadmin → `/nonadmin/dashboard`

### Phase 5: Controllers

**Admin Controllers:**
- `Admin/DashboardController` — statistik global (total revenue, expense, net, outstanding receivables per bulan berjalan, chart data, top 5 produk, transaksi terbaru)
- `Admin/UserController` — CRUD users (non-admin), toggleStatus
- `Admin/CategoryController` — CRUD categories, dengan RESTRICT delete
- `Admin/ProductController` — CRUD products
- `Admin/TransactionController` — CRUD transactions + stock management logic
- `Admin/ReportController` — profitLoss, cashFlow, receivables, markAsPaid

**Non-Admin Controllers:**
- `NonAdmin/DashboardController` — greeting, ringkasan hari ini (income/expense/net milik sendiri)
- `NonAdmin/TransactionController` — create, store, history (hanya milik sendiri, is_credit=false)
- `NonAdmin/ProductController` — index, show (read-only)
- `NonAdmin/ReportController` — daily (laporan harian milik sendiri)

### Phase 6: Stock Management Logic (di TransactionController)

Implementasi di `store`/`update`/`destroy`:

| Aksi | Efek Stok |
|------|-----------|
| Store income + details | `stock -= quantity` per produk |
| Store expense + details | `stock += quantity` per produk |
| Store income/expense tanpa details | Tidak ada efek stok |
| Update transaksi | Rollback stok lama → apply stok baru (dalam DB transaction) |
| Delete transaksi | Rollback stok (kebalikan dari saat create) |

Validasi stok cukup sebelum income transaction: cek `quantity <= product.stock` untuk setiap detail.

### Phase 7: Seeder (`database/seeders/`)

Buat seeder:
1. **UserSeeder**: 1 admin default + 2-3 nonadmin
2. **CategorySeeder**: 5-10 kategori contoh
3. **ProductSeeder**: 10-20 produk dengan kategori
4. **SettingSeeder**: `cash_opening_balance=0`, `cash_opening_date`, `low_stock_threshold=10`
5. **TransactionSeeder**: 15-20 transaksi contoh (income & expense, beberapa dengan piutang)
6. Panggil semua dari `DatabaseSeeder.php`

### Phase 8: Routes (`routes/web.php`)
Implementasi sesuai PRD Bagian 10:
- Guest routes (login, register)
- Authenticated root redirect (by role)
- Admin group: prefix `/admin`, middleware `['auth','role:admin']`
- NonAdmin group: prefix `/nonadmin`, middleware `['auth','role:nonadmin']`

## Validation (FormRequest)
Buat FormRequest untuk setiap modul sesuai PRD Bagian 13.

## Checklist
- [ ] Semua migration bisa migrate & rollback
- [ ] Semua model dengan relasi benar
- [ ] Middleware CheckRole berfungsi (403 untuk akses salah role)
- [ ] Login cek is_active
- [ ] Redirect by role setelah login
- [ ] Admin CRUD semua modul berfungsi
- [ ] Non-Admin hanya bisa create & read transaksi sendiri
- [ ] Stock management benar (auto update, rollback)
- [ ] Seeder menghasilkan data contoh yang cukup
- [ ] Semua route terdaftar & middleware terpasang
