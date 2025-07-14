# Odoo Restore Gagal & Server Restart? Ini Dia 'Penjaga Waktu' Tersembunyi di Odoo!

Anda mungkin pernah berada di situasi ini: proses krusial memindahkan database Odoo sedang berjalan, bar kemajuan bergerak, namun tiba-tiba... koneksi terputus! Anda memeriksa log server dan menemukan pesan misterius: `virtual real time limit reached`. Apa sebenarnya yang terjadi?

Jangan khawatir, server Anda tidak rusak. Anda baru saja bertemu dengan "penjaga waktu" tersembunyi di dalam Odoo. Mari kita kenali siapa dia dan bagaimana cara bernegosiasi dengannya.

---

### ⏱️ Mengenal `limit_time_real`, Penjaga Waktu Odoo

Secara default, Odoo sangat protektif terhadap sumber dayanya. Ia tidak ingin satu proses tunggal—seperti restore database yang besar atau pembuatan laporan yang sangat kompleks—menyandera salah satu "pekerja" (worker) terlalu lama dan membuat pengguna lain tidak bisa dilayani.

Untuk mencegah hal ini, Odoo memasang batas waktu default selama **120 detik (2 menit)** untuk setiap permintaan web. Jika sebuah proses melebihi batas ini, Odoo akan menganggapnya "macet" dan secara paksa menghentikannya lalu me-restart service untuk kembali ke kondisi normal.

Inilah mengapa di log Anda muncul pesan `limit (130/120s) reached`. Proses restore Anda butuh 130 detik, melebihi batas 120 detik, sehingga sang "penjaga waktu" pun beraksi.

---

### ⚙️ Solusi Cepat: Mengatur Ulang Jam Sang Penjaga

Jika Anda ingin memberikan kelonggaran waktu, Anda bisa mengatur ulang "jam" milik si penjaga. Caranya adalah dengan mengubah file konfigurasi Odoo.

#### Langkah 1: Buka File Konfigurasi Odoo
Buka file `odoo.conf` Anda di server. Untuk instalasi yang telah kita lakukan, lokasinya ada di:
```bash
sudo nano /etc/odoo18.conf
```

#### Langkah 2: Tambahkan Parameter Timeout
Tambahkan dua baris berikut di bawah bagian `[options]`. Kita akan memberinya batas waktu baru yang lebih panjang, misalnya 3600 detik (1 jam).

```ini
[options]
# ...konfigurasi Anda yang lain...

# Menaikkan batas waktu untuk request (dalam detik)
limit_time_real = 3600
limit_time_cpu = 3600
```
* **`limit_time_real`**: Ini adalah batas waktu utama berdasarkan jam dinding, yang paling relevan untuk kasus restore.
* **`limit_time_cpu`**: Batas berdasarkan waktu penggunaan prosesor.

#### Langkah 3: Simpan dan Restart Service Odoo
Simpan perubahan file (Ctrl+O, Enter, lalu Ctrl+X) dan terapkan dengan me-restart service Odoo.
```bash
sudo systemctl restart odoo18.service
```
Sekarang, Odoo Anda akan jauh lebih sabar menunggu proses restore dari web selesai.

---

### ⚠️ Peringatan & Metode Profesional (Rekomendasi Terbaik)

Meskipun solusi di atas berhasil, menaikkan batas waktu secara global bukanlah praktik terbaik untuk lingkungan produksi. Jika ada proses lain yang benar-benar macet karena bug, ia akan menggantung server selama 1 jam, bukan hanya 2 menit.

> **Jalan Profesional: Restore via Command Line**
>
> Untuk operasi database yang berat dan penting seperti restore, metode terbaik adalah selalu melalui **terminal (command line)**.
> ```bash
> psql -U nama_user_db -d nama_db_tujuan < /lokasi/file_backup.dump
> ```
> Metode ini tidak memiliki batas waktu, lebih stabil, dan tidak mengganggu alur kerja utama Odoo yang melayani pengguna.

### Kesimpulan 🚀

Kini Anda tahu bahwa di balik layar Odoo ada sebuah mekanisme pengaman yang cerdas. Anda memiliki dua senjata baru: kemampuan untuk mengatur "penjaga waktu" Odoo untuk fleksibilitas, dan pengetahuan tentang metode command line yang lebih tangguh untuk tugas-tugas berat. Pilih senjata yang tepat untuk situasi yang tepat!
