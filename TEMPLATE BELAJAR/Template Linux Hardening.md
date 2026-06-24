# 🐧 Template Linux Hardening — Modul A

  

> **Cara pakai:** Copy-paste blok `cat << "EOF"` langsung ke terminal. Jangan ketik manual!

  

---

  

## ✅ CHECKLIST URUTAN EKSEKUSI

  

```

[ ] 1. Update & upgrade sistem

[ ] 2. Konfigurasi sysctl (kernel hardening)

[ ] 3. Konfigurasi PAM (password policy + lockout)

[ ] 4. Setup auditd rules

[ ] 5. Konfigurasi SSH hardening

[ ] 6. Setup Fail2ban

[ ] 7. Firewall (UFW/iptables)

[ ] 8. Disable layanan tidak perlu

[ ] 9. File permission hardening

[ ] 10. Cron & SUID/SGID check

```

  

---

  

## 1. UPDATE SISTEM

  

```bash

apt update && apt upgrade -y

# atau untuk RedHat-based:

yum update -y

```

  

---

  

## 2. KERNEL SYSCTL — `/etc/sysctl.conf`

  

```bash

cat << "EOF" >> /etc/sysctl.conf

# === NETWORK HARDENING ===

net.ipv4.tcp_syncookies = 1

net.ipv4.ip_forward = 0

net.ipv4.conf.all.accept_redirects = 0

net.ipv4.conf.default.accept_redirects = 0

net.ipv4.conf.all.secure_redirects = 0

net.ipv4.conf.default.secure_redirects = 0

net.ipv4.conf.all.send_redirects = 0

net.ipv4.conf.default.send_redirects = 0

net.ipv4.conf.all.accept_source_route = 0

net.ipv4.conf.default.accept_source_route = 0

net.ipv4.conf.all.log_martians = 1

net.ipv4.conf.default.log_martians = 1

net.ipv4.icmp_echo_ignore_broadcasts = 1

net.ipv4.icmp_ignore_bogus_error_responses = 1

net.ipv6.conf.all.accept_redirects = 0

net.ipv6.conf.default.accept_redirects = 0

net.ipv6.conf.all.accept_source_route = 0

  

# === KERNEL HARDENING ===

kernel.randomize_va_space = 2

kernel.dmesg_restrict = 1

kernel.kptr_restrict = 2

kernel.sysrq = 0

kernel.core_uses_pid = 1

kernel.pid_max = 65536

fs.suid_dumpable = 0

fs.protected_hardlinks = 1

fs.protected_symlinks = 1

EOF

  

sysctl -p

```

  

---

  

## 3. PAM — PASSWORD POLICY

  

### `/etc/security/pwquality.conf`

  

```bash

cat << "EOF" > /etc/security/pwquality.conf

minlen = 14

dcredit = -1

ucredit = -1

lcredit = -1

ocredit = -1

minclass = 4

maxrepeat = 3

maxsequence = 3

gecoscheck = 1

dictcheck = 1

EOF

```

  

### `/etc/security/faillock.conf`

  

```bash

cat << "EOF" > /etc/security/faillock.conf

deny = 5

fail_interval = 900

unlock_time = 900

audit

silent

even_deny_root

EOF

```

  

### `/etc/pam.d/common-password` (Debian/Ubuntu)

  

```bash

# Pastikan baris ini ada (tambahkan jika tidak ada):

# password requisite pam_pwquality.so retry=3

# password required pam_pwhistory.so remember=5 use_authtok

sed -i '/pam_pwquality/d' /etc/pam.d/common-password

sed -i '1s/^/password requisite pam_pwquality.so retry=3\n/' /etc/pam.d/common-password

```

  

### `/etc/login.defs` — Password aging

  

```bash

sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   90/' /etc/login.defs

sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   7/' /etc/login.defs

sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE   14/' /etc/login.defs

```

  

---

  

## 4. AUDITD RULES — `/etc/audit/rules.d/hardening.rules`

  

```bash

cat << "EOF" > /etc/audit/rules.d/hardening.rules

# Delete all rules

-D

  

# Buffer size

-b 8192

  

# Failure mode

-f 1

  

# === IDENTITY & AUTH ===

-w /etc/passwd -p wa -k identity

-w /etc/group -p wa -k identity

-w /etc/shadow -p wa -k identity

-w /etc/gshadow -p wa -k identity

-w /etc/security/opasswd -p wa -k identity

  

# === SUDOERS ===

-w /etc/sudoers -p wa -k sudoers

-w /etc/sudoers.d/ -p wa -k sudoers

  

# === LOGIN & LOGOUT ===

-w /var/log/lastlog -p wa -k logins

-w /var/run/faillock/ -p wa -k logins

-w /var/log/tallylog -p wa -k logins

  

# === SESSION ===

-w /var/run/utmp -p wa -k session

-w /var/log/wtmp -p wa -k session

-w /var/log/btmp -p wa -k session

  

# === PRIVILEGE ESCALATION ===

-a always,exit -F arch=b64 -S setuid -F a0=0 -F exe=/usr/bin/su -k elevated_privs

-a always,exit -F arch=b64 -S setresuid -F a0=0 -F exe=/usr/bin/sudo -k elevated_privs

-a always,exit -F arch=b64 -S execve -C uid!=euid -F euid=0 -k elevated_privs

  

# === NETWORK CONFIG ===

-a always,exit -F arch=b64 -S sethostname -S setdomainname -k network_modifications

-w /etc/hosts -p wa -k network_modifications

-w /etc/network/ -p wa -k network_modifications

-w /etc/sysconfig/network -p wa -k network_modifications

  

# === SYSTEM STARTUP ===

-w /etc/inittab -p wa -k init

-w /etc/init.d/ -p wa -k init

-w /etc/init/ -p wa -k init

-w /etc/crontab -p wa -k cron

-w /etc/cron.hourly/ -p wa -k cron

-w /etc/cron.daily/ -p wa -k cron

-w /etc/cron.weekly/ -p wa -k cron

-w /etc/cron.monthly/ -p wa -k cron

-w /etc/cron.d/ -p wa -k cron

-w /var/spool/cron/ -p wa -k cron

  

# === KERNEL MODULE ===

-w /sbin/insmod -p x -k modules

-w /sbin/rmmod -p x -k modules

-w /sbin/modprobe -p x -k modules

-a always,exit -F arch=b64 -S init_module -k modules

  

# === SSH CONFIG ===

-w /etc/ssh/sshd_config -p wa -k sshd

  

# === PAM ===

-w /etc/pam.d/ -p wa -k pam

-w /etc/security/limits.conf -p wa -k pam

  

# === SYSCALL — FILE DELETION ===

-a always,exit -F arch=b64 -S unlink -S unlinkat -S rename -S renameat -F auid>=1000 -F auid!=4294967295 -k delete

  

# === MAKE IMMUTABLE ===

-e 2

EOF

  

service auditd restart

# atau:

systemctl restart auditd

```

  

---

  

## 5. SSH HARDENING — `/etc/ssh/sshd_config`

  

```bash

cat << "EOF" >> /etc/ssh/sshd_config

  

# === HARDENING BLOCK ===

Protocol 2

PermitRootLogin no

MaxAuthTries 4

PubkeyAuthentication yes

PasswordAuthentication yes

PermitEmptyPasswords no

ChallengeResponseAuthentication no

UsePAM yes

X11Forwarding no

PrintLastLog yes

TCPKeepAlive yes

MaxStartups 10:30:60

LoginGraceTime 60

AllowTcpForwarding no

EOF

  

sshd -t && systemctl restart sshd

```

  

---

  

## 6. FAIL2BAN — `/etc/fail2ban/jail.local`

  

```bash

apt install fail2ban -y

  

cat << "EOF" > /etc/fail2ban/jail.local

[DEFAULT]

bantime  = 3600

findtime = 600

maxretry = 5

backend  = auto

  

[sshd]

enabled  = true

port     = ssh

filter   = sshd

logpath  = /var/log/auth.log

maxretry = 4

bantime  = 7200

EOF

  

systemctl enable fail2ban

systemctl restart fail2ban

  

# Cek status:

fail2ban-client status sshd

```

  

---

  

## 7. FIREWALL — UFW

  

```bash

apt install ufw -y

ufw default deny incoming

ufw default allow outgoing

ufw allow ssh

ufw allow 22/tcp

# Tambahkan port sesuai kebutuhan soal:

# ufw allow 80/tcp

# ufw allow 443/tcp

ufw --force enable

ufw status verbose

```

  

---

  

## 8. DISABLE LAYANAN TIDAK PERLU

  

```bash

# Cek service yang berjalan:

systemctl list-units --type=service --state=running

  

# Disable service umum yang tidak perlu:

for svc in telnet rsh rlogin rexec cups avahi-daemon; do

    systemctl disable --now $svc 2>/dev/null && echo "Disabled: $svc"

done

  

# Disable protokol jaringan tidak perlu:

cat << "EOF" >> /etc/modprobe.d/hardening.conf

install dccp /bin/true

install sctp /bin/true

install rds /bin/true

install tipc /bin/true

EOF

```

  

---

  

## 9. FILE PERMISSION HARDENING

  

```bash

# Cek file SUID/SGID:

find / -perm /4000 -type f 2>/dev/null

find / -perm /2000 -type f 2>/dev/null

  

# Hardening permission kritis:

chmod 640 /etc/shadow

chmod 644 /etc/passwd

chmod 644 /etc/group

chmod 600 /etc/gshadow

chmod 644 /etc/crontab

chmod 700 /root

chmod 600 /root/.bashrc

  

# World-writable files (wajib dicek):

find / -xdev -type f -perm -0002 2>/dev/null

```

  

---

  

## 10. VERIFIKASI AKHIR

  

```bash

# Cek semua service yang listen:

ss -tulnp

netstat -tulnp

  

# Cek user dengan UID 0 (seharusnya hanya root):

awk -F: '($3 == "0") {print}' /etc/passwd

  

# Cek user tanpa password:

awk -F: '($2 == "") {print}' /etc/shadow

  

# Cek auditd berjalan:

systemctl status auditd

  

# Cek fail2ban berjalan:

fail2ban-client status

  

# Cek sysctl sudah apply:

sysctl net.ipv4.tcp_syncookies

sysctl kernel.randomize_va_space

```

  

---

  

> 💡 **Tips Lomba:** Selesaikan hardening dari atas ke bawah secara berurutan. Verifikasi setiap langkah sebelum lanjut ke langkah berikutnya!