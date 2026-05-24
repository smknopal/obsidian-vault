# 01 — Network Service Security
#lks #cyber-security #linux #hardening #ssh #fail2ban #ufw #network

> **Peran:** Defender | **OS:** Debian/Ubuntu-based | **Konteks:** LKS Cyber Security 2026
> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Network Service Security

---

## 🎯 Tujuan

Mengamankan jalur komunikasi jaringan (terutama SSH) dari:
- Akses tidak sah via password lemah
- Serangan brute-force dari luar
- Port yang terbuka tidak perlu

---

> 🧠 **Sebelum mulai, pahami ini dulu:**
> SSH adalah pintu utama masuk ke server Linux. Kalau SSH tidak aman → server bisa dikuasai attacker.
> UFW adalah penjaga gerbang jaringan.
> Fail2ban adalah alarm + pengunci otomatis.
>
> Analogi rumah:
> - **SSH** = pintu utama rumah
> - **UFW** = pagar + satpam di depan
> - **Fail2ban** = CCTV yang otomatis blokir tamu yang mencurigakan

---

## 1. SSH — Autentikasi Public Key (Tanpa Password)

### Kenapa Harus Pakai Key, Bukan Password?

Password bisa ditebak dengan brute-force. SSH Key tidak bisa ditebak karena menggunakan matematika kriptografi yang sangat rumit.

```
TANPA KEY (berbahaya):
Attacker → coba password "admin123" → coba "password" → coba "root123" → ...
Server hanya bisa menolak jika fail2ban aktif

DENGAN KEY (aman):
Attacker → tidak punya private key → LANGSUNG DITOLAK
```

### Konsep Dasar

SSH Key Auth menggunakan pasangan kunci **asymmetric** (dua kunci yang berpasangan):
- **Private Key** → disimpan di **client** (laptopmu). **JANGAN DIBAGI KE SIAPAPUN.**
- **Public Key** → dipasang di **server**. Boleh diketahui umum.

Analogi: Public key = gembok. Private key = kunci gembok-nya.
Kamu memasang gembok di server, dan hanya kamu yang punya kuncinya.

```
Client (Kamu)          Server
-----------            ------
Private Key   <---->   Public Key (di ~/.ssh/authorized_keys)
"Kunci"                "Gembok"
```

---

### Langkah 1: Buat Key Pair di Client

```bash
ssh-keygen -t rsa -b 4096
```

Penjelasan parameter:
- `-t rsa` → pakai algoritma RSA
- `-b 4096` → panjang kunci 4096 bit (makin panjang = makin aman)

Saat ditanya lokasi simpan → tekan **Enter** saja (simpan default).
Saat ditanya passphrase → boleh diisi atau dikosongkan (isi = lebih aman).

**Hasilnya:**
```
~/.ssh/id_rsa       ← Private Key (RAHASIA MUTLAK! JANGAN DIBAGI!)
~/.ssh/id_rsa.pub   ← Public Key (yang dipasang di server)
```

**Cek hasilnya:**
```bash
ls -la ~/.ssh/
# Output yang diharapkan:
# -rw------- 1 kamu kamu 3389 May 24 id_rsa        ← permission 600 ✅
# -rw-r--r-- 1 kamu kamu  743 May 24 id_rsa.pub    ← permission 644 ✅
```

---

### Langkah 2: Kirim Public Key ke Server

**Cara otomatis (paling mudah):**
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server_ip
```

Perintah ini otomatis memasukkan public key ke server di `~/.ssh/authorized_keys`.

**Cara manual (jika ssh-copy-id tidak tersedia):**
```bash
# Di CLIENT: lihat isi public key
cat ~/.ssh/id_rsa.pub
# Copy semua outputnya (mulai dari "ssh-rsa AAAA..." sampai akhir)

# Di SERVER:
mkdir -p ~/.ssh                          # Buat folder .ssh jika belum ada
chmod 700 ~/.ssh                         # Set permission folder
nano ~/.ssh/authorized_keys              # Buka/buat file authorized_keys
# Paste isi public key di sini, simpan
chmod 600 ~/.ssh/authorized_keys         # Set permission file
```

---

### Langkah 3: Konfigurasi SSH Server

Ini adalah langkah terpenting. File yang kita edit: `/etc/ssh/sshd_config`

```bash
# SELALU test config sebelum restart!
sudo sshd -t

# Baru edit
sudo nano /etc/ssh/sshd_config
```

**Konfigurasi lengkap yang aman:**

```ini
# ========== PORT ==========
Port 2222
# Ganti dari default 22. Port 22 adalah target utama scanner otomatis.
# Ini bukan keamanan utama, tapi mengurangi noise di log.

# ========== AUTENTIKASI ==========
PubkeyAuthentication yes
# Izinkan login pakai key

PasswordAuthentication no
# MATIKAN login pakai password!
# PERINGATAN: Pastikan key sudah terpasang sebelum set ini ke 'no'!

PermitEmptyPasswords no
# LARANG password kosong

# ========== AKSES ROOT ==========
PermitRootLogin no
# Larang login langsung sebagai root!
# Attacker selalu mencoba login sebagai root. Matikan ini!

# ========== PEMBATASAN PERCOBAAN ==========
MaxAuthTries 3
# Maksimal 3 percobaan login per koneksi

MaxSessions 5
# Maksimal 5 sesi SSH bersamaan

LoginGraceTime 20
# Batas waktu 20 detik untuk proses login
# Setelah 20 detik tidak berhasil → koneksi diputus

# ========== PEMBATASAN USER ==========
AllowUsers alice bob
# Hanya user 'alice' dan 'bob' yang boleh SSH
# ATAU gunakan AllowGroups:
# AllowGroups sshusers

# DenyUsers untuk user yang dilarang:
# DenyUsers baduser hacker

# ========== KEAMANAN TAMBAHAN ==========
X11Forwarding no
# Matikan X11 forwarding (GUI over SSH — tidak perlu di server)

AllowTcpForwarding no
# Matikan TCP forwarding (bisa disalahgunakan untuk tunneling)

GatewayPorts no
# Matikan gateway ports

PermitUserEnvironment no
# Cegah override environment variable via SSH

StrictModes yes
# SSH akan cek permission file ~/.ssh secara ketat
# Jika permission salah → login ditolak (ini perlindungan tambahan)

UseDNS no
# Percepat login dengan skip DNS lookup

LogLevel VERBOSE
# Catat semua detail login ke log (penting untuk monitoring)

AuthorizedKeysFile .ssh/authorized_keys
# Lokasi file authorized_keys

# ========== SESSION TIMEOUT ==========
ClientAliveInterval 300
# Kirim sinyal ke client setiap 300 detik (5 menit)
ClientAliveCountMax 2
# Setelah 2 kali tidak respon → putus koneksi
# Efek: Sesi idle lebih dari 10 menit (300 x 2 = 600 detik) → otomatis putus

# ========== BANNER ==========
Banner /etc/ssh/banner.txt
# Tampilkan pesan peringatan sebelum login
```

**Setelah selesai edit:**
```bash
# WAJIB: Test dulu ada error tidak?
sudo sshd -t
# Jika tidak ada output = tidak ada error ✅
# Jika ada output error = perbaiki dulu sebelum restart!

# Restart SSH
sudo systemctl restart sshd

# Verifikasi SSH berjalan
sudo systemctl status sshd
```

> ⚠️ **PERINGATAN PENTING — JANGAN SAMPAI TERKUNCI DARI SERVER!**
> Urutan yang BENAR:
> 1. Pasang public key di server
> 2. Test login dengan key (buka terminal baru, jangan tutup yang lama!)
> 3. Baru set `PasswordAuthentication no`
> 4. Restart SSH
> 5. Test lagi dari terminal baru
>
> Kalau kamu langsung set `PasswordAuthentication no` tanpa pasang key → kamu TIDAK BISA MASUK LAGI!

---

### Langkah 4: Buat SSH Banner

Banner adalah pesan peringatan yang muncul sebelum login. Penting untuk aspek legal.

```bash
sudo nano /etc/ssh/banner.txt
```

Isi:
```
============================================================
  SISTEM INI HANYA UNTUK PENGGUNA YANG BERWENANG.
  Semua aktivitas dicatat dan dipantau.
  Akses tanpa izin akan ditindaklanjuti secara hukum.
============================================================
```

Banner sudah diaktifkan di sshd_config di atas dengan baris `Banner /etc/ssh/banner.txt`.

---

### Langkah 5: Permission File SSH yang Benar

```bash
# Di SERVER:
chmod 700 ~/.ssh                        # Folder .ssh
chmod 600 ~/.ssh/authorized_keys        # File authorized_keys

# Di CLIENT:
chmod 700 ~/.ssh                        # Folder .ssh
chmod 600 ~/.ssh/id_rsa                 # Private key (WAJIB ketat!)
chmod 644 ~/.ssh/id_rsa.pub             # Public key

# Verifikasi:
ls -la ~/.ssh/
```

**Tabel permissions:**

| File/Folder | Harus | Kalau Salah |
|-------------|-------|-------------|
| `~/.ssh/` | `700` | SSH tidak mau pakai key di dalamnya |
| `~/.ssh/authorized_keys` | `600` | SSH tidak mau baca file ini |
| `~/.ssh/id_rsa` (private) | `600` | SSH tidak mau pakai key ini |
| `~/.ssh/id_rsa.pub` (public) | `644` | Tidak bermasalah, tapi standarnya begini |

---

### Troubleshooting SSH

| Masalah | Kemungkinan Penyebab | Solusi |
|---------|---------------------|--------|
| `Permission denied (publickey)` | Permission ~/.ssh salah | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| Key tidak dikenali | Isi authorized_keys tidak cocok | Cek dengan `cat ~/.ssh/authorized_keys` |
| Tidak bisa konek sama sekali | SSH mati / port salah | `sudo systemctl status sshd` |
| Koneksi ditolak | UFW memblok | `sudo ufw status` |
| Koneksi lambat | DNS lookup | Tambah `UseDNS no` di sshd_config |

```bash
# Debug SSH dari sisi client (tampilkan detail proses koneksi)
ssh -vvv user@server_ip

# Cek log SSH di server
sudo tail -f /var/log/auth.log

# Test syntax konfigurasi SSH
sudo sshd -t
```

---

## 2. UFW — Firewall Dasar

UFW (Uncomplicated Firewall) = cara mudah mengatur firewall di Ubuntu/Debian.

> 🧠 **Analogi:** UFW seperti pagar + satpam di pintu masuk rumah. Dia menentukan siapa boleh masuk dari pintu mana.

### Prinsip Dasar Firewall

```
TANPA firewall:
Internet → [semua port terbuka] → Server

DENGAN firewall (default deny):
Internet → [UFW] → Hanya port 22 (SSH) dan 80 (HTTP) yang boleh masuk
                 → Port lain otomatis diblokir
```

### Setup UFW Step by Step

```bash
# STEP 1: Cek status sekarang
sudo ufw status verbose

# STEP 2: Set policy default DULU (sebelum enable!)
sudo ufw default deny incoming    # Tolak SEMUA koneksi masuk
sudo ufw default allow outgoing   # Izinkan SEMUA koneksi keluar

# STEP 3: Izinkan SSH DULU sebelum enable (atau kamu kekunci!)
sudo ufw allow 22/tcp             # Jika SSH di port 22
# ATAU:
sudo ufw allow 2222/tcp           # Jika SSH di port 2222

# STEP 4: Baru aktifkan UFW
sudo ufw enable

# STEP 5: Verifikasi
sudo ufw status verbose
```

### Izinkan dan Blokir Port

```bash
# ===== IZINKAN PORT =====
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS

# Izinkan dari IP spesifik saja (lebih aman!)
sudo ufw allow from 192.168.1.100 to any port 22     # Hanya 1 IP boleh SSH
sudo ufw allow from 192.168.1.0/24 to any port 443  # Seluruh subnet boleh HTTPS

# ===== BLOKIR PORT =====
sudo ufw deny 21/tcp       # Blokir FTP
sudo ufw deny 23/tcp       # Blokir Telnet
sudo ufw deny 3306/tcp     # Blokir MySQL dari luar
sudo ufw deny 6379/tcp     # Blokir Redis dari luar

# ===== RATE LIMITING (anti brute-force) =====
sudo ufw limit 22/tcp
# Otomatis block IP yang coba koneksi lebih dari 6 kali dalam 30 detik
```

### Kelola Aturan UFW

```bash
# Lihat semua aturan dengan nomor
sudo ufw status numbered

# Output contoh:
# Status: active
#      To                         Action      From
#      --                         ------      ----
# [ 1] 22/tcp                     ALLOW IN    Anywhere
# [ 2] 80/tcp                     ALLOW IN    Anywhere
# [ 3] 443/tcp                    ALLOW IN    Anywhere
# [ 4] 21/tcp                     DENY IN     Anywhere

# Hapus aturan berdasarkan nomor
sudo ufw delete 4              # Hapus aturan nomor 4

# Hapus aturan berdasarkan definisi
sudo ufw delete deny 21/tcp

# Reload UFW (terapkan perubahan)
sudo ufw reload

# Reset semua aturan (hati-hati! dari awal lagi)
sudo ufw reset
```

---

## 3. Fail2ban — Pencegah Brute-Force Otomatis

Fail2ban memantau log secara real-time dan otomatis mem-block IP yang mencoba brute-force.

> 🧠 **Analogi:** Fail2ban seperti satpam yang melihat CCTV. Kalau ada orang yang 3 kali salah ketuk kode pintu → langsung dikunci dari luar.

### Cara Kerja

```
[/var/log/auth.log]
"Failed password for root from 1.2.3.4 port 12345"    ← baris 1
"Failed password for root from 1.2.3.4 port 23456"    ← baris 2
"Failed password for root from 1.2.3.4 port 34567"    ← baris 3
                    ↓
               [fail2ban]
         Deteksi: IP 1.2.3.4 sudah 3x gagal!
                    ↓
         Tambahkan aturan ke iptables:
         "DROP semua paket dari 1.2.3.4"
                    ↓
         IP 1.2.3.4 tidak bisa konek ke server selama 1 jam
```

### Install Fail2ban

```bash
sudo apt install fail2ban -y
```

### Konfigurasi

**PENTING:** Jangan edit `jail.conf` langsung! Buat salinan `jail.local` yang akan di-baca oleh fail2ban.

```bash
# Buat salinan
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit file .local (BUKAN .conf!)
sudo nano /etc/fail2ban/jail.local
```

Isi konfigurasi yang perlu kamu set:

```ini
[DEFAULT]
# Setting default berlaku untuk semua jail

# Jangan pernah ban IP ini (localhost + IP admin kamu)
ignoreip = 127.0.0.1/8 ::1

# Hitung percobaan gagal dalam 10 menit terakhir
findtime = 600

# Ban setelah 3 kali gagal
maxretry = 3

# Durasi ban: 3600 detik = 1 jam
bantime = 3600
# Gunakan bantime = -1 untuk PERMANENT BAN (sangat bagus untuk lomba!)

# Kirim email notifikasi (jika ada mail server)
# destemail = admin@example.com

[sshd]
# Konfigurasi khusus untuk SSH
enabled = true

# Sesuaikan dengan port SSH-mu!
port = 2222
# Jika port masih 22, ubah ke:
# port = 22

# Filter yang digunakan (file bawaan untuk SSH)
filter = sshd

# File log yang dipantau
logpath = /var/log/auth.log

# Override default
maxretry = 3
bantime = 3600
```

### Aktifkan dan Kelola Fail2ban

```bash
# Aktifkan agar jalan otomatis saat boot
sudo systemctl enable fail2ban

# Jalankan sekarang
sudo systemctl start fail2ban

# Cek status
sudo systemctl status fail2ban

# Restart jika ada perubahan konfigurasi
sudo systemctl restart fail2ban
```

### Perintah Monitoring Fail2ban

```bash
# Lihat status semua jail
sudo fail2ban-client status

# Lihat detail jail SSH (berapa IP yang di-ban, dll)
sudo fail2ban-client status sshd

# Output contoh:
# Status for the jail: sshd
# |- Filter
# |  |- Currently failed: 2
# |  |- Total failed:     8
# `- Actions
#    |- Currently banned: 1
#    |- Total banned:     3
#    `- Banned IP list:   192.168.1.100

# Lihat log fail2ban
sudo tail -f /var/log/fail2ban.log

# Unban IP tertentu
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Ban IP secara manual
sudo fail2ban-client set sshd banip 192.168.1.200
```

---

## 4. TCP Wrappers (hosts.allow & hosts.deny)

Lapisan keamanan tambahan. Catatan: sudah deprecated di Ubuntu 22+, tapi masih sering muncul di lomba.

```bash
# File yang diizinkan
sudo nano /etc/hosts.allow
```

```
sshd : 192.168.1.0/24    # Hanya jaringan 192.168.1.x yang boleh SSH
ALL  : 127.0.0.1         # Localhost boleh akses semua
```

```bash
# File yang ditolak
sudo nano /etc/hosts.deny
```

```
# Tolak semua yang tidak ada di hosts.allow
ALL : ALL
```

> **Urutan pemrosesan:**
> 1. Cek `hosts.allow` dulu → kalau ada izin → langsung masuk
> 2. Cek `hosts.deny` → kalau ada larangan → ditolak
> 3. Tidak ada di keduanya → diizinkan (makanya isi `hosts.deny` dengan `ALL : ALL`)

---

## ✅ Checklist Network Service Security (Untuk Lomba!)

### SSH
- [ ] Key pair sudah dibuat (`ssh-keygen -t rsa -b 4096`)
- [ ] Public key sudah dipasang di server (`~/.ssh/authorized_keys`)
- [ ] Test login dengan key berhasil sebelum nonaktifkan password
- [ ] `PasswordAuthentication no` di `sshd_config`
- [ ] `PermitRootLogin no` di `sshd_config`
- [ ] `PermitEmptyPasswords no` di `sshd_config`
- [ ] `MaxAuthTries 3` di `sshd_config`
- [ ] `X11Forwarding no` di `sshd_config`
- [ ] `AllowTcpForwarding no` di `sshd_config`
- [ ] `ClientAliveInterval 300` dan `ClientAliveCountMax 2`
- [ ] `AllowUsers` atau `AllowGroups` dikonfigurasi
- [ ] `StrictModes yes` di `sshd_config`
- [ ] `PermitUserEnvironment no` di `sshd_config`
- [ ] Permission `~/.ssh` = 700, `authorized_keys` = 600, private key = 600
- [ ] Port SSH bukan default 22
- [ ] Banner peringatan aktif
- [ ] `sudo sshd -t` tidak ada error

### UFW
- [ ] UFW aktif (`sudo ufw status`)
- [ ] Default policy: `deny incoming`, `allow outgoing`
- [ ] Port SSH diizinkan sebelum enable UFW
- [ ] Port berbahaya (21, 23, 3306, dll.) diblokir
- [ ] Rate limiting aktif untuk SSH (`sudo ufw limit ssh`)

### Fail2ban
- [ ] Fail2ban terinstall (`dpkg -l | grep fail2ban`)
- [ ] Fail2ban aktif (`sudo systemctl status fail2ban`)
- [ ] File `jail.local` dikonfigurasi (bukan `jail.conf`)
- [ ] Jail SSH dikonfigurasi di `jail.local`
- [ ] `maxretry = 3` di konfigurasi
- [ ] `bantime = 3600` (atau -1 untuk permanent)
- [ ] Verifikasi: `sudo fail2ban-client status sshd`

---

## 🔗 Navigasi

← `00b_Konsep_Dasar_Linux`
→ `02_PAM_Password_Complexity`
