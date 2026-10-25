# Panduan Deployment OPSi

## A. Siapkan Google Sheets dan Drive
1. Buat Google Spreadsheet baru. Salin ID dari URL, bagian di antara `/d/` dan `/edit`.
2. Buat folder Google Drive untuk dokumentasi foto. Salin folder ID dari URL.
3. Buka Extensions → Apps Script pada spreadsheet atau buka `script.google.com`.
4. Ganti isi `Code.gs` dengan file `backend/Code.gs`. Di Project Settings, aktifkan tampilan `appsscript.json` lalu salin konfigurasi dari `backend/appsscript.json`.
5. Pada Project Settings → Script Properties, tambahkan:
   - `SPREADSHEET_ID`: ID spreadsheet
   - `DRIVE_FOLDER_ID`: ID folder dokumentasi
   - `SESSION_SECRET`: string acak panjang (minimal 32 karakter)
6. Simpan. Pilih fungsi `setupDatabase` dan klik Run. Berikan izin Google Sheets dan Drive saat diminta. Pastikan sheet `Users`, `Master_Area`, `Laporan_Kunjungan`, `Settings`, `Sessions`, `Audit_Log` muncul.
7. Buat admin awal dengan menambahkan sementara `ADMIN_USERNAME`, `ADMIN_PASSWORD` (minimal 10 karakter), dan `ADMIN_NAME` ke Script Properties.
8. Pilih fungsi `createFirstAdmin` → Run. Setelah sukses, hapus `ADMIN_PASSWORD` dan sebaiknya hapus `ADMIN_USERNAME` serta `ADMIN_NAME`.
9. Periksa sheet `Users`: kolom `password_hash` harus berisi hash, bukan password asli.

## B. Deploy backend
1. Di Apps Script, pilih Deploy → New deployment.
2. Select type: Web app.
3. Execute as: Me (akun deployer).
4. Who has access: Anyone (endpoint harus dapat menerima request dari frontend publik). Semua tindakan dilindungi oleh validasi session dan role server-side; jangan mengandalkan URL yang sulit ditebak.
5. Deploy dan salin URL yang berakhir `/exec`.
6. Saat diminta otorisasi, periksa akses yang diminta sebelum menyetujui. Gunakan akun perusahaan yang kepemilikannya terkelola.
7. Jika mengubah backend, buat versi deployment baru atau edit deployment aktif ke versi terbaru.
8. Endpoint memakai POST JSON dan `Content-Type: text/plain` untuk mengurangi masalah preflight. Uji GET endpoint di browser; harus mengembalikan status service.

## C. Frontend lokal
1. Instal Node.js LTS.
2. Clone atau unduh repository.
3. Jalankan `npm install`.
4. Salin `.env.example` menjadi `.env`.
5. Ganti `VITE_OPSI_API_URL` dengan URL `/exec` hasil deployment.
6. Jalankan `npm run dev`, lalu uji dari browser.

## D. Upload ke GitHub
1. Buat repository GitHub bernama `OPSi`.
2. Jangan unggah `.env`, data klien asli, spreadsheet, foto, password, token, atau credential.
3. Commit semua source code kecuali file yang diabaikan `.gitignore`.
4. Pastikan URL API yang benar tersedia pada konfigurasi build. Untuk GitHub Actions, simpan URL API sebagai repository variable, bukan secret jika URL saja; jangan menaruh credential rahasia di frontend.
5. Jika repository publik, ingat bahwa source frontend dan URL endpoint akan terlihat publik.

## E. Deploy frontend
### Pilihan 1 — Netlify/Vercel (disarankan untuk kemudahan)
- Hubungkan repository GitHub.
- Build command: `npm run build`.
- Publish directory: `dist`.
- Tambahkan environment variable `VITE_OPSI_API_URL` dengan URL Apps Script `/exec`.
- Deploy.

### Pilihan 2 — GitHub Pages
- Konfigurasikan workflow Pages atau proses build untuk menerbitkan `dist`.
- Karena `vite.config.ts` menggunakan `base: './'`, asset path relatif cocok untuk repository project Pages.
- Set `VITE_OPSI_API_URL` saat build melalui GitHub Actions. Variabel `VITE_*` masuk ke bundle publik, sehingga hanya boleh berisi URL API, bukan rahasia.
- Set repository Pages ke artifact/build yang benar dan uji routing halaman.

## F. Uji penerimaan sebelum digunakan
- Login berhasil, password salah ditolak, akun nonaktif ditolak.
- Setiap role hanya dapat menjalankan aksi yang diizinkan.
- Area Koordinator tidak dapat mengakses area/laporan milik koordinator lain meski payload dimanipulasi.
- Input laporan, validasi tanggal/teks, dan foto JPG/PNG ukuran maksimal 5 MB.
- Foto dapat dibuka oleh pengguna berwenang sesuai kebijakan Drive.
- Dashboard, filter, statistik area unik, dan ekspor sesuai data sheet.
- Uji dua pengguna menyimpan laporan bersamaan.
- Uji refresh berkala, putus jaringan, logout, dan sesi kedaluwarsa.
- Uji tampilan desktop, tablet, dan smartphone.

## Catatan keamanan dan keterbatasan
- SHA-256 di backend ini bukan pengganti ideal Argon2id/bcrypt untuk sistem sensitif. Jika diwajibkan standar keamanan perusahaan, pindahkan autentikasi ke backend yang mendukung password hashing adaptif.
- Apps Script memiliki kuota eksekusi, batas waktu, dan batas layanan. LockService hanya melindungi operasi yang secara eksplisit menggunakannya; lakukan uji konkurensi sebelum produksi.
- Drive file URL tidak dibuat publik otomatis. Jika akun lain perlu melihat foto, berikan akses melalui shared drive/folder atau implementasikan endpoint foto yang memvalidasi sesi. Jangan mengaktifkan `anyone` tanpa persetujuan keamanan.
- Pengaturan Web App `Anyone` dapat tidak tersedia pada sebagian akun Google Workspace. Jika dibatasi, gunakan hosting/API yang disetujui administrator domain.
- CORS dan redirect Google Apps Script bisa berbeda menurut hosting/browser. Uji pada domain deployment final; bila tidak kompatibel, gunakan backend proxy/serverless yang dikelola.
- Endpoint merespons JSON; Apps Script tidak menyediakan push realtime native. Polling 15–300 detik digunakan sebagai kompromi.
