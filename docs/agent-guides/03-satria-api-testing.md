# Agent Guide: Satria — Backend Developer (API Testing)

## Tanggung Jawab
- Unit test endpoints (PHPUnit)
- Validasi form request testing
- Dokumentasi Postman Collection

## Source of Truth
- **API Endpoints:** `prd.md` Bagian 11 (endpoint lengkap)
- **Validation Rules:** `prd.md` Bagian 13
- **QA Scenarios:** `prd.md` Bagian 14 (21 skenario)
- **Routes:** `prd.md` Bagian 10
- **Business Rules:** `prd.md` Bagian 7

## Tugas Detail

### Phase 1: Setup Testing Environment
1. Pastikan PHPUnit terinstall (built-in Laravel)
2. Setup `.env.testing` atau gunakan SQLite in-memory untuk testing
3. Buat `DatabaseMigrations` trait di base TestCase

### Phase 2: Unit Tests — Feature Tests

Buat file test di `tests/Feature/`:

**1. Authentication Tests** (`AuthTest.php`)
- Test login admin → redirect ke `/admin/dashboard`
- Test login nonadmin → redirect ke `/nonadmin/dashboard`
- Test login dengan akun nonaktif (`is_active=false`) → gagal, pesan error
- Test register → user baru dengan role default nonadmin
- Test logout
- Test akses route tanpa login → redirect ke login

**2. Middleware Tests** (`MiddlewareTest.php`)
- Test nonadmin akses `/admin/dashboard` → HTTP 403
- Test admin akses `/nonadmin/dashboard` → OK (boleh)
- Test guest akses `/admin/*` → redirect login

**3. Admin — User Tests** (`AdminUserTest.php`)
- Test index (list users)
- Test create & store user valid
- Test store user dengan email duplikat → validasi error
- Test edit & update user
- Test delete user
- Test toggle status (aktif/nonaktif)
- Test admin tidak bisa menghapus diri sendiri (test case tambahan)

**4. Admin — Category Tests** (`AdminCategoryTest.php`)
- Test CRUD lengkap
- Test delete kategori yang masih punya produk → 422 error
- Test store dengan nama duplikat → validasi error

**5. Admin — Product Tests** (`AdminProductTest.php`)
- Test CRUD lengkap
- Test store dengan stock negatif → validasi error
- Test delete produk yang punya transaction_details → 422 error

**6. Admin — Transaction Tests** (`AdminTransactionTest.php`)
- Test CRUD lengkap
- Test store income + details (stok cukup) → stok berkurang
- Test store income + details (stok tidak cukup) → 422 error "Stok [nama] tidak cukup"
- Test store expense + details → stok bertambah (restock)
- Test store expense tanpa details → tidak ada efek stok, amount manual
- Test store income tanpa details → tidak ada efek stok
- Test update transaksi (ubah quantity) → stok dirollback & dihitung ulang
- Test delete transaksi → stok dirollback
- Test store income dengan `is_credit=true` tanpa `due_date` → validasi error
- Test store dengan `is_credit=true` tapi type=expense → 422

**7. Admin — Report Tests** (`AdminReportTest.php`)
- Test profit-loss dengan parameter from, to
- Test profit-loss dengan filter category & user
- Test cash-flow dengan parameter from, to (cek opening balance, running balance, closing balance)
- Test receivables list
- Test markAsPaid → payment_status jadi 'paid'
- Test piutang overdue (due_date < hari ini & unpaid) → label "Overdue"

**8. Non-Admin — Transaction Tests** (`NonAdminTransactionTest.php`)
- Test create transaction → user_id otomatis = auth user
- Test history → hanya menampilkan transaksi milik sendiri
- Test edit/hapus transaksi → 403 atau tombol tidak tampil
- Test history tidak menampilkan transaksi user lain

**9. Non-Admin — Product Tests** (`NonAdminProductTest.php`)
- Test index (read-only)
- Test show detail
- Test create/edit/delete → 403 atau route tidak ada

**10. Non-Admin — Report Tests** (`NonAdminReportTest.php`)
- Test daily report → hanya transaksi sendiri, hari tertentu

### Phase 3: Postman Collection
Buat Postman Collection dengan seluruh endpoint (PRD Bagian 11):

1. **Folder: Auth**
   - POST `/login` (admin & nonadmin)
   - POST `/register`
   - POST `/logout`
   - POST `/forgot-password`
   - POST `/reset-password`

2. **Folder: Admin**
   - Users: semua endpoint (GET, POST, PUT, DELETE, PATCH toggle-status)
   - Categories: semua endpoint
   - Products: semua endpoint
   - Transactions: semua endpoint
   - Reports: profit-loss, cash-flow, receivables, mark-paid

3. **Folder: Non-Admin**
   - Transactions: create, store, history
   - Products: index, show
   - Reports: daily

Setiap request harus memiliki:
- Contoh request body (JSON untuk API testing, atau form-data untuk web)
- Expected response
- Variabel environment (base_url, token/session)

### Phase 4: Validasi Form Request Testing
Buat test khusus untuk memvalidasi FormRequest rules (PRD Bagian 13):
- Field required
- Format valid (email, date, numeric)
- Unique constraint
- Custom validations (stok cukup, is_credit hanya untuk income)

## Checklist
- [ ] Semua auth test (login, register, logout, is_active check) lulus
- [ ] Semua middleware test (403 untuk akses salah role) lulus
- [ ] Admin CRUD test untuk semua modul lulus
- [ ] Non-admin access restriction test lulus
- [ ] Stock management test (auto update, rollback) lulus
- [ ] Report calculation test lulus
- [ ] Piutang/credit test lulus
- [ ] Postman Collection lengkap & siap pakai
- [ ] Coverage minimal: semua endpoint utama ter-test
