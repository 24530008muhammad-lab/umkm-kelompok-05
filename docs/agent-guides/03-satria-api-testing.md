# Agent Guide: Satria — Backend Developer (API Testing)

## Tanggung Jawab
- Unit test endpoints (PHPUnit) — 10 file test
- Validasi form request testing
- Dokumentasi Postman Collection + environment

## Source of Truth
- **API Endpoints:** `prd.md` Bagian 11 (endpoint lengkap)
- **Validation Rules:** `prd.md` Bagian 13
- **QA Scenarios:** `prd.md` Bagian 14 (21 skenario)
- **Routes:** `prd.md` Bagian 10 (termasuk bulk delete & export)
- **Business Rules:** `prd.md` Bagian 7
- **Factories:** `database/factories/` (6 factory sudah dibuat Fadzhil)

## Tugas Detail

### Phase 1: Setup Testing Environment

**1. Konfigurasi `phpunit.xml`**
Pastikan `phpunit.xml` sudah mengatur environment variable untuk testing:
```xml
<php>
    <env name="APP_ENV" value="testing"/>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
    <env name="CACHE_DRIVER" value="array"/>
    <env name="SESSION_DRIVER" value="array"/>
    <env name="QUEUE_CONNECTION" value="sync"/>
    <env name="MAIL_MAILER" value="array"/>
</php>
```

**2. Base TestCase Setup**
```php
// tests/TestCase.php
abstract class TestCase extends BaseTestCase
{
    use CreatesApplication, RefreshDatabase, DatabaseMigrations;

    protected function setUp(): void
    {
        parent::setUp();
        $this->seed(ProductionSeeder::class); // admin + settings
        // Setup user untuk test
        $this->admin = User::factory()->admin()->create();
        $this->nonadmin = User::factory()->nonadmin()->create();
    }
}
```

**3. .env.testing (opsional, fallback)**
```
DB_CONNECTION=sqlite
DB_DATABASE=:memory:
```

**4. Coverage target** — konfigurasi di `phpunit.xml`:
```xml
<coverage includeUncoveredFiles="true">
    <include>
        <directory suffix=".php">app/Services</directory>
        <directory suffix=".php">app/Http/Controllers</directory>
        <directory suffix=".php">app/Models</directory>
        <directory suffix=".php">app/Http/Middleware</directory>
    </include>
</coverage>
```
Target minimum: **70%** overall, **90%** untuk Services (StockService, TransactionService, ReportService).

### Phase 2: Unit Tests — Feature Tests

Buat 10 file test di `tests/Feature/`. Setiap file butuh tambahan edge case test.

**1. Authentication Tests** (`AuthTest.php`)
- Test login admin → redirect ke `/admin/dashboard`
- Test login nonadmin → redirect ke `/nonadmin/dashboard`
- Test login dengan akun nonaktif (`is_active=false`) → gagal, pesan error
- Test register → user baru dengan role default nonadmin
- Test logout
- Test akses route tanpa login → redirect ke login
- **Edge case:** login dengan email tidak terdaftar → validasi error
- **Edge case:** login dengan password salah → validasi error

**2. Middleware Tests** (`MiddlewareTest.php`)
- Test nonadmin akses `/admin/dashboard` → HTTP 403
- Test admin akses `/nonadmin/dashboard` → **OK** (boleh, sesuai aturan)
- Test guest akses `/admin/*` → redirect login
- **Edge case:** admin akses `/admin/*` → OK

**3. Admin — User Tests** (`AdminUserTest.php`)
- Test index (list users)
- Test create & store user valid
- Test store user dengan email duplikat → validasi error
- Test edit & update user
- Test delete user
- Test toggle status (aktif/nonaktif)
- Test admin tidak bisa menghapus diri sendiri
- **Edge case:** store user dengan password < 8 karakter → error
- **Edge case:** store user dengan role invalid → error

**4. Admin — Category Tests** (`AdminCategoryTest.php`)
- Test CRUD lengkap
- Test delete kategori yang masih punya produk → 422 error
- Test store dengan nama duplikat → validasi error
- **Edge case:** store dengan nama kosong → error
- **Edge case:** update kategori tidak ada → 404

**5. Admin — Product Tests** (`AdminProductTest.php`)
- Test CRUD lengkap
- Test store dengan stock negatif → validasi error
- Test delete produk yang punya transaction_details → 422 error
- **Edge case:** price diisi huruf → validasi error
- **Edge case:** price negatif → error
- **Edge case:** stock 0 → valid

**6. Admin — Transaction Tests** (`AdminTransactionTest.php`)
- Test store income + details (stok cukup) → stok berkurang, amount = sum subtotal
- Test store income + details (stok tidak cukup) → 422 error "Stok [nama] tidak cukup. Tersedia: X"
- Test store expense + details → stok bertambah (restock)
- Test store expense tanpa details → tidak ada efek stok, amount manual
- Test store income tanpa details → tidak ada efek stok
- Test update transaksi (ubah quantity) → stok dirollback & dihitung ulang
- Test delete transaksi → stok dirollback
- Test store income dengan `is_credit=true` tanpa `due_date` → validasi error
- Test store dengan `is_credit=true` tapi type=expense → 422
- **Test bulk delete** → beberapa transaksi terhapus, stok dirollback
- **Test export CSV** → response header Content-Type text/csv
- **Test export PDF** → response header application/pdf
- **Edge case:** quantity = 0 → error
- **Edge case:** product_id tidak ada → error
- **Edge case:** transaction_date di masa depan → valid (atau depend on requirement)
- **Edge case:** amount = 0 pada transaksi tanpa detail → valid
- **Edge case:** due_date < transaction_date → error

**7. Admin — Report Tests** (`AdminReportTest.php`)
- Test profit-loss dengan parameter from, to → angka sesuai SUM
- Test profit-loss dengan filter category & user
- Test profit-loss dengan filter kombinasi
- Test cash-flow: opening balance = settings, running balance per baris, closing balance benar
- Test cash-flow: tidak ada transaksi di rentang → opening = closing
- Test receivables list → hanya transaksi is_credit=true
- Test markAsPaid → payment_status jadi 'paid', tidak ubah stok/amount
- Test piutang overdue (due_date < hari ini & unpaid) → label "Overdue"
- **Test report export PDF** → response valid
- **Edge case:** from > to → 422 atau handle dengan swap
- **Edge case:** tidak ada data di rentang → array kosong, bukan error

**8. Non-Admin — Transaction Tests** (`NonAdminTransactionTest.php`)
- Test create transaction → user_id otomatis = auth user
- Test create transaction → is_credit dipaksa false
- Test history → hanya menampilkan transaksi milik sendiri
- Test edit/hapus transaksi → 403
- Test history tidak menampilkan transaksi user lain
- **Edge case:** non-admin membuat transaksi dengan parameter is_credit=true → tetap false

**9. Non-Admin — Product Tests** (`NonAdminProductTest.php`)
- Test index (read-only, search, filter)
- Test show detail
- Test create/edit/delete → 403

**10. Non-Admin — Report Tests** (`NonAdminReportTest.php`)
- Test daily report → hanya transaksi sendiri, tanggal tertentu
- Test daily report default tanggal = hari ini
- **Edge case:** tidak ada transaksi di tanggal tersebut → data kosong

### Phase 3: Concurrent Stock Update Test

Buat test khusus untuk race condition:
```php
/** @test */
public function concurrent_income_transactions_prevent_overselling()
{
    $product = Product::factory()->create(['stock' => 5]);
    $user = $this->admin;

    // Simulasi 3 request simultan, masing-masing beli 2 unit (total = 6 > 5)
    $responses = ParallelHttp::fake([
        fn() => $this->actingAs($user)->post('/admin/transactions', [
            'type' => 'income',
            'transaction_date' => now()->format('Y-m-d'),
            'details' => [['product_id' => $product->id, 'quantity' => 2, 'price_per_unit' => 10000]]
        ]),
        fn() => $this->actingAs($user)->post('/admin/transactions', [
            'type' => 'income',
            'transaction_date' => now()->format('Y-m-d'),
            'details' => [['product_id' => $product->id, 'quantity' => 2, 'price_per_unit' => 10000]]
        ]),
        fn() => $this->actingAs($user)->post('/admin/transactions', [
            'type' => 'income',
            'transaction_date' => now()->format('Y-m-d'),
            'details' => [['product_id' => $product->id, 'quantity' => 2, 'price_per_unit' => 10000]]
        ]),
    ]);

    // Maksimal 2 berhasil (2+2=4 <= 5), 1 gagal (stok tidak cukup)
    $successCount = collect($responses)->filter(fn($r) => $r->status() === 302)->count();
    $failCount = collect($responses)->filter(fn($r) => $r->status() === 422)->count();
    $this->assertGreaterThanOrEqual(1, $failCount);
    $this->assertLessThanOrEqual(2, $successCount);
    // Stok akhir: 5 - (2 * successCount)
    $this->assertEquals(5 - (2 * $successCount), $product->fresh()->stock);
}
```

### Phase 4: Postman Collection

Buat Postman Collection (`docs/umkm-api-collection.postman_collection.json`) dengan:

1. **Environment variables:**
   - `base_url` = `http://localhost:8000`
   - `admin_email` = `admin@example.com`
   - `admin_password` = `password`
   - `nonadmin_email` = `kasir@example.com`
   - `nonadmin_password` = `password`
   - `csrf_token` (diambil dari cookie)

2. **Folder: Auth** — login admin, login nonadmin, register, logout
3. **Folder: Admin**
   - Users: GET/POST/PUT/DELETE + toggle-status
   - Categories: GET/POST/PUT/DELETE
   - Products: GET/POST/PUT/DELETE + export CSV
   - Transactions: GET/POST/PUT/DELETE + bulk-delete + export CSV/PDF
   - Reports: profit-loss, cash-flow, receivables, mark-paid, export PDF
4. **Folder: Non-Admin**
   - Transactions: create, store, history
   - Products: index, show
   - Reports: daily

Simpan file Postman Collection di `docs/postman/umkm-api-collection.json`.

### Phase 5: Validasi Form Request Testing

Buat test khusus untuk FormRequest rules:
- Field required: semua required field dikosongkan → error
- Format valid: email format salah, date format salah, numeric diisi huruf
- Unique constraint: email duplikat, nama kategori duplikat
- Custom validations: stok cukup (income), is_credit hanya untuk income, due_date required_if is_credit, due_date >= transaction_date

## Checklist
- [ ] phpunit.xml terkonfigurasi dengan SQLite in-memory
- [ ] .env.testing siap
- [ ] Semua auth test lulus (login admin/nonadmin, is_active, register, logout)
- [ ] Semua middleware test lulus (403 nonadmin ke admin, admin ke nonadmin OK, guest redirect)
- [ ] Admin CRUD test semua modul lulus (users, categories, products, transactions)
- [ ] Stock management test lulus (auto update, rollback, bulk delete)
- [ ] Concurrent stock test lulus (race condition handling)
- [ ] Report calculation test lulus (profitLoss, cashFlow, receivables, daily)
- [ ] Piutang/credit test lulus
- [ ] Non-admin access restriction test lulus
- [ ] Export test lulus (CSV header, PDF header)
- [ ] Edge case test lulus (boundary dates, zero stock, quantity=0, dll)
- [ ] Postman Collection lengkap (5 folder, semua endpoint, environment variables)
- [ ] Coverage minimal 70% overall, 90% Services
