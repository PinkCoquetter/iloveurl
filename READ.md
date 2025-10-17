(https://raw.githubusercontent.com/shlinkio/shlink.io/main/public/images/shlink-hero.png)

# 📦 Shlink - URL Shortener

[🏠 Sekilas Tentang](#sekilas-tentang) • [⚙️ Instalasi](#instalasi) • [🧩 Konfigurasi](#konfigurasi) • [🚀 Otomatisasi](#otomatisasi) • [🧠 Cara Pemakaian](#cara-pemakaian) • [📚 Referensi](#referensi)

---

## 🏠 Sekilas Tentang
Shlink adalah aplikasi open-source URL shortener yang bisa dijalankan sendiri (self-hosted).  
Shlink dibuat menggunakan PHP dan menyediakan REST API. Shlink juga memungkinkan pengguna untuk:
- Memendekkan URL panjang menjadi pendek
- Melihat statistik klik dan lokasi pengguna
- Mengelola link lewat antarmuka web maupun API

Shlink bisa diintegrasikan dengan Shlink Web Client untuk tampilan visual dan manajemen yang lebih mudah.

---

## ⚙️ Instalasi

### Kebutuhan Sistem
- Linux / WSL / Windows (via Docker Desktop)
- Docker & Docker Compose
- Git
- RAM minimal 512 MB

### Langkah Instalasi Backend
1. Clone repositori Shlink:
   ```bash
   git clone https://github.com/shlinkio/shlink.git
   cd shlink

2. Install Docker Cli:
   sudo apt-get update && sudo apt-get upgrade
   sudo apt update && sudo apt install docker.io docker-compose -y

3. Migrasi database mysql:
   docker exec -it shlink_php sh
   ls -la /home/shlink/www/bin (cek file Namanya cli)
   php /home/shlink/www/bin/cli db:create (kalau nggak ada file cli)
   php /home/shlink/www/bin/cli db:migrate

4. Jalankan Docker Compose:
   docker compose up -d --build

5. Cek Backend:
   http://localhost:8000/rest/health
   (bila muncul status pass/ok maka backend sudah jalan)

### Frontend/web-client-server
1. Clone repositori:
   https://github.com/shlinkio/shlink-web-client.git
   cd shlink-web-client

2. Buat file .env:
   nano .env
   REACT_APP_SHLINK_SERVER_URL=http://<IP_NUM>:8080
   REACT_APP_SHLINK_SERVER_NAME=MyShlink (paste ini didalam .env nya)

3. Jalankan Frontend:
   docker compose up -d --build

 
