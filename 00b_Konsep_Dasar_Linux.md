# 00b — Konsep Dasar Linux (Fondasi Wajib!)
#lks #cyber-security #linux #dasar #fondasi

> **⚠️ File ini BARU — tidak ada di materi aslimu!**
> Baca ini dulu sebelum masuk ke file 01–06.
> Kalau kamu skip ini, kamu akan sering bingung dengan perintah-perintah di file berikutnya.

---

## 🎯 Tujuan

Memahami konsep dasar Linux yang **wajib dikuasai** sebelum bisa melakukan hardening:
- Struktur direktori Linux
- Sistem user dan group
- Sistem permission (hak akses file)
- Proses dan service
- Cara membaca output perintah

---

## Bagian 1 — Struktur Direktori Linux

Di Linux, **semua adalah file** dan semua berawal dari satu titik: `/` (root).

```
/                   ← Root direktori (induk dari semua)
├── /etc/           ← File konfigurasi sistem (SERING KAMU EDIT!)
│   ├── passwd      ← Daftar user
│   ├── shadow      ← Password user (terenkripsi)
│   ├── group       ← Daftar group
│   ├── sudoers     ← Siapa boleh pakai sudo
│   ├── ssh/        ← Konfigurasi SSH
│   ├── pam.d/      ← Konfigurasi PAM
│   └── security/   ← Konfigurasi keamanan (faillock, pwquality)
│
├── /var/log/       ← File log sistem (SERING KAMU BACA!)
│   ├── auth.log    ← Log login/SSH/sudo
│   ├── syslog      ← Log sistem umum
│   └── audit/      ← Log auditd
│
├── /home/          ← Folder home user biasa (/home/alice, /home/bob)
├── /root/          ← Folder home user root
├── /bin/           ← Program dasar (ls, cat, cp, dll)
├── /sbin/          ← Program admin sistem (shutdown, fdisk, dll)
├── /usr/bin/       ← Program tambahan user
├── /usr/sbin/      ← Program tambahan admin
├── /tmp/           ← File sementara (BERBAHAYA jika tidak dikonfigurasi)
├── /proc/          ← Informasi proses (virtual filesystem)
└── /sys/           ← Informasi hardware/kernel (virtual filesystem)
```

**Yang paling sering kamu akses saat hardening:**

| Direktori/File | Fungsi |
|----------------|--------|
| `/etc/ssh/sshd_config` | Konfigurasi SSH server |
| `/etc/pam.d/` | Konfigurasi autentikasi PAM |
| `/etc/security/` | Konfigurasi keamanan (pwquality, faillock) |
| `/etc/sudoers` | Konfigurasi sudo |
| `/var/log/auth.log` | Log login dan autentikasi |
| `/etc/sysctl.conf` | Konfigurasi kernel |

---

## Bagian 2 — User dan Group

### Jenis User di Linux

Linux mengenal 3 jenis user:

```
1. root (UID = 0)
   → Superuser, bisa lakukan APA SAJA
   → Seperti "Tuhan" di sistem Linux
   → Sangat berbahaya jika diakses attacker

2. System users (UID 1–999)
   → User untuk menjalankan service/daemon
   → Contoh: www-data (nginx), mysql, sshd
   → TIDAK boleh bisa login ke terminal

3. Regular users (UID 1000+)
   → User manusia biasa
   → Contoh: alice (UID 1000), bob (UID 1001)
```

### File /etc/passwd — Daftar Semua User

```bash
cat /etc/passwd
```

Format setiap baris:
```
username:x:UID:GID:keterangan:home_dir:shell
```

Contoh nyata:
```
root:x:0:0:root:/root:/bin/bash       ← root, UID 0, bisa login
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin  ← system user, TIDAK bisa login
alice:x:1000:1000:Alice:/home/alice:/bin/bash    ← user biasa, bisa login
```

> **Cara baca:**
> - Kolom `x` di posisi 2 = password tersimpan di `/etc/shadow`
> - Shell `/usr/sbin/nologin` atau `/bin/false` = user tidak bisa login (aman!)
> - Shell `/bin/bash` = user bisa login ke terminal

### File /etc/shadow — Password User (Terenkripsi)

```bash
sudo cat /etc/shadow    # Hanya root yang bisa baca!
```

Format:
```
username:$hash$:tanggal_ganti:min:max:warn:inactive:expire:
```

Contoh:
```
alice:$6$salt$hashedpassword...:19500:0:90:7:::
root:!:19500:0:99999:7:::    ← Tanda ! = akun terkunci / tidak ada password
```

> **Penting:** Tanda `!` atau `*` di field password = akun tidak bisa login. Tanda `!!` = password belum pernah di-set.

### Perintah Manajemen User

```bash
# Tambah user baru
sudo adduser alice                  # Cara interaktif (rekomendasi)
sudo useradd -m -s /bin/bash alice  # Cara manual

# Hapus user
sudo deluser alice
sudo userdel -r alice               # -r = hapus folder home juga

# Ganti password user
sudo passwd alice

# Kunci akun (tidak bisa login tapi akun tidak dihapus)
sudo usermod -L alice               # L = Lock

# Buka kunci akun
sudo usermod -U alice               # U = Unlock

# Set shell (ganti ke nologin agar tidak bisa login)
sudo usermod -s /usr/sbin/nologin alice

# Tambahkan user ke group
sudo usermod -aG sudo alice         # Tambah alice ke grup sudo
sudo usermod -aG www-data alice     # Tambah alice ke grup www-data

# Cek info lengkap user
id alice                            # UID, GID, dan semua grup
groups alice                        # Daftar grup alice
```

---

## Bagian 3 — Sistem Permission (Hak Akses File)

> **Ini SANGAT PENTING. Salah permission = celah keamanan.**

### Cara Baca Permission

```bash
ls -la /etc/passwd
```

Output:
```
-rw-r--r-- 1 root root 2156 May 24 10:00 /etc/passwd
│└─┘└──┘└──┘  │    │
│ │  │  └──── Others (semua orang selain owner & group)
│ │  └─────── Group
│ └────────── Owner (pemilik)
└──────────── Tipe file
```

### Arti Karakter Permission

| Karakter | Arti | Nilai |
|----------|------|-------|
| `r` | read (baca) | 4 |
| `w` | write (tulis/edit) | 2 |
| `x` | execute (jalankan) | 1 |
| `-` | tidak ada hak | 0 |

### Contoh Permission Angka

```
chmod 755 file  →  rwx r-x r-x
chmod 644 file  →  rw- r-- r--
chmod 600 file  →  rw- --- ---
chmod 700 file  →  rwx --- ---
chmod 777 file  →  rwx rwx rwx  ← BAHAYA! Semua orang bisa apa saja!
chmod 440 file  →  r-- r-- ---
```

**Cara hitung:**
```
r=4, w=2, x=1 → tambahkan nilainya

rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

### Tabel Permission Penting untuk Hardening

| File | Permission Benar | Angka | Kenapa? |
|------|-----------------|-------|---------|
| `/etc/passwd` | `rw-r--r--` | 644 | Semua bisa baca, hanya root yang bisa edit |
| `/etc/shadow` | `rw-r-----` | 640 | Hanya root dan grup shadow yang bisa baca |
| `/etc/sudoers` | `r--r-----` | 440 | Tidak ada yang bisa edit kecuali via visudo |
| `~/.ssh/` (folder) | `rwx------` | 700 | Hanya pemilik yang bisa akses |
| `~/.ssh/authorized_keys` | `rw-------` | 600 | Hanya pemilik yang bisa baca/tulis |
| `~/.ssh/id_rsa` (private key) | `rw-------` | 600 | Hanya pemilik yang bisa baca |
| `/tmp` | `rwxrwxrwt` | 1777 | Semua bisa tulis, tapi hanya pemilik bisa hapus (sticky bit) |

### Mengubah Permission dan Kepemilikan

```bash
# Ubah permission
chmod 644 file.txt          # Cara angka (paling umum)
chmod u+x script.sh         # Tambah execute untuk owner
chmod o-w file.txt          # Hapus write untuk others
chmod a-x file.txt          # Hapus execute untuk semua (a=all)

# Ubah kepemilikan
chown root:root file.txt    # Ganti owner dan group ke root
chown alice:alice file.txt  # Ganti ke alice
chown -R root /etc/ssh/     # -R = rekursif (semua isi folder)

# Lihat permission
ls -la file.txt             # Detail satu file
ls -la /direktori/          # Semua isi direktori
```

### Bit Khusus: SUID, SGID, Sticky

```bash
# SUID (Set User ID) — file berjalan dengan hak pemiliknya
chmod u+s file    # Aktifkan SUID
chmod u-s file    # Nonaktifkan SUID
# Tanda: 's' di posisi execute owner → -rwSr-xr-x atau -rwsr-xr-x

# SGID (Set Group ID) — file berjalan dengan hak group pemiliknya
chmod g+s file    # Aktifkan SGID
chmod g-s file    # Nonaktifkan SGID

# Sticky Bit — di direktori: hanya pemilik yang bisa hapus file miliknya
chmod +t /tmp     # Aktifkan sticky bit
# Tanda: 't' di posisi execute others → drwxrwxrwt

# Cari semua file SUID di sistem (command yang wajib hafal!)
find / -perm -4000 -type f 2>/dev/null
```

---

## Bagian 4 — Proses dan Service

### Konsep Proses

Setiap program yang berjalan = satu proses. Setiap proses punya:
- **PID** (Process ID) — nomor unik
- **UID** — siapa yang menjalankannya
- **Status** — running, sleeping, zombie

```bash
# Lihat semua proses yang berjalan
ps aux

# Lihat proses secara interaktif (real-time)
top
htop    # Versi lebih canggih (install dulu: sudo apt install htop)

# Lihat proses yang mendengarkan di port tertentu
sudo ss -tulnp
sudo lsof -i :80    # Siapa yang pakai port 80?

# Matikan proses
kill PID            # Kirim sinyal terminate
kill -9 PID         # Force kill (tidak bisa diabaikan)
pkill nama_program  # Kill berdasarkan nama
```

### Konsep Service (Daemon)

Service = program yang berjalan terus di background melayani permintaan.

```
SSH service (sshd)  → menunggu koneksi SSH di port 22
Web service (nginx) → menunggu request HTTP di port 80
MySQL (mysqld)      → menunggu query database di port 3306
```

### Perintah systemctl (Kelola Service)

```bash
# Cek status service
sudo systemctl status ssh        # Apakah SSH berjalan?
sudo systemctl status nginx      # Apakah nginx berjalan?

# Mulai / hentikan service
sudo systemctl start ssh         # Jalankan SSH sekarang
sudo systemctl stop ssh          # Hentikan SSH sekarang
sudo systemctl restart ssh       # Restart SSH

# Aktifkan / nonaktifkan saat boot
sudo systemctl enable ssh        # Otomatis jalan saat boot
sudo systemctl disable ssh       # Tidak jalan saat boot

# PENTING: Stop + Disable sekaligus (untuk matikan service berbahaya)
sudo systemctl stop telnetd && sudo systemctl disable telnetd

# Lihat semua service yang sedang berjalan
sudo systemctl list-units --type=service --state=running

# Lihat semua service yang auto-start saat boot
sudo systemctl list-unit-files --type=service | grep enabled
```

---

## Bagian 5 — Cara Membaca Output `ss -tulnp`

Perintah `ss -tulnp` adalah salah satu perintah terpenting saat lomba. Kamu HARUS bisa membaca outputnya.

```bash
sudo ss -tulnp
```

Contoh output:
```
Netid  State    Recv-Q Send-Q  Local Address:Port   Peer Address:Port   Process
tcp    LISTEN   0      128     0.0.0.0:22            0.0.0.0:*           users:(("sshd",pid=1234))
tcp    LISTEN   0      80      127.0.0.1:3306        0.0.0.0:*           users:(("mysqld",pid=5678))
tcp    LISTEN   0      128     0.0.0.0:21            0.0.0.0:*           users:(("vsftpd",pid=9012))
udp    UNCONN   0      0       0.0.0.0:161           0.0.0.0:*           users:(("snmpd",pid=3456))
```

**Cara baca kolom `Local Address:Port`:**

| Yang kamu lihat | Artinya | Aman? |
|----------------|---------|-------|
| `127.0.0.1:3306` | Hanya bisa diakses dari server itu sendiri | ✅ AMAN |
| `0.0.0.0:22` | Bisa diakses dari semua IP di jaringan | ⚠️ Perlu dikontrol |
| `0.0.0.0:21` | FTP terbuka ke semua IP | ❌ BERBAHAYA, matikan! |
| `:::80` | IPv6, semua IP, port 80 | ⚠️ Perlu dikontrol |

**Cara baca dari output di atas:**
```
ssh  (port 22)   → terbuka ke 0.0.0.0 = semua IP → perlu dibatasi via firewall
mysql (port 3306) → hanya localhost (127.0.0.1) → AMAN ✅
ftp  (port 21)   → terbuka ke 0.0.0.0 = MATIKAN SEGERA! ❌
snmp (port 161)  → UDP, terbuka ke semua → MATIKAN SEGERA! ❌
```

---

## Bagian 6 — Text Editor di Terminal

Kamu harus bisa edit file konfigurasi di terminal. Ada dua pilihan:

### nano — Mudah untuk Pemula

```bash
sudo nano /etc/ssh/sshd_config
```

Shortcut penting:
```
CTRL+O  → Simpan file
CTRL+X  → Keluar dari nano
CTRL+W  → Cari teks
CTRL+K  → Cut (potong) baris
CTRL+U  → Paste baris
CTRL+G  → Bantuan
```

### vim — Lebih Powerful (Opsional)

```bash
sudo vim /etc/ssh/sshd_config
```

Mode vim:
```
Normal mode → default saat buka vim
Insert mode → tekan 'i' untuk masuk, tekan 'ESC' untuk keluar
Command mode → tekan ':' dari normal mode
```

Command penting:
```
:w    → Simpan
:q    → Keluar
:wq   → Simpan dan keluar
:q!   → Keluar tanpa simpan (paksa)
/kata → Cari "kata" dalam file (tekan n untuk next)
```

> **Saran:** Untuk lomba, gunakan **nano** saja dulu. Lebih sederhana dan tidak membingungkan.

---

## Bagian 7 — Perintah Dasar yang Wajib Dikuasai

```bash
# ===== NAVIGASI =====
pwd             # Print Working Directory — kamu sekarang di mana?
ls -la          # Lihat isi direktori + permission
cd /etc         # Pindah ke direktori /etc
cd ~            # Pindah ke home directory
cd ..           # Mundur satu level

# ===== FILE OPERATIONS =====
cat file.txt    # Tampilkan isi file
less file.txt   # Tampilkan isi file (bisa scroll, tekan q untuk keluar)
head -10 file   # Tampilkan 10 baris pertama
tail -10 file   # Tampilkan 10 baris terakhir
tail -f file    # Tampilkan baris baru secara real-time (live)

# ===== PENCARIAN =====
grep "kata" file.txt           # Cari "kata" dalam file
grep -r "kata" /direktori/     # Cari rekursif di semua file
grep -v "kata" file.txt        # Tampilkan baris yang TIDAK mengandung "kata"
find / -name "file.txt"        # Cari file bernama "file.txt"
find / -perm -4000 -type f     # Cari file dengan permission tertentu

# ===== PRIVILEGE =====
sudo command    # Jalankan command sebagai root
su -            # Switch ke user root (perlu password root)
su username     # Switch ke user lain
whoami          # Tampilkan siapa kamu sekarang

# ===== NETWORK =====
ip addr         # Lihat IP address
ip route        # Lihat routing table
sudo ss -tulnp  # Lihat semua port yang terbuka
ping 8.8.8.8    # Test konektivitas jaringan

# ===== PROSES =====
ps aux          # Lihat semua proses
kill PID        # Matikan proses berdasarkan PID
killall nama    # Matikan proses berdasarkan nama

# ===== PACKAGE MANAGEMENT =====
sudo apt update                 # Update daftar package
sudo apt upgrade -y             # Update semua package
sudo apt install nama -y        # Install package
sudo apt remove nama -y         # Hapus package
sudo apt purge nama -y          # Hapus package + konfigurasinya
dpkg -l | grep nama             # Cek apakah package terinstall
```

---

## Bagian 8 — Konsep Penting: stdin, stdout, stderr, dan Pipe

Ini penting agar kamu paham command-command yang ada di modul berikutnya.

```bash
# stdout → output normal (ke layar)
ls -la

# stderr → output error
ls /tidak/ada/direktori 2>/dev/null   # 2>/dev/null = buang error, jangan tampilkan

# Pipe | → output dari kiri dijadikan input ke kanan
grep "Failed" /var/log/auth.log | wc -l    # Hitung berapa baris yang mengandung "Failed"

# Redirect output ke file
ls -la > output.txt        # Simpan ke file (timpa jika sudah ada)
ls -la >> output.txt       # Tambahkan ke file (tidak menimpa)

# Kombinasi yang sering dipakai:
find / -perm -4000 -type f 2>/dev/null | sort
# Artinya: cari file SUID, buang pesan error, urutkan hasilnya
```

---

## ✅ Checklist Konsep Dasar (Pastikan Kamu Paham Ini Semua!)

- [ ] Bisa navigasi direktori dengan `cd`, `ls`, `pwd`
- [ ] Paham struktur `/etc`, `/var/log`, `/home`
- [ ] Bisa baca file `/etc/passwd` dan memahami setiap kolomnya
- [ ] Paham permission rwx dan bisa hitung angkanya (chmod)
- [ ] Bisa ganti permission dengan `chmod` dan kepemilikan dengan `chown`
- [ ] Paham apa itu SUID dan kenapa berbahaya
- [ ] Bisa kelola service dengan `systemctl`
- [ ] Bisa baca output `ss -tulnp`
- [ ] Bisa edit file dengan `nano`
- [ ] Paham penggunaan `grep`, `find`, dan pipe `|`

---

## 🔗 Navigasi

← `00_INDEX_Linux_Hardening` (Kembali ke Index)
→ `01_Network_Service_Security`
