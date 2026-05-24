arkdown

````
---
title: Rangkuman Tugas Docker, Nginx, & Apache2
date: 2026-05-20
tags:
  - tugas
  - docker
  - wsl
---

# Panduan Lengkap Tugas Docker (WSL Ubuntu)

> [!info] Daftar Screenshot (SS) yang Dikumpulkan
> 1. Tampilan eksekusi perintah `docker run`
> 2. Isi dari file `docker-compose.yml`
> 3. Tampilan `docker ps`
> 4. Halaman Nginx diakses via IP
> 5. Halaman Apache2 diakses via IP

---

## Tahap 1: Instalasi Docker Engine

Jalankan perintah berikut secara berurutan di terminal Ubuntu:

**1. Update sistem dan hapus versi lama (jika ada)**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt remove docker docker-engine docker.io containerd runc
````

**2. Instal dependensi**

Bash

```
sudo apt install ca-certificates curl gnupg lsb-release -y
```

**3. Tambahkan GPG Key Resmi**

Bash

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**4. Tambahkan Repositori Docker**

Bash

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**5. Instal Docker & Compose Plugin**

Bash

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

## Tahap 2: Konfigurasi Akses Docker (Khusus WSL)

Langkah ini agar mesin Docker menyala dan tidak perlu terus mengetik `sudo`.

Bash

```
sudo usermod -aG docker $USER
newgrp docker
sudo service docker start
```

## Tahap 3: Pembuatan Container & Pengambilan SS

### 📸 SS 1: Perintah `docker run`

Jalankan perintah ini untuk membuat satu container manual. **Langsung screenshot saat muncul ID panjang (kombinasi angka & huruf) di bawahnya.**

Bash

```
docker run -d -p 8083:80 --name tugas-run nginx
```

### 📸 SS 2: Isi `docker-compose.yml`

Buat folder dan file konfigurasi untuk menjalankan Nginx dan Apache2 bersamaan:

Bash

```
mkdir tugas-docker-web && cd tugas-docker-web
nano docker-compose.yml
```

Masukkan konfigurasi berikut ke dalam Nano:

YAML

```
services:
  web-nginx:
    image: nginx:latest
    container_name: server-nginx
    ports:
      - "8080:80"

  web-apache:
    image: httpd:latest
    container_name: server-apache
    ports:
      - "8081:80"
```

_(Simpan dengan menekan `Ctrl+X`, lalu `Y`, lalu `Enter`)_.

Setelah tersimpan, ketik perintah di bawah ini dan **Screenshot hasilnya**:

Bash

```
cat docker-compose.yml
```

### Menjalankan Docker Compose

Nyalakan container dari file yaml yang baru saja dibuat:

Bash

```
docker compose up -d
```

### 📸 SS 3: Status `docker ps`

Cek semua container yang sedang berjalan dengan perintah ini dan **Screenshot hasilnya**:

Bash

```
docker ps
```

_(Tabel akan menampilkan `tugas-run`, `server-nginx`, dan `server-apache` dengan status "Up")_.

## Tahap 4: Tampilan Web dengan IP WSL

Cek IP WSL kamu dengan perintah:

Bash

```
ip -4 a show eth0
```

_(IP yang digunakan adalah **172.21.150.100**)_.

### 📸 SS 4: Bukti Nginx

Buka browser di Windows, ketik alamat `http://172.21.150.100:8080` **Screenshot** saat muncul halaman **"Welcome to nginx!"**

### 📸 SS 5: Bukti Apache2

Buka tab baru di browser, ketik alamat `http://172.21.150.100:8081` **Screenshot** saat muncul halaman **"It works!"**