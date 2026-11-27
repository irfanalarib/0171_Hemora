# Hemora

**Aplikasi Android Pengelolaan Layanan Donor Darah: Stok, Permintaan, Jadwal, dan Riwayat Donor**

| Informasi | Keterangan |
|---|---|
| Mata kuliah | Pengembangan Aplikasi Mobile (Project Akhir 2026) |
| Pembuat | Muhamad Irfan Al Arib (20240140171) |
| Platform | Android (Kotlin, Jetpack Compose), minSdk 26 (Android 8.0) |
| Repositori API | [0171_API_Hemora](https://github.com/USERNAME/0171_API_Hemora) |
| Dokumen acuan | [SRS Hemora](docs/SRS_Hemora_0171.pdf) |
| Status | Tahap perancangan selesai, implementasi belum dimulai |

---

## Daftar Isi

1. [Ringkasan](#1-ringkasan)
2. [Tujuan](#2-tujuan)
3. [Ruang Lingkup](#3-ruang-lingkup)
4. [Pengguna](#4-pengguna)
5. [Alur Singkat](#5-alur-singkat)
6. [Teknologi](#6-teknologi)
7. [Status Fitur terhadap Pedoman PAM 2026](#7-status-fitur-terhadap-pedoman-pam-2026)
8. [Status Fitur per Use Case](#8-status-fitur-per-use-case)
9. [Matriks CRUDS](#9-matriks-cruds)
10. [Entitas Basis Data](#10-entitas-basis-data)
11. [Struktur Proyek](#11-struktur-proyek)
12. [Cara Menjalankan](#12-cara-menjalankan)
13. [Progress Mingguan](#13-progress-mingguan)
14. [Rencana Pengerjaan](#14-rencana-pengerjaan)

---

## 1. Ringkasan

Hemora adalah aplikasi Android untuk mengelola layanan donor darah yang menghubungkan tiga peran: **Admin**, **Pendonor**, dan **Pemohon**. Pemohon (misalnya keluarga pasien) mengajukan permintaan darah beserta foto surat pengantar. Admin memverifikasi permintaan tersebut dan menyalurkan darah sesuai stok yang tersedia. Pendonor dapat membuat jadwal donor pribadi dan memantau riwayat serta status jeda donornya. Setiap donor yang berhasil dicatat Admin otomatis menambah stok darah.

## 2. Tujuan

1. Mencatat stok darah per golongan secara terpusat, dengan peringatan saat stok menipis.
2. Memudahkan Pemohon mengajukan dan memantau status permintaan darah.
3. Menyalurkan darah sesuai stok yang tersedia sehingga stok tidak pernah minus.
4. Membantu Pendonor mengatur jadwal donor dengan memperhatikan jeda antar-donor [60 hari] (acuan Permenkes Nomor 91 Tahun 2015).

## 3. Ruang Lingkup

**Termasuk**

- Register (Pendonor dan Pemohon), login, refresh token, dan logout
- Tiga peran: Admin, Pendonor, dan Pemohon, dengan hak akses berbeda
- Pengelolaan dan verifikasi data pendonor, serta pembaruan profil Pendonor dan Pemohon
- Pengelolaan stok darah per golongan dengan peringatan stok menipis
- Permintaan darah dengan surat pengantar dan penyaluran sesuai stok tersedia
- Jadwal donor kegiatan (Admin) dan jadwal donor pribadi (Pendonor)
- Riwayat donor yang otomatis memperbarui stok dan tanggal donor terakhir
- Penyimpanan dan pembacaan file lokal (surat pengantar dan dokumen pendonor)

**Tidak termasuk**

- Registrasi mandiri akun Admin (akun Admin dibuat langsung di database)
- Penyerahan darah secara fisik (dilakukan di luar aplikasi)
- Penggunaan Room Database

## 4. Pengguna

| Peran | Cara akun dibuat | Kegunaan |
|---|---|---|
| Admin | Dibuat langsung di database (tanpa daftar) | Kelola data pendonor, stok darah, jadwal donor, dan riwayat donor. Verifikasi pendonor dan permintaan darah |
| Pendonor | Daftar lewat aplikasi, golongan darah wajib, status awal `menunggu` | Perbarui profil, lihat permintaan aktif, buat jadwal donor pribadi, lihat riwayat dan status jeda |
| Pemohon | Daftar lewat aplikasi, golongan darah opsional, akun langsung aktif | Perbarui profil, lihat stok darah, kelola permintaan darah beserta surat pengantar |

Login memakai satu halaman untuk ketiga peran. Setelah masuk, aplikasi membaca peran dari token lalu menampilkan menu sesuai peran.

## 5. Alur Singkat

**Alur Pendonor**

1. Pendonor mendaftar. Status awal akun adalah **menunggu**.
2. Admin memverifikasi data pendonor sehingga statusnya menjadi **terverifikasi**.
3. Pendonor terverifikasi dapat membuat jadwal donor pribadi jika sudah melewati jeda [60 hari] sejak donor terakhir.
4. Setelah donor, Admin mencatat riwayat donor. Jika hasilnya **berhasil**, stok darah bertambah dan tanggal donor terakhir diperbarui.

**Alur Pemohon**

1. Pemohon membuat permintaan darah (golongan darah, jumlah kantong, lokasi, foto surat pengantar). Status awal adalah **menunggu**.
2. Admin menyetujui atau menolak permintaan. Saat disetujui, jumlah yang disalurkan adalah nilai terkecil antara jumlah yang diminta dan stok yang tersedia, lalu stok berkurang.
3. Setelah darah diserahkan di luar aplikasi, Admin menandai permintaan sebagai **terpenuhi**.
4. Pemohon melihat status permintaan beserta jumlah yang diminta dan yang disalurkan.

Alur status permintaan: `menunggu` → `disetujui` / `ditolak`, lalu `disetujui` → `terpenuhi`.

## 6. Teknologi

| Bagian | Teknologi |
|---|---|
| UI | Jetpack Compose, Material 3 |
| Navigasi | Navigation Compose |
| State management | MVVM (ViewModel + Repository) |
| REST client dan JSON | Retrofit + Gson (rencana) |
| Backend | Node.js + Express ([0171_API_Hemora](https://github.com/USERNAME/0171_API_Hemora)) |
| Database | MySQL |
| Autentikasi | JWT (access token [15 menit] dan refresh token [7 hari]), kata sandi di-hash dengan bcrypt |
| File storage | Penyimpanan lokal aplikasi (app-specific storage). Surat pengantar juga diunggah ke server agar dapat dibuka Admin |

## 7. Status Fitur terhadap Pedoman PAM 2026

Keterangan: ⏳ Belum dikerjakan · ✅ Selesai

| Ketentuan pedoman | Rencana implementasi | Status |
|---|---|---|
| Login, register, JWT, refresh token | Layar Login dan Register, endpoint autentikasi, refresh token otomatis saat access token kedaluwarsa | ⏳ |
| Otorisasi berbasis peran | Peran Admin, Pendonor, Pemohon. Menu berbeda per peran, backend menolak akses yang bukan haknya (403) | ⏳ |
| Navigasi multi-halaman | Navigation Compose | ⏳ |
| State management MVVM | ViewModel + Repository per fitur | ⏳ |
| Minimal 4 entitas saling terhubung | 7 tabel: admin, pendonor, pemohon, stok_darah, permintaan_darah, jadwal_donor, riwayat_donor | ⏳ |
| Minimal 5 fitur CRUDS penuh | Data Pendonor, Stok Darah, Permintaan Darah, Jadwal Donor, Riwayat Donor | ⏳ |
| REST API dan parsing JSON | Request ke backend sendiri dan parsing JSON | ⏳ |
| Database MySQL | MySQL, tanpa Room | ⏳ |
| File storage lokal | Simpan dan buka surat pengantar serta dokumen pendonor | ⏳ |
| UI hanya Jetpack Compose | Seluruh layar dibuat dengan Compose | ⏳ |
| README berisi status fitur dan progress mingguan | Dokumen ini | ✅ |

## 8. Status Fitur per Use Case

Keterangan: ⏳ Belum dikerjakan · ✅ Selesai

| Kode | Use Case | Aktor | Status |
|---|---|---|---|
| UC-01 | Register | Pendonor, Pemohon | ⏳ |
| UC-02 | Login | Admin, Pendonor, Pemohon | ⏳ |
| UC-03 | Logout | Admin, Pendonor, Pemohon | ⏳ |
| UC-04 | Perbarui Profil | Pendonor, Pemohon | ⏳ |
| UC-05 | Lihat dan Cari Permintaan Darah Aktif | Pendonor | ⏳ |
| UC-06 | Lihat dan Buat Jadwal Donor | Pendonor | ⏳ |
| UC-07 | Lihat Riwayat Donor dan Status Jeda | Pendonor | ⏳ |
| UC-08 | Lihat dan Cari Stok Darah | Pemohon | ⏳ |
| UC-09 | Kelola Permintaan Darah | Pemohon | ⏳ |
| UC-10 | Unggah Surat Pengantar | Pemohon | ⏳ |
| UC-11 | Simpan dan Buka File Lokal | Pendonor, Pemohon, Admin | ⏳ |
| UC-12 | Kelola Data Pendonor | Admin | ⏳ |
| UC-13 | Verifikasi Pendonor | Admin | ⏳ |
| UC-14 | Kelola Stok Darah | Admin | ⏳ |
| UC-15 | Verifikasi Permintaan Darah | Admin | ⏳ |
| UC-16 | Kelola Jadwal Donor | Admin | ⏳ |
| UC-17 | Kelola Riwayat Donor | Admin | ⏳ |

## 9. Matriks CRUDS

Rencana endpoint (prefix `/api`):

| Fitur | Create | Read | Update | Delete | Search |
|---|---|---|---|---|---|
| Data Pendonor | `POST /pendonor` | `GET /pendonor` | `PUT /pendonor/{id}` | `DELETE /pendonor/{id}` | `?q=` |
| Stok Darah | `POST /stok` | `GET /stok` | `PUT /stok/{id}` | `DELETE /stok/{id}` | `?q=` |
| Permintaan Darah | `POST /permintaan` | `GET /permintaan` | `PUT /permintaan/{id}` | `DELETE /permintaan/{id}` | `?q=`, `?status=` |
| Jadwal Donor | `POST /jadwal` | `GET /jadwal` | `PUT /jadwal/{id}` | `DELETE /jadwal/{id}` | `?q=` |
| Riwayat Donor | `POST /riwayat` | `GET /riwayat` | `PUT /riwayat/{id}` | `DELETE /riwayat/{id}` | `?q=` |

Endpoint tambahan: verifikasi pendonor, persetujuan, penolakan, dan penandaan terpenuhi untuk permintaan darah, serta unduh surat pengantar. Daftar lengkap ada di repositori API.

**Hak akses:**

- Admin mengelola Data Pendonor, Stok Darah, Jadwal Donor, dan Riwayat Donor.
- Pemohon hanya dapat mengubah atau menghapus permintaannya sendiri selama statusnya masih `menunggu`.
- Pendonor membuat jadwal pribadi dan hanya melihat riwayat miliknya.
- Data yang masih dirujuk tabel lain tidak dapat dihapus.

## 10. Entitas Basis Data

| Tabel | Isi utama | Relasi |
|---|---|---|
| `admin` | nama lengkap, email, kata sandi hash, no telepon | 1 admin memverifikasi banyak `permintaan_darah`, membuat banyak `jadwal_donor`, mencatat banyak `riwayat_donor` |
| `pendonor` | nama lengkap, email, kata sandi hash, golongan darah, status verifikasi, tanggal donor terakhir | 1 pendonor punya banyak `jadwal_donor` (pribadi) dan `riwayat_donor` |
| `pemohon` | nama lengkap, email, kata sandi hash, golongan darah (opsional), no telepon | 1 pemohon punya banyak `permintaan_darah` |
| `stok_darah` | golongan darah (unik), jumlah kantong, batas minimum | Tanpa foreign key, terhubung secara logika lewat golongan darah |
| `permintaan_darah` | golongan darah, jumlah kantong, jumlah disalurkan, lokasi, path surat, status | Milik 1 `pemohon`, diverifikasi 1 `admin` |
| `jadwal_donor` | lokasi, alamat, tanggal, waktu, kuota, status | Dibuat 1 `admin` (kegiatan) atau 1 `pendonor` (pribadi) |
| `riwayat_donor` | golongan darah, jumlah kantong, tanggal donor, status hasil | Milik 1 `pendonor`, dapat merujuk 1 `jadwal_donor` dan 1 `permintaan_darah` |

Refresh token bersifat stateless (JWT) sehingga tidak disimpan dalam tabel.

**Aturan Penyaluran Sesuai Stok:** jumlah disalurkan = nilai terkecil antara jumlah diminta dan stok tersedia. Proses ini dijalankan dalam satu transaksi. Jika stok golongan darah yang diminta 0, permintaan tidak dapat disetujui.

## 11. Struktur Proyek

Rencana struktur folder (MVVM):

```
0171_Hemora/
├── app/src/main/java/com/example/hemora/
│   ├── data/
│   │   ├── model/          # Data class request dan response
│   │   ├── remote/         # Retrofit, ApiService, interceptor token
│   │   ├── local/          # Penyimpanan token dan file lokal
│   │   └── repository/     # Repository per fitur
│   ├── ui/
│   │   ├── navigation/     # NavHost dan daftar rute
│   │   ├── theme/          # Warna, tipografi, tema
│   │   ├── components/     # Komponen Compose reusable
│   │   └── screen/         # auth, admin, pendonor, pemohon
│   └── viewmodel/          # ViewModel per fitur
├── docs/
│   ├── SRS_Hemora_0171.pdf
│   └── diagram/            # Use Case, Activity Diagram, ERD
└── README.md
```

## 12. Cara Menjalankan

Bagian ini akan dilengkapi setelah implementasi: kebutuhan (Android Studio, Node.js, MySQL), konfigurasi backend, alamat API, dan akun Admin untuk demo.

## 13. Progress Mingguan

Kewajiban pedoman: minimal satu progress per minggu, disertai commit.

### Minggu 1 (16 November 2026 – 22 November 2026)

- **Penentuan Tema dan Konsep Aplikasi** (`-`): Penentuan tema donor darah, peran pengguna, dan fitur utama.
- **Rancangan Awal Data dan Alur** (`-`): Perancangan entitas, relasi, peran, dan alur pengguna.

### Minggu 2 (23 November 2026 – 29 November 2026)

- **Penyusunan SRS** (`-`): Penyusunan SRS Bab 1 sampai Bab 6, Glossary, Analysis Models, dan Issues List.
- **Diagram Perancangan** (`-`): Use Case Diagram, Activity Diagram 3 swimlane, ERD, dan Kamus Data.
- **Inisialisasi Repositori** (`isi hash commit`): Pembuatan repositori, README, serta unggah SRS dan diagram.

## 14. Rencana Pengerjaan

Target: minimal 30 commit, dikerjakan bertahap. Pekerjaan backend ada di repositori [0171_API_Hemora](https://github.com/USERNAME/0171_API_Hemora).

Keterangan: ⏳ Belum dikerjakan · ✅ Selesai

| No | Kelompok pekerjaan | Perkiraan commit | Target minggu | Status |
|---|---|---|---|---|
| 1 | Setup proyek Android (minSdk 26, dependensi, izin internet) | 2 | 3 | ⏳ |
| 2 | Tema Material 3 dan komponen reusable | 2 | 3 | ⏳ |
| 3 | Model data, Retrofit, dan ApiService | 2 | 3 | ⏳ |
| 4 | Penyimpanan token dan interceptor refresh token | 2 | 3 | ⏳ |
| 5 | Navigation Compose dan rute per peran | 2 | 3 | ⏳ |
| 6 | Login dan Register (ViewModel, layar, validasi) | 3 | 4 | ⏳ |
| 7 | Perbarui profil dan logout | 2 | 4 | ⏳ |
| 8 | Dashboard per peran dan peringatan stok menipis | 2 | 4 | ⏳ |
| 9 | CRUDS Data Pendonor dan verifikasi pendonor | 3 | 4 | ⏳ |
| 10 | CRUDS Stok Darah | 2 | 4 | ⏳ |
| 11 | CRUDS Permintaan Darah dan unggah surat pengantar | 3 | 5 | ⏳ |
| 12 | Verifikasi permintaan darah (Admin) | 2 | 5 | ⏳ |
| 13 | Jadwal Donor (Admin) dan jadwal pribadi (Pendonor) | 3 | 5 | ⏳ |
| 14 | Riwayat Donor dan status jeda | 3 | 6 | ⏳ |
| 15 | Lihat dan cari permintaan aktif dan stok darah | 2 | 6 | ⏳ |
| 16 | File storage lokal (simpan dan buka file) | 2 | 6 | ⏳ |
| 17 | Validasi input, penanganan error, dan penyempurnaan UI | 2 | 6 | ⏳ |
| 18 | Pengujian dan perbaikan bug | 2 | 6 | ⏳ |
| 19 | README final: status fitur dan cara menjalankan | 1 | 7 | ⏳ |

**Jadwal minggu:**

| Minggu | Tanggal |
|---|---|
| 3 | 30 November – 6 Desember 2026 |
| 4 | 7 – 13 Desember 2026 |
| 5 | 14 – 20 Desember 2026 |
| 6 | 21 – 27 Desember 2026 |
| 7 | 28 – 29 Desember 2026 (batas pengumpulan) |
