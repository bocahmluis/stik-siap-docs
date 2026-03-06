# Panduan Setup STIK SIAP — Dari Input Mahasiswa Sampai Siap Dipakai

Panduan ini menjelaskan **urutan langkah** yang harus dilakukan oleh admin agar sistem siap digunakan: mulai dari mengisi data master, input mahasiswa, mengatur paket biaya, generate tagihan, pengaturan pembayaran, sampai mahasiswa bisa login dan pembayaran bisa dicatat. Bahasa dibuat **sederhana dan runtun** agar mudah diikuti.

---

## Siapa yang Boleh Melakukan Setup?

- **Super Admin** dan **Admin Keuangan** dapat mengisi dan mengubah **semua data master** (kampus, prodi, angkatan, semester, jenis biaya, paket biaya).
- **Admin Akademik** hanya dapat mengelola **mahasiswa kampus sendiri**; tidak bisa menambah/mengubah kampus, prodi, semester, paket biaya, dll.
- **Pimpinan** dan **Mahasiswa** tidak punya akses untuk setup; mereka hanya melihat dan memakai sistem yang sudah diatur.

Pastikan Anda login dengan akun **Super Admin** atau **Admin Keuangan** saat mengikuti panduan ini.

---

## Urutan Setup (Harus Berurutan)

Sistem membutuhkan data dalam **urutan tertentu** karena ada ketergantungan (misalnya mahasiswa butuh kampus dan prodi; paket biaya butuh prodi dan angkatan). Ikuti urutan di bawah **jangan dibalik**.

| No | Langkah | Menu / Halaman | Singkat |
|----|---------|----------------|--------|
| 1 | Data Kampus | Data Master → Kampus | Isi kampus tempat mahasiswa terdaftar. |
| 2 | Program Studi | Data Master → Program Studi | Isi prodi (jurusan). |
| 3 | Angkatan | Data Master → Angkatan | Isi tahun angkatan (mis. 2023, 2024). |
| 4 | Semester | Data Master → Semester | Isi semester (ganjil/genap, tahun akademik). |
| 5 | Jenis Biaya | Data Master → Jenis Biaya | Isi jenis biaya (UKT, UTS, SPI, dll.). |
| 6 | Data Mahasiswa | Data Master → Mahasiswa | Input mahasiswa (NIM, nama, prodi, angkatan, kampus). |
| 7 | Paket Biaya | Data Master → Paket Biaya | Buat paket per prodi+angkatan, isi item (jenis biaya + nominal + semester ke berapa). |
| 8 | Semester Aktif | Data Master → Semester | Tandai satu semester sebagai **aktif** (untuk tagihan berjalan). |
| 9 | Pengaturan Pembayaran | Pengaturan → Pengaturan Pembayaran | Isi rekening, nomor WA, bank (per kampus/channel). |
| 10 | Generate Tagihan | Keuangan → Generate Tagihan | Buat tagihan per mahasiswa per semester sesuai paket. |
| 11 | Aktivasi Lisensi (jika perlu) | Pengaturan → Aktivasi Lisensi | Agar Admin Keuangan/Akademik/Pimpinan/Mahasiswa bisa akses. |
| 12 | Siap dipakai | — | Input pembayaran, mahasiswa bisa login portal. |

---

## Langkah 1: Data Kampus

**Menu:** Sidebar → **Data Master** → **Kampus**.

**Yang diisi:**
- **Kode:** Singkat dan unik (mis. `PLB`, `LMP`). Dipakai untuk membedakan kampus.
- **Nama:** Nama lengkap kampus (mis. "Kampus Plumbon").
- **Alamat:** Opsional.

**Contoh:** Kampus Plumbon (kode PLB), Kampus Limpung (kode LMP).

**Yang perlu diperhatikan:**
- Kode **tidak boleh sama** antar kampus.
- Jangan hapus kampus yang sudah dipakai oleh mahasiswa atau pengaturan pembayaran (sistem bisa error atau data jadi tidak konsisten).

---

## Langkah 2: Program Studi

**Menu:** Data Master → **Program Studi**.

**Yang diisi:**
- **Kode:** Singkat dan unik (mis. `ES`, `PAI`, `PGMI`).
- **Nama:** Nama program studi (mis. "Ekonomi Syariah", "Pendidikan Agama Islam").
- **Jenjang:** Opsional (mis. S1, D3).

**Contoh:** ES (Ekonomi Syariah, S1), PAI (Pendidikan Agama Islam, S1).

**Yang perlu diperhatikan:**
- Kode **tidak boleh sama** antar prodi.
- Jangan hapus prodi yang sudah dipakai oleh mahasiswa atau paket biaya.

---

## Langkah 3: Angkatan

**Menu:** Data Master → **Angkatan**.

**Yang diisi:**
- **Tahun:** Tahun angkatan (angka 4 digit, mis. 2023, 2024).
- **Nama:** Opsional (mis. "Angkatan 2024").

**Contoh:** 2023, 2024, 2025.

**Yang perlu diperhatikan:**
- Satu tahun angkatan **hanya satu record** (tidak boleh duplikat tahun).
- Jangan hapus angkatan yang sudah dipakai oleh mahasiswa atau paket biaya.

---

## Langkah 4: Semester

**Menu:** Data Master → **Semester**.

**Yang diisi:**
- **Kode:** Unik (mis. `20251`, `20252`). Biasanya tahun + nomor (1 = ganjil, 2 = genap).
- **Nama:** Nama semester (mis. "Gasal 2024/2025", "Genap 2024/2025").
- **Tahun akademik:** Mis. `2024/2025`.
- **Tipe:** Ganjil atau Genap.
- **Tanggal mulai / selesai:** Opsional.
- **Semester aktif:** Centang **hanya satu** semester yang sedang berjalan (untuk generate tagihan dan tampilan "semester berjalan").

**Contoh:** 20251 (Gasal 2024/2025, ganjil), 20252 (Genap 2024/2025, genap).

**Yang perlu diperhatikan:**
- Kode **tidak boleh sama**.
- **Hanya satu semester** yang boleh ditandai aktif. Ini dipakai saat generate tagihan dan laporan.
- Jangan hapus semester yang sudah punya tagihan.

---

## Langkah 5: Jenis Biaya

**Menu:** Data Master → **Jenis Biaya**.

**Yang diisi:**
- **Kode:** Unik (mis. `UKT`, `UTS`, `UAS`, `SPI`).
- **Nama:** Nama jenis biaya (mis. "UKT", "Uang UTS", "SPI").
- **Tipe:** Pilih salah satu:
  - **Sekali** — Biaya yang hanya sekali selama kuliah (SPI, Wisuda, KKN, dll.).
  - **Per semester** — Biaya tiap semester (UKT, UTS, UAS).
  - **Insidental** — Untuk keperluan khusus (bisa dipakai nanti jika fitur lengkap).
- **Deskripsi:** Opsional.
- **Aktif:** Centang jika jenis biaya ini masih dipakai.

**Contoh:** UKT (per semester), UTS (per semester), SPI (sekali), Wisuda (sekali).

**Yang perlu diperhatikan:**
- Kode **tidak boleh sama**.
- Jangan hapus jenis biaya yang sudah dipakai di paket biaya atau tagihan.

---

## Langkah 6: Data Mahasiswa

**Menu:** Data Master → **Mahasiswa**.

**Yang diisi:**
- **NIM:** Nomor induk mahasiswa. **Harus unik** dan akan dipakai sebagai **username login** mahasiswa.
- **Nama:** Nama lengkap mahasiswa.
- **Program Studi:** Pilih dari data prodi yang sudah diisi.
- **Angkatan:** Pilih tahun angkatan.
- **Kampus:** Pilih kampus.
- **Tanggal lahir:** Opsional (bisa dipakai untuk reset password default).
- **Status:** Aktif / Cuti / Lulus / DO.

**Setelah disimpan:** Sistem **otomatis membuat akun login** untuk mahasiswa dengan:
- **Username** = NIM
- **Password default** = sesuai kebijakan kampus (biasanya tanggal lahir YYYYMMDD atau bisa direset dari menu edit mahasiswa / Master Akun).

**Yang perlu diperhatikan:**
- **NIM tidak boleh sama** dengan mahasiswa lain.
- Pastikan **Prodi, Angkatan, dan Kampus** sudah ada sebelum input mahasiswa.
- Jika mahasiswa salah kampus/prodi/angkatan, **edit** dari daftar mahasiswa; jangan hapus lalu buat baru kalau sudah ada tagihan/pembayaran (agar riwayat tidak kacau).

---

## Langkah 7: Paket Biaya

**Menu:** Data Master → **Paket Biaya**.

Paket biaya = **aturan** "mahasiswa prodi X angkatan Y dikenai biaya apa saja, berapa nominal, dan di semester ke berapa".

**Langkah 7a — Buat paket:**
- **Nama:** Mis. "Paket S1 Ekonomi Syariah 2024".
- **Program Studi:** Pilih prodi.
- **Angkatan:** Pilih angkatan.
- **Aktif:** Centang jika paket ini dipakai.

**Langkah 7b — Tambah item paket:**
- Setiap **item** = satu jenis biaya + nominal + **semester urutan** (muncul di semester ke berapa).
- **Jenis biaya:** Pilih dari master Jenis Biaya.
- **Semester urutan:** Nomor urut semester (1, 2, 3, …). Untuk biaya **per semester**, isi urutan yang sama untuk tiap semester (mis. UKT selalu urutan 1 per semester). Untuk biaya **sekali**, isi semester kapan biaya itu muncul (mis. SPI di semester 1, Wisuda di semester 8).
- **Nominal:** Jumlah rupiah.
- **Wajib:** Centang jika wajib dibayar.

**Contoh:**  
- Prodi Ekonomi Syariah, Angkatan 2024: item UKT semester urutan 1–8 (masing-masing nominal beda kalau mau), UTS semester 1–8, SPI semester 1, Wisuda semester 8.

**Yang perlu diperhatikan:**
- Satu kombinasi **Prodi + Angkatan** sebaiknya **satu paket** (jangan duplikat paket yang sama).
- Pastikan **Semester** dan **Jenis Biaya** sudah ada sebelum mengisi item.
- Jangan hapus paket yang sudah dipakai untuk generate tagihan.

---

## Langkah 8: Tandai Semester Aktif

**Menu:** Data Master → **Semester**.

- Buka semester yang **sedang berjalan** (mis. Gasal 2024/2025).
- Pastikan **Semester aktif** dicentang **hanya untuk semester ini**.
- Semester lain yang sudah lewat atau belum jalan **jangan** dicentang aktif.

Ini penting agar saat **Generate Tagihan** dan laporan, sistem tahu "semester berjalan" yang mana.

---

## Langkah 9: Pengaturan Pembayaran

**Menu:** Sidebar → **Pengaturan** → **Pengaturan Pembayaran**.

Ini untuk **info rekening dan kontak** yang akan dipakai mahasiswa saat transfer (bisa lebih dari satu channel per kampus).

**Yang diisi per baris (per channel):**
- **Kampus:** Pilih kampus (boleh kosong = global).
- **Keterangan:** Nama channel (mis. "Admin Keuangan Plumbon", "Rekening BCA Kampus A").
- **Nomor WhatsApp:** Nomor WA admin/keuangan (format 62xxx atau 08xxx).
- **Nomor rekening:** Nomor rekening bank.
- **Nama bank:** Mis. BCA, BNI, Mandiri.
- **Default:** Centang **satu** saja sebagai default (tombol "Bayar" di portal mahasiswa pakai ini).
- **Urutan:** Angka untuk urutan tampil.

**Yang perlu diperhatikan:**
- Minimal isi **satu** pengaturan agar mahasiswa punya referensi kemana transfer.
- **Jangan** kosongkan semua; kalau ada dua kampus, isi minimal satu per kampus atau satu global.

---

## Langkah 10: Generate Tagihan

**Menu:** Keuangan → **Generate Tagihan**.

Ini langkah yang **membuat tagihan riil** per mahasiswa per semester sesuai **paket biaya**.

**Yang Anda lakukan:**
1. Pilih **Semester** (biasanya semester aktif).
2. Filter **Program Studi**, **Angkatan**, **Kampus** (bisa salah satu atau kombinasi).
3. Klik **Generate** / **Cek dan Generate**.
4. Sistem akan menghitung: mahasiswa mana saja yang memenuhi filter dan punya **paket biaya** untuk prodi+angkatan mereka; lalu membuat **tagihan** + **item tagihan** (per jenis biaya) sesuai paket.

**Hasil:** Setiap mahasiswa yang terpilih punya **satu tagihan** untuk semester tersebut dengan item-item (UKT, UTS, SPI, dll.) dan nominal sesuai paket.

**Yang perlu diperhatikan:**
- **Jangan** generate dua kali untuk kombinasi semester + prodi + angkatan + kampus yang sama (bisa duplikat tagihan). Cek dulu status "sudah generate" di halaman tersebut.
- Pastikan **Paket Biaya** untuk prodi+angkatan yang dipilih **sudah ada dan berisi item**.
- Pastikan **Semester** yang dipilih **sudah ada** di master Semester.
- Setelah generate, mahasiswa bisa melihat tagihan di **portal** dan petugas bisa **input pembayaran**.

---

## Langkah 11: Aktivasi Lisensi (Agar Role Lain Bisa Akses)

**Menu:** Pengaturan → **Aktivasi Lisensi** (atau sistem akan mengarahkan ke sini jika belum ada lisensi).

- **Super Admin** tetap bisa akses penuh **tanpa** lisensi.
- **Admin Keuangan, Admin Akademik, Pimpinan, dan Mahasiswa** **tidak bisa** pakai sistem sampai lisensi diaktivasi.

**Yang Anda lakukan:**
- Dapatkan **kunci lisensi** dari pengembang / Super Admin yang generate.
- Masukkan kunci di form **Aktivasi Lisensi** → klik Aktivasi.
- Jika valid, semua role di atas bisa login dan akses sesuai hak masing-masing.

**Catatan:** Jika hanya dipakai di satu lingkungan dan hanya Super Admin yang dipakai, langkah ini bisa ditunda. Kalau ada Admin Keuangan / Mahasiswa yang harus login, aktivasi wajib.

---

## Langkah 12: Siap Dipakai

Setelah langkah 1–11:

- **Mahasiswa** bisa login dengan **username = NIM** dan password default (atau yang sudah direset), lalu melihat **Pembiayaan**, **Tagihan**, **Tunggakan**, **Riwayat Pembayaran** di portal.
- **Petugas** bisa:
  - **Input pembayaran** (Keuangan → Input Pembayaran → cari mahasiswa → isi nominal per item).
  - **Void** pembayaran jika salah input (hanya Admin Keuangan / Super Admin).
  - **Catat refund** jika mahasiswa punya lebih bayar (Admin Keuangan / Super Admin).
  - Lihat **Laporan** (ringkasan, mutasi, rekap kekurangan, export Excel).
- **Pengaturan Pembayaran** yang diisi di langkah 9 akan tampil di portal mahasiswa (rekening, WA) untuk panduan transfer.

---

## Ringkasan: Data dan Setting yang Wajib Diisi

| Yang wajib | Keterangan |
|------------|------------|
| Minimal 1 **Kampus** | Untuk mahasiswa dan pengaturan pembayaran. |
| Minimal 1 **Program Studi** | Untuk mahasiswa dan paket biaya. |
| Minimal 1 **Angkatan** | Untuk mahasiswa dan paket biaya. |
| Minimal 1 **Semester** (dan 1 ditandai aktif) | Untuk generate tagihan. |
| Minimal 1 **Jenis Biaya** | Untuk item paket dan tagihan. |
| **Mahasiswa** | Sesuai kebutuhan; NIM unik, prodi/angkatan/kampus harus sudah ada. |
| **Paket Biaya** (prodi+angkatan) + **Item** | Agar generate tagihan bisa jalan. |
| Minimal 1 **Pengaturan Pembayaran** | Rekening/WA untuk mahasiswa. |
| **Generate Tagihan** (per semester yang dipakai) | Agar ada tagihan yang bisa dibayar. |
| **Aktivasi Lisensi** | Jika selain Super Admin harus bisa login. |

---

## Larangan dan Hal yang Harus Dihindari

| Jangan | Alasannya |
|--------|-----------|
| **Hapus kampus/prodi/angkatan/semester/jenis biaya** yang sudah dipakai mahasiswa, paket biaya, atau tagihan. | Bisa error foreign key atau data tidak konsisten. |
| **Hapus mahasiswa** yang sudah punya tagihan atau pembayaran. | Riwayat keuangan jadi putus; laporan dan audit kacau. |
| **Hapus paket biaya** yang sudah dipakai untuk generate tagihan. | Referensi tagihan hilang. |
| **Generate tagihan** dua kali untuk kombinasi semester + prodi + angkatan + kampus yang sama. | Bisa duplikat tagihan. |
| **Kosongkan semester aktif** (tidak ada yang dicentang). | Generate tagihan dan laporan "semester berjalan" tidak jelas. |
| **Edit manual** nominal di tabel database tanpa lewat aplikasi. | State tagihan (terbayar, sisa) bisa tidak cocok dengan riwayat pembayaran; pakai **Rekonsiliasi** di Pemeliharaan Data jika sudah terlanjur. |
| **Lupa isi Pengaturan Pembayaran.** | Mahasiswa tidak tahu rekening/WA untuk transfer. |
| **Input mahasiswa** sebelum kampus, prodi, dan angkatan ada. | Form tidak bisa disimpan (harus pilih dari dropdown). |
| **Buat paket biaya** sebelum semester dan jenis biaya ada. | Item paket butuh jenis biaya; generate tagihan butuh semester. |
| **Share kunci lisensi** ke pihak yang tidak berwenang. | Siapa saja yang punya kunci bisa mengaktivasi instalasi. |

---

## Jika Ada Kesalahan Input

- **Salah nominal pembayaran / salah mahasiswa:** Jangan edit manual. Pakai **Void (batalkan)** transaksi pembayaran, lalu **input ulang** dengan data yang benar.
- **Mahasiswa salah prodi/kampus/angkatan:** **Edit** data mahasiswa dari Data Master → Mahasiswa (jangan hapus).
- **State tagihan tidak cocok dengan riwayat** (setelah restore DB atau insiden): Buka **Pemeliharaan Data Keuangan** → **Cek konsistensi** → jika ada selisih, pakai **Perbaiki state sekarang**.
- **Ingin mulai dari awal (kosongkan semua data):** Pakai **Reset Data** (menu Admin) dengan checklist; atau dari terminal: `php artisan stik_siap:reset-data`. **Backup dulu** sebelum reset.

---

## Referensi Singkat Menu

| Yang ingin Anda lakukan | Menu |
|-------------------------|------|
| Isi kampus, prodi, angkatan, semester, jenis biaya | Data Master → (nama master) |
| Input / edit mahasiswa | Data Master → Mahasiswa |
| Buat aturan biaya per prodi+angkatan | Data Master → Paket Biaya |
| Buat tagihan per mahasiswa per semester | Keuangan → Generate Tagihan |
| Input pembayaran | Keuangan → Input Pembayaran |
| Atur rekening & WA untuk mahasiswa | Pengaturan → Pengaturan Pembayaran |
| Aktivasi sistem untuk role lain | Pengaturan → Aktivasi Lisensi |
| Cek konsistensi angka terbayar | Pemeliharaan Data Keuangan → Cek konsistensi |
| Backup / restore database | Backup & Restore |
| Hapus data terpilih (mulai dari awal) | Reset Data |

---

Dokumen ini adalah **panduan setup** agar admin mampu memahami urutan input data dan setting yang diperlukan, serta hal-hal yang harus dihindari. Untuk konsep teknis dan fitur lengkap, lihat `docs/RINGKASAN_SISTEM.md`.
