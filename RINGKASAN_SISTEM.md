# 🎓 RINGKASAN SISTEM ADMINISTRASI PEMBAYARAN MAHASISWA

**STIK SIAP** — Sistem Informasi Administrasi Pembayaran  
STIK Sekolah Tinggi Islam Kendal · Kampus 3  

Dokumen ini menjelaskan konsep, fitur yang tersedia, keamanan, dan desain sistem secara menyeluruh dan detail — **dari login hingga keseluruhan sistem**: mahasiswa login (username = NIM) → portal pembiayaan, tagihan, tunggakan, riwayat pembayaran; petugas login (umum `/login` atau admin `/msiap`) → dashboard (statistik keuangan, cek kekurangan), generate tagihan, input pembayaran (termasuk gunakan lebih bayar), void pembayaran, refund (catat & nota), laporan (ringkasan, mutasi pembayaran & refund, rekap kekurangan, export Excel), data master, pengaturan pembayaran & lisensi. Semua role, route, dan aturan akses diuraikan di bawah.

---

## Daftar Isi

1. [Tujuan Sistem](#-tujuan-sistem)
2. [Konsep Utama & Alur](#-1-konsep-utama-sistem)
3. [Jenis Pembiayaan](#-2-jenis-pembiayaan)
4. [Struktur Database (Konsep)](#-3-struktur-konsep-database-high-level)
5. [Fitur per Pengguna](#-4-fitursesuai-role)
6. [Role & Hak Akses (RBAC)](#-5-role-sistem-rbac)
7. [Lisensi & Keanggotaan](#-6-lisensi--keanggotaan)
8. [Penanganan Kasus Khusus & Salah Input](#-7-penanganan-kasus-khusus)
9. [Dashboard & Laporan](#-8-dashboard--laporan)
10. [Keamanan & Audit](#-9-prinsip-keamanan--audit)
11. [Bagian Teknis](#-bagian-teknis)
12. [Bagian Desain UI/UX](#-bagian-desain-uiux)
13. [Data Dummy & Seeder](#-10-data-dummy--seeder)
14. [Yang Masih Kurang / Dapat Dikembangkan](#-11-yang-masih-kurang--dapat-dikembangkan)
15. [Catatan implementasi (Pemeliharaan Data, Rekonsiliasi, Backup, Reset Data)](#catatan-implementasi-pemeliharaan-data-rekonsiliasi-backup-reset-data)

---

## 🎯 Tujuan Sistem

Sistem digunakan untuk:

- **Mengelola seluruh kewajiban pembayaran mahasiswa** dari semester 1 sampai lulus.
- **Mencatat pembayaran secara detail dan transparan** per item tagihan, dengan audit trail.
- **Melacak tunggakan per semester** (tagihan semester berjalan vs. tunggakan semester sebelumnya).
- **Menyediakan laporan keuangan yang akurat** (ringkasan semester dengan tunggakan/lebih bayar terpisah, mutasi pembayaran + mutasi refund, rekap kekurangan dengan kolom lebih bayar, export Excel terpisah untuk pembayaran dan refund).
- **Portal mahasiswa** untuk melihat tagihan, tunggakan, lebih bayar (sisa bayar), dan riwayat pembayaran.
- **Penanganan salah input:** void pembayaran (dengan konfirmasi kode transaksi); **lebih bayar** (bayar melebihi tagihan) dapat dipakai untuk semester berjalan atau dicatat **refund** (pengembalian) untuk mahasiswa semester akhir/lulus.
- **Nota** untuk pembayaran dan untuk refund (dengan keterangan refund dan keterangan tambahan).

Sistem bersifat:

- **Dinamis** — tambah jenis biaya tanpa ubah struktur database.
- **Siap jangka panjang** — scalable dan aman untuk audit bertahun-tahun.

---

## 🏗 1. Konsep Utama Sistem

Sistem berbasis **TAGIHAN**, bukan langsung pembayaran.

### Alur Utama

```
Master (Kampus, Prodi, Angkatan, Semester, Jenis Biaya)
    → Paket Biaya (per Prodi + Angkatan)
        → Generate Tagihan (per Mahasiswa per Semester)
            → Pembayaran (input per item tagihan; boleh lebih bayar)
                → Void (batalkan) atau Gunakan Lebih Bayar / Refund
            → Laporan & Export & Nota (pembayaran + refund)
```

| Tahap | Deskripsi |
|-------|------------|
| **Master** | Kampus, Program Studi, Angkatan, Semester, Jenis Biaya. |
| **Paket Biaya** | Aturan: jenis biaya + semester ke berapa muncul + nominal (bisa beda per angkatan). Satu paket per prodi+angkatan, berisi banyak item. |
| **Generate Tagihan** | Tagihan riil per mahasiswa per semester sesuai paket. Filter: semester, prodi, angkatan, kampus. |
| **Pembayaran** | Pencatatan bayar per item tagihan (bisa sebagian / beberapa item sekaligus). Nominal boleh melebihi sisa (lebih bayar); sisa item boleh negatif. |
| **Void / Lebih Bayar / Refund** | Void: batalkan pembayaran (konfirmasi kode). Lebih bayar: dipakai untuk semester berjalan (input "Gunakan lebih bayar") atau dicatat refund untuk mahasiswa tanpa tunggakan (semester akhir/lulus). |
| **Laporan** | Ringkasan (tagihan, terbayar, tunggakan, lebih bayar), mutasi pembayaran + mutasi refund, rekap kekurangan, tren bulanan/tahunan. Export Excel (pembayaran dan refund terpisah). Nota PDF (pembayaran dan refund). |

---

## 💰 2. Jenis Pembiayaan

### A. Sekali Selama Masa Kuliah

**Contoh:** Pendaftaran, OMABA, SPI, KKL, PPL, KKN, Seminar Proposal, Ujian Munaqasah, Infaq, Wisuda.

- Dibuat **1 kali** per mahasiswa.
- Muncul di **semester tertentu** (semester_urutan di paket).
- Nominal **bisa beda tiap angkatan**.

### B. Rutin Setiap Semester

**Contoh:** UKT, UTS, UAS.

- Muncul **setiap semester aktif** (berulang di paket).
- Nominal **bisa berbeda antar semester**.

### C. Insidental

Tipe tersedia di master **jenis_biaya** (`tipe: insidental`) untuk biaya di luar paket tetap (implementasi penuh dapat ditambah belakangan).

---

## 🧱 3. Struktur Konsep Database (High Level)

### Tabel Master

| Tabel | Fungsi |
|-------|--------|
| `kampus` | Kode, nama, alamat. |
| `program_studi` | Kode, nama, jenjang. |
| `angkatan` | Tahun angkatan (2019, 2020, …). |
| `semester` | Kode, nama, tahun_akademik, tipe (ganjil/genap), tanggal_mulai/selesai, is_aktif. |
| `jenis_biaya` | Kode, nama, tipe (sekali/per_semester/insidental), is_aktif. |
| `paket_biaya` | Prodi + angkatan, is_aktif. |
| `paket_biaya_items` | Item paket: jenis_biaya_id, semester_urutan, nominal, is_wajib. |
| `mahasiswa` | user_id, nim, nama, program_studi_id, angkatan_id, kampus_id, tanggal_lahir, status (aktif/cuti/lulus/do). |
| `users` | Login: name, username, email, password, kampus_id (untuk admin akademik), is_active, lock fields, 2FA fields. |

### Tabel Transaksi

| Tabel | Fungsi |
|-------|--------|
| `tagihan` | mahasiswa_id, semester_id, kode_tagihan, total_nominal, total_terbayar, total_sisa (SIGNED; boleh negatif = lebih bayar), status (lunas/parsial/belum_lunas). Soft delete. |
| `tagihan_items` | tagihan_id, jenis_biaya_id, nominal, terbayar, sisa (SIGNED; negatif = lebih bayar), status. |
| `pembayaran` | mahasiswa_id, kode_transaksi, tanggal_bayar, total_bayar (tunai saja), status (valid/void), keterangan_void (opsional). |
| `pembayaran_items` | pembayaran_id, tagihan_item_id, nominal_bayar. |
| `refunds` | mahasiswa_id, nominal, tanggal_refund, keterangan (opsional), user_id. Pencatatan pengembalian lebih bayar (refund). |

### Tabel Pendukung

| Tabel | Fungsi |
|-------|--------|
| `pengaturan_pembayaran` | Per kampus: rekening, kontak, info pembayaran. |
| `system_license` | license_key, licensed_to, expires_at, validated_at, integrity_hash (deteksi ubah manual). |
| `activity_logs` | Audit: user, action, model, description, ip, user_agent. |
| `user_devices` | Device terdaftar (untuk 2FA / keamanan). |
| `roles`, `permissions`, `model_has_roles`, dll. | Spatie Permission (RBAC). |

**Sinkronisasi pembiayaan dan tagihan:** Semua tampilan yang menampilkan “pembiayaan” (portal mahasiswa, dashboard cek kekurangan) dan “tagihan” (daftar tagihan, tunggakan, laporan ringkasan, rekap kekurangan) **mengambil dari satu sumber yang sama**: tabel **tagihan** dan **tagihan_items** (kolom state: `total_nominal`, `total_terbayar`, `total_sisa`, `terbayar`, `sisa`). Jadi dashboard, laporan, dan portal mahasiswa **selalu sinkron** satu sama lain selama state di database konsisten. Satu-satunya risiko adalah **state tidak sesuai dengan riwayat event** (pembayaran/void/refund); itu dideteksi dan bisa diperbaiki lewat **Rekonsiliasi** di menu Pemeliharaan Data Keuangan.

---

## 📋 4. Fitur (Sesuai Role)

### 4.1 Fitur Mahasiswa (Portal Mahasiswa)

| Fitur | Route / Menu | Deskripsi |
|-------|----------------|-----------|
| **Pembiayaan** | `/mahasiswa/pembiayaan` | Ringkasan: semester berjalan, kekurangan semester berjalan, tunggakan per semester sebelumnya, **total lebih bayar (sisa bayar)** jika ada, paket biaya per semester (terkunci/terbuka). Jika ada lebih bayar dan tidak ada tunggakan: pesan semester akhir/hubungi keuangan untuk refund. Landing default setelah login mahasiswa. |
| **Tagihan** | `/mahasiswa/tagihan` | Daftar tagihan per semester (filter semester). Detail per tagihan: item, nominal, terbayar, sisa. |
| **Tunggakan** | `/mahasiswa/tunggakan` | Total tunggakan + **lebih bayar** (jika ada) + detail per semester/jenis biaya (nominal, terbayar, sisa). Pesan refund jika tidak ada tunggakan. |
| **Riwayat Pembayaran** | `/mahasiswa/riwayat-pembayaran` | Daftar pembayaran (valid dan void) dengan detail item; transaksi void ditandai badge + keterangan. Paginated. |

Login mahasiswa: **username = NIM**, password default sesuai kebijakan (mis. tanggal lahir YYYYMMDD atau dari seeder).

---

### 4.2 Fitur Dashboard (Petugas: Super Admin, Admin Keuangan, Pimpinan, Admin Akademik)

| Fitur | Deskripsi |
|-------|------------|
| **Statistik ringkasan** | **Lima kartu:** Total tagihan, Total terbayar, **Tunggakan (kekurangan)** (hanya sisa positif), **Lebih Bayar (sisa bayar mahasiswa)**, Jumlah mahasiswa aktif. Selaras dengan Laporan Keuangan (identitas: Total Tagihan = Total Terbayar + Tunggakan − Lebih Bayar). Super Admin & Admin Keuangan & Pimpinan: seluruh data; Admin Akademik: terbatas kampus sendiri. |
| **Cek Kekurangan Mahasiswa** | Akses cepat tanpa pindah halaman: input **NIM atau nama**. Jika satu hasil → tampil grid 3 kartu: **Total Kekurangan**, **Tagihan Semester Sekarang**, **Tunggakan Semester Sebelumnya** + link "Input pembayaran". Jika banyak hasil (pencarian nama) → tampil **daftar kandidat**; user pilih satu → baru tampil 3 kartu. Endpoint: `POST /dashboard/cek-kekurangan` (body: `nim` atau `mahasiswa_id`). |
| **Akses Cepat (link)** | Generate Tagihan, Input Pembayaran, Laporan Keuangan, Refund (Admin Keuangan & Super Admin); Data Master (sesuai role). |
| **Panduan singkat** | Per role: langkah kerja, batasan akses. |

---

### 4.3 Fitur Keuangan

| Fitur | Route | Role | Deskripsi |
|-------|--------|------|-----------|
| **Generate Tagihan** | `GET/POST /keuangan/generate-tagihan` | Super Admin, Admin Keuangan | Pilih semester, prodi, angkatan, kampus → generate tagihan untuk mahasiswa yang memenuhi paket. Status per kombinasi (sudah/belum generate, jumlah mahasiswa). |
| **Input Pembayaran** | `GET /keuangan/pembayaran`, `GET /keuangan/pembayaran/mahasiswa/{id}`, `POST /keuangan/pembayaran` | Super Admin, Admin Keuangan, Admin Akademik | Cari mahasiswa (NIM/nama). Halaman mahasiswa: daftar tunggakan per item, **total lebih bayar** (jika ada). Input nominal per item (boleh melebihi sisa); **Gunakan lebih bayar (Rp)** untuk alokasi dari kredit ke tagihan. Simpan. Admin Akademik hanya kampus sendiri. Jika lebih bayar > 0 dan tunggakan = 0: tombol **Catat Refund** (Admin Keuangan/Super Admin). |
| **Void Pembayaran** | `POST /keuangan/pembayaran/{id}/void` | Admin Keuangan, Super Admin | Batalkan transaksi: konfirmasi **kode transaksi** + keterangan opsional. Nominal dikembalikan ke tagihan; status = void. Nota void tidak dapat dicetak. |
| **Refund** | `GET /keuangan/refund`, `GET /keuangan/refund/create`, `POST /keuangan/refund`, `GET /keuangan/refund/{id}/nota` | Admin Keuangan, Super Admin | **Daftar Refund:** filter tanggal, nama/NIM; total refund. **Catat Refund:** pilih mahasiswa dengan lebih bayar, isi nominal, tanggal, keterangan → simpan (sumber lebih bayar dikurangi). **Nota Refund:** PDF dengan keterangan refund + keterangan tambahan. |
| **Laporan** | `GET /keuangan/laporan` (index, mutasi, bulanan, tahunan, rekap) | Semua admin + Pimpinan | **Ringkasan:** Total tagihan, Terbayar, **Tunggakan (hanya sisa positif)**, **Lebih Bayar**, % terbayar. Identitas: Total Tagihan = Terbayar + Tunggakan − Lebih Bayar. **Mutasi:** tabel pembayaran (valid/void) + **Mutasi Refund** (tabel refund periode sama, total, link Nota, Export Excel refund). **Bulanan/Tahunan:** tren. **Rekap Kekurangan:** per mahasiswa + kolom **Lebih Bayar**; Export Excel/PDF. **Tab Refund:** link ke halaman Refund. |
| **Nota Pembayaran** | `GET /keuangan/pembayaran/{id}/nota` | Semua admin + Pimpinan | Cetak nota PDF per transaksi pembayaran (transaksi void tidak bisa cetak). |
| **Nota Refund** | `GET /keuangan/refund/{id}/nota` | Admin Keuangan, Super Admin | Cetak nota PDF refund: data mahasiswa, nominal, tanggal, keterangan refund, keterangan tambahan. |
| **Export Pembayaran** | `GET /keuangan/export/pembayaran` | Semua admin + Pimpinan | Export Excel mutasi pembayaran (filter tanggal, kampus, dll.). |
| **Export Refund** | `GET /keuangan/export/refund` | Semua admin + Pimpinan | Export Excel mutasi refund (filter sama; terpisah dari export pembayaran). |

---

### 4.4 Fitur Data Master

| Master | CRUD | Role (tambah/ubah) | Hapus |
|--------|------|---------------------|-------|
| **Kampus** | Index, Create, Edit | Super Admin, Admin Keuangan | Hanya Super Admin |
| **Program Studi** | Index, Create, Edit | Super Admin, Admin Keuangan | Hanya Super Admin |
| **Angkatan** | Index, Create, Edit | Super Admin, Admin Keuangan | Hanya Super Admin |
| **Semester** | Index, Create, Edit | Super Admin, Admin Keuangan | Hanya Super Admin |
| **Jenis Biaya** | Index, Create, Edit | Super Admin, Admin Keuangan | Hanya Super Admin |
| **Paket Biaya** | Index, Create, Edit (+ items) | Super Admin, Admin Keuangan | Hanya Super Admin |
| **Mahasiswa** | Index, Create, Edit, Reset Password | Super Admin, Admin Keuangan, Admin Akademik (kampus sendiri) | Hanya Super Admin |

- **Admin Akademik:** menu Data Master hanya **ringkasan** (lihat semester aktif, paket, jumlah mahasiswa) + **kelola mahasiswa kampus sendiri** (tambah, edit, reset password). Tidak bisa ubah Kampus, Prodi, Angkatan, Semester, Jenis Biaya, Paket Biaya.

---

### 4.5 Fitur Admin (Super Admin / Pengaturan)

| Fitur | Route | Role | Deskripsi |
|-------|--------|------|------------|
| **Master Akun** | `GET/POST /admin/master-akun`, `GET/PUT /admin/master-akun/{user}/edit`, reset password, toggle active | Super Admin | Daftar user (bukan mahasiswa), tambah/edit akun admin, assign role & kampus_id, reset password, aktif/nonaktif. |
| **Activity Log** | `GET /admin/activity-log`, `DELETE /admin/activity-log` | Super Admin | Log aktivitas (model, action, user, deskripsi, IP). Bisa clear log. |
| **Pengaturan Pembayaran** | CRUD `/admin/pengaturan-pembayaran` | Super Admin, Admin Keuangan | Rekening, kontak, informasi pembayaran **per kampus**. |
| **Backup & Restore** | `GET/POST /admin/backup`, download .sql, restore dari file | Super Admin | Download backup database (.sql); restore dari file .sql. Hanya untuk MySQL/MariaDB. Lihat subbagian Backup & Restore. |
| **Reset Data** | `GET/POST /admin/reset-data` | Super Admin | Hapus data terpilih lewat **UI**: checklist per grup (transaksi, mahasiswa, master, snapshot/arsip, activity log), **cek relasi tabel** (bergantung pada / memengaruhi), konfirmasi "HAPUS DATA TERPILIH". Akun admin dan role tidak dihapus. Urutan hapus aman: transaksi → mahasiswa → master → snapshot/arsip → activity log. |
| **Lisensi: Required** | `GET /admin/license/required` | — | Halaman "Sistem belum diaktivasi" (untuk role selain Super Admin & Admin Keuangan). |
| **Aktivasi Lisensi** | `GET/POST /admin/license/activate` | Super Admin, Admin Keuangan | Form masukkan kunci lisensi (untuk mengaktifkan sistem bagi role lain). Super Admin tetap akses penuh tanpa lisensi; Admin Keuangan yang biasanya mengaktivasi. Verifikasi signed atau via LICENSE_VALIDATION_URL. Simpan ke `system_license` + integrity_hash. |
| **Generate Kunci Lisensi** | `GET/POST /admin/license/generate` | Super Admin | Form: nama instansi, berlaku hingga → generate kunci signed. Jika LICENSE_SIGNING_KEY belum diatur: tampil petunjuk + tombol "Generate kunci acak untuk .env" (copy ke .env). |

---

### 4.6 Fitur Akun & Keamanan (Semua User yang Login)

| Fitur | Route | Deskripsi |
|-------|--------|-----------|
| **Ganti password pertama** | `/password/first-change` | Wajib jika password masih default; tidak bisa akses menu lain sebelum ganti. |
| **Profil** | `GET /account/profile`, edit, update | Lihat dan ubah nama, email, dll. |
| **Ubah password** | `GET/POST /account/password` | Password baru (validasi: minimal 8 karakter, kombinasi huruf & angka, tidak sama dengan tanggal lahir). **UI:** tombol tampilkan/sembunyikan password per field; **konfirmasi password** dengan style benar/salah (border hijau/merah + pesan "Password cocok" / "Password tidak cocok") saat mengetik. |
| **Keamanan & 2FA** | `GET /account/security` | Aktifkan/nonaktifkan 2FA (TOTP). Daftar perangkat (sessions/devices); bisa revoke perangkat. |

---

### 4.7 Login & Lupa Password

| Fitur | Route | Deskripsi |
|-------|--------|-----------|
| **Login umum** | `GET/POST /login` | Username (NIM untuk mahasiswa) + password. Throttle 5 percobaan per menit. Setelah login, redirect mahasiswa ke `/mahasiswa/pembiayaan`, lainnya ke `/dashboard`. |
| **Login admin (petugas)** | `/{adminPath}` (default: `/msiap`) | URL tersembunyi untuk petugas. Rate limit, delay bertahap, captcha setelah N percobaan gagal (konfigurasi: max_attempts, lock_minutes, require_captcha_after_attempts). Lock akun sementara jika brute force. |
| **Lupa password (email)** | `/forgot-password`, `/reset-password/{token}` | Request link reset via email; reset dengan token. |
| **Reset password by NIM** | `/forgot-password/nim`, `/reset-password-nim` | Mahasiswa tanpa email: verifikasi NIM → set password baru. |
| **Two-factor challenge** | `/two-factor-challenge` | Setelah username/password benar, jika 2FA aktif: input kode TOTP. |
| **Logout** | `POST /logout` | Keluar dari aplikasi. |

**Setelah login (redirect & akses):**

| Role | Redirect | Akses utama |
|------|----------|-------------|
| **Mahasiswa** | `/mahasiswa/pembiayaan` | Portal: Pembiayaan, Tagihan, Tunggakan, Riwayat Pembayaran. |
| **Super Admin / Admin Keuangan / Admin Akademik** | `/dashboard` | Dashboard (statistik, cek kekurangan), Keuangan (Generate Tagihan, Input Pembayaran, Refund*, Laporan), Data Master, Pengaturan. *Void & Refund hanya Admin Keuangan & Super Admin. |
| **Pimpinan** | `/dashboard` | Dashboard (statistik), Laporan (baca), Export, Nota pembayaran. |

---

## 🧑‍💼 5. Role Sistem (RBAC)

| Role | Deskripsi Singkat |
|------|-------------------|
| **Super Admin** | Full akses: semua master (termasuk hapus), Master Akun, Activity Log, Generate Tagihan, Input Pembayaran, **Void**, **Refund** (daftar, catat, nota), Laporan, Export (pembayaran + refund), Nota (pembayaran + refund), Pengaturan Pembayaran, Generate Lisensi. |
| **Admin Keuangan** | Sama seperti Super Admin kecuali: tidak ada Master Akun & Activity Log; hapus master hanya Super Admin. Termasuk **Void pembayaran** dan **Refund** (daftar, catat, cetak nota). |
| **Admin Akademik** | Input Pembayaran & Laporan **kampus sendiri**; Data Master **hanya ringkasan** + kelola **mahasiswa kampus sendiri**. Tidak ada Generate Tagihan, Void, Refund; tidak bisa ubah Kampus/Prodi/Angkatan/Semester/Jenis Biaya/Paket. |
| **Pimpinan** | Hanya baca: Dashboard, Laporan (ringkasan, mutasi termasuk refund, bulanan, tahunan, rekap), Export (pembayaran & refund), Nota pembayaran. Tidak ada input, Void, atau catat Refund. |
| **Mahasiswa** | Portal: Pembiayaan, Tagihan, Tunggakan (dengan info lebih bayar), Riwayat Pembayaran. |

**Ringkas per fitur:**

| Fitur | Super Admin | Admin Keuangan | Admin Akademik | Pimpinan |
|-------|:-----------:|:--------------:|:--------------:|:--------:|
| Generate Tagihan | ✅ | ✅ | ❌ | ❌ |
| Input Pembayaran | ✅ | ✅ (semua) | ✅ (kampus sendiri) | ❌ |
| Void Pembayaran | ✅ | ✅ | ❌ | ❌ |
| Refund (catat, daftar, nota) | ✅ | ✅ | ❌ | ❌ |
| Laporan / Export / Nota | ✅ | ✅ | ✅ (kampus) | ✅ (read) |
| Pengaturan Pembayaran | ✅ | ✅ | ❌ | ❌ |
| Data Master (CRUD) | ✅ (full + hapus) | ✅ (tambah/ubah) | ✅ ringkasan + mahasiswa kampus | ❌ |
| Master Akun / Activity Log | ✅ | ❌ | ❌ | ❌ |
| Generate / Aktivasi Lisensi | ✅ (bypass; full akses) | ✅ Aktivasi | ❌ | ❌ |
| Dashboard Cek Kekurangan | ✅ | ✅ | ✅ | ❌ |

Admin Akademik terikat **kampus** (`kampus_id` di user). Semua aksi Admin Akademik tercatat di Activity Log (via Super Admin).

---

## 🔐 6. Lisensi & Keanggotaan

Sistem dirancang **keanggotaan**: **Super Admin** punya **full kontrol** dan selalu bisa akses **tanpa perlu memasukkan lisensi**. Role lain (Admin Keuangan, Admin Akademik, Pimpinan, Mahasiswa) butuh lisensi valid; bila belum diaktivasi, hanya Admin Keuangan yang bisa mengisi form aktivasi. Ini memberi kendali penuh pada Super Admin sekaligus melindungi penggunaan oleh role lain tanpa lisensi.

### 6.1 Cara Kerja Lisensi (Langkah demi Langkah)

**Pemahaman singkat:** Yang **generate** kunci lisensi hanya **Super Admin**. Kunci itu **diberikan** ke **Admin Keuangan** (atau petugas yang bertugas mengaktifkan sistem). **Admin Keuangan** (atau Super Admin) lalu **mengaktivasi** dengan memasukkan kunci di aplikasi. Setelah aktivasi berhasil, **semua akun** (Admin Keuangan, Admin Akademik, Pimpinan, Mahasiswa) **bisa menggunakan** aplikasi sampai lisensi berakhir.

| Pihak | Tugas |
|-------|--------|
| **Super Admin** | Membuat kunci lisensi (Generate Lisensi), lalu memberikan kunci tersebut ke Admin Keuangan / instansi. |
| **Admin Keuangan** (atau Super Admin di lokasi klien) | Memasukkan kunci di form Aktivasi Lisensi agar sistem aktif. |
| **Semua role lain** | Setelah aktivasi, bisa pakai aplikasi sesuai hak akses masing-masing. |

**Langkah praktis:**

1. **Super Admin** login → masuk menu **Admin → Generate Lisensi** (atau **Pengaturan** → **Generate Lisensi** jika ada di menu).
2. Isi **Nama instansi / pemegang lisensi** (mis. "STIK Kampus 3") dan **Berlaku hingga** (mis. satu tahun ke depan) → klik **Generate kunci lisensi**.
3. **Salin** kunci yang muncul (string panjang). Berikan kunci ini ke **Admin Keuangan** (atau petugas yang akan mengaktifkan) melalui saluran aman (email, chat, dll.).
4. Di **instalasi yang akan dipakai** (kampus/klien): pastikan di file **.env** sudah ada **LICENSE_SIGNING_KEY** dengan **nilai yang sama** dengan yang dipakai Super Admin saat generate. Tanpa ini, kunci tidak bisa diverifikasi.
5. **Admin Keuangan** (atau Super Admin) login di aplikasi tersebut → masuk **Pengaturan → Aktivasi Lisensi** (atau saat belum ada lisensi, sistem akan mengarahkan ke halaman ini).
6. **Tempel (paste)** kunci yang diterima ke kolom **Kunci Lisensi** → klik **Aktivasi**.
7. Jika kunci valid, sistem menyimpan lisensi dan menampilkan sukses. **Mulai saat itu, semua akun** (Admin Keuangan, Admin Akademik, Pimpinan, Mahasiswa) **dapat menggunakan** aplikasi sampai tanggal berakhir lisensi.

**Aturan akses:** **Super Admin** tidak wajib memasukkan lisensi; ia selalu bisa akses aplikasi (full kontrol). **Admin Keuangan** (dan Super Admin jika mau) dapat membuka form Aktivasi Lisensi untuk mengaktifkan sistem bagi role lain. Alur baku: Super Admin buat kunci → beri ke Admin Keuangan → Admin Keuangan yang aktivasi; Super Admin tetap bisa akses dengan atau tanpa lisensi.

### 6.2 Alur Lisensi (Ringkas)

- **Super Admin:** selalu bisa akses aplikasi (tidak cek lisensi).
- **Belum/tidak valid (untuk role lain):** Admin Keuangan diarahkan ke **form Aktivasi Lisensi**; role lain ke halaman **"Sistem belum diaktivasi"**.
- **Admin Keuangan** (atau Super Admin) memasukkan kunci valid di **Pengaturan → Aktivasi Lisensi** → lisensi tersimpan → semua role selain Super Admin bisa pakai sistem (Super Admin tetap bisa dengan atau tanpa lisensi).

### 6.3 Pembuatan Kunci (Super Admin)

- **Admin → Generate Lisensi:** isi Nama instansi, Berlaku hingga → **Generate kunci lisensi**. Kunci (kode signed) disalin dan diberikan ke instansi.
- Jika **LICENSE_SIGNING_KEY** belum diatur di `.env`: halaman menampilkan petunjuk dan tombol **"Generate kunci acak untuk .env"** (nilai acak untuk disalin ke `.env`), lalu refresh dan generate kunci.

### 6.4 Bentuk Kunci

- Kunci **signed**: payload (nama instansi, tanggal berakhir) di-encode base64url + HMAC-SHA256 dengan `LICENSE_SIGNING_KEY`. Format: `base64url(payload).base64url(signature)`.
- Verifikasi **lokal** di aplikasi (tanpa wajib API). Agar valid, `LICENSE_SIGNING_KEY` di .env instansi **harus sama** dengan di instalasi yang membuat kunci.

### 6.5 Konfigurasi (.env)

| Variabel | Deskripsi |
|----------|-----------|
| `LICENSE_BYPASS` | Default **false**. Set `true` hanya untuk development (abaikan cek lisensi). |
| `LICENSE_EXPIRES_AT` | Tanggal berakhir Y-m-d (opsional, tanpa API). |
| `LICENSE_VALIDATION_URL` | URL API validasi kunci (opsional). Contoh: `http://localhost:8000/api/v1/license/validate`. |
| `LICENSE_SIGNING_KEY` | Rahasia untuk generate & verifikasi kunci signed. Wajib sama di instalasi pembuat kunci dan instansi. |

### 6.6 API Validasi Lisensi

- **POST /api/v1/license/validate**  
  Body: `license_key` (wajib), `domain`, `app` opsional.  
  Response JSON: `valid`, `expires_at`, `licensed_to`, `message`.  
  Digunakan jika instansi klien memakai validasi via server (set `LICENSE_VALIDATION_URL` ke URL ini).

### 6.7 Arsitektur Keamanan

- **Di mana kunci dibuat?** Hanya di lingkungan yang punya `LICENSE_SIGNING_KEY` di `.env` (biasanya instalasi developer/Super Admin).
- **Kunci dipakai ulang?** Satu kunci bisa dipakai di banyak instalasi selama key sama dan belum kedaluwarsa. Kebijakan satu kunci per instansi di sisi developer; kelak bisa ditambah validasi API/domain.
- **Proteksi ubah manual di DB:** Kolom **integrity_hash** (HMAC dari expires_at, licensed_to, license_key) disimpan saat aktivasi. Setiap cek lisensi memverifikasi hash; jika data diubah manual di DB, hash tidak cocok → lisensi dianggap tidak valid.

### 6.8 Perpanjangan Lisensi

1. Sebelum/sesudah tanggal berakhir: sistem menampilkan halaman aktivasi atau "belum diaktivasi".
2. Developer memberikan **kunci lisensi baru** ke instansi.
3. Super Admin/Admin Keuangan memasukkan kunci baru di **Aktivasi Lisensi**.
4. Sistem validasi (signed atau API); jika valid, data lisensi diperbarui.

**Perintah uji (development):** `php artisan license:simulate-expired` — mengatur lisensi saat ini kedaluwarsa untuk uji alur aktivasi.

---

## 🔄 7. Penanganan Kasus Khusus & Salah Input

| Kasus | Penanganan |
|-------|------------|
| Mahasiswa nunggak | Tidak mengganggu semester berjalan; setiap semester punya tagihan sendiri; laporan memisahkan tagihan semester sekarang vs. tunggakan. |
| UKT berbeda tiap semester | Nominal disimpan per semester di paket biaya / item tagihan. |
| Mahasiswa lulus / DO | Status mahasiswa diubah (aktif/cuti/lulus/do); data tidak dihapus untuk audit. |
| Tambah jenis biaya baru | Cukup tambah di master **jenis_biaya** dan atur di **paket biaya**; tidak perlu ubah struktur database. |
| Banyak hasil pencarian nama | Dashboard Cek Kekurangan menampilkan daftar kandidat; user pilih satu lalu tampil total kekurangan. |
| **Salah input pembayaran** | **Void (batalkan):** hanya Admin Keuangan & Super Admin. Konfirmasi **kode transaksi** + keterangan opsional. Nominal dikembalikan ke tagihan; status pembayaran = void. Riwayat mahasiswa dan laporan mutasi tetap tampilkan transaksi void (dengan badge) agar status jelas. |
| **Lebih bayar (bayar melebihi sisa)** | Diizinkan: `tagihan_items.sisa` dan `tagihan.total_sisa` boleh negatif. Total lebih bayar ditampilkan di input pembayaran, portal mahasiswa (Pembiayaan, Tunggakan). **Gunakan untuk semester ini:** input "Gunakan lebih bayar (Rp)" saat bayar → alokasi ke item tagihan + pengurangan sumber lebih bayar. **Semester akhir / tidak ada tunggakan:** tidak bisa dipakai di sistem; petugas **Catat Refund** (menu Refund) atau proses pengembalian di luar sistem. |
| **Refund (pengembalian lebih bayar)** | Menu **Refund:** daftar refund (filter), **Catat Refund** (pilih mahasiswa dengan lebih bayar, nominal, tanggal, keterangan). Sistem mengurangi sumber lebih bayar; tercatat di `refunds` dan activity log. **Nota Refund** berisi keterangan refund + keterangan tambahan. |

---

### 7.1 Detail: Penanganan Salah Input Pembayaran (Void, Lebih Bayar, Refund)

#### Kondisi yang sudah ada di sistem

**1. Struktur data**

- Tabel `pembayaran` punya kolom **`status`** dengan nilai `valid` atau `void`.
- Model `Pembayaran` memakai **SoftDeletes** (data tidak benar-benar dihapus).
- Laporan ringkasan mutasi dan export Excel **hanya menghitung** pembayaran `status = 'valid'` untuk Total Penerimaan; daftar transaksi di tab Mutasi menampilkan valid dan void agar status jelas.

**2. Fitur void (sudah ada)**

- Menu **Batalkan (void)** ada di halaman Input Pembayaran per mahasiswa (riwayat pembayaran).
- Hanya role **admin_keuangan** dan **super_admin** yang bisa void.
- Konfirmasi wajib: user harus mengetik **kode transaksi** di modal sebelum submit.
- **Keterangan opsional**: alasan pembatalan disimpan di kolom `keterangan_void` dan di activity log.
- Setelah void: nominal dikembalikan ke TagihanItem, status pembayaran = `void`, tercatat di activity log. Nota tidak dapat dicetak untuk transaksi void.
- **Riwayat pembayaran mahasiswa (portal):** transaksi valid dan void **ditampilkan**. Void dengan badge "Dibatalkan" dan keterangan (jika ada). Total dibayar hanya dihitung dari transaksi valid.
- **Laporan mutasi (admin):** tabel menampilkan transaksi **valid dan void** (status jelas); ringkasan Total Penerimaan hanya dari transaksi valid.
- Export Excel keuangan hanya memuat pembayaran valid.

**3. Lebih bayar (overpayment)**

- Mahasiswa boleh membayar **melebihi sisa** tagihan per item.
- Kolom **`tagihan_items.sisa`** dan **`tagihan.total_sisa`** diizinkan **negatif** (tipe SIGNED). Arti: `sisa = nominal - terbayar`; jika terbayar > nominal maka sisa negatif = **lebih bayar**.
- Kelebihan tercatat di item tagihan yang sama: `terbayar` bertambah, `sisa` jadi negatif. Status item = lunas. Kelebihan = kredit (bisa dipakai untuk semester berjalan atau refund).
- **Tampilan:** Total lebih bayar ditampilkan di: (1) halaman Input Pembayaran admin, (2) Portal Mahasiswa – Pembiayaan Saya, (3) Portal Mahasiswa – Tunggakan.

#### Penggunaan lebih bayar (sisa bayar)

**Untuk apa?**  
Lebih bayar = uang yang sudah dibayar mahasiswa melebihi tagihan pada item tertentu. Sistem mencatatnya sebagai **kredit** mahasiswa.

**Cara menggunakan (sudah diimplementasi):**

1. **Dipakai untuk semester berjalan**  
   Saat input pembayaran untuk mahasiswa yang punya tunggakan dan punya lebih bayar, petugas dapat mengisi **"Gunakan lebih bayar (Rp)"** di form. Nominal akan: (a) dialokasikan ke tagihan item yang sedang dibayar (total alokasi = tunai + lebih bayar), (b) dikurangi dari sumber lebih bayar (item dengan sisa negatif). **Total bayar** di catatan pembayaran = hanya tunai yang diterima.
2. **Semester akhir / mahasiswa lulus**  
   Jika mahasiswa **tidak punya tunggakan**, lebih bayar tidak bisa dipakai untuk tagihan berikutnya di sistem. Aplikasi menampilkan pesan dan tombol **Catat Refund** (Admin Keuangan/Super Admin). Pencatatan refund di menu Refund: daftar dengan filter, form Catat Refund (pilih mahasiswa, nominal, tanggal, keterangan). Tabel `refunds`; saat simpan, sumber lebih bayar dikurangi. Nota Refund untuk bukti.
3. **Rekomendasi kebijakan kampus:** Tentukan secara tertulis: (a) maksimal nominal lebih bayar yang boleh dipakai per semester, (b) syarat refund, (c) batas waktu klaim refund untuk mahasiswa lulus.
4. **Fitur yang tidak ada (disengaja):** Tidak ada fitur edit pembayaran (ubah nominal atau mahasiswa) atau hapus pembayaran dari antarmuka. Koreksi selalu via **void + input ulang**.

#### Void (batalkan) pembayaran — alur dan dampak

- **Alur:** Petugas pilih pembayaran → Batalkan (void) → konfirmasi kode transaksi (+ keterangan opsional) → sistem mengubah `pembayaran.status` menjadi `void`, mengembalikan nominal ke TagihanItem (kurangi `terbayar`, update `sisa` dan status, recalculate agregat tagihan), mencatat activity log.
- **Dampak:** Nominal kembali ke tagihan; data pembayaran tetap di DB untuk audit. Ringkasan keuangan (Total Penerimaan, export) hanya dari transaksi valid.
- **Kapan dipakai:** Salah input ke mahasiswa A → void → input ulang ke mahasiswa B. Salah nominal → void → input ulang dengan nominal benar.

#### Tidak disarankan: edit langsung nominal / mahasiswa

- Mengubah nominal atau mahasiswa = logic rumit dan rawan salah; audit lebih jelas jika transaksi lama dibatalkan (void) dan transaksi baru dicatat.
- **Rekomendasi:** Selalu **void + input ulang** untuk koreksi.

#### Kejelasan laporan keuangan (administrasi)

- **Ringkasan Semester:** Total Tagihan, Total Terbayar, Tunggakan (hanya sisa positif), Lebih Bayar, % Terbayar. Keterangan singkat menjelaskan istilah. Identitas: Total Tagihan = Terbayar + Tunggakan − Lebih Bayar.
- **Mutasi Pembayaran:** Total Penerimaan = tunai; tabel menampilkan transaksi valid dan void (status jelas).
- **Rekap Kekurangan:** Tabel per mahasiswa: Total Biaya, Sudah Bayar, Kekurangan (sem. ini & lalu), **Lebih Bayar**, Keterangan. Export Excel dan PDF menyertakan kolom Lebih Bayar. Baris dengan lebih bayar saja (tanpa tunggakan) tetap ditampilkan agar admin bisa tindak lanjut refund.

#### Ringkasan penanganan salah input

| Situasi         | Penanganan                                                                 |
|-----------------|----------------------------------------------------------------------------|
| Salah nominal   | Void pembayaran → input ulang dengan nominal benar                        |
| Salah mahasiswa | Void pembayaran → input ulang ke mahasiswa yang benar                     |
| Fitur void      | Sudah ada (admin_keuangan + super_admin, konfirmasi kode transaksi)       |
| Edit pembayaran | Tidak disarankan; pakai void + input ulang                                |

Dengan fitur **void** (status + kembalikan nominal ke tagihan + audit log) dan **refund** (pencatatan pengembalian lebih bayar), salah input dan lebih bayar dapat ditangani dengan aman dan terdokumentasi.

---

## 📊 8. Dashboard & Laporan

### Dashboard

- **Statistik (selaras dengan Laporan Keuangan):** Lima kartu — **Total Tagihan**, **Total Terbayar**, **Tunggakan (Kekurangan)** (hanya sisa positif), **Lebih Bayar (Sisa Bayar)**, **Mahasiswa Aktif**. Keterangan singkat: Tunggakan = kekurangan saja; Lebih Bayar = sisa bayar mahasiswa (kredit/refund). Identitas: Total Tagihan = Total Terbayar + Tunggakan − Lebih Bayar.
- **Cek Kekurangan:** Input NIM/nama → hasil langsung atau daftar pilih → grid Total Kekurangan, Tagihan Semester Sekarang, Tunggakan Sebelumnya + link Input Pembayaran.
- **Akses Cepat:** Link ke Generate Tagihan, Input Pembayaran, Laporan, Refund (Admin Keuangan & Super Admin), Data Master.
- **Panduan:** Langkah kerja dan batasan per role.

### Laporan Keuangan

- **Ringkasan semester:** Filter semester, kampus, prodi, angkatan. **Total Tagihan**, **Total Terbayar**, **Tunggakan** (hanya sisa positif), **Lebih Bayar**, **% Terbayar**. Keterangan istilah + identitas (Total Tagihan = Terbayar + Tunggakan − Lebih Bayar). Breakdown per prodi/angkatan dan per jenis biaya.
- **Mutasi Pembayaran:** Filter tanggal, semester, kampus, prodi, angkatan. Tabel pembayaran (valid & void; status jelas). Total Penerimaan = tunai (tidak termasuk potongan lebih bayar). **Mutasi Refund:** tabel refund periode sama, total refund, link Nota per baris, **Export Excel** (terpisah dari export pembayaran).
- **Bulanan:** Tren pemasukan per bulan (tahun pilih).
- **Tahunan:** Rekap per semester (tagihan, terbayar, tunggakan).
- **Rekap Kekurangan:** Filter berjenjang (kampus → prodi → semester → angkatan). Tabel per mahasiswa: Total Biaya, Sudah Bayar, Kekurangan (sem. ini & lalu), **Lebih Bayar**, Keterangan. Export Excel & PDF (termasuk kolom Lebih Bayar).
- **Tab Refund:** Link ke halaman Refund (Admin Keuangan & Super Admin).
- **Export Excel:** Pembayaran (`/keuangan/export/pembayaran`) dan Refund (`/keuangan/export/refund`) terpisah.
- **Nota PDF:** Pembayaran (per transaksi; void tidak bisa cetak) dan **Nota Refund** (keterangan refund + keterangan tambahan).

---

## 🔐 9. Prinsip Keamanan & Audit

- Semua pembayaran punya **detail per item** (pembayaran_items).
- **Tidak ada penghapusan fisik** transaksi (soft delete pada tagihan).
- Nominal di **tagihan** tidak diubah mundur ketika paket diubah (data transaksi riil).
- **Void:** hanya Admin Keuangan & Super Admin; **konfirmasi kode transaksi** wajib; keterangan void opsional; nominal dikembalikan ke tagihan; tercatat di activity log.
- **Refund:** tercatat di tabel `refunds` dan activity log; nota refund untuk bukti.
- **Activity Log** mencatat aksi penting (user, model, action, deskripsi, IP, subject).
- **Rate limit** login (umum & admin), **lock** akun sementara, **delay** bertahap, **captcha** setelah N gagal (admin).
- **2FA (TOTP)** opsional; daftar perangkat bisa di-revoke.
- **Lisensi** dengan integrity_hash mencegah pemalsuan data lisensi di DB.

---

## 🚀 Kelebihan Desain

- Fleksibel: beda nominal tiap semester/angkatan, tambah biaya kapan saja.
- Scalable dan aman untuk audit jangka panjang.
- Siap dikembangkan ke payment gateway atau integrasi lain.

---

# 🔧 BAGIAN TEKNIS

## 1. Stack Teknologi

- **Backend:** Laravel (PHP). Autentikasi, RBAC (Spatie Permission), export Excel/PDF.
- **Frontend:** Blade (SSR), Tailwind CSS. Bukan SPA.
- **Database:** MySQL/MariaDB (via Laravel migrations).

## 2. Login & Keamanan Teknis

- **Username:** NIM (mahasiswa) atau username lain (admin).
- **Password default:** Sesuai kebijakan (contoh: tanggal lahir YYYYMMDD; seeder: `20050101`).
- **Ganti password pertama:** Wajib jika masih default; aturan: minimal 8 karakter, huruf+angka, tidak sama dengan tanggal lahir.
- **Admin login path:** Konfigurasi `config('stik_siap.admin_login_path')` (default `msiap`). URL login petugas: `/{path}`.
- **Admin security:** max_attempts, lock_minutes, delay bertahap, require_captcha_after_attempts (lihat `config/stik_siap.php`).
- **2FA:** Kolom di `users` + `user_devices`; TOTP; revoke device dari halaman Keamanan.

## 3. API (Publik, Tanpa Auth)

- **GET /api/info** — Info aplikasi (nama, env, version).
- **POST /api/v1/license/validate** — Validasi kunci lisensi (body: `license_key`); response JSON.

## 4. Tabel Database (Ringkas)

- **Master:** kampus, program_studi, angkatan, semester, mahasiswa, jenis_biaya, paket_biaya, paket_biaya_items.
- **Transaksi:** tagihan (soft deletes; total_sisa SIGNED), tagihan_items (sisa SIGNED untuk lebih bayar), pembayaran (status valid/void, keterangan_void), pembayaran_items, **refunds** (mahasiswa_id, nominal, tanggal_refund, keterangan, user_id).
- **Pendukung:** users (lock, 2FA), pengaturan_pembayaran, system_license (integrity_hash), activity_logs (subject_type, subject_id), user_devices, roles/permissions (Spatie).

## 5. Arsitektur Aplikasi

- Monolithic Laravel, SSR, mobile responsive.
- Middleware: auth (web atau admin path), force.password (ganti password pertama), reject.locked (akun terkunci), check.license (lisensi valid atau bypass).

---

# 🎨 BAGIAN DESAIN UI/UX

- **Konsep:** Modern clean dashboard, minimalis, soft shadow, rounded corner.
- **Warna:** Primary indigo/blue; secondary slate; status: lunas (hijau), belum (merah), parsial (amber).
- **Layout:** Sidebar (desktop), navbar atas, content dengan card; mobile: hamburger, card full width.
- **Font:** Plus Jakarta Sans (rekomendasi proyek).

---

# 10. Data Dummy & Seeder

**DummyDataSeeder** mengisi contoh: Program Studi (ES, PAI, PGMI), Angkatan 2025, Semester (8 semester 2025/2026), Jenis Biaya (sekali + per semester), Mahasiswa + user (username = NIM, password `20050101`), Paket Biaya per prodi.

```bash
php artisan db:seed --class=DummyDataSeeder
# atau
php artisan migrate:fresh --seed
```

---

# 11. Yang Masih Kurang / Dapat Dikembangkan

Ringkasan hal yang **belum ada** atau **belum lengkap** di sistem saat ini, sebagai acuan pengembangan ke depan.

| Aspek | Yang kurang | Keterangan |
|-------|-------------|------------|
| **Jatuh tempo & denda** | Batas waktu bayar per semester (due date) dan perhitungan denda keterlambatan | Saat ini tidak ada kolom jatuh_tempo di tagihan/semester; denda tidak dihitung otomatis. Bisa ditambah jika kebijakan kampus menghendaki. |
| **Notifikasi ke mahasiswa** | Pengiriman email/WhatsApp reminder tunggakan atau tagihan baru | Ada notifikasi **di dalam portal** (panel kekurangan/tagihan); belum ada kirim email/WA otomatis ke mahasiswa. Pengaturan pembayaran sudah ada nomor WA (untuk link hubungi), bukan untuk broadcast. |
| **Biaya insidental** | Alur lengkap tagihan jenis insidental (di luar paket) | Tipe `insidental` ada di master jenis_biaya, tetapi generate tagihan dan paket mengandalkan paket biaya; penagihan insidental (tambah manual per mahasiswa) belum diimplementasi penuh. |
| **Payment gateway** | Integrasi pembayaran online (midtrans, xendit, dll.) | Disebut "siap dikembangkan" di desain; belum ada integrasi. Pembayaran saat ini dicatat manual oleh petugas. |
| **Portal mahasiswa – bukti** | Upload bukti transfer dari portal oleh mahasiswa | Bukti transfer hanya di-upload oleh **admin** saat input pembayaran. Mahasiswa tidak bisa upload bukti dulu untuk kemudian diverifikasi keuangan. |
| **Portal mahasiswa – riwayat refund** | Tampilan riwayat refund di portal | Refund hanya tercatat di sisi admin (menu Refund, mutasi refund). Mahasiswa tidak melihat daftar refund yang pernah diterima di portal. |
| **Laporan arus kas** | Satu view "Penerimaan − Refund = Net" per periode | Mutasi menampilkan penerimaan dan refund terpisah; belum ada satu kartu/ringkas "Net Penerimaan" (penerimaan − refund) per periode di dashboard atau laporan. Bisa ditambah di Ringkasan atau Mutasi. |
| **Export Activity Log** | Unduh/export log aktivitas (Excel/CSV) | Activity Log hanya tampil di halaman dan bisa dihapus; belum ada fitur export untuk keperluan audit eksternal. |
| **Import data** | Import mahasiswa atau tagihan dari Excel/CSV | Tambah/ubah data mahasiswa dan tagihan manual; belum ada import batch dari file. Berguna untuk migrasi atau sinkronasi dari sistem lain. |
| **Dokumentasi API** | Dokumentasi terbuka untuk API (jika nanti ada API untuk pihak ketiga) | Saat ini hanya API publik info dan validasi lisensi; belum ada API terauthentikasi untuk integrasi atau dokumentasi OpenAPI/Swagger. |
| **Backup & restore** | — | **Sudah tersedia** lewat menu **Backup & Restore** (Super Admin): download backup database (.sql) dan restore dari file .sql. Lihat subbagian di bawah. |

**Catatan:** Daftar di atas bukan daftar bug, melainkan **celah fitur** atau **ruang pengembangan**. Sistem sudah layak dipakai untuk administrasi pembayaran harian; poin di atas dapat diprioritaskan sesuai kebutuhan kampus.

### ➡️ Event-based vs Financial state-based & ketahanan data jangka panjang

**Apakah sistem ini "event based" atau "financial state based"?**

Sistem ini **sudah berbasis state keuangan (financial state–based)** untuk posisi saat ini:

- **Sumber kebenaran "berapa tunggakan / lebih bayar hari ini"** adalah kolom **state**: `tagihan.total_sisa`, `tagihan_items.terbayar`, `tagihan_items.sisa`. Dashboard, laporan ringkasan, rekap kekurangan, dan portal mahasiswa **membaca dari state ini**, bukan dari penjumlahan seluruh event pembayaran.
- **Event** (pembayaran, void, refund) tetap disimpan di `pembayaran`, `pembayaran_items`, `refunds` untuk **audit dan riwayat**. Setiap kali ada event, kode **mengubah state** (update `terbayar`/`sisa` lalu `recalculateAggregate()`), lalu menulis event.

**Di mana gap-nya? Bukan di “apakah state ada”, melainkan: apakah state bisa dijamin benar dalam jangka panjang?**

| Saat ini | Keterangan |
|----------|------------|
| State = disimpan ✔️ | `tagihan_items.terbayar`, `sisa`, `tagihan.total_terbayar`, `total_sisa`. |
| Event = disimpan ✔️ | `pembayaran`, `pembayaran_items`, `refunds` (valid/void tercatat). |

**Masalah: state bisa drift dari event.**

Dalam kondisi normal, seharusnya:

`tagihan_items.terbayar` = SUM(`pembayaran_items.nominal_bayar`) untuk item tersebut, hanya dari pembayaran **valid** (plus alokasi dari “gunakan lebih bayar” dan dikurangi saat void/refund). Tetapi **tidak ada mekanisme yang menjamin persamaan ini**. Jadi bisa terjadi:

`tagihan_items.terbayar` ≠ SUM(pembayaran_items valid)

**Penyebab drift (contoh):** bug di kode, rollback manual transaksi, restore DB (partial/selectif), hotfix data langsung di DB, race condition saat update state.

**Dampak saat drift terjadi:**

- ➡️ Sistem tetap jalan (tidak error).
- ➡️ Angka finansial bisa salah: tunggakan, lebih bayar, total terbayar di laporan/dashboard tidak lagi mencerminkan riwayat pembayaran yang sebenarnya.

**Yang belum ada:** mekanisme **rekonsiliasi otomatis** (periodic check + optional repair) yang memastikan state selaras dengan event. Tanpa itu, ketahanan jangka panjang state tidak terjamin; penjaga ketahanan yang dimaksud adalah: deteksi drift dan kemampuan perbaikan state dari event (recompute).

**Implikasi lain untuk data > 5 tahun:**

| Aspek | Keadaan saat ini | Risiko / yang kurang |
|-------|------------------|----------------------|
| **Posisi saat ini** | Dibaca dari state. | Tanpa rekonsiliasi, nilai state bisa salah tanpa kita tahu. |
| **State historis (point-in-time)** | Tidak ada snapshot state per tanggal. | Pertanyaan "total tunggakan per 31 Des 2022?" tidak bisa dijawab dari state sekarang. |
| **Volume tabel event** | Tabel event tumbuh tanpa batas. | Perlu strategi partisi/arsip untuk performa jangka panjang. |

**Kesimpulan:** Gap utama ketahanan finansial jangka panjang ada di **jaminan kebenaran state**: state dan event sama-sama disimpan, tetapi state bisa menyimpang dari event; belum ada rekonsiliasi otomatis. Rekomendasi: tambah **artisan command (atau job)** untuk: (1) memverifikasi `terbayar` vs SUM(pembayaran_items valid) per item, (2) opsional: memperbaiki state dari event (recompute), dijalankan berkala atau setelah restore/hotfix.

### Solusi untuk data > 5 tahun (ringkas)

| Masalah | Solusi yang disarankan |
|--------|-------------------------|
| **State drift** (terbayar ≠ sum event valid) | **Rekonsiliasi:** Artisan command (atau scheduled job) yang: (1) hitung per `tagihan_item`: `terbayar_harus = SUM(pembayaran_items.nominal_bayar)` dari pembayaran **valid** saja (plus alokasi lebih bayar, minus void/refund sesuai aturan); (2) bandingkan dengan `tagihan_items.terbayar`; (3) laporkan selisih (dry-run) atau perbaiki state (update `terbayar`, `sisa`, lalu `recalculateAggregate()`). Jalankan **periodik** (mis. mingguan) atau **setelah restore DB / hotfix**. |
| **Tidak ada state historis** (point-in-time) | Jika butuh laporan "tunggakan per tanggal X": (1) **Snapshot periodik** — job yang menyimpan salinan ringkas (mis. total_tunggakan, total_lebih_bayar per kampus/semester) ke tabel `snapshot_keuangan` dengan kolom `tanggal_snapshot`; atau (2) **Laporan dari event** — query mutasi sampai tanggal X dan hitung saldo (lebih rumit dan berat). |
| **Tabel event membesar** (pembayaran, pembayaran_items, activity_logs) | (1) **Partisi tabel** per tahun (mis. `pembayaran` partition by year(tanggal_bayar)) agar query dengan filter tanggal tetap cepat; (2) **Arsip** — pindahkan data lama (mis. > 5 tahun) ke tabel arsip atau DB terpisah, dan pastikan laporan harian hanya mengandalkan state + event periode aktif; (3) **Indeks** — pastikan indeks pada kolom filter (tanggal_bayar, status, mahasiswa_id, dll.) memadai. |

**Implementasi solusi (sudah tersedia di codebase):**

| Solusi | Command / fitur | Cara pakai |
|--------|------------------|------------|
| **Rekonsiliasi state** | `php artisan keuangan:reconcile` | Tanpa opsi: dry-run (hanya laporkan selisih). `--fix`: perbaiki state dari event. `--brief`: hanya jumlah, tanpa tabel. Jadwalkan mingguan atau jalankan setelah restore/hotfix. |
| **Snapshot historis** | `php artisan keuangan:snapshot` | Simpan ringkasan keuangan saat ini ke tabel `snapshot_keuangan`. Opsi: `--date=Y-m-d`, `--scope=global|kampus|semester`, `--id=` (untuk kampus/semester). Jadwalkan mis. tiap akhir bulan. |
| **Arsip pembayaran lama** | `php artisan keuangan:archive` | Tanpa `--execute`: dry-run (hanya hitung). `--execute`: salin ke `pembayaran_archive` & `pembayaran_items_archive` lalu hapus dari tabel utama. Opsi: `--years=5`, `--before=Y-m-d`, `--chunk=500`. |

**UI (hanya Super Admin):** Menu **Pemeliharaan Data** di sidebar (bagian Admin) mengarah ke halaman **Pemeliharaan Data Keuangan**. Di atas konten ada **ringkasan** berisi: tujuan halaman, apa saja yang bisa dilakukan, **SOP singkat** (kapan baiknya dipakai), dan **cara melihat serta membaca hasil** aksi. Tiga bagian berikutnya:

- **1. Cek konsistensi & perbaikan angka terbayar (Rekonsiliasi):** Tujuan = memastikan angka terbayar = jumlah dari riwayat pembayaran. SOP: minimal sekali seminggu atau setelah restore DB. Aksi: "Cek konsistensi" (hanya tampilkan hasil) dan "Perbaiki state sekarang" (jika ada selisih). Hasil: pesan di atas halaman; jika ada selisih, tabel (aktual vs expected, selisih) + penjelasan cara baca.
- **2. Cuplikan kondisi keuangan (Snapshot):** Tujuan = simpan ringkasan keuangan per tanggal untuk laporan point-in-time. SOP: tiap akhir bulan. Aksi: pilih tanggal + scope → "Simpan snapshot". Hasil: pesan sukses di atas; riwayat di tautan "Lihat riwayat snapshot" (tabel dengan total tagihan/terbayar/tunggakan/lebih bayar per tanggal).
- **3. Arsip pembayaran lama:** Tujuan = pindah transaksi sangat lama ke arsip agar tabel utama ringan. SOP: sekali setahun atau saat data > 5 tahun. Aksi: "Cek yang akan diarsipkan" (tahun) → tampil jumlah transaksi; "Arsipkan sekarang" (konfirmasi). Hasil: kotak "Hasil cek" dengan jumlah dan batas tanggal; setelah arsip, pesan sukses di atas.

Semua aksi hanya untuk role **super_admin**.

**Scheduler (otomatis):** Di `routes/console.php` dijadwalkan: **rekonsiliasi** tiap Minggu pukul 02:00 (WIB); **snapshot** tiap tanggal 1 bulan pukul 00:00 (WIB). Arsip tidak dijadwalkan otomatis (hanya manual lewat UI atau CLI). Agar scheduler berjalan di server: pasang cron `* * * * * php /path-to-artisan schedule:run` atau jalankan `php artisan schedule:work` (development).

### Backup & Restore database (UI)

**Akses:** Menu **Backup & Restore** di sidebar (Admin, hanya Super Admin). Halaman ini hanya berfungsi jika aplikasi memakai database **MySQL** atau **MariaDB**.

**Yang di-backup:** Seluruh isi database aplikasi (semua tabel: master, tagihan, pembayaran, refund, user, lisensi, activity log, dll.). Satu file berisi perintah SQL.

**Format file:** **.sql** (teks, UTF-8). Kompatibel dengan **MySQL** dan **MariaDB**. File dapat dibuka dengan editor teks atau di-restore ke server MySQL/MariaDB lain (misalnya setelah migrasi server).

**Cara pakai:** (1) **Download backup** — klik "Download backup sekarang (.sql)"; file akan terunduh dengan nama `stik_siap_backup_YYYY-MM-DD_HHmmss.sql`. (2) **Restore** — upload file .sql yang pernah di-backup (atau dari sumber terpercaya), ketik **RESTORE** (huruf besar) untuk konfirmasi, lalu klik "Restore database". **Peringatan:** Restore akan mengganti isi database saat ini; lakukan hanya untuk pemulihan atau impor yang direncanakan. Disarankan backup dulu sebelum restore.

**Catatan teknis backup:** Jika gagal dengan pesan "mysqldump tidak ditemukan", atur di `.env`: `MYSQLDUMP_PATH=/path/ke/mysqldump` (di XAMPP macOS sering: `/Applications/XAMPP/xamppfiles/bin/mysqldump`). Di server MariaDB baru, `mariadb-dump` disarankan (mysqldump deprecated): aplikasi mencari `/usr/bin/mariadb-dump` dulu; bisa dipaksa lewat `MARIADB_DUMP_PATH` atau `MYSQLDUMP_PATH`. Jika error **"Access denied ... @'::1'"**: gunakan **DB_HOST=127.0.0.1** (bukan `localhost`) di `.env` agar koneksi lewat IPv4; aplikasi juga menormalisasi host ke 127.0.0.1 untuk perintah backup/restore jika host = localhost. Backup sengaja tidak memakai opsi `--routines` dan `--triggers` agar tidak memicu error "Column count of mysql.proc is wrong" pada MariaDB yang pernah di-upgrade. Isi backup tetap lengkap untuk semua tabel dan data aplikasi.

**Reset Data (UI):** Dari halaman Backup & Restore ada link **"Buka halaman Reset Data"**. Menu sidebar Admin: **Reset Data**. Halaman Reset Data menyediakan: (1) **Checklist** lima grup — Transaksi (tagihan, pembayaran, refund), Mahasiswa & akun login mahasiswa, Data master (kampus, prodi, angkatan, semester, jenis biaya, paket biaya, pengaturan pembayaran), Snapshot keuangan & arsip pembayaran, Activity log — dengan jumlah data saat ini per grup; (2) **Cek relasi tabel** (panel yang bisa dibuka/tutup): tabel berisi kolom "Tabel", "Bergantung pada", "Memengaruhi" untuk memahami dependensi; (3) Konfirmasi ketik **HAPUS DATA TERPILIH**; (4) Tombol "Hapus data terpilih". Penghapusan dilakukan dengan urutan aman (transaksi → mahasiswa → master → snapshot/arsip → activity log). Akun admin dan role tidak pernah dihapus. Alternatif dari terminal: `php artisan stik_siap:reset-data` (opsi: `--force`, `--only-transaksi`, `--keep-activity-log`).

---

### Catatan implementasi (Pemeliharaan Data, Rekonsiliasi, Backup, Reset Data)

Ringkasan perubahan yang tercatat di dokumen ini:

| Area | Yang diimplementasikan | Lokasi kode / UI |
|------|------------------------|-------------------|
| **Sinkronisasi data** | Pembiayaan, tagihan, dan laporan memakai satu sumber (tabel `tagihan` & `tagihan_items`). Rekonsiliasi memastikan state = event. | Diuraikan di § 3 (Struktur Database) subbagian "Sinkronisasi pembiayaan dan tagihan". |
| **Rekonsiliasi — cek** | Cek konsistensi: bandingkan `tagihan_items.terbayar` dengan jumlah dari pembayaran valid + simulasi refund. | `KeuanganReconcileService::getDrifts()`. UI: tombol "Cek konsistensi", hasil di kotak "Hasil aksi" dan tabel jika ada selisih. |
| **Rekonsiliasi — perbaikan** | Aksi "Perbaiki state sekarang": menyesuaikan N item yang selisih (terbayar = expected, update sisa/status, recalculate tagihan). | `KeuanganReconcileService::fixDrifts($drifts)`; dipanggil dari `PemeliharaanDataController::reconcileFix()` dan dari CLI `keuangan:reconcile --fix`. UI: tombol tampil hanya ketika cek menemukan selisih; teks "Ada ketidaksesuaian — solusi aksi" menjelaskan penggunaan tombol. |
| **Snapshot** | Simpan ringkasan keuangan per tanggal + scope (global/kampus/semester). Riwayat snapshot dengan filter. | Command `keuangan:snapshot`; UI form + halaman "Lihat riwayat snapshot". Tabel `snapshot_keuangan`. |
| **Arsip** | Cek jumlah yang akan diarsipkan (per tahun) + eksekusi: salin ke `pembayaran_archive` / `pembayaran_items_archive`, hapus dari utama. | Command `keuangan:archive`; UI tombol "Cek yang akan diarsipkan" dan "Arsipkan sekarang". |
| **Scheduler** | Rekonsiliasi tiap Minggu 02:00 WIB; snapshot tiap tanggal 1 pukul 00:00 WIB. | `routes/console.php` (Schedule::command). |
| **Backup & Restore** | Download backup .sql (seluruh database); restore dari file .sql dengan konfirmasi RESTORE. Pencarian path mysqldump/mariadb-dump (env MYSQLDUMP_PATH, MARIADB_DUMP_PATH, atau path umum); normalisasi host localhost → 127.0.0.1 untuk hindari error "Access denied @'::1'"; tanpa routines/triggers untuk kompatibilitas MariaDB. | `BackupDatabaseController`; menu "Backup & Restore" di sidebar Admin. |
| **Reset Data (UI)** | Halaman checklist: pilih grup data (transaksi, mahasiswa, master, snapshot/arsip, activity log); panel "Cek relasi tabel" (bergantung pada / memengaruhi); konfirmasi "HAPUS DATA TERPILIH"; hapus dengan urutan aman. Service: hitung jumlah per grup, definisi relasi, eksekusi hapus. CLI: `stik_siap:reset-data` (opsi: --only-transaksi, --keep-activity-log, --force). | `ResetDataController`, `ResetDataService`; menu "Reset Data" di sidebar Admin; view `admin/reset-data/index.blade.php`. |

---

**Dokumen ini adalah ringkasan lengkap dan terperinci sistem STIK SIAP.**  
Terakhir diperbarui: mencakup login (umum & admin), portal mahasiswa, dashboard (statistik 5 kartu selaras laporan), fitur keuangan (generate tagihan, input pembayaran, void, lebih bayar, refund, nota pembayaran & nota refund), laporan (ringkasan dengan tunggakan/lebih bayar terpisah, mutasi pembayaran + mutasi refund, rekap kekurangan dengan kolom lebih bayar, export Excel terpisah), data master, pengaturan, lisensi, keamanan, teknis, **Pemeliharaan Data Keuangan** (rekonsiliasi dengan cek + perbaikan N item, snapshot, arsip, scheduler), **Backup & Restore** (download .sql, restore, normalisasi host & mariadb-dump, catatan teknis), **Reset Data** (UI checklist + cek relasi tabel, hapus data terpilih; CLI `stik_siap:reset-data`), **Ubah password** (tampilkan/sembunyikan password, validasi konfirmasi real-time), **sinkronisasi pembiayaan & tagihan**, dan **catatan implementasi** di atas. Semua fitur dari login hingga keseluruhan sistem dijelaskan di sini.
