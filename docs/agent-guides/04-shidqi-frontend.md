# Agent Guide: Shidqi — Frontend Developer

## Tanggung Jawab
- Layout (app, auth)
- View per role (Admin & Non-Admin)
- Integrasi Tailwind CSS
- Komponen shared (sidebar, alert, product-card)
- Responsive design

## Source of Truth
- **View Specification:** `prd.md` Bagian 12 (layout, halaman & komponen kunci)
- **Stok UI Indikator:** `prd.md` Bagian 7.9
- **Shared Components:** `prd.md` Bagian 12.5

## Tugas Detail

### Phase 1: Layouts

**1. `layouts/auth.blade.php`**
- Layout terpusat tanpa sidebar
- Untuk halaman login, register, forgot-password, reset-password
- Background sederhana, card di tengah

**2. `layouts/app.blade.php`**
- Navbar (atas)
- Sidebar dinamis (berdasarkan role)
- `@yield('content')` — konten utama
- Footer
- Komponen `shared.alert` (session flash messages)
- CSRF token meta
- Toggle menu mobile (hamburger)

### Phase 2: Shared Components

**1. `shared/sidebar-admin.blade.php`**
Menu:
- Dashboard (`route('admin.dashboard')`)
- Products (`route('admin.products.index')`)
- Categories (`route('admin.categories.index')`)
- Users (`route('admin.users.index')`)
- Transactions (`route('admin.transactions.index')`)
- Reports (dropdown):
  - Profit & Loss (`route('admin.reports.profit-loss')`)
  - Cash Flow (`route('admin.reports.cash-flow')`)
  - Receivables (`route('admin.reports.receivables')`)
- Logout

**2. `shared/sidebar-nonadmin.blade.php`**
Menu:
- Dashboard (`route('nonadmin.dashboard')`)
- Create Transaction (`route('nonadmin.transactions.create')`)
- Transaction History (`route('nonadmin.transactions.history')`)
- Products (`route('nonadmin.products.index')`)
- Daily Report (`route('nonadmin.reports.daily')`)
- Logout

**3. `shared/alert.blade.php`**
- Support session keys: `success`, `error`, `warning`, `info`
- Warna: success=hijau, error=merah, warning=kuning, info= biru
- Tombol close (X)
- Auto-dismiss setelah 5 detik (opsional, via JS)

**4. `shared/product-card.blade.php`**
Untuk tampilan card produk (non-admin):
- Nama produk
- Badge kategori
- Harga (format Rupiah)
- Indikator stok (warna badge)
- Tombol View/Select

### Phase 3: Stok UI Indikator (PRD 7.9)

| Kondisi | Label | Warna |
|---------|-------|-------|
| `stock = 0` | "Out of Stock" | Merah |
| `0 < stock <= low_stock_threshold` | "Low Stock" | Kuning |
| `stock > low_stock_threshold` | "Normal" | Hijau |

Buat helper/component `stock-badge` yang menerima parameter `$stock` dan menampilkan badge sesuai kondisi.

### Phase 4: Admin Views

**1. `admin/dashboard.blade.php`**
- KPI Cards: Total Revenue (month), Total Expense, Net Profit, Outstanding Receivables
- Chart pendapatan (line/bar) — bisa gunakan Chart.js sederhana
- Chart breakdown pengeluaran (pie)
- Tabel 5-10 transaksi terbaru
- Top 5 produk terjual
- Quick-access buttons ke setiap modul

**2. `admin/transactions/index.blade.php`**
- Search bar
- Filter: type (income/expense), date range, user
- Tabel: ID, Date, Type, Description, Amount (Rupiah format), User, Actions (edit, delete)
- Pagination
- Summary bar: Total Income, Total Expense, Net

**3. `admin/transactions/form.blade.php`** (create & edit)
- Date picker (`transaction_date`)
- Radio button type (income/expense)
- Textarea description
- Dynamic product details table (JS untuk tambah/hapus baris, modal pilih produk)
- Checkbox "Transaksi Kredit (Piutang)" + due_date input (muncul jika type=income & checkbox dicentang)
- Amount: auto-calc (readonly jika ada detail produk) / manual input (jika tanpa detail)
- Submit button

**4. `admin/products/index.blade.php`**
- Search, filter kategori dropdown
- Tabel: ID, Name, Category, Price, Stock, Status (badge stok), Actions
- Pagination, export button (opsional)

**5. `admin/products/form.blade.php`**
- Input name, price (decimal), stock (integer), select category

**6. `admin/categories/index.blade.php`**
- Search, tabel: ID, Name, Description, Product Count, Actions (edit, delete)

**7. `admin/categories/form.blade.php`**
- Input name, textarea description

**8. `admin/users/index.blade.php`**
- Search (name/email), filter role
- Tabel: ID, Name, Email, Role, Status (Active/Inactive badge), Actions
- Toggle Active/Inactive button

**9. `admin/users/form.blade.php`**
- Input name, email, password (required create, optional edit), select role (admin/nonadmin), checkbox is_active

**10. Admin Report Views**
- `admin/reports/profit-loss.blade.php`: Date range picker, filter (category, user), KPI cards (Income/Expense/Net), profit margin %, tabel breakdown, chart
- `admin/reports/cash-flow.blade.php`: Date range picker, opening balance, tabel (Date, Description, Inflow, Outflow, Running Balance), total inflow/outflow, closing balance, line chart
- `admin/reports/receivables.blade.php`: Date range, filter status (unpaid/paid/overdue) & user, tabel (Transaction ID, Date, User, Amount, Due Date, Status badge), total piutang paid vs unpaid, highlight overdue, tombol "Mark as Paid", aging report

### Phase 5: Non-Admin Views

**1. `nonadmin/dashboard.blade.php`**
- Greeting user
- Ringkasan hari ini: Income, Expense, Net
- Jumlah transaksi hari ini
- Transaksi terbaru milik sendiri
- Quick view produk (stok rendah)
- Quick action buttons

**2. `nonadmin/transactions/create.blade.php`**
- Sama seperti form admin, **tanpa** opsi `is_credit` (selalu false)
- `user_id` tidak ditampilkan (otomatis auth user)

**3. `nonadmin/transactions/history.blade.php`**
- Search, filter (type, date range)
- Tabel: ID, Date, Type, Description, Amount
- Pagination, grouping per tanggal

**4. `nonadmin/products/index.blade.php`**
- Search, filter kategori (card atau tabel read-only)
- Indikator stok badge

**5. `nonadmin/products/show.blade.php`**
- Detail produk lengkap (read-only)

**6. `nonadmin/reports/daily.blade.php`**
- Date picker (default: hari ini)
- Summary: Income, Expense, Net
- Tabel transaksi hari itu
- Chart sederhana (optional)

### Phase 6: Responsive Design
- Sidebar collapsible di mobile (hamburger menu)
- Tabel horizontal scroll di mobile
- Cards grid responsive (1 col mobile, 2-3 col tablet, 4 col desktop)
- Gunakan utility class Tailwind: `grid`, `grid-cols-1`, `md:grid-cols-2`, `lg:grid-cols-3`, dll

### Phase 7: Integrasi dengan Backend
Pastikan semua form:
- Mengirim CSRF token (`@csrf`)
- Menggunakan method spoofing untuk PUT/PATCH/DELETE (`@method('PUT')`)
- Menampilkan error validasi (`$errors->first('field')`)
- Menampilkan flash message dari session

## Checklist
- [ ] Layout auth & app selesai dengan sidebar dinamis
- [ ] Alert component berfungsi (success, error, warning, info)
- [ ] Sidebar admin & nonadmin sesuai menu
- [ ] Stock badge indicator (Out of Stock/Low Stock/Normal)
- [ ] Admin dashboard dengan KPI cards, charts, tabel
- [ ] Admin CRUD views: transactions, products, categories, users
- [ ] Admin report views: profit-loss, cash-flow, receivables
- [ ] Non-admin dashboard
- [ ] Non-admin transaction create & history
- [ ] Non-admin products (read-only)
- [ ] Non-admin daily report
- [ ] Responsive design (mobile, tablet, desktop)
- [ ] Semua form terintegrasi dengan backend (CSRF, validation errors, flash messages)
