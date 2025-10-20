# Project Hosting Shlink [Kelompok 6 / Paralel 1]

<p align="center">
  <img src="https://raw.githubusercontent.com/shlinkio/shlink.io/main/public/images/shlink-hero.png" alt="Shlink Logo" width="600">
</p>

<h1 align="center">SHLINK - Self-Hosted URL Shortener & Link Manager</h1>
<p align="center" style="font-size: 18px;"><b><i>Powerful, open-source, and privacy-first link shortener</i></b></p>

<div align="center">

| [Deskripsi](#deskripsi-aplikasi) | [Anggota Kelompok](#anggota-kelompok) | [Instalasi](#instalasi) | [Cara Pemakaian](#cara-pemakaian) | [Perbandingan](#perbandingan-shlink-vs-linktree) | [Referensi](#referensi) |
|----------------------------------|---------------------------------------|-------------------------|-----------------------------------|--------------------------------------------------|-------------------------|

</div>

---

## Deskripsi Aplikasi
<p align="justify">
**Shlink** adalah layanan pemendekan URL yang dihosting sendiri dan bersifat Open Source yang memungkinkan pengguna membuat dan mengelola URL pendek di bawah domain mereka sendiri. Aplikasi ini menyediakan antarmuka yang powerful namun sederhana untuk menghasilkan URL pendek, melacak klik, dan menganalisis data pengunjung. Shlink menawarkan berbagai fitur termasuk kode pendek kustom, akses API untuk integrasi yang mulus dengan aplikasi lain, dan antarmuka command-line untuk manajemen tingkat lanjut. Progressive web app (PWA) yang dimilikinya memberikan pengalaman pengguna yang intuitif. Dibangun dengan PHP dan memanfaatkan framework modern seperti Mezzio, Doctrine, dan Symfony, memastikan stabilitas dan performa. Shlink dirancang untuk pengguna yang menghargai kontrol atas data mereka dan lebih memilih solusi self-hosted dengan fitur yang ekstensif.
</p>
---

## Anggota Kelompok

| Nama | NIM |
|------|-----|
| Tsabitha Naylasafa Aurora | G6401231036 |
| Givari Mirzacky | G6401231098 |
| Naufal Rama Koswara | G6401231113 |
| Benadeo Eldian Manting | G6401231117 |
| Tristian Yosa | G6401231122 |

---

## Instalasi

### 1. Prasyarat
- Server / VPS (Ubuntu 22.04 atau lebih baru)
- Domain aktif (`dashboard.iloveurl.site` dan `short.iloveurl.site`)
- Sudah terinstal **Docker**, **Docker Compose**, dan **Nginx**

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install docker.io docker-compose nginx -y
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

### 2. Clone Repository

Clone repository Shlink Backend:

```bash
git clone https://github.com/shlinkio/shlink.git
cd shlink
```

---

### 3. Konfigurasi Docker Compose

Edit file `docker-compose.yml` dan sesuaikan konfigurasi berikut:

#### Port Configuration
Pastikan port yang digunakan sesuai dengan konfigurasi NGINX:
- Shlink server: `8800:8080` (port 8800 di host, 8080 di container)
- Dashboard: `3000:80` (port 3000 di host, 80 di container)

#### Database Configuration
Atur password database di environment variables:
```yaml
environment:
  MYSQL_DATABASE: shlink
  MYSQL_USER: shlink
  MYSQL_PASSWORD: your_secure_password
  MYSQL_ROOT_PASSWORD: your_root_password
```

---

### 4. Jalankan Docker Compose

```bash
sudo docker-compose up -d
```

---

### 5. Generate API Key

Setelah container berjalan, masuk ke container Shlink dan generate API key:

```bash
docker exec -it shlink_php sh
php /home/shlink/www/bin/cli api-key:generate
exit
```

Salin API key yang dihasilkan dan masukkan ke file `docker-compose.yml` pada bagian `SHLINK_ADMIN_API_KEY`, lalu restart container:

```bash
sudo docker-compose restart shlink
```

---

### 6. Setup Database

Masuk ke container Shlink dan jalankan migrasi database:

```bash
docker exec -it shlink_php sh
php /home/shlink/www/bin/cli db:create
php /home/shlink/www/bin/cli db:migrate
exit
```

---

### 7. Clone dan Setup Web Client

Pindah ke direktori terpisah untuk web client:

```bash
cd ..
git clone https://github.com/shlinkio/shlink-web-client.git
cd shlink-web-client
```

---

### 10. Akses Layanan

#### File: `/etc/nginx/sites-available/shlink.site`

```nginx
server {
    listen 80;
    server_name iloveurl.site short.iloveurl.site;
    location / {
        proxy_pass http://127.0.0.1:8800;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan konfigurasi:

```bash
sudo ln -s /etc/nginx/sites-available/shlink.site /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### File: `/etc/nginx/sites-available/shlink.dashboard`

```nginx
server {
    listen 80;
    server_name dashboard.iloveurl.site;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan konfigurasi:

```bash
sudo ln -s /etc/nginx/sites-available/shlink.dashboard /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### File: `/etc/nginx/sites-available/shlink.site`

```nginx
server {
    listen 80;
    server_name iloveurl.site short.iloveurl.site;
    location / {
        proxy_pass http://127.0.0.1:8800;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan konfigurasi:

```bash
sudo ln -s /etc/nginx/sites-available/shlink.site /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### File: `/etc/nginx/sites-available/shlink.dashboard`

```nginx
server {
    listen 80;
    server_name dashboard.iloveurl.site;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan konfigurasi:

```bash
sudo ln -s /etc/nginx/sites-available/shlink.dashboard /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

### 10. Akses Layanan

| Komponen             | URL                                                                | Deskripsi                                 |
| -------------------- | ------------------------------------------------------------------ | ----------------------------------------- |
| **Dashboard Shlink** | [https://dashboard.iloveurl.site](https://dashboard.iloveurl.site) | Antarmuka untuk membuat dan memantau link |
| **Backend API**      | [http://103.226.138.119/rest/v3](http://103.226.138.119/rest/v3)   | Endpoint API REST                         |
| **Short Domain**     | [https://short.iloveurl.site](https://short.iloveurl.site)         | Domain untuk tautan pendek                |

---

## Cara Pemakaian

### 1. Login ke Dashboard

Masuk ke:

```
https://dashboard.iloveurl.site
```

Masukkan **API Key:** (contoh saja)

```
11dca439-54f1-4df3-a078-88480a1adfa5
```

---

### 2. Membuat Link Pendek

Isi formulir dengan:

* **Long URL**: tautan asli
* **Custom Slug (opsional)**: nama pendek unik
* **Tag / Expiry (opsional)**: keterangan tambahan

Hasil contoh:

```
https://short.iloveurl.site/kelas-paralel1
```

---

### 3. Melihat Statistik Klik

Fitur analitik tersedia langsung di dashboard:

* Total klik
* Lokasi negara pengunjung
* Browser / sistem operasi
* Waktu dan sumber referer

---

## Integrasi API

Contoh membuat short URL melalui terminal:

```bash
curl -X POST "http://103.226.138.119/rest/v3/short-urls" \
  -H "X-Api-Key: 0p+mDvbpZGLPGVCXnV+EDduR9Blkv27Dhq9XSzSbdQY=" \
  -d "longUrl=https://example.com&customSlug=promo"
```

---

## Perbandingan Shlink vs Linktree

| Aspek | Shlink | Linktree |
| ----------------- | -------------------- | --------------- |
| **Hosting** | Self-hosted (dikelola sendiri di server pribadi atau VPS) | Cloud-based (dikelola oleh penyedia layanan) |
| **Privasi Data** | Kontrol penuh di tangan pengguna, data tersimpan di server sendiri | Data dikelola oleh pihak ketiga, bergantung pada kebijakan privasi mereka |
| **Analitik** | Lengkap dan real-time dengan detail klik, lokasi geografis, browser, sistem operasi, dan referer | Terbatas pada paket gratis, fitur analitik lengkap hanya tersedia di paket berbayar |
| **Domain Kustom** | Dapat menggunakan domain sendiri tanpa biaya tambahan | Tersedia hanya untuk pengguna paket berbayar |
| **Multi-link Profil** | Tidak tersedia (fokus pada URL shortening) | Tersedia, cocok untuk bio link di media sosial |
| **API Integration** | API REST lengkap untuk integrasi dengan sistem lain | Tidak tersedia atau sangat terbatas |
| **Biaya** | Gratis (open-source), hanya biaya server/hosting | Gratis dengan fitur terbatas, fitur premium berbayar |
| **Kemudahan Setup** | Membutuhkan pengetahuan teknis (Docker, server management) | Sangat mudah, tanpa setup teknis |
| **Kustomisasi** | Penuh, dapat dimodifikasi sesuai kebutuhan | Terbatas pada template yang tersedia |
| **QR Code** | Otomatis generate untuk setiap short URL | Tersedia di paket berbayar |

---

## Kesimpulan

### Kelebihan Shlink
- **Gratis dan Open Source**: Tidak ada biaya lisensi, dapat dimodifikasi sesuai kebutuhan
- **Kontrol Penuh**: Data dan infrastruktur sepenuhnya di bawah kendali pengguna
- **Privasi Terjamin**: Tidak ada pihak ketiga yang mengakses data klik dan analitik
- **Analitik Lengkap**: Tracking detail real-time tanpa batasan
- **Domain Kustom**: Menggunakan domain sendiri tanpa biaya tambahan
- **API Lengkap**: Integrasi mudah dengan aplikasi dan sistem lain
- **QR Code Generator**: Generate QR code otomatis untuk setiap URL
- **Scalable**: Dapat disesuaikan dengan kebutuhan traffic dan storage

### Kekurangan Shlink
- **Setup Teknis**: Membutuhkan pengetahuan tentang server, Docker, dan networking
- **Maintenance**: Perlu maintenance rutin (update, backup, monitoring)
- **Biaya Hosting**: Memerlukan server atau VPS untuk menjalankan aplikasi
- **Tidak Ada Multi-link Profil**: Tidak cocok untuk kebutuhan bio link seperti Instagram
- **Learning Curve**: Butuh waktu untuk mempelajari cara penggunaan dan konfigurasi

### Gunakan Shlink Jika:
- Membutuhkan kontrol penuh atas data dan privasi
- Mengintegrasikan URL shortener dengan sistem internal perusahaan
- Memerlukan analitik detail dan real-time tanpa batasan
- Ingin menggunakan domain sendiri tanpa biaya tambahan
- Memiliki kemampuan teknis untuk setup dan maintenance server
- Mengelola volume URL dalam jumlah besar
- Membutuhkan API untuk automasi dan integrasi
- Tim atau organisasi yang memerlukan solusi self-hosted

### Gunakan Linktree Jika:
- Membutuhkan bio link untuk profil media sosial (Instagram, TikTok, dll)
- Ingin setup cepat tanpa pengetahuan teknis
- Tidak ingin repot dengan server maintenance
- Fokus pada tampilan profil dengan multiple links
- Analitik dasar sudah cukup untuk kebutuhan
- Budget terbatas dan tidak keberatan dengan fitur terbatas
- Pengguna individu atau personal branding
- Tidak memerlukan integrasi API atau kustomisasi mendalam

---

## Referensi

1. [Shlink Official Docs](https://shlink.io/documentation)
2. [Docker Hub: shlinkio/shlink](https://hub.docker.com/r/shlinkio/shlink)
3. [Linktree Website](https://linktr.ee/)
4. [Chatgpt](https://chatgpt.com/)
5. [Claude AI](https://claude.ai/)
