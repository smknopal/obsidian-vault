=f# Modul 1: Fondasi Digital Forensik untuk Blue Team

## Lomba Kompetensi Siswa (LKS) Nasional XXXIV – 2026

### Bidang: Cyber Security | Format: Defensive CTF Jeopardy Style

  

---

  

> **📋 Metadata Modul**

> | Atribut | Detail |

> |---|---|

> | **Lomba** | LKS Nasional XXXIV Tahun 2026 |

> | **Bidang** | Cyber Security (Keamanan Siber) |

> | **Tingkat** | Nasional – Siswa SMK Seluruh Indonesia |

> | **Cakupan** | Elemen Digital Forensic dasar (kisi-kisi resmi LKS 2026) |

> | **Durasi Estimasi** | 90–120 menit |

> | **Catatan** | ✅ Boleh bawa catatan digital offline |

> | **Bobot** | ~25–30% dari skor Blue Team |

  

---

  

## 📑 Daftar Isi

  

1. [Bab 1: OS Forensic (Windows & Linux)](#bab-1-os-forensic-windows--linux)

2. [Bab 2: File Carving](#bab-2-file-carving)

3. [Bab 3: Steganography & Analisis Metadata](#bab-3-steganography--analisis-metadata)

4. [Bab 4: Network Forensic (PCAP/PCAPNG)](#bab-4-network-forensic-pcappcapng)

5. [Bab 5: Log Forensic](#bab-5-log-forensic)

6. [Bab 6: Toolchain CLI & Automasi Cepat](#bab-6-toolchain-cli--automasi-cepat)

7. [Bab 7: Simulasi Mini-CTF Modul 1](#bab-7-simulasi-mini-ctf-modul-1)

8. [Catatan Penting](#catatan-penting)

  

---

  

## Bab 1: OS Forensic (Windows & Linux)

  

### 🪟 Windows Forensic

  

#### Browser Forensic

- Ekstraksi **history**, **cookies**, dan **download** dari browser Chrome/Firefox

- Lokasi profil Chrome: `%LOCALAPPDATA%\Google\Chrome\User Data\Default\`

- File penting: `History` (SQLite), `Cookies` (SQLite), `Downloads` (dalam `History`)

  

```bash

# Baca SQLite database Chrome history

sqlite3 History "SELECT url, title, last_visit_time FROM urls ORDER BY last_visit_time DESC LIMIT 20;"

```

  

#### AppData & Third-Party App Forensic

- **Telegram**: `%APPDATA%\Telegram Desktop\tdata\`

- **Discord**: `%APPDATA%\discord\Local Storage\leveldb\`

- **Zoom**: `%APPDATA%\Zoom\logs\`

  

#### Digital Artifact Discovery

  

| Artefak | Lokasi | Kegunaan |

|---|---|---|

| `NTUSER.DAT` | `C:\Users\<user>\` | Registry hive per-user (MRU, RecentDocs) |

| `SOFTWARE` | `C:\Windows\System32\config\` | Program yang terinstall, OS info |

| **Prefetch** | `C:\Windows\Prefetch\*.pf` | Program yang pernah dijalankan |

| **Jump Lists** | `%APPDATA%\Microsoft\Windows\Recent\` | File & folder yang baru dibuka |

| **Recycle Bin** | `C:\$Recycle.Bin\` | File yang dihapus + metadata `$I` |

  

```bash

# Parse Prefetch dengan strings (quick & dirty)

strings C:\Windows\Prefetch\NOTEPAD.EXE-XXXXXXXX.pf | grep -i ".txt"

  

# Lihat isi Recycle Bin (Linux mount Windows disk)

ls -la /mnt/windows/\$Recycle.Bin/S-1-5-21-*/

```

  

---

  

### 🐧 Linux Forensic

  

#### File & Direktori Kritis

  

| File/Direktori | Kegunaan Forensik |

|---|---|

| `~/.bash_history` | Riwayat perintah user |

| `/var/spool/cron/crontabs/<user>` | Scheduled tasks per-user |

| `/etc/crontab` | Crontab sistem |

| `/etc/passwd` | Daftar user & shell default |

| `/etc/shadow` | Password hash (butuh root) |

| `/tmp/` | File sementara (sering dipakai malware) |

  

```bash

# Lihat bash history user tertentu

cat /home/targetuser/.bash_history

  

# Cek crontab tersembunyi

cat /var/spool/cron/crontabs/*

  

# Tampilkan user dengan UID 0 (root backdoor)

awk -F: '($3 == "0") {print}' /etc/passwd

  

# Cek file di /tmp yang dibuat dalam 24 jam terakhir

find /tmp -mtime -1 -type f -ls

```

  

#### Analisis Log

  

```bash

# SSH login gagal (brute force detection)

grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

  

# SSH login berhasil

grep "Accepted password" /var/log/auth.log

  

# Syslog: lihat aktivitas mencurigakan

grep -i "error\|warning\|fail" /var/log/syslog | tail -50

```

  

#### Timestamp Analysis

  

```bash

# Temukan file yang dimodifikasi dalam 7 hari terakhir

find / -mtime -7 -type f 2>/dev/null | grep -v proc

  

# Bandingkan MAC times (Modify, Access, Change)

stat /path/to/suspicious_file

  

# Timeline sederhana

find /var/log -type f -exec ls -la {} \; | sort -k6,7

```

  

---

  

## Bab 2: File Carving

  

### 🔪 Konsep Dasar

  

File carving adalah teknik mengekstrak file dari media mentah (disk image) berdasarkan **magic bytes** (file signature), tanpa bergantung pada filesystem.

  

### Magic Bytes Penting

  

| Format | Magic Bytes (Hex) | Ekstensi |

|---|---|---|

| JPEG | `FF D8 FF` | `.jpg`, `.jpeg` |

| PNG | `89 50 4E 47 0D 0A 1A 0A` | `.png` |

| PDF | `25 50 44 46` (`%PDF`) | `.pdf` |

| ZIP | `50 4B 03 04` | `.zip` |

| ELF | `7F 45 4C 46` | (Linux binary) |

| PE (Windows EXE) | `4D 5A` (`MZ`) | `.exe`, `.dll` |

| GIF | `47 49 46 38` | `.gif` |

  

```bash

# Verifikasi magic bytes file

xxd file.jpg | head -3

file suspicious_file          # deteksi tipe otomatis

```

  

### Tools File Carving

  

#### `binwalk` — Analisis & Ekstraksi Firmware/Disk

  

```bash

# Scan file untuk embedded content

binwalk disk.img

  

# Ekstrak semua file yang ditemukan

binwalk -e disk.img

# Output masuk ke direktori: _disk.img.extracted/

  

# Ekstrak dengan verbose

binwalk -e -v disk.img

  

# Cari hanya format tertentu

binwalk --dd='png:png' disk.img

```

  

#### `foremost` — File Carving Klasik

  

```bash

# Carving dengan konfigurasi default

foremost -i disk.img -o output_dir/

  

# Hanya carve format tertentu

foremost -t jpg,png,pdf -i disk.img -o output_dir/

  

# Audit log hasil carving

cat output_dir/audit.txt

```

  

#### `photorec` — Carving untuk Foto & Dokumen

  

```bash

# Mode CLI

photorec disk.img

  

# Jalankan dengan pilihan non-interaktif (untuk skrip)

photorec /d output_dir/ /cmd disk.img partition_i_whole,fileopt,jpg,enable,search

```

  

### Workflow File Carving

  

```

disk.img / .raw / .dd

    │

    ├── binwalk -e      → _disk.img.extracted/

    │                        ├── 0.jpg

    │                        ├── 1234.png

    │                        └── ABCD.zip

    │

    ├── foremost -t all → output_dir/

    │                        ├── jpg/

    │                        ├── png/

    │                        └── pdf/

    │

    └── strings + grep  → flag{...} langsung dari raw

```

  

---

  

## Bab 3: Steganography & Analisis Metadata

  

### 🕵️ Konsep Dasar

  

Steganografi menyembunyikan data di dalam file media (gambar, audio) tanpa terlihat secara kasat mata. Dalam CTF, flag sering disembunyikan menggunakan teknik LSB atau tools seperti `steghide`.

  

### Tools Utama

  

#### `exiftool` — Baca/Tulis Metadata

  

```bash

# Baca semua metadata

exiftool image.jpg

  

# Baca metadata spesifik

exiftool -Comment -Author -GPS:all image.jpg

  

# Hapus semua metadata

exiftool -all= image.jpg

  

# Cari semua file JPG dan tampilkan komentar

exiftool -Comment *.jpg

  

# Ekspor metadata ke format JSON

exiftool -j image.jpg

```

  

**🎯 Tips CTF:** Flag sering disembunyikan di field `Comment`, `UserComment`, `Description`, atau `Author`.

  

#### `steghide` — Sembunyikan/Ekstrak Data di JPEG/BMP/WAV

  

```bash

# Ekstrak data (dengan passphrase)

steghide extract -sf secret.jpg -p "password123"

  

# Ekstrak data (tanpa passphrase / passphrase kosong)

steghide extract -sf secret.jpg -p ""

  

# Info tentang file (tanpa passphrase)

steghide info secret.jpg

  

# Sembunyikan file ke dalam gambar

steghide embed -cf cover.jpg -sf flag.txt -p "mypassword"

```

  

**🎯 Workflow CTF:**

1. Cek `.bash_history` atau file lain untuk passphrase

2. Gunakan passphrase tersebut di `steghide extract`

  

#### `zsteg` — Deteksi LSB di PNG/BMP

  

```bash

# Scan semua channel LSB

zsteg image.png

  

# Scan channel tertentu (LSB merah)

zsteg -c 1 image.png

  

# Verbose mode

zsteg -v image.png

  

# Ekstrak data dari channel spesifik

zsteg -E "b1,rgb,lsb,xy" image.png > output.bin

```

  

#### `stegsolve` (GUI) — Visual Analysis

  

```bash

# Jalankan via Java

java -jar stegsolve.jar

```

  

Gunakan untuk: bit plane analysis, color channel isolation, frame analysis (GIF).

  

### Teknik LSB (Least Significant Bit)

  

```

Pixel original:  10110011

Pixel + flag:    10110010  ← bit terakhir diubah untuk menyimpan data

```

  

Data tersembunyi di bit paling kecil → tidak terlihat secara visual, tapi bisa diekstrak dengan tools.

  

### Workflow Steganografi CTF

  

```

File mencurigakan (secret.jpg / data.png)

    │

    ├── exiftool secret.jpg          → Cek metadata (Comment, GPS, Author)

    │

    ├── strings secret.jpg           → Cari teks tersembunyi

    │

    ├── steghide info secret.jpg     → Cek apakah ada embedded data

    │

    ├── zsteg secret.png             → Cek LSB (khusus PNG/BMP)

    │

    └── steghide extract -sf secret.jpg -p <passphrase>

              │

              └── Passphrase dari: .bash_history / metadata / soal

```

  

---

  

## Bab 4: Network Forensic (PCAP/PCAPNG)

  

### 🌐 Konsep Dasar

  

Analisis traffic jaringan dari file `.pcap` atau `.pcapng` untuk menemukan credential, flag, atau bukti serangan.

  

### Wireshark — Analisis Visual

  

#### Filter Display Penting

  

```

# Filter protokol dasar

http

dns

tcp

icmp

ftp

smtp

  

# Filter berdasarkan IP

ip.src == 192.168.1.100

ip.dst == 10.0.0.1

  

# Filter kombinasi

ip.src == 192.168.1.100 && http

  

# Follow TCP stream ke-3

tcp.stream eq 3

  

# Cari string dalam payload

frame contains "flag{"

frame contains "password"

  

# Filter HTTP method

http.request.method == "POST"

  

# Filter DNS query

dns.qry.name contains "tunnel"

  

# Filter ICMP data (cek DNS/ICMP tunneling)

icmp && data.len > 100

```

  

#### Rekonstruksi File dari HTTP

  

```

Wireshark → File → Export Objects → HTTP

```

  

Pilih file yang ingin diunduh ulang (gambar, dokumen, binary) dari sesi HTTP.

  

### tshark — Analisis CLI

  

```bash

# Daftar semua konverasi HTTP dengan host

tshark -r capture.pcap -Y "http" -T fields -e http.host -e http.request.uri

  

# Cari POST request (kemungkinan ada credential)

tshark -r capture.pcap -Y "http.request.method == POST" -T fields \

  -e http.host -e http.request.uri -e http.file_data

  

# Ekstrak payload DNS (deteksi tunneling)

tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name

  

# Follow TCP stream via CLI

tshark -r capture.pcap -q -z follow,tcp,ascii,0

  

# Daftar semua IP yang berkomunikasi

tshark -r capture.pcap -T fields -e ip.src -e ip.dst | sort | uniq

  

# Simpan output ke file

tshark -r capture.pcap -Y "http" -T json > http_traffic.json

```

  

### Identifikasi Serangan di PCAP

  

#### DNS Tunneling

```bash

# Query DNS yang panjang & tidak wajar = tunneling

tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | \

  awk 'length($0) > 50'

```

  

#### ICMP Tunneling

```bash

# ICMP dengan payload besar

tshark -r capture.pcap -Y "icmp" -T fields -e data | grep -v "^$"

```

  

#### Credential dalam Payload

```bash

# Cari kata kunci umum dalam payload

tshark -r capture.pcap -Y "http" -T fields -e http.file_data | \

  strings | grep -iE "password|user|flag|secret|token"

```

  

---

  

## Bab 5: Log Forensic

  

### 📋 Analisis Log Web Server

  

#### Apache/Nginx Access Log Format

  

```

192.168.1.100 - - [22/Jun/2026:10:15:30 +0700] "GET /index.php?id=1 HTTP/1.1" 200 1024

   [IP]          [timestamp]                    [method path]              [status] [size]

```

  

#### Deteksi SQL Injection (SQLi)

  

```bash

# Cari pattern SQLi umum

grep -iE "union|select|from|where|--|'|1=1|or 1|sleep\(|benchmark" access.log

  

# Hitung IP yang paling banyak melakukan SQLi

grep -iE "union select|' OR '1'='1" access.log | \

  awk '{print $1}' | sort | uniq -c | sort -rn | head -10

  

# Cari URL-encoded SQLi

grep -E "%27|%20OR%20|%20UNION%20|%20SELECT%20" access.log

```

  

#### Deteksi Local File Inclusion (LFI)

  

```bash

# Pattern LFI

grep -E "\.\./|etc/passwd|etc/shadow|proc/self" access.log

  

# LFI dengan URL encoding

grep -E "%2e%2e%2f|%2e%2e\/|\.\.%2f" access.log

```

  

#### Deteksi Brute Force

  

```bash

# IP dengan banyak request POST ke /login (brute force)

grep "POST /login" access.log | awk '{print $1}' | sort | uniq -c | sort -rn

  

# IP dengan respons 401 (Unauthorized) berulang

awk '$9 == "401" {print $1}' access.log | sort | uniq -c | sort -rn | head -20

  

# Brute force SSH dari auth.log

grep "Failed password" /var/log/auth.log | \

  awk '{print $11}' | sort | uniq -c | sort -rn | head -20

```

  

#### Analisis Statistik Log

  

```bash

# Top 10 IP berdasarkan jumlah request

awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

  

# Distribusi HTTP status code

awk '{print $9}' access.log | sort | uniq -c | sort -rn

  

# Request per jam

awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c

  

# URL yang paling banyak diakses

awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -20

```

  

### 🔍 Splunk SPL Dasar (Jika Tersedia)

  

```spl

-- Semua event dari IP tertentu

source="access.log" src_ip="192.168.1.100"

  

-- Hitung request per IP

source="access.log" | stats count by src_ip | sort -count

  

-- Deteksi SQLi

source="access.log" | search uri="*union*" OR uri="*select*"

  

-- Login gagal per user

source="auth.log" "Failed password" | stats count by user, src_ip

  

-- Timeline serangan

source="access.log" status=500 | timechart count by src_ip

```

  

---

  

## Bab 6: Toolchain CLI & Automasi Cepat

  

### ⚡ Skrip Praktis untuk CTF

  

#### Cari Flag Langsung dari Disk Image

  

```bash

# Cari flag pattern di raw disk image

strings disk.img | grep -E "flag\{[^}]+\}" | sort -u

  

# Lebih luas: cari berbagai format flag CTF

strings disk.img | grep -E "(CTF|flag|FLAG)\{" | sort -u

  

# Jika flag di-encode base64

strings disk.img | grep -E "^[A-Za-z0-9+/]{20,}={0,2}$" | while read line; do

  decoded=$(echo "$line" | base64 -d 2>/dev/null)

  echo "$decoded" | grep -i "flag" && echo "Source: $line"

done

```

  

#### Workflow Binwalk + Search

  

```bash

# Ekstrak semua file dengan binwalk, lalu cari flag

binwalk -e disk.img

find _disk.img.extracted/ -type f -exec grep -l "CTF\|flag{" {} \;

  

# Cari di semua file hasil ekstraksi

find _disk.img.extracted/ -type f | while read f; do

  strings "$f" | grep -iE "flag\{|CTF\{" && echo "Found in: $f"

done

```

  

#### Analisis PCAP Cepat

  

```bash

# Daftar semua host HTTP dari PCAP

tshark -r capture.pcap -Y "http" -T fields -e http.host | sort -u

  

# Ekstrak semua string dari PCAP

tshark -r capture.pcap -T fields -e data | xxd -r -p | strings | grep "flag"

  

# Cari credential di HTTP POST

tshark -r capture.pcap -Y "http.request.method==POST" \

  -T fields -e http.file_data | strings

```

  

#### Log Analysis One-Liners

  

```bash

# IP dengan request terbanyak

awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -5

  

# Semua URL unik yang diakses

awk '{print $7}' access.log | sort -u

  

# Waktu serangan pertama

grep -iE "union|select|'|--" access.log | head -1 | awk '{print $4}'

  

# Payload SQLi terlengkap

grep -iE "union select" access.log | awk '{print $7}' | python3 -c "

import sys, urllib.parse

for line in sys.stdin:

    print(urllib.parse.unquote(line.strip()))

"

```

  

#### Ekstraksi Metadata Batch

  

```bash

# Metadata semua gambar di direktori

exiftool -r /path/to/images/ | grep -E "Comment|Author|Description|GPS"

  

# Simpan metadata ke CSV

exiftool -csv /path/to/images/*.jpg > metadata.csv

```

  

---

  

## Bab 7: Simulasi Mini-CTF Modul 1

  

### 🏆 3 Soal Terintegrasi

  

---

  

### Soal 1: File Carving + Steganografi

  

**Skenario:** Kamu mendapatkan `evidence.img` — disk image dari laptop tersangka. Diduga ada pesan rahasia yang disembunyikan dalam sebuah gambar.

  

**Langkah Penyelesaian:**

  

```bash

# Step 1: Scan disk image

binwalk evidence.img

  

# Step 2: Ekstrak semua file

binwalk -e evidence.img

cd _evidence.img.extracted/

  

# Step 3: Temukan file gambar

find . -name "*.jpg" -o -name "*.png" | head -20

  

# Step 4: Cek metadata gambar

exiftool image.jpg

# → Perhatikan field "Comment" → mungkin ada passphrase!

  

# Step 5: Cek steganografi

steghide info image.jpg

steghide extract -sf image.jpg -p "passphrase_dari_metadata"

  

# Step 6: Verifikasi flag

cat flag.txt

```

  

**Flag format:** `CTF{h1dd3n_1n_pl41n_s1gh7}`

  

---

  

### Soal 2: Log Analysis — SQLi Attack

  

**Skenario:** Website mengalami serangan. Kamu mendapat file `access.log`. Temukan:

- IP penyerang

- Payload SQLi pertama

- Data yang berhasil di-dump

  

**Langkah Penyelesaian:**

  

```bash

# Step 1: Identifikasi IP dengan request mencurigakan

grep -iE "union|select|'|--" access.log | awk '{print $1}' | sort | uniq -c | sort -rn

  

# Step 2: Filter request dari IP penyerang

grep "192.168.1.200" access.log | grep -iE "union|select"

  

# Step 3: Decode URL untuk baca payload

grep "192.168.1.200" access.log | awk '{print $7}' | python3 -c "

import sys, urllib.parse

for line in sys.stdin:

    print(urllib.parse.unquote(line.strip()))

" | grep -i union

  

# Step 4: Lihat response code untuk cari request sukses

grep "192.168.1.200" access.log | awk '{print $7, $9}' | grep " 200"

  

# Step 5: Cari payload yang mengembalikan flag

grep "192.168.1.200" access.log | grep "200" | tail -5

```

  

**Jawaban yang dicari:**

- IP: `192.168.1.200`

- Payload: `' UNION SELECT 1,flag,3 FROM flags--`

- Flag: `CTF{sql_1nj3ct10n_d3t3ct3d}`

  

---

  

### Soal 3: Browser Forensic — Chrome History

  

**Skenario:** Kamu mendapat disk image laptop Windows. Temukan URL terakhir yang dikunjungi oleh user sebelum laptop dimatikan.

  

**Langkah Penyelesaian:**

  

```bash

# Step 1: Mount atau akses disk image

# Untuk disk image Linux:

sudo mount -o loop,ro disk.img /mnt/evidence

  

# Step 2: Temukan profil Chrome

find /mnt/evidence -name "History" -path "*/Chrome/*" 2>/dev/null

  

# Step 3: Salin database (tidak bisa dibuka jika ter-lock)

cp "/mnt/evidence/Users/victim/AppData/Local/Google/Chrome/User Data/Default/History" /tmp/ChromeHistory

  

# Step 4: Query SQLite

sqlite3 /tmp/ChromeHistory "

SELECT

  url,

  title,

  datetime(last_visit_time/1000000-11644473600, 'unixepoch', 'localtime') AS visit_time

FROM urls

ORDER BY last_visit_time DESC

LIMIT 10;

"

  

# Step 5: Cari URL mencurigakan

sqlite3 /tmp/ChromeHistory "

SELECT url FROM urls

WHERE url LIKE '%pastebin%' OR url LIKE '%flag%' OR url LIKE '%secret%'

ORDER BY last_visit_time DESC;

"

```

  

**Flag format:** URL terakhir mengandung flag atau mengarah ke halaman dengan flag.

  

---

  

## Catatan Penting

  

### ⚠️ Yang Tidak Ada di Modul 1

  

| Topik | Ada di Kisi-kisi? | Kapan Diajarkan |

|---|---|---|

| Memory Forensic (Volatility) | ✅ | Modul 2 (Minggu 8+) |

| Malware Analysis | ✅ | Modul 2 (Minggu 8+) |

| Advanced Threat Hunting | ✅ | Modul 3 |

  

### 📝 Tentang Reporting

  

- Mini-writeup muncul sejak latihan awal

- Dalam LKS: penilaian utama = **keberhasilan ekstraksi flag**

- Format laporan teknikal bukan prioritas utama penilaian

  

### ✅ Checklist Kesiapan Modul 1

  

**OS Forensic:**

- [ ] Tahu lokasi registry Windows (`NTUSER.DAT`, `SOFTWARE`)

- [ ] Bisa baca Chrome History dengan SQLite

- [ ] Bisa analisis `/var/log/auth.log` Linux

- [ ] Paham timestamp analysis dengan `stat` dan `find`

  

**File Carving:**

- [ ] Hafal magic bytes: JPEG, PNG, PDF, ZIP, ELF

- [ ] Bisa gunakan `binwalk -e` dan `foremost`

- [ ] Bisa cari flag dengan `strings` + `grep`

  

**Steganografi:**

- [ ] Bisa baca metadata dengan `exiftool`

- [ ] Bisa ekstrak data dengan `steghide extract`

- [ ] Bisa scan LSB dengan `zsteg`

  

**Network Forensic:**

- [ ] Hafal filter Wireshark penting (`http`, `dns`, `tcp.stream`)

- [ ] Bisa gunakan `tshark` untuk extract field

- [ ] Bisa deteksi DNS/ICMP tunneling

  

**Log Forensic:**

- [ ] Bisa identifikasi SQLi dari access.log

- [ ] Bisa hitung IP terbanyak dengan `awk` + `sort` + `uniq`

- [ ] Paham format log Apache/Nginx

  

**CLI & Automasi:**

- [ ] Bisa tulis one-liner bash untuk cari flag

- [ ] Paham pipeline: `grep | awk | sort | uniq | head`

- [ ] Bisa decode URL dengan Python

  

---

  

> 📚 **Referensi:**

> - Kisi-kisi Resmi LKS Cyber 2026

> - Deskripsi Teknis LKS Bidang Cyber Security 2026

> - Jadwal Pelatihan Blue Team LKS 2026

  

---

  

*Modul ini 100% selaras dengan kisi-kisi resmi, deskripsi teknis, dan jadwal pelatihan — tanpa kelebihan atau kekurangan materi fondasi.*