# Open Journal Systems (OJS 3.4) Docker Setup

Setup Docker Compose siap pakai untuk **Open Journal Systems (OJS 3.4)** dengan database **MariaDB 10.11 LTS** dan optimasi PHP.

Ketika folder ini di-*pull* ke server dan dijalankan `docker compose up -d`, OJS langsung dapat diakses pada port **3200**.

---

## 🚀 Fitur Setup

- **OJS 3.4 (Stable)**: Menggunakan image resmi `pkpofficial/ojs:stable-3_4_0`.
- **Port 3200**: Langsung siap diakses di `http://localhost:3200` atau `http://<ip-server>:3200`.
- **MariaDB 10.11 LTS**: Database stabil dengan konfigurasi character set `utf8mb4` (sesuai standar OJS 3.4+).
- **Healthcheck Synchronization**: OJS otomatis menunggu hingga MariaDB benar-benar siap menerima koneksi sebelum booting.
- **Persistent Data**: Menggunakan Docker Named Volumes (`ojs_files_data`, `ojs_public_data`, `ojs_db_data`) sehingga data artikel/naskah dan database tetap aman saat container di-*restart* atau di-*update*, serta bebas dari kendala perizinan hak akses (*permission denied*) Linux.
- **Optimasi PHP (`php-custom.ini`)**: Batas upload diperbesar hingga `128M`, memory limit `512M`, dan timezone `Asia/Jakarta`.
- **Database Management GUI (Opsional)**: Tersedia phpMyAdmin di port `3201` (dapat dijalankan kapan saja jika dibutuhkan).

---

## 📁 Struktur Folder

```text
journal_system/
├── docker-compose.yml   # Konfigurasi container OJS, MariaDB, & phpMyAdmin
├── .env                 # File environment aktif (port & kredensial)
├── .env.example         # Template environment
├── php-custom.ini       # Konfigurasi custom PHP (upload limit, timezone)
├── .gitignore           # File git ignore
└── README.md            # Dokumentasi panduan
```

---

## ⚡ Cara Menjalankan di Server

### 1. Masuk ke Folder Project
Jika folder di-pull dari Git di server:
```bash
git clone <URL_REPOSITORY_ANDA>
cd journal_system
```

### 2. Buat & Sesuaikan Pengaturan di `.env`
Salin template `.env.example` menjadi `.env` (jika belum ada) lalu sesuaikan:
```bash
cp .env.example .env
nano .env
```
Default bawaan:
- **Port OJS**: `3200`
- **Port phpMyAdmin**: `3201`
- **Database Name**: `ojs`
- **Database User**: `ojs`
- **Database Password**: `ojs_secure_pass123`
- **Database Root Password**: `root_secure_pass123`

### 3. Jalankan Docker Compose
Jalankan perintah berikut:
```bash
docker compose up -d
```
Tunggu hingga container selesai di-download dan berstatus *Running*.

Periksa status container:
```bash
docker compose ps
```

---

## 🌐 Menyelesaikan Instalasi OJS via Browser

1. Buka browser dan kunjungi:
   - Local: **`http://localhost:3200`**
   - Server: **`http://<IP_SERVER_ANDA>:3200`**
2. Halaman **Open Journal Systems Installation** akan tampil.
3. Isi form instalasi:
   - **Administrator Account**:
     - *Username*, *Password*, dan *Email* untuk login admin Anda.
   - **File Settings**:
     - *Directory for uploads*: Biarkan default `/var/www/files`.
   - **Database Settings**:
     - *Database driver*: `MySQLi`
     - *Host*: `db` (sesuai nama service database di compose)
     - *Username*: `ojs` (atau sesuai `.env`)
     - *Password*: `ojs_secure_pass123` (atau sesuai `.env`)
     - *Database name*: `ojs`
     - **PENTING**: **Hapus centang (uncheck)** pada *"Create new database"* karena database sudah dibuat otomatis oleh MariaDB.
4. Klik tombol **Install Open Journal Systems** di bagian paling bawah.
5. Tunggu proses instalasi database selesai. Anda akan diarahkan langsung ke halaman dashboard OJS!
6. **PENTING: Amankan File `config.inc.php` ke Host**:
   Saat OJS diinstal via browser, file konfigurasi tersimpan di dalam container `ojs_app`. Agar konfigurasi ini tidak terhapus jika container di-`docker compose down`, salin file tersebut ke host:
   ```bash
   docker compose cp ojs_app:/var/www/html/config.inc.php ./config.inc.php
   ```
   Lalu buka `docker-compose.yml` dan hapus tanda `#` pada baris volume konfigurasi:
   ```yaml
       volumes:
         ...
         - ./config.inc.php:/var/www/html/config.inc.php
   ```
   Dengan langkah ini, OJS Anda 100% persisten dan aman dari `docker compose down`.

---

## 🛠️ Perintah Berguna (Maintenance)

### Melihat Log Aplikasi
```bash
# Log OJS
docker compose logs -f ojs

# Log Database
docker compose logs -f db
```

### Membuka phpMyAdmin (Opsional)
Jika Anda membutuhkan antarmuka web untuk melihat/mengelola database MariaDB:
```bash
docker compose --profile tools up -d
```
Akses di browser: **`http://<IP_SERVER_ANDA>:3201`**
- Server: `db`
- Username: `ojs`
- Password: `ojs_secure_pass123`

Untuk mematikan phpMyAdmin kembali:
```bash
docker compose --profile tools stop phpmyadmin
```

### Backup Database
Untuk membuat backup database OJS ke file `.sql`:
```bash
docker compose exec db mariadb-dump -u ojs -pojs_secure_pass123 ojs > backup_ojs.sql
```

### Restore Database
Untuk me-restore file backup `.sql`:
```bash
docker compose exec -T db mariadb -u ojs -pojs_secure_pass123 ojs < backup_ojs.sql
```

### Menghentikan / Restart Container
```bash
# Menghentikan sementara
docker compose stop

# Menjalankan kembali
docker compose start

# Restart container
docker compose restart

# Menghentikan dan menghapus container (data volume tetap aman)
docker compose down
```
