# Project Hosting Shlink [Kelompok 4 / Paralel 1]

<p align="center">
  <img src="https://shlink.io/images/shlink-logo.svg" alt="Shlink Logo" width="600">
</p>

<h1 align="center">SHLINK - Self-Hosted URL Shortener & Link Manager</h1>
<p align="center" style="font-size: 18px;"><b><i>Powerful, open-source, and privacy-first link shortener</i></b></p>

---

## Deskripsi Aplikasi

**Shlink** adalah aplikasi *self-hosted URL shortener* berbasis PHP dan Node.js yang memungkinkan pembuatan tautan pendek, analitik klik, serta integrasi API dengan domain kustom.

Dalam proyek ini, Shlink dijalankan menggunakan **Docker Compose** dengan konfigurasi sebagai berikut:

| Komponen | Fungsi | URL |
|-----------|---------|-----|
| Backend (API Server) | Endpoint utama API | `http://103.226.138.119` |
| Web Client (Dashboard) | UI manajemen link | `https://dashboard.iloveurl.site` |
| Short Domain | Domain URL pendek | `https://short.iloveurl.site` |

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
- Sudah terinstal **Docker** dan **Docker Compose**

```bash
sudo apt update && sudo apt install docker.io docker-compose -y
```

---

### 2. Clone Repository

```bash
git clone https://github.com/kelompok4-iloveurl/shlink.git
cd shlink
```

---

### 3. Struktur Direktori

```
/shlink
 ├── docker-compose.yml
 ├── nginx/
 │    ├── shlink.site
 │    └── shlink.dashboard
 ├── data/
 └── README.md
```

---

### 4. Konfigurasi `docker-compose.yml`

Isi `docker-compose.yml` seperti ini:

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
      IS_HTTPS_ENABLED: "true"
      DB_DRIVER: mysql
      DB_HOST: db
      DB_NAME: shlink
      DB_USER: shlink
      DB_PASSWORD: shlink123
      GEOLITE_LICENSE_KEY: ""
      SHLINK_ADMIN_API_KEY: "0p+mDvbpZGLPGVCXnV+EDduR9Blkv27Dhq9XSzSbdQY="
    ports:
      - "8080:8080"

  web_client:
    image: shlinkio/shlink-web-client:stable
    container_name: shlink_dashboard
    restart: always
    environment:
      SHLINK_API_URL: "http://103.226.138.119"
    ports:
      - "3000:80"

  nginx:
    image: nginx:alpine
    container_name: shlink_nginx
    restart: always
    volumes:
      - ./nginx/shlink.site:/etc/nginx/conf.d/shlink.site
      - ./nginx/shlink.dashboard:/etc/nginx/conf.d/shlink.dashboard
    ports:
      - "80:80"
      - "443:443"
```

---

### 5. Konfigurasi NGINX

#### File: `nginx/shlink.site`

```nginx
server {
    listen 80;
    server_name short.iloveurl.site;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name short.iloveurl.site;

    ssl_certificate /etc/letsencrypt/live/short.iloveurl.site/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/short.iloveurl.site/privkey.pem;

    location / {
        proxy_pass http://shlink:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### File: `nginx/shlink.dashboard`

```nginx
server {
    listen 80;
    server_name dashboard.iloveurl.site;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name dashboard.iloveurl.site;

    ssl_certificate /etc/letsencrypt/live/dashboard.iloveurl.site/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dashboard.iloveurl.site/privkey.pem;

    location / {
        proxy_pass http://web_client:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
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
