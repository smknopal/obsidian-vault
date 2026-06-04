---

title: Linux CLI Intensif — grep, awk, find, pipe, redirect

tags: [LKS2026, BlueTeam, Linux, CTF, CLI, Security]

date: 2026-06-02

cssclasses: [wide-page]

---
---

tags:

  - LKS2026

  - BlueTeam

  - Linux

  - CTF

  - Forensik

cssclasses:

  - wide-page

created: 2026-06-02

status: aktif

---

  

# 🐧 Linux CLI Intensif — Part 1

### `grep` · `awk` · `find` · `pipe` · `redirect`

> **LKS 2026 · Blue Team Preparation** | Defensive CTF & Log Analysis

  

---

  

## 🧠 Kenapa Ini Krusial di CTF Blue Team?

  

> [!tip] Konteks LKS Blue Team

> Modul C (Defensive CTF) mensyaratkan **threat hunting**, **anomaly detection**, dan **log forensics**. Semua itu = baca, filter, dan analisis log secara cepat dan presisi.

>

> Tanpa `grep`+`awk`+`find`, kamu akan manual scroll ratusan ribu baris log → **wasted time = wasted point**.

  

| Tool | Fungsi Utama | Kapan Dipakai di CTF |

|------|-------------|----------------------|

| `grep` | Filter teks berdasarkan pola/regex | Cari IP, keyword, error di log |

| `awk` | Ekstrak & proses kolom data terstruktur | Parse log `/var/log/auth.log`, CSV |

| `find` | Cari file di filesystem | Temukan evidence file, malware artifact |

| `\|` (pipe) | Sambung output → input antar command | Rantai filter multi-tahap |

| `>` / `>>` | Simpan output ke file | Dokumentasi temuan / write-up |

  

---

  

## ⚡ GREP — Filter Teks Secara Bedah

  

### Konsep

`grep` = **G**lobal **R**egular **E**xpression **P**rint. Ambil baris yang cocok pola dari file atau stdin.

  

```

grep [OPTIONS] "PATTERN" [FILE]

```

  

### Cheatsheet Lengkap

  

| Flag | Fungsi | Contoh |

|------|--------|--------|

| *(none)* | Case-sensitive match | `grep "Failed" auth.log` |

| `-i` | Case-insensitive | `grep -i "failed" auth.log` |

| `-n` | Tampilkan nomor baris | `grep -n "error" app.log` |

| `-c` | Hitung jumlah baris cocok | `grep -c "404" access.log` |

| `-v` | Invert — tampilkan yang **tidak** cocok | `grep -v "GET" access.log` |

| `-r` | Rekursif ke semua file dalam direktori | `grep -r "password" /var/log/` |

| `-l` | Tampilkan hanya nama file yang cocok | `grep -rl "malware" /tmp/` |

| `-E` | Extended regex (pakai `+`, `\|`, `()`) | `grep -E "192\.168\.(1\|2)\." access.log` |

| `-o` | Tampilkan hanya bagian yang cocok | `grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" auth.log` |

| `-A N` | Tampilkan N baris **setelah** match | `grep -A 3 "CRITICAL" syslog` |

| `-B N` | Tampilkan N baris **sebelum** match | `grep -B 2 "segfault" syslog` |

| `-C N` | Tampilkan N baris **sebelum+sesudah** | `grep -C 2 "login failed" auth.log` |

| `--color` | Highlight pattern di output | `grep --color "root" passwd` |

  

### Contoh Output Nyata

  

**Input: `auth.log` (cuplikan)**

```

Jun  2 01:23:41 server sshd[1234]: Failed password for root from 10.10.10.5 port 22

Jun  2 01:23:42 server sshd[1234]: Failed password for root from 10.10.10.5 port 22

Jun  2 01:23:45 server sshd[1234]: Accepted password for admin from 10.10.10.7 port 22

Jun  2 01:23:50 server sshd[1234]: Failed password for invalid user hacker from 10.10.10.9

```

  

```bash

# Berapa kali login gagal?

grep -c "Failed" auth.log

```

```

3

```

  

```bash

# Siapa saja yang berhasil login?

grep "Accepted" auth.log

```

```

Jun  2 01:23:45 server sshd[1234]: Accepted password for admin from 10.10.10.7 port 22

```

  

```bash

# Ekstrak semua IP unik dari log (kombinasi grep+sort+uniq)

grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" auth.log | sort | uniq -c | sort -rn

```

```

3 10.10.10.5

1 10.10.10.9

1 10.10.10.7

```

  

> [!example] **Praktek Langsung**

> Buat file `test.log` berisi 5 baris, 2 di antaranya mengandung kata "ERROR".

> ```bash

> echo -e "INFO start\nERROR disk full\nINFO running\nERROR timeout\nINFO done" > test.log

> grep -n "ERROR" test.log

> grep -c "ERROR" test.log

> grep -v "ERROR" test.log

> ```

> Perhatikan perbedaan output masing-masing flag!

  

---

  

## ⚙️ AWK — Bedah Kolom Data

  

### Konsep

`awk` = prosesor teks berbasis **field (kolom)**. Setiap baris dibagi jadi field `$1`, `$2`, ..., `$NF`. Default separator = spasi/tab.

  

```

awk [OPTIONS] 'CONDITION { ACTION }' [FILE]

```

  

### Variable Built-in Paling Penting

  

| Variable | Arti |

|----------|------|

| `$0` | Seluruh baris |

| `$1`, `$2`, ... | Kolom ke-1, ke-2, dst. |

| `$NF` | Kolom **terakhir** |

| `NR` | Nomor baris saat ini |

| `NF` | Jumlah kolom di baris saat ini |

| `FS` | Field Separator (default: spasi) |

| `OFS` | Output Field Separator |

  

### Cheatsheet Lengkap

  

```bash

# Print kolom tertentu

awk '{print $1, $4}' access.log

  

# Ganti separator input (misal: colon di /etc/passwd)

awk -F':' '{print $1, $3}' /etc/passwd

  

# Filter baris dengan kondisi

awk '$9 == "404" {print $1, $7}' access.log

  

# Hitung total

awk '{sum += $10} END {print "Total bytes:", sum}' access.log

  

# Hitung jumlah baris

awk 'END {print NR}' auth.log

  

# Filter baris tertentu berdasarkan nomor

awk 'NR==5, NR==10 {print}' log.txt

  

# Cari baris yang field ke-3 lebih dari 1000

awk '$3 > 1000 {print}' data.txt

  

# Tampilkan field terakhir setiap baris

awk '{print $NF}' log.txt

  

# Ganti separator output menjadi koma

awk -F':' 'OFS="," {print $1,$3,$6}' /etc/passwd

  

# Gunakan regex sebagai kondisi

awk '/Failed/ {print $NR, $0}' auth.log

  

# Hitung kemunculan per value (IP counter)

awk '{count[$1]++} END {for(ip in count) print count[ip], ip}' access.log

```

  

### Contoh Output Nyata

  

**Input: `/etc/passwd` (cuplikan)**

```

root:x:0:0:root:/root:/bin/bash

daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

attacker:x:1337:1337::/home/attacker:/bin/bash

```

  

```bash

awk -F':' '$3 >= 1000 {print "User:", $1, "| UID:", $3, "| Shell:", $7}' /etc/passwd

```

```

User: attacker | UID: 1337 | Shell: /bin/bash

```

  

```bash

awk -F':' 'END {print "Total user:", NR}' /etc/passwd

```

```

Total user: 3

```

  

> [!example] **Praktek Langsung**

> ```bash

> # Simulasi access.log

> echo -e "10.10.10.1 - - [02/Jun] \"GET /login\" 200 512\n10.10.10.2 - - [02/Jun] \"POST /login\" 401 128\n10.10.10.1 - - [02/Jun] \"GET /flag\" 404 64" > access.log

>

> # Ambil IP dan status code

> awk '{print $1, $9}' access.log

>

> # Filter hanya status 4xx

> awk '$9 >= 400 {print "SUSPICIOUS:", $1, $9, $7}' access.log

> ```

  

---

  

## 🔍 FIND — Pemburu File Forensik

  

### Konsep

`find` = traversal filesystem secara rekursif dengan filter kondisi. Penting untuk **artifact hunting** dan **malware detection**.

  

```

find [PATH] [OPTIONS] [EXPRESSION]

```

  

### Cheatsheet Lengkap

  

```bash

# Cari berdasarkan nama (exact)

find /var/log -name "auth.log"

  

# Cari dengan wildcard

find /tmp -name "*.php"

  

# Cari berdasarkan tipe (f=file, d=direktori, l=symlink)

find /home -type f

find /etc -type d

  

# Cari berdasarkan ukuran

find / -size +10M          # lebih dari 10MB

find /tmp -size -1k         # kurang dari 1KB

find / -size +5M -size -50M # antara 5MB-50MB

  

# Cari berdasarkan waktu modifikasi

find / -mtime -1      # dimodifikasi dalam 24 jam terakhir

find / -mtime +30     # dimodifikasi lebih dari 30 hari lalu

find / -newer /etc/passwd  # lebih baru dari /etc/passwd

  

# Cari berdasarkan permission

find / -perm 777       # exactly 777

find / -perm -4000     # setuid bit aktif (BAHAYA!)

find / -perm -2000     # setgid bit aktif

  

# Cari berdasarkan owner

find /home -user attacker

find /tmp -group www-data

  

# Exclude direktori tertentu

find / -path /proc -prune -o -name "*.sh" -print

  

# Jalankan command pada hasil find

find /tmp -name "*.sh" -exec file {} \;

find /var/log -name "*.log" -exec grep -l "CRITICAL" {} \;

  

# Kombinasi kondisi (AND default, OR dengan -o)

find /tmp -name "*.php" -size +100k

find /tmp \( -name "*.php" -o -name "*.sh" \) -type f

```

  

### Contoh Output Nyata

  

```bash

# Cari file PHP mencurigakan di /var/www

find /var/www -name "*.php" -newer /var/www/index.php -type f

```

```

/var/www/html/uploads/c99shell.php

/var/www/html/tmp/.hidden_backdoor.php

```

  

```bash

# Cari file dengan SUID bit (privilege escalation vector!)

find / -perm -4000 -type f 2>/dev/null

```

```

/usr/bin/sudo

/usr/bin/passwd

/usr/bin/python3.9     ← SUSPICIOUS!

```

  

> [!example] **Praktek Langsung**

> ```bash

> # Buat beberapa file untuk latihan

> mkdir -p /tmp/latihan/{normal,suspicious}

> touch /tmp/latihan/normal/readme.txt

> echo "<?php system(\$_GET['cmd']); ?>" > /tmp/latihan/suspicious/shell.php

>

> # Temukan semua file .php

> find /tmp/latihan -name "*.php" -type f

>

> # Tampilkan isinya langsung

> find /tmp/latihan -name "*.php" -exec cat {} \;

> ```

  

---

  

## 🔗 PIPE (`|`) — Rantai Command

  

### Konsep

Pipe mengalirkan **stdout** dari command kiri → **stdin** ke command kanan. Ini inti dari "one-liner" yang kuat.

  

```

command1 | command2 | command3 | ...

```

  

### Pola Pipe yang Paling Sering Dipakai di CTF

  

```bash

# Pattern 1: Grep → Sort → Uniq (frekuensi)

grep "Failed" auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

  

# Pattern 2: Find → Grep (cari isi dalam banyak file)

find /var/log -name "*.log" | xargs grep -l "backdoor"

  

# Pattern 3: Cat → Grep → Awk (extract field dari log terfilter)

cat access.log | grep "404" | awk '{print $1}' | sort | uniq -c

  

# Pattern 4: Find → Exec awk (proses setiap file)

find /tmp -name "*.csv" | xargs awk -F',' '{print $1}'

  

# Pattern 5: Grep → Wc (hitung)

grep "CRITICAL" syslog | wc -l

  

# Pattern 6: Multi-filter berantai

cat auth.log | grep "Failed" | grep -v "invalid" | awk '{print $11}' | sort -u

```

  

### Contoh Workflow Lengkap

  

```bash

# Skenario: "Temukan IP yang login gagal lebih dari 5 kali"

cat auth.log \

  | grep "Failed password" \

  | awk '{print $11}' \

  | sort \

  | uniq -c \

  | sort -rn \

  | awk '$1 > 5 {print "BRUTE FORCE:", $2, "| Count:", $1}'

```

```

BRUTE FORCE: 10.10.10.5 | Count: 47

BRUTE FORCE: 192.168.1.100 | Count: 12

```

  

---

  

## 📤 REDIRECT (`>`, `>>`, `2>`) — Kontrol Output

  

### Konsep

Setiap proses Linux punya 3 stream: **stdin (0)**, **stdout (1)**, **stderr (2)**. Redirect = alihkan stream ke file atau tempat lain.

  

### Cheatsheet Lengkap

  

| Operator | Fungsi | Contoh |

|----------|--------|--------|

| `>` | Simpan stdout ke file (overwrite) | `grep "error" app.log > errors.txt` |

| `>>` | Append stdout ke file | `echo "timestamp" >> report.txt` |

| `<` | Baca stdin dari file | `awk -F',' '{print $1}' < data.csv` |

| `2>` | Simpan stderr ke file | `find / -name "flag" 2> /dev/null` |

| `2>&1` | Gabung stderr ke stdout | `command > output.txt 2>&1` |

| `/dev/null` | Buang output (silent) | `find / -perm -4000 2>/dev/null` |

| `tee` | Tampilkan DAN simpan ke file | `grep "error" log \| tee errors.txt` |

| `&>` | Redirect stdout+stderr sekaligus | `command &> all_output.txt` |

  

### Contoh Nyata

  

```bash

# Simpan semua IP yang brute force ke file untuk laporan

grep "Failed" auth.log | awk '{print $11}' | sort | uniq -c | sort -rn > bruteforce_ips.txt

cat bruteforce_ips.txt

```

```

47 10.10.10.5

12 192.168.1.100

 3 172.16.0.50

```

  

```bash

# Gabungkan stdout dan stderr saat mencari

find / -name "*.php" -newer /var/www/index.php 2>/dev/null | tee suspicious_php.txt

```

  

```bash

# Append temuan ke laporan yang sama

echo "=== Login Failures ===" >> report.txt

grep -c "Failed" auth.log >> report.txt

echo "=== Suspicious IPs ===" >> report.txt

grep "Failed" auth.log | awk '{print $11}' | sort -u >> report.txt

```

  

> [!example] **Praktek Langsung**

> ```bash

> # Buat laporan otomatis

> echo "LAPORAN INVESTIGASI - $(date)" > laporan.txt

> echo "---" >> laporan.txt

> echo "File diperiksa: $(find /var/log -name "*.log" 2>/dev/null | wc -l) file" >> laporan.txt

> cat laporan.txt

> ```

  

---

  

## 🗺️ WORKFLOW — Langkah Kerja Saat Dapat Soal Log Forensics

  

```

┌─────────────────────────────────────────────────────┐

│              TERIMA SOAL + FILE LOG                 │

└─────────────────────┬───────────────────────────────┘

                      │

                      ▼

┌─────────────────────────────────────────────────────┐

│  STEP 1: RECONNAISSANCE FILE                        │

│  file log.zip → unzip → ls -lah                     │

│  wc -l *.log  (berapa baris?)                       │

│  head -20 auth.log  (lihat struktur log)            │

└─────────────────────┬───────────────────────────────┘

                      │

                      ▼

┌─────────────────────────────────────────────────────┐

│  STEP 2: IDENTIFIKASI POLA MENCURIGAKAN             │

│  grep -i "fail\|error\|denied\|invalid" *.log       │

│  grep -c keyword log (hitung frekuensi)             │

└─────────────────────┬───────────────────────────────┘

                      │

                      ▼

┌─────────────────────────────────────────────────────┐

│  STEP 3: EKSTRAK DATA TERSTRUKTUR                   │

│  awk untuk ambil kolom IP, timestamp, user          │

│  sort + uniq -c untuk frekuensi                     │

└─────────────────────┬───────────────────────────────┘

                      │

                      ▼

┌─────────────────────────────────────────────────────┐

│  STEP 4: BURU ARTIFACT / EVIDENCE                   │

│  find untuk cari file terkait                       │

│  grep -r pattern pada direktori tertentu            │

└─────────────────────┬───────────────────────────────┘

                      │

                      ▼

┌─────────────────────────────────────────────────────┐

│  STEP 5: DOKUMENTASI & SUBMIT                       │

│  Redirect hasil ke .txt                             │

│  Copy flag ke CTFd                                  │

│  Tulis writeup singkat                              │

└─────────────────────────────────────────────────────┘

```

  

### Command Recon Standar (Jalankan ini pertama kali!)

  

```bash

# 1. Lihat semua file yang ada

ls -lah && find . -type f | head -20

  

# 2. Kenali struktur setiap log

head -5 *.log

  

# 3. Hitung skala data

wc -l *.log

  

# 4. Cari keyword mencurigakan sekaligus

grep -iE "fail|error|denied|backdoor|shell|cmd|exec|passwd|sudo|root" *.log | head -30

  

# 5. Temukan semua IP yang muncul

grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" *.log | sort | uniq -c | sort -rn | head -10

```

  

---

  

## 🏁 SOAL LATIHAN CTF-STYLE

  

> [!danger] Challenge: **"Who Did It?"**

> **Difficulty:** ⭐⭐ Easy-Medium

> **Kategori:** Log Forensics / Blue Team

  

### Skenario

  

Kamu mendapat file `auth.log` dari sebuah server yang dicurigai diserang. Tugasmu:

  

1. Temukan **IP address** yang melakukan brute force (login gagal terbanyak)

2. Temukan **username** yang berhasil dilogin secara ilegal

3. Tentukan **jam berapa** serangan brute force terjadi paling intens

4. Flag format: `FLAG{IP_ATTACKER:USERNAME:JAM}` (contoh: `FLAG{10.0.0.1:admin:03}`)

  

**Download file latihan (simulasi):**

  

```bash

# Buat file latihan sendiri

cat > auth.log << 'EOF'

Jun  2 02:13:11 server sshd[100]: Failed password for root from 192.168.10.99 port 22

Jun  2 02:13:13 server sshd[101]: Failed password for root from 192.168.10.99 port 22

Jun  2 02:13:15 server sshd[102]: Failed password for admin from 192.168.10.99 port 22

Jun  2 02:13:17 server sshd[103]: Failed password for admin from 192.168.10.99 port 22

Jun  2 02:13:19 server sshd[104]: Failed password for root from 192.168.10.99 port 22

Jun  2 02:13:21 server sshd[105]: Failed password for root from 192.168.10.99 port 22

Jun  2 03:45:02 server sshd[200]: Failed password for backup from 10.10.0.5 port 22

Jun  2 03:45:10 server sshd[201]: Accepted password for backup from 10.10.0.5 port 22

Jun  2 02:13:25 server sshd[106]: Failed password for root from 192.168.10.99 port 22

Jun  2 02:13:27 server sshd[107]: Failed password for admin from 192.168.10.99 port 22

Jun  2 02:13:29 server sshd[108]: Failed password for admin from 192.168.10.99 port 22

Jun  2 02:13:31 server sshd[109]: Accepted password for admin from 192.168.10.99 port 22

EOF

```

  

### Jawaban

  

> [!success] Solusi Lengkap

  

```bash

# === LANGKAH 1: IP Brute Force ===

grep "Failed" auth.log | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" \

  | sort | uniq -c | sort -rn

```

```

9 192.168.10.99   ← ATTACKER UTAMA

1 10.10.0.5

```

  

```bash

# === LANGKAH 2: Username yang berhasil dilogin ===

grep "Accepted" auth.log | awk '{print $9}'

```

```

backup

admin        ← ini yang mencurigakan (login dari IP attacker!)

```

  

```bash

# Verifikasi: siapa yang login berhasil dari IP attacker?

grep "Accepted" auth.log | grep "192.168.10.99" | awk '{print "User:", $9, "| IP:", $11}'

```

```

User: admin | IP: 192.168.10.99

```

  

```bash

# === LANGKAH 3: Jam paling intens ===

grep "Failed" auth.log | grep "192.168.10.99" | awk '{print $3}' \

  | cut -d: -f1 | sort | uniq -c | sort -rn

```

```

9 02    ← jam 02 paling banyak

```

  

```bash

# === FINAL FLAG ===

echo "FLAG{192.168.10.99:admin:02}"

```

```

FLAG{192.168.10.99:admin:02}

```

  

---

  

## ⚠️ 3 HAL YANG JANGAN SAMPAI LUPA

  

> [!warning] ❶ — `2>/dev/null` Saat Pakai `find`

> `find` di sistem nyata **selalu lempar error** "Permission denied" ke ratusan direktori sistem.

> Tanpa `2>/dev/null`, outputmu penuh sampah error dan kamu kehilangan hasil yang relevan.

> ```bash

> # SALAH (output kotor):

> find / -name "*.php" -newer /var/www/html

>

> # BENAR (bersih):

> find / -name "*.php" -newer /var/www/html 2>/dev/null

> ```

  

> [!warning] ❷ — `sort` Dulu Sebelum `uniq`

> `uniq` hanya menghapus duplikat **yang berurutan**. Kalau tidak di-`sort` dulu, IP yang sama tapi tersebar di log tidak akan tergabung.

> ```bash

> # SALAH (uniq tidak efektif):

> grep "Failed" auth.log | awk '{print $11}' | uniq -c

>

> # BENAR:

> grep "Failed" auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

> ```

  

> [!warning] ❸ — Perhatikan Struktur Log Sebelum `awk`

> Kolom `$1`, `$2`... `awk` **bergantung format log**. Auth.log berbeda strukturnya dengan Apache access.log dan syslog.

> Selalu `head -3 log.txt` dulu untuk tahu field mana yang perlu diambil.

> ```bash

> # Selalu kenali struktur log lebih dulu!

> head -3 auth.log | cat -A   # lihat karakter tersembunyi juga

> head -3 access.log | awk '{print NF, "fields:", $0}'

> ```

  

---

  

## 📎 Quick Reference One-Liner

  

```bash

# TOP 10 IP paling banyak akses

awk '{print $1}' access.log | sort | uniq -c | sort -rn | head 10

  

# Semua URL yang diakses user tertentu

grep "10.10.10.5" access.log | awk '{print $7}' | sort -u

  

# Semua user yang pernah sudo

grep "sudo" auth.log | awk '{print $6}' | sort -u

  

# File PHP yang dibuat dalam 24 jam terakhir

find /var/www -name "*.php" -mtime -1 2>/dev/null

  

# Rekap status HTTP

awk '{print $9}' access.log | sort | uniq -c | sort -rn

  

# Cari string di semua log sekaligus

grep -r "attacker_keyword" /var/log/ 2>/dev/null | head -20

  

# Extract semua email dari file

grep -oE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt | sort -u

  

# Hitung bytes total transferred

awk '{sum+=$10} END {print sum/1024/1024, "MB"}' access.log

```

  

---

  

*📅 Dibuat: 2026-06-02 | 🔄 Part 2: `sed`, `cut`, `sort`, `xargs`, `strings`*

*📌 #LKS2026 #BlueTeam #LinuxCLI #LogForensics*