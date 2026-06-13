# Agent Guide: Shidqi — Frontend Developer

## Tanggung Jawab
- Layout Blade (app, auth)
- Seluruh view per role (Admin: 12+ halaman, Non-Admin: 6 halaman)
- Shared components (sidebar, alert, product-card, stock-badge, confirm-dialog)
- Integrasi Chart.js v4 (dashboard, reports)
- Export button UI (CSV & PDF)
- Responsive design (mobile, tablet, desktop)
- Error pages (403, 404)
- Confirmation dialogs untuk delete & bulk delete
- Empty/Loading/Error states

## Source of Truth
- **View Specification:** `prd.md` Bagian 12 (layout, halaman & komponen kunci)
- **Stok UI Indikator:** `prd.md` Bagian 7.9
- **Shared Components:** `prd.md` Bagian 12.5
- **Routes:** `prd.md` Bagian 10 (untuk nama route)
- **Business Rules:** `prd.md` Bagian 7

## Tugas Detail

### Phase 0: Setup Frontend Assets

```bash
npm install chart.js       # Chart.js v4 untuk grafik
npm install @tailwindcss/forms  # Form styling Tailwind
```

Di `resources/js/app.js`:
```js
import './bootstrap';
import Chart from 'chart.js';

// Register Chart.js components
import { CategoryScale, LinearScale, BarController, LineController, PieController,
         BarElement, LineElement, PointElement, ArcElement, Title, Tooltip, Legend } from 'chart.js';
Chart.register(CategoryScale, LinearScale, BarController, LineController, PieController,
               BarElement, LineElement, PointElement, ArcElement, Title, Tooltip, Legend);
window.Chart = Chart;
```

Di `tailwind.config.js`:
```js
module.exports = {
    content: ['./resources/**/*.blade.php'],
    theme: { extend: {} },
    plugins: [require('@tailwindcss/forms')],
};
```

### Phase 1: Layouts

**1. `layouts/auth.blade.php`**
- Layout terpusat tanpa sidebar
- Untuk halaman login, register (forgot-password & reset-password dihapus dari route)
- Background gradient sederhana (Tailwind: `bg-gradient-to-br from-blue-50 to-indigo-100`)
- Card putih di tengah (`max-w-md mx-auto mt-10`)
- Flash message component untuk error

**2. `layouts/app.blade.php`**
```blade
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>@yield('title', 'UMKM Keuangan')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-gray-100">
    <div class="flex h-screen">
        @includeFirst(['shared.sidebar-' . auth()->user()->role, 'shared.sidebar-admin'])
        <div class="flex-1 flex flex-col overflow-hidden">
            @include('shared.alert')
            <main class="flex-1 overflow-y-auto p-6">
                @yield('content')
            </main>
        </div>
    </div>
    @stack('scripts')
</body>
</html>
```
- Sidebar dinamis berdasarkan role: `@includeFirst(['shared.sidebar-' . auth()->user()->role, ...])`
- Toggle menu mobile: hamburger button untuk show/hide sidebar (overlay di mobile)
- Navbar (atas) dengan user info + logout button

### Phase 2: Shared Components

**1. `shared/sidebar-admin.blade.php`**
- Brand/logo area (nama aplikasi)
- Menu items (active state berdasarkan route name)
- Reports dropdown (submenu Profit-Loss, Cash-Flow, Receivables)
- Logout button (form POST)

**2. `shared/sidebar-nonadmin.blade.php`**
- Brand/logo area
- Menu: Dashboard, Create Transaction, Transaction History, Products, Daily Report
- Logout button

**3. `shared/alert.blade.php`**
```blade
@if(session('success'))
    <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded relative" role="alert">
        <span>{{ session('success') }}</span>
        <button type="button" class="absolute top-0 right-0 px-4 py-3" onclick="this.parentElement.remove()">&times;</button>
    </div>
@endif
{{-- sama untuk error, warning, info --}}
```

**4. `shared/product-card.blade.php`**
```blade
<div class="bg-white rounded-lg shadow p-4">
    <h3 class="font-semibold">{{ $product->name }}</h3>
    <span class="text-xs bg-gray-200 rounded px-2 py-1">{{ $product->category->name }}</span>
    <p class="text-lg font-bold mt-2">Rp {{ number_format($product->price, 0, ',', '.') }}</p>
    <x-stock-badge :stock="$product->stock" />
    <button class="mt-2 bg-blue-500 text-white px-4 py-1 rounded">Pilih</button>
</div>
```

**5. `shared/stock-badge.blade.php`** (komponen Blade atau inline partial)
```blade
@props(['stock'])
@php
    $threshold = \App\Models\Setting::get('low_stock_threshold') ?? 10;
    if ($stock <= 0) { $label = 'Out of Stock'; $class = 'bg-red-100 text-red-800'; }
    elseif ($stock <= $threshold) { $label = 'Low Stock'; $class = 'bg-yellow-100 text-yellow-800'; }
    else { $label = 'Normal'; $class = 'bg-green-100 text-green-800'; }
@endphp
<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium {{ $class }}">
    {{ $label }}
</span>
```

**6. `shared/confirm-dialog.blade.php`** — modal konfirmasi untuk delete
```blade
<div id="confirmModal" class="hidden fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center">
    <div class="bg-white rounded-lg p-6 max-w-sm">
        <h3 class="text-lg font-semibold">Konfirmasi Hapus</h3>
        <p class="mt-2 text-gray-600">Apakah Anda yakin ingin menghapus data ini?</p>
        <div class="mt-4 flex justify-end space-x-2">
            <button onclick="closeModal()" class="px-4 py-2 bg-gray-300 rounded">Batal</button>
            <button id="confirmDelete" class="px-4 py-2 bg-red-500 text-white rounded">Hapus</button>
        </div>
    </div>
</div>
```

### Phase 3: Stok UI Indikator

Buat partial/component `stock-badge` yang menerima `$stock` dan menampilkan:
- `stock = 0` → badge merah "Out of Stock"
- `0 < stock <= threshold` → badge kuning "Low Stock"
- `stock > threshold` → badge hijau "Normal"

Gunakan di semua tempat yang menampilkan stok (products/index, products/show, transaction form, product-card, dll).

### Phase 4: Admin Views

**1. `admin/dashboard.blade.php`**
- KPI cards (4 cards): Total Revenue, Total Expense, Net Profit, Outstanding Receivables
  - Styling: card putih dengan icon, nilai besar (format Rp), perubahan (↑/↓)
- Chart pendapatan (line chart): sum income per hari/bulan
  - Data dari controller: `data-labels`, `data-values` di `<canvas>`
  ```blade
  <canvas id="incomeChart" data-labels="{{ json_encode($chartLabels) }}" data-values="{{ json_encode($chartValues) }}"></canvas>
  ```
- Chart breakdown pengeluaran (pie chart): expense per kategori
- Tabel 5-10 transaksi terbaru (kolom: Date, Type, Description, Amount)
- Top 5 produk terjual (nama, total quantity terjual)
- Quick-access buttons: ke setiap modul admin

**2. `admin/transactions/index.blade.php`**
- Search bar (cari berdasarkan description)
- Filter: type (dropdown: all/income/expense), date range (from, to date inputs), user (dropdown)
- Tabel: ID, Date, Type (badge warna: green=income, red=expense), Description, Amount (Rp), User, Actions (edit, delete)
- **Bulk delete**: checkbox di setiap baris + "Delete Selected" button + confirm dialog
- **Export buttons**: "Export CSV" dan "Export PDF" di atas tabel
- **Pagination**: 15 per page, dengan links
- **Default sort**: `transaction_date DESC`
- **Empty state**: jika tidak ada data → "Belum ada transaksi" dengan icon
- Summary bar di bawah: Total Income, Total Expense, Net

**3. `admin/transactions/form.blade.php`** (create & edit)
- Date picker: `<input type="date" name="transaction_date">` (native HTML5)
- Radio button type: income / expense
- Textarea description
- **Dynamic product details table** (lihat detail di bawah)
- Checkbox is_credit + due_date: muncul jika type=income & checkbox dicentang
- Amount: readonly (auto-calc) jika ada detail produk, manual input jika tidak ada
- Submit button

**Desain modal pilih produk:**
```
Tombol "Tambah Produk" → muncul modal
Modal berisi:
  - Search input (cari produk by name)
  - Filter kategori dropdown
  - Tabel: Name | Category | Price | Stock | [Pilih]
  - Setiap klik [Pilih] → tambah baris ke tabel detail, tutup modal
Setelah produk dipilih, tabel detail menampilkan:
  - Product Name | Quantity [input number] | Price/Unit [input, auto-filled] | Subtotal (auto-calc) | [Hapus]
  - Bisa tambah produk lagi dengan klik "Tambah Produk"
```

**4. `admin/products/index.blade.php`**
- Search (by name), filter kategori dropdown
- Tabel: ID, Name, Category, Price (Rp), Stock, Status (stock-badge), Actions (edit, delete)
- **Export button** (CSV)
- Pagination (20 per page)
- **Confirmation dialog** untuk delete
- **Empty state**: "Belum ada produk"

**5. `admin/products/form.blade.php`**
- Input name, price (type="number" step="0.01"), stock (type="number" min="0"), select category
- Client-side validation: required, min, type

**6. `admin/categories/index.blade.php`**
- Search (by name)
- Tabel: ID, Name, Description, Product Count, Actions (edit, delete)
- **Confirmation dialog** untuk delete (dengan pesan risiko jika ada produk)
- **Empty state**

**7. `admin/categories/form.blade.php`**
- Input name, textarea description

**8. `admin/users/index.blade.php`**
- Search (name/email), filter role dropdown
- Tabel: ID, Name, Email, Role (badge), Status (Active=green/Inactive=red badge), Actions
- Toggle Active/Inactive via PATCH button (AJAX atau form)
- **Confirmation dialog** untuk toggle & delete

**9. `admin/users/form.blade.php`**
- Input name, email (type="email"), password (required create, optional edit), select role (admin/nonadmin), checkbox is_active
- Client-side validation

**10. Admin Report Views**
- **`admin/reports/profit-loss.blade.php`**
  - Date range picker (from, to) + filter category + filter user + [Filter] button
  - KPI cards: Total Income (green), Total Expense (red), Net Profit/Loss (blue), Profit Margin % (yellow)
  - Tabel breakdown transaksi per tanggal
  - Chart: bar chart income vs expense per periode
  - **Export PDF button**
- **`admin/reports/cash-flow.blade.php`**
  - Date range picker + [Filter] button
  - Opening Balance display
  - Tabel: Date, Description, Inflow, Outflow, Running Balance
  - Total Inflow, Total Outflow, Closing Balance
  - Line chart running balance
  - **Export PDF button**
- **`admin/reports/receivables.blade.php`**
  - Filter: status (unpaid/paid/overdue/all), user + [Filter] button
  - Tabel: Transaction ID, Date, User, Amount, Due Date, Status badge (paid=green, unpaid=yellow, overdue=red)
  - Total piutang paid vs unpaid
  - **Highlight overdue** (row merah atau badge merah)
  - Tombol "Mark as Paid" (PATCH via form)
  - Aging report: grouping 0-30 hari, 30-60 hari, 60+ hari

### Phase 5: Non-Admin Views

**1. `nonadmin/dashboard.blade.php`**
- Greeting: "Halo, {{ auth()->user()->name }}"
- Ringkasan hari ini: Income, Expense, Net (dari daily report)
- Jumlah transaksi hari ini
- Tabel transaksi terbaru milik sendiri
- Quick view: produk dengan stok rendah
- Quick action buttons: "Buat Transaksi", "Riwayat Transaksi"

**2. `nonadmin/transactions/create.blade.php`**
- Sama seperti form admin, **tanpa** opsi is_credit/piutang
- user_id tidak ditampilkan (otomatis)
- Dynamic product details table (sama seperti admin)

**3. `nonadmin/transactions/history.blade.php`**
- Search + filter (type, date range)
- Tabel: ID, Date, Type, Description, Amount
- Pagination (15 per page)
- Grouping per tanggal
- **Tidak ada** tombol edit/hapus

**4. `nonadmin/products/index.blade.php`**
- Search, filter kategori
- Tampilan card grid (product-card component) atau tabel read-only
- Stock badge

**5. `nonadmin/products/show.blade.php`**
- Detail produk: name, category, price, stock, stock status badge

**6. `nonadmin/reports/daily.blade.php`**
- Date picker (default: hari ini)
- Summary cards: Income, Expense, Net
- Tabel transaksi hari itu
- Chart sederhana

### Phase 6: Responsive Design

| Breakpoint | Behavior |
|------------|----------|
| < 768px (mobile) | Sidebar hidden, hamburger button, overlay sidebar saat diklik. Tabel horizontal scroll (`overflow-x-auto`). Cards 1 kolom. Forms full-width. |
| 768-1024px (tablet) | Sidebar collapsed (icon only) atau mini. Cards 2 kolom. |
| > 1024px (desktop) | Sidebar full. Cards grid: 4 kolom untuk KPI, 3 kolom untuk produk. |

Gunakan Tailwind utility classes:
```blade
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
    {{-- KPI cards --}}
</div>
<div class="overflow-x-auto">
    <table>{{-- tabel dengan min-width agar scroll --}}</table>
</div>
```

### Phase 7: Error Pages

Buat `resources/views/errors/403.blade.php` dan `resources/views/errors/404.blade.php`:
- Extend `layouts.auth.blade.php` (tanpa sidebar)
- Kode error besar (403 / 404)
- Pesan user-friendly: "403 — Akses Ditolak" / "404 — Halaman Tidak Ditemukan"
- Tombol "Kembali ke Dashboard"

### Phase 8: Integrasi dengan Backend

Pastikan semua form:
```blade
<form method="POST" action="{{ route('...') }}">
    @csrf
    @method('PUT') {{-- untuk update --}}
    {{-- input fields --}}
    @error('field_name')
        <p class="text-red-500 text-sm mt-1">{{ $message }}</p>
    @enderror
</form>
```

**Client-side validation:**
- Gunakan HTML5 attributes: `required`, `min`, `max`, `type="number"`, `type="email"`
- Field error styling: `border-red-500` pada input yang error
- Tampilkan `$errors->first('field')` di bawah setiap input

**AJAX untuk Chart.js:**
```javascript
document.addEventListener('DOMContentLoaded', function() {
    const ctx = document.getElementById('incomeChart');
    if (ctx) {
        const labels = JSON.parse(ctx.dataset.labels);
        const values = JSON.parse(ctx.dataset.values);
        new Chart(ctx, {
            type: 'line',
            data: { labels, datasets: [{ label: 'Pendapatan', data: values }] },
            options: { responsive: true }
        });
    }
});
```

### Phase 9: Loading & Empty States

- **Loading state**: saat form submit, disable tombol submit dan tampilkan spinner/teks "Menyimpan..."
- **Empty state**: gunakan `@forelse` dengan `@empty` untuk tabel:
```blade
@forelse($transactions as $transaction)
    <tr>...</tr>
@empty
    <tr><td colspan="6" class="text-center py-8 text-gray-500">
        <svg class="mx-auto h-12 w-12 text-gray-400" ...>...</svg>
        <p>Belum ada transaksi</p>
    </td></tr>
@endforelse
```

## Checklist
- [ ] Layout auth & app selesai dengan sidebar dinamis (by role)
- [ ] Alert component: success/error/warning/info + auto dismiss
- [ ] Stock badge: Out of Stock (merah), Low Stock (kuning), Normal (hijau)
- [ ] Sidebar admin: Dashboard, Products, Categories, Users, Transactions, Reports (dropdown)
- [ ] Sidebar nonadmin: Dashboard, Create Transaction, History, Products, Daily Report
- [ ] Admin dashboard: KPI cards, 2 chart (line income, pie expense), top 5 produk, transaksi terbaru
- [ ] Admin transactions/index: search, filter, tabel, bulk delete checkbox, export CSV/PDF, pagination, summary
- [ ] Admin transactions/form: date picker, radio type, dynamic product table (modal), is_credit checkbox, auto-calc amount
- [ ] Product selection modal: search, filter, table, pilih button → tambah baris
- [ ] Admin products/index: search, filter, stock badge, export, pagination
- [ ] Admin categories/index: search, product count, delete confirm
- [ ] Admin users/index: search, filter role, toggle active/inactive, confirm dialog
- [ ] Admin reports: profit-loss, cash-flow, receivables (chart + export PDF)
- [ ] Non-admin dashboard: greeting, today summary, recent transactions, low stock view
- [ ] Non-admin transaction create: form tanpa is_credit, dynamic product table
- [ ] Non-admin transaction history: search, filter, pagination, no edit/delete
- [ ] Non-admin products: card grid read-only
- [ ] Non-admin daily report: date picker, summary, tabel, chart
- [ ] Responsive: sidebar collapsible mobile, scroll table, grid responsive
- [ ] Error pages 403/404 dengan Tailwind
- [ ] Confirmation dialog untuk semua delete action
- [ ] Empty state (@forelse) di semua tabel
- [ ] Form validation: error messages + border styling + client-side HTML5
- [ ] Chart.js terintegrasi: data dari data attribute, init di DOMContentLoaded
