# Panduan Manual Backup dan Restore Database Odoo di Server

Dokumen ini menjelaskan prosedur standar untuk melakukan backup dan restore database Odoo langsung dari terminal server. Metode ini adalah cara yang paling **andal dan cepat**, terutama untuk database berukuran besar, karena menghindari masalah *timeout* yang sering terjadi pada antarmuka web.

---
## Langkah 1: Melakukan Backup Database 💾

Proses pertama adalah membuat salinan (backup) dari database yang sudah ada.

### 1. Masuk sebagai User Odoo
Untuk memastikan hak akses yang tepat, masuk sebagai user sistem yang didedikasikan untuk Odoo.

```bash
sudo su - odoo18 -s /bin/bash
```

### 2. Jalankan Perintah Backup
Gunakan perintah `pg_dump` untuk membuat file backup. Sebaiknya simpan file di lokasi sementara seperti `/tmp` untuk menghindari masalah izin penulisan.

```bash
# Format: pg_dump -U [user_db] -Fc [nama_db_sumber] > /lokasi/nama_file.dump
pg_dump -U odoo18 -Fc demo > /tmp/demo_backup.dump
```
* **`-Fc`**: Membuat backup dalam format *custom* yang terkompresi.
* **`demo`**: Nama database yang akan di-backup.
* **`/tmp/demo_backup.dump`**: File hasil backup yang akan dibuat.

---
## Langkah 2: Melakukan Restore Database 🔄

Setelah file backup siap, Anda dapat me-restore-nya ke database baru yang masih kosong.

### 1. Siapkan Database Tujuan
Pastikan database tujuan sudah ada dan dalam keadaan kosong. Jika database sudah ada dan berisi data, Anda perlu menghapusnya terlebih dahulu.

```bash
# (Opsional) Hapus database lama jika ada
dropdb -U odoo18 demo2

# Buat database baru yang kosong
createdb -U odoo18 -T template0 demo2
```

### 2. Jalankan Perintah Restore
Gunakan `pg_restore` untuk mengimpor data dari file backup ke database tujuan yang baru dibuat.

```bash
# Format: pg_restore -U [user_db] -d [nama_db_tujuan] /lokasi/file_backup.dump
pg_restore -U odoo18 -d demo2 /tmp/demo_backup.dump
```
* **`-d demo2`**: Menentukan `demo2` sebagai database tujuan.
* **`/tmp/demo_backup.dump`**: File backup yang akan dipulihkan.

Setelah proses ini selesai, database `demo2` akan menjadi salinan identik dari database `demo` saat di-backup.
