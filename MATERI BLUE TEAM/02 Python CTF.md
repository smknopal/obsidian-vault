---

title: Python CTF — bytes, struct, base64, hashlib & File Forensics

tags: [LKS2026, BlueTeam, Python, CTF, Forensics, FileAnalysis]

date: 2026-06-04

cssclasses: [wide-page]

---

  

# 🐍 Python CTF — Part 2

## `bytes` · `struct` · `base64` · `hashlib` · Magic Bytes · `xxd`

  

> [!quote] Misi Malam Ini

> **Target:** Diberikan file aneh yang tidak bisa dibuka → bisa decode & analisis pakai Python dalam 10 baris.

> Kamu **pemula**? Bagus. Semua dijelaskan dari nol. Ikuti urutannya.

  

---

  

## 🧠 Fondasi Dulu — Apa Itu "File" Sebenarnya?

  

> [!info] Konsep Paling Dasar

> **Semua file di komputer hanyalah deretan angka 0 dan 1 (bit).**

> Karena bit terlalu kecil, kita kelompokkan 8 bit menjadi 1 **byte**.

> 1 byte bisa merepresentasikan angka **0 sampai 255**.

>

> File teks, gambar, video, executable — **semuanya byte**.

> Yang membedakan adalah **bagaimana byte itu diinterpretasikan**.

  

```

File "hello.txt" berisi teks: H e l l o

Dalam byte (desimal):        72 101 108 108 111

Dalam hex:                   48  65  6C  6C  6F

Dalam biner:           01001000 01100101 ...

```

  

> [!tip] Mengapa Hex (Heksadesimal)?

> Manusia susah baca biner (0101001...). Hex lebih ringkas:

> - 1 byte = 2 digit hex

> - `72` desimal = `0x48` hex = `01001000` biner

> - Semua sama, hanya **cara nulis yang berbeda**

>

> Di CTF, kamu akan **selalu berurusan dengan hex** karena itulah cara standar melihat isi file.

  

---

  

## ⚡ XXD & HEX DUMP — Melihat "Isi Asli" File

  

> [!example] Konsep

> `xxd` adalah tool Linux untuk melihat isi file dalam format hex.

> Ini adalah **langkah pertama** setiap kali dapat file misterius di CTF.

  

### Cara Pakai `xxd`

  

```bash

# Lihat isi file dalam hex

xxd file_misterius.bin

  

# Tampilkan hanya N byte pertama

xxd -l 32 file_misterius.bin

  

# Tampilkan dalam format plain hex (tanpa teks di kanan)

xxd -p file_misterius.bin

  

# Convert hex string KEMBALI ke binary

echo "48656c6c6f" | xxd -r -p    # Output: Hello

  

# Simpan hasil xxd ke file teks

xxd file.bin > dump.txt

```

  

### Membaca Output `xxd`

  

```bash

$ xxd gambar_aneh.jpg | head -4

  

00000000: ffd8 ffe0 0010 4a46 4946 0001 0100 0001  ......JFIF......

00000010: 0001 0000 ffdb 0\\043 0008 0606 0706 0508  .......C........

  

│         │                              │         │

│         │                              │         └─ Teks ASCII (. = non-printable)

│         │                              └─ 16 byte dalam hex

│         └─ Isi file (hex)

└─ Offset (posisi byte dari awal file)

```

  

> [!warning] Cara Baca Offset

> `00000010` artinya byte ke-16 (hex 10 = desimal 16).

> Offset selalu dalam **heksadesimal**.

  

---

  

## 🔮 MAGIC BYTES — Sidik Jari File

  

> [!info] Konsep

> Setiap tipe file punya **tanda tangan unik di byte pertamanya** yang disebut **Magic Bytes** atau **File Signature**.

> Ekstensi file (`.jpg`, `.png`) bisa diubah/dihapus — tapi **magic bytes tidak bohong**.

>

> Di CTF, sering dikasih file dengan ekstensi salah atau tanpa ekstensi → cek magic bytes!

  

### Tabel Magic Bytes yang WAJIB Hafal

  

| File Type | Magic Bytes (Hex) | ASCII | Cara Baca di xxd |

|-----------|-------------------|-------|------------------|

| **JPEG** | `FF D8 FF` | `ÿØÿ` | Selalu mulai `ffd8 ff` |

| **PNG** | `89 50 4E 47 0D 0A 1A 0A` | `‰PNG....` | `8950 4e47` |

| **PDF** | `25 50 44 46` | `%PDF` | `2550 4446` |

| **ZIP** | `50 4B 03 04` | `PK..` | `504b 0304` |

| **ELF** (Linux binary) | `7F 45 4C 46` | `.ELF` | `7f45 4c46` |

| **GIF** | `47 49 46 38` | `GIF8` | `4749 4638` |

| **RAR** | `52 61 72 21` | `Rar!` | `5261 7221` |

| **7-ZIP** | `37 7A BC AF` | `7z..` | `377a bcaf` |

  

### Cara Identifikasi File di CTF

  

```bash

# Metode 1: pakai command `file` (paling mudah)

file file_misterius.bin

# Output: file_misterius.bin: PNG image data, 800 x 600, 8-bit/color RGB

  

# Metode 2: lihat magic bytes pakai xxd

xxd -l 8 file_misterius.bin

# 00000000: 8950 4e47 0d0a 1a0a  .PNG....

# → Ini PNG!

  

# Metode 3: pakai Python

with open("file_misterius.bin", "rb") as f:

    magic = f.read(8)

    print(magic.hex())

    # Output: 89504e470d0a1a0a → PNG ✓

```

  

> [!tip] Latihan Cepat

> ```bash

> # Kamu dapat file "data.txt" tapi sepertinya bukan teks biasa

> file data.txt               # Cek dulu

> xxd -l 16 data.txt          # Lihat 16 byte pertama

> # Setelah tahu tipenya → rename dengan ekstensi yang benar

> mv data.txt data.png        # jika ternyata PNG

> ```

  

---

  

## 🐍 PYTHON BYTES — Bekerja dengan Data Mentah

  

> [!info] Konsep dari Nol

> Di Python, ada dua cara simpan data:

> - `str` → untuk **teks** (manusia bisa baca): `"Hello"`

> - `bytes` → untuk **data biner/raw** (angka 0-255): `b"Hello"` atau `b'\x48\x65\x6c\x6c\x6f'`

>

> Saat buka file apapun (gambar, binary, zip), **selalu pakai mode `"rb"` (read binary)**.

  

### Cheatsheet Python `bytes`

  

```python

# ── MEMBUAT BYTES ──────────────────────────────────────────────

b1 = b"Hello"              # Dari string literal

b2 = bytes([72, 101, 108]) # Dari list angka desimal → b'Hel'

b3 = bytes.fromhex("48656c6c6f")  # Dari string hex → b'Hello'

b4 = b'\x48\x65\x6c'      # Dari escape hex langsung

  

# ── MEMBACA FILE BINER ────────────────────────────────────────

with open("file.bin", "rb") as f:

    data = f.read()         # Baca semua isi file sebagai bytes

    header = f.read(4)      # Baca 4 byte pertama

  

# ── KONVERSI ──────────────────────────────────────────────────

data = b"Hello"

  

data.hex()           # → '48656c6c6f'   (bytes ke hex string)

data.decode("utf-8") # → 'Hello'        (bytes ke string)

list(data)           # → [72, 101, 108, 108, 111] (bytes ke list angka)

  

"Hello".encode("utf-8")   # → b'Hello'  (string ke bytes)

bytes.fromhex("48656c6c6f")  # → b'Hello' (hex string ke bytes)

  

# ── SLICING (POTONG-POTONG BYTES) ─────────────────────────────

data = b'\x89PNG\r\n\x1a\nINFODATA'

data[0:4]       # → b'\x89PNG'   (ambil byte 0 sampai 3)

data[4:]        # → b'\r\n\x1a\nINFODATA' (dari byte ke-4 sampai akhir)

data[-4:]       # → b'DATA'      (4 byte terakhir)

data[0]         # → 137          (satu byte = angka integer!)

  

# ── XOR (OPERASI PALING SERING DI CTF!) ───────────────────────

# XOR: a ^ b. Sifat ajaib: (a ^ key) ^ key = a  (bisa di-reverse!)

data   = b'\x1b\x01\x11\x0b'

key    = 0x73                       # key satu byte

result = bytes([b ^ key for b in data])  # XOR setiap byte dengan key

print(result)   # → b'helo' atau hasil decode lainnya

  

# XOR dengan key multi-byte

key_bytes = b"KEY"

result = bytes([data[i] ^ key_bytes[i % len(key_bytes)] for i in range(len(data))])

  

# ── MENULIS FILE BINER ─────────────────────────────────────────

with open("output.bin", "wb") as f:

    f.write(b'\x89PNG\r\n\x1a\n')   # Tulis magic bytes PNG

    f.write(data)

```

  

### Contoh Output Nyata

  

```python

>>> data = b"CTF{secret}"

>>> data.hex()

'4354467b73656372657d'

  

>>> bytes.fromhex('4354467b73656372657d')

b'CTF{secret}'

  

>>> data[0]

67        # ← satu byte = integer!

  

>>> data[0:3]

b'CTF'

  

>>> [chr(b) for b in data]

['C', 'T', 'F', '{', 's', 'e', 'c', 'r', 'e', 't', '}']

```

  

> [!tip] Latihan Cepat

> ```python

> # Kamu dapat string hex ini: 4c4b5332303236

> # Decode ke teks biasa!

>

> hex_str = "4c4b5332303236"

> result = bytes.fromhex(hex_str).decode("utf-8")

> print(result)   # → LKS2026

> ```

  

---

  

## 📦 BASE64 — Encoding yang Sering Muncul di CTF

  

> [!info] Konsep dari Nol

> **Base64** adalah cara mengubah data biner menjadi **teks yang bisa dikirim lewat media teks** (email, JSON, URL).

> Base64 bukan enkripsi — **siapapun bisa decode** tanpa kunci.

>

> **Ciri khas Base64:**

> - Hanya menggunakan karakter: `A-Z a-z 0-9 + / =`

> - Biasanya diakhiri `=` atau `==` (padding)

> - Panjangnya selalu **kelipatan 4**

> - String `SGVsbG8=` → decode → `Hello`

  

### Kenapa Ada di CTF?

File gambar / binary yang di-embed dalam teks, password tersembunyi, flag yang disamarkan — **sering di-encode base64** karena terlihat seperti "random string" tapi mudah di-decode.

  

### Cheatsheet `base64`

  

```python

import base64

  

# ── ENCODE (teks → base64) ────────────────────────────────────

teks = b"Hello CTF!"

encoded = base64.b64encode(teks)

print(encoded)           # → b'SGVsbG8gQ1RGIa=='

  

# Dari string biasa:

base64.b64encode("Hello".encode())  # → b'SGVsbG8='

  

# ── DECODE (base64 → teks) ────────────────────────────────────

encoded = b"SGVsbG8gQ1RGIa=="

decoded = base64.b64decode(encoded)

print(decoded)           # → b'Hello CTF!'

print(decoded.decode())  # → 'Hello CTF!'

  

# ── HANDLE STRING (bukan bytes) ───────────────────────────────

encoded_str = "SGVsbG8="    # Kadang dikasih sebagai string biasa

decoded = base64.b64decode(encoded_str.encode())

# atau langsung:

decoded = base64.b64decode(encoded_str)  # Python otomatis convert

  

# ── URL-SAFE BASE64 (+ dan / diganti - dan _) ─────────────────

base64.urlsafe_b64encode(b"Hello")   # → b'SGVsbG8='

base64.urlsafe_b64decode(b"SGVsbG8=")  # → b'Hello'

  

# ── DECODE FILE YANG ISINYA BASE64 ────────────────────────────

with open("encoded.txt", "r") as f:

    content = f.read().strip()   # strip() hapus newline

  

decoded = base64.b64decode(content)

  

with open("decoded_output.bin", "wb") as f:

    f.write(decoded)

# Setelah ini, cek magic bytes file output!

  

# ── MULTI-LAYER BASE64 (sering di CTF!) ───────────────────────

data = "U0dWc2JHOD0="     # Double-encoded!

layer1 = base64.b64decode(data)   # → b'SGVsbG8='

layer2 = base64.b64decode(layer1) # → b'Hello'

print(layer2.decode())            # → Hello

```

  

### Contoh Output Nyata

  

```python

>>> import base64

>>> base64.b64encode(b"flag{ini_flagnya}")

b'ZmxhZ3tpbmlfZmxhZ255YX0='

  

>>> base64.b64decode(b'ZmxhZ3tpbmlfZmxhZ255YX0=')

b'flag{ini_flagnya}'

  

# Cara detect kalau itu base64:

>>> import re

>>> teks = "ZmxhZ3tpbmlfZmxhZ255YX0="

>>> bool(re.match(r'^[A-Za-z0-9+/]*={0,2}$', teks))

True

```

  

> [!tip] Latihan Cepat

> ```python

> import base64

>

> # Decode ini → apa isinya?

> misteri = "TFRTRG92YW5nU2Fua3Qh"

> print(base64.b64decode(misteri).decode())

> # Coba sendiri dulu sebelum lihat jawaban!

> # Jawaban: LTSLovangSankt!

> ```

  

---

  

## 🔐 HASHLIB — Hashing & Identifikasi

  

> [!info] Konsep dari Nol

> **Hash** adalah fungsi satu arah: input apapun → output panjang tetap.

> - `"Hello"` → MD5 → `8b1a9953c4611296a827abf8c47804d7`

> - Tidak bisa di-reverse (teoritis)

> - Sama input → **selalu** sama output

> - Beda 1 karakter → output **sangat berbeda**

>

> Di CTF, hash dipakai untuk: verifikasi file, password cracking, challenge-response.

  

### Cheatsheet `hashlib`

  

```python

import hashlib

  

# ── HASH DASAR ────────────────────────────────────────────────

teks = b"Hello"

  

md5    = hashlib.md5(teks).hexdigest()     # → '8b1a9953c4611296...'

sha1   = hashlib.sha1(teks).hexdigest()    # → 'f7ff9e8b7bb2e09b...'

sha256 = hashlib.sha256(teks).hexdigest()  # → '185f8db32921bd46...'

  

# Dari string (harus encode dulu ke bytes):

hashlib.md5("Hello".encode()).hexdigest()

hashlib.md5("Hello".encode("utf-8")).hexdigest()

  

# ── HASH FILE ─────────────────────────────────────────────────

with open("file.bin", "rb") as f:

    data = f.read()

md5_file = hashlib.md5(data).hexdigest()

sha256_file = hashlib.sha256(data).hexdigest()

print(f"MD5:    {md5_file}")

print(f"SHA256: {sha256_file}")

  

# ── CEK INTEGRITAS FILE ───────────────────────────────────────

expected_hash = "d8e8fca2dc0f896fd7cb4cb0031ba249"

actual_hash   = hashlib.md5(open("file.bin","rb").read()).hexdigest()

  

if expected_hash == actual_hash:

    print("File VALID ✓")

else:

    print("File KORUP atau DIMANIPULASI ✗")

  

# ── BRUTE FORCE HASH (CTF Classic!) ───────────────────────────

target = "827ccb0eea8a706c4c34a16891f84e7b"   # MD5 dari password pendek

  

for i in range(10000):

    guess = str(i).encode()

    if hashlib.md5(guess).hexdigest() == target:

        print(f"Password ditemukan: {i}")

        break

# → Password ditemukan: 1234

  

# ── HASH DARI LIST KATA (WORDLIST) ────────────────────────────

wordlist = ["password", "123456", "admin", "secret", "ctf2026"]

target   = "5ebe2294ecd0e0f08eab7690d2a6ee69"

  

for word in wordlist:

    if hashlib.md5(word.encode()).hexdigest() == target:

        print(f"Cracked! Password: {word}")

        break

# → Cracked! Password: secret

```

  

### Identifikasi Jenis Hash

  

```

Panjang hash → jenis:

32 karakter  → MD5

40 karakter  → SHA1

56 karakter  → SHA224

64 karakter  → SHA256

96 karakter  → SHA384

128 karakter → SHA512

```

  

> [!tip] Latihan Cepat

> ```python

> import hashlib

>

> # Hash ini MD5 dari angka 0-9999. Crack!

> target = "c4ca4238a0b923820dcc509a6f75849b"

>

> for i in range(10000):

>     if hashlib.md5(str(i).encode()).hexdigest() == target:

>         print(f"Cracked: {i}")

>         break

> # Jawaban: 1

> ```

  

---

  

## 🔩 STRUCT — Membaca Binary Data Terstruktur

  

> [!info] Konsep dari Nol

> `struct` dipakai untuk membaca data biner yang punya **format terstruktur** — misalnya header file, protocol network, atau custom binary format.

>

> Bayangkan file binary sebagai **formulir yang diisi angka** — struct adalah cara membaca setiap "kolom" formulir itu.

  

### Format String `struct`

  

| Karakter | Arti | Ukuran |

|----------|------|--------|

| `B` | Unsigned byte (0-255) | 1 byte |

| `H` | Unsigned short | 2 byte |

| `I` | Unsigned int | 4 byte |

| `Q` | Unsigned long long | 8 byte |

| `s` | String/bytes | N byte |

| `<` | Little-endian (Intel) | — |

| `>` | Big-endian (Network) | — |

  

> [!info] Little vs Big Endian

> Angka `0x12345678` disimpan berbeda:

> - **Big-endian:** `12 34 56 78` (byte terbesar duluan)

> - **Little-endian:** `78 56 34 12` (byte terkecil duluan)

> PC modern (Intel/AMD) = **little-endian**. Network = **big-endian**.

  

### Cheatsheet `struct`

  

```python

import struct

  

# ── UNPACK (baca dari bytes) ──────────────────────────────────

data = b'\x01\x00\x02\x00\x03\x00\x00\x00'

  

# Baca sebagai 3 unsigned short (little-endian)

hasil = struct.unpack('<HHI', data)

print(hasil)   # → (1, 2, 3)

  

# Baca 4 byte sebagai unsigned int

nilai = struct.unpack('<I', data[0:4])[0]

print(nilai)   # → 2 (little-endian: 01 00 02 00 = 0x00020001 = 131073)

  

# ── PACK (tulis ke bytes) ─────────────────────────────────────

packed = struct.pack('<HHI', 1, 2, 3)

print(packed.hex())   # → 010002000300 0000

  

# ── BACA HEADER FILE CUSTOM ───────────────────────────────────

# Contoh: file punya header format:

# - 4 byte: magic number

# - 2 byte: version

# - 4 byte: file size

# - 10 byte: nama file

  

with open("custom.bin", "rb") as f:

    header_data = f.read(20)

  

magic, version, filesize = struct.unpack('>4sHI', header_data[:10])

filename = header_data[10:20].rstrip(b'\x00').decode()

  

print(f"Magic:    {magic}")

print(f"Version:  {version}")

print(f"Size:     {filesize}")

print(f"Filename: {filename}")

  

# ── STRUCT DARI HEX STRING ────────────────────────────────────

raw_hex = "d2040000"

value = struct.unpack('<I', bytes.fromhex(raw_hex))[0]

print(value)   # → 1234  (0x000004D2 = 1234 dalam little-endian)

```

  

> [!tip] Latihan Cepat

> ```python

> import struct

>

> # Kamu dapat 4 byte ini dari file: b'\x2e\x05\x00\x00'

> # Baca sebagai unsigned int little-endian → hasilnya apa?

>

> data = b'\x2e\x05\x00\x00'

> nilai = struct.unpack('<I', data)[0]

> print(nilai)   # Jawaban: 1326

> ```

  

---

  

## 🎯 WORKFLOW — Saat Dapat File Misterius di CTF

  

> [!success] Langkah Kerja Sistematis

  

```

FILE MISTERIUS DATANG

        │

        ▼

┌───────────────────────────────────────┐

│ STEP 1: IDENTIFIKASI TIPE             │

│  $ file mystery.bin                   │

│  $ xxd -l 16 mystery.bin              │

│  Lihat magic bytes → tipe file apa?   │

└──────────────┬────────────────────────┘

               │

               ▼

┌───────────────────────────────────────┐

│ STEP 2: COBA BUKA LANGSUNG            │

│  Kalau PNG/JPG/ZIP/PDF → buka dulu    │

│  Kalau tidak bisa → lanjut step 3     │

│  $ mv mystery.bin mystery.png         │

└──────────────┬────────────────────────┘

               │

               ▼

┌───────────────────────────────────────┐

│ STEP 3: LIHAT ISI MENTAH              │

│  $ xxd mystery.bin | head -20         │

│  $ strings mystery.bin | grep "CTF"   │

│  $ strings mystery.bin | grep "flag"  │

└──────────────┬────────────────────────┘

               │

               ▼

┌───────────────────────────────────────┐

│ STEP 4: CEK ENCODING                  │

│  Lihat apakah ada base64?             │

│  Ada hex string panjang?              │

│  Ada pola XOR / caesar?               │

└──────────────┬────────────────────────┘

               │

               ▼

┌───────────────────────────────────────┐

│ STEP 5: DECODE PAKAI PYTHON           │

│  Script 10 baris → decode → simpan   │

│  Cek hasil → apakah ada flag?         │

└──────────────┬────────────────────────┘

               │

               ▼

┌───────────────────────────────────────┐

│ STEP 6: VERIFIKASI                    │

│  strings output.bin | grep "CTF"      │

│  file output.bin → tipe file apa?     │

└───────────────────────────────────────┘

```

  

### Template Python 10 Baris — Siap Pakai

  

```python

# === TEMPLATE 1: Decode Base64 File ===

import base64

with open("input.b64", "r") as f:

    content = f.read().strip()

decoded = base64.b64decode(content)

with open("output.bin", "wb") as f:

    f.write(decoded)

print(f"Magic bytes: {decoded[:4].hex()}")

print(f"As text: {decoded[:100]}")

  

# === TEMPLATE 2: XOR Brute Force (key 1 byte) ===

with open("input.bin", "rb") as f:

    data = f.read()

for key in range(256):

    result = bytes([b ^ key for b in data])

    if b"CTF" in result or b"flag" in result.lower():

        print(f"Key: {hex(key)} → {result}")

        break

  

# === TEMPLATE 3: Analisis File Lengkap ===

import hashlib, struct

with open("mystery.bin", "rb") as f:

    data = f.read()

print(f"Size:        {len(data)} bytes")

print(f"Magic bytes: {data[:8].hex()}")

print(f"MD5:         {hashlib.md5(data).hexdigest()}")

print(f"As text:     {data[:50]}")

print(f"Strings:     {[s for s in data.split(b'\\x00') if len(s)>4][:5]}")

```

  

---

  

## 🏆 SOAL LATIHAN CTF-STYLE

  

> [!danger] SOAL — File Forensics Challenge

  

**Nama Soal:** `mysterious_data`

**Deskripsi:** Tim Blue Team menerima file mencurigakan dari email phishing. File bernama `data.txt` tapi tidak terbuka. Analisis dan temukan flag.

  

**File `data.txt` berisi:**

```

iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4

BQQMAAIDAQABAAIEAAEEAAEFAAEGAAEHAAEIAAEJAAEKAABXAAPQAAAAElFT

kSuQmCCZmxhZ3toZXhfbWFnaWNfbWFzdGVyfQ==

```

  

**Pertanyaan:**

1. File ini sebenarnya tipe apa?

2. Apa isi flag-nya?

3. Bagaimana cara Python 10 baris untuk decode-nya?

  

---

  

> [!check] JAWABAN LENGKAP

  

**Step 1 — Identifikasi Encoding**

```python

# String panjang berakhir == → ini BASE64!

# Ciri: A-Z a-z 0-9 + / = → PASTI base64

```

  

**Step 2 — Decode Base64**

```python

import base64

  

encoded = """iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4

BQQMAAIDAQABAAIEAAEEAAEFAAEGAAEHAAEIAAEJAAEKAABXAAPQAAAAElFT

kSuQmCCZmxhZ3toZXhfbWFnaWNfbWFzdGVyfQ=="""

  

# Bersihkan whitespace/newline

clean = encoded.replace("\n", "").replace(" ", "")

  

# Decode

decoded = base64.b64decode(clean)

  

# Lihat magic bytes

print(decoded[:8].hex())

# → 89504e470d0a1a0a → INI PNG!

print(f"Tipe file: {decoded[:4]}")

# → b'\x89PNG'

  

# Simpan ke file

with open("output.png", "wb") as f:

    f.write(decoded)

print("File disimpan sebagai output.png")

```

  

**Step 3 — Cari Flag**

```python

# Di akhir data PNG ada teks base64 lagi!

# Kita lihat bagian akhir file yang sudah di-decode

  

print(decoded[-50:])

# → b'...fZmxhZ3toZXhfbWFnaWNfbWFzdGVyfQ=='

  

# Ternyata ada base64 lagi di akhir!

import re

# Cari semua pola base64 dalam binary

matches = re.findall(b'[A-Za-z0-9+/]{20,}={0,2}', decoded)

for m in matches:

    try:

        candidate = base64.b64decode(m).decode()

        if "flag" in candidate or "CTF" in candidate:

            print(f"FLAG DITEMUKAN: {candidate}")

    except:

        pass

  

# → FLAG DITEMUKAN: flag{hex_magic_master}

```

  

**Script Lengkap 10 Baris:**

```python

import base64, re

  

data = open("data.txt").read().replace("\n","")

decoded = base64.b64decode(data)

  

print(f"[*] Magic bytes: {decoded[:4].hex()} → {decoded[:4]}")

open("output.png","wb").write(decoded)

  

matches = re.findall(b'[A-Za-z0-9+/]{16,}={0,2}', decoded)

for m in matches:

    try:

        flag = base64.b64decode(m).decode()

        if any(k in flag for k in ["flag","CTF","LKS"]):

            print(f"[+] FLAG: {flag}")

    except: pass

```

  

**Output:**

```

[*] Magic bytes: 89504e47 → b'\x89PNG'

[+] FLAG: flag{hex_magic_master}

```

  

> [!tip] Insight Soal

> Soal ini melatih **multi-layer**: base64 → PNG → base64 lagi → flag.

> Di CTF nyata, bisa sampai 5+ layer. Kuncinya: **setelah decode, selalu analisis lagi**.

  

---

  

## ⚠️ 3 HAL YANG JANGAN SAMPAI LUPA

  

> [!danger] #1 — Selalu Buka File Biner dengan `"rb"` bukan `"r"`

> ```python

> # SALAH — mode "r" untuk teks biasa

> with open("file.bin", "r") as f:       # ← ERROR atau data corrupt!

>     data = f.read()

>

> # BENAR — mode "rb" untuk file biner

> with open("file.bin", "rb") as f:      # ← rb = read binary

>     data = f.read()

>

> # Aturan: kalau ragu → pakai "rb". Aman untuk semua tipe file.

> ```

  

> [!danger] #2 — `bytes` vs `str` — Jangan Campur Aduk

> ```python

> # SALAH — tidak bisa langsung gabung bytes dan str

> hasil = b"Hello" + " World"    # ← TypeError!

>

> # BENAR — samakan dulu tipenya

> hasil = b"Hello" + " World".encode()   # → b'Hello World'

> hasil = b"Hello".decode() + " World"   # → 'Hello World'

>

> # Cek tipe sebelum operasi:

> type(data)   # → <class 'bytes'> atau <class 'str'>

> ```

  

> [!danger] #3 — Saat Decode Base64, Selalu `strip()` Dulu

> ```python

> # SALAH — newline/spasi bikin base64 error

> data = open("encoded.txt").read()

> result = base64.b64decode(data)    # ← Error: Invalid base64!

>

> # BENAR — hapus whitespace dulu

> data = open("encoded.txt").read().strip()

> result = base64.b64decode(data)    # ← OK

>

> # Untuk data multi-baris:

> data = open("encoded.txt").read().replace("\n","").replace(" ","")

> result = base64.b64decode(data)    # ← OK

> ```

  

---

  

## 📚 QUICK REFERENCE CARD

  

```

┌────────────────────────────────────────────────────────────────┐

│                  MAGIC BYTES CHEATSHEET                        │

├───────────┬──────────────────┬──────────────────────────────── │

│  JPEG     │  FF D8 FF        │  xxd: ffd8 ff                   │

│  PNG      │  89 50 4E 47     │  xxd: 8950 4e47                 │

│  PDF      │  25 50 44 46     │  xxd: 2550 4446 → "%PDF"        │

│  ZIP      │  50 4B 03 04     │  xxd: 504b 0304 → "PK.."        │

│  ELF      │  7F 45 4C 46     │  xxd: 7f45 4c46 → ".ELF"       │

│  GIF      │  47 49 46 38     │  xxd: 4749 4638 → "GIF8"       │

└───────────┴──────────────────┴────────────────────────────────┘

  

┌────────────────────────────────────────────────────────────────┐

│                  KONVERSI CEPAT PYTHON                         │

├────────────────────────────┬───────────────────────────────────┤

│  hex str → bytes           │  bytes.fromhex("4865")           │

│  bytes → hex str           │  b"He".hex()                     │

│  str → bytes               │  "Hello".encode()                │

│  bytes → str               │  b"Hello".decode()               │

│  base64 encode             │  base64.b64encode(b"data")       │

│  base64 decode             │  base64.b64decode("SGVsbG8=")    │

│  hash MD5                  │  hashlib.md5(b"x").hexdigest()   │

│  hash SHA256               │  hashlib.sha256(b"x").hexdigest()│

│  XOR satu byte             │  bytes([b ^ 0x42 for b in data]) │

└────────────────────────────┴───────────────────────────────────┘

  

┌────────────────────────────────────────────────────────────────┐

│                  STRUCT FORMAT STRINGS                         │

├────┬────────────────────┬────────────────────────────────────  │

│ <  │ little-endian      │ Intel/AMD (paling umum)             │

│ >  │ big-endian         │ Network, PowerPC                    │

│ B  │ 1 byte unsigned    │ 0-255                               │

│ H  │ 2 byte unsigned    │ 0-65535                             │

│ I  │ 4 byte unsigned    │ 0-4294967295                        │

│ Q  │ 8 byte unsigned    │ 0-18446744073709551615              │

│ Ns │ N byte string      │ b"ABC..." (N = angka)               │

└────┴────────────────────┴────────────────────────────────────  ┘

  

┌────────────────────────────────────────────────────────────────┐

│                  IDENTIFIKASI HASH                             │

├───────────────────┬────────────────────────────────────────────┤

│  32 char hex      │  MD5                                      │

│  40 char hex      │  SHA1                                     │

│  64 char hex      │  SHA256                                   │

│  128 char hex     │  SHA512                                   │

└───────────────────┴────────────────────────────────────────────┘

```

  

---

  

## 🔗 Senjata Tambahan — One-Liner Favorit CTF

  

```python

# ── STRINGS DALAM BINARY ──────────────────────────────────────

import re

data = open("file.bin", "rb").read()

strings = re.findall(b'[\x20-\x7e]{4,}', data)   # Semua printable string

for s in strings:

    print(s.decode())

  

# ── CARI FLAG PATTERN ─────────────────────────────────────────

flags = re.findall(b'(?:CTF|flag|LKS)\{[^}]+\}', data, re.IGNORECASE)

print(flags)

  

# ── ROT13 ─────────────────────────────────────────────────────

import codecs

print(codecs.decode("synt{ebg_guvegrra}", "rot_13"))   # → flag{rot_thirteen}

  

# ── CAESAR BRUTE FORCE ────────────────────────────────────────

cipher = "iodj{fdhvdu}"

for shift in range(26):

    result = ''.join(chr((ord(c) - 97 - shift) % 26 + 97)

                     if c.isalpha() else c for c in cipher)

    if "flag" in result:

        print(f"Shift {shift}: {result}")

  

# ── HEX DUMP MANUAL DENGAN PYTHON ────────────────────────────

data = open("file.bin", "rb").read()

for i in range(0, min(64, len(data)), 16):

    chunk = data[i:i+16]

    hex_part = ' '.join(f'{b:02x}' for b in chunk)

    asc_part = ''.join(chr(b) if 32 <= b < 127 else '.' for b in chunk)

    print(f'{i:08x}: {hex_part:<48}  {asc_part}')

```

  

---

  

## 🗺️ Peta Besar — Semua Koneksi

  

```

File Misterius

      │

      ├─ xxd → lihat hex → identifikasi magic bytes

      │                              │

      │                    ┌─────────┴──────────┐

      │                 Tipe dikenal?         Tipe tidak dikenal

      │                 (PNG/ZIP/PDF)              │

      │                    │                       │

      │              Buka langsung           Analisis lebih dalam

      │                                            │

      ├─ strings → cari teks tersembunyi           │

      │                                       bytes manipulation

      ├─ base64 → decode → file baru         struct unpack

      │                                       XOR brute force

      ├─ hashlib → verifikasi / crack hash         │

      │                                            ▼

      └─────────────────────────────────────── FLAG! 🚩

```

  

---

  

*📅 Dibuat: 2026-06-04 | LKS 2026 Blue Team Preparation*

*🎯 Next: Part 3 — Network Forensics: Wireshark, pcap analysis, TCP stream*

  

---

#LKS2026 #BlueTeam #Python #CTF #bytes #base64 #hashlib #struct #MagicBytes #FileForensics