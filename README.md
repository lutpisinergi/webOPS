# OPSi — Operational Performance System

Aplikasi web operasional PT. SINERGI Integra Services untuk laporan kunjungan Area Koordinator. Frontend React + TypeScript + Vite, backend Google Apps Script, database Google Sheets, dokumentasi foto di Google Drive.

## Fitur
- Login berbasis akun admin dan session token dengan masa berlaku.
- Role Direktur, Operasional Manager, Admin Operasional, Area Koordinator.
- Form laporan, validasi, upload JPG/PNG hingga 5 MB.
- Dashboard KPI, tren kunjungan, dan 10 laporan terbaru.
- Riwayat laporan, pencarian/filter, detail dokumentasi.
- Ekspor XLSX, CSV, dan cetak/Simpan PDF.
- Master area, pengelolaan akun, pengaturan target dan interval polling.
- Refresh berkala (default 30 detik), refresh manual, audit log.

## Persyaratan
- Node.js 20 LTS atau versi LTS yang kompatibel.
- Akun Google yang dapat membuat Spreadsheet, folder Drive, dan Apps Script.
- Repository GitHub dan hosting statis (GitHub Pages, Netlify, atau sejenisnya).

## Jalankan frontend
```bash
npm install
cp .env.example .env
# Ubah VITE_OPSI_API_URL dengan URL Web App Apps Script
npm run dev
```
Tanpa backend yang telah disiapkan, login tidak dapat dilakukan; aplikasi tidak menggunakan data dummy sebagai pengganti database.

## Setup backend
Lihat [DEPLOYMENT.md](./DEPLOYMENT.md). Penting: jalankan `setupDatabase()` sekali dari editor Apps Script, buat admin pertama, lalu hapus properti password sementara.

## Keamanan penting
- Jangan commit `.env`, token, password, atau Script Properties.
- Apps Script Web App yang dapat dipanggil frontend publik bukan batas keamanan tersendiri. Semua aksi tetap memvalidasi sesi dan role di server.
- Password diimplementasikan sebagai SHA-256 dengan salt/secret aplikasi karena batasan runtime Apps Script. Untuk lingkungan berisiko tinggi atau skala besar, gunakan backend dengan Argon2id/bcrypt dan database transaksional.
- Script Properties bukan vault khusus; batasi akses editor Apps Script hanya kepada administrator tepercaya.
- Folder Drive tidak otomatis dibuat publik. Pastikan kebijakan berbagi file cocok dengan kebijakan perusahaan.
- Google Sheets + Apps Script menggunakan polling, bukan push real-time native. Untuk beban tinggi, migrasikan database ke layanan transaksional.
- Kode ini merupakan fondasi implementasi yang perlu diuji di project Google milik perusahaan sebelum produksi.

## Build
```bash
npm run build
npm run preview
```
