#lks #cyber-security #troubleshooting #network #virtualbox


# 🛑 SOLUSI BOOTING LAMA (systemd-networkd-wait-online)
#lks #cyber-security #linux #troubleshooting #cheatsheet

> **Gejala:** VM tertahan (*stuck*) berlama-lama di layar hitam dengan teks:
> `Job systemd-networkd-wait-online.service/start running (30s / no limit)`


# 🌐 KAMUS CEPAT NETPLAN (Ubuntu Network Config)
#lks #cyber-security #linux #networking #cheatsheet

> ⚠️ **ATURAN MUTLAK NETPLAN (Format YAML):**
> 1. **DILARANG pakai tombol TAB!** Gunakan murni **Spasi** (tiap level menjorok 2 spasi).
> 2. **DILARANG duplikasi!** Jangan menulis nama *interface* yang sama (misal: `enp0s8`) dua kali dalam satu file.
> 3. **Lokasi File:** Biasanya berada di `/etc/netplan/00-installer-config.yaml` atau `50-cloud-init.yaml`.

---

## 1️⃣ TEMPLATE IP STATIS (Manual)
Gunakan ini jika panitia menyuruh mengatur IP ke angka spesifik.
*(Edit menggunakan: `sudo nano /etc/netplan/nama_file.yaml`)*

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      dhcp4: no
      dhcp6: false
      addresses:
        - 192.168.56.10/24

## 🔍 Penyebab Utama
Sistem operasi tertahan karena sedang memaksakan diri "menunggu" balasan IP otomatis (DHCP) dari jaringan. Hal ini terjadi karena konfigurasi `netplan` yang bermasalah, atau DHCP Server dari *router*/VirtualBox memang tidak merespons. Sistem akan terus menunggu sampai batas waktunya (*timeout*) habis.

---

## 🛠️ CARA MEMPERBAIKI (Pilih Salah Satu)

### 🚀 OPSI 1: Jurus Bypass Darurat (Sangat Disarankan saat Lomba)
Jika kamu tidak ingin membuang waktu 2 menit setiap kali terpaksa me-*restart* VM, matikan saja fitur "menunggu" ini. Waktu *booting* VM akan langsung melesat kembali normal.

Jalankan perintah ini di terminal:
```bash
sudo systemctl disable systemd-networkd-wait-online.service
## 🚨 Gejala (Symptoms)

1. Saat menjalankan `ip a`, _adapter_ jaringan kedua (misalnya `enp0s8`) statusnya `UP`, tetapi **tidak ada baris `inet` (IPv4 address)**.
    
2. Saat mencoba memancing IP dengan perintah `sudo dhclient enp0s8`, terminal hanya _loading_ (nge-_hang_) atau menampilkan layar hitam tanpa _output_ apa pun.
    
3. Tidak bisa di-SSH dari komputer fisik.
    

## 🔍 Akar Masalah (Root Causes)

1. **Di sisi OS Linux:** Antarmuka jaringan belum dikonfigurasi secara otomatis oleh sistem operasi saat pertama kali diinstal (hanya antarmuka pertama/NAT yang otomatis jalan).
    
2. **Di sisi VirtualBox:** Fitur **DHCP Server** untuk jalur _Host-Only Network_ dalam keadaan mati (Nonaktif), sehingga saat Linux "berteriak" meminta IP, tidak ada _server_ yang membalas.
    

## ✅ Solusi Bertahap (Playbook)

### TAHAP 1: Pembatalan dan Cek Status

Jika terminal sedang _loading_ karena `dhclient`:

1. Tekan `Ctrl + C` untuk membatalkan proses yang _hang_.
    
2. Pastikan nama _interface_ yang bermasalah dengan perintah:
    
    Bash
    
    ```
    ip a
    ```
    
    _(Catat namanya, misalnya: `enp0s8`)_.
    

### TAHAP 2: Injeksi IP Manual (Cara Cepat / Darurat)

Gunakan ini jika kamu sedang buru-buru butuh akses SSH dan tidak mau repot mengatur VirtualBox. Ini akan memasang IP statis secara instan.

1. Tambahkan IP statis (sesuaikan dengan _subnet default_ VirtualBox, biasanya `192.168.56.x`):
    
    Bash
    
    ```
    sudo ip addr add 192.168.56.10/24 dev enp0s8
    ```
    
2. Nyalakan ulang _interface_ tersebut:
    
    Bash
    
    ```
    sudo ip link set dev enp0s8 up
    ```
    
3. Langsung buka terminal di Windows dan eksekusi SSH ke `192.168.56.10`. _(Kekurangan: IP ini akan hilang saat VM di-restart)._
    

### TAHAP 3: Mengaktifkan DHCP di VirtualBox (Agar `dhclient` Jalan)

Ini adalah akar masalah mengapa `dhclient` nge-_hang_.

1. Buka jendela utama aplikasi **VirtualBox**.
    
2. Klik menu **Tools** ➔ **Network Manager** (atau Host Network Manager).
    
3. Di tab **Host-only Networks**, pilih adapter jaringan yang terhubung ke VM kamu.
    
4. Buka tab **DHCP Server**.
    
5. Centang **Enable Server**.
    
    - _Server Address:_ `192.168.56.100`
        
    - _Server Mask:_ `255.255.255.0`
        
    - _Lower Address:_ `192.168.56.101`
        
    - _Upper Address:_ `192.168.56.254`
        
6. Klik **Apply**.
    
7. Kembali ke VM Linux, jalankan ulang:
    
    Bash
    
    ```
    sudo dhclient enp0s8
    ```
    
    _(Terminal tidak akan hang lagi, dan IP akan langsung terisi)._
    

### TAHAP 4: Konfigurasi Permanen di Linux (Praktik Terbaik)

Agar tidak perlu mengetik `dhclient` atau `ip addr add` setiap kali _server_ dihidupkan ulang, konfigurasikan Netplan.

1. Cari tahu nama _file_ konfigurasi jaringan:
    
    Bash
    
    ```
    ls /etc/netplan/
    ```
    
2. Edit _file_ tersebut (misal namanya `00-installer-config.yaml`):
    
    Bash
    
    ```
    sudo nano /etc/netplan/00-installer-config.yaml
    ```
    
3. Tambahkan konfigurasi `enp0s8` di bawah `ethernets`. **WAJIB gunakan spasi, dilarang menggunakan Tab!**
    
    YAML
    
    ```
    network:
      ethernets:
        enp0s3:
          dhcp4: true
        enp0s8:
          dhcp4: true
      version: 2
    ```
    
4. Terapkan konfigurasi:
    
    Bash
    
    ```
    sudo netplan apply
    ```
    
5. Verifikasi bahwa IP sudah menempel permanen:
    
    Bash
    
    ```
    ip a
    ```