# Panduan Lengkap: Instalasi Odoo 18 di Samping Odoo 17 pada Ubuntu

Dokumen ini memandu Anda untuk menginstal **Odoo 18 Community Edition** pada server Ubuntu yang sudah memiliki **Odoo 17**.

Kunci utama dari panduan ini adalah **isolasi** total antara kedua versi Odoo untuk menghindari konflik. Kita akan menggunakan:
* User sistem yang berbeda (`odoo18`).
* Versi Python yang berbeda (**Python 3.11** untuk Odoo 18) dalam *virtual environment* terpisah.
* Direktori file yang terpisah (`/opt/odoo18`).
* Port yang berbeda (`8018` & `8019`).
* Service systemd yang terpisah (`odoo18.service`).

---

##  Langkah 1: Persiapan Sistem ⚙️

Langkah pertama adalah memastikan semua *tools* yang dibutuhkan untuk kompilasi dan instalasi tersedia di sistem.

### 1.1. Update Paket
```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2. Install Build Tools
Paket-paket ini penting untuk meng-compile dependensi Python dari source code.

```bash
sudo apt install build-essential libpq-dev libldap2-dev libsasl2-dev -y
```

---

## Langkah 2: Instalasi Python 3.11 🐍

Odoo 18 dirancang untuk berjalan optimal di Python 3.11. Menggunakan versi ini akan menghindarkan kita dari error dependensi (`gevent`) yang sering muncul di Python 3.10.

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.11 python3.11-dev python3.11-venv -y
```
> **Catatan:** Perintah ini tidak akan mengganggu atau menghapus versi Python lain yang sudah ada di sistem Anda.

---

## Langkah 3: Buat User & Database Khusus 👤

Setiap versi Odoo harus memiliki user sistem dan user database sendiri.

### 3.1. Buat User Sistem `odoo18`
```bash
sudo adduser --system --home=/opt/odoo18 --group odoo18
```

### 3.2. Buat User Database `odoo18`
```bash
sudo su - postgres -c "createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo18"
```
> Anda akan diminta membuat password untuk user database `odoo18`. **Simpan password ini** karena akan digunakan pada file konfigurasi nanti.

---

## Langkah 4: Unduh & Siapkan File Odoo 18 📂

Kita akan mengunduh source code Odoo 18 dan menyiapkan struktur direktorinya.

### 4.1. Login sebagai User `odoo18`
```bash
sudo su - odoo18 -s /bin/bash
```

### 4.2. Unduh Source Code Odoo 18
```bash
git clone [https://www.github.com/odoo/odoo](https://www.github.com/odoo/odoo) --depth 1 --branch 18.0 --single-branch /opt/odoo18/odoo
```

### 4.3. Buat Direktori Tambahan
```bash
mkdir /opt/odoo18/custom_addons
mkdir /opt/odoo18/logs
touch /opt/odoo18/logs/odoo18.log
```
> Setelah ini, Anda masih tetap login sebagai user `odoo18` untuk langkah berikutnya.

---

## Langkah 5: Instalasi Dependensi Python di Virtual Environment 📦

Ini adalah langkah krusial dimana kita menggunakan **Python 3.11** yang sudah diinstal.

### 5.1. Buat dan Aktifkan Virtual Environment
```bash
# Pastikan Anda berada di direktori /opt/odoo18
cd /opt/odoo18

# Buat venv menggunakan python3.11
python3.11 -m venv venv

# Aktifkan venv
source venv/bin/activate
```
Prompt Anda sekarang akan diawali dengan `(venv)`.

### 5.2. Upgrade PIP dan Install Dependensi
```bash
# Upgrade alat packaging
pip install --upgrade pip wheel setuptools

# Install semua dependensi dari requirements.txt
pip install -r odoo/requirements.txt
```

### 5.3. Install Psycopg2
Paket ini wajib untuk koneksi ke database PostgreSQL.
```bash
pip install psycopg2-binary
```

### 5.4. Nonaktifkan Virtual Environment
```bash
deactivate
```

### 5.5. Kembali ke User Utama
```bash
exit
```

---

## Langkah 6: Konfigurasi Odoo 18 ⚙️

Kita akan membuat file konfigurasi yang memberitahu Odoo 18 untuk menggunakan port dan user database yang benar.

### 6.1. Buat File Konfigurasi
```bash
sudo nano /etc/odoo18.conf
```

### 6.2. Isi File Konfigurasi
Salin dan tempel konten di bawah ini, lalu **sesuaikan nilainya**.

```ini
[options]
; Password master untuk manajemen database dari UI
admin_passwd = GANTI_DENGAN_PASSWORD_MASTER_YANG_AMAN

; Konfigurasi Database
db_host = False
db_port = False
db_user = odoo18
db_password = GANTI_DENGAN_PASSWORD_DB_YANG_DIBUAT_TADI

; Path untuk semua addons
addons_path = /opt/odoo18/odoo/addons,/opt/odoo18/custom_addons

; ===== KONFIGURASI PORT (SANGAT PENTING!) =====
xmlrpc_port = 8018
longpolling_port = 8019

; Lokasi file log
logfile = /opt/odoo18/logs/odoo18.log

; Tampilkan list database di halaman login
list_db = True
```
> **PENTING:** Pastikan port `8018` dan `8019` tidak sedang digunakan oleh aplikasi lain.

### 6.3. Atur Kepemilikan & Hak Akses
```bash
sudo chown -R odoo18:odoo18 /opt/odoo18/
sudo chown odoo18:odoo18 /etc/odoo18.conf
sudo chmod 640 /etc/odoo18.conf
```

---

## Langkah 7: Membuat Odoo 18 Menjadi Service 🚀

Agar Odoo 18 berjalan otomatis saat server booting, kita buatkan service systemd.

### 7.1. Buat File Service
```bash
sudo nano /etc/systemd/system/odoo18.service
```

### 7.2. Isi File Service
```ini
[Unit]
Description=Odoo 18
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo18
PermissionsStartOnly=true
User=odoo18
Group=odoo18
ExecStart=/opt/odoo18/venv/bin/python3 /opt/odoo18/odoo/odoo-bin -c /etc/odoo18.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```
> Perhatikan `ExecStart` yang mengarah ke Python di dalam **venv** dan file konfigurasi `/etc/odoo18.conf`.

### 7.3. Aktifkan dan Jalankan Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable odoo18.service
sudo systemctl start odoo18.service
```

---

## Langkah 8: Verifikasi Akhir ✅

Pastikan semuanya berjalan sesuai rencana.

### 8.1. Cek Status Service
```bash
sudo systemctl status odoo18.service
```
Anda seharusnya melihat status `active (running)` berwarna hijau.

### 8.2. Cek Log File (jika ada masalah)
```bash
sudo tail -f /opt/odoo18/logs/odoo18.log
```

### 8.3. Akses Odoo 18 di Browser
Buka browser dan navigasi ke:
**`http://<alamat_ip_server_anda>:8018`**

**Selesai!** Anda kini memiliki Odoo 17 dan Odoo 18 yang berjalan berdampingan secara aman di satu server.
