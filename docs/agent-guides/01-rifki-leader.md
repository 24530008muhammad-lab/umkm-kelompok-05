# Agent Guide: Rifki — Project Leader & System Analyst

## Tanggung Jawab
- Spesifikasi kebutuhan & rancangan database (ERD)
- Dokumen API & struktur folder
- Pembagian & koordinasi tugas
- Integrasi modul & merge PR
- **Setup Laravel 11 + Breeze**
- **Dockerfile & cloudbuild.yaml** untuk GCR deployment
- **Branch protection & CI/CD pipeline**

## Source of Truth
- **PRD Master:** `prd.md` (seluruh dokumen — sudah revisi v1.0)
- **Readme Project:** `README.md` (root) dan `docs/readme.md`
- **Changelog:** `docs/changelog.md`

## Tugas Detail

### 1. Setup Repository & Branch Structure

```bash
# Clone
git clone https://github.com/24530008muhammad-lab/umkm-kelompok-05.git
cd umkm-kelompok-05

# Branch structure
git branch develop
git push -u origin develop
# Branch per anggota akan dibuat dari develop
# feature/fadzhil-backend
# feature/satria-testing
# feature/shidqi-frontend
# feature/lita-qa
```

**Branch rules (GitHub):**
1. `main` — production, protected, require PR review
2. `develop` — integration, protected, require PR review
3. `feature/*` — per anggota, boleh push langsung

### 2. Install Laravel 11 di backend/

**Prasyarat:** Pastikan XAMPP sudah terinstall dan Apache + MySQL sudah running.

```bash
mkdir backend && cd backend
composer create-project laravel/laravel .
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run build
```

**Buat database via phpMyAdmin:**
- Buka `http://localhost/phpmyadmin`
- Database → isi "uas-pw" → charset `utf8mb4_unicode_ci` → Create

**Konfigurasi .env (sesuai XAMPP/Laragon):**
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=uas-pw
DB_USERNAME=root
DB_PASSWORD=
SESSION_DRIVER=database
MAIL_MAILER=log
```

**Nonaktifkan email verification & password reset:**
- Breeze Blade stack tidak include email verification secara default
- Hapus route `forgot-password` dan `reset-password` dari `routes/auth.php` atau jangan include

**AppServiceProvider:**
```php
// app/Providers/AppServiceProvider.php
use Illuminate\Support\Facades\Schema;
public function boot(): void {
    Schema::defaultStringLength(191);
}
```

### 3. Setup Production Database (Cloud SQL)

Di Google Cloud Console:
1. Buat Cloud SQL instance (MySQL 8, db-n1-standard-1, 10GB)
2. Buat database `uas-pw`
3. Buat user
4. Catat connection name, database name, user, password

Tambahkan ke `.env.production`:
```
DB_CONNECTION=mysql
DB_HOST=/cloudsql/<PROJECT_ID>:<REGION>:<INSTANCE>
DB_DATABASE=uas-pw
DB_USERNAME=root
DB_PASSWORD=<secret>
SESSION_DRIVER=database
APP_ENV=production
APP_DEBUG=false
```

### 4. Dockerfile

Buat `backend/Dockerfile`:

```dockerfile
# Multi-stage build
FROM composer:latest AS composer
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader

FROM node:20 AS node
WORKDIR /app
COPY package.json package-lock.json vite.config.js ./
RUN npm ci && npm run build

FROM php:8.2-fpm-alpine
RUN docker-php-ext-install pdo_mysql
COPY --from=composer /app/vendor /app/vendor
COPY --from=node /app/public/build /app/public/build
COPY . /app
WORKDIR /app
RUN php artisan optimize

EXPOSE 8080
CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8080"]
```

### 5. cloudbuild.yaml

Buat `backend/cloudbuild.yaml`:

```yaml
steps:
  # Run migration (sebelum deploy)
  - name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud sql instances describe umkm-db --format='value(connectionName)'
        # Run migration via Cloud SQL proxy atau langsung
    waitFor: ['-']

  # Build Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/umkm-backend', '.']

  # Push to Google Container Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/umkm-backend']

  # Deploy to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'umkm-backend'
      - '--image=gcr.io/$PROJECT_ID/umkm-backend'
      - '--platform=managed'
      - '--region=asia-southeast1'
      - '--allow-unauthenticated'
      - '--add-cloudsql-instances=<INSTANCE_CONNECTION_NAME>'
      - '--set-env-vars=APP_ENV=production,APP_DEBUG=false,DB_DATABASE=uas-pw,DB_USERNAME=root,SESSION_DRIVER=database'
      - '--set-secrets=DB_PASSWORD=db-password:latest,APP_KEY=app-key:latest'

options:
  machineType: 'E2_HIGHCPU_8'

images:
  - 'gcr.io/$PROJECT_ID/umkm-backend'
```

### 6. Integrasi Modul

1. Terima PR dari setiap anggota:
   - Fadzhil: `feature/fadzhil-backend` → `develop`
   - Satria: `feature/satria-testing` → `develop`
   - Shidqi: `feature/shidqi-frontend` → `develop`
   - Lita: `feature/lita-qa` → `develop` (dokumen, bukan kode)

2. Setiap merge:
   ```bash
   git checkout develop
   git merge --no-ff feature/fadzhil-backend
   git push origin develop
   ```

3. Setelah semua terintegrasi:
   ```bash
   php artisan migrate:fresh --seed
   php artisan test
   npm run build
   ```

4. Jika semua OK, merge ke main:
   ```bash
   git checkout main
   git merge --no-ff develop
   git push origin main
   ```

### 7. Deployment ke GCR

```bash
gcloud config set project <PROJECT_ID>
gcloud builds submit backend/ --config backend/cloudbuild.yaml
```

Setelah deploy, dapatkan URL Cloud Run:
```bash
gcloud run services describe umkm-backend --region=asia-southeast1 --format='value(status.url)'
```

**Jalankan migration production:**
```bash
gcloud run jobs execute migrate-job
```
Atau jalankan via Cloud Build step (sudah ada di cloudbuild.yaml).

**Seed production (hanya admin + settings):**
```bash
php artisan db:seed --class=ProductionSeeder --force
```

### 8. Koordinasi Tim

- Bagikan docs/agent-guides/* ke masing-masing anggota
- Monitor progres via daily check (standup)
- Bantu resolve blocking issues
- Pastikan 6 phase berjalan sesuai urutan:
  | Phase | Waktu | Pengerja |
  |-------|-------|----------|
  | 1. Foundation | Day 1 | Rifki |
  | 2. Database & Auth | Day 1-2 | Fadzhil |
  | 3. Admin Backend | Day 2-4 | Fadzhil |
  | 4. Reports + Export | Day 4-5 | Fadzhil |
  | 5. Non-Admin Backend | Day 5-6 | Fadzhil |
  | 6. Frontend | Day 3-7 (paralel) | Shidqi |
  | 7. Testing | Day 6-7 | Satria |
  | 8. QA | Day 7-8 | Lita |
  | 9. Deployment | Day 8 | Rifki |

## Deliverables
1. Repository GitHub dengan kode lengkap (`github.com/24530008muhammad-lab/umkm-kelompok-05`)
2. ERD final (dokumen diagram)
3. Postman Collection (dari Satria)
4. Demo aplikasi (walkthrough fitur admin & non-admin)
5. Aplikasi terdeploy di GCR (URL)

## Checklist Akhir
- [ ] Setup Laravel 11 + Breeze + MySQL berhasil
- [ ] Branch structure: main, develop, feature/*
- [ ] Branch protection: main & develop require PR
- [ ] Dockerfile & cloudbuild.yaml siap
- [ ] Production .env dengan Cloud SQL + Secret Manager
- [ ] Semua migration berjalan sukses di local & production
- [ ] Semua seeder berfungsi (ProductionSeeder & DevelopmentSeeder)
- [ ] Semua route terdaftar & middleware jalan (403 untuk akses salah role)
- [ ] php artisan test lulus (Satria punya coverage)
- [ ] Tidak ada konflik merge
- [ ] npm run build berhasil
- [ ] Aplikasi terdeploy di GCR
- [ ] Demo aplikasi siap
