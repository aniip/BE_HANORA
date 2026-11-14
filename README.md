# HANORA CAFE — Backend

Backend HANORA CAFE dibangun dengan **ASP.NET Core (.NET 10)**, **EF Core**, dan **PostgreSQL**.

Repo ini hanya berisi kode backend:

```
src/
  Hanora.Api/              Web API, controller, konfigurasi, Dockerfile entrypoint
  Hanora.Application/      DTO, kontrak service, aturan aplikasi
  Hanora.Domain/           Entity dan enum
  Hanora.Infrastructure/   EF Core, service, Midtrans, storage, export, seeding
tests/
  Hanora.UnitTests/        Unit test
  Hanora.IntegrationTests/ Integration test (Testcontainers PostgreSQL)
Dockerfile
Hanora.slnx
global.json
```

## Prasyarat

- .NET SDK **10.0.403** (lihat `global.json`)
- PostgreSQL 13+ (disarankan 17/18)
- Docker (untuk integration test dan deployment)

## Konfigurasi

Jangan menyimpan kredensial asli di `appsettings.json`. Gunakan environment variable atau
User Secrets.

Environment variable utama:

| Variabel | Keterangan |
|---|---|
| `ConnectionStrings__Default` | Connection string Npgsql |
| `JWT_SIGNING_KEY` | Kunci JWT minimal 32 byte |
| `APP_BASE_URL` | URL publik frontend/QR, misal `https://domain-anda` |
| `MIDTRANS_MODE` | `simulation`, `sandbox`, atau `production` |
| `MIDTRANS_SERVER_KEY` | Server key Midtrans (hanya backend) |
| `MIDTRANS_CLIENT_KEY` | Client key Midtrans |
| `MIDTRANS_MERCHANT_ID` | Merchant ID Midtrans |
| `MIDTRANS_QRIS_ENABLED` | `true` / `false` |
| `Database__AutoMigrate` | `true` untuk development |
| `Database__Seed` | `true` untuk membuat role/owner awal |
| `SEED_OWNER_EMAIL` / `SEED_OWNER_PASSWORD` | Owner awal (dipakai sekali) |
| `RateLimiting__Enabled` | `true` / `false` |

Contoh PowerShell:

```powershell
$env:ConnectionStrings__Default='Host=localhost;Port=5432;Database=hanora;Username=hanora;Password=...;Maximum Pool Size=20'
$env:JWT_SIGNING_KEY='nilai-acak-minimal-32-byte'
$env:APP_BASE_URL='http://localhost:5173'
$env:MIDTRANS_MODE='simulation'
```

## Menjalankan

```bash
dotnet restore
dotnet run --project src/Hanora.Api
```

API berjalan di `http://localhost:5080` (lihat `Properties/launchSettings.json`).
Health check:

```
GET /api/v1/health
GET /api/v1/health/ready
```

## Migrasi Database

```bash
# terapkan semua migration
dotnet run --project src/Hanora.Api -- --migrate

# migration + seed role/owner/pengaturan
dotnet run --project src/Hanora.Api -- --migrate --seed
```

Production sebaiknya memakai migration eksplisit, bukan auto-migrate.

## Test

```bash
dotnet test Hanora.slnx
```

Integration test memakai Testcontainers dan membutuhkan Docker aktif.

## Docker

```bash
docker build -t hanora-api .

docker run --rm -p 5080:8080 \
  -e ASPNETCORE_ENVIRONMENT=Development \
  -e ConnectionStrings__Default='Host=host.docker.internal;Port=5432;Database=hanora;Username=hanora;Password=...;Maximum Pool Size=20' \
  hanora-api
```

## Catatan Keamanan

- `appsettings.json` hanya berisi default **development**.
- Jangan commit `.env`, password database, server key Midtrans, atau JWT production.
- Mode `simulation` Midtrans ditolak saat `ASPNETCORE_ENVIRONMENT=Production`.
- Endpoint simulasi pembayaran hanya aktif pada mode `simulation`.
- Gunakan HTTPS dan reverse proxy (Caddy/nginx) di production.

## Lisensi

Kode ini dibuat untuk proyek HANORA CAFE.
