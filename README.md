[![License: CC BY-NC 4.0](https://img.shields.io/badge/license-CC%20BY--NC%204.0-lightgrey.svg)]

## STIK SIAP – Sistem Informasi Administrasi Pembayaran Mahasiswa

**STIK SIAP** adalah aplikasi web untuk mengelola administrasi pembayaran mahasiswa (tagihan, pembayaran, tunggakan, laporan) yang digunakan di lingkungan STIK Sekolah Tinggi Islam Kendal (dan kampus sejenis).

Dokumen teknis dan konsep lengkap ada di `docs/RINGKASAN_SISTEM.md`.  
**Panduan setup untuk admin** (urutan input mahasiswa, master, paket biaya, pengaturan pembayaran, sampai siap dipakai): `docs/PANDUAN_SETUP_ADMIN.md`.

---

## Fitur Utama

- **Berbasis Tagihan per Semester**
    - Paket biaya per prodi + angkatan.
    - Generate tagihan per mahasiswa per semester.
    - Biaya sekali (SPI, Wisuda, dsb.) dan rutin per semester (UKT, UTS, UAS).

- **Pembayaran & Tunggakan**
    - Input pembayaran per item tagihan (parsial atau beberapa item sekaligus).
    - Rekap total tagihan, total terbayar, total tunggakan.
    - Tunggakan per semester.

- **Portal Mahasiswa**
    - Halaman pembiayaan (ringkasan kewajiban).
    - Tagihan per semester.
    - Tunggakan.
    - Riwayat pembayaran.

- **Dashboard & Laporan**
    - Dashboard ringkasan (total tagihan, terbayar, tunggakan, jumlah mahasiswa).
    - Akses cepat **Cek Kekurangan Mahasiswa** (NIM / nama) langsung dari dashboard.
    - Laporan semester, mutasi, bulanan, tahunan.
    - Export Excel dan nota pembayaran (PDF).

- **Role & Keamanan**
    - Role: Super Admin, Admin Keuangan, Admin Akademik, Pimpinan, Mahasiswa.
    - RBAC dengan Spatie Permission.
    - Login mahasiswa & admin terpisah (admin path disembunyikan).
    - Proteksi brute-force login (delay, captcha, lock per akun).
    - 2FA (OTP) untuk admin/akun penting.

- **Lisensi / Keanggotaan**
    - Super Admin punya full akses tanpa wajib memasukkan lisensi.
    - Role lain (Admin Keuangan, Admin Akademik, Pimpinan, Mahasiswa) bergantung pada lisensi yang diaktivasi.
    - Kunci lisensi signed (HMAC) + integrity hash di DB untuk deteksi ubah manual.

---

## Teknologi

- **Framework**: Laravel (PHP)
- **Tampilan**: Blade + Tailwind CSS (SSR, bukan SPA)
- **Database**: MySQL / MariaDB

---

## Instalasi (Development)

1. Clone repo ke folder web server Anda (mis. `xampp/htdocs/stik_siap`).
2. Install dependency:

```bash
composer install
npm install
npm run build   # atau npm run dev saat development
```

3. Salin environment:

```bash
cp .env.example .env
php artisan key:generate
```

4. Sesuaikan `.env`:

- Konfigurasi database:

```env
DB_DATABASE=stik_siap
DB_USERNAME=root
DB_PASSWORD=
```

- Lisensi (development, contoh):

```env
LICENSE_BYPASS=true
LICENSE_SIGNING_KEY=isi_kunci_acak_anda_di_sini
```

5. Migrasi & seeder:

```bash
php artisan migrate --seed
```

6. Jalankan server:

```bash
php artisan serve
```

7. Buka di browser: `http://localhost:8000`

Untuk detail alur setup, role, dan lisensi, lihat `docs/RINGKASAN_SISTEM.md`.

---

## Lisensi Proyek

Kode aplikasi ini **bukan** open source bebas-komersial. Lisensi proyek ada di file `LICENSE`:

> Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)  
> Copyright (c) 2026 Nailul Ghufron

Ringkasan:

- **Boleh**:
    - Menggunakan, memodifikasi, dan membagikan kode ini.
- **Wajib**:
    - Memberikan atribusi yang layak kepada pemilik hak cipta (**Nailul Ghufron**) pada setiap penggunaan, modifikasi, atau distribusi.
- **Tidak boleh**:
    - Menggunakan untuk **tujuan komersial**.
    - Menjual, mensublicense, atau mendistribusikan untuk profit **tanpa izin tertulis**.
- **Untuk penggunaan institusional / operasional**:
    - Penggunaan di universitas, perusahaan, atau organisasi **membutuhkan perjanjian kerjasama (MoU)** resmi dengan pemilik hak cipta.

Teks lengkap lisensi:  
`https://creativecommons.org/licenses/by-nc/4.0/`

---

## Catatan

- Dokumen sistem (konsep bisnis, arsitektur, peran, lisensi, dsb.) : `docs/RINGKASAN_SISTEM.md`.
- Jika Anda ingin menggunakan atau mengembangkan aplikasi ini di luar konteks non-komersial, silakan hubungi pemilik hak cipta sesuai ketentuan di file `LICENSE`.

### Penggunaan Komersial

Setiap penggunaan untuk operasional institusi, layanan berbayar,
atau lingkungan produksi dianggap sebagai penggunaan komersial
dan memerlukan perjanjian tertulis (MoU) dengan pemilik hak cipta.

Penggunaan komersial dapat diajukan melalui perjanjian lisensi resmi.
