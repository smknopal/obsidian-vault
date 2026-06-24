# 🛡️ WRITE-UP BLUE TEAM CTF LKS — LEVEL RINGAN (⭐)

> **Untuk siapa:** Kamu yang baru mulai belajar cybersecurity / forensik digital  

> **Soal yang dibahas:** RE-00, RE-01, FOR-00, FOR-01, MOB-01, SIEM-00  

> **Prinsip utama:** Baca soalnya dulu → pahami tujuannya → baru ikuti langkahnya  

  

---

  

## 📖 Cara Baca Write-up Ini

  

Setiap soal punya struktur yang sama:

1. **📋 SOAL** — pertanyaan / tantangan aslinya (biar kamu tahu ini soal tentang apa)

2. **🧠 Penjelasan Konsep** — kenapa soal ini bisa diselesaikan dengan cara tertentu

3. **🔧 Langkah-Langkah** — cara mengerjakannya step by step

4. **🏁 Flag / Jawaban** — hasil akhirnya

5. **💡 Pelajaran** — takeaway yang perlu diingat

  

`[SCREENSHOT: deskripsi]` = penanda posisi foto yang harus kamu ambil saat mengerjakan soal asli.

  

---

  

---

  

# 🔬 BAGIAN 1 — REVERSE ENGINEERING DASAR

  

---

  

## RE-00 | Membaca Source Code Sebelum Binary

  

**Kategori:** Reverse Engineering — Dasar  

**Tingkat:** ⭐ Pemula  

  

---

  

### 📋 SOAL

  

> *Diberikan sebuah file script Python yang tampilannya membingungkan (nama variabelnya acak, susah dibaca). Di dalamnya tersembunyi flag. Temukan flagnya tanpa menggunakan tools berat seperti debugger atau disassembler.*

  

Contoh file yang diberikan (`challenge.py`):

```python

import base64

_0x1a = "ZmxhZ3tiNHMzXzY0XzFzXzNhc3l9"

_0x2b = base64.b64decode(_0x1a)

if input("Enter key: ").encode() == _0x2b:

    print("Correct!")

else:

    print("Wrong!")

```

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu "obfuscated script"?**  

Obfuscated = disengaja dibuat membingungkan. Nama variabelnya seperti `_0x1a`, `_0x2b` — bukan nama yang bermakna. Tujuannya supaya orang malas membacanya.

  

**Tapi kuncinya sederhana:** program ini hanya melakukan 3 hal:

1. Ambil string acak yang ada di kode

2. Decode dengan base64

3. Bandingkan dengan input kamu

  

**Artinya:** hasil decode base64 dari string itu = flag yang dicari!

  

**Apa itu Base64?**  

Base64 adalah cara mengubah data jadi teks yang aman untuk dikirim. Ciri-cirinya:

- Karakter yang dipakai: `A-Z`, `a-z`, `0-9`, `+`, `/`, `=`

- Panjangnya selalu kelipatan 4

- Sering diakhiri dengan `=` atau `==`

  

Contoh: `ZmxhZw==` kalau di-decode menghasilkan `flag`

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Baca script baris per baris (jangan panik)**

  

```bash

cat challenge.py

```

  

[SCREENSHOT: Terminal menampilkan isi file challenge.py dengan perintah `cat`, semua baris terlihat]

  

Terjemahkan tiap baris dalam bahasa manusia:

- Baris 1: `import base64` → "saya pakai library untuk encode/decode base64"

- Baris 2: `_0x1a = "ZmxhZ3tiN..."` → "simpan string ini di variabel"

- Baris 3: `_0x2b = base64.b64decode(_0x1a)` → "decode string itu dari base64"

- Baris 4-7: bandingkan input dengan hasil decode → **hasil decode = flag!**

  

**Langkah 2 — Ambil string base64-nya**

  

String yang perlu kita decode adalah: `ZmxhZ3tiNHMzXzY0XzFzXzNhc3l9`

  

Cek dulu: apakah ini base64?

- Ada huruf besar, kecil, angka ✓

- Diakhiri `9` (bukan `=` tapi masih valid untuk base64 tanpa padding) ✓

- Panjangnya 32 karakter (kelipatan 4) ✓

  

**Langkah 3 — Decode di terminal (satu perintah!)**

  

```bash

echo "ZmxhZ3tiNHMzXzY0XzFzXzNhc3l9" | base64 -d

```

  

Penjelasan perintah:

- `echo "..."` → cetak string ke layar

- `|` → "kirim outputnya ke perintah berikutnya"

- `base64 -d` → decode base64 (`-d` artinya decode)

  

Output:

```

flag{b4s3_64_1s_3asy}

```

  

[SCREENSHOT: Terminal menampilkan perintah echo + base64 -d dan hasilnya `flag{b4s3_64_1s_3asy}` dalam satu frame]

  

**Langkah 4 — Verifikasi (pastikan jawaban benar)**

  

```bash

python3 challenge.py

# Saat diminta input, ketik: flag{b4s3_64_1s_3asy}

# Output harus: Correct!

```

  

[SCREENSHOT: Terminal menjalankan python3 challenge.py, input flag, dan output "Correct!"]

  

---

  

### 🏁 Flag

  

```

flag{b4s3_64_1s_3asy}

```

  

---

  

### 💡 Pelajaran

  

> Sebelum pakai tools canggih, **baca dulu logika programnya**. Banyak soal "reverse engineering" tingkat pemula sebenarnya cukup diselesaikan dengan membaca kode dan decode encoding sederhana (base64, hex, ROT13).

>

> Cara cepat kenali encoding:

> - Terlihat seperti `ZmxhZw==` → Base64

> - Terlihat seperti `666c6167` → Hex

> - Terlihat seperti `synt{...}` → ROT13 (huruf geser 13)

  

---

  

---

  

## RE-01 | Strings Hunt

  

**Kategori:** Reverse Engineering — Static Analysis  

**Tingkat:** ⭐ Pemula  

  

---

  

### 📋 SOAL

  

> *Diberikan sebuah file binary (program yang sudah dikompilasi, tidak bisa dibaca langsung). Kamu tidak tahu isinya. Tugasmu menemukan flag yang tersembunyi di dalamnya tanpa perlu menjalankan program atau membuka debugger.*

  

File yang diberikan: `strings_hunt` (binary ELF Linux)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu binary/ELF?**  

Binary adalah file program yang sudah dikompilasi — kalau kamu buka dengan text editor, isinya tampak kacau (karakter aneh semua). Tapi di dalam binary, **string teks yang hardcoded** (ditulis langsung di kode oleh programmer) tetap tersimpan sebagai teks biasa.

  

**Apa itu `strings`?**  

`strings` adalah tool Linux yang tugasnya satu: **cari semua teks yang bisa dibaca manusia di dalam file apapun**. Termasuk di dalam binary yang terlihat "rusak".

  

Ibarat mencari kata dalam buku yang halamannya rusak — tool `strings` tetap bisa menemukan kata-kata yang masih utuh.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Identifikasi tipe file dulu (wajib!)**

  

```bash

file strings_hunt

```

  

Output yang muncul:

```

strings_hunt: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),

dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,

BuildID[sha1]=..., for GNU/Linux 3.2.0, stripped

```

  

Artinya:

- `ELF 64-bit` → ini binary Linux 64-bit ✓

- `stripped` → simbol debug dihapus (tapi flag bisa tetap ada)

- `dynamically linked` → pakai library eksternal

  

[SCREENSHOT: Terminal menjalankan `file strings_hunt` dengan output lengkap tipe ELF]

  

**Langkah 2 — Cari string yang mengandung kata "flag"**

  

```bash

strings strings_hunt | grep -i "flag\|ctf\|lks\|key\|pass"

```

  

Penjelasan perintah:

- `strings strings_hunt` → ekstrak semua teks dari binary

- `| grep -i "flag\|ctf\|..."` → filter hanya baris yang mengandung kata-kata itu

- `-i` → case-insensitive (FLAG = flag = Flag, semua sama)

- `\|` → artinya "ATAU" (cari flag ATAU ctf ATAU lks)

  

Output:

```

flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}

```

  

[SCREENSHOT: Terminal menjalankan perintah `strings strings_hunt | grep -i "flag"` dan hasilnya menampilkan flag]

  

**Langkah 3 — Coba filter lain jika cara pertama tidak berhasil**

  

Kadang flag tidak menyebut kata "flag" secara eksplisit, tapi selalu dalam kurung kurawal `{}`:

  

```bash

strings strings_hunt | grep "{"

```

  

Output:

```

flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}

```

  

**Langkah 4 — Lihat konteks sekitar flag (untuk writeup yang lebih lengkap)**

  

```bash

strings strings_hunt | grep -A3 -B3 "flag"

```

  

`-A3` = tampilkan 3 baris After (setelah), `-B3` = 3 baris Before (sebelum)

  

Output:

```

Enter password:

Wrong password!

flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}

Correct! You found it!

```

  

Ini membuktikan flag disimpan sebagai string biasa (hardcoded) di dalam binary.

  

[SCREENSHOT: Terminal menjalankan grep -A3 -B3, tampak konteks sekitar flag termasuk "Enter password" dan "Correct!"]

  

---

  

### 🏁 Flag

  

```

flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}

```

  

---

  

### 💡 Pelajaran

  

> `strings` adalah senjata pertama dan paling cepat saat menerima file binary apapun. Selalu lakukan ini **sebelum** membuka Ghidra atau GDB.

>

> Urutan analisis binary (dari paling mudah ke susah):

> 1. `file namafile` — kenali dulu tipenya

> 2. `strings namafile | grep "flag\|{"` — cari flag langsung

> 3. `ltrace ./namafile` — lihat fungsi library yang dipanggil

> 4. `gdb` / `ghidra` — baru disassembly kalau cara sebelumnya gagal

  

---

  

---

  

# 🔍 BAGIAN 2 — DIGITAL FORENSICS DASAR

  

---

  

## FOR-00 | Identifikasi Tipe File Tanpa Asumsi Ekstensi

  

**Kategori:** Media Forensics — Dasar  

**Tingkat:** ⭐ Pemula  

  

---

  

### 📋 SOAL

  

> *Diberikan file bernama `dokumen.txt`. Tapi saat kamu coba buka, isinya bukan teks biasa. Selidiki tipe file sebenarnya, dan temukan flag yang tersembunyi di dalamnya.*

  

---

  

### 🧠 Penjelasan Konsep

  

**Kenapa ekstensi file bisa berbohong?**  

Di komputer, nama file dan isi file itu terpisah. Kamu bisa rename `gambar.png` jadi `dokumen.txt` — nama berubah, tapi isi filenya tetap file gambar.

  

**Cara komputer tahu tipe file sebenarnya: Magic Bytes**  

Setiap tipe file punya "tanda tangan" di bagian awal isinya yang disebut **magic bytes** (bytes pertama file). Ini tidak bisa dipalsukan hanya dengan rename.

  

| Tipe File | Magic Bytes (hex) | Cara Baca |

|-----------|-------------------|-----------|

| PNG | `89 50 4E 47` | `.PNG` |

| JPG | `FF D8 FF` | (tidak terbaca) |

| ZIP | `50 4B 03 04` | `PK` |

| PDF | `25 50 44 46` | `%PDF` |

| ELF | `7F 45 4C 46` | `.ELF` |

  

**Tool `file`** membaca magic bytes ini untuk menentukan tipe file sebenarnya.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Jangan buka filenya dulu. Cek tipe aslinya!**

  

```bash

file dokumen.txt

```

  

Output:

```

dokumen.txt: PNG image data, 800 x 600, 8-bit/color RGB, non-interlaced

```

  

**Ini PNG, bukan text file!** Meski namanya `.txt`, isinya gambar.

  

[SCREENSHOT: Terminal menjalankan `file dokumen.txt` dan output menunjukkan "PNG image data" bukan text file]

  

**Langkah 2 — Konfirmasi dengan melihat magic bytes langsung**

  

```bash

xxd dokumen.txt | head -2

```

  

Penjelasan: `xxd` menampilkan isi file dalam format hexadecimal (hex)

  

Output:

```

00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR

```

  

Perhatikan:

- `8950 4e47` dalam hex = `89 50 4E 47` = `.PNG` ✓

- Di kolom kanan (`ASCII representation`) terlihat `.PNG`

  

[SCREENSHOT: Terminal menjalankan `xxd dokumen.txt | head -2` dengan magic bytes PNG terlihat di kolom hex]

  

**Langkah 3 — Copy dengan ekstensi yang benar, lalu analisis**

  

```bash

cp dokumen.txt dokumen_asli.png

exiftool dokumen_asli.png

```

  

`exiftool` membaca semua metadata (informasi tersembunyi) di dalam file gambar.

  

Output:

```

ExifTool Version Number         : 12.60

File Name                       : dokumen_asli.png

MIME Type                       : image/png

Image Width                     : 800

Image Height                    : 600

Comment                         : flag{trust_magic_bytes_not_extension}

```

  

**Flag tersembunyi di field `Comment` metadata!**

  

[SCREENSHOT: Terminal menjalankan `exiftool dokumen_asli.png` dan output menampilkan field Comment berisi flag]

  

---

  

### 🏁 Flag

  

```

flag{trust_magic_bytes_not_extension}

```

  

---

  

### 💡 Pelajaran

  

> **Jangan pernah percaya ekstensi file.** Selalu jalankan `file namafile` sebagai langkah pertama untuk setiap file yang kamu terima di soal forensik. Ini adalah jebakan paling umum di LKS.

>

> Hafal magic bytes PNG: `89 50 4E 47` — kamu akan sering menemuinya di soal forensik.

  

---

  

---

  

## FOR-01 | Hidden in Image (Steganografi)

  

**Kategori:** Media Forensics  

**Tingkat:** ⭐ Pemula  

  

---

  

### 📋 SOAL

  

> *Diberikan file gambar `secret_image.png`. Penampilannya normal — gambar biasa. Tapi di dalamnya tersembunyi sesuatu. Temukan pesan atau flag yang disembunyikan di dalam gambar ini.*

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu steganografi?**  

Steganografi = menyembunyikan data di dalam data lain. Berbeda dengan enkripsi (data dikacak tapi kelihatan ada), steganografi membuat data terlihat **tidak ada** — tersembunyi di dalam gambar, audio, atau file lain.

  

**Cara umum menyembunyikan data di gambar:**

1. **Di metadata** — field tersembunyi seperti `Comment`, `Artist`, dll.

2. **Append/menempel** — file lain (ZIP, TXT) ditempelkan di akhir file gambar

3. **LSB (Least Significant Bit)** — bit terakhir setiap pixel diubah untuk menyimpan data

4. **Di dalam file gambar** — gambar disisipkan ke dalam gambar lain

  

**Urutan pengecekan yang benar (jangan loncat-loncat):**

```

file → exiftool → strings → binwalk → zsteg → steghide

```

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Verifikasi tipe file**

  

```bash

file secret_image.png

```

  

Output:

```

secret_image.png: PNG image data, 1280 x 720, 8-bit/color RGB, non-interlaced

```

  

File PNG valid. ✓

  

[SCREENSHOT: Terminal menjalankan `file secret_image.png`, output PNG valid]

  

**Langkah 2 — Cek metadata (kadang flag ada di sini)**

  

```bash

exiftool secret_image.png

```

  

Output:

```

File Name     : secret_image.png

Image Width   : 1280

Image Height  : 720

Bit Depth     : 8

Color Type    : RGB

```

  

Tidak ada yang mencurigakan di metadata. Lanjut ke langkah berikutnya.

  

[SCREENSHOT: Output exiftool, tidak ada string mencurigakan di metadata]

  

**Langkah 3 — Cari string tersembunyi**

  

```bash

strings secret_image.png | grep -i "flag\|ctf\|lks"

```

  

Output: kosong. Tidak ada string flag langsung. Lanjut.

  

**Langkah 4 — Scan dengan binwalk (ini kuncinya!)**

  

```bash

binwalk secret_image.png

```

  

Penjelasan: `binwalk` menganalisis seluruh isi file dan mencari "file di dalam file" berdasarkan magic bytes.

  

Output:

```

DECIMAL       HEXADECIMAL     DESCRIPTION

--------------------------------------------------------------------------------

0             0x0             PNG image, 1280 x 720, 8-bit/color RGB

156423        0x2634F         Zip archive data, at least v2.0 to extract, name: secret.txt

156821        0x264D5         End of Zip archive, footer length: 22

```

  

**Ada file ZIP tersembunyi di dalam PNG!** Di offset `156423` (sekitar 153KB dari awal file).

  

[SCREENSHOT: Output binwalk menampilkan dua entry: PNG image dan Zip archive data dengan offset hexadecimal]

  

**Langkah 5 — Ekstrak file tersembunyi**

  

```bash

binwalk -e secret_image.png

```

  

`-e` artinya "extract" (ekstrak semua yang ditemukan). `binwalk` akan membuat folder bernama `_secret_image.png.extracted/`

  

```bash

ls -la _secret_image.png.extracted/

```

  

Output:

```

-rw-r--r-- 1 kali kali  398 ... 2634F.zip

-rw-r--r-- 1 kali kali   43 ... secret.txt

```

  

[SCREENSHOT: Output ls menampilkan isi folder extracted, ada file .zip dan secret.txt]

  

**Langkah 6 — Baca hasilnya!**

  

```bash

cat _secret_image.png.extracted/secret.txt

```

  

Output:

```

flag{st3g0_h1dd3n_1n_pl41n_s1ght_w1th_b1nwalk}

```

  

[SCREENSHOT: Terminal menjalankan `cat secret.txt` dan flag tampil di layar]

  

---

  

### 🏁 Flag

  

```

flag{st3g0_h1dd3n_1n_pl41n_s1ght_w1th_b1nwalk}

```

  

---

  

### 💡 Pelajaran

  

> `binwalk` adalah tool terpenting untuk mendeteksi "file di dalam file". Cara kerjanya: scan seluruh isi file dari awal sampai akhir, cari magic bytes dari semua tipe file yang dikenali.

>

> Urutan wajib steganografi:  

> `file` → `exiftool` → `strings` → **`binwalk`** → `zsteg` → `steghide`  

> Jangan loncat ke `steghide` dulu — `binwalk` jauh lebih sering berhasil!

  

---

  

---

  

# 📱 BAGIAN 3 — PHONE FORENSICS DASAR

  

---

  

## MOB-01 | Android Backup Analysis

  

**Kategori:** Phone Forensics — Android  

**Tingkat:** ⭐ Pemula  

  

---

  

### 📋 SOAL

  

> *Diberikan file `backup.ab` — ini adalah file backup Android yang dibuat dengan perintah `adb backup`. Di dalam backup tersebut tersembunyi informasi penting (pesan SMS berisi flag). Ekstrak backup dan temukan flag-nya.*

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu file `.ab`?**  

File `.ab` adalah format backup Android (Android Backup). Dibuat dengan perintah `adb backup` yang menyimpan data aplikasi dari ponsel.

  

**Kenapa tidak bisa langsung dibuka?**  

Format `.ab` bukan ZIP biasa — perlu tool khusus bernama **ABE (Android Backup Extractor)** untuk mengubahnya jadi `.tar` yang bisa diekstrak.

  

**Struktur backup Android:**

```

backup.ab → (decode dengan ABE) → backup.tar → (extract tar) →

apps/

  com.android.providers.telephony/

    db/

      mmssms.db    ← database SMS di sini!

      contacts2.db ← database kontak

  com.whatsapp/

    db/

      msgstore.db  ← pesan WhatsApp

```

  

**Apa itu SQLite?**  

Database yang digunakan Android untuk menyimpan hampir semua data lokal (SMS, kontak, riwayat browser). File `.db` = database SQLite. Dibaca dengan tool `sqlite3`.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Verifikasi file backup**

  

```bash

file backup.ab

xxd backup.ab | head -3

```

  

Output:

```

backup.ab: Android backup

00000000: 414e 4452 4f49 4420 4241 434b 5550 0a   ANDROID BACKUP.

```

  

Magic header: `ANDROID BACKUP` — terkonfirmasi. ✓

  

[SCREENSHOT: Terminal menjalankan `xxd backup.ab | head -3` dan tampak header "ANDROID BACKUP" di kolom ASCII]

  

**Langkah 2 — Convert .ab ke .tar dengan ABE**

  

Jika `abe.jar` belum ada, download dulu:

```bash

# Download abe.jar dari GitHub nelenkov/android-backup-extractor

wget https://github.com/nelenkov/android-backup-extractor/releases/latest/download/abe.jar

```

  

Lalu jalankan konversi:

```bash

java -jar abe.jar unpack backup.ab backup.tar

```

  

Output:

```

Decrypting backup...

Backup successfully unpacked to backup.tar

```

  

> ℹ️ Jika backup diberi password: `java -jar abe.jar unpack backup.ab backup.tar passwordnya`

  

[SCREENSHOT: Terminal menjalankan java -jar abe.jar dan output menampilkan "Backup successfully unpacked"]

  

**Langkah 3 — Ekstrak isi backup**

  

```bash

mkdir extracted_backup

tar -xvf backup.tar -C extracted_backup/

```

  

Output menampilkan daftar file:

```

apps/com.android.providers.telephony/db/mmssms.db

apps/com.android.providers.telephony/db/mmssms.db-shm

apps/com.android.contacts/db/contacts2.db

apps/com.android.providers.settings/sp/settings.xml

...

```

  

[SCREENSHOT: Output tar -xvf menampilkan daftar file yang diekstrak, termasuk mmssms.db]

  

**Langkah 4 — Buka database SMS**

  

```bash

sqlite3 extracted_backup/apps/com.android.providers.telephony/db/mmssms.db

```

  

Di dalam sqlite3 shell, ketik perintah SQL:

```sql

-- Lihat semua tabel yang ada

.tables

```

  

Output:

```

android_metadata  drafts  sms  threads

```

  

```sql

-- Baca isi SMS (kolom: id, nomor pengirim, waktu, isi pesan)

SELECT _id, address, date, body FROM sms LIMIT 10;

```

  

Output:

```

1|+6281234567890|1673852400000|Hei, ini pesannya ya

2|+6289876543210|1673852460000|flag{andr01d_sms_s3cr3t_f0und}

3|+6281234567890|1673852520000|Jangan kasih tau siapapun!

```

  

**Flag ada di isi SMS nomor 2!**

  

[SCREENSHOT: SQLite3 shell menampilkan output SELECT dari tabel sms, baris kedua berisi flag]

  

**Langkah 5 — Keluar dari sqlite3**

  

```sql

.quit

```

  

---

  

### 🏁 Flag

  

```

flag{andr01d_sms_s3cr3t_f0und}

```

  

---

  

### 💡 Pelajaran

  

> Data Android tersimpan di database SQLite `.db`. Tiga database paling penting:

> - `mmssms.db` → SMS dan MMS

> - `contacts2.db` → Kontak

> - `msgstore.db` (WhatsApp) → Pesan WhatsApp

>

> Perintah SQL paling sering dipakai: `SELECT * FROM namatabel;` — menampilkan semua isi tabel.

  

---

  

---

  

# 📊 BAGIAN 4 — SIEM DASAR

  

---

  

## SIEM-00 | Membaca Log Mentah Sebelum Query

  

**Kategori:** SIEM — Dasar  

**Tingkat:** ⭐ Pemula  

  

---

  

### 📋 SOAL

  

> *Diberikan file `access.log` — ini adalah log web server Apache yang merekam semua request yang masuk. Tanpa menggunakan tools SIEM (Splunk/ELK), temukan IP mana yang melakukan serangan brute force dengan hanya menggunakan perintah Linux dasar (`grep`, `awk`, `sort`, `uniq`).*

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu access log?**  

Setiap request yang masuk ke web server dicatat di file log. Satu baris = satu request.

  

**Format baris log Apache:**

```

192.168.1.105 - - [15/Jan/2025:09:00:02 +0700] "POST /login HTTP/1.1" 401 - "-" "python-requests"

[  IP  ]          [     waktu      ]     [metode] [URL]         [kode] [bytes]

```

  

**Kode status HTTP yang penting:**

| Kode | Artinya |

|------|---------|

| 200 | Sukses |

| 301/302 | Redirect (login berhasil → pindah halaman) |

| 401 | Unauthorized (login gagal) |

| 403 | Forbidden |

| 404 | Not Found |

| 500 | Server Error |

  

**Pola brute force di log:**  

Banyak `401` berulang dari IP yang sama ke endpoint `/login` = brute force!

  

**Tool Linux yang dipakai:**

- `awk '{print $1}'` → ambil kolom pertama (IP address)

- `sort` → urutkan

- `uniq -c` → hitung kemunculan unik (`-c` = count)

- `sort -nr` → urut dari terbesar (`-n` = numerik, `-r` = reverse/terbalik)

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Lihat dulu isi log (orientasi)**

  

```bash

head -5 access.log

```

  

Output:

```

192.168.1.100 - - [15/Jan/2025:09:00:01 +0700] "GET /index.html HTTP/1.1" 200 4523

192.168.1.105 - - [15/Jan/2025:09:00:02 +0700] "POST /login HTTP/1.1" 401 - "python-requests/2.28"

192.168.1.105 - - [15/Jan/2025:09:00:02 +0700] "POST /login HTTP/1.1" 401 - "python-requests/2.28"

192.168.1.105 - - [15/Jan/2025:09:00:02 +0700] "POST /login HTTP/1.1" 401 - "python-requests/2.28"

192.168.1.100 - - [15/Jan/2025:09:00:03 +0700] "GET /about.html HTTP/1.1" 200 1823

```

  

Sudah terlihat ada yang mencurigakan: `192.168.1.105` muncul berulang dengan `POST /login` dan status `401`.

  

[SCREENSHOT: Terminal menjalankan `head -5 access.log` dan output 5 baris pertama log Apache terlihat jelas]

  

**Langkah 2 — Hitung berapa request per IP (one-liner ajaib)**

  

```bash

awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -5

```

  

Cara membacanya (dari kiri ke kanan):

1. `awk '{print $1}'` — ambil kolom 1 (kolom IP) dari semua baris

2. `sort` — urutkan A-Z (supaya IP yang sama berurutan)

3. `uniq -c` — hitung IP yang sama berturut-turut

4. `sort -nr` — urutkan dari jumlah terbanyak ke sedikit

5. `head -5` — tampilkan 5 teratas saja

  

Output:

```

    854 192.168.1.105    ← ini! 854 request dari satu IP

     43 192.168.1.100

     22 10.0.0.1

```

  

[SCREENSHOT: Output awk command dengan IP 192.168.1.105 dan angka 854 di posisi teratas]

  

**Langkah 3 — Cek apa yang dilakukan IP mencurigakan itu**

  

```bash

grep "192.168.1.105" access.log | awk '{print $6, $7}' | sort | uniq -c | sort -nr

```

  

- `$6` = metode HTTP (GET/POST)

- `$7` = URL yang diakses

  

Output:

```

    853 "POST /login

      1 "GET /dashboard

```

  

853 request POST ke `/login` = **brute force attack!**  

1 request GET ke `/dashboard` = **berhasil masuk!**

  

**Langkah 4 — Tentukan waktu serangan berlangsung**

  

```bash

# Waktu request pertama

grep "192.168.1.105" access.log | awk '{print $4}' | head -1

  

# Waktu request terakhir

grep "192.168.1.105" access.log | awk '{print $4}' | tail -1

```

  

Output:

```

[15/Jan/2025:09:00:02

[15/Jan/2025:09:02:56

```

  

Serangan berlangsung ~3 menit dengan 853 percobaan = hampir 5 request/detik. Ini jelas otomatis (pakai script), bukan manusia mengetik manual.

  

---

  

### 🏁 Jawaban / Flag

  

```

IP Penyerang    : 192.168.1.105

Jumlah Attempt  : 853 (gagal) + 1 (berhasil masuk)

Endpoint Target : /login (POST)

Durasi Serangan : 09:00:02 - 09:02:56 (±3 menit)

User Agent      : python-requests/2.28 (pakai script Python!)

Status Akhir    : Login berhasil → akses /dashboard

```

  

---

  

### 💡 Pelajaran

  

> Skill membaca log mentah dengan `awk + sort + uniq + grep` adalah **fallback** penting ketika SIEM down atau lambat. Di lomba, kalau Splunk/ELK tidak responsif, kamu masih bisa investigasi manual.

>

> Pola brute force di log:

> - Banyak request POST ke `/login` atau `/wp-login.php`

> - Status code `401` berulang dari satu IP

> - User-Agent menunjukkan tool otomatis (`python-requests`, `curl`, `Hydra`, dll.)

  

---

  

---

  

# 📋 RINGKASAN LEVEL RINGAN

  

| Soal | Kategori | Tools | Flag |

|------|----------|-------|------|

| RE-00 | Reverse Engineering | `base64 -d` | `flag{b4s3_64_1s_3asy}` |

| RE-01 | Reverse Engineering | `strings`, `grep` | `flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}` |

| FOR-00 | Media Forensics | `file`, `xxd`, `exiftool` | `flag{trust_magic_bytes_not_extension}` |

| FOR-01 | Media Forensics | `file`, `binwalk` | `flag{st3g0_h1dd3n_1n_pl41n_s1ght_w1th_b1nwalk}` |

| MOB-01 | Phone Forensics | `abe.jar`, `sqlite3` | `flag{andr01d_sms_s3cr3t_f0und}` |

| SIEM-00 | Log Analysis | `awk`, `grep`, `sort`, `uniq` | Jawaban investigasi |

  

---

  

## 🛠️ Cheat Sheet Tools — Level Ringan

  

```bash

# Cek tipe file (WAJIB pertama kali)

file namafile

  

# Lihat teks di dalam binary apapun

strings namafile | grep -i "flag\|{"

  

# Lihat hex/magic bytes

xxd namafile | head -5

  

# Cek metadata gambar

exiftool namafile.png

  

# Cari file tersembunyi di dalam file

binwalk namafile.png

binwalk -e namafile.png   # ekstrak

  

# Decode base64

echo "stringbase64" | base64 -d

  

# Analisis log: hitung request per IP

awk '{print $1}' access.log | sort | uniq -c | sort -nr

  

# Buka database SQLite

sqlite3 database.db

# Di dalam: .tables → SELECT * FROM namatabel;

  

# Decode Android backup

java -jar abe.jar unpack backup.ab output.tar

tar -xvf output.tar -C folder_output/

```

  

---

  

*Level selanjutnya: Buka file `02_LEVEL_SEDANG_Bintang2.md` untuk soal ⭐⭐*  

*Semua materi diambil dari platform latihan publik: PicoCTF, TryHackMe, HackTheBox*