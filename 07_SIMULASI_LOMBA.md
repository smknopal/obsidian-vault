# 07 — Simulasi Lomba: Hardening Server dari Nol
#lks #cyber-security #simulasi #latihan #lomba

> **⏱️ Target Waktu:** 3 jam (sama persis seperti di lomba!)
> **Tingkat:** Pemula → langsung ke praktik nyata
> **Tujuan:** Melatih otot jarimu agar command keluar otomatis tanpa berpikir lama

---

> 🧠 **Cara Pakai File Ini:**
> 1. Siapkan VM Linux (Ubuntu/Debian)
> 2. Set timer 3 jam
> 3. Ikuti step by step — **JANGAN LIHAT CATATAN DULU**
> 4. Setelah selesai, cek dengan checklist di bawah
> 5. Ulangi sampai bisa selesai dalam < 2.5 jam

---

## 📋 SKENARIO LOMBA

Kamu dapat sebuah server Ubuntu yang baru dipasang dengan kondisi tidak aman. Tugasmu adalah hardening server ini sesuai security best practice.

**Kondisi awal server (yang sengaja dibuat "bobrok"):**
- Service FTP, Telnet, dan SNMP berjalan
- SSH masih bisa login dengan password
- Root bisa login langsung via SSH
- Tidak ada password complexity policy
- Tidak ada account lockout
- Redis terbuka ke semua IP
- Tidak ada firewall
- Tidak ada logging yang proper
- Ada file SUID yang berbahaya

---

## ✅ FASE 1: RECON (Target: 10 menit)

> **Prinsip: Kenali dulu musuhmu sebelum mulai perang.**

```bash
# ===== LANGKAH 1: Cek semua port yang terbuka =====
sudo ss -tulnp
# CATAT: port apa saja yang terbuka? Mana yang berbahaya?

# ===== LANGKAH 2: Cek semua service yang berjalan =====
sudo systemctl list-units --type=service --state=running
# CATAT: service apa yang aneh/tidak dikenal?

# ===== LANGKAH 3: Cek semua service yang auto-start =====
sudo systemctl list-unit-files --type=service | grep enabled
# CATAT: service apa yang auto-start tapi tidak diperlukan?

# ===== LANGKAH 4: Audit user =====
cat /etc/passwd
# CATAT: user apa yang bisa login (shell bukan nologin)?

# Cek UID 0 selain root
awk -F: '($3 == 0) {print $1}' /etc/passwd
# Harusnya HANYA root! Jika ada lain → BAHAYA!

# Cek user tanpa password
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
# Jika ada output → BERBAHAYA, harus segera diset password atau dikunci

# ===== LANGKAH 5: Cari file SUID =====
find / -perm -4000 -type f 2>/dev/null
# CATAT: file SUID apa yang ada dan mana yang mencurigakan?
```

**Hasil recon — tulis di sini:**
```
Port berbahaya yang ditemukan: ___________________________
Service berbahaya: ______________________________________
User yang mencurigakan: _________________________________
SUID mencurigakan: ______________________________________
```

---

## ✅ FASE 2: MATIKAN SERVICE BERBAHAYA (Target: 10 menit)

```bash
# ===== FTP (port 21) =====
sudo systemctl stop vsftpd
sudo systemctl disable vsftpd
sudo apt purge vsftpd -y
# Verifikasi:
sudo ss -tulnp | grep :21     # Harus KOSONG ✅

# ===== Telnet (port 23) =====
sudo systemctl stop telnetd 2>/dev/null || true
sudo systemctl disable telnetd 2>/dev/null || true
sudo apt purge telnetd telnet -y 2>/dev/null || true
# Verifikasi:
sudo ss -tulnp | grep :23     # Harus KOSONG ✅

# ===== SNMP (port 161) =====
sudo systemctl stop snmpd 2>/dev/null || true
sudo systemctl disable snmpd 2>/dev/null || true
sudo apt purge snmpd -y 2>/dev/null || true
# Verifikasi:
sudo ss -tulnp | grep :161    # Harus KOSONG ✅

# ===== rpcbind (port 111) =====
sudo systemctl stop rpcbind 2>/dev/null || true
sudo systemctl disable rpcbind 2>/dev/null || true

# ===== Verifikasi semua port berbahaya sudah hilang =====
echo "=== Port yang masih terbuka ==="
sudo ss -tulnp
```

---

## ✅ FASE 3: HARDENING SSH (Target: 20 menit)

```bash
# ===== LANGKAH 1: Buat SSH key (jika belum ada) =====
ls ~/.ssh/id_rsa 2>/dev/null || ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa

# ===== LANGKAH 2: Pasang public key ke server =====
# (Jika server lokal, langsung copy ke authorized_keys)
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# ===== LANGKAH 3: Test login dengan key dulu! =====
# Buka terminal baru, coba login:
# ssh -i ~/.ssh/id_rsa user@localhost
# JANGAN lanjut ke langkah 4 jika login dengan key belum berhasil!

# ===== LANGKAH 4: Edit konfigurasi SSH =====
sudo nano /etc/ssh/sshd_config
```

**Isi/ubah bagian ini di sshd_config:**
```ini
Port 2222
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 20
X11Forwarding no
AllowTcpForwarding no
GatewayPorts no
PermitUserEnvironment no
StrictModes yes
UseDNS no
LogLevel VERBOSE
ClientAliveInterval 300
ClientAliveCountMax 2
Banner /etc/ssh/banner.txt
```

```bash
# ===== LANGKAH 5: Buat banner =====
sudo bash -c 'cat > /etc/ssh/banner.txt << "EOF"
============================================================
  SISTEM INI HANYA UNTUK PENGGUNA YANG BERWENANG.
  Semua aktivitas dicatat dan dipantau.
  Akses tanpa izin akan ditindaklanjuti secara hukum.
============================================================
EOF'

# ===== LANGKAH 6: Test syntax SEBELUM restart =====
sudo sshd -t
# Jika tidak ada output = tidak ada error ✅
# Jika ada error = PERBAIKI DULU sebelum restart!

# ===== LANGKAH 7: Restart SSH =====
sudo systemctl restart sshd
sudo systemctl status sshd

# ===== LANGKAH 8: Test login dari terminal baru =====
# ssh -p 2222 -i ~/.ssh/id_rsa user@localhost
```

---

## ✅ FASE 4: PAM — PASSWORD COMPLEXITY (Target: 10 menit)

```bash
# ===== LANGKAH 1: Install modul =====
sudo apt install libpam-pwquality -y

# ===== LANGKAH 2: Konfigurasi pwquality =====
sudo bash -c 'cat > /etc/security/pwquality.conf << "EOF"
minlen = 12
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
minclass = 4
maxrepeat = 3
maxsequence = 3
difok = 5
dictcheck = 1
usercheck = 1
badwords = password admin root linux server
EOF'

# ===== LANGKAH 3: Tambahkan ke PAM =====
# Cek apakah sudah ada
grep "pwquality" /etc/pam.d/common-password

# Jika TIDAK ada, tambahkan baris ini di baris PERTAMA common-password:
# Gunakan: sudo nano /etc/pam.d/common-password
# Tambahkan DI BARIS PALING ATAS:
# password requisite pam_pwquality.so retry=3

# ===== LANGKAH 4: Tambahkan password history =====
# Tambahkan juga baris ini (setelah baris pwquality):
# password required pam_pwhistory.so remember=5 use_authok

# ===== LANGKAH 5: Konfigurasi login.defs =====
sudo sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS\t90/' /etc/login.defs
sudo sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS\t1/' /etc/login.defs
sudo sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE\t7/' /etc/login.defs

# Verifikasi
grep "PASS_" /etc/login.defs

# ===== LANGKAH 6: Terapkan untuk user yang sudah ada =====
for user in $(awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd); do
    sudo chage -M 90 -m 1 -W 7 "$user"
    echo "Set expiry: $user"
done
```

---

## ✅ FASE 5: PAM — ACCOUNT LOCKOUT (Target: 10 menit)

```bash
# ===== LANGKAH 1: Konfigurasi faillock.conf =====
sudo bash -c 'cat > /etc/security/faillock.conf << "EOF"
deny = 3
unlock_time = 900
fail_interval = 900
audit
silent
EOF'

# ===== LANGKAH 2: Tambahkan ke PAM common-auth =====
# Buka file:
sudo nano /etc/pam.d/common-auth

# File harus terlihat seperti ini (edit agar sesuai):
# auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900
# auth [success=1 default=ignore] pam_unix.so nullok
# auth [default=die] pam_faillock.so authfail audit deny=3 unlock_time=900
# auth requisite pam_deny.so
# auth required pam_permit.so

# ===== LANGKAH 3: Tambahkan ke common-account =====
# Cek apakah sudah ada:
grep "faillock" /etc/pam.d/common-account
# Jika tidak ada, tambahkan:
# account required pam_faillock.so

# ===== LANGKAH 4: Verifikasi =====
# Coba login salah 3 kali, kemudian:
sudo faillock
# Harus terlihat catatan percobaan gagal
```

---

## ✅ FASE 6: UFW FIREWALL (Target: 10 menit)

```bash
# ===== Pastikan UFW terinstall =====
sudo apt install ufw -y

# ===== Setup policy default =====
sudo ufw default deny incoming
sudo ufw default allow outgoing

# ===== Izinkan port yang diperlukan (URUTAN INI PENTING!) =====
sudo ufw allow 2222/tcp    # SSH (port baru!)
sudo ufw allow 80/tcp      # HTTP (jika ada web server)
sudo ufw allow 443/tcp     # HTTPS (jika ada web server)

# ===== Blokir port berbahaya secara eksplisit =====
sudo ufw deny 21/tcp       # FTP
sudo ufw deny 23/tcp       # Telnet
sudo ufw deny 3306/tcp     # MySQL dari luar

# ===== Rate limiting SSH (anti brute-force) =====
sudo ufw limit 2222/tcp

# ===== Aktifkan UFW =====
sudo ufw enable

# ===== Verifikasi =====
sudo ufw status verbose
```

---

## ✅ FASE 7: FAIL2BAN (Target: 10 menit)

```bash
# ===== Install =====
sudo apt install fail2ban -y

# ===== Buat konfigurasi =====
sudo bash -c 'cat > /etc/fail2ban/jail.local << "EOF"
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1
findtime = 600
maxretry = 3
bantime = 3600

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF'

# ===== Aktifkan dan jalankan =====
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# ===== Verifikasi =====
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

---

## ✅ FASE 8: FIX SUID BERBAHAYA (Target: 15 menit)

```bash
# ===== Cari semua file SUID =====
find / -perm -4000 -type f 2>/dev/null | sort

# ===== Daftar SUID yang BOLEH (jangan hapus!) =====
# /usr/bin/sudo
# /usr/bin/passwd
# /usr/bin/su
# /usr/bin/newgrp
# /usr/bin/chsh
# /usr/bin/gpasswd
# /bin/ping atau /usr/bin/ping
# /usr/bin/mount
# /usr/bin/pkexec

# ===== Hapus SUID dari binary yang tidak perlu =====
# Periksa satu per satu, jika ada yang BUKAN di daftar di atas → hapus SUID-nya!

# Contoh jika menemukan find, vim, python3, dll:
sudo chmod u-s /usr/bin/find 2>/dev/null && echo "Removed SUID: find"
sudo chmod u-s /usr/bin/vim 2>/dev/null && echo "Removed SUID: vim"
sudo chmod u-s /usr/bin/python3 2>/dev/null && echo "Removed SUID: python3"
sudo chmod u-s /bin/bash 2>/dev/null && echo "Removed SUID: bash"  # SANGAT PENTING!
sudo chmod u-s /bin/sh 2>/dev/null && echo "Removed SUID: sh"
sudo chmod u-s /usr/bin/nc 2>/dev/null && echo "Removed SUID: nc"
sudo chmod u-s /usr/bin/perl 2>/dev/null && echo "Removed SUID: perl"

# ===== Verifikasi ulang =====
echo "=== SUID yang tersisa ==="
find / -perm -4000 -type f 2>/dev/null
```

---

## ✅ FASE 9: PERBAIKI PERMISSION FILE KRITIS (Target: 10 menit)

```bash
# /etc/passwd → 644
sudo chmod 644 /etc/passwd
sudo chown root:root /etc/passwd

# /etc/shadow → 640
sudo chmod 640 /etc/shadow
sudo chown root:shadow /etc/shadow

# /etc/sudoers → 440
sudo chmod 440 /etc/sudoers
sudo chown root:root /etc/sudoers

# /etc/group → 644
sudo chmod 644 /etc/group
sudo chown root:root /etc/group

# /etc/gshadow → 640
sudo chmod 640 /etc/gshadow
sudo chown root:shadow /etc/gshadow

# /tmp sticky bit
sudo chmod 1777 /tmp
sudo chmod 1777 /var/tmp

# Verifikasi
ls -la /etc/passwd /etc/shadow /etc/sudoers /etc/group /etc/gshadow
ls -ld /tmp /var/tmp
```

---

## ✅ FASE 10: KERNEL HARDENING (Target: 10 menit)

```bash
sudo bash -c 'cat >> /etc/sysctl.conf << "EOF"

# === Linux Hardening LKS 2026 ===
# Network security
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1

# Kernel security
kernel.randomize_va_space = 2
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
kernel.sysrq = 0
fs.suid_dumpable = 0
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
EOF'

# Terapkan tanpa reboot
sudo sysctl -p

# Verifikasi
sudo sysctl net.ipv4.tcp_syncookies
sudo sysctl kernel.randomize_va_space
```

---

## ✅ FASE 11: DATABASE SECURITY — BIND LOCALHOST (Target: 10 menit)

```bash
# ===== MySQL (jika ada) =====
if systemctl is-active mysql &>/dev/null; then
    sudo sed -i 's/bind-address.*/bind-address = 127.0.0.1/' /etc/mysql/mysql.conf.d/mysqld.cnf
    sudo systemctl restart mysql
    echo "MySQL: bind-address diubah ke 127.0.0.1"
fi

# ===== Redis (jika ada) =====
if systemctl is-active redis &>/dev/null || systemctl is-active redis-server &>/dev/null; then
    sudo sed -i 's/^bind.*/bind 127.0.0.1/' /etc/redis/redis.conf
    # Tambah password Redis
    echo "requirepass RedisPassword2026!@#" | sudo tee -a /etc/redis/redis.conf
    sudo systemctl restart redis
    echo "Redis: bind diubah ke 127.0.0.1"
fi

# Verifikasi
sudo ss -tulnp | grep -E "3306|6379"
# Harus tampil 127.0.0.1, bukan 0.0.0.0
```

---

## ✅ FASE 12: LOGGING & AUDITD (Target: 20 menit)

```bash
# ===== Pastikan rsyslog berjalan =====
sudo systemctl enable rsyslog
sudo systemctl start rsyslog

# ===== Install dan setup auditd =====
sudo apt install auditd audispd-plugins -y
sudo systemctl enable auditd
sudo systemctl start auditd

# ===== Buat aturan audit =====
sudo bash -c 'cat > /etc/audit/rules.d/hardening.rules << "EOF"
-D
-b 8192
-w /etc/passwd -p wa -k identity_change
-w /etc/shadow -p wa -k identity_change
-w /etc/group -p wa -k identity_change
-w /etc/gshadow -p wa -k identity_change
-w /etc/pam.d/ -p wa -k pam_change
-w /etc/security/pwquality.conf -p wa -k pam_change
-w /etc/security/faillock.conf -p wa -k pam_change
-w /etc/ssh/sshd_config -p wa -k sshd_change
-w /etc/sudoers -p wa -k sudoers_change
-w /usr/bin/sudo -p x -k sudo_usage
-w /usr/sbin/useradd -p x -k user_management
-w /usr/sbin/userdel -p x -k user_management
-w /usr/sbin/usermod -p x -k user_management
-w /etc/cron.d/ -p wa -k cron_change
-w /etc/crontab -p wa -k cron_change
-w /etc/hosts -p wa -k hosts_change
-w /etc/sysctl.conf -p wa -k sysctl_change
-e 2
EOF'

# Terapkan aturan
sudo augenrules --load
sudo systemctl restart auditd

# Verifikasi
sudo auditctl -l

# ===== Konfigurasi journald persistent =====
sudo mkdir -p /etc/systemd/journald.conf.d/
sudo bash -c 'cat > /etc/systemd/journald.conf.d/persistent.conf << "EOF"
[Journal]
Storage=persistent
MaxRetentionSec=30day
Compress=yes
ForwardToSyslog=yes
EOF'
sudo systemctl restart systemd-journald

# ===== Proteksi log dengan chattr =====
sudo chattr +a /var/log/auth.log
sudo chattr +a /var/log/syslog
# Verifikasi:
lsattr /var/log/auth.log    # Harus ada huruf 'a'
```

---

## ✅ FASE 13: AUDIT CRON JOBS (Target: 10 menit)

```bash
# ===== Lihat semua cron =====
echo "=== /etc/crontab ==="
cat /etc/crontab

echo "=== /etc/cron.d/ ==="
ls -la /etc/cron.d/
for f in /etc/cron.d/*; do echo "--- $f ---"; cat "$f" 2>/dev/null; done

echo "=== crontab root ==="
sudo crontab -l

# ===== Cek script yang dipanggil cron =====
# Untuk setiap script yang muncul di crontab, cek permissionnya!
# Script cron yang aman: dimiliki root, permission 700, TIDAK di /tmp
# 
# Jika ada script mencurigakan:
# sudo chown root:root /path/to/script.sh
# sudo chmod 700 /path/to/script.sh
```

---

## ✅ FASE 14: VERIFIKASI FINAL (Target: 20 menit)

```bash
echo "=========================================="
echo "VERIFIKASI FINAL — LINUX HARDENING"
echo "=========================================="

echo ""
echo "1. PORT YANG MASIH TERBUKA:"
sudo ss -tulnp
echo ""

echo "2. SERVICE YANG MASIH BERJALAN:"
sudo systemctl list-units --type=service --state=running
echo ""

echo "3. STATUS UFW:"
sudo ufw status verbose
echo ""

echo "4. STATUS FAIL2BAN:"
sudo fail2ban-client status
echo ""

echo "5. SUID FILES:"
find / -perm -4000 -type f 2>/dev/null | sort
echo ""

echo "6. FILE PERMISSION KRITIS:"
ls -la /etc/passwd /etc/shadow /etc/sudoers /etc/group /etc/gshadow
echo ""

echo "7. KERNEL PARAMETERS:"
sudo sysctl net.ipv4.tcp_syncookies kernel.randomize_va_space kernel.dmesg_restrict
echo ""

echo "8. AUDITD RULES:"
sudo auditctl -l
echo ""

echo "9. LOG TERBARU:"
sudo tail -5 /var/log/auth.log
echo ""

echo "10. USER CHECK:"
echo "--- UID 0 ---"
awk -F: '($3 == 0) {print $1}' /etc/passwd
echo "--- User tanpa password ---"
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
echo "--- Locked accounts ---"
sudo faillock
```

---

## 📝 CHECKLIST MASTER FINAL

Centang semua ini sebelum waktu habis:

### Service
- [ ] FTP (port 21) tidak ada di `ss -tulnp`
- [ ] Telnet (port 23) tidak ada
- [ ] SNMP (port 161) tidak ada
- [ ] rpcbind (port 111) tidak ada (jika tidak diperlukan)
- [ ] MySQL/Redis/MongoDB hanya di 127.0.0.1 (bukan 0.0.0.0)

### SSH
- [ ] `PasswordAuthentication no`
- [ ] `PermitRootLogin no`
- [ ] `MaxAuthTries 3`
- [ ] `sudo sshd -t` tidak ada error
- [ ] SSH berjalan: `sudo systemctl status sshd`

### PAM
- [ ] `minlen = 12` di pwquality.conf
- [ ] `deny = 3` di faillock.conf
- [ ] Baris preauth dan authfail ada di common-auth
- [ ] `PASS_MAX_DAYS 90` di login.defs

### Firewall
- [ ] `sudo ufw status` → active
- [ ] Default: deny incoming

### Permissions
- [ ] `/etc/shadow` = 640
- [ ] `/etc/sudoers` = 440
- [ ] `/tmp` = 1777 (sticky bit)
- [ ] Tidak ada SUID pada bash, sh, find, vim, python3

### Kernel
- [ ] `kernel.randomize_va_space = 2`
- [ ] `net.ipv4.tcp_syncookies = 1`
- [ ] `sudo sysctl -p` tidak ada error

### Logging
- [ ] `sudo systemctl status auditd` → active
- [ ] `sudo auditctl -l` → tampilkan aturan
- [ ] `lsattr /var/log/auth.log` → ada huruf 'a'

---

## 🔗 Navigasi

← `06_Logging.md`
→ `CHEATSHEET_CEPAT.md`
