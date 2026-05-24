---
title: Tugas Instalasi Docker, Nginx, dan Apache2
date: 2026-05-19
tags:
  - tugas
  - docker
  - wsl
  - tjkt
---
Karena langkah pertama tadi sudah aman, aku buatkan **panduan lengkap dari awal sampai akhir (sampai tugasmu selesai)** dalam format Obsidian.

Catatan ini sudah mencakup pembuatan `docker-compose.yml`, cara menjalankan Nginx dan Apache2, serta panduan mengambil _screenshot_ sesuai permintaan tugasmu.

Silakan salin seluruh teks di bawah ini ke dalam _note_ Obsidian-mu:

Markdown

````
---
title: Tugas Instalasi Docker, Nginx, dan Apache2
date: 2026-05-19
tags:
  - tugas
  - docker
  - nginx
  - apache2
---

# Panduan Tugas Docker: Nginx & Apache2 di WSL

> [!info] Target Pengumpulan Tugas (Screenshots)
> - [ ] Screenshot isi file `docker-compose.yml`
> - [ ] Screenshot hasil perintah `docker ps`
> - [ ] Screenshot tampilan web Nginx berhasil (localhost:8080)
> - [ ] Screenshot tampilan web Apache2 berhasil (localhost:8081)

---

## Tahap 1: Instalasi Docker & Docker Compose

Langkah ini untuk menginstal Docker Engine di sistem Ubuntu (WSL). Jalankan di terminal secara berurutan.

### 1. Persiapan Sistem & Hapus Versi Lama
*(Abaikan jika muncul tulisan `Package 'docker' is not installed` atau `Unable to locate package`)*
```bash
sudo apt update && sudo apt upgrade -y
sudo apt remove docker docker-engine docker.io containerd runc
````

### 2. Instalasi Dependensi

Bash

```
sudo apt install ca-certificates curl gnupg lsb-release -y
```

### 3. Tambahkan GPG Key & Repositori Resmi Docker

Bash

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 4. Instalasi Docker Engine

Bash

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

---

## Tahap 2: Konfigurasi Akses Docker

Agar tidak perlu mengetik `sudo` terus-menerus dan memastikan _daemon_ berjalan di WSL.

**1. Masukkan User ke Grup Docker**

Bash

```
sudo usermod -aG docker $USER
```

> [!warning] Penting Tutup terminal Ubuntu kamu, lalu buka kembali agar perubahan akses ini aktif.

**2. Jalankan Service Docker**

Bash

```
sudo service docker start
```

---

## Tahap 3: Membuat `docker-compose.yml`

Kita akan membuat satu file konfigurasi untuk menjalankan Nginx dan Apache2 secara bersamaan tanpa bentrok port.

**1. Buat folder untuk tugas ini dan masuk ke dalamnya**

Bash

```
mkdir tugas-docker-web && cd tugas-docker-web
```

**2. Buat dan edit file compose menggunakan Nano**

Bash

```
nano docker-compose.yml
```

**3. Masukkan kode berikut ke dalam editor Nano:**

> [!tip] Konfigurasi Port
> 
> - Nginx akan berjalan di port **8080**
>     
> - Apache2 akan berjalan di port **8081**
>     

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

_(Cara simpan di Nano: Tekan `Ctrl + X`, lalu tekan `Y`, lalu `Enter`)_

📸 **SCREENSHOT 1:** Ketik perintah `cat docker-compose.yml` di terminal dan _screenshot_ hasilnya.

---

## Tahap 4: Menjalankan Container & Verifikasi Akhir

**1. Jalankan Docker Compose** Perintah ini akan mengunduh image Nginx & Apache2 (httpd) lalu menjalankannya di latar belakang.

Bash

```
docker compose up -d
```

**2. Cek Status Container**

Bash

```
docker ps
```

📸 **SCREENSHOT 2:** _Screenshot_ output dari perintah `docker ps` ini. Pastikan statusnya "Up".

**3. Uji Coba Tampilan Web di Browser Windows** Buka browser favoritmu (Chrome/Edge/Firefox) di Windows, lalu akses dua alamat berikut:

- Buka `http://localhost:8080` (Akan muncul tulisan **"Welcome to nginx!"**) 📸 **SCREENSHOT 3:** _Screenshot_ halaman Nginx ini.
    
- Buka `http://localhost:8081` (Akan muncul tulisan **"It works!"**) 📸 **SCREENSHOT 4:** _Screenshot_ halaman Apache2 ini.
    

> [!success] Selesai! Jika keempat screenshot sudah terkumpul, tugasmu sudah siap untuk dikumpulkan.