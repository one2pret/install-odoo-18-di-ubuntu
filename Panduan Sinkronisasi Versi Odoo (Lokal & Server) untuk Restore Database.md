# Panduan Sinkronisasi Versi Odoo (Lokal & Server) untuk Restore Database

Tujuan dari panduan ini adalah untuk membuat source code Odoo di server Anda **100% identik** dengan versi yang terinstal di komputer lokal Anda. Ini adalah kunci untuk menghindari error saat memindahkan database.

Proses ini dibagi menjadi empat bagian utama:
1.  Menemukan versi tepat di Odoo lokal Anda.
2.  Mencari versi kode yang sesuai di GitHub.
3.  Mengupdate kode di server.
4.  Melakukan backup dan restore dengan benar.

---

## Bagian 1: 🔍 Menemukan Versi Tepat di Lokal (Windows)

Karena Odoo Anda diinstal dari file `.exe`, kita akan mencari tahu versi build-nya melalui antarmuka web.

### Langkah 1.1: Aktifkan Mode Developer
* Buka Odoo di browser lokal Anda.
* Masuk ke menu **Settings**.
* Scroll ke bawah dan klik **Activate the developer mode**.

### Langkah 1.2: Buka Menu "About"
* Setelah mode developer aktif, sebuah ikon **kutu/bug (🐞)** akan muncul di pojok kanan atas.
* Klik ikon tersebut, lalu pilih menu **About**.

### Langkah 1.3: Catat Tanggal Versi
* Sebuah popup akan muncul. Perhatikan versinya, contoh: `Odoo 18.0-20250401`.
* Catat tanggalnya. Dalam contoh ini, tanggalnya adalah **1 April 2025**. Inilah "tanda pengenal" versi Anda.

---

## Bagian 2: ☁️ Mencari Commit yang Sesuai di GitHub

Sekarang kita akan mencari kode yang sesuai dengan tanggal tersebut di repository resmi Odoo.

### Langkah 2.1: Buka Halaman Commit Odoo
Buka link berikut di browser Anda, yang merupakan riwayat commit untuk Odoo 18:
[https://github.com/odoo/odoo/commits/18.0](https://github.com/odoo/odoo/commits/18.0)

### Langkah 2.2: Cari Commit Berdasarkan Tanggal
* Gunakan fitur pencarian browser (Ctrl+F) atau scroll halaman untuk menemukan commit yang dibuat pada atau paling mendekati tanggal yang Anda catat (misal: `Apr 1, 2025`).

### Langkah 2.3: Salin Commit Hash Lengkap
* Setelah menemukan commit yang paling sesuai, klik ikon salin (📋) di sebelah kanan kode hash singkatnya untuk menyalin **commit hash** yang lengkap.

---

## Bagian 3: 🖥️ Mengupdate Kode di Server Ubuntu

Dengan *commit hash* di tangan, saatnya mengupdate server.

### Langkah 3.1: Login dan Masuk sebagai User Odoo
```bash
# Login ke server Anda
ssh ykp@alamat_server_anda

# Masuk sebagai user odoo18
sudo su - odoo18 -s /bin/bash
```

### Langkah 3.2: Masuk ke Direktori Odoo dan Fetch Riwayat
```bash
# Pindah ke direktori source code
cd /opt/odoo18/odoo

# Ambil semua data commit terbaru dari repository
git fetch origin
```

### Langkah 3.3: Checkout ke Commit Spesifik
Gunakan `git checkout` dengan *commit hash* yang Anda salin dari GitHub.

```bash
# Ganti dengan commit hash yang Anda dapatkan
git checkout <commit_hash_anda_dari_github>
```

### Langkah 3.4: Restart Service Odoo
Perubahan kode baru akan aktif setelah service di-restart.

```bash
# Keluar dari user odoo18
exit

# Restart service Odoo
sudo systemctl restart odoo18.service
```

---

## Bagian 4: ✅ Proses Backup dan Restore yang Benar

Setelah kode di lokal dan server sama persis, lakukan proses ini:

### Langkah 4.1: Buat Backup BARU dari Lokal
Buat file backup baru dari database Odoo di komputer lokal Anda.

### Langkah 4.2: Restore ke Server (via Command Line)
Pindahkan file backup baru tersebut ke server, dan lakukan restore menggunakan metode terminal (`psql`) yang sudah kita bahas sebelumnya untuk hasil yang paling andal.

Dengan mengikuti langkah-langkah ini, Anda memastikan tidak ada lagi konflik versi, dan proses restore database Anda akan berjalan dengan sukses.
