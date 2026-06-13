# Agent Guide: Rifki — Project Leader & System Analyst

## Tanggung Jawab
- Spesifikasi kebutuhan
- Rancangan database (ERD)
- Dokumen API
- Struktur folder
- Pembagian & koordinasi tugas
- Integrasi modul
- Merge hasil kerja anggota tim

## Source of Truth
- **PRD Master:** `prd.md` (seluruh dokumen)
- **Readme Project:** `docs/readme.md`
- **Changelog:** `docs/changelog.md`

## Tugas Utama

### 1. Setup Infrastructure
1. Buat repository private di GitHub (atau gunakan `github.com/RCS15/uas-pw`)
2. Setup struktur folder:
   ```
   umkm-project/
   ├── backend/        # Laravel application
   ├── frontend/       # (opsional, jika pisah)
   ├── docs/           # Dokumentasi
   └── prd.md          # PRD Master
   ```
3. Install Laravel 13 di `backend/`:
   ```bash
   composer create-project laravel/laravel backend
   ```
4. Konfigurasi `.env` untuk MySQL
5. Setup Laravel Breeze (Blade stack):
   ```bash
   cd backend
   composer require laravel/breeze --dev
   php artisan breeze:install blade
   npm install && npm run build
   ```
6. Pastikan semua anggota tim punya akses ke repo
7. Buat branch structure:
   - `main` — production
   - `develop` — integration
   - `feature/*` — per anggota

### 2. Integrasi Modul
1. Review & merge PR dari setiap anggota
2. Pastikan tidak ada konflik kode
3. Jalankan `php artisan migrate:fresh --seed` untuk verifikasi
4. Jalankan `php artisan test` untuk verifikasi testing

### 3. Koordinasi
- Bagikan docs/agent-guides/* ke masing-masing anggota
- Monitor progres setiap anggota
- Bantu resolve masalah blocking
- Pastikan semua deliverable (PRD Bagian 15) terpenuhi

## Deliverables
1. Repository GitHub dengan kode lengkap
2. ERD final
3. Aplikasi terdeploy di GCR
4. Demo aplikasi

## Checklist Akhir
- [ ] Setup Laravel + Breeze + MySQL
- [ ] Semua migration berjalan sukses
- [ ] Semua seeder berfungsi
- [ ] Semua route terdaftar & middleware jalan
- [ ] Tidak ada konflik merge
- [ ] Aplikasi siap deploy
