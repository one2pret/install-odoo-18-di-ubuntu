# Mengatasi Error WebSocket di Odoo: "Couldn't bind the websocket"

Saat mengembangkan atau menjalankan Odoo di lingkungan produksi, Anda mungkin menemui sebuah error yang berulang di log dan mengganggu fungsionalitas *real-time* seperti notifikasi atau live chat. Error tersebut adalah:

```
RuntimeError: Couldn't bind the websocket. Is the connection opened on the evented port (8072)?
```

Artikel ini akan mengupas tuntas penyebab error ini dan memberikan dua solusi praktis untuk mengatasinya, baik untuk lingkungan development maupun produksi.

---

## 💡 Memahami Akar Masalah

Secara sederhana, error ini muncul karena adanya **ketidakcocokan antara cara Odoo dijalankan dengan cara Odoo diakses**.

Ketika Odoo dikonfigurasi untuk berjalan dengan **mode *multi-processing*** (yaitu, nilai `workers` di file konfigurasi lebih dari 0), ia mendelegasikan tugas penanganan koneksi *real-time* (WebSocket) ke sebuah "perantara". Odoo mengharapkan ada **Reverse Proxy** (seperti Nginx atau Apache) di depannya yang akan menerima permintaan WebSocket dan meneruskannya ke port *longpolling* yang sesuai.

Error terjadi ketika Anda mengakses Odoo secara langsung ke port aplikasinya (misal, `8018`) padahal ia sedang berjalan dalam mode *workers* dan mengharapkan Nginx sebagai perantaranya.

---

## 🔧 Solusi dan Panduan Praktis

Terdapat dua cara untuk menyelesaikan masalah ini, tergantung pada tujuan penggunaan server Odoo Anda.

### Solusi 1 (Untuk Produksi): Menggunakan Nginx sebagai Reverse Proxy

Ini adalah metode yang **sangat direkomendasikan** untuk lingkungan produksi karena lebih aman, scalable, dan berperforma tinggi.

#### Langkah 1: Install Nginx
Pastikan Nginx sudah terpasang di server Anda.
```bash
sudo apt update
sudo apt install nginx -y
```

#### Langkah 2: Buat File Konfigurasi Nginx untuk Odoo
Buat sebuah file konfigurasi baru untuk situs Odoo Anda.
```bash
sudo nano /etc/nginx/sites-available/odoo.conf
```

#### Langkah 3: Isi Konfigurasi Nginx
Salin dan tempel konfigurasi berikut. Konfigurasi ini akan mengarahkan lalu lintas web biasa ke port aplikasi Odoo dan lalu lintas WebSocket ke port *longpolling*-nya.

```nginx
# Mendefinisikan server Odoo utama dan websocket
upstream odoo {
    server 127.0.0.1:8018;
}
upstream odoo_websocket {
    server 127.0.0.1:8019;
}

server {
    listen 80;
    # Ganti dengan domain atau alamat IP server Anda
    server_name odoo.domainanda.com;

    # Pengaturan untuk meneruskan request ke Odoo
    location / {
        proxy_pass http://odoo;
        # Header penting untuk Odoo
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Pengaturan khusus untuk menangani koneksi WebSocket
    location /websocket {
        proxy_pass http://odoo_websocket;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    # Pengaturan agar Nginx dapat menangani file upload yang besar
    client_max_body_size 200m;
}
```
> **Catatan Penting:** Pastikan port `8018` dan `8019` pada konfigurasi di atas sesuai dengan `xmlrpc_port` dan `longpolling_port` di file `odoo.conf` Anda.

#### Langkah 4: Aktifkan Konfigurasi dan Restart Nginx
```bash
# Aktifkan situs dengan membuat symbolic link
sudo ln -s /etc/nginx/sites-available/odoo.conf /etc/nginx/sites-enabled/

# Uji konfigurasi untuk memastikan tidak ada error
sudo nginx -t

# Terapkan perubahan dengan me-restart Nginx
sudo systemctl restart nginx
```
Setelah ini, akses Odoo Anda melalui alamat IP atau domain (di port 80), bukan lagi melalui port `8018`.

---

### Solusi 2 (Untuk Development): Menonaktifkan Mode Workers

Jika Anda hanya menjalankan Odoo untuk tujuan development atau pengujian dan tidak ingin repot dengan Nginx, solusi termudahnya adalah memaksa Odoo berjalan dalam mode *threaded*.

#### Langkah 1: Edit File Konfigurasi Odoo
Buka file konfigurasi Odoo Anda.
```bash
sudo nano /etc/odoo18.conf
```

#### Langkah 2: Atur Workers Menjadi Nol
Cari parameter `workers` dan pastikan nilainya adalah `0`. Jika baris tersebut tidak ada, itu sudah default ke mode *threaded*.
```ini
workers = 0
```

#### Langkah 3: Restart Service Odoo
Simpan perubahan dan restart service Odoo agar konfigurasi baru terbaca.
```bash
sudo systemctl restart odoo18.service
```
Dalam mode ini, Odoo akan menangani semua jenis koneksi sendiri, dan error WebSocket akan hilang.

---

## Kesimpulan 🚀

Error `Couldn't bind the websocket` bukanlah sebuah bug, melainkan sebuah pesan bahwa konfigurasi deployment Anda belum lengkap. Dengan memilih salah satu dari dua solusi di atas, Anda dapat memastikan semua fitur *real-time* Odoo berjalan dengan lancar sesuai dengan kebutuhan Anda.
