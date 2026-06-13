# Agent Guide: Fadzhil — Backend Developer (Model & Migration)

## Tanggung Jawab
- Migration (6 tabel + indexes)
- Model & relasi (6 model + casts + accessors + scopes)
- Soft Delete on Transaction
- Autentikasi (extend Breeze — nonaktifkan email verification & password reset)
- Middleware CheckRole (allow admin on nonadmin routes)
- Controller (Admin & NonAdmin) — panggil Service layer
- Service layer (StockService, TransactionService, ReportService, ExportService)
- FormRequest (8 classes)
- Seeder (admin + sample data + settings)
- Routes (+ bulk delete, export)
- Event/Listener untuk stock changes

## Source of Truth
- **Database Schema:** `prd.md` Bagian 5 (Skema Database + indexes)
- **Model & Relasi:** `prd.md` Bagian 6 (lengkap dengan casts, scopes, accessors)
- **Controllers:** `prd.md` Bagian 9.3
- **Services:** `prd.md` Bagian 9.4
- **Business Rules:** `prd.md` Bagian 7 (termasuk race condition locking)
- **Middleware:** `prd.md` Bagian 8.2 (allow admin on nonadmin)
- **Validation:** `prd.md` Bagian 13
- **Routes:** `prd.md` Bagian 10 (termasuk bulk delete & export)
- **FormRequest:** `prd.md` Bagian 9.5

## Tugas Detail

### Phase 0: Setup Laravel 11 + Breeze
```bash
cd backend
composer create-project laravel/laravel .
composer require laravel/breeze --dev
php artisan breeze:install blade
# Nonaktifkan email verification: skip --email
npm install && npm run build
```

**Prasyarat:** Pastikan XAMPP (atau Laragon) sudah berjalan dengan Apache & MySQL.

**Konfigurasi AppServiceProvider:**
```php
// app/Providers/AppServiceProvider.php
use Illuminate\Support\Facades\Schema;
public function boot(): void {
    Schema::defaultStringLength(191);
}
```

**.env config (XAMPP / Laragon):**
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=uas-pw
DB_USERNAME=root
DB_PASSWORD=
SESSION_DRIVER=database
MAIL_MAILER=log       # Email ke log (password reset disabled via route)
```

> **Catatan:** Database `uas-pw` harus dibuat terlebih dahulu via phpMyAdmin (`http://localhost/phpmyadmin`) atau via CLI: `mysql -u root -e "CREATE DATABASE IF NOT EXISTS uas-pw CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci"`

### Phase 1: Migration (`database/migrations/`)

Buat 6 file migration (urutan penting karena FK dependencies):

1. **`2026_01_01_000001_create_users_table.php`**
   - Modify default Breeze migration: tambah `role` ENUM('admin','nonadmin') default 'nonadmin', `is_active` BOOLEAN default true
   - Retain: `email_verified_at` nullable, `remember_token`
   - Index: none needed (PK + email UNIQUE sudah cukup)

2. **`2026_01_01_000002_create_categories_table.php`**
   - `id` BIGINT UNSIGNED PK AUTO_INCREMENT
   - `name` VARCHAR(100) NOT NULL UNIQUE
   - `description` TEXT NULLABLE
   - Index: `$table->index('name')`

3. **`2026_01_01_000003_create_products_table.php`**
   - `id`, `name` VARCHAR(100), `price` DECIMAL(15,2), `stock` INT DEFAULT 0
   - `category_id` BIGINT UNSIGNED NOT NULL, FK → categories(id) **ON DELETE RESTRICT**
   - Index: `$table->index('category_id')`
   - Gunakan `foreignId(...)->constrained()->restrictOnDelete()` untuk FK

4. **`2026_01_01_000004_create_transactions_table.php`**
   - `id`, `transaction_date` DATE, `type` ENUM('income','expense'), `description` VARCHAR(255) nullable
   - `amount` DECIMAL(15,2), `user_id` BIGINT UNSIGNED FK → users(id)
   - `is_credit` BOOLEAN DEFAULT FALSE, `due_date` DATE nullable, `payment_status` ENUM('unpaid','paid') nullable
   - **Soft Delete**: tambahkan `softDeletes()` column
   - Indexes: `$table->index('transaction_date')`, `$table->index('type')`, `$table->index('user_id')`, `$table->index('is_credit')`, `$table->index('payment_status')`

5. **`2026_01_01_000005_create_transaction_details_table.php`**
   - `id`, `transaction_id` BIGINT UNSIGNED FK → transactions(id) **ON DELETE CASCADE**
   - `product_id` BIGINT UNSIGNED FK → products(id) **ON DELETE RESTRICT**
   - `quantity` INT > 0, `price_per_unit` DECIMAL(15,2), `subtotal` DECIMAL(15,2)
   - Indexes: `$table->index('transaction_id')`, `$table->index('product_id')`
   - Untuk `restrictOnDelete()` gunakan: `foreignId('product_id')->constrained()->restrictOnDelete()`

6. **`2026_01_01_000006_create_settings_table.php`**
   - `id`, `key` VARCHAR(100) UNIQUE NOT NULL, `value` TEXT NOT NULL

### Phase 2: Models (`app/Models/`)

Buat 6 model dengan casts, accessors, scopes:

**User.php**
```php
protected $casts = ['is_active' => 'boolean'];
// Relasi
public function transactions(): HasMany { return $this->hasMany(Transaction::class); }
// Scopes
public function scopeActive($query) { return $query->where('is_active', true); }
```

**Category.php**
```php
// Relasi
public function products(): HasMany { return $this->hasMany(Product::class); }
// Accessors
public function getProductCountAttribute(): int { return $this->products()->count(); }
// Appends (agar ikut serialisasi)
protected $appends = ['product_count'];
```

**Product.php**
```php
protected $casts = ['price' => 'decimal:2', 'stock' => 'integer'];
// Relasi
public function category(): BelongsTo { return $this->belongsTo(Category::class); }
public function transactionDetails(): HasMany { return $this->hasMany(TransactionDetail::class); }
// Accessors
public function getStockStatusAttribute(): string {
    $threshold = Setting::get('low_stock_threshold') ?? 10;
    if ($this->stock <= 0) return 'out_of_stock';
    if ($this->stock <= (int)$threshold) return 'low_stock';
    return 'normal';
}
// Scopes
public function scopeLowStock($query, ?int $threshold = null) {
    $threshold = $threshold ?? (Setting::get('low_stock_threshold') ?? 10);
    return $query->where('stock', '>', 0)->where('stock', '<=', $threshold);
}
public function scopeInStock($query) { return $query->where('stock', '>', 0); }
public function scopeOutOfStock($query) { return $query->where('stock', '<=', 0); }
```

**Transaction.php**
```php
use Illuminate\Database\Eloquent\SoftDeletes;
protected $casts = [
    'amount' => 'decimal:2',
    'transaction_date' => 'date',
    'due_date' => 'date',
    'is_credit' => 'boolean',
];
// Relasi
public function user(): BelongsTo { return $this->belongsTo(User::class); }
public function details(): HasMany { return $this->hasMany(TransactionDetail::class); }
// Accessors
public function getPaymentStatusLabelAttribute(): string {
    if ($this->payment_status === 'unpaid' && $this->due_date?->isPast()) return 'Overdue';
    return $this->payment_status === 'paid' ? 'Paid' : ($this->payment_status === 'unpaid' ? 'Unpaid' : '-');
}
public function getIsOverdueAttribute(): bool {
    return $this->payment_status === 'unpaid' && $this->due_date?->isPast();
}
// Scopes
public function scopeType($query, string $type) { return $query->where('type', $type); }
public function scopeDateRange($query, $from, $to) {
    return $query->whereBetween('transaction_date', [$from, $to]);
}
public function scopeCredit($query) { return $query->where('is_credit', true); }
```

**TransactionDetail.php**
```php
protected $casts = ['price_per_unit' => 'decimal:2', 'subtotal' => 'decimal:2'];
// Relasi
public function transaction(): BelongsTo { return $this->belongsTo(Transaction::class); }
public function product(): BelongsTo { return $this->belongsTo(Product::class); }
```

**Setting.php**
```php
// Key-value helper — tidak perlu relasi
public static function get(string $key, $default = null): ?string {
    return static::where('key', $key)->first()?->value ?? $default;
}
public static function set(string $key, string $value): void {
    static::updateOrCreate(['key' => $key], ['value' => $value]);
}
```

### Phase 3: Event/Listener untuk Stock (opsional tapi direkomendasikan)

```bash
php artisan make:event TransactionCreated
php artisan make:event TransactionUpdated
php artisan make:event TransactionDeleted
php artisan make:listener UpdateStockOnTransactionCreated --event=TransactionCreated
php artisan make:listener UpdateStockOnTransactionUpdated --event=TransactionUpdated
php artisan make:listener UpdateStockOnTransactionDeleted --event=TransactionDeleted
```

Atau approach lebih sederhana: panggil `StockService` langsung dari controller. Pilih salah satu — jika waktu terbatas, gunakan direct call di controller.

### Phase 4: Middleware (`app/Http/Middleware/CheckRole.php`)

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

Registrasi di `bootstrap/app.php`:
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'role' => \App\Http\Middleware\CheckRole::class,
    ]);
})
```

### Phase 5: Auth — Extend Breeze

Override `AuthenticatedSessionController`:
1. Cek `is_active` setelah login sukses → jika false, logout & flash error: *"Akun Anda tidak aktif. Hubungi admin."*
2. Redirect by role: admin → `route('admin.dashboard')`, nonadmin → `route('nonadmin.dashboard')`

**Nonaktifkan password reset via email:** hapus atau komen route forgot-password dan reset-password di `routes/auth.php`. Admin cukup reset password user via menu Users.

### Phase 6: Services (`app/Services/`)

**StockService.php**
```php
class StockService {
    // Panggil lockForUpdate SEBELUM operasi write
    public function validateStock(Collection $details): void {
        $productIds = $details->pluck('product_id');
        $products = Product::whereIn('id', $productIds)->lockForUpdate()->get()->keyBy('id');
        foreach ($details as $detail) {
            $product = $products->get($detail['product_id']);
            if (!$product || $product->stock < $detail['quantity']) {
                throw new InsufficientStockException($product->name, $product->stock ?? 0);
            }
        }
    }

    public function updateStock(Transaction $transaction, string $action = 'store'): void {
        DB::transaction(function () use ($transaction, $action) {
            $details = $transaction->details;
            if ($details->isEmpty()) return;

            $productIds = $details->pluck('product_id');
            $products = Product::whereIn('id', $productIds)->lockForUpdate()->get()->keyBy('id');

            foreach ($details as $detail) {
                $product = $products->get($detail->product_id);
                if (!$product) continue;

                $qty = $detail->quantity;
                if ($action === 'store' || $action === 'rollback') {
                    // store income = stock--
                    // store expense = stock++
                    if ($transaction->type === 'income') {
                        $product->stock -= ($action === 'store' ? $qty : -$qty);
                    } else {
                        $product->stock += ($action === 'store' ? $qty : -$qty);
                    }
                }
                // (logika untuk update lebih kompleks — lihat TransactionController)
                $product->save();
            }
        }, 3); // retry 3x
    }
}
```

**TransactionService.php** — orchestrasi: validasi → simpan → panggil StockService
**ReportService.php** — query laporan murni (profitLoss, cashFlow, receivables, daily)
**ExportService.php** — generate CSV stream & PDF via Dompdf

### Phase 7: Controllers

Setiap controller menerima request → validasi via FormRequest → panggil Service → kembalikan response.

**Admin Controllers:**
- `Admin/DashboardController` — total revenue/expense/net bulan berjalan, outstanding receivables, chart data labels+values, top 5 produk, 5-10 transaksi terbaru
- `Admin/UserController` — CRUD + toggleStatus (soft delete? user hard delete)
- `Admin/CategoryController` — CRUD, delete dengan validasi `$category->products()->exists()`
- `Admin/ProductController` — CRUD + exportCsv + exportPdf
- `Admin/TransactionController` — CRUD + bulkDelete + exportCsv + exportPdf. Panggil StockService untuk setiap operasi yang mempengaruhi stok. Param `details` diterima sebagai array di request.
- `Admin/ReportController` — profitLoss, cashFlow, receivables, markAsPaid + export PDF masing-masing

**Non-Admin Controllers:**
- `NonAdmin/DashboardController` — greeting, ringkasan hari ini (income/expense/net milik sendiri), jumlah transaksi hari ini, transaksi terbaru, quick view low stock products
- `NonAdmin/TransactionController` — create, store (is_credit dipaksa false, user_id = auth), history (hanya milik sendiri, search, filter, pagination)
- `NonAdmin/ProductController` — index (read-only, search, filter kategori), show
- `NonAdmin/ReportController` — daily (default today, hanya transaksi auth user)

### Phase 8: FormRequest (`app/Http/Requests/`)

Buat 8 class FormRequest sesuai PRD Bagian 13:

```php
// StoreTransactionRequest.php — validasi custom:
public function rules(): array { /* PRD Bagian 13 */ }
public function withValidator($validator): void {
    $validator->after(function ($validator) {
        // Jika type=income dan ada details: validasi stok cukup
        // Jika is_credit=true tapi type=expense: tolak
    });
}
```

Daftar: `StoreUserRequest`, `UpdateUserRequest`, `StoreCategoryRequest`, `UpdateCategoryRequest`, `StoreProductRequest`, `UpdateProductRequest`, `StoreTransactionRequest`, `UpdateTransactionRequest`.

### Phase 9: Seeder (`database/seeders/`)

**ProductionSeeder** (bisa jalan di production):
```php
// 1 admin: admin@example.com / password
// Settings: cash_opening_balance=0, cash_opening_date=today, low_stock_threshold=10
```

**DevelopmentSeeder** (extends ProductionSeeder + sample data):
```php
// 2-3 nonadmin: kasir@example.com, staff@example.com
// 5-10 categories
// 10-20 products
// 15-20 transactions (income & expense, beberapa dengan piutang)
```

Panggil dari `DatabaseSeeder.php`:
```php
$this->call([
    ProductionSeeder::class,
    DevelopmentSeeder::class, // comment this di production
]);
```

### Phase 10: Routes (`routes/web.php`)

Implementasi sesuai PRD Bagian 10, tambahan:

```php
Route::middleware(['role:admin'])->prefix('admin')->name('admin.')->group(function () {
    // ... semua route resource ...
    // Bulk delete
    Route::delete('transactions/bulk', [Admin\TransactionController::class, 'bulkDelete'])->name('transactions.bulk-delete');
    // Export
    Route::get('transactions/export/csv', [Admin\TransactionController::class, 'exportCsv'])->name('transactions.export-csv');
    Route::get('transactions/export/pdf', [Admin\TransactionController::class, 'exportPdf'])->name('transactions.export-pdf');
    Route::get('products/export/csv', [Admin\ProductController::class, 'exportCsv'])->name('products.export-csv');
    // Report export
    Route::get('reports/profit-loss/export/pdf', [Admin\ReportController::class, 'profitLossExportPdf'])->name('reports.profit-loss.export-pdf');
    Route::get('reports/cash-flow/export/pdf', [Admin\ReportController::class, 'cashFlowExportPdf'])->name('reports.cash-flow.export-pdf');
});
```

### Phase 11: Model Factories (`database/factories/`)

Buat factory untuk testing:
- `UserFactory` — state `admin()`, `nonadmin()`, `inactive()`
- `CategoryFactory` — random name
- `ProductFactory` — belongs to category, random price/stock
- `TransactionFactory` — state `income()`, `expense()`, `credit()`
- `TransactionDetailFactory` — belongs to transaction & product
- `SettingFactory` — predefined key/value

## Checklist
- [ ] Semua migration bisa migrate:fresh & rollback tanpa error
- [ ] Semua model dengan casts, accessors, scopes, soft delete benar
- [ ] Setting helper static accessor berfungsi
- [ ] Middleware CheckRole: admin akses /nonadmin/* OK, nonadmin akses /admin/* → 403
- [ ] Login: cek is_active, redirect by role
- [ ] Password reset via email dinonaktifkan
- [ ] Admin CRUD semua modul + bulkDelete + export
- [ ] Non-Admin hanya bisa create & read transaksi sendiri
- [ ] StockService: validasi stok, update, rollback, pessimistic locking
- [ ] ReportService: profitLoss, cashFlow, receivables, daily akurat
- [ ] ExportService: CSV & PDF generate benar
- [ ] Seeder: ProductionSeeder (admin + settings) & DevelopmentSeeder (sample data)
- [ ] Semua route terdaftar, middleware terpasang
- [ ] FormRequest validasi berfungsi (custom validasi stok, is_credit)
- [ ] Model Factory siap untuk testing
