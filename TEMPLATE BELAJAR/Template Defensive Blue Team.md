# 🔵 Template Defensive Blue Team CTF — Modul C

  

> **Modul C mencakup:** Digital Forensics, Reverse Engineering, SIEM/Threat Hunting, Network Forensics

  

---

  

## ✅ CHECKLIST ANALISIS

  

```

[ ] 1. File Forensics (file, strings, binwalk, exiftool)

[ ] 2. Image/Media Forensics (steghide, stegsolve, zsteg)

[ ] 3. Network Forensics (Wireshark, tshark)

[ ] 4. Memory Forensics (Volatility)

[ ] 5. Reverse Engineering (Ghidra, strings, ltrace)

[ ] 6. SIEM / Splunk Threat Hunting

[ ] 7. Submit flag + Writeup

```

  

---

  

## 1. FILE ANALYSIS — MULAI DI SINI

  

```bash

# Identifikasi tipe file (SELALU mulai sini):

file <filename>

file *

  

# Magic bytes manual:

xxd <filename> | head -20

hexdump -C <filename> | head -20

  

# Strings (cari teks tersembunyi):

strings <filename>

strings -n 8 <filename>          # minimum 8 karakter

strings <filename> | grep -i flag

strings <filename> | grep -iE "CTF|FLAG|key|password|secret"

  

# Metadata:

exiftool <filename>

  

# Binwalk (cari file tersembunyi di dalam file):

binwalk <filename>

binwalk -e <filename>            # extract otomatis

binwalk -M -e <filename>         # recursive extract

  

# Entropy analysis:

binwalk -E <filename>            # plot entropy (butuh matplotlib)

```

  

---

  

## 2. IMAGE / MEDIA FORENSICS

  

### Steganografi

  

```bash

# Steghide (sembunyikan/ekstrak di JPG/BMP):

steghide info <image.jpg>                    # cek apakah ada data

steghide extract -sf <image.jpg>             # ekstrak tanpa passphrase

steghide extract -sf <image.jpg> -p "password"

  

# zsteg (untuk PNG/BMP):

zsteg <image.png>

zsteg -a <image.png>                         # semua channel

zsteg -E "b1,r,lsb,xy" <image.png>         # channel tertentu

  

# stegsolve (GUI, untuk visual analysis):

java -jar stegsolve.jar

  

# outguess:

outguess -r <image.jpg> output.txt

  

# LSB manual (Python):

python3 << "EOF"

from PIL import Image

  

img = Image.open("image.png")

pixels = list(img.getdata())

  

bits = ""

for pixel in pixels[:1000]:  # ambil 1000 pixel pertama

    for value in pixel[:3]:   # R, G, B

        bits += str(value & 1)  # LSB

  

# Konversi bits ke karakter:

chars = [chr(int(bits[i:i+8], 2)) for i in range(0, len(bits), 8)]

print(''.join(chars[:50]))

EOF

```

  

### Fix PNG yang Rusak

  

```python

# Script fix_png.py — copy-paste, ganti nama file saja

import struct

  

def fix_png(filename):

    with open(filename, 'rb') as f:

        data = bytearray(f.read())

    # Magic bytes PNG yang benar: 89 50 4E 47 0D 0A 1A 0A

    correct_magic = bytes([0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A])

    if data[:8] != correct_magic:

        print(f"[!] Magic bytes rusak: {data[:8].hex()}")

        data[:8] = correct_magic

        print(f"[+] Magic bytes diperbaiki!")

    # Fix IHDR jika perlu:

    # Offset 8-12: panjang IHDR (harus 0x0000000D = 13)

    # Offset 12-16: "IHDR"

    output = filename.replace('.', '_fixed.')

    with open(output, 'wb') as f:

        f.write(data)

    print(f"[+] File disimpan: {output}")

  

fix_png("NAMA_FILE.png")  # <-- GANTI INI

```

  

### Fix JPG yang Rusak

  

```python

# Script fix_jpg.py

def fix_jpg(filename):

    with open(filename, 'rb') as f:

        data = bytearray(f.read())

    # Magic bytes JPG: FF D8 FF

    correct_magic = bytes([0xFF, 0xD8, 0xFF])

    if data[:3] != correct_magic:

        print(f"[!] Magic bytes JPG rusak: {data[:3].hex()}")

        data[:3] = correct_magic

        print(f"[+] Magic bytes diperbaiki!")

    output = filename.replace('.', '_fixed.')

    with open(output, 'wb') as f:

        f.write(data)

    print(f"[+] File disimpan: {output}")

  

fix_jpg("NAMA_FILE.jpg")  # <-- GANTI INI

```

  

---

  

## 3. NETWORK FORENSICS — WIRESHARK / TSHARK

  

### Filter Wireshark — Siap Pakai

  

```

# HTTP traffic:

http

  

# Hanya request GET:

http.request.method == "GET"

  

# Cari string dalam payload:

frame contains "password"

frame contains "flag"

frame contains "CTF"

  

# IP tertentu:

ip.addr == 192.168.1.1

ip.src == 192.168.1.1

ip.dst == 192.168.1.1

  

# Port tertentu:

tcp.port == 80

tcp.port == 443

udp.port == 53

  

# DNS queries:

dns

  

# FTP:

ftp || ftp-data

  

# Credentials via FTP:

ftp.request.command == "USER" || ftp.request.command == "PASS"

  

# TCP stream yang mengandung kata kunci:

tcp contains "login"

tcp contains "password"

  

# SMB:

smb || smb2

  

# HTTP POST (biasanya ada credentials):

http.request.method == "POST"

  

# Cari file yang ditransfer:

http.response.code == 200 && http contains "attachment"

```

  

### TShark — Command Line

  

```bash

# Konversi pcap ke text:

tshark -r capture.pcap

  

# Ekstrak semua HTTP request:

tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

  

# Ekstrak credentials:

tshark -r capture.pcap -Y "ftp.request.command == PASS" -T fields -e ftp.request.arg

  

# Ikuti TCP stream:

tshark -r capture.pcap -q -z follow,tcp,ascii,0

  

# Export objek HTTP (file yang didownload):

tshark -r capture.pcap --export-objects http,./extracted/

  

# Statistik koneksi:

tshark -r capture.pcap -q -z conv,tcp

```

  

---

  

## 4. MEMORY FORENSICS — VOLATILITY

  

```bash

# Volatility 2 (vol.py) — paling umum di CTF

# Volatility 3 (vol3) — syntax berbeda

  

# ===== VOLATILITY 2 =====

  

# Identifikasi profil OS:

vol.py -f memory.raw imageinfo

vol.py -f memory.raw kdbgscan

  

# Setelah dapat profil (contoh: Win7SP1x64):

PROFILE="Win7SP1x64"

  

# List proses:

vol.py -f memory.raw --profile=$PROFILE pslist

vol.py -f memory.raw --profile=$PROFILE pstree

vol.py -f memory.raw --profile=$PROFILE psscan       # deteksi proses tersembunyi

  

# Koneksi jaringan:

vol.py -f memory.raw --profile=$PROFILE netscan

vol.py -f memory.raw --profile=$PROFILE connections   # XP/2003

  

# Command history:

vol.py -f memory.raw --profile=$PROFILE cmdscan

vol.py -f memory.raw --profile=$PROFILE consoles

  

# Registry:

vol.py -f memory.raw --profile=$PROFILE hivelist

vol.py -f memory.raw --profile=$PROFILE printkey -K "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

  

# Dump file dari memory:

vol.py -f memory.raw --profile=$PROFILE filescan | grep -i ".txt\|.docx\|.pdf"

vol.py -f memory.raw --profile=$PROFILE dumpfiles -Q 0x<OFFSET> -D ./dump/

  

# Dump proses:

vol.py -f memory.raw --profile=$PROFILE procdump -p <PID> -D ./dump/

vol.py -f memory.raw --profile=$PROFILE memdump -p <PID> -D ./dump/

  

# Malware detection:

vol.py -f memory.raw --profile=$PROFILE malfind

  

# Cari string di memory:

strings memory.raw | grep -i "flag\|CTF\|password"

strings -el memory.raw | grep -i "flag"   # Unicode

  

# ===== VOLATILITY 3 =====

vol3 -f memory.raw windows.pslist

vol3 -f memory.raw windows.netscan

vol3 -f memory.raw windows.cmdline

vol3 -f memory.raw windows.malfind

vol3 -f memory.raw windows.filescan | grep -i flag

```

  

---

  

## 5. REVERSE ENGINEERING

  

### Static Analysis

  

```bash

# Cek tipe binary:

file binary

checksec binary

  

# Strings:

strings binary | grep -iE "flag|key|pass|secret|CTF"

  

# Library yang dipakai:

ldd binary

objdump -d binary | head -100

  

# ltrace (lihat function calls):

ltrace ./binary

  

# strace (lihat syscalls):

strace ./binary

  

# Ghidra (GUI decompiler) — buka via:

# ghidraRun → New Project → Import File → Binary

# Analyze → Double-click fungsi main di Symbol Tree

  

# radare2 (command line):

r2 ./binary

# Di dalam r2:

# aaa      → analyze all

# afl      → list functions

# pdf @main → disassemble main

# iz       → strings

# q        → quit

```

  

### Bypass Sederhana

  

```python

# Jika binary tanya password, coba:

# 1. strings untuk lihat password hardcoded

# 2. ltrace untuk intersep strcmp

# 3. GDB: break di strcmp, lihat argumen

  

# GDB bypass:

# gdb ./binary

# b strcmp       → break di strcmp

# run

# x/s $rdi      → lihat argumen pertama

# x/s $rsi      → lihat argumen kedua

```

  

---

  

## 6. SIEM / SPLUNK — QUERY TEMPLATE

  

### Basic SPL Queries

  

```splunk

# Semua event:

index=* | head 100

  

# Filter berdasarkan EventCode:

index=* EventCode=4625 | table _time, Account_Name, IpAddress, Failure_Reason

  

# Cari brute force (banyak login failed):

index=* EventCode=4625

| stats count by IpAddress, Account_Name

| where count > 10

| sort -count

  

# Deteksi brute force + success (spray attack):

index=* (EventCode=4625 OR EventCode=4624)

| stats

    count(eval(EventCode=4625)) as failed_logins,

    count(eval(EventCode=4624)) as success_logins

    by IpAddress, Account_Name

| where failed_logins > 5 AND success_logins > 0

| sort -failed_logins

  

# Timeline serangan:

index=* EventCode=4625

| timechart span=1m count by IpAddress

  

# Cari akun yang di-disable/enable:

index=* (EventCode=4722 OR EventCode=4725)

| table _time, Target_Account_Name, Subject_Account_Name, EventCode

  

# Deteksi mimikatz/pass-the-hash:

index=* EventCode=4624 Logon_Type=3

| where NOT (Account_Name="ANONYMOUS LOGON" OR Account_Name="-")

| stats count by IpAddress, Account_Name

| where count > 5

  

# Cari privilege escalation:

index=* EventCode=4672

| table _time, Account_Name, Privileges

  

# PowerShell execution:

index=* source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104

| rex field=Message "ScriptBlockText = (?P<script>.*)"

| table _time, script

| search script="*invoke*" OR script="*download*" OR script="*encode*"

  

# Proses mencurigakan:

index=* source="WinEventLog:Security" EventCode=4688

| where

    Process_Command_Line LIKE "%powershell%" OR

    Process_Command_Line LIKE "%cmd%" OR

    Process_Command_Line LIKE "%wscript%" OR

    Process_Command_Line LIKE "%mshta%"

| table _time, Account_Name, Process_Name, Process_Command_Line

```

  

### Splunk Dashboard Cepat

  

```splunk

# Top source IPs:

index=* | top limit=20 src_ip

  

# Unusual outbound connections:

index=* dest_port!=80 dest_port!=443 dest_port!=53

| stats count by src_ip, dest_ip, dest_port

| sort -count

  

# DNS exfiltration detection:

index=* sourcetype=stream:dns

| eval query_length=len(query)

| where query_length > 50

| stats count by query, src_ip

| sort -query_length

```

  

---

  

## 7. DIGITAL FORENSICS — PHONE & DISK

  

```bash

# Disk image:

# Mount image:

mount -o loop,ro disk.img /mnt/evidence

  

# Autopsy (GUI):

autopsy   # buka di browser: http://localhost:9999/autopsy

  

# Volatility untuk Android memory:

# Biasanya gunakan: adb, apktool, jadx

  

# APK Analysis:

apktool d app.apk -o output/

jadx -d output/ app.apk

# Cari: strings, hardcoded credentials, URLs

grep -r "password\|secret\|key\|api" output/

  

# SQLite database (Android):

sqlite3 database.db

.tables

.schema

SELECT * FROM tablename;

  

# Artifact lokasi umum Android:

# /data/data/<app_package>/databases/

# /data/data/<app_package>/shared_prefs/

# /sdcard/

```

  

---

  

## 8. YARA RULES — TEMPLATE

  

```yara

rule DetectMalware {

    meta:

        author = "peserta_lks"

        description = "Deteksi malware berdasarkan string"

        date = "2026"

    strings:

        $str1 = "malicious_string" nocase

        $str2 = { 4D 5A 90 00 }  // MZ header

        $str3 = /CTF\{[a-zA-Z0-9_]+\}/ // Regex flag

    condition:

        any of them

}

  

# Jalankan YARA:

# yara rules.yar /path/to/files

# yara -r rules.yar /folder/   # recursive

```

  

---

  

## 9. SIGMA RULES — TEMPLATE

  

```yaml

title: Deteksi Brute Force Login

id: 12345678-1234-1234-1234-123456789012

status: test

description: Mendeteksi percobaan login berulang yang gagal

author: peserta_lks

date: 2026/01/01

logsource:

    product: windows

    service: security

detection:

    selection:

        EventID: 4625

    condition: selection | count() > 10

fields:

    - IpAddress

    - Account_Name

falsepositives:

    - Pengguna yang lupa password

level: medium

tags:

    - attack.credential_access

    - attack.t1110

```

  

---

  

## 10. z3 SOLVER — TEMPLATE

  

```python

# Template z3 untuk CTF challenge matematika / constraint solving

from z3 import *

  

# Inisiasi solver:

s = Solver()

  

# Deklarasi variabel:

x = Int('x')

y = Int('y')

z = Int('z')

# Untuk karakter: c = BitVec('c', 8)

  

# Tambahkan constraint (sesuai kondisi soal):

s.add(x + y == 100)

s.add(x * 2 - z == 50)

s.add(y > 0, z > 0, x > 0)

  

# Selesaikan:

if s.check() == sat:

    model = s.model()

    print(f"x = {model[x]}")

    print(f"y = {model[y]}")

    print(f"z = {model[z]}")

else:

    print("Tidak ada solusi!")

  

# ===== Contoh untuk reverse engineering flag =====

from z3 import *

  

s = Solver()

flag = [BitVec(f'c{i}', 8) for i in range(20)]  # flag 20 karakter

  

# Constraint karakter printable:

for c in flag:

    s.add(c >= 32, c <= 126)

  

# Constraint dari binary (sesuaikan dengan hasil reverse):

# Contoh: flag[0] + flag[1] == 200

s.add(flag[0] + flag[1] == 200)

s.add(flag[0] ^ flag[2] == 0x41)

# ... tambahkan constraint lainnya ...

  

if s.check() == sat:

    m = s.model()

    result = ''.join(chr(m[c].as_long()) for c in flag)

    print(f"Flag: {result}")

```

  

---

  

## 11. TEMPLATE WRITEUP / LAPORAN

  

```markdown

# Writeup: [NAMA SOAL]

  

## Informasi Soal

- **Kategori:** [Web/Forensics/Crypto/Rev/Pwn]

- **Points:** [poin]

- **Description:** [deskripsi soal]

  

## Analisis Awal

[Jelaskan langkah pertama, apa yang diidentifikasi]

  

## Langkah Penyelesaian

  

### Step 1: [Nama langkah]

[Deskripsi singkat apa yang dilakukan]

  

```bash

# Perintah yang digunakan:

command -arg output

```

  

[Screenshot/output relevan]

  

### Step 2: [Nama langkah]

...

  

## Flag

```

CTF{flag_disini}

```

  

## Tools yang Digunakan

- [tool 1]

- [tool 2]

  

## Lesson Learned

[Apa yang dipelajari dari soal ini]

```

  

---

  

> 💡 **Tips Modul C:**

> - Selalu mulai dengan `file` dan `strings` — sering sudah cukup untuk soal mudah

> - Di Splunk, mulai dengan query luas lalu persempit dengan filter

> - Writeup adalah bagian dari penilaian (Judgement) — buat jelas dan ada screenshot!

> - Untuk forensics, jangan modifikasi file original — selalu buat copy dulu!