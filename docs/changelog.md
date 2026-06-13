# Changelog

## v1.0 — 2026-06-13

### Dokumentasi & Setup
- [x] PRD Master v1.0 final untuk development
- [x] Inisialisasi repository Git
- [x] Pembuatan docs/readme.md — acuan project
- [x] Pembuatan docs/changelog.md — catatan perubahan ini
- [x] Pembagian tugas ke 5 anggota tim
- [x] Panduan peran untuk AI agent di docs/agent-guides/
- [x] Revisi root README.md — tech stack detail, 9 phase urutan pengerjaan, arsitektur, RBAC, business rules, dokumentasi terkait

### Revisi PRD (Berdasarkan Gap Analysis)
- [x] Laravel 13 → Laravel 11 (semua referensi)
- [x] Registrasi middleware: `app/Http/Kernel.php` → `bootstrap/app.php` (Laravel 11)
- [x] CheckRole middleware: admin boleh akses non-admin routes
- [x] RBAC matrix: tegas 403, hapus opsi redirect
- [x] FK `products.category_id` → ON DELETE RESTRICT
- [x] FK `transaction_details.product_id` → ON DELETE RESTRICT
- [x] `settings.value` → TEXT (dari VARCHAR(255))
- [x] Indexes ditambahkan ke semua tabel
- [x] Race condition: `DB::transaction()` + `lockForUpdate()` + retry 3x
- [x] Cash Flow final: akrual (hapus opsi cash basis)
- [x] Bulk delete route + controller spec
- [x] Export CSV & PDF: `barryvdh/laravel-dompdf`
- [x] Default credentials: `admin@example.com` / `password`
- [x] Model casts, accessors, scopes di Section 6
- [x] Soft Delete hanya pada Transaction
- [x] Service layer: StockService, TransactionService, ReportService, ExportService
- [x] FormRequest: 8 class eksplisit
- [x] Migration timestamps: 2026

### Revisi Agent Guides
- [x] 01-rifki-leader.md: Laravel 11, Dockerfile, cloudbuild.yaml, branch protection, production seeder, Cloud SQL, CI/CD
- [x] 02-fadzhil-backend.md: indexes, model factories, service layer, FormRequest, casts/scopes, SoftDeletes, events, CheckRole fix, export, session driver
- [x] 03-satria-api-testing.md: model factories, SQLite in-memory wajib, concurrent stock test, edge cases (20+), coverage target 70%/90%, phpunit.xml config
- [x] 04-shidqi-frontend.md: Chart.js v4, native HTML5 date picker, confirmation dialog, empty/loading states, product modal design spec, client validation, responsive breakpoints, error pages 403/404, export buttons
- [x] 05-lita-qa.md: 40+ test cases (dari 21), export test, bulk delete test, cross-browser test, edge case validasi, credential fix

### Keputusan Arsitektur
| Keputusan | Opsi Dipilih |
|-----------|-------------|
| Laravel Version | 11 (stable) |
| Email/Password Reset | Nonaktifkan |
| Export | CSV + PDF (dompdf) |
| Soft Deletes | Transactions only |
| Chart Library | Chart.js v4 |
| Cash Flow Basis | Akrual |
| Bulk Delete | Diimplementasikan |
| Password Reset Email | Nonaktifkan (admin reset via Users) |

### Rencana Development (Todo)

**Phase 1 — Foundation (Rifki)**
- [ ] Setup Laravel 11 + Breeze + MySQL
- [ ] Dockerfile & cloudbuild.yaml
- [ ] Branch structure & protection

**Phase 2 — Database & Auth (Fadzhil)**
- [ ] Migration & Model (6 tabel + indexes + soft deletes)
- [ ] Middleware CheckRole
- [ ] Auth override + Model Factories
- [ ] Seeder (Production + Development)

**Phase 3 — Admin Backend (Fadzhil)**
- [ ] Service layer (StockService, TransactionService, ReportService, ExportService)
- [ ] FormRequest (8 classes)
- [ ] CRUD Controllers (users, categories, products, transactions)
- [ ] Stock management logic + pessimistic locking

**Phase 4 — Reports & Export (Fadzhil)**
- [ ] ReportController (profit-loss, cash-flow, receivables, markAsPaid)
- [ ] Export CSV & PDF
- [ ] Bulk delete

**Phase 5 — Non-Admin Backend (Fadzhil)**
- [ ] Dashboard, Transaction (create & history), Products (read-only), Daily Report

**Phase 6 — Frontend (Shidqi)**
- [ ] Layouts + Shared Components + Sidebars
- [ ] Admin views (dashboard, forms, tables, reports)
- [ ] Non-Admin views
- [ ] Chart.js integration
- [ ] Product selection modal
- [ ] Responsive design + error pages

**Phase 7 — Testing (Satria)**
- [ ] PHPUnit feature tests (10 files)
- [ ] Concurrent stock test
- [ ] Postman Collection
- [ ] Coverage 70%+

**Phase 8 — QA (Lita)**
- [ ] Test plan & 40+ test cases
- [ ] Manual testing execution
- [ ] Bug reports & regression

**Phase 9 — Deployment (Rifki)**
- [ ] GCR deployment
- [ ] Cloud SQL configuration
- [ ] Demo aplikasi
