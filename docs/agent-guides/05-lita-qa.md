# Agent Guide: Lita — Quality Assurance (QA)

## Tanggung Jawab
- Test plan & test case (40+ skenario)
- Pengujian manual (seluruh fitur)
- Laporan bug
- Regression testing

## Source of Truth
- **QA Checklist:** `prd.md` Bagian 14 (21 skenario dasar)
- **Role & Hak Akses:** `prd.md` Bagian 4 (RBAC Matrix)
- **Business Rules:** `prd.md` Bagian 7 (stok, piutang, arus kas)
- **Validation:** `prd.md` Bagian 13
- **View Specification:** `prd.md` Bagian 12

## Tugas Detail

### Phase 1: Create Test Plan
Buat dokumen `docs/qa-test-plan.md`:
- **Scope:** Semua fitur di PRD Bagian 2.2 (In-Scope)
- **Lingkungan:** Localhost (php artisan serve), lalu staging di GCR
- **Tools:** Browser (Chrome/Firefox/Edge), Developer Tools, Postman
- **Prioritasi:** Critical (auth, transaksi, stok) > High (laporan, master data) > Medium (UI, export) > Low (responsive)

### Phase 2: Test Case Documentation

**Test Case Template:**
```
| ID | Modul | Skenario | Langkah | Expected Result | Status |
|----|-------|----------|---------|-----------------|--------|
```

**Credentials untuk testing:**
- Admin: `admin@example.com` / `password`
- Non-Admin: `kasir@example.com` / `password`
- Non-Admin 2: `staff@example.com` / `password`

#### TC-AUTH (Autentikasi) — 8 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-AUTH-01 | Login sebagai admin | Redirect ke `/admin/dashboard`, akses semua menu admin |
| TC-AUTH-02 | Login sebagai non-admin (kasir) | Redirect ke `/nonadmin/dashboard`, sidebar hanya menu non-admin |
| TC-AUTH-03 | Non-admin akses `/admin/dashboard` via URL langsung | HTTP 403 Forbidden |
| TC-AUTH-04 | Login dengan akun `is_active=false` | Login gagal, pesan "Akun Anda tidak aktif. Hubungi admin." |
| TC-AUTH-05 | Register user baru | User terdaftar dengan role default nonadmin |
| TC-AUTH-06 | Akses route tanpa login | Redirect ke halaman login |
| TC-AUTH-07 | Admin akses `/nonadmin/dashboard` | **OK** (boleh) — verifikasi admin bisa akses halaman nonadmin |
| TC-AUTH-08 | Login dengan email tidak terdaftar | Pesan error "Email tidak terdaftar" |

#### TC-TRX (Transaksi & Stok) — 9 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-TRX-01 | Buat transaksi income dengan detail produk, stok cukup | Stok produk berkurang sesuai quantity, amount = total subtotal |
| TC-TRX-02 | Buat transaksi income dengan detail produk, stok tidak cukup | Ditolak, pesan error "Stok [nama produk] tidak cukup. Tersedia: X" |
| TC-TRX-03 | Buat transaksi expense dengan detail produk (restock) | Stok produk bertambah sesuai quantity |
| TC-TRX-04 | Buat transaksi expense tanpa detail (misal: bayar listrik) | Tidak ada perubahan stok, amount diinput manual |
| TC-TRX-05 | Edit transaksi (ubah quantity produk) | Stok di-rollback dan dihitung ulang dengan benar |
| TC-TRX-06 | Hapus transaksi income dengan detail | Stok di-rollback (stok bertambah kembali) |
| TC-TRX-07 | Hapus transaksi expense dengan detail | Stok di-rollback (stok berkurang kembali) |
| TC-TRX-08 | **NEW** — Bulk delete transaksi | Pilih 3 transaksi → klik "Delete Selected" → confirm → semua terhapus, stok dirollback |
| TC-TRX-09 | **NEW** — Export CSV transaksi | Klik "Export CSV" → download file CSV berisi data transaksi |

#### TC-CREDIT (Piutang) — 7 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-CREDIT-01 | Buat transaksi income dengan `is_credit=true` + `due_date` | Transaksi tersimpan, payment_status='unpaid' |
| TC-CREDIT-02 | Buat transaksi income dengan `is_credit=true` tanpa `due_date` | Ditolak, validasi "due_date wajib diisi" |
| TC-CREDIT-03 | Tandai piutang sebagai "Paid" | payment_status berubah jadi 'paid', stok/amount tidak berubah |
| TC-CREDIT-04 | Piutang dengan due_date < hari ini & payment_status=unpaid | Tampil sebagai "Overdue" di laporan (badge merah) |
| TC-CREDIT-05 | Set is_credit=true pada transaksi expense | Ditolak, pesan "Piutang hanya berlaku untuk transaksi pemasukan." |
| TC-CREDIT-06 | Non-admin membuat transaksi — opsi is_credit tidak tersedia | is_credit selalu false |
| TC-CREDIT-07 | **NEW** — Mark as Paid pada piutang yang sudah Paid | Tidak error, tetap paid |

#### TC-RPT (Laporan) — 8 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-RPT-01 | Hitung laporan laba/rugi untuk periode tertentu | Total income/expense/net sesuai SUM data transaksi di periode tsb |
| TC-RPT-02 | Filter laporan laba/rugi by category | Hanya transaksi dengan produk di kategori tersebut |
| TC-RPT-03 | Filter laporan laba/rugi by user | Hanya transaksi user tertentu |
| TC-RPT-04 | Hitung laporan arus kas | Opening balance sesuai settings, running balance per baris benar, closing balance benar |
| TC-RPT-05 | Laporan piutang menampilkan status overdue | Overdue terhighlight (baris merah) |
| TC-RPT-06 | Non-admin lihat daily report | Hanya transaksi milik sendiri, tanggal tertentu |
| TC-RPT-07 | **NEW** — Export PDF laporan laba/rugi | Klik "Export PDF" → download file PDF dengan data laporan |
| TC-RPT-08 | **NEW** — Laporan arus kas tanpa transaksi di rentang | Opening balance = closing balance, tabel kosong |

#### TC-MASTER (Master Data) — 7 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-MASTER-01 | Hapus kategori yang masih punya produk | Ditolak, pesan "Kategori tidak dapat dihapus karena masih memiliki N produk." |
| TC-MASTER-02 | Hapus produk yang punya riwayat transaksi | Ditolak, pesan "Produk tidak dapat dihapus karena memiliki riwayat transaksi." |
| TC-MASTER-03 | Non-admin akses halaman kategori | 403 atau menu tidak tampil |
| TC-MASTER-04 | Non-admin akses halaman users | 403 atau menu tidak tampil |
| TC-MASTER-05 | Admin toggle status user nonaktif | User tidak bisa login |
| TC-MASTER-06 | **NEW** — Admin ganti password user via form edit | User bisa login dengan password baru |
| TC-MASTER-07 | **NEW** — Export CSV produk | Download file CSV berisi data produk |

#### TC-VAL (Validasi Form) — 9 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-VAL-01 | Input harga dengan huruf pada form produk | Pesan validasi "price must be a number" |
| TC-VAL-02 | Input stock negatif | Pesan validasi "stock must be at least 0" |
| TC-VAL-03 | Submit form dengan field kosong (required) | Pesan validasi sesuai field |
| TC-VAL-04 | Input email duplikat pada form user | Pesan validasi "email already taken" |
| TC-VAL-05 | Input nama kategori duplikat | Pesan validasi "name already taken" |
| TC-VAL-06 | **NEW** — due_date < transaction_date pada credit transaction | Error validasi "due_date harus setelah transaction_date" |
| TC-VAL-07 | **NEW** — Quantity = 0 pada detail transaksi | Error validasi "quantity minimal 1" |
| TC-VAL-08 | **NEW** — Password < 8 karakter | Error validasi "password minimal 8 karakter" |
| TC-VAL-09 | **NEW** — Transaction amount negatif (transaksi tanpa detail) | Error validasi "amount minimal 0" |

#### TC-UI (Tampilan & Responsive) — 9 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-UI-01 | Tampilan di mobile (viewport 375px) | Layout menyesuaikan, sidebar collapsible |
| TC-UI-02 | Tampilan di tablet (viewport 768px) | Layout menyesuaikan |
| TC-UI-03 | Tampilan di desktop (viewport 1280px) | Layout penuh |
| TC-UI-04 | Indikator stok: stock=0 | Badge merah "Out of Stock" |
| TC-UI-05 | Indikator stok: 0 < stock <= threshold | Badge kuning "Low Stock" |
| TC-UI-06 | Indikator stok: stock > threshold | Badge hijau "Normal" |
| TC-UI-07 | Alert flash message muncul setelah aksi | Sesuai tipe (success/error/warning/info) |
| TC-UI-08 | **NEW** — Cross-browser test (Chrome, Firefox, Edge) | Semua fitur berfungsi sama, layout tidak rusak |
| TC-UI-09 | **NEW** — Error page 403 diakses | Tampil halaman "403 — Akses Ditolak" dengan tombol kembali |

#### TC-EXPORT (Export) — 3 test case
| ID | Skenario | Expected |
|----|----------|----------|
| TC-EXPORT-01 | **NEW** — Export CSV dari halaman transaksi | File CSV terdownload, kolom sesuai data transaksi |
| TC-EXPORT-02 | **NEW** — Export PDF dari halaman laporan laba/rugi | File PDF terdownload, data laporan sesuai |
| TC-EXPORT-03 | **NEW** — Export PDF dari laporan arus kas | File PDF terdownload dengan data running balance |

### Phase 3: Bug Reporting Template
```
## Bug Report
**ID:** BR-xxx
**Module:**
**Environment:** Local / Staging
**Browser:** Chrome / Firefox / Edge
**Severity:** Critical / High / Medium / Low
**Steps to Reproduce:**
1.
2.
3.
**Expected Result:**
**Actual Result:**
**Screenshot/Link:**
**Assigned to:**
```

### Phase 4: Manual Testing Execution
1. Export test case ke spreadsheet (Google Sheets / Excel)
2. Jalankan setiap test case, catat hasil (Pass/Fail/Blocked)
3. Jika fail → buat bug report → assign ke anggota terkait
4. Retest setelah bug fix

### Phase 5: Regression Testing
Setelah bug fix:
1. Uji ulang test case yang terkait dengan bug
2. Uji ulang fitur terkait yang mungkin terpengaruh
3. Pastikan tidak ada regression

### Phase 6: Final Report
Buat laporan QA final:
- Ringkasan jumlah test case per modul (total, pass, fail, blocked)
- Daftar bug (open, fixed, closed) dengan severity
- Rekomendasi: **Release** / **Delayed** / **Conditional Release**
- Screenshot untuk bug yang masih open

## Checklist
- [ ] Test plan dibuat & disetujui lead
- [ ] 40+ test case siap (Auth: 8, TRX: 9, Credit: 7, Report: 8, Master: 7, Validation: 9, UI: 9, Export: 3)
- [ ] Auth & middleware testing selesai
- [ ] Transaksi & stok management testing selesai
- [ ] Piutang/credit testing selesai
- [ ] Laporan (profit-loss, cash-flow, receivables, daily) testing selesai
- [ ] Master data (kategori, produk, user) testing selesai
- [ ] Validasi form testing selesai (termasuk edge cases)
- [ ] Export CSV & PDF testing selesai
- [ ] UI & responsive testing selesai (3 viewport sizes)
- [ ] Cross-browser testing selesai (Chrome, Firefox, Edge)
- [ ] Bug report lengkap dengan screenshot
- [ ] Regression testing setelah bug fix
- [ ] Laporan QA final selesai
