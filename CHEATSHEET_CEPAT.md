# 🔥 CHEATSHEET CEPAT — Bawa Ke Lomba!
#lks #cheatsheet #hardening #quick-reference

> **PRINT atau simpan offline. Baca ini saat lomba dimulai.**

---

## ⚡ URUTAN HARDENING DI LOMBA (3 Jam!)

```
1. RECON (10 mnt)  →  2. MATIKAN SERVICE (10 mnt)  →  3. SSH (20 mnt)
         ↓
4. PAM PASSWORD (10 mnt)  →  5. PAM LOCKOUT (10 mnt)  →  6. UFW (10 mnt)
         ↓
7. FAIL2BAN (10 mnt)  →  8. SUID FIX (15 mnt)  →  9. PERMISSION (10 mnt)
         ↓
10. SYSCTL (10 mnt)  →  11. DATABASE (10 mnt)  →  12. LOGGING (20 mnt)
         ↓
13. CRON AUDIT (10 mnt)  →  14. VERIFIKASI (20 mnt)
```

---

## 🔍 RECON — Jalankan Ini PERTAMA!

```bash
sudo ss -tulnp                                            # Port terbuka
sudo systemctl list-units --type=service --state=running  # Service jalan
awk -F: '($3 == 0) {print $1}' /etc/passwd               # User UID 0
sudo awk -F: '($2 == "") {print $1}' /etc/shadow          # User tanpa password
find / -perm -4000 -type f 2>/dev/null                   # SUID files
```

---

## 🛑 MATIKAN SERVICE BERBAHAYA

```bash
# Template: stop + disable + purge + verifikasi
sudo systemctl stop vsftpd && sudo systemctl disable vsftpd && sudo apt purge vsftpd -y
sudo systemctl stop telnetd && sudo systemctl disable telnetd && sudo apt purge telnetd -y
sudo systemctl stop snmpd && sudo systemctl disable snmpd && sudo apt purge snmpd -y
sudo systemctl stop rpcbind && sudo systemctl disable rpcbind
sudo ss -tulnp | grep -E ":21|:23|:111|:161"  # Harus KOSONG!
```

---

## 🔐 SSH HARDENING

```bash
# sshd_config wajib:
Port 2222
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
MaxAuthTries 3
X11Forwarding no
AllowTcpForwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
LogLevel VERBOSE

# Test + restart:
sudo sshd -t && sudo systemctl restart sshd
```

---

## 🔑 PAM PASSWORD COMPLEXITY

```bash
# /etc/security/pwquality.conf:
minlen = 12
dcredit = -1 / ucredit = -1 / lcredit = -1 / ocredit = -1
minclass = 4 / maxrepeat = 3 / dictcheck = 1

# /etc/pam.d/common-password (tambahkan di baris paling atas):
password requisite pam_pwquality.so retry=3
password required pam_pwhistory.so remember=5 use_authok

# /etc/login.defs:
PASS_MAX_DAYS 90 / PASS_MIN_DAYS 1 / PASS_WARN_AGE 7
```

---

## 🔒 PAM ACCOUNT LOCKOUT

```bash
# /etc/security/faillock.conf:
deny = 3
unlock_time = 900
fail_interval = 900
audit
silent

# /etc/pam.d/common-auth (urutan ini PENTING!):
auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900
auth [success=1 default=ignore] pam_unix.so nullok
auth [default=die] pam_faillock.so authfail audit deny=3 unlock_time=900
auth requisite pam_deny.so
auth required pam_permit.so

# /etc/pam.d/common-account:
account required pam_faillock.so

# Cek lockout:
sudo faillock --user USERNAME
# Reset lockout:
sudo faillock --user USERNAME --reset
```

---

## 🧱 UFW FIREWALL

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp        # SSH dulu!
sudo ufw allow 80/tcp          # HTTP (jika perlu)
sudo ufw allow 443/tcp         # HTTPS (jika perlu)
sudo ufw limit 2222/tcp        # Rate limit SSH
sudo ufw enable
sudo ufw status verbose
```

---

## 🚫 FAIL2BAN

```bash
# /etc/fail2ban/jail.local:
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1
findtime = 600 / maxretry = 3 / bantime = 3600

[sshd]
enabled = true / port = 2222
filter = sshd / logpath = /var/log/auth.log

sudo systemctl enable fail2ban && sudo systemctl start fail2ban
sudo fail2ban-client status sshd
```

---

## 🏴‍☠️ SUID FIX

```bash
# Cari:
find / -perm -4000 -type f 2>/dev/null

# BOLEH ada SUID: sudo, passwd, su, newgrp, chsh, gpasswd, ping, mount
# HAPUS SUID dari: bash, sh, find, vim, python3, perl, nc, cp, less

sudo chmod u-s /bin/bash /usr/bin/find /usr/bin/vim /usr/bin/python3 2>/dev/null
```

---

## 🛡️ PERMISSION FILE KRITIS

```bash
sudo chmod 644 /etc/passwd && sudo chown root:root /etc/passwd
sudo chmod 640 /etc/shadow && sudo chown root:shadow /etc/shadow
sudo chmod 440 /etc/sudoers && sudo chown root:root /etc/sudoers
sudo chmod 644 /etc/group && sudo chown root:root /etc/group
sudo chmod 640 /etc/gshadow && sudo chown root:shadow /etc/gshadow
sudo chmod 1777 /tmp /var/tmp   # Sticky bit
```

---

## 🔧 KERNEL HARDENING (sysctl)

```bash
# Tambahkan ke /etc/sysctl.conf:
net.ipv4.tcp_syncookies = 1        # Anti SYN flood
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.log_martians = 1
kernel.randomize_va_space = 2      # ASLR max
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
kernel.sysrq = 0
fs.suid_dumpable = 0
fs.protected_hardlinks = 1
fs.protected_symlinks = 1

# Terapkan:
sudo sysctl -p
```

---

## 📊 AUDITD

```bash
sudo apt install auditd -y
sudo systemctl enable auditd && sudo systemctl start auditd

# /etc/audit/rules.d/hardening.rules:
-D
-b 8192
-w /etc/passwd -p wa -k identity_change
-w /etc/shadow -p wa -k identity_change
-w /etc/sudoers -p wa -k sudoers_change
-w /usr/bin/sudo -p x -k sudo_usage
-w /etc/ssh/sshd_config -p wa -k sshd_change
-w /etc/crontab -p wa -k cron_change
-e 2

sudo augenrules --load && sudo systemctl restart auditd
sudo auditctl -l
```

---

## 📋 PROTEKSI LOG

```bash
sudo chattr +a /var/log/auth.log
sudo chattr +a /var/log/syslog
sudo chattr +a /var/log/audit/audit.log
lsattr /var/log/auth.log    # Harus ada 'a'
```

---

## 🔍 MONITORING LOG — COMMAND PENTING

```bash
sudo tail -f /var/log/auth.log                  # Live monitor
grep "Failed password" /var/log/auth.log        # Login gagal
grep "Accepted" /var/log/auth.log               # Login berhasil
grep "Invalid user" /var/log/auth.log           # User tidak dikenal
sudo journalctl -u sshd -f                      # Log SSH live
w                                               # Siapa yang login sekarang
last | head -20                                 # Riwayat login
sudo faillock                                   # Cek lockout
sudo aureport --summary                         # Ringkasan audit
sudo ausearch -k identity_change --start today  # Perubahan file user hari ini
```

---

## ❓ Q&A CEPAT (Sering Ditanya!)

| Pertanyaan | Jawaban |
|-----------|---------|
| Kunci akun setelah 3x gagal? | `deny = 3` di `/etc/security/faillock.conf` |
| Password minimal 12 karakter? | `minlen = 12` di `/etc/security/pwquality.conf` |
| Matikan SSH password login? | `PasswordAuthentication no` di `sshd_config` |
| Lihat port terbuka? | `sudo ss -tulnp` |
| Matikan service? | `systemctl stop X && systemctl disable X` |
| Monitor perubahan /etc/passwd? | `auditd` rule: `-w /etc/passwd -p wa -k identity_change` |
| Cari SUID berbahaya? | `find / -perm -4000 -type f 2>/dev/null` |
| Aktifkan ASLR? | `kernel.randomize_va_space = 2` di `sysctl.conf` |
| Cegah SYN flood? | `net.ipv4.tcp_syncookies = 1` |
| Log tidak bisa dihapus? | `sudo chattr +a /var/log/auth.log` |
| Paksa ganti password? | `sudo chage -d 0 username` |
| Cek user tanpa password? | `sudo awk -F: '($2 == "") {print $1}' /etc/shadow` |
| Reset akun yang terkunci? | `sudo faillock --user USERNAME --reset` |
| Test syntax SSH? | `sudo sshd -t` |
| Terapkan sysctl? | `sudo sysctl -p` |
| Cek aturan audit aktif? | `sudo auditctl -l` |
