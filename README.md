# 🎫 Sistem Ticketing & Request — LAZ Al Azhar

Aplikasi web portal terpadu berbasis **Google Apps Script (GAS)** untuk mengelola permohonan layanan internal secara terpusat. Menggunakan Google Sheets sebagai basis data (*serverless*) dan terintegrasi dengan Gmail (MailApp) serta WhatsApp Gateway untuk notifikasi otomatis.

---

## 📋 Daftar Isi

1. [Gambaran Umum](#1-gambaran-umum)
2. [Fitur Lengkap](#2-fitur-lengkap)
3. [Arsitektur & Alur Kerja](#3-arsitektur--alur-kerja)
4. [Struktur Database (Google Sheets)](#4-struktur-database-google-sheets)
5. [Fitur Keamanan](#5-fitur-keamanan)
6. [Sistem Lampiran (Upload File)](#6-sistem-lampiran-upload-file)
7. [Cara Instalasi & Deploy](#7-cara-instalasi--deploy)
8. [Konfigurasi Awal Pasca-Deploy](#8-konfigurasi-awal-pasca-deploy)
9. [Panduan Penggunaan Lengkap](#9-panduan-penggunaan-lengkap)
   - [A. Mengajukan Tiket (Pemohon / User Publik)](#a-mengajukan-tiket-pemohon--user-publik)
   - [B. Melacak Status Tiket (Pemohon)](#b-melacak-status-tiket-pemohon)
   - [C. Login Admin](#c-login-admin)
   - [D. Memproses & Merespons Tiket (Admin)](#d-memproses--merespons-tiket-admin)
   - [E. Membuat Akun Admin Baru](#e-membuat-akun-admin-baru)
   - [F. Mengubah Password Admin](#f-mengubah-password-admin)
   - [G. Mengelola Unit Layanan (Kategori)](#g-mengelola-unit-layanan-kategori)
   - [H. Mengelola Kategori Pengajuan (Sub-Kategori)](#h-mengelola-kategori-pengajuan-sub-kategori)
   - [I. Mengatur Form Dinamis (Pengaturan Form)](#i-mengatur-form-dinamis-pengaturan-form)
   - [J. Menggunakan Lobby Display Monitor](#j-menggunakan-lobby-display-monitor)
   - [K. Mengekspor Data Tiket](#k-mengekspor-data-tiket)
   - [L. Konfigurasi Notifikasi Email & WhatsApp](#l-konfigurasi-notifikasi-email--whatsapp)
10. [Referensi Tipe Field Form Dinamis](#10-referensi-tipe-field-form-dinamis)
11. [Referensi Status Tiket](#11-referensi-status-tiket)
12. [Teknologi yang Digunakan](#12-teknologi-yang-digunakan)
13. [Pertanyaan Umum (FAQ)](#13-pertanyaan-umum-faq)

---

## 1. Gambaran Umum

Aplikasi ini menyediakan **3 antarmuka utama** dalam satu URL Web App:

| Antarmuka | URL | Keterangan |
|---|---|---|
| **Halaman Publik** | `/exec` (default) | Form pengajuan wizard 5 langkah + lacak tiket |
| **Admin Portal** | `/exec` (setelah login) | Dashboard, manajemen tiket, pengaturan sistem |
| **Lobby Display** | `/exec?view=display` | Monitor TV lobby, auto-refresh 15 detik |

---

## 2. Fitur Lengkap

### 👥 Portal Pengaju (Publik)
- **Form Wizard 5 Langkah** — Identitas Pemohon → Pilih Layanan → Detail Form → Lampiran → Review & Kirim
- **Logika Form Dinamis (Branching)** — Field tambahan muncul otomatis sesuai jenis layanan yang dipilih
- **Multi-Select Checkbox** — Admin dapat membuat field pilihan ganda tanpa coding
- **Upload Lampiran** — Hingga 3 file (JPG, PNG, PDF, DOCX), maks 4 MB/file, tersimpan di Google Drive
- **Copyable Ticket ID** — Salin ID Tiket langsung dari dialog sukses
- **Lacak Status Tiket** — Masukkan ID Tiket untuk cek status & catatan admin secara instan
- **QR Code Tiket** *(jika diaktifkan)* — QR Code ID Tiket ditampilkan di dialog sukses

### 🔧 Portal Admin
- **Visual Dashboard** — Grafik Chart.js: Donat status tiket, Batang bertumpuk YTD per kategori, Tren harian MTD
- **Daftar Tiket Lengkap** — Filter status, filter kategori, filter tanggal, pencarian teks bebas
- **Detail Tiket Modal** — Tabel identitas pemohon, rincian form kustom, dan lampiran file (tombol Buka File)
- **Approval & Response** — Ubah status tiket, tambah catatan, penugasan driver (kendaraan)
- **Notifikasi Otomatis** — Email & WhatsApp terkirim ke pemohon setiap kali status berubah
- **Pengaturan Form Dinamis** — Buat/ubah/hapus field form per jenis pengajuan tanpa ubah kode
- **Manajemen Unit & Kategori** — Tambah/edit/hapus Unit Layanan dan Sub-Kategori
- **Manajemen Admin** — Buat akun admin baru, ubah password, hapus akun
- **Konfigurasi Aplikasi** — Atur nama app, headline, email admin, WA Gateway
- **Ekspor Data** — Download tiket ke Excel (.xlsx) atau PDF (A4 Landscape)
- **Users / Klien** — Daftar seluruh pemohon, total request, kontak, terakhir aktif
- **Booking Otomatis** — Ruangan dan kendaraan langsung terdaftar ke jadwal booking saat tiket diajukan
- **Deteksi Konflik Jadwal** — Sistem menolak booking yang bentrok secara otomatis

### 📺 Lobby Display Monitor
- **Auto-Refresh 15 Detik** — Data selalu terkini tanpa reload manual
- **Statistik Real-Time** — Kartu Total, Open, On Proses, Pending, Selesai, Ditolak
- **Tabel Tiket Terbaru** — Tampilan ringkas tiket masuk dengan badge status berwarna
- **Jam Digital** — Jam & tanggal berjalan di pojok kanan atas
- **Distribusi Layanan** — Widget distribusi tiket per unit layanan
- **Ringkasan Status** — Progress ring penyelesaian + bar status

---

## 3. Arsitektur & Alur Kerja

```
   [ Browser Pengaju ]              [ Browser Admin ]          [ TV Lobby ]
            │                               │                       │
            ▼ (Form Submit)                 ▼ (Update/Config)       ▼ (?view=display)
┌───────────────────────────────────────────────────────────────────────────────┐
│                        Google Apps Script (Web App)                           │
│                                                                               │
│   doGet(e)  →  routing view: 'Index' (default) atau 'Display'                │
│   google.script.run  →  RPC async ke semua fungsi backend                    │
│   LockService  →  mutex lock untuk operasi tulis concurrent                  │
│   SHA-256 Hash  →  validasi login & session token                            │
└─────────────────────────────────────┬─────────────────────────────────────────┘
                                      │
                                      ▼ (Read / Write)
┌───────────────────────────────────────────────────────────────────────────────┐
│                          Google Sheets (Database)                             │
│                                                                               │
│   Master_Tickets        Ticket_Details        Ticket_Fields / Form_Settings  │
│   Users (hashed)        Categories            Sub_Categories                 │
│   Requesters            App_Config            Rooms / Room_Bookings          │
│   Vehicles / Drivers    Vehicle_Bookings                                     │
└─────────────────────────────────────┬─────────────────────────────────────────┘
                                      │
                                      ▼ (Trigger Notifikasi)
┌───────────────────────────────────────────────────────────────────────────────┐
│                           Layanan Eksternal                                   │
│                                                                               │
│   Gmail / MailApp  →  Email konfirmasi tiket baru & update status            │
│   WhatsApp Gateway API  →  Notifikasi WA ke pemohon                          │
│   Google Drive API  →  Simpan & share berkas lampiran                        │
└───────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Struktur Database (Google Sheets)

### `Master_Tickets`
Sheet utama penyimpanan semua tiket.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | Number | Unix timestamp pembuatan |
| `ticket_id` | String | ID unik (`TKT-XXXX-XXXX`) |
| `nama` | String | Nama lengkap pemohon |
| `kontak` | String | Email atau nomor WA pemohon |
| `divisi_pengaju` | String | Unit/divisi asal pemohon |
| `sub_divisi_pengaju` | String | Sub-divisi asal pemohon |
| `jabatan_pengaju` | String | Jabatan pemohon |
| `kategori` | String | Unit layanan yang dituju |
| `sub_layanan` | String | Jenis pengajuan spesifik |
| `deskripsi` | String | Deskripsi detail permohonan |
| `status` | String | `Open` / `On Proses` / `Pending` / `Solved` / `Reject` |
| `catatan` | String | Catatan balasan dari admin |
| `diupdate_oleh` | String | Nama admin terakhir memproses |
| `created_at` | DateTime | Waktu pengajuan tiket |
| `approved_by` | String | Admin yang menyetujui |
| `approved_at` | DateTime | Waktu persetujuan |
| `id_alat` | String | ID alat (khusus peminjaman) |
| `nama_alat` | String | Nama alat (khusus peminjaman) |
| `jumlah_pinjam` | Number | Jumlah unit pinjam |
| `tgl_mulai_pinjam` | Date | Tanggal mulai pinjam |
| `tgl_selesai_pinjam` | Date | Tanggal selesai pinjam |
| `tgl_kembali_aktual` | Date | Tanggal aktual pengembalian |
| `kondisi_kembali` | String | Kondisi saat dikembalikan |

### `Users`
Akun admin — password disimpan dalam bentuk hash SHA-256.

| Kolom | Keterangan |
|---|---|
| `username` | Login unik |
| `password` | Hash SHA-256 64-karakter hex |
| `fullname` | Nama lengkap admin |
| `role` | `Admin` atau role lain |

> **Kredensial Default:** `admin` / `Demo2026!`
> Ganti segera setelah instalasi pertama.

### `Ticket_Details`
Menyimpan nilai field form kustom (skema EAV).

| Kolom | Keterangan |
|---|---|
| `ticket_id` | Relasi ke `Master_Tickets` |
| `field_name` | Kode field (contoh: `nomor_sim`) |
| `field_label` | Label tampilan (contoh: `Nomor SIM`) |
| `field_value` | Nilai isian atau URL Drive (untuk lampiran) |

### `Requesters`
Daftar pemohon yang pernah mengajukan tiket.

| Kolom | Keterangan |
|---|---|
| `whatsapp` | Nomor WA pemohon |
| `email` | Alamat email pemohon |
| `nama` | Nama lengkap |
| `total_request` | Jumlah tiket yang pernah diajukan |
| `last_request` | Waktu pengajuan terakhir |

### `App_Config`
Pasangan key-value konfigurasi aplikasi.

| Key | Keterangan |
|---|---|
| `app_name` | Nama aplikasi di navbar |
| `public_headline` | Judul hero halaman publik |
| `public_subheadline` | Sub-judul hero |
| `popup_success_message` | Teks di dialog sukses |
| `admin_email` | Email penerima notifikasi tiket baru |
| `finance_admin_email` | Email admin keuangan (opsional) |
| `enable_email` | `true` / `false` |
| `enable_whatsapp` | `true` / `false` |
| `wa_gateway_url` | URL endpoint WA Gateway |
| `wa_api_key` | API Key WA Gateway |
| `maintenance` | `true` untuk mode maintenance |

### Sheet Pendukung Lainnya

| Sheet | Fungsi |
|---|---|
| `Categories` | Master unit layanan (IT Support, GA, Humas, dll) |
| `Sub_Categories` | Daftar jenis pengajuan per unit |
| `Ticket_Fields` | Template field form kustom (sistem lama) |
| `Form_Settings` | Template field form kustom (sistem baru, lebih lengkap) |
| `Rooms` | Master data ruangan |
| `Room_Bookings` | Jadwal pemakaian ruangan |
| `Vehicles` | Master data kendaraan |
| `Drivers` | Master data driver |
| `Vehicle_Bookings` | Jadwal pemakaian kendaraan |

---

## 5. Fitur Keamanan

### Enkripsi Password SHA-256
Password admin tidak pernah disimpan sebagai teks biasa. Alur hash:
1. Admin input password → dikirim ke fungsi `hashPassword()` di server
2. GAS `Utilities.computeDigest(SHA_256, ...)` menghasilkan digest 64-karakter hex
3. Saat login, hash input dibandingkan dengan hash tersimpan — password asli tidak pernah diekspos

### Session Token & Auto-Logout
- Setiap login sukses menghasilkan token sesi acak unik
- Token disimpan di `sessionStorage` browser (hilang saat tab ditutup)
- Timer aktivitas memantau mouse, keyboard, dan klik
- **Tidak ada aktivitas selama 20 menit → logout otomatis** ke halaman publik
- Token diverifikasi ulang di setiap pemanggilan fungsi backend sensitif

### LockService (Mutex)
Semua operasi tulis ke Google Sheets menggunakan `LockService.getScriptLock()` untuk mencegah race condition ketika banyak pengguna submit tiket secara bersamaan.

---

## 6. Sistem Lampiran (Upload File)

1. Pengguna memilih file di wizard Step 4 (drag & drop atau klik)
2. File dibaca sebagai Base64 di browser, dikirim ke backend via `google.script.run`
3. Backend mendekode Base64 → Blob → disimpan ke folder **`Ticket_Attachments`** di Google Drive
4. File diberi akses **"Anyone with link can view"** secara otomatis
5. URL file disimpan ke sheet `Ticket_Details` dengan `field_name = 'attachment'`
6. Di modal detail admin, URL file muncul sebagai tombol **"Buka File"** yang bisa diklik
7. Setelah semua file terunggah, email terpisah dikirim ke admin berisi daftar lampiran dan link-nya

**Batasan upload:**
- Maks 3 file per tiket
- Maks 4 MB per file
- Format: JPG, JPEG, PNG, PDF, DOC, DOCX

---

## 7. Cara Instalasi & Deploy

### Langkah 1 — Siapkan Google Sheets

1. Buka [Google Drive](https://drive.google.com) dan buat Spreadsheet baru
2. Beri nama, misalnya: **`Sistem Ticketing LAZ Al Azhar`**
3. Dari menu atas Spreadsheet, klik **Ekstensi → Apps Script**
4. Editor Apps Script akan terbuka di tab baru

### Langkah 2 — Buat File di Apps Script

Di editor Apps Script, hapus semua konten yang ada lalu buat 3 file berikut:

**File 1 — `Code.gs`** (sudah ada secara default, ganti isinya)
- Salin seluruh isi file `Code.txt` ke dalamnya

**File 2 — `Index.html`** (klik ikon `+` di panel kiri → pilih HTML)
- Beri nama: `Index`
- Salin seluruh isi file `Index.txt` ke dalamnya

**File 3 — `Display.html`** (klik ikon `+` lagi → pilih HTML)
- Beri nama: `Display`
- Salin seluruh isi file `Display.txt` ke dalamnya

> ⚠️ Nama file harus **persis** seperti di atas (case-sensitive): `Code`, `Index`, `Display`

### Langkah 3 — Inisialisasi Database

1. Di dropdown fungsi di toolbar atas editor, pilih **`setupDatabase`**
2. Klik tombol **▶ Run**
3. Pop-up izin OAuth akan muncul → klik **Review permissions**
4. Pilih akun Google Anda → klik **Allow**
5. Tunggu hingga eksekusi selesai (status bar bawah menampilkan "Execution completed")
6. Buka kembali tab Spreadsheet — akan muncul belasan sheet baru secara otomatis

### Langkah 4 — Deploy sebagai Web App

1. Di editor Apps Script, klik tombol **Deploy** (kanan atas) → **New deployment**
2. Klik ikon ⚙️ di samping "Select type" → pilih **Web app**
3. Isi form deployment:
   - **Description**: `v1.0 - Initial Release` (bebas)
   - **Execute as**: `Me (nama.email@gmail.com)`
   - **Who has access**: `Anyone`
4. Klik **Deploy**
5. Salin **Web App URL** yang tampil — ini adalah URL utama aplikasi Anda

> 💡 **Setiap kali ada perubahan kode**, Anda harus **New Deployment** lagi (bukan Manage Deployments) agar perubahan aktif untuk pengguna.

---

## 8. Konfigurasi Awal Pasca-Deploy

Lakukan langkah-langkah ini setelah pertama kali deploy:

### 8.1 Login pertama kali
1. Buka Web App URL di browser
2. Klik tombol **Admin Login** di navbar kanan atas
3. Masukkan:
   - Username: `admin`
   - Password: `Demo2026!`
4. Klik **Login System**

### 8.2 Ganti password default (WAJIB)
1. Setelah login, klik menu **Manajemen Admin** di sidebar kiri
2. Temukan akun `admin` di tabel
3. Klik tombol pensil (✏️) di kolom Aksi
4. Ubah password ke kata sandi yang kuat
5. Klik **Simpan**

### 8.3 Atur email notifikasi
1. Klik menu **Konfigurasi** di sidebar
2. Ubah nilai `admin_email` → isi dengan email yang akan menerima notifikasi tiket masuk
3. Ubah `enable_email` → `true`
4. Klik **Simpan Konfigurasi**

### 8.4 Atur nama & tampilan aplikasi
1. Masih di menu **Konfigurasi**
2. Ubah `app_name` → nama sistem Anda
3. Ubah `public_headline` → judul hero halaman publik
4. Ubah `public_subheadline` → sub-judul halaman publik
5. Klik **Simpan Konfigurasi**

### 8.5 Aktifkan WhatsApp (Opsional)
1. Pastikan Anda memiliki akun WA Gateway (contoh: Fonnte, WablasTech, dll)
2. Di menu **Konfigurasi**, isi:
   - `wa_gateway_url` → URL endpoint API gateway Anda
   - `wa_api_key` → API Key dari provider gateway
   - `enable_whatsapp` → `true`
3. Klik **Simpan Konfigurasi**

---

## 9. Panduan Penggunaan Lengkap

---

### A. Mengajukan Tiket (Pemohon / User Publik)

Pemohon tidak perlu login. Cukup buka URL Web App.

**Langkah 1 — Identitas Pemohon**
1. Isi **Nama Lengkap**
2. Isi **Kontak** — boleh email atau nomor WhatsApp (format: `08xxxxxxxxxx`)
3. Pilih **Divisi** dengan mengklik salah satu kartu (Sekretariat, LAZ, Keuangan, Wakaf)
4. Pilih **Sub-Divisi** yang muncul setelah memilih divisi
5. Pilih **Jabatan** dari dropdown
6. Klik **Lanjut →**

**Langkah 2 — Pilih Layanan**
1. Pilih **Unit** dari dropdown (IT Support, General Affair, Humas, dll)
2. Pilih **Kategori / Jenis Pengajuan** yang muncul setelah unit dipilih
3. Isi **Deskripsi Singkat** — jelaskan masalah atau permohonan Anda
4. Klik **Lanjut →**

**Langkah 3 — Detail Pengajuan**
- Form tambahan muncul sesuai jenis layanan yang dipilih
- Isi semua field yang bertanda bintang merah (`*`)
- Untuk **Peminjaman Alat**: pilih alat dari katalog, isi jumlah dan tanggal
- Untuk **Penggunaan Ruangan**: pilih ruangan, isi tanggal & jam (sistem cek konflik otomatis)
- Untuk **Penggunaan Kendaraan**: pilih kendaraan, isi tujuan & waktu
- Klik **Lanjut →**

**Langkah 4 — Lampiran (Opsional)**
1. Klik area unggah atau seret file ke zona unggah
2. Pilih hingga 3 file (JPG/PNG/PDF/DOCX, maks 4 MB/file)
3. Preview file akan muncul dengan tombol hapus (×) di pojok
4. Klik **Lanjut →**

**Langkah 5 — Review & Kirim**
1. Periksa ringkasan seluruh data yang Anda isi
2. Jika ada yang salah, klik **← Kembali** untuk koreksi
3. Jika sudah benar, klik **Kirim Permohonan**
4. Dialog sukses akan muncul berisi **ID Tiket** (format `TKT-XXXX-XXXX`)
5. **Salin dan simpan ID Tiket** — Anda akan butuhkan untuk melacak status

> 📧 Setelah submit berhasil, notifikasi email/WhatsApp konfirmasi dikirim ke kontak Anda dan ke admin secara otomatis.

---

### B. Melacak Status Tiket (Pemohon)

1. Buka URL Web App
2. Di bagian kanan halaman, temukan kotak **"Lacak Status"**
3. Ketik ID Tiket Anda (contoh: `TKT-4521-8834`)
4. Klik tombol **Cek**
5. Informasi tiket akan muncul:
   - Status terkini (Open / On Proses / Pending / Solved / Reject)
   - Catatan dari admin (jika ada)
   - Tanggal pengajuan

---

### C. Login Admin

1. Buka URL Web App
2. Klik tombol **Admin Login** di navbar
3. Masukkan **Username** dan **Password**
4. Klik **Login System**
5. Setelah berhasil, Anda akan masuk ke **Dashboard Admin**

> 🔒 Sesi admin aktif selama 20 menit sejak aktivitas terakhir. Sistem akan logout otomatis jika tidak ada aktivitas.

---

### D. Memproses & Merespons Tiket (Admin)

#### Melihat Daftar Tiket
1. Klik menu **Ticketing** di sidebar kiri
2. Tabel daftar tiket akan tampil dengan kolom: ID Tiket, Informasi Pemohon, Unit & Kategori, Status, Tanggal, Aksi
3. Gunakan filter di atas tabel untuk menyaring:
   - **Filter Status** — pilih Open / On Proses / Pending / Solved / Reject
   - **Filter Kategori** — pilih unit layanan tertentu
   - **Rentang Tanggal** — filter berdasarkan tanggal pengajuan
   - **Kotak Pencarian** — cari berdasarkan nama, ID tiket, atau deskripsi

#### Melihat Detail Tiket
1. Di baris tiket yang diinginkan, klik tombol **⋮** (tiga titik) di kolom Aksi
2. Pilih **Detail**
3. Modal detail akan terbuka berisi:
   - Identitas lengkap pemohon (nama, kontak, divisi, jabatan)
   - Detail pengajuan (unit, kategori, deskripsi)
   - Rincian form kustom (semua field yang diisi pemohon)
   - Lampiran file (tombol **Buka File** untuk membuka di tab baru)

#### Mengubah Status & Memberi Tanggapan
1. Di modal detail tiket, temukan bagian **Update Status**
2. Pilih status baru dari dropdown:
   - `Open` → Tiket baru masuk, belum diproses
   - `On Proses` → Sedang dikerjakan
   - `Pending` → Menunggu informasi/tindakan dari pemohon
   - `Solved` → Selesai dikerjakan
   - `Reject` → Ditolak
3. Isi **Catatan / Tanggapan** — teks ini akan terkirim ke pemohon via email & WA
4. Untuk tiket **Penggunaan Kendaraan**: pilih driver dari dropdown sebelum mengubah status
5. Klik **Simpan Perubahan**
6. Notifikasi otomatis terkirim ke pemohon

---

### E. Membuat Akun Admin Baru

1. Klik menu **Manajemen Admin** di sidebar kiri
2. Klik tombol **+ Tambah Admin** di kanan atas
3. Isi form:
   - **Nama Lengkap**: Nama admin baru
   - **Username**: ID login unik (huruf kecil, tanpa spasi)
   - **Password**: Kata sandi (minimal 6 karakter)
   - **Role**: Pilih `Admin`
4. Klik **Simpan**
5. Akun baru akan muncul di tabel dan password otomatis di-hash SHA-256

> ℹ️ Admin baru dapat langsung login menggunakan username dan password yang baru dibuat.

---

### F. Mengubah Password Admin

#### Mengubah password sendiri
1. Klik menu **Manajemen Admin**
2. Temukan baris akun Anda sendiri
3. Klik ikon pensil (✏️)
4. Isi field **Password Baru**
5. Klik **Simpan**

#### Admin mengubah password admin lain
1. Langkah sama seperti di atas, pilih akun lain
2. Hanya Admin dengan role `Admin` yang dapat mengubah akun lain

---

### G. Mengelola Unit Layanan (Kategori)

Unit layanan adalah kelompok utama seperti IT Support, General Affair, Humas.

**Menambah Unit Baru**
1. Klik menu **Unit** di sidebar
2. Klik **+ Tambah Unit**
3. Isi Nama Unit, Deskripsi, dan Nama Sheet (tanpa spasi, contoh: `Cat_IT_Baru`)
4. Klik **Simpan**

**Mengedit Unit**
1. Klik ikon pensil (✏️) di kartu unit yang ingin diubah
2. Ubah informasi yang diinginkan
3. Klik **Simpan**

> ⚠️ Jangan mengubah Nama Sheet unit yang sudah memiliki data tiket — ini akan memutus relasi data.

---

### H. Mengelola Kategori Pengajuan (Sub-Kategori)

Sub-kategori adalah jenis pengajuan spesifik di bawah sebuah unit, contoh: "Peminjaman Alat" di bawah "General Affair".

**Menambah Kategori Baru**
1. Klik menu **Kategori** di sidebar
2. Klik **+ Tambah Kategori**
3. Pilih **Unit Induk** dari dropdown
4. Isi **Nama Kategori** (contoh: `Penggantian Toner Printer`)
5. Klik **Simpan**

**Menonaktifkan Kategori**
- Klik ikon hapus (🗑️) di baris kategori yang ingin dinonaktifkan
- Kategori yang sudah memiliki data tiket akan dinonaktifkan, bukan dihapus permanen

---

### I. Mengatur Form Dinamis (Pengaturan Form)

Form dinamis memungkinkan admin menambahkan field input tambahan pada form pengajuan tanpa menyentuh kode sama sekali.

**Membuka Halaman Pengaturan Form**
1. Klik menu **Pengaturan Form** di sidebar

**Menambah Field Baru**
1. Klik **+ Tambah Field**
2. Isi form di modal yang muncul:

   | Kolom | Penjelasan |
   |---|---|
   | **Sub-Kategori Layanan** | Jenis pengajuan yang akan ditambah field-nya |
   | **Nama Field (ID Unik)** | Kode field, huruf kecil & underscore (contoh: `nama_vendor`) |
   | **Label yang Ditampilkan** | Teks label yang dilihat pemohon (contoh: `Nama Vendor`) |
   | **Tipe Field** | Lihat tabel tipe field di [Bab 10](#10-referensi-tipe-field-form-dinamis) |
   | **Bagian Form** | Step 3 (Detail) atau Step 4 (Lampiran) |
   | **Opsi Pilihan** | Untuk dropdown/radio/multi-pilih: isi opsi pisah koma (contoh: `Opsi A,Opsi B,Opsi C`) |
   | **Urutan Tampil** | Angka urut tampilnya field di form (1, 2, 3, ...) |
   | **Wajib Diisi** | Centang jika field harus diisi sebelum bisa lanjut |

3. **Logika Kondisional (Opsional)** — field hanya muncul jika field lain bernilai tertentu:
   - **Field Acuan**: Nama ID field yang menjadi syarat (contoh: `jenis_ajuan`)
   - **Nilai Pemicu**: Nilai dari field acuan yang memicu kemunculan field ini (contoh: `Buat Baru`)
4. Klik **Simpan Field**

**Mengedit Field**
1. Klik ikon pensil (✏️) di baris field yang ingin diubah
2. Ubah konfigurasi yang diinginkan (Nama Field dan Sub-Kategori tidak bisa diubah saat edit)
3. Klik **Simpan Field**

**Menghapus Field**
1. Klik ikon hapus (🗑️) di baris field
2. Konfirmasi penghapusan

> ✅ Perubahan form dinamis langsung aktif tanpa perlu deploy ulang.

---

### J. Menggunakan Lobby Display Monitor

Monitor lobby adalah tampilan khusus untuk dipasang di layar TV di area tunggu.

**Cara Mengakses**
- Dari halaman publik: klik tombol **Live Monitor** di navbar
- Atau buka langsung: `[URL Web App]?view=display`

**Fitur tampilan:**
- Statistik tiket real-time (Total, Open, On Proses, Pending, Selesai, Ditolak)
- Tabel tiket terbaru masuk dengan status berwarna
- Ringkasan status dengan progress ring
- Distribusi tiket per unit layanan
- Jam digital berjalan
- Auto-refresh setiap **15 detik**

**Tips setup di TV:**
1. Buka URL display di browser TV atau Chromecast
2. Tekan `F11` untuk mode fullscreen
3. Nonaktifkan screen saver / sleep mode di TV/komputer
4. Tidak perlu login — halaman ini akses publik

---

### K. Mengekspor Data Tiket

1. Klik menu **Ticketing** di sidebar
2. Gunakan filter untuk menyeleksi tiket yang ingin diekspor (opsional)
3. Klik tombol **Export** di kanan atas tabel
4. Pilih format:
   - **Excel (.xlsx)** — untuk diolah lebih lanjut di spreadsheet
   - **PDF** — untuk laporan cetak A4 Landscape
5. File akan otomatis terunduh ke komputer Anda

---

### L. Konfigurasi Notifikasi Email & WhatsApp

#### Email (Gmail / MailApp)
1. Klik menu **Konfigurasi**
2. Set `enable_email` → `true`
3. Isi `admin_email` → email yang menerima notifikasi tiket baru
4. Isi `finance_admin_email` → email admin keuangan (untuk tiket pengadaan barang > Rp 5 juta)
5. Klik **Simpan**

Email dikirim otomatis pada kejadian:
- Tiket baru masuk (ke admin & ke pemohon)
- Status tiket diubah (ke pemohon)
- Lampiran berhasil diunggah (ringkasan ke admin)

#### WhatsApp Gateway
1. Daftar ke provider WA Gateway (contoh: [Fonnte.com](https://fonnte.com), [WablasTech](https://wablas.com))
2. Dapatkan URL endpoint dan API Key dari dashboard provider
3. Di menu **Konfigurasi**, isi:
   - `wa_gateway_url` → URL API dari provider
   - `wa_api_key` → API Key Anda
   - `enable_whatsapp` → `true`
4. Klik **Simpan**

Format nomor kontak yang diterima: `08xxxxxxxxxx` atau `628xxxxxxxxxx` — sistem normalisasi otomatis ke format `628...`.

---

## 10. Referensi Tipe Field Form Dinamis

| Tipe | Nama di UI | Keterangan | Kolom "Opsi" |
|---|---|---|---|
| `text` | Teks Pendek | Input satu baris | Tidak perlu |
| `textarea` | Teks Panjang (Paragraf) | Input multi-baris | Tidak perlu |
| `number` | Angka | Input numerik | Tidak perlu |
| `date` | Tanggal | Date picker | Tidak perlu |
| `time` | Waktu | Time picker | Tidak perlu |
| `select` | Pilihan (Dropdown) | Dropdown satu pilihan | `Opsi A,Opsi B,Opsi C` |
| `radio` | Pilihan (Radio Button) | Tombol pilih satu | `Opsi A,Opsi B` |
| `checkbox` | Centang (Checkbox) | Satu centang persetujuan | Teks persetujuan (opsional) |
| `multi_checkbox` | Centang Banyak (Multi-Pilih) | Pilih lebih dari satu opsi | `Opsi A,Opsi B,Opsi C` |
| `info` | ℹ️ Info / Peraturan | Teks baca-saja, tidak ada input | Isi teks peraturan lengkap |

> **Catatan `multi_checkbox`:** Nilai yang dipilih disimpan sebagai teks dipisah koma (contoh: `"Opsi A, Opsi C"`) dan otomatis muncul di notifikasi email & WhatsApp.

---

## 11. Referensi Status Tiket

| Status | Warna | Artinya | Tindakan Selanjutnya |
|---|---|---|---|
| `Open` | 🟡 Kuning | Tiket baru, belum diproses | Admin perlu segera merespons |
| `On Proses` | 🔵 Biru | Sedang dikerjakan admin | Tunggu penyelesaian |
| `Pending` | ⚫ Abu-abu | Menunggu info/tindakan dari pemohon | Pemohon perlu memberikan info tambahan |
| `Solved` | 🟢 Hijau | Selesai dikerjakan | Tiket ditutup |
| `Reject` | 🔴 Merah | Ditolak admin | Pemohon dapat mengajukan ulang jika diperlukan |

---

## 12. Teknologi yang Digunakan

| Komponen | Teknologi |
|---|---|
| Backend & Database | Google Apps Script + Google Sheets API |
| Frontend Form & Admin | Bootstrap 5.3 |
| Frontend Display Monitor | Tailwind CSS |
| Grafik & Chart | Chart.js 4.x |
| Notifikasi Email | Google MailApp Service |
| Notifikasi WhatsApp | REST API WA Gateway (Fonnte / Wablas / lainnya) |
| Penyimpanan File | Google Drive API |
| Ikon | Font Awesome 6.4 |
| Pop-up Dialog | SweetAlert2 |
| QR Code | QRCode.js |

---

## 13. Pertanyaan Umum (FAQ)

**Q: Apakah pemohon perlu akun Google untuk mengajukan tiket?**
A: Tidak. Halaman pengajuan sepenuhnya publik, pemohon hanya perlu mengisi nama dan kontak.

**Q: Bagaimana jika saya lupa password admin?**
A: Buka sheet `Users` di Google Spreadsheet secara langsung. Hapus baris akun tersebut. Jalankan ulang fungsi `setupDatabase` untuk membuat ulang akun `admin` default. Kemudian segera ganti password default.

**Q: Perubahan konfigurasi (nama app, email) langsung aktif?**
A: Ya, perubahan di menu Konfigurasi langsung tersimpan ke Google Sheets dan aktif saat halaman di-refresh. Tidak perlu deploy ulang.

**Q: Perubahan form dinamis (tambah/edit field) langsung aktif?**
A: Ya, langsung aktif tanpa deploy ulang karena data field dibaca dari Google Sheets setiap kali form dimuat.

**Q: Apakah ada batasan jumlah tiket?**
A: Tidak ada batasan dari sisi aplikasi. Google Sheets mendukung hingga 10 juta sel per spreadsheet. Untuk penggunaan normal organisasi, ini lebih dari cukup.

**Q: Bagaimana cara update/upgrade aplikasi setelah ada perubahan kode?**
A: Di editor Apps Script, perbarui kode yang berubah, lalu klik **Deploy → New deployment** untuk membuat versi baru. URL lama tetap aktif, URL baru juga tersedia. Disarankan menggunakan satu URL stabil dengan mengelola versi di **Manage deployments**.

**Q: Apakah notifikasi WhatsApp gratis?**
A: Bergantung pada provider WA Gateway yang Anda gunakan. Sebagian besar provider menawarkan paket berbayar. Cek halaman harga provider masing-masing.

**Q: Bagaimana cara menonaktifkan sementara pengajuan baru (maintenance)?**
A: Di menu Konfigurasi, ubah nilai `maintenance` menjadi `true`. Halaman publik akan menampilkan pesan maintenance.

**Q: Kenapa tombol Live Monitor membuka halaman kosong?**
A: Pastikan kode sudah di-deploy ulang setelah update terakhir. URL display hanya bisa diakses dari URL `/exec` yang valid, bukan dari preview editor Apps Script.

---

*Dibuat & dikelola oleh @gbrnmewing — LAZ Al Azhar Internal Ticketing System*
*Versi Dokumentasi: Juni 2026*
#   t i k e t i n g  
 