---

title: Linux CLI Intensif — grep, awk, find, pipe, redirect

tags: [LKS2026, BlueTeam, Linux, CTF, CLI, Security]

date: 2026-06-02

cssclasses: [wide-page]

---

  

# 🐧 Linux CLI Intensif — Part 1

## `grep` · `awk` · `find` · `pipe` · `redirect`

  

> [!quote] Misi Malam Ini

> **Target:** Bisa filter log dengan `grep` + `awk` tanpa googling.

> Semua yang ada di sini — hafalkan pola, bukan syntax-nya.

  

---

  

## 📌 Kenapa Ini Penting di CTF / Blue Team?

  

> [!info] Konteks Real-World

> Di CTF kategori **Forensics**, **Log Analysis**, dan **Incident Response**, kamu akan dikasih file log raksasa (ribuan baris) dan harus menemukan:

> - IP penyerang dalam ribuan request

> - Timestamp anomali dari ratusan event

> - Flag tersembunyi di dalam output command

>

> **Tool GUI tidak akan cukup cepat.** Satu pipeline yang benar bisa selesai dalam detik.

  

| Tool | Fungsi Utama | Kapan Dipakai di CTF |

|------|-------------|----------------------|

| `grep` | Filter baris berdasarkan pola | Cari IP, keyword, error dalam log |

| `awk` | Ekstrak & manipulasi kolom | Ambil kolom tertentu, hitung frekuensi |

| `find` | Temukan file berdasarkan kriteria | Cari file tersembunyi, setuid, timestamp |

| `pipe \|` | Sambungkan output → input | Rangkai semua tool jadi satu alur |

| `redirect >` | Simpan output ke file | Simpan hasil analisis |

  

---

  

## ⚡ GREP — Filter Baris

  

> [!example] Konsep

> `grep` membaca setiap baris, dan **hanya menampilkan baris yang cocok** dengan pola yang kamu berikan.

> **Pola bisa berupa:** string biasa, regex, atau kombinasi flag.

  

### Syntax Dasar

```bash

grep [OPTIONS] "PATTERN" file

```

  

### Cheatsheet Lengkap

  

```bash

# ── DASAR ────────────────────────────────────────────

grep "error" access.log          # Cari kata "error"

grep -i "error" access.log       # Case-insensitive (Error, ERROR, error)

grep -v "200" access.log         # INVERT — tampilkan yang TIDAK cocok

grep -n "error" access.log       # Tampilkan nomor baris

grep -c "error" access.log       # Hitung jumlah baris yang cocok

  

# ── KONTEKS ──────────────────────────────────────────

grep -A 3 "FAILED" auth.log      # 3 baris SETELAH match

grep -B 3 "FAILED" auth.log      # 3 baris SEBELUM match

grep -C 3 "FAILED" auth.log      # 3 baris SEBELUM + SESUDAH

  

# ── REGEX ─────────────────────────────────────────────

grep -E "192\.168\.[0-9]+\.[0-9]+" access.log   # Extended regex

grep -P "\d{3}-\d{4}" file.txt                   # Perl regex (lebih powerful)

grep "^2024" access.log          # Baris yang DIMULAI dengan 2024

grep "\.php$" access.log         # Baris yang BERAKHIR dengan .php

  

# ── MULTIPLE PATTERN ──────────────────────────────────

grep -E "error|warning|critical" syslog         # OR — salah satu

grep "error" file | grep "login"                 # AND — keduanya harus ada

  

# ── REKURSIF ──────────────────────────────────────────

grep -r "password" /var/log/     # Cari di semua file dalam folder

grep -rl "password" /var/log/    # Tampilkan NAMA FILE saja (bukan isinya)

  

# ── OUTPUT CONTROL ────────────────────────────────────

grep -o "192\.[0-9.]*" access.log   # Tampilkan HANYA bagian yang cocok

grep --color=auto "error" log       # Highlight match (default di banyak distro)

```

  

### Contoh Output Nyata

  

```bash

$ grep -n "Failed password" /var/log/auth.log | head -5

142:Jun  1 02:14:33 server sshd[1234]: Failed password for root from 10.0.0.1

143:Jun  1 02:14:35 server sshd[1234]: Failed password for root from 10.0.0.1

144:Jun  1 02:14:37 server sshd[1234]: Failed password for admin from 10.0.0.1

161:Jun  1 02:18:01 server sshd[5678]: Failed password for root from 172.16.0.5

209:Jun  1 03:00:12 server sshd[9999]: Failed password for pi from 192.168.1.10

```

  

> [!tip] Latihan Cepat

> **Kamu punya file `web.log`.** Coba:

> ```bash

> grep -c "404" web.log             # Berapa kali 404?

> grep -o '"GET [^"]*"' web.log | sort | uniq -c | sort -rn | head -10

> # ^ Top 10 URL yang paling sering diakses

> ```

  

---

  

## ⚡ AWK — Ekstrak & Proses Kolom

  

> [!example] Konsep

> `awk` membaca file **per baris**, lalu memecah setiap baris menjadi **kolom** berdasarkan separator (default: spasi/tab).

> Kolom diakses dengan `$1`, `$2`, ..., `$NF` (kolom terakhir).

  

### Anatomy AWK

```

awk 'BEGIN{...} PATTERN{ACTION} END{...}' file

      │              │                │

      │              │                └─ Setelah semua baris diproses

      │              └─ Dijalankan untuk setiap baris

      └─ Dijalankan sekali di awal

```

  

### Cheatsheet Lengkap

  

```bash

# ── EKSTRAK KOLOM ─────────────────────────────────────

awk '{print $1}' access.log         # Cetak kolom pertama (IP)

awk '{print $1, $7}' access.log     # Cetak kolom 1 dan 7

awk '{print $NF}' access.log        # Cetak kolom TERAKHIR

awk '{print NR, $0}' file           # Tambahkan nomor baris

  

# ── CUSTOM SEPARATOR ──────────────────────────────────

awk -F':' '{print $1}' /etc/passwd  # Pakai ':' sebagai pemisah

awk -F',' '{print $2}' data.csv     # Parse CSV, ambil kolom 2

awk -F'\t' '{print $3}' data.tsv    # Tab-separated

  

# ── FILTER BARIS ──────────────────────────────────────

awk '$9 == "404"' access.log        # Hanya baris dengan kolom 9 = 404

awk '$9 >= 500' access.log          # Status code >= 500

awk 'NR > 10' file                  # Mulai dari baris ke-11

awk 'NR>=5 && NR<=10' file          # Baris 5 sampai 10

  

# ── PATTERN MATCHING ──────────────────────────────────

awk '/Failed password/' auth.log    # Mirip grep

awk '/192\.168/ {print $1}' log     # Baris yang mengandung 192.168, ambil $1

awk '!/^#/' config.conf             # Abaikan baris komentar

  

# ── KALKULASI ─────────────────────────────────────────

awk '{sum += $10} END {print sum}' access.log         # Total bytes

awk '{sum += $10} END {print sum/NR}' access.log      # Rata-rata bytes

awk 'END {print NR}' access.log                       # Hitung total baris

  

# ── FREKUENSI / COUNTING ──────────────────────────────

awk '{count[$1]++} END {for(ip in count) print count[ip], ip}' access.log

# ^ Hitung berapa kali setiap IP muncul

  

# ── FORMAT OUTPUT ─────────────────────────────────────

awk '{printf "IP: %-15s  Status: %s\n", $1, $9}' access.log

awk 'BEGIN{print "=== REPORT ==="} {print $1} END{print "=== DONE ==="}' log

```

  

### Contoh Output Nyata

  

```bash

# Input: access.log (Apache/Nginx format)

# 192.168.1.5 - - [01/Jun/2024:10:23:45 +0700] "GET /login.php HTTP/1.1" 200 1024

  

$ awk '{print $1, $9}' access.log | head -5

192.168.1.5 200

192.168.1.5 302

10.0.0.1 404

172.16.0.2 500

192.168.1.5 200

  

# Hitung IP paling sering:

$ awk '{count[$1]++} END {for(ip in count) print count[ip], ip}' access.log | sort -rn | head -3

847 192.168.1.5

312 10.0.0.1

89  172.16.0.2

```

  

> [!tip] Latihan Cepat

> **File `access.log` format Apache:**

> ```bash

> # Tampilkan semua request yang status-nya 404, ambil IP dan URL-nya saja

> awk '$9 == "404" {print $1, $7}' access.log

>

> # Hitung total bytes yang dikirim

> awk '{total += $10} END {print "Total bytes:", total}' access.log

> ```

  

---

  

## ⚡ FIND — Temukan File

  

> [!example] Konsep

> `find` menelusuri direktori secara rekursif untuk mencari file berdasarkan **nama, tipe, ukuran, izin, waktu modifikasi**, dan banyak lagi.

  

### Cheatsheet Lengkap

  

```bash

# ── DASAR ────────────────────────────────────────────

find /path -name "*.log"             # Cari semua file .log

find /path -name "flag*"             # File yang namanya mulai "flag"

find /path -iname "*.PHP"            # Case-insensitive

  

# ── TIPE FILE ─────────────────────────────────────────

find / -type f -name "*.conf"        # f = regular file

find / -type d -name "secret"        # d = directory

find / -type l                       # l = symbolic link

  

# ── PERMISSION (PENTING UNTUK CTF!) ───────────────────

find / -perm -4000 2>/dev/null       # SUID files — privilege escalation!

find / -perm -2000 2>/dev/null       # SGID files

find / -perm -o+w 2>/dev/null        # World-writable files

find / -perm 777 2>/dev/null         # Full permission

  

# ── WAKTU ─────────────────────────────────────────────

find /var/log -mmin -60              # Dimodifikasi dalam 60 menit terakhir

find /tmp -mtime -1                  # Dimodifikasi dalam 1 hari terakhir

find / -newer /etc/passwd            # Lebih baru dari /etc/passwd

  

# ── UKURAN ────────────────────────────────────────────

find / -size +10M                    # Lebih besar dari 10MB

find / -size -1k                     # Lebih kecil dari 1KB

find / -empty                        # File/folder kosong

  

# ── USER/GROUP ────────────────────────────────────────

find / -user root                    # File milik root

find / -group www-data               # File milik grup www-data

find / -nouser 2>/dev/null           # File tanpa owner (suspicious!)

  

# ── EKSEKUSI SETELAH FIND ──────────────────────────────

find /tmp -name "*.sh" -exec cat {} \;     # Baca semua .sh di /tmp

find / -perm -4000 -exec ls -la {} \;     # List semua SUID files

find . -name "*.log" -exec grep -l "error" {} \;  # Log yang ada "error"

```

  

### Contoh Output Nyata

  

```bash

$ find / -perm -4000 -type f 2>/dev/null

/usr/bin/passwd

/usr/bin/sudo

/usr/bin/pkexec

/usr/lib/openssh/ssh-keysign

/tmp/.hidden_backdoor    # ← INI MENCURIGAKAN!

  

$ find /var/log -mmin -30 -type f

/var/log/auth.log

/var/log/syslog

/var/log/apache2/access.log   # ← Log yang baru diupdate

```

  

> [!tip] Latihan Cepat

> ```bash

> # Cari semua file yang bisa dieksekusi oleh siapa saja di /tmp

> find /tmp -perm -111 -type f

>

> # Cari file yang dimodifikasi dalam 1 jam terakhir

> find /var -mmin -60 -type f 2>/dev/null

> ```

  

---

  

## ⚡ PIPE `|` — Sambungkan Segalanya

  

> [!example] Konsep

> `|` (pipe) mengambil **output** dari command kiri dan menjadikannya **input** untuk command kanan.

> Ini adalah kekuatan sejati Linux — rangkaikan tool sederhana jadi pipeline yang powerful.

  

```

command1 | command2 | command3 | ...

   OUTPUT──▶INPUT    OUTPUT──▶INPUT

```

  

### Kombinasi Killer untuk CTF

  

```bash

# ── SORT + UNIQ ───────────────────────────────────────

cat access.log | awk '{print $1}' | sort | uniq        # IP unik

cat access.log | awk '{print $1}' | sort | uniq -c     # + hitung frekuensi

cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn  # Urutkan terbanyak

  

# ── GREP + AWK ────────────────────────────────────────

grep "Failed" auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# ^ IP yang paling banyak gagal login

  

grep "404" access.log | awk '{print $7}' | sort | uniq -c | sort -rn

# ^ URL yang paling sering 404

  

# ── PIPELINE LENGKAP ──────────────────────────────────

cat auth.log \

  | grep "Failed password" \

  | awk '{print $11}' \

  | sort \

  | uniq -c \

  | sort -rn \

  | head -10

# ^ Top 10 IP yang melakukan brute force SSH

  

# ── CUT (ALTERNATIF AWK SIMPEL) ───────────────────────

cat /etc/passwd | cut -d':' -f1        # Ambil kolom 1, separator ':'

cat access.log | cut -d'"' -f2         # Ambil request line

  

# ── WC (HITUNG) ───────────────────────────────────────

cat access.log | grep "POST" | wc -l   # Berapa banyak POST request?

  

# ── TEE (SIMPAN + TERUS) ──────────────────────────────

grep "error" syslog | tee errors.txt | wc -l

# ^ Simpan ke errors.txt DAN tampilkan hitungannya sekaligus

```

  

---

  

## ⚡ REDIRECT `>` `>>` `<` — Kontrol Aliran Data

  

> [!example] Konsep

> Redirect mengalihkan **stdout** (output normal) atau **stderr** (error) ke file atau sebaliknya.

  

```bash

# ── OUTPUT REDIRECT ───────────────────────────────────

command > output.txt       # Tulis ke file (OVERWRITE — hati-hati!)

command >> output.txt      # Append ke file (tambahkan di akhir)

command 2> error.txt       # Redirect STDERR ke file

command 2>/dev/null        # Buang semua error (tidak ditampilkan)

command > out.txt 2>&1     # Gabungkan stdout + stderr ke satu file

command &> all.txt         # Shorthand: gabungkan stdout + stderr

  

# ── INPUT REDIRECT ────────────────────────────────────

command < input.txt        # Baca input dari file

  

# ── HEREDOC ───────────────────────────────────────────

cat << EOF > config.txt

line1

line2

EOF

  

# ── CONTOH PRAKTIS ────────────────────────────────────

# Simpan hasil analisis

grep "Failed" auth.log | awk '{print $11}' | sort | uniq -c \

  | sort -rn > attacker_ips.txt

  

# Jalankan find, buang error "permission denied"

find / -name "flag*" 2>/dev/null > found_flags.txt

  

# Log output DAN error terpisah

nmap -sV 192.168.1.1 > nmap_result.txt 2> nmap_errors.txt

```

  

> [!warning] Jebakan Umum

> `>` akan **MENGHAPUS** isi file yang sudah ada!

> Gunakan `>>` kalau mau menambahkan data.

> ```bash

> echo "result1" > hasil.txt    # Isi: result1

> echo "result2" > hasil.txt    # Isi: result2 (result1 HILANG!)

> echo "result3" >> hasil.txt   # Isi: result2\nresult3 ✓

> ```

  

---

  

## 🔄 WORKFLOW — Saat Dapat Soal Log Analysis

  

> [!success] Langkah Kerja Sistematis

  

```

SOAL DATANG

    │

    ▼

┌─────────────────────────────────────┐

│ STEP 1: RECON FILE                  │

│  wc -l file.log    → berapa baris?  │

│  head -5 file.log  → format log?    │

│  ls -lh file.log   → ukuran file?   │

└────────────────┬────────────────────┘

                 │

                 ▼

┌─────────────────────────────────────┐

│ STEP 2: IDENTIFIKASI KOLOM          │

│  head -1 file.log  → lihat struktur │

│  Kolom apa? Separator apa?          │

│  Tentukan: grep dulu atau awk dulu? │

└────────────────┬────────────────────┘

                 │

                 ▼

┌─────────────────────────────────────┐

│ STEP 3: FILTER KASAR (grep)         │

│  grep "keyword" file.log            │

│  Sempitkan dulu jangkauan pencarian │

└────────────────┬────────────────────┘

                 │

                 ▼

┌─────────────────────────────────────┐

│ STEP 4: EKSTRAK DATA (awk/cut)      │

│  Ambil kolom yang relevan           │

│  Kombinasikan dengan pipe           │

└────────────────┬────────────────────┘

                 │

                 ▼

┌─────────────────────────────────────┐

│ STEP 5: ANALISIS (sort/uniq/wc)     │

│  sort | uniq -c | sort -rn          │

│  Temukan anomali / frekuensi        │

└────────────────┬────────────────────┘

                 │

                 ▼

┌─────────────────────────────────────┐

│ STEP 6: SIMPAN HASIL                │

│  > hasil.txt                        │

│  Screenshot / catat flagnya         │

└─────────────────────────────────────┘

```

  

### Template Pipeline Siap Pakai

  

```bash

# === TEMPLATE 1: Cari Penyerang ===

grep "KEYWORD_ATTACK" logfile \

  | awk '{print $KOLOM_IP}' \

  | sort | uniq -c | sort -rn \

  | head -20

  

# === TEMPLATE 2: Cari Anomali Waktu ===

grep "TANGGAL_SPESIFIK" logfile \

  | awk '{print $KOLOM_WAKTU, $KOLOM_IP, $KOLOM_ACTION}'

  

# === TEMPLATE 3: Cari File Mencurigakan ===

find / -perm -4000 -o -newer /etc/passwd -type f 2>/dev/null \

  | grep -v "^/proc"

  

# === TEMPLATE 4: Ekstrak Flag dari Log ===

grep -oP 'CTF\{[^}]+\}' logfile

# atau

grep -E 'flag\{.*\}' logfile

```

  

---

  

## 🏆 SOAL LATIHAN CTF-STYLE

  

> [!danger] SOAL — Blue Team Log Analysis

  

**Skenario:**

Kamu adalah SOC Analyst. Server web `production.company.id` mengalami insiden. Kamu diberi file `access.log` (Apache) dengan 50.000+ baris.

  

**Format log:**

```

192.168.1.5 - admin [01/Jun/2024:10:23:45 +0700] "GET /dashboard HTTP/1.1" 200 4523

```

*(Kolom: IP, -, user, timestamp, request, status, bytes)*

  

**Pertanyaan:**

1. IP mana yang paling banyak melakukan request dengan status `403`?

2. Berapa total request `POST` yang terjadi pada tanggal `01/Jun/2024`?

3. URL endpoint apa yang paling sering muncul di request `404`?

4. Apakah ada request yang mengandung pola **SQL Injection** (`UNION SELECT`, `OR 1=1`, `--`)?

5. 🚩 Flag ada di dalam User-Agent salah satu request yang mencurigakan. Format: `CTF{...}`

  

---

  

> [!check] JAWABAN

  

```bash

# === Q1: IP paling banyak dengan status 403 ===

grep '"403"' access.log \

  | awk '{print $1}' \

  | sort | uniq -c | sort -rn | head -1

# Output: 847 10.10.10.55  → IP penyerang: 10.10.10.55

  

# === Q2: Total POST pada tanggal 01/Jun/2024 ===

grep '01/Jun/2024' access.log \

  | grep '"POST' \

  | wc -l

# Output: 1337

  

# === Q3: URL paling sering di 404 ===

awk '$9 == "404" {print $7}' access.log \

  | sort | uniq -c | sort -rn | head -5

# Output:

# 234 /wp-admin/

# 123 /admin/login.php

#  89 /.env

#  67 /config.php

#  45 /backup.zip

  

# === Q4: Deteksi SQL Injection ===

grep -iE "(UNION.SELECT|OR.1=1|--|'|%27)" access.log \

  | awk '{print $1, $7}' \

  | sort | uniq

  

# === Q5: Cari Flag di User-Agent ===

grep -oP 'CTF\{[^}]+\}' access.log

# atau jika log-nya format berbeda:

awk -F'"' '{print $6}' access.log | grep -oP 'CTF\{[^}]+\}'

# Output: CTF{log_4n4lyst_pr0}  ← 🚩 FLAG!

```

  

> [!tip] Insight Soal

> - Q1–Q3: Pure log analysis dengan grep + awk + sort + uniq

> - Q4: Pakai `-iE` untuk case-insensitive extended regex

> - Q5: `-oP` (only match + Perl regex) adalah senjata utama untuk extract flag

  

---

  

## ⚠️ 3 HAL YANG JANGAN SAMPAI LUPA

  

> [!danger] #1 — `2>/dev/null` saat pakai `find`

> ```bash

> # SALAH — output dibanjiri error "Permission denied"

> find / -name "flag*"

>

> # BENAR — error dibuang, output bersih

> find / -name "flag*" 2>/dev/null

> ```

> Tanpa ini, kamu tidak akan bisa baca hasilnya karena tertimbun error.

  

> [!danger] #2 — Urutan `sort | uniq -c | sort -rn`

> ```bash

> # SALAH — uniq hanya menghapus baris BERURUTAN yang sama

> cat ips.txt | uniq -c          # HASIL SALAH jika tidak disort dulu!

>

> # BENAR — selalu sort dulu sebelum uniq

> cat ips.txt | sort | uniq -c | sort -rn   # ← pola ini WAJIB dihafal

> ```

> Ini adalah **pipeline paling sering dipakai** di CTF log analysis.

  

> [!danger] #3 — Perbedaan `>` vs `>>`

> ```bash

> # > = OVERWRITE (menghapus isi lama)

> grep "error" log > hasil.txt    # Kalau hasil.txt sudah ada → TERHAPUS!

>

> # >> = APPEND (menambahkan di bawah)

> grep "error" log >> hasil.txt   # Aman, menambahkan di akhir

>

> # Aturan: pakai > hanya untuk file BARU, >> untuk file yang sudah ada

> ```

  

---

  

## 📚 QUICK REFERENCE CARD

  

```

┌──────────────────────────────────────────────────────────────┐

│                    GREP FLAGS                                 │

├─────────┬────────────────────────────────────────────────────┤

│   -i    │ case insensitive                                    │

│   -v    │ invert match (yang TIDAK cocok)                     │

│   -n    │ tampilkan nomor baris                               │

│   -c    │ hitung baris yang cocok                             │

│   -o    │ tampilkan HANYA bagian yang cocok                   │

│   -r    │ rekursif ke subdirektori                           │

│   -l    │ tampilkan nama file saja                            │

│   -E    │ extended regex                                      │

│   -P    │ Perl regex (paling powerful)                       │

│  -A N   │ N baris setelah match                              │

│  -B N   │ N baris sebelum match                              │

│  -C N   │ N baris sebelum + sesudah                          │

└─────────┴────────────────────────────────────────────────────┘

  

┌──────────────────────────────────────────────────────────────┐

│                    AWK VARIABLES                              │

├─────────┬────────────────────────────────────────────────────┤

│  $0     │ seluruh baris                                      │

│  $1..N  │ kolom ke-N                                         │

│  $NF    │ kolom terakhir                                      │

│  NR     │ nomor baris saat ini                               │

│  NF     │ jumlah kolom di baris ini                          │

│  FS     │ field separator (default: spasi)                   │

│  OFS    │ output field separator                             │

└─────────┴────────────────────────────────────────────────────┘

  

┌──────────────────────────────────────────────────────────────┐

│                    FIND FLAGS PENTING                         │

├──────────────┬───────────────────────────────────────────────┤

│  -type f/d/l │ file / direktori / symlink                   │

│  -name "X"   │ nama file (case sensitive)                   │

│  -iname "X"  │ nama file (case insensitive)                 │

│  -perm -4000 │ SUID bit (privilege escalation!)             │

│  -mmin -N    │ dimodifikasi < N menit lalu                  │

│  -mtime -N   │ dimodifikasi < N hari lalu                   │

│  -size +NM   │ lebih besar dari N megabyte                  │

│  -user NAME  │ dimiliki oleh user NAME                      │

│  -exec CMD   │ jalankan CMD untuk setiap hasil              │

└──────────────┴───────────────────────────────────────────────┘

```

  

---

  

## 🔗 Kombinasi Command Lainnya yang Sering Muncul

  

```bash

# strings — ekstrak string readable dari binary

strings binary_file | grep -i flag

  

# xxd / hexdump — lihat isi hex

xxd suspicious.bin | grep -A2 "4354 46"   # CTF dalam hex

  

# base64 decode dari log

grep "base64" access.log | awk '{print $NF}' | base64 -d

  

# Cari kredensial tersembunyi

grep -rE "(password|passwd|pwd|secret|token|key)\s*[=:]" /var/www/ 2>/dev/null

  

# Analisis timestamps

awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -rn

# ^ Jam berapa paling banyak request? (deteksi time-based attack)

```

  

---

  

*📅 Dibuat: 2026-06-02 | LKS 2026 Blue Team Preparation*

*🎯 Next: Part 2 — `sed`, `tr`, `netstat`, `ss`, `ps`, Process & Network Forensics*

  

---

#LKS2026 #BlueTeam #Linux #CTF #grep #awk #find #LogAnalysis