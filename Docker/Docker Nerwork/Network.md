# Docker Network Configuration

Konfigurasi Docker Network ini dibuat sesuai dengan konfigurasi yang terdapat pada Portainer, yaitu dua buah network:

- **backend** (`172.18.0.0/16`)
- **frontend** (`172.19.0.0/16`)

## Topologi Network

| Network | Driver | Subnet | Gateway |
|---------|--------|---------|----------|
| backend | bridge | 172.18.0.0/16 | 172.18.0.1 |
| frontend | bridge | 172.19.0.0/16 | 172.19.0.1 |

---

# Membuat Docker Network

Buat file bernama `create-networks.sh` kemudian isi dengan script berikut.

```bash
#!/bin/bash

# =========================================
# Membuat Docker Network: backend & frontend
# =========================================

echo "Membuat network backend..."
docker network create \
  --driver bridge \
  --subnet=172.18.0.0/16 \
  --gateway=172.18.0.1 \
  backend

echo "Membuat network frontend..."
docker network create \
  --driver bridge \
  --subnet=172.19.0.0/16 \
  --gateway=172.19.0.1 \
  frontend

echo
echo "Daftar Docker Network:"
docker network ls

echo
echo "Detail backend:"
docker network inspect backend

echo
echo "Detail frontend:"
docker network inspect frontend
```

---

# Cara Menjalankan

Berikan hak akses eksekusi pada script.

```bash
chmod +x create-networks.sh
```

Kemudian jalankan.

```bash
./create-networks.sh
```

---

# Versi One-Line

Apabila tidak ingin menggunakan script, jalankan perintah berikut.

```bash
docker network create --driver bridge --subnet=172.18.0.0/16 --gateway=172.18.0.1 backend

docker network create --driver bridge --subnet=172.19.0.0/16 --gateway=172.19.0.1 frontend
```

---

# Docker Compose

Jika ingin Docker Compose membuat network secara otomatis, tambahkan konfigurasi berikut pada `docker-compose.yml`.

```yaml
networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16
          gateway: 172.18.0.1

  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16
          gateway: 172.19.0.1
```

## Contoh Penggunaan

```yaml
services:
  app:
    image: nginx:latest
    container_name: app
    restart: unless-stopped

    networks:
      - frontend
      - backend

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16
          gateway: 172.18.0.1

  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16
          gateway: 172.19.0.1
```

---

# Verifikasi

Pastikan network berhasil dibuat.

```bash
docker network ls
```

Melihat detail network **backend**.

```bash
docker network inspect backend
```

Melihat detail network **frontend**.

```bash
docker network inspect frontend
```

---

# Hasil

Setelah berhasil dijalankan, akan terbentuk dua Docker Network seperti berikut.

| Network | Driver | Subnet | Gateway |
|---------|--------|---------|----------|
| backend | bridge | 172.18.0.0/16 | 172.18.0.1 |
| frontend | bridge | 172.19.0.0/16 | 172.19.0.1 |

---

## Catatan

- Driver yang digunakan adalah **bridge**.
- Konfigurasi subnet dan gateway mengikuti konfigurasi pada Portainer.
- Network ini dapat digunakan untuk komunikasi antar container pada arsitektur **frontend** dan **backend**.
- Jalankan script sebagai pengguna yang memiliki akses ke Docker (misalnya `root` atau pengguna yang tergabung dalam grup `docker`).
