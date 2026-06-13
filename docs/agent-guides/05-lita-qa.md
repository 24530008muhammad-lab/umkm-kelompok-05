# Agent Guide: Lita — Quality Assurance (QA)

## Tanggung Jawab
- Test plan & test case
- Pengujian manual
- Laporan bug

## Source of Truth
- **QA Checklist:** `prd.md` Bagian 14 (21 skenario)
- **Role & Hak Akses:** `prd.md` Bagian 4 (RBAC Matrix)
- **Business Rules:** `prd.md` Bagian 7
- **Validation:** `prd.md` Bagian 13
- **View Specification:** `prd.md` Bagian 12

## Tugas Detail

### Phase 1: Create Test Plan
Buat dokumen `docs/qa-test-plan.md` yang mencakup:
- Scope pengujian
- Lingkungan pengujian (local, staging)
- Tools yang digunakan
- Skenario prioritasi (Critical, High, Medium, Low)

### Phase 2: Test Case Documentation

Buat test case untuk setiap skenario di PRD Bagian 14, ditambah skenario tambahan:

**Test Case Template:**
```
| ID | Modul | Skenario | Langkah | Expected Result | Status |
|----|-------|----------|---------|-----------------|--------|
```

#### TC-AUTH (Autentikasi)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-AUTH-01 | Login sebagai admin (email: admin@example.com, password: password) | Redirect ke `/admin/dashboard`, akses semua menu admin |
| TC-AUTH-02 | Login sebagai non-admin | Redirect ke `/nonadmin/dashboard`, sidebar hanya menu non-admin |
| TC-AUTH-03 | Non-admin akses `/admin/dashboard` via URL langsung | HTTP 403 Forbidden |
| TC-AUTH-04 | Login dengan akun `is_active=false` | Login gagal, pesan "Akun Anda tidak aktif. Hubungi admin." |
| TC-AUTH-05 | Register user baru | User terdaftar dengan role default nonadmin |
| TC-AUTH-06 | Akses route tanpa login | Redirect ke halaman login |

#### TC-TRX (Transaksi & Stok)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-TRX-01 | Buat transaksi income dengan detail produk, stok cukup | Stok produk berkurang sesuai quantity, amount = total subtotal |
| TC-TRX-02 | Buat transaksi income dengan detail produk, stok tidak cukup | Ditolak, pesan error "Stok [nama produk] tidak cukup. Tersedia: X" |
| TC-TRX-03 | Buat transaksi expense dengan detail produk (restock) | Stok produk bertambah sesuai quantity |
| TC-TRX-04 | Buat transaksi expense tanpa detail (misal: bayar listrik) | Tidak ada perubahan stok, amount diinput manual |
| TC-TRX-05 | Edit transaksi (ubah quantity produk) | Stok di-rollback dan dihitung ulang dengan benar |
| TC-TRX-06 | Hapus transaksi income dengan detail | Stok di-rollback (stok bertambah kembali) |
| TC-TRX-07 | Hapus transaksi expense dengan detail | Stok di-rollback (stok berkurang kembali) |

#### TC-CREDIT (Piutang)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-CREDIT-01 | Buat transaksi income dengan `is_credit=true` + `due_date` | Transaksi tersimpan, payment_status='unpaid' |
| TC-CREDIT-02 | Buat transaksi income dengan `is_credit=true` tanpa `due_date` | Ditolak, validasi "due_date wajib diisi" |
| TC-CREDIT-03 | Tandai piutang sebagai "Paid" | payment_status berubah jadi 'paid', stok/amount tidak berubah |
| TC-CREDIT-04 | Piutang dengan due_date < hari ini & payment_status=unpaid | Tampil sebagai "Overdue" di laporan |
| TC-CREDIT-05 | Set is_credit=true pada transaksi expense | Ditolak, pesan "Piutang hanya berlaku untuk transaksi pemasukan." |
| TC-CREDIT-06 | Non-admin membuat transaksi — opsi is_credit tidak tersedia | is_credit selalu false |

#### TC-RPT (Laporan)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-RPT-01 | Hitung laporan laba/rugi untuk periode tertentu | Total income/expense/net sesuai SUM data transaksi di periode tsb |
| TC-RPT-02 | Filter laporan laba/rugi by category | Hanya transaksi dengan produk di kategori tersebut |
| TC-RPT-03 | Filter laporan laba/rugi by user | Hanya transaksi user tertentu |
| TC-RPT-04 | Hitung laporan arus kas | Opening balance sesuai settings, running balance per baris benar, closing balance benar |
| TC-RPT-05 | Laporan piutang menampilkan status overdue | Overdue terhighlight |
| TC-RPT-06 | Non-admin lihat daily report | Hanya transaksi milik sendiri, tanggal tertentu |

#### TC-MASTER (Master Data)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-MASTER-01 | Hapus kategori yang masih punya produk | Ditolak, pesan "Kategori tidak dapat dihapus karena masih memiliki N produk." |
| TC-MASTER-02 | Hapus produk yang punya riwayat transaksi | Ditolak, pesan "Produk tidak dapat dihapus karena memiliki riwayat transaksi." |
| TC-MASTER-03 | Non-admin akses halaman kategori | 403 atau menu tidak tampil |
| TC-MASTER-04 | Non-admin akses halaman users | 403 atau menu tidak tampil |
| TC-MASTER-05 | Admin toggle status user nonaktif | User tidak bisa login |

#### TC-VAL (Validasi Form)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-VAL-01 | Input harga dengan huruf pada form produk | Pesan validasi "price must be a number" |
| TC-VAL-02 | Input stock negatif | Pesan validasi "stock must be at least 0" |
| TC-VAL-03 | Submit form dengan field kosong (required) | Pesan validasi sesuai field |
| TC-VAL-04 | Input email duplikat pada form user | Pesan validasi "email already taken" |
| TC-VAL-05 | Input nama kategori duplikat | Pesan validasi "name already taken" |

#### TC-UI (Tampilan & Responsive)
| ID | Skenario | Expected |
|----|----------|----------|
| TC-UI-01 | Tampilan di mobile (viewport 375px) | Layout menyesuaikan, sidebar collapsible |
| TC-UI-02 | Tampilan di tablet (viewport 768px) | Layout menyesuaikan |
| TC-UI-03 | Tampilan di desktop (viewport 1280px) | Layout penuh |
| TC-UI-04 | Indikator stok: stock=0 | Badge merah "Out of Stock" |
| TC-UI-05 | Indikator stok: 0 < stock <= threshold | Badge kuning "Low Stock" |
| TC-UI-06 | Indikator stok: stock > threshold | Badge hijau "Normal" |
| TC-UI-07 | Alert flash message muncul setelah aksi | Sesuai tipe (success/error/warning/info) |

### Phase 3: Bug Reporting Template
Buat template laporan bug:

```
## Bug Report
**ID:** BR-xxx
**Module:**
**Environment:**
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
1. Print/export test case ke spreadsheet
2. Jalankan setiap test case satu per satu
3. Catat hasil (Pass/Fail/Blocked)
4. Jika fail → buat bug report → assign ke anggota terkait
5. Retest setelah bug fix

### Phase 5: Regression Testing
Setelah bug fix:
1. Uji ulang test case yang terkait
2. Uji ulang fitur terkait yang mungkin terpengaruh
3. Pastikan tidak ada regression

### Phase 6: Final Report
Buat laporan QA final:
- Ringkasan jumlah test case (total, pass, fail, blocked)
- Daftar bug yang ditemukan (open, fixed, closed)
- Rekomendasi: Release / Delayed

## Checklist
- [ ] Test plan sudah dibuat & disetujui
- [ ] Test case untuk semua 21 skenario PRD + skenario tambahan siap
- [ ] Auth & middleware testing selesai
- [ ] Transaksi & stok management testing selesai
- [ ] Piutang/credit testing selesai
- [ ] Laporan (profit-loss, cash-flow, receivables, daily) testing selesai
- [ ] Master data (kategori, produk, user) testing selesai
- [ ] Validasi form testing selesai
- [ ] UI & responsive testing selesai
- [ ] Bug report lengkap dengan screenshot
- [ ] Regression testing setelah bug fix
- [ ] Laporan QA final selesai
