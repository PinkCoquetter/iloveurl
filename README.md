# Project Hosting Shlink [Kelompok 4 / Paralel 1]

<p align="center">
  <img src="https://raw.githubusercontent.com/shlinkio/shlink.io/main/public/images/shlink-hero.png" alt="Shlink Logo" width="600">
</p>

<h1 align="center">SHLINK - Self-Hosted URL Shortener & Link Manager</h1>
<p align="center" style="font-size: 18px;"><b><i>Powerful, open-source, and privacy-first link shortener</i></b></p>

---

## Deskripsi Aplikasi
<p align="justify">
Shlink adalah layanan pemendekan URL yang dihosting sendiri dan bersifat Open Source yang memungkinkan pengguna membuat dan mengelola URL pendek di bawah domain mereka sendiri. Aplikasi ini menyediakan antarmuka yang powerful namun sederhana untuk menghasilkan URL pendek, melacak klik, dan menganalisis data pengunjung. Shlink menawarkan berbagai fitur termasuk kode pendek kustom, akses API untuk integrasi yang mulus dengan aplikasi lain, dan antarmuka command-line untuk manajemen tingkat lanjut. Progressive web app (PWA) yang dimilikinya memberikan pengalaman pengguna yang intuitif. Dibangun dengan PHP dan memanfaatkan framework modern seperti Mezzio, Doctrine, dan Symfony, memastikan stabilitas dan performa. Shlink dirancang untuk pengguna yang menghargai kontrol atas data mereka dan lebih memilih solusi self-hosted dengan fitur yang ekstensif.
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

```bash
git clone https://github.com/kelompok4-iloveurl/shlink.git
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

#### API Key Configuration
Generate dan set API key untuk Shlink:
```yaml
environment:
  SHLINK_ADMIN_API_KEY: "your_generated_api_key"
```

Contoh konfigurasi lengkap `docker-compose.yml`:
```yaml
version: "3.8"

services:
  db:
    image: mariadb:10.11
    container_name: shlink_db
    restart: always
    environment:
      MYSQL_DATABASE: shlink
      MYSQL_USER: shlink
      MYSQL_PASSWORD: shlink123
      MYSQL_ROOT_PASSWORD: rootpass
    volumes:
      - ./data/db:/var/lib/mysql

  shlink:
    image: shlinkio/shlink:stable
    container_name: shlink_server
    restart: always
    depends_on:
      - db
    environment:
      DEFAULT_DOMAIN: short.iloveurl.site
      IS_HTTPS_ENABLED: "false"
      DB_DRIVER: mysql
      DB_HOST: db
      DB_NAME: shlink
      DB_USER: shlink
      DB_PASSWORD: shlink123
      GEOLITE_LICENSE_KEY: ""
      SHLINK_ADMIN_API_KEY: "0p+mDvbpZGLPGVCXnV+EDduR9Blkv27Dhq9XSzSbdQY="
    ports:
      - "8800:8080"

  web_client:
    image: shlinkio/shlink-web-client:stable
    container_name: shlink_dashboard
    restart: always
    environment:
      SHLINK_API_URL: "http://103.226.138.119:8800"
    ports:
      - "3000:80"
```

---

### 4. Konfigurasi NGINX

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

### 5. Konfigurasi Vite

Tambahkan konfigurasi berikut di file `vite.config.ts` untuk mengizinkan akses dari domain dashboard:

```typescript
server: {
  port: 3000,
  allowedHosts: ['dashboard.iloveurl.site', 'localhost', '127.0.0.1'],
  watch: {
    // Do not watch test files or generated files, avoiding the dev server to constantly reload when not needed
    ignored: ['**/.idea/**', '**/.git/**', '**/build/**', '**/coverage/**', '**/test/**'],
  },
},
```

---

### 6. Jalankan Docker Compose

```bash
sudo docker-compose up -d
```

---

### 7. Akses Layanan

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

Masukkan **API Key:**

```
0p+mDvbpZGLPGVCXnV+EDduR9Blkv27Dhq9XSzSbdQY=
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

| Aspek             | Shlink               | Linktree        |
| ----------------- | -------------------- | --------------- |
| Hosting           | Self-hosted          | Cloud           |
| Privasi Data      | Penuh di tangan user | Di pihak ketiga |
| Analitik          | Lengkap & real-time  | Terbatas        |
| Domain Kustom     | Bisa                 | Berbayar        |
| Multi-link Profil | Tidak ada            | Ada             |
| API Integration   | Lengkap              | Tidak tersedia  |

---

## Kesimpulan

* **Kelebihan:** gratis, open-source, analitik lengkap, bisa diintegrasikan ke sistem lain.
* **Kekurangan:** setup lebih teknis, tidak ada multi-link profil seperti Linktree.
* **Cocok untuk:** tim internal, organisasi, atau proyek yang butuh sistem shortlink privat dan aman.

---

## Referensi

1. [Shlink Official Docs](https://shlink.io/documentation)
2. [Docker Hub: shlinkio/shlink](https://hub.docker.com/r/shlinkio/shlink)
3. [Linktree Website](https://linktr.ee/)
4. [Certbot for SSL](https://certbot.eff.org/)
