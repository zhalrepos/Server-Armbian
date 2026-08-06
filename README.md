# Topologi Server Armbian (STB HG680P)

Dokumentasi topologi dan arsitektur server **Armbian berbasis STB HG680P** yang digunakan untuk menjalankan beberapa layanan Docker seperti **Frontend**, **Backend**, **Cloudflared**, **Portainer**, **Mikhmon**, dan **DSB**.

---

## Gambaran Umum

Server menggunakan **Armbian** sebagai sistem operasi utama dengan beberapa optimasi:

* **Free Cache** → membersihkan cache RAM otomatis.
* **Docker** → menjalankan layanan dalam container.
* **Timezone** → sinkronisasi zona waktu sistem.
* **ZRAM** → kompresi RAM untuk perangkat dengan memori terbatas.
* **Auto Restart** → memastikan layanan otomatis aktif kembali setelah reboot atau crash.

---

## Diagram Topologi

```text
                            SERVER
                            SERVER
                               │
                       ┌───────▼────────┐
                       │    ARMBIAN         │
                       └───────┬────────┘
                               │
                    ┌──────────┴───────┬──────────────────┬────────────────┬─────────────────┐
                    │                  │                  │                │                 │
            ┌───────▼───────┐     ┌────▼─────┐    ┌───────▼──────┐    ┌────▼───┐     ┌───────▼───────┐
            │   FREE CACHE     │     │  DOCKER    │    │   TIMEZONE      │    │  ZRAM   │     │  AUTO RESTART    │
            └───────────────┘     └────┬─────┘    └──────────────┘    └────────┘     └───────────────┘
                                       │
                                       │
                        ┌──────────────┴─────────────────────────────────────────────┐
                        │                    DOCKER ENVIRONMENT                      │
                        └──────────────┬─────────────────────────────────────────────┘
                                       │
                 ┌─────────────────────┴─────────────────────┐
                 │                                           │
         ┌───────▼────────┐                         ┌────────▼────────┐
         │ Docker Network │                         │ Docker Container│
         └───────┬────────┘                         └────────┬────────┘
                 │                                           │
       ┌─────────┼─────────┬───────────────────┬─────────────┼─────────────┬───────────────────┐
       │         │         │                   │             │             │                   │
┌──────▼──────┐ ┌▼───────┐ ┌▼──────────────┐ ┌▼──────────┐ ┌▼──────────┐ ┌▼──────────────┐ ┌▼──────────────┐
│  Frontend   │ │ Backend│ │ Cloudflared  │ │ Portainer │ │  Mikhmon  │ │      DSB      │ │    Future     │
└─────────────┘ └────────┘ └──────────────┘ └───────────┘ └───────────┘ └───────────────┘ └───────────────┘
```

---

## Struktur Layanan

### Armbian Host

| Komponen             | Fungsi                                               |
| -------------------- | ---------------------------------------------------- |
| **Free Cache**       | Membersihkan page cache untuk menjaga RAM tetap lega |
| **Docker Engine**    | Menjalankan seluruh container aplikasi               |
| **Timezone Service** | Sinkronisasi zona waktu sistem                       |
| **ZRAM**             | Menambah efisiensi penggunaan RAM                    |
| **Auto Restart**     | Menjalankan ulang layanan saat boot atau crash       |

---

### Docker Container

| Container       | Fungsi                                      |
| --------------- | ------------------------------------------- |
| **Frontend**    | Antarmuka web utama pengguna                |
| **Backend**     | API dan logika aplikasi                     |
| **Cloudflared** | Tunnel Cloudflare tanpa membuka port publik |
| **Portainer**   | Manajemen Docker berbasis web               |
| **Mikhmon**     | Manajemen MikroTik Hotspot                  |
| **DSB**         | Dashboard / monitoring service              |

---

## Spesifikasi Target

### Perangkat

* **STB HG680P**
* CPU: **Amlogic S905X**
* RAM: **2 GB**
* Storage: **eMMC / SD Card**
* OS: **Armbian Bookworm / Jammy**

---

## Rekomendasi Konfigurasi Docker

### docker-compose.yml

```yaml
version: '3.9'

services:
  frontend:
    image: nginx:alpine
    container_name: frontend
    restart: unless-stopped
    networks:
      - app-network

  backend:
    image: php:8.4-fpm
    container_name: backend
    restart: unless-stopped
    networks:
      - app-network

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel run

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - '9000:9000'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  app-network:
    driver: bridge
```

---

## Optimasi untuk STB HG680P

### Aktifkan ZRAM

```bash
sudo apt update
sudo apt install zram-tools -y
```

Contoh konfigurasi:

```bash
ALGO=lz4
PERCENT=50
PRIORITY=100
```

---

### Auto Free Cache

Buat file:

```bash
sudo nano /usr/local/bin/freecache.sh
```

Isi:

```bash
#!/bin/bash
sync
echo 3 > /proc/sys/vm/drop_caches
```

Tambahkan cron:

```bash
chmod +x /usr/local/bin/freecache.sh
(crontab -l ; echo '*/30 * * * * /usr/local/bin/freecache.sh') | crontab -
```

---

### Auto Restart Docker

Aktifkan service Docker saat boot:

```bash
sudo systemctl enable docker
sudo systemctl restart docker
```

---

## Alur Koneksi

### Akses Lokal

```text
Client Browser
       │
       ▼
Frontend (Nginx)
       │
       ▼
Backend API (PHP 8.4)
       │
       ▼
Database / MikroTik / Service lain
```

### Akses Publik

```text
Internet
    │
    ▼
Cloudflare Tunnel
    │
    ▼
Cloudflared Container
    │
    ▼
Frontend / Backend
```

---

## Monitoring

### Portainer

Akses:

```text
http://IP-SERVER:9000
```

Fitur:

* Monitoring container
* Restart / stop / update container
* Melihat log aplikasi
* Monitoring resource CPU & RAM

---

## Keamanan yang Disarankan

* Gunakan **Cloudflare Tunnel** dibanding membuka port langsung.
* Nonaktifkan login root via SSH.
* Gunakan **SSH key**.
* Tambahkan **UFW Firewall**:

```bash
sudo apt install ufw -y
sudo ufw allow 22/tcp
sudo ufw allow 9000/tcp
sudo ufw enable
```

---

## Struktur Direktori yang Direkomendasikan

```text
/opt/server/
├── docker-compose.yml
├── frontend/
├── backend/
├── mikhmon/
├── dsb/
├── cloudflared/
├── portainer/
└── scripts/
    ├── freecache.sh
    └── backup.sh
```

---

## Status Topologi

| Layer          | Status |
| -------------- | ------ |
| Armbian Host   | ✅      |
| Docker Engine  | ✅      |
| Docker Network | ✅      |
| Frontend       | ✅      |
| Backend        | ✅      |
| Cloudflared    | ✅      |
| Portainer      | ✅      |
| Mikhmon        | ✅      |
| DSB            | ✅      |

---

## Catatan

Topologi ini dirancang khusus untuk **STB HG680P dengan RAM 2 GB**, sehingga fokus utama adalah:

* **Ringan**
* **Stabil**
* **Mudah dimonitor**
* **Mudah di-backup**
* **Siap dikembangkan untuk sistem hotspot MikroTik dan web app berbasis PHP 8.4**

---

## Lisensi

MIT License

---

**Author:** Rizal Ali Akbar
**Platform:** Armbian + Docker + STB HG680P
