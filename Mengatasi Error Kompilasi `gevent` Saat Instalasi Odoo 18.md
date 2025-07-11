# Mengatasi Error Kompilasi `gevent` Saat Instalasi Odoo 18

Panduan ini menjelaskan cara mengatasi error kompilasi paket `gevent` yang umum terjadi saat menginstal Odoo 18 menggunakan Python 3.10.

---

## 🔍 Akar Masalah: Inkompatibilitas Versi

Saat Anda menjalankan `pip install`, Anda mungkin melihat error berikut:

```
Cython.Compiler.Errors.CompileError: src/gevent/libev/corecext.pyx
undeclared name not builtin: long
```

Ini terjadi karena **`gevent` versi lama** yang dibutuhkan oleh Odoo 18 tidak sepenuhnya kompatibel dengan **Python 3.10** ke atas. Solusinya adalah menggunakan versi Python yang direkomendasikan untuk Odoo 18, yaitu **Python 3.11**.

---

## ✅ Kenapa Solusi Ini Aman?

Langkah-langkah di bawah ini **100% aman** dan **tidak akan mengganggu** instalasi Odoo 17 Anda. Alasannya, semua perubahan hanya dilakukan di dalam lingkungan Odoo 18 yang **sepenuhnya terisolasi**:
* Anda hanya bekerja di dalam direktori `/opt/odoo18`.
* Anda hanya memodifikasi *virtual environment* (`venv`) milik Odoo 18.

---

## 🛠️ Solusi: Upgrade Lingkungan Odoo 18 ke Python 3.11

Kita akan mengganti "mesin" Python di lingkungan Odoo 18 tanpa menyentuh bagian lain dari server Anda.

### Langkah 1: Install Python 3.11

Pertama, tambahkan Python 3.11 ke server Anda. Perintah ini tidak akan menggantikan Python versi lama.

```bash
# Tambahkan repository PPA deadsnakes
sudo add-apt-repository ppa:deadsnakes/ppa -y

# Update daftar paket dan install Python 3.11
sudo apt update
sudo apt install python3.11 python3.11-dev python3.11-venv -y
```

### Langkah 2: Buat Ulang Virtual Environment Odoo 18

Sekarang, kita hapus lingkungan lama yang bermasalah dan buat yang baru menggunakan Python 3.11.

> **Peringatan:** Pastikan Anda menjalankan perintah ini dari dalam user `odoo18`.

```bash
# Pindah ke direktori kerja Odoo 18
cd /opt/odoo18

# Hapus virtual environment lama
rm -rf venv

# Buat venv baru menggunakan python3.11
python3.11 -m venv venv

# Aktifkan venv baru
source venv/bin/activate

# Upgrade alat instalasi di dalam venv
pip install --upgrade pip wheel setuptools
```

### Langkah 3: Install Kembali Dependensi Odoo 18

Dengan lingkungan baru yang aktif, jalankan kembali instalasi dependensi.

```bash
pip install -r odoo/requirements.txt
```

➡️ Instalasi sekarang seharusnya berjalan lancar karena Anda menggunakan versi Python yang didukung penuh oleh Odoo 18 dan semua dependensinya.
