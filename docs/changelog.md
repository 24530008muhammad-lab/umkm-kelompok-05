# Changelog

## v1.0 — 2026-06-13

### Dokumentasi & Setup
- [x] PRD Master v1.0 final untuk development
- [x] Inisialisasi repository Git
- [x] Pembuatan docs/readme.md — acuan project
- [x] Pembuatan docs/changelog.md — catatan perubahan ini
- [x] Pembagian tugas ke 5 anggota tim
- [x] Panduan peran untuk AI agent di docs/agent-guides/

### Rencana Development (Todo)

**Phase 1 — Foundation (Backend)**
- Setup Laravel 13 + Breeze + MySQL
- Migration & Model (6 tabel)
- Seeder (admin default, kategori, produk, settings)
- Middleware CheckRole
- Auth override (is_active check, redirect by role)

**Phase 2 — Admin Module (Backend)**
- CRUD Users, Categories, Products, Transactions
- Stock management logic
- Reports (Profit-Loss, Cash-Flow, Receivables)
- Mark as Paid untuk piutang

**Phase 3 — Non-Admin Module (Backend)**
- Dashboard, Transaction (create & history), Products (read-only), Daily Report

**Phase 4 — Frontend (Blade + Tailwind)**
- Layouts (app, auth), Sidebars, Alert component
- Admin views (dashboard, forms, tables, reports)
- Non-Admin views
- Responsive design

**Phase 5 — Testing & QA**
- Unit test endpoints
- Postman collection
- Manual test plan & execution
- Bug reporting

**Phase 6 — Deployment**
- Google Cloud Run deployment
- Demo aplikasi
