# 04 — Dangerous & Exposed Services
#lks #cyber-security #linux #hardening #services #ports #network

> **Peran:** Defender | **OS:** Debian/Ubuntu-based | **Konteks:** LKS Cyber Security 2026
> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Dangerous/Exposed Services

---

## 🎯 Tujuan

Menerapkan prinsip **Least Privilege** di level jaringan:
> "Jika sebuah layanan tidak diperlukan → MATIKAN.
> Jika memang perlu → BATASI aksesnya sekecil mungkin."

---

> 🧠 **Analogi:**
> Bayangkan server kamu adalah rumah.
> Setiap service yang berjalan = pintu atau jendela yang terbuka.
> Semakin banyak yang terbuka = semakin banyak jalan masuk untuk maling.
> Tutup semua pintu dan jendela yang tidak perlu!

---

## 1. Identifikasi Layanan yang Berjalan (LAKUKAN INI PERTAMA!)

Sebelum hardening, kenali dulu APA SAJA yang berjalan di server.

### Perintah Cek Port dan Service

```bash
# ===== CARA 1: ss (socket statistics) - YANG PALING UMUM =====
sudo ss -tulnp

# Penjelasan flag:
# -t = tampilkan TCP
# -u = tampilkan UDP
# -l = hanya yang sedang LISTEN (menunggu koneksi)
# -n = tampilkan angka port (bukan nama layanan)
# -p = tampilkan nama proses/PID yang memakainya

# ===== CARA 2: netstat (jika ss tidak tersedia) =====
sudo netstat -tulnp

# ===== CARA 3: lsof =====
sudo lsof -i -P -n | grep LISTEN

# ===== CARA 4: nmap (scan dari luar) =====
sudo nmap -sS -O localhost   # Scan localhost sendiri
nmap -sV 192.168.1.10        # Scan server dari sisi luar (simulasi attacker)
```

### Cara Baca Output `ss -tulnp`

```
Netid  State    Local Address:Port   Peer Address:Port  Process
tcp    LISTEN   0.0.0.0:22           0.0.0.0:*          ("sshd",pid=1234)
tcp    LISTEN   127.0.0.1:3306       0.0.0.0:*          ("mysqld",pid=5678)
tcp    LISTEN   0.0.0.0:21           0.0.0.0:*          ("vsftpd",pid=9012)
tcp    LISTEN   0.0.0.0:6379         0.0.0.0:*          ("redis-server",pid=3456)
udp    UNCONN   0.0.0.0:161          0.0.0.0:*          ("snmpd",pid=7890)
```

**Membaca kolom `Local Address:Port`:**

| Yang Kamu Lihat | Artinya | Status Keamanan |
|----------------|---------|-----------------|
| `127.0.0.1:3306` | Hanya bisa diakses dari server itu sendiri | ✅ AMAN |
| `0.0.0.0:22` | Terbuka ke semua IP di jaringan | ⚠️ Perlu dikendalikan |
| `0.0.0.0:21` | FTP terbuka ke semua IP | ❌ MATIKAN SEGERA! |
| `:::80` | IPv6 terbuka ke semua IP | ⚠️ Perlu dikendalikan |

**Analisis dari output di atas:**
```
ssh  port 22  → 0.0.0.0 = terbuka semua → kontrol via firewall
mysql port 3306 → 127.0.0.1 = hanya lokal → AMAN ✅
ftp  port 21  → 0.0.0.0 = BAHAYA → MATIKAN SEKARANG! ❌
redis port 6379 → 0.0.0.0 = BAHAYA → BATASI ke 127.0.0.1 ❌
snmp port 161 → UDP, 0.0.0.0 = BAHAYA → MATIKAN! ❌
```

### Cek Service yang Aktif

```bash
# Semua service yang sedang jalan
sudo systemctl list-units --type=service --state=running

# Semua service yang auto-start saat boot (yang enabled)
sudo systemctl list-unit-files --type=service | grep enabled

# Detail satu service
sudo systemctl status vsftpd
```

---

## 2. Daftar Layanan Berbahaya

### 🔴 SANGAT BERBAHAYA — Matikan Segera!

| Port | Layanan | Kenapa Berbahaya | Tindakan |
|------|---------|-----------------|---------|
| **21** | FTP (vsftpd) | Data & password dikirim **plain text** → bisa disniffer | Matikan + Purge |
| **23** | Telnet | Semua komunikasi **plain text** → attacker baca semua | Matikan + Purge |
| **512** | rexec | Remote exec tanpa enkripsi | Matikan + Purge |
| **513** | rlogin | Remote login tanpa enkripsi | Matikan + Purge |
| **514** | rsh | Remote shell tanpa enkripsi | Matikan + Purge |
| **2049** | NFS | Sering salah konfigurasi, bisa expose seluruh filesystem | Matikan jika tidak perlu |
| **111** | rpcbind | Sering dieksploitasi untuk pivot | Matikan jika tidak perlu |
| **161** | SNMP (UDP) | Default community "public" → info leakage | Matikan atau konfigurasi ulang |
| **79** | Finger | Memberikan info user yang login → info leakage | Matikan + Purge |

> 🧠 **Kenapa plain text berbahaya?**
> Plain text = data tidak dienkripsi.
> Attacker yang ada di jaringan yang sama bisa "mendengarkan" traffic dengan tools seperti Wireshark.
> Dia bisa membaca username, password, dan semua data yang dikirim!
> SSH menggantikan Telnet karena SSH mengenkripsi semua traffic.

### 🟡 PERLU PERHATIAN — Batasi Aksesnya

| Port | Layanan | Risiko & Solusi |
|------|---------|-----------------|
| **3306** | MySQL | Jika listen di `0.0.0.0` → ekspos ke internet. Ubah ke `127.0.0.1` |
| **5432** | PostgreSQL | Sama seperti MySQL |
| **6379** | Redis | Sering tanpa auth! Harus bind localhost + set password |
| **27017** | MongoDB | Sering tanpa auth! Harus bind localhost |
| **5900** | VNC | Enkripsi lemah. Batasi via firewall |
| **8080** | HTTP alternatif | Admin panel sering di sini. Batasi atau password protect |
| **9200** | Elasticsearch | Sering tanpa auth! Harus bind localhost |

---

## 3. Cara Matikan Layanan Berbahaya

### Template Umum

```bash
# Langkah 1: Hentikan service sekarang
sudo systemctl stop NAMA_SERVICE

# Langkah 2: Cegah service hidup lagi saat reboot (WAJIB!)
sudo systemctl disable NAMA_SERVICE

# Langkah 3: Hapus package (opsional tapi lebih aman)
sudo apt purge NAMA_PACKAGE -y

# Langkah 4: Verifikasi sudah mati
sudo systemctl status NAMA_SERVICE
# Harus tampil: "inactive (dead)" atau "not found"

# Langkah 5: Verifikasi port sudah tidak ada
sudo ss -tulnp | grep PORT_NOMOR
# Harus tidak ada output
```

### Contoh Nyata Setiap Service Berbahaya

```bash
# ===== FTP (port 21) =====
sudo systemctl stop vsftpd
sudo systemctl disable vsftpd
sudo apt purge vsftpd -y
# Verifikasi:
sudo ss -tulnp | grep :21     # Harus kosong ✅

# ===== Telnet (port 23) =====
sudo systemctl stop telnetd
sudo systemctl disable telnetd
sudo apt purge telnetd telnet -y
# Verifikasi:
sudo ss -tulnp | grep :23     # Harus kosong ✅

# ===== rpcbind (port 111) =====
sudo systemctl stop rpcbind
sudo systemctl disable rpcbind
sudo apt purge rpcbind -y

# ===== NFS =====
sudo systemctl stop nfs-server nfs-kernel-server
sudo systemctl disable nfs-server nfs-kernel-server
sudo apt purge nfs-kernel-server -y

# ===== SNMP (port 161) =====
sudo systemctl stop snmpd
sudo systemctl disable snmpd
sudo apt purge snmpd -y

# ===== Finger (port 79) =====
sudo apt purge finger fingerd -y
```

---

## 4. Perbaiki Service Database yang Terbuka

### MySQL — Ubah Bind Address

**Cek kondisi sekarang:**
```bash
sudo ss -tulnp | grep 3306
# Jika terlihat 0.0.0.0:3306 → BERBAHAYA! Harus diperbaiki
```

**Perbaikan:**
```bash
# Cari file konfigurasi MySQL
sudo find /etc/mysql -name "*.cnf" 2>/dev/null
# Biasanya: /etc/mysql/mysql.conf.d/mysqld.cnf

sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Cari dan ubah baris `bind-address`:
```ini
[mysqld]
bind-address = 127.0.0.1
# Ubah dari 0.0.0.0 menjadi 127.0.0.1
# Jika tidak ada baris ini → tambahkan di bawah [mysqld]
```

```bash
# Restart MySQL
sudo systemctl restart mysql

# Verifikasi (harus tampil 127.0.0.1:3306)
sudo ss -tulnp | grep 3306
```

---

### PostgreSQL — Ubah Listen Address

```bash
# Cari file konfigurasi
sudo find /etc/postgresql -name "postgresql.conf"
# Biasanya: /etc/postgresql/*/main/postgresql.conf

sudo nano /etc/postgresql/*/main/postgresql.conf
```

```ini
# Ubah dari '*' ke 'localhost'
listen_addresses = 'localhost'
```

```bash
sudo systemctl restart postgresql
sudo ss -tulnp | grep 5432
# Harus tampil 127.0.0.1:5432 ✅
```

---

### Redis — Bind Localhost + Password

```bash
sudo nano /etc/redis/redis.conf
```

```ini
# Ubah bind address ke localhost saja
bind 127.0.0.1

# Tambahkan password autentikasi (wajib!)
requirepass PasswordKuatRedis2026!@#

# Nonaktifkan perintah berbahaya (opsional tapi bagus)
rename-command FLUSHDB ""       # Larang hapus semua data
rename-command FLUSHALL ""      # Larang hapus semua database
rename-command CONFIG ""        # Larang ubah konfigurasi
rename-command DEBUG ""         # Larang perintah debug
```

```bash
sudo systemctl restart redis
sudo ss -tulnp | grep 6379
# Harus tampil 127.0.0.1:6379 ✅
```

---

### MongoDB — Bind Localhost

```bash
sudo nano /etc/mongod.conf
```

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1
  # Ubah dari 0.0.0.0 ke 127.0.0.1
```

```bash
sudo systemctl restart mongod
sudo ss -tulnp | grep 27017
```

---

## 5. NFS — Hardening jika Harus Tetap Jalan

```bash
# Cek ekspor yang ada
cat /etc/exports
```

**Konfigurasi BURUK (jangan lakukan!):**
```
/home  *(rw,no_root_squash)   ← Semua orang bisa akses dengan hak root!
```

**Konfigurasi BAIK:**
```bash
sudo nano /etc/exports
```

```
/data  192.168.1.0/24(ro,sync,no_subtree_check,root_squash)
# ro = read-only (hanya baca)
# sync = tulis langsung ke disk
# no_subtree_check = lebih aman saat rename file
# root_squash = map root dari client ke nobody (cegah root privilege dari client!)
```

```bash
sudo exportfs -ra
sudo systemctl restart nfs-server
```

---

## 6. SNMP — Hardening jika Harus Tetap Jalan

```bash
sudo nano /etc/snmp/snmpd.conf
```

```ini
# JANGAN pakai community string "public" (default, mudah ditebak!)
# Ganti ke string yang unik dan random
com2sec notConfigUser default  my_custom_secret_string_2026

# Batasi akses hanya dari IP tertentu (lebih aman)
com2sec localnet  192.168.1.0/24  my_custom_secret_string_2026

# Lebih baik: gunakan SNMPv3 dengan autentikasi (lebih aman dari v1/v2)
```

```bash
sudo systemctl restart snmpd
```

---

## 7. Verifikasi Final

Setelah semua konfigurasi, lakukan verifikasi menyeluruh:

```bash
# Cek port yang masih terbuka
sudo ss -tulnp

# Yang BOLEH masih terbuka (contoh minimal):
# ssh (22 atau 2222) → untuk akses admin
# http/https (80/443) → jika ada webserver

# Yang TIDAK BOLEH terbuka:
# ftp (21), telnet (23), rpcbind (111), snmpd (161)
# mysql/postgres/redis ke 0.0.0.0

# Scan dengan nmap untuk double-check
sudo nmap -sS localhost

# Cek service yang masih enabled (auto-start)
sudo systemctl list-unit-files --type=service | grep enabled
```

---

## ✅ Checklist Dangerous Services (Untuk Lomba!)

### Langkah Identifikasi
- [ ] Jalankan `sudo ss -tulnp` dan catat semua port
- [ ] Jalankan `systemctl list-units --type=service --state=running`

### Matikan Service Berbahaya
- [ ] FTP (port 21) → `systemctl stop vsftpd && systemctl disable vsftpd`
- [ ] Telnet (port 23) → matikan + purge
- [ ] rpcbind (port 111) → matikan jika tidak pakai NFS
- [ ] SNMP (port 161) → matikan atau konfigurasi ulang
- [ ] Finger (port 79) → purge
- [ ] rsh/rexec/rlogin (512/513/514) → purge

### Perbaiki Database
- [ ] MySQL: `bind-address = 127.0.0.1` → `ss -tulnp | grep 3306` harus tampil `127.0.0.1`
- [ ] PostgreSQL: `listen_addresses = 'localhost'` → verifikasi
- [ ] Redis: `bind 127.0.0.1` + `requirepass` → verifikasi
- [ ] MongoDB: `bindIp: 127.0.0.1` → verifikasi

### Verifikasi
- [ ] `sudo ss -tulnp` tidak menampilkan port berbahaya
- [ ] Semua service yang dimatikan juga sudah di-`disable`
- [ ] `nmap localhost` menunjukkan hanya port yang diperlukan

---

## 🔗 Navigasi

← `03_PAM_Account_Lockout`
→ `05_Common_Linux_Misconfigurations`
