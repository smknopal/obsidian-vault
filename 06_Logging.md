# 06 — Logging
#lks #cyber-security #linux #hardening #logging #auditd #syslog #monitoring

> **Peran:** Defender | **OS:** Debian/Ubuntu-based | **Konteks:** LKS Cyber Security 2026
> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Logging

---

## 🎯 Tujuan

- Memastikan semua aktivitas sistem tercatat dengan baik
- Mampu membaca dan menganalisis log untuk mendeteksi serangan
- Melindungi log agar tidak bisa dihapus/dimodifikasi attacker

---

> 🧠 **Kenapa logging sangat penting?**
> "Tanpa log → kamu BUTA. Attacker bisa masuk, buat backdoor, curi data, dan keluar — tanpa kamu tahu apa yang terjadi."
>
> Log yang baik menjawab pertanyaan investigasi:
> - **SIAPA** yang login?
> - **DARI MANA** (IP address)?
> - **KAPAN** kejadiannya?
> - **APA** yang dilakukan?
> - **APAKAH** ada yang mencurigakan?

---

## Peta Sistem Logging Linux

```
Sumber Log                    Dikumpulkan oleh          Disimpan di
─────────────────             ─────────────────         ──────────────────────────
Login/SSH/PAM    ──────────→  rsyslog / journald  →     /var/log/auth.log
Kernel           ──────────→  rsyslog / journald  →     /var/log/kern.log
Sistem umum      ──────────→  rsyslog / journald  →     /var/log/syslog
Audit syscalls   ──────────→  auditd              →     /var/log/audit/audit.log
Firewall UFW     ──────────→  rsyslog             →     /var/log/ufw.log
Fail2ban         ──────────→  rsyslog             →     /var/log/fail2ban.log
Cron jobs        ──────────→  rsyslog             →     /var/log/cron.log
```

---

## 1. File Log Penting di Linux

| File | Isi | Prioritas |
|------|-----|-----------|
| `/var/log/auth.log` | Login, sudo, SSH, PAM, pam_faillock | 🔴 PALING PENTING |
| `/var/log/audit/audit.log` | Log auditd (paling detail) | 🔴 PALING PENTING |
| `/var/log/syslog` | Pesan sistem umum | 🟡 Penting |
| `/var/log/kern.log` | Pesan kernel | 🟡 Penting |
| `/var/log/ufw.log` | Aktivitas firewall UFW | 🟡 Penting |
| `/var/log/fail2ban.log` | IP yang di-ban Fail2ban | 🟡 Penting |
| `/var/log/sudo.log` | Perintah sudo | 🔴 Penting (jika dikonfigurasi) |
| `/var/log/dpkg.log` | Instalasi/uninstall package | 🟢 Berguna |
| `/var/log/cron.log` | Eksekusi cron job | 🟢 Berguna |

---

## 2. Membaca Log — Cara Baca Format yang Benar

Ini penting! Kamu harus bisa membaca log secara manual saat lomba.

### Format Log auth.log

```
May 24 10:15:32 server sshd[1234]: Accepted publickey for alice from 192.168.1.5 port 52341 ssh2
│              │       │    │       │                     │         │              │
│              │       │    │       │                     │         │              └── Port client
│              │       │    │       │                     │         └── IP address client
│              │       │    │       │                     └── Username yang login
│              │       │    │       └── Pesan (Accepted = berhasil)
│              │       │    └── PID proses sshd
│              │       └── Nama service yang menulis log
│              └── Hostname server
└── Timestamp (Bulan Tanggal Jam:Menit:Detik)
```

### Contoh Log auth.log yang Nyata

```
# Login SSH berhasil dengan key
May 24 10:15:32 server sshd[1234]: Accepted publickey for alice from 192.168.1.5 port 52341

# Login SSH gagal (salah password)
May 24 10:16:01 server sshd[1235]: Failed password for alice from 192.168.1.100 port 43210

# Percobaan login sebagai root (BAHAYA!)
May 24 10:16:05 server sshd[1236]: Invalid user root from 10.0.0.5 port 22341

# Akun dikunci oleh pam_faillock
May 24 10:16:10 server unix_chkpwd[1237]: password check failed for user (alice)
May 24 10:16:10 server pam_faillock[1237]: Blocking user alice for 900 seconds

# Penggunaan sudo berhasil
May 24 10:20:00 server sudo[2345]: alice : TTY=pts/0 ; PWD=/home/alice ; USER=root ; COMMAND=/usr/bin/apt upgrade

# Penggunaan sudo ditolak
May 24 10:21:00 server sudo[2346]: alice : command not allowed ; TTY=pts/0 ; USER=root ; COMMAND=/bin/bash
```

---

## 3. Perintah Baca Log (Wajib Dikuasai!)

### Real-time Monitoring

```bash
# Monitor auth.log secara live (seperti TV streaming log)
sudo tail -f /var/log/auth.log

# Monitor beberapa file sekaligus
sudo tail -f /var/log/auth.log /var/log/fail2ban.log

# Lihat 100 baris terakhir
sudo tail -n 100 /var/log/auth.log
```

### Filter dengan grep (Yang Paling Sering Dipakai!)

```bash
# ===== LOGIN EVENTS =====
# Semua login GAGAL (brute force?)
grep "Failed password" /var/log/auth.log

# Semua login BERHASIL
grep "Accepted" /var/log/auth.log

# Login berhasil via public key
grep "Accepted publickey" /var/log/auth.log

# ===== SUDO EVENTS =====
# Semua perintah sudo
grep "sudo" /var/log/auth.log

# Percobaan sudo yang ditolak (tidak punya hak)
grep "sudo.*NOT allowed\|sudo.*not allowed" /var/log/auth.log

# ===== SSH EVENTS =====
# Semua aktivitas sshd
grep "sshd" /var/log/auth.log

# User yang tidak ada di sistem mencoba login
grep "Invalid user" /var/log/auth.log

# ===== AKUN =====
# Akun yang dikunci pam_faillock
grep "faillock\|Blocking user" /var/log/auth.log

# User baru ditambahkan
grep "useradd\|new user" /var/log/auth.log

# ===== FILTER WAKTU =====
# Event hari ini (tanggal 24)
grep "May 24" /var/log/auth.log

# Event jam 10
grep "May 24 10:" /var/log/auth.log

# ===== KOMBINASI (sangat berguna!) =====
# Login gagal dari IP tertentu
grep "Failed password" /var/log/auth.log | grep "192.168.1.100"

# Event sejak 1 jam lalu (pakai journalctl)
sudo journalctl --since "1 hour ago" | grep "sshd"
```

### Analisis Brute Force — Temukan IP Penyerang

```bash
# Hitung percobaan gagal per IP — siapa yang paling agresif?
grep "Failed password" /var/log/auth.log \
  | awk '{print $(NF-3)}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -20

# Contoh output:
#   456 192.168.1.200   ← IP ini paling banyak coba (KEMUNGKINAN ATTACKER!)
#    23 10.0.0.5
#     3 172.16.0.1

# Cari username yang sering jadi target
grep "Failed password" /var/log/auth.log \
  | awk '{print $9}' \
  | sort \
  | uniq -c \
  | sort -rn

# Cari jam berapa serangan paling banyak
grep "Failed password" /var/log/auth.log \
  | awk '{print $3}' \
  | cut -d: -f1 \
  | sort \
  | uniq -c \
  | sort -rn
```

---

## 4. rsyslog — Sistem Logging Standar

**rsyslog** adalah daemon yang mengumpulkan dan mendistribusikan log.

```bash
# Cek status
sudo systemctl status rsyslog

# Jika tidak aktif:
sudo systemctl enable rsyslog
sudo systemctl start rsyslog
```

### Konfigurasi rsyslog

```bash
sudo nano /etc/rsyslog.conf
```

Pastikan baris-baris ini ada dan tidak dikomen:

```bash
# Log semua auth messages (login, sudo, SSH)
auth,authpriv.*                 /var/log/auth.log

# Log semua pesan sistem (kecuali auth)
*.*;auth,authpriv.none          -/var/log/syslog

# Log kernel
kern.*                          /var/log/kern.log

# Log cron
cron.*                          /var/log/cron.log

# Log error dan yang lebih serius ke file tersendiri
*.err                           /var/log/error.log
```

### Kirim Log ke Server Remote (Log Centralization)

Ini penting: attacker yang masuk ke server bisa hapus log lokal. Solusinya kirim log ke server lain!

```bash
sudo nano /etc/rsyslog.conf
```

Tambahkan di akhir file:
```bash
# @@  = kirim via TCP (reliable, pastikan terima)
# @   = kirim via UDP (lebih cepat tapi bisa hilang)
*.* @@192.168.1.200:514
```

```bash
sudo systemctl restart rsyslog
```

---

## 5. journald — Sistem Logging Modern

**journald** adalah sistem logging dari systemd. Format binary, lebih terstruktur.

### Perintah journalctl yang Wajib Dikuasai

```bash
# Lihat semua log (tekan q untuk keluar)
sudo journalctl

# Real-time log (seperti tail -f)
sudo journalctl -f

# ===== FILTER BERDASARKAN SERVICE =====
sudo journalctl -u sshd             # Log SSH
sudo journalctl -u fail2ban         # Log Fail2ban
sudo journalctl -u auditd           # Log auditd
sudo journalctl -u sshd -f          # Log SSH real-time

# ===== FILTER BERDASARKAN WAKTU =====
sudo journalctl --since "1 hour ago"                    # 1 jam terakhir
sudo journalctl --since "2026-05-24 08:00:00"           # Dari waktu tertentu
sudo journalctl --since "2026-05-24 08:00" --until "2026-05-24 10:00"  # Rentang waktu
sudo journalctl --since today                           # Hari ini

# ===== FILTER BERDASARKAN LEVEL KEPARAHAN =====
sudo journalctl -p err              # Hanya error
sudo journalctl -p warning          # Warning dan yang lebih serius
# Level: 0=emerg, 1=alert, 2=crit, 3=err, 4=warning, 5=notice, 6=info, 7=debug

# ===== TAMPILKAN N BARIS =====
sudo journalctl -n 50               # 50 baris terakhir

# ===== LOG DARI BOOT SEBELUMNYA =====
sudo journalctl -b                  # Boot saat ini
sudo journalctl -b -1               # Boot sebelumnya
sudo journalctl --list-boots        # Daftar semua sesi boot
```

### Konfigurasi journald

```bash
sudo nano /etc/systemd/journald.conf
```

```ini
[Journal]
# Simpan log secara persistent (tidak hilang saat reboot)
Storage=persistent

# Batas ukuran log
SystemMaxUse=1G

# Berapa lama log disimpan
MaxRetentionSec=30day

# Kompres log lama
Compress=yes

# Forward ke rsyslog (agar log juga masuk ke /var/log/)
ForwardToSyslog=yes
```

```bash
sudo systemctl restart systemd-journald
```

---

## 6. auditd — Audit Framework Tingkat Lanjut

**auditd** memberikan logging paling detail. Bisa catat:
- File mana yang dibuka/dimodifikasi/dihapus
- Perintah apa yang dijalankan oleh siapa
- System call apa yang dilakukan program

### Install dan Aktifkan

```bash
sudo apt install auditd audispd-plugins -y
sudo systemctl enable auditd
sudo systemctl start auditd
```

### Konfigurasi Aturan Audit

```bash
sudo nano /etc/audit/rules.d/hardening.rules
```

Isi file ini:

```bash
## =======================================================
## ATURAN AUDIT — Linux Hardening LKS 2026
## =======================================================

## ===== HAPUS SEMUA ATURAN LAMA =====
-D

## ===== UKURAN BUFFER =====
-b 8192

## ===== MONITOR PERUBAHAN FILE IDENTITAS =====
## Catat jika ada yang mengedit /etc/passwd, /etc/shadow, dll.
-w /etc/passwd -p wa -k identity_change
-w /etc/shadow -p wa -k identity_change
-w /etc/group -p wa -k identity_change
-w /etc/gshadow -p wa -k identity_change

## Penjelasan flag:
## -w = watch (pantau file/direktori ini)
## -p = permission: r=read, w=write, a=append, x=execute
## -k = key (label untuk pencarian nanti dengan ausearch)

## ===== MONITOR PERUBAHAN KONFIGURASI PAM =====
-w /etc/pam.d/ -p wa -k pam_change
-w /etc/security/pwquality.conf -p wa -k pam_change
-w /etc/security/faillock.conf -p wa -k pam_change

## ===== MONITOR PERUBAHAN SSH =====
-w /etc/ssh/sshd_config -p wa -k sshd_change

## ===== MONITOR PENGGUNAAN SUDO =====
-w /etc/sudoers -p wa -k sudoers_change
-w /etc/sudoers.d/ -p wa -k sudoers_change
-w /usr/bin/sudo -p x -k sudo_usage

## ===== MONITOR MANAJEMEN USER =====
-w /usr/sbin/useradd -p x -k user_management
-w /usr/sbin/userdel -p x -k user_management
-w /usr/sbin/usermod -p x -k user_management
-w /usr/sbin/groupadd -p x -k user_management
-w /usr/sbin/groupdel -p x -k user_management

## ===== MONITOR CRON =====
-w /etc/cron.d/ -p wa -k cron_change
-w /etc/crontab -p wa -k cron_change
-w /var/spool/cron/ -p wa -k cron_change

## ===== MONITOR PERUBAHAN JARINGAN =====
-w /etc/hosts -p wa -k hosts_change
-w /etc/network/ -p wa -k network_change
-w /etc/sysctl.conf -p wa -k sysctl_change

## ===== MONITOR AKSES FILE SENSITIF =====
-w /root/.ssh/ -p rwa -k ssh_key_access
-a always,exit -F path=/etc/sudoers -F perm=r -k sudoers_read

## ===== MONITOR SEMUA EKSEKUSI PERINTAH =====
-a always,exit -F arch=b64 -S execve -k command_exec
-a always,exit -F arch=b32 -S execve -k command_exec

## ===== MONITOR PERUBAHAN PERMISSION =====
-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -k permission_change
-a always,exit -F arch=b32 -S chmod,fchmod,fchmodat -k permission_change
-a always,exit -F arch=b64 -S chown,fchown,fchownat -k ownership_change
-a always,exit -F arch=b32 -S chown,fchown,fchownat -k ownership_change

## ===== IMMUTABLE MODE =====
## Aktifkan ini TERAKHIR — setelah semua aturan selesai!
## Setelah ini aktif, aturan TIDAK BISA diubah tanpa reboot
-e 2
```

**Terapkan aturan:**
```bash
# Load aturan
sudo augenrules --load
sudo systemctl restart auditd

# Verifikasi aturan aktif
sudo auditctl -l
```

### Membaca Log auditd

```bash
# ===== ausearch — Cari Event Berdasarkan Key =====

# Cari event perubahan file identitas (modifikasi passwd/shadow/group)
sudo ausearch -k identity_change

# Cari event penggunaan sudo
sudo ausearch -k sudo_usage

# Cari event dari hari ini saja
sudo ausearch -k identity_change --start today

# Cari event dari user tertentu (ganti 1000 dengan UID-nya)
sudo ausearch -ua 1000 -i    # -i = tampilkan nama user, bukan angka UID

# ===== aureport — Laporan Ringkasan =====

# Laporan ringkasan semua kategori
sudo aureport --summary

# Laporan autentikasi (login berhasil/gagal)
sudo aureport --auth

# Laporan semua kegagalan
sudo aureport --failed

# Laporan eksekusi perintah
sudo aureport --executable

# Laporan perubahan file
sudo aureport --file
```

---

## 7. Proteksi Log dari Modifikasi

Attacker yang berhasil masuk akan **mencoba hapus log** untuk hapus jejak. Cegah ini!

### A. Permission yang Benar

```bash
# Set permission file log
sudo chown root:adm /var/log/auth.log
sudo chmod 640 /var/log/auth.log

sudo chown root:adm /var/log/syslog
sudo chmod 640 /var/log/syslog

sudo chown root:root /var/log/audit/audit.log
sudo chmod 600 /var/log/audit/audit.log
```

### B. chattr — Buat Log Append-Only

Dengan atribut ini, bahkan root pun tidak bisa hapus file!

```bash
# Pasang atribut append-only (bisa ditambah, TIDAK BISA dihapus)
sudo chattr +a /var/log/auth.log
sudo chattr +a /var/log/syslog
sudo chattr +a /var/log/audit/audit.log

# Verifikasi atribut terpasang
lsattr /var/log/auth.log
# Output: -----a--------e--- /var/log/auth.log
# Huruf 'a' = append-only aktif ✅

# Jika perlu maintenance, hapus atribut dulu:
sudo chattr -a /var/log/auth.log
```

### C. Logrotate — Kelola Ukuran Log

Log yang tidak dirotasi bisa penuh dan sistem crash!

```bash
sudo nano /etc/logrotate.d/security-logs
```

```
/var/log/auth.log
/var/log/sudo.log
/var/log/fail2ban.log
/var/log/ufw.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root adm
    postrotate
        /usr/lib/rsyslog/rsyslog-rotate
    endscript
}
```

```bash
# Test konfigurasi (dry run, tidak benar-benar dieksekusi)
sudo logrotate --debug /etc/logrotate.d/security-logs
```

---

## 8. Monitoring Aktif — Deteksi Anomali

### Cek Siapa yang Login Sekarang

```bash
w               # User yang login + apa yang sedang dilakukan
who             # Daftar user yang login
last | head -20 # Riwayat login + logout
lastb | head -10 # Riwayat login GAGAL
lastlog         # Login terakhir setiap user
```

### Deteksi Perubahan Mencurigakan

```bash
# File yang dimodifikasi dalam 24 jam terakhir di direktori sistem
find /etc /usr /bin /sbin -mtime -1 -type f 2>/dev/null

# Verifikasi integritas package (cek file yang berubah dari versi aslinya)
sudo dpkg --verify 2>/dev/null
# Jika ada output → file dari package sudah dimodifikasi! BAHAYA!

# Cari file baru di /tmp (sering dipakai attacker)
find /tmp /var/tmp -type f 2>/dev/null

# Cari proses yang binary-nya sudah dihapus (kemungkinan malware!)
ls -la /proc/*/exe 2>/dev/null | grep deleted

# Koneksi jaringan yang mencurigakan
sudo ss -tulnp
sudo lsof -i -P -n | grep LISTEN
```

### Deteksi Privilege Escalation

```bash
# Cek binary SUID baru (dibuat setelah /etc/passwd)
find / -perm -4000 -type f -newer /etc/passwd 2>/dev/null

# Cek user yang baru ditambahkan ke grup sudo
grep "sudo\|wheel\|admin" /etc/group

# Cek UID 0 (siapa yang punya hak root?)
awk -F: '($3 == 0) {print $1}' /etc/passwd
# Harus HANYA "root" yang muncul!
```

---

## Ringkasan Perintah Penting (Hafalkan!)

```bash
# ===== CEK LOG SEKARANG =====
sudo tail -f /var/log/auth.log                          # Monitor real-time
grep "Failed password" /var/log/auth.log                # Login gagal
grep "Accepted" /var/log/auth.log                       # Login berhasil
grep "Invalid user" /var/log/auth.log                   # User tidak ada coba login
sudo journalctl -u sshd -f                              # Log SSH real-time
sudo journalctl --since "1 hour ago"                    # Log 1 jam terakhir

# ===== ANALISIS =====
w                                                       # Siapa yang login sekarang
last | head -20                                         # Riwayat login
lastb | head -10                                        # Login gagal
sudo aureport --summary                                 # Ringkasan audit
sudo ausearch -k identity_change --start today          # Perubahan file user hari ini

# ===== STATUS SERVICE =====
sudo systemctl status auditd
sudo systemctl status rsyslog
sudo fail2ban-client status sshd
```

---

## ✅ Checklist Logging (Untuk Lomba!)

### rsyslog
- [ ] rsyslog aktif: `sudo systemctl status rsyslog`
- [ ] `/var/log/auth.log` ada dan berisi data: `sudo tail -5 /var/log/auth.log`
- [ ] Konfigurasi `/etc/rsyslog.conf` sudah benar
- [ ] Log remote dikonfigurasi jika diminta soal

### journald
- [ ] `Storage=persistent` di `/etc/systemd/journald.conf`
- [ ] `ForwardToSyslog=yes` dikonfigurasi
- [ ] Bisa baca log dengan `journalctl -u sshd`

### auditd
- [ ] auditd terinstall: `dpkg -l | grep auditd`
- [ ] auditd aktif: `sudo systemctl status auditd`
- [ ] File aturan di `/etc/audit/rules.d/` sudah dikonfigurasi
- [ ] `sudo auditctl -l` menampilkan aturan aktif
- [ ] Monitor `/etc/passwd`, `/etc/shadow`, sudo, cron
- [ ] Bisa gunakan `ausearch -k identity_change`
- [ ] Bisa gunakan `aureport --summary`

### Proteksi Log
- [ ] Permission log benar (640, root:adm)
- [ ] `chattr +a` dipasang pada file log kritis
- [ ] Logrotate dikonfigurasi

### Monitoring
- [ ] Bisa baca log dengan grep dan tail
- [ ] Bisa analisis IP brute force dari log
- [ ] Bisa deteksi perubahan file mencurigakan
- [ ] Bisa cek siapa yang login dengan `w`, `last`, `lastb`

---

## 🔗 Navigasi

← `05_Common_Linux_Misconfigurations`
→ `00_INDEX_Linux_Hardening` (Kembali ke Index)
