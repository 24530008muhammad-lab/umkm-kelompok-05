# Aplikasi Pengelolaan Keuangan UMKM

**Kelompok 5 | UAS Pemrograman Web**

| Role | Nama |
|------|------|
| Project Leader & System Analyst | Rifki |
| Backend (Model & Migration) | Fadzhil |
| Backend (API Testing) | Satria |
| Frontend Developer | Shidqi |
| Quality Assurance | Lita |

## Tech Stack

- **Backend:** Laravel 13 + MySQL
- **Frontend:** Blade Templating + Tailwind CSS
- **Auth:** Laravel Breeze (extended dengan role & status aktif)
- **Deployment:** Google Cloud Run (GCR)

## Struktur Project

```
umkm-project/
├── backend/           # Laravel application (monolith)
│   ├── app/
│   ├── database/
│   ├── resources/views/
│   ├── routes/
│   └── tests/
├── frontend/          # (opsional, jika pisah frontend)
├── docs/
│   ├── readme.md              # Ini — acuan utama project
│   ├── changelog.md           # Catatan perubahan
│   ├── agent-guides/          # Panduan per anggota tim
│   │   ├── 01-rifki-leader.md
│   │   ├── 02-fadzhil-backend.md
│   │   ├── 03-satria-api-testing.md
│   │   ├── 04-shidqi-frontend.md
│   │   └── 05-lita-qa.md
├── prd.md                     # Product Requirements Document (Master)
└── README.md                  # Root README ini
```

## Panduan Memulai

Lihat `prd.md` Bagian 16 untuk setup & instalasi lengkap.

## Role & Hak Akses

| Modul | Admin | Non-Admin |
|-------|-------|-----------|
| Dashboard | Statistik global | Ringkasan harian sendiri |
| Transaksi | CRUD semua | Create & Read milik sendiri |
| Produk | CRUD penuh | Read-only |
| Kategori | CRUD penuh | Tidak bisa akses |
| Pengguna | CRUD + aktivasi | Tidak bisa akses |
| Laporan Laba/Rugi | Akses penuh | Tidak bisa |
| Laporan Arus Kas | Akses penuh | Tidak bisa |
| Laporan Piutang | Akses penuh | Tidak bisa |
| Laporan Harian | - | Transaksi sendiri |

## Aturan Penting

- Non-Admin yang akses `/admin/*` → **HTTP 403 Forbidden**
- Stok otomatis: income = stok berkurang, expense (dengan produk) = stok bertambah
- Piutang hanya untuk transaksi income (`is_credit=true` + `due_date`)
- Kategori tidak bisa dihapus jika masih punya produk
- Produk tidak bisa dihapus jika punya riwayat transaksi

## Deployment

Aplikasi dideploy ke **Google Cloud Run (GCR)**. Repository: `github.com/RCS15/uas-pw`.
