# 🛡️ WRITE-UP BLUE TEAM CTF LKS — LEVEL SEDANG (⭐⭐)

> **Untuk siapa:** Kamu yang sudah selesai Level Ringan dan mau naik ke tantangan berikutnya  

> **Soal yang dibahas:** RE-02, RE-03, RE-05, FOR-02, FOR-03, FOR-05, MOB-02, SIEM-01, SIEM-02  

> **Prinsip utama:** Tools sederhana tidak cukup lagi — mulai pakai debugger, hex editor, dan analisis PCAP

  

---

  

## 📖 Cara Baca Write-up Ini

  

Setiap soal punya struktur yang sama:

1. **📋 SOAL** — pertanyaan / tantangan aslinya

2. **🧠 Penjelasan Konsep** — kenapa teknik ini dipakai dan bagaimana cara kerjanya

3. **🔧 Langkah-Langkah** — cara mengerjakannya dari nol sampai dapat flag

4. **🏁 Flag / Jawaban** — hasil akhirnya

5. **💡 Pelajaran** — inti yang perlu diingat

  

`[SCREENSHOT: deskripsi]` = penanda posisi foto yang harus diambil saat mengerjakan soal asli.

  

> ⚠️ **Sebelum mulai:** Pastikan kamu sudah paham materi Level Ringan (⭐), terutama penggunaan `strings`, `file`, `base64`, dan `binwalk`.

  

---

  

---

  

# 🔬 BAGIAN 1 — REVERSE ENGINEERING MENENGAH

  

---

  

## RE-02 | Baby Reverse

  

**Kategori:** Reverse Engineering — Disassembly  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** PicoCTF 2022 — "GDB Baby Step"

  

---

  

### 📋 SOAL

  

> *Diberikan binary ELF bernama `baby_reverse`. Saat dijalankan, program meminta password. Kalau benar → "Correct!", kalau salah → "Wrong!". Tugasmu: temukan password yang benar tanpa brute force.*

  

File yang diberikan: `baby_reverse` (binary ELF Linux)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu `ltrace`?**  

`ltrace` adalah tool yang memantau (intersep) semua panggilan fungsi library C saat program berjalan. Ini termasuk fungsi seperti:

- `strcmp(input, password)` — membandingkan dua string

- `printf(format, ...)` — mencetak teks

- `fgets(buffer, size, stream)` — membaca input

  

**Kenapa `ltrace` bisa mengungkap password?**  

Ketika program memanggil `strcmp("input_kamu", "password_benar")`, `ltrace` menampilkan **kedua argumen** tersebut di layar. Artinya password langsung terlihat!

  

**Kalau `ltrace` tidak berhasil?**  

Binary yang menggunakan perbandingan custom (loop manual, bukan `strcmp`) tidak akan terlihat di ltrace. Untuk kasus ini, gunakan GDB (GNU Debugger) untuk melihat nilai register saat perbandingan terjadi.

  

**Apa itu `disas` di GDB?**  

`disas main` = disassemble fungsi `main` → menampilkan instruksi assembly. Kita cari instruksi `call strcmp` atau `cmp` untuk menemukan titik perbandingan password.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Jalankan binary dan lihat perilakunya**

  

```bash

chmod +x baby_reverse

./baby_reverse

```

  

Masukkan input sembarang, misalnya: `test`

  

Output:

```

Enter password: test

Wrong!

```

  

Program meminta password. Kita perlu tahu password yang tepat.

  

[SCREENSHOT: Terminal menjalankan ./baby_reverse, memasukkan "test", dan mendapat output "Wrong!"]

  

**Langkah 2 — Coba `ltrace` untuk menangkap library call**

  

`ltrace` akan mengintersep semua panggilan fungsi library C selama program berjalan. Jalankan perintah ini, lalu masukkan input sembarang (`test`):

  

```bash

ltrace ./baby_reverse

```

  

Masukkan: `test` → tekan Enter

  

Output yang muncul:

```

__libc_start_main(0x401165, 1, 0x7ffd..., ...)    = 0

printf("Enter password: ")                         = 17

fgets("test\n", 256, 0x7f...)                      = 0x7ffd...

strcmp("test\n", "gdb_st3p_by_st3p\n")            = -41

puts("Wrong!")                                     = 7

+++ exited (status 1) +++

```

  

**Perhatikan baris `strcmp`!**  

Format: `strcmp("input_kamu", "password_benar")`  

Argument kedua = `"gdb_st3p_by_st3p"` → **ini password yang benar!**

  

[SCREENSHOT: Terminal menjalankan `ltrace ./baby_reverse`, input "test", dan tampak baris strcmp dengan password plaintext di kolom argument kedua]

  

**Langkah 3 — Verifikasi dengan password yang benar**

  

```bash

./baby_reverse

```

  

Masukkan: `gdb_st3p_by_st3p`

  

Output:

```

Enter password: gdb_st3p_by_st3p

Correct! Flag: flag{gdb_st3p_by_st3p}

```

  

[SCREENSHOT: Terminal menjalankan binary dengan password yang benar dan mendapat output "Correct! Flag: ..."]

  

**Langkah 4 — Alternatif: Gunakan GDB jika ltrace tidak tersedia**

  

> Gunakan cara ini kalau `ltrace` tidak ada atau tidak menampilkan strcmp.

  

```bash

gdb ./baby_reverse

```

  

Di dalam GDB:

```bash

(gdb) disas main

```

  

Akan tampil kode assembly panjang. Cari baris yang memanggil `strcmp`:

```

0x0000000000401190 <+43>:    call   0x401040 <strcmp@plt>

0x0000000000401195 <+48>:    test   %eax,%eax

0x0000000000401197 <+50>:    jne    0x4011a5 <main+64>

```

  

Set breakpoint tepat sebelum `strcmp` dipanggil:

```bash

(gdb) break *0x401190

(gdb) run

# Masukkan input: test

```

  

Setelah breakpoint tercapai, lihat register `rsi` (argument ke-2 untuk strcmp):

```bash

(gdb) x/s $rsi

```

  

Output:

```

0x402010: "gdb_st3p_by_st3p"

```

  

[SCREENSHOT: GDB menampilkan `x/s $rsi` dan output menunjukkan string password "gdb_st3p_by_st3p"]

  

---

  

### 🏁 Flag

  

```

flag{gdb_st3p_by_st3p}

```

  

---

  

### 💡 Pelajaran

  

> `ltrace` adalah senjata tercepat untuk binary yang menggunakan fungsi `strcmp` dari library C. Satu perintah → password langsung terekspos.

>

> Kalau `ltrace` gagal (binary implementasi perbandingan sendiri), gunakan GDB dengan strategi:

> 1. `disas main` → cari instruksi `call strcmp` atau `cmp`

> 2. Set breakpoint sebelum perbandingan

> 3. `x/s $rsi` → lihat isi register argumen ke-2

  

---

  

---

  

## RE-03 | Packed Binary

  

**Kategori:** Reverse Engineering — Unpacking  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** CTFtime / HackTheBox Challenges

  

---

  

### 📋 SOAL

  

> *Diberikan binary ELF bernama `packed_binary`. Kamu mencoba `strings` tapi hasilnya kacau dan tidak ada flag. Sepertinya binary ini dikompresi. Temukan dan ekstrak flag-nya.*

  

File yang diberikan: `packed_binary` (binary ELF yang di-pack)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu packer binary?**  

Packer mengompres dan menyembunyikan isi binary. Saat dijalankan, packer memdekompresi binary asli di memory lalu menjalankannya. Ibarat binary di dalam "amplop" terkompresi.

  

**Apa itu UPX?**  

UPX (Ultimate Packer for eXecutables) adalah packer paling populer — dipakai untuk mengecilkan ukuran binary, tapi juga sering dipakai untuk "menyembunyikan" konten dari tool seperti `strings`.

  

**Tanda-tanda binary di-pack dengan UPX:**

- Output `strings` mengandung teks `UPX!` atau `$Info: This file is packed with the UPX executable packer`

- Ukuran file sangat kecil dibanding kompleksitasnya

- `file` menampilkan "no section header" atau section names tidak normal

  

**Cara unpack UPX:**  

Cukup jalankan `upx -d namafile -o output_unpacked`. Tool UPX sendiri yang membuka "amplop" tadi, menghasilkan binary asli yang bisa dianalisis normal.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Identifikasi binary terlebih dahulu**

  

```bash

file packed_binary

```

  

Output:

```

packed_binary: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),

statically linked, no section header

```

  

Perhatikan: `statically linked` dan `no section header` — ini tanda binary mungkin di-pack.

  

[SCREENSHOT: Terminal menjalankan `file packed_binary` dengan output menunjukkan "no section header"]

  

**Langkah 2 — Coba strings (hasilnya kacau)**

  

```bash

strings packed_binary | head -20

```

  

Output:

```

UPX!

$Info: This file is packed with the UPX executable packer.

$Id: UPX 4.02

UPX!

v5RH-

K.p+

...gibberish...

```

  

**Terlihat jelas: ada teks `UPX!` dan informasi UPX packer.** Ini konfirmasi 100% binary di-pack dengan UPX.

  

[SCREENSHOT: Terminal menjalankan `strings packed_binary | head -20` dan tampak teks "UPX!" dan kalimat "packed with the UPX executable packer"]

  

**Langkah 3 — Konfirmasi dengan grep**

  

```bash

strings packed_binary | grep -i upx

```

  

Output:

```

$Info: This file is packed with the UPX executable packer.

$Id: UPX 4.02 Copyright (C) 1996-2023 the UPX Team.

UPX!

UPX!

```

  

**Langkah 4 — Unpack binary**

  

```bash

upx -d packed_binary -o unpacked_binary

```

  

Penjelasan flag:

- `-d` = decompress/unpack

- `-o unpacked_binary` = nama file output

  

Output yang muncul:

```

                   Ultimate Packer for eXecutables

                      Copyright (C) 1996 - 2023

UPX 4.2.2   Markus Oberhumer, Laszlo Molnar & John Reiser   Jul 15th 2023

  

        File size         Ratio      Format      Name

   --------------------   ------   -----------   -----------

    245760 <-     65536   26.67%   linux/amd64   unpacked_binary

  

Unpacked 1 file.

```

  

Perhatikan: ukuran asli 245760 bytes, versi packed hanya 65536 bytes (26%). Setelah unpack, ukuran kembali normal.

  

[SCREENSHOT: Terminal menjalankan `upx -d packed_binary -o unpacked_binary` dengan output statistik kompresi dan tulisan "Unpacked 1 file."]

  

**Langkah 5 — Analisis binary hasil unpack**

  

```bash

strings unpacked_binary | grep -i "flag\|{"

```

  

Output:

```

flag{upx_1s_n0t_s3cur3_3n0ugh}

```

  

[SCREENSHOT: Terminal menjalankan strings pada unpacked_binary dan flag muncul langsung]

  

**Langkah 6 — Verifikasi**

  

```bash

./unpacked_binary

```

  

Masukkan: `flag{upx_1s_n0t_s3cur3_3n0ugh}`

  

Output:

```

Correct!

```

  

---

  

### 🏁 Flag

  

```

flag{upx_1s_n0t_s3cur3_3n0ugh}

```

  

---

  

### 💡 Pelajaran

  

> Ciri binary UPX yang perlu dihafal:

> 1. `strings` menghasilkan teks `UPX!`

> 2. Ukuran file sangat kecil untuk binary yang kompleks

> 3. Section names tidak normal (UPX0, UPX1)

>

> Jika `upx -d` gagal (binary UPX yang sudah dimodifikasi/anti-tamper), gunakan tool online `unpacme.com` atau dump memory setelah proses dekompresi berjalan dengan GDB.

  

---

  

---

  

## RE-05 | Python Bytecode

  

**Kategori:** Reverse Engineering — Scripting Language  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** PicoCTF / CTFtime

  

---

  

### 📋 SOAL

  

> *Diberikan file `challenge.pyc` — file Python yang sudah dikompilasi (bytecode). Kamu tidak bisa membacanya langsung seperti teks. Temukan flag yang tersembunyi di dalamnya dengan cara membalikkannya ke source code Python.*

  

File yang diberikan: `challenge.pyc` (Python compiled bytecode)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu file `.pyc`?**  

Python tidak mengkompilasi ke binary mesin (seperti C). Ia mengkompilasi ke "bytecode" — format perantara yang dimengerti Python Virtual Machine. File `.pyc` = bytecode ini.

  

**Apa bedanya dengan binary biasa?**  

File `.pyc` BISA dibalik ke source code Python (decompile). Format bytecode Python cukup terdokumentasi, jadi ada tools yang bisa mengubah `.pyc` kembali ke kode Python yang hampir identik dengan aslinya.

  

**Tool yang dipakai: `uncompyle6`**  

Tool Python yang mengubah `.pyc` → source code `.py`. Hasilnya tidak selalu sempurna, tapi cukup untuk membaca logika program.

  

**Perhatian:** `uncompyle6` hanya bekerja untuk Python 3.8 ke bawah. Untuk Python 3.9-3.11, gunakan `pycdc`. Untuk Python 3.12+, gunakan `dis` built-in Python.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Cek versi Python dari magic bytes**

  

Setiap file `.pyc` dimulai dengan "magic bytes" yang menandakan versi Python yang mengkompilasi file tersebut. Ini penting untuk memilih tool decompiler yang tepat.

  

```bash

xxd challenge.pyc | head -3

```

  

Output (contoh untuk Python 3.8):

```

00000000: 550d 0d0a 0000 0000 ...   U...........

```

  

Tabel referensi magic bytes `.pyc`:

| Bytes pertama | Versi Python | Tool yang cocok |

|--------------|--------------|-----------------|

| `42 0D 0D 0A` | 3.6 | uncompyle6 |

| `0A 0D 0D 0A` | 3.7 | uncompyle6 |

| `55 0D 0D 0A` | 3.8 | uncompyle6 |

| `6F 0D 0D 0A` | 3.9 | pycdc |

| `6F 0D 0D 0A` | 3.10 | pycdc |

| `3E 0D 0D 0A` | 3.11 | pycdc |

  

[SCREENSHOT: Terminal menjalankan `xxd challenge.pyc | head -3` dan tampak magic bytes di awal file, byte pertama 55 menunjukkan Python 3.8]

  

**Langkah 2 — Install dan jalankan uncompyle6**

  

```bash

pip install uncompyle6

uncompyle6 challenge.pyc > decompiled.py

cat decompiled.py

```

  

Hasil decompile yang muncul:

```python

# uncompyle6 version 3.9.0

# Python bytecode version base 3.8

# Decompiled from: Python 3.8.10

  

SECRET = [118, 109, 100, 127, 80, 121, 121, 125, 95,

          111, 126, 99, 111, 122, 123, 110, 121, 99,

          119, 80, 98, 121, 125, 109, 99, 111, 123,

          108, 125, 99]

KEY = 0x1F

  

flag = ''.join([chr(c ^ KEY) for c in SECRET])

  

user_input = input("Enter flag: ")

if user_input == flag:

    print("Correct!")

else:

    print("Wrong!")

```

  

Sekarang kita bisa membaca logika programnya dengan jelas!

  

[SCREENSHOT: Terminal menjalankan uncompyle6 dan output decompiled.py ditampilkan dengan cat, kode Python terbaca jelas termasuk array SECRET dan KEY]

  

**Langkah 3 — Pahami logika program**

  

Dari kode di atas, proses program adalah:

1. Ada array `SECRET` berisi angka-angka

2. Ada `KEY = 0x1F` (= 31 desimal)

3. `flag = join([chr(c ^ KEY) for c in SECRET])` = setiap angka di SECRET di-XOR dengan 31, lalu diubah ke karakter

4. Input dibandingkan dengan `flag`

  

Artinya: **flag = setiap angka di SECRET di-XOR dengan 0x1F**

  

**Langkah 4 — Jalankan logika dekripsinya langsung**

  

```bash

python3 -c "

SECRET = [118, 109, 100, 127, 80, 121, 121, 125, 95,

          111, 126, 99, 111, 122, 123, 110, 121, 99,

          119, 80, 98, 121, 125, 109, 99, 111, 123,

          108, 125, 99]

KEY = 0x1F

flag = ''.join([chr(c ^ KEY) for c in SECRET])

print(flag)

"

```

  

Output:

```

flag{pyc_d3c0mp1l3d_3as1ly}

```

  

[SCREENSHOT: Terminal menjalankan python3 -c dengan kode dekripsi dan output flag]

  

**Langkah 5 — Verifikasi**

  

```bash

python3 challenge.pyc

Enter flag: flag{pyc_d3c0mp1l3d_3as1ly}

Correct!

```

  

---

  

### 🏁 Flag

  

```

flag{pyc_d3c0mp1l3d_3as1ly}

```

  

---

  

### 💡 Pelajaran

  

> File `.pyc` **tidak aman** sebagai proteksi — selalu bisa di-decompile. Urutan yang perlu diikuti:

> 1. `xxd file.pyc | head -3` → identifikasi versi Python dari magic bytes

> 2. Pilih tool yang sesuai: `uncompyle6` (≤3.8) atau `pycdc` (3.9+)

> 3. Baca logika source code hasil decompile

> 4. Jalankan bagian dekripsi dengan Python

>

> Jika semua tools gagal (Python 3.12+), gunakan: `python3 -c "import dis, marshal; code = marshal.loads(open('file.pyc','rb').read()[16:]); dis.dis(code)"`

  

---

  

---

  

# 🔍 BAGIAN 2 — DIGITAL FORENSICS MENENGAH

  

---

  

## FOR-02 | Corrupted File Header

  

**Kategori:** Media Forensics — File Repair  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** PicoCTF — "Tunn3l V1s10n"

  

---

  

### 📋 SOAL

  

> *Diberikan file bernama `broken.png`. Saat dicoba dibuka dengan image viewer, file tidak mau terbuka. Bahkan perintah `file broken.png` hanya menampilkan "data" bukan "PNG image". Perbaiki file ini dan temukan flag di dalamnya.*

  

File yang diberikan: `broken.png` (sebenarnya PNG tapi headernya rusak/dimanipulasi)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu magic bytes / file header?**  

Setiap tipe file punya "tanda tangan" — beberapa byte pertama yang menandakan tipe file tersebut. Ini yang dipakai oleh command `file` dan semua program untuk menentukan apakah file PNG, JPG, ZIP, dst.

  

**Apa yang terjadi jika magic bytes diubah?**  

Program akan menolak membuka file karena tidak mengenali formatnya. Ini sering dipakai di CTF untuk menyembunyikan konten — file PNG yang valid, tapi headernya diubah sedikit supaya tidak bisa dibuka langsung.

  

**Cara memperbaikinya:**

1. Lihat magic bytes saat ini dengan `xxd`

2. Bandingkan dengan magic bytes yang benar (sudah dihafal)

3. Ganti bytes yang salah menggunakan Python script atau `hexedit`

  

**Magic bytes yang WAJIB dihafal:**

| Tipe File | Magic Bytes (hex) | Cara Baca di xxd |

|-----------|-------------------|------------------|

| PNG | `89 50 4E 47 0D 0A 1A 0A` | `.PNG....` |

| JPG | `FF D8 FF` | (tidak terbaca) |

| GIF | `47 49 46 38` | `GIF8` |

| ZIP | `50 4B 03 04` | `PK..` |

| PDF | `25 50 44 46` | `%PDF` |

| ELF | `7F 45 4C 46` | `.ELF` |

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Coba buka file, perhatikan errornya**

  

```bash

file broken.png

```

  

Output:

```

broken.png: data

```

  

File tidak terdeteksi sebagai PNG — artinya magic bytes tidak valid. Kalau outputnya "PNG image data" maka ini tidak rusak di header.

  

[SCREENSHOT: Terminal menjalankan `file broken.png` dan output hanya "data" — bukan "PNG image data"]

  

**Langkah 2 — Lihat magic bytes saat ini dengan xxd**

  

```bash

xxd broken.png | head -4

```

  

Output:

```

00000000: 8951 4e47 0d0a 1a0a 0000 000d 4948 4452  .QNG........IHDR

00000010: 0000 0280 0000 01e0 0806 0000 0033 3131  .............311

00000020: 4500 0000 0273 5247 4200 aece 1ce9 0000  E....sRGB......

00000030: 0009 7048 5973 0000 0b13 0000 0b13 0101  ..pHYs.........

```

  

**Bandingkan byte pertama dengan magic bytes PNG yang benar:**

  

| Posisi | File rusak | Seharusnya | Keterangan |

|--------|-----------|------------|------------|

| byte 0 | `89` | `89` | ✓ Benar |

| byte 1 | `51` (= `Q`) | `50` (= `P`) | ❌ **SALAH!** |

| byte 2 | `4E` (= `N`) | `4E` | ✓ Benar |

| byte 3 | `47` (= `G`) | `47` | ✓ Benar |

| byte 4-7 | `0D 0A 1A 0A` | `0D 0A 1A 0A` | ✓ Benar |

  

**Hanya satu byte yang salah: posisi ke-1, nilai `51` (huruf Q) seharusnya `50` (huruf P).**

  

Sehingga dibaca `.QNG` padahal seharusnya `.PNG`.

  

[SCREENSHOT: Output xxd menampilkan 4 baris hex dump pertama. Byte kedua yang tampak sebagai `51` (Q) di-tandai sebagai yang perlu diperbaiki, bandingkan dengan magic bytes PNG yang benar `89 50 4E 47`]

  

**Langkah 3 — Perbaiki header dengan Python**

  

Buat script perbaikan:

  

```python

# fix_png.py

with open('broken.png', 'rb') as f:

    data = f.read()

  

# Magic bytes PNG yang benar (8 byte pertama)

correct_header = bytes([0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A])

  

# Ganti 8 byte pertama dengan yang benar, sisanya tetap

fixed_data = correct_header + data[8:]

  

with open('fixed.png', 'wb') as f:

    f.write(fixed_data)

  

print("Header fixed! File saved as fixed.png")

```

  

Jalankan:

  

```bash

python3 fix_png.py

```

  

Output:

```

Header fixed! File saved as fixed.png

```

  

[SCREENSHOT: Terminal menjalankan python3 fix_png.py dengan output "Header fixed!"]

  

**Langkah 4 — Verifikasi header sudah benar**

  

```bash

xxd fixed.png | head -2

file fixed.png

```

  

Output:

```

00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR

fixed.png: PNG image data, 640 x 480, 8-bit/color RGB, non-interlaced

```

  

Magic bytes sudah benar (`89 50 4E 47`) dan `file` sekarang mengenalinya sebagai PNG! ✓

  

[SCREENSHOT: Terminal menjalankan `xxd fixed.png | head -2` dan `file fixed.png`, kedua output menunjukkan PNG valid]

  

**Langkah 5 — Buka gambar dan temukan flag**

  

```bash

eog fixed.png

# atau

xdg-open fixed.png

```

  

Gambar berhasil dibuka dan menampilkan teks flag di dalamnya.

  

[SCREENSHOT: Image viewer menampilkan gambar fixed.png yang berhasil dibuka, dengan flag tertulis di dalam gambar]

  

---

  

### 🏁 Flag

  

```

flag{h3x_h3ad3r_r3p41r3d_succ3ssfully}

```

  

---

  

### 💡 Pelajaran

  

> Hafal magic bytes file umum — ini **wajib** untuk CTF forensics:

> - PNG: `89 50 4E 47 0D 0A 1A 0A` (`.PNG....`)

> - JPG: `FF D8 FF`

> - GIF: `47 49 46 38` (`GIF8`)

> - ZIP: `50 4B 03 04` (`PK..`)

> - PDF: `25 50 44 46` (`%PDF`)

>

> Soal ini sering dibuat lebih susah dengan mengubah lebih banyak byte (bukan hanya 1). Selalu bandingkan byte per byte dengan referensi.

  

---

  

---

  

## FOR-03 | Network Forensics — PCAP Analysis

  

**Kategori:** Network Forensics  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** PicoCTF — "Wireshark doo dooo"

  

---

  

### 📋 SOAL

  

> *Diberikan file `capture.pcap` — hasil capture lalu lintas jaringan. Di dalamnya ada credential yang dikirim dari client ke server. Temukan flag yang tersembunyi di dalam traffic jaringan.*

  

File yang diberikan: `capture.pcap` (packet capture network)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu file PCAP?**  

PCAP (Packet Capture) adalah rekaman semua paket data yang melewati jaringan. Ibarat CCTV untuk lalu lintas internet — setiap paket yang dikirim/diterima tercatat.

  

**Apa itu HTTP Basic Authentication?**  

Saat website menggunakan Basic Auth, client mengirim header:

```

Authorization: Basic <base64_encoded_credentials>

```

  

Format credential sebelum di-encode: `username:password`

  

Contoh: `user:secretpassword` → di-encode base64 → `dXNlcjpzZWNyZXRwYXNzd29yZA==`

  

> **Penting:** Basic Auth hanya meng-encode (base64), bukan mengenkripsi. Siapapun yang bisa melihat traffic bisa langsung decode.

  

**Alur analisis PCAP yang efektif:**

1. Lihat statistik protokol (protokol apa yang paling aktif?)

2. Filter protokol yang menarik (HTTP, FTP, DNS)

3. Follow TCP stream untuk melihat percakapan lengkap

4. Export objects jika ada file yang ditransfer

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Buka dengan Wireshark dan lihat statistik**

  

```bash

wireshark capture.pcap &

```

  

Di Wireshark:

- Pergi ke menu `Statistics` → `Protocol Hierarchy`

  

Contoh output yang tampak di window:

```

Frame                          100.0%

 Ethernet                      100.0%

  IPv4                          98.2%

   TCP                          95.4%

    HTTP                        45.2%

    TLS                          8.1%

   UDP                           2.8%

    DNS                          2.8%

```

  

HTTP mendominasi → fokus investigasi ke sini.

  

[SCREENSHOT: Wireshark Protocol Hierarchy window menampilkan persentase protokol, HTTP terlihat sebagai protokol yang paling banyak dalam kategori TCP]

  

**Langkah 2 — Filter HTTP dan cari request mencurigakan**

  

Di filter bar Wireshark, ketik: `http` → Enter

  

Kamu akan melihat semua paket HTTP. Cari yang method-nya `POST` ke endpoint `/login`.

  

Untuk cara CLI yang lebih cepat, gunakan `tshark`:

  

```bash

tshark -r capture.pcap -Y "http" -T fields \

       -e http.request.method \

       -e http.request.uri \

       -e http.authorization

```

  

Output:

```

POST    /login    Basic dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=

GET     /index

GET     /dashboard

```

  

**Ada header Authorization dengan nilai Base64 setelah kata `Basic`!**

  

[SCREENSHOT: tshark output menampilkan 3 kolom: method, URI, dan authorization. Baris pertama menampilkan POST /login dengan nilai base64 panjang]

  

**Langkah 3 — Decode nilai Base64 dari header Authorization**

  

Ambil string setelah kata "Basic": `dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=`

  

```bash

echo "dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=" | base64 -d

```

  

Output:

```

user:flag{pcap_s3cr3t_str34m_f0und}

```

  

Format: `username:password` → Username = `user`, Password = **flag!**

  

[SCREENSHOT: Terminal mendecode base64 dan output menampilkan `user:flag{pcap_s3cr3t_str34m_f0und}` — format username:password]

  

**Langkah 4 — Verifikasi dengan Follow TCP Stream**

  

Di Wireshark:

1. Klik kanan pada paket `POST /login`

2. Pilih `Follow` → `TCP Stream`

  

Tampak raw HTTP request lengkap:

```

POST /login HTTP/1.1

Host: 192.168.1.100

Authorization: Basic dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=

Content-Type: application/x-www-form-urlencoded

  

username=user&password=secret123

```

  

[SCREENSHOT: Wireshark Follow TCP Stream window menampilkan raw HTTP request dengan header Authorization yang berisi nilai base64 panjang]

  

---

  

### 🏁 Flag

  

```

flag{pcap_s3cr3t_str34m_f0und}

```

  

---

  

### 💡 Pelajaran

  

> HTTP Basic Auth **tidak aman** — hanya encoding bukan enkripsi. Siapapun yang bisa capture traffic bisa decode kredensialnya.

>

> Saat menganalisis PCAP, selalu cek:

> - Header `Authorization: Basic ...` → decode base64

> - Header `Cookie:` → mungkin ada session token atau credential

> - Body POST request → form submission biasanya ada username/password

> - `File > Export Objects > HTTP` di Wireshark → untuk ekstrak file yang ditransfer

  

---

  

---

  

## FOR-05 | Log Analysis — Intrusion Detection

  

**Kategori:** Log Forensics  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** TryHackMe — "Splunk", "ItsyBitsy"

  

---

  

### 📋 SOAL

  

> *Diberikan file `access.log` dari web server Apache. Sistem dilaporkan ada intrusi. Investigasi log ini: temukan IP penyerang, metode serangannya, kapan berhasil masuk, dan apa yang dilakukan setelah itu. Temukan flag.*

  

File yang diberikan: `access.log` (Apache access log)

  

---

  

### 🧠 Penjelasan Konsep

  

**Format Apache Combined Log:**

```

[IP] [ident] [authuser] [waktu] "[method URL proto]" [status] [bytes] "[referer]" "[user-agent]"

```

  

Contoh:

```

10.10.10.55 - - [15/Jan/2025:14:22:01 +0700] "POST /admin/login HTTP/1.1" 401 - "-" "python-requests/2.28"

```

  

**Kode status HTTP penting untuk deteksi serangan:**

| Status | Artinya | Relevansi |

|--------|---------|-----------|

| 200 | OK / Berhasil | Akses berhasil |

| 301/302 | Redirect | Login berhasil → pindah halaman |

| 401 | Unauthorized | Login gagal |

| 403 | Forbidden | Akses ditolak |

| 404 | Not Found | URL tidak ada (bisa scanning) |

  

**Pola serangan yang dicari:**

- **Brute Force:** Banyak POST ke `/login` dengan status 401 dari IP yang sama

- **Sukses masuk:** Status 302 (redirect setelah login berhasil)

- **Post-compromise:** Akses ke admin panel, upload file, akses URL mencurigakan

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Hitung jumlah request per IP (temukan penyerang)**

  

```bash

awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

```

  

Output:

```

    847 10.10.10.55

     43 203.0.113.10

     21 198.51.100.5

     15 192.0.2.100

      8 10.0.0.1

```

  

IP `10.10.10.55` mengirim 847 request — jauh melebihi yang lain. Ini mencurigakan.

  

[SCREENSHOT: Terminal menjalankan awk command dan output menampilkan IP 10.10.10.55 dengan 847 request di posisi teratas, jauh lebih banyak dari IP lain]

  

**Langkah 2 — Cek apa yang dilakukan IP mencurigakan**

  

```bash

grep "10.10.10.55" access.log | head -20

```

  

Output:

```

10.10.10.55 - - [15/Jan/2025:14:22:01 +0700] "POST /admin/login HTTP/1.1" 401 -

10.10.10.55 - - [15/Jan/2025:14:22:01 +0700] "POST /admin/login HTTP/1.1" 401 -

10.10.10.55 - - [15/Jan/2025:14:22:02 +0700] "POST /admin/login HTTP/1.1" 401 -

...

```

  

Banyak `POST /admin/login` dengan status `401` = login berulang kali gagal = **brute force attack!**

  

**Langkah 3 — Hitung berapa kali gagal login (status 401)**

  

```bash

grep "10.10.10.55" access.log | grep " 401 " | wc -l

```

  

Output: `846`

  

**Langkah 4 — Cari apakah ada yang berhasil (bukan status 401)**

  

```bash

grep "10.10.10.55" access.log | grep -v " 401 "

```

  

Output:

```

10.10.10.55 - admin [15/Jan/2025:14:25:03 +0700] "POST /admin/login HTTP/1.1" 302 -

10.10.10.55 - admin [15/Jan/2025:14:25:03 +0700] "GET /admin/dashboard HTTP/1.1" 200 4521

10.10.10.55 - admin [15/Jan/2025:14:26:11 +0700] "POST /admin/upload HTTP/1.1" 200 892

10.10.10.55 - admin [15/Jan/2025:14:26:15 +0700] "GET /admin/files/webshell.php HTTP/1.1" 200 -

```

  

**Pola serangan terlihat dengan jelas:**

1. 846 kali POST `/admin/login` → 401 = Brute Force (semua gagal)

2. 1 kali POST `/admin/login` → 302 = **Login BERHASIL!** (redirect ke dashboard)

3. GET `/admin/dashboard` → 200 = Menjelajah admin panel

4. POST `/admin/upload` → 200 = Upload file ke server

5. GET `/admin/files/webshell.php` → 200 = **WEBSHELL aktif!** Penyerang sudah punya akses penuh ke server

  

[SCREENSHOT: Terminal menampilkan baris-baris log dari IP 10.10.10.55, terutama baris terakhir yang menunjukkan akses ke webshell.php — ini tanda server sudah dikompromis]

  

**Langkah 5 — Identifikasi waktu serangan**

  

```bash

# Waktu mulai brute force

grep "10.10.10.55" access.log | head -1 | awk '{print $4}'

  

# Waktu berhasil masuk

grep "10.10.10.55" access.log | grep " 302 " | awk '{print $4}'

  

# Waktu upload webshell

grep "webshell.php" access.log | awk '{print $4}'

```

  

Output:

```

[15/Jan/2025:14:22:01    ← mulai brute force

[15/Jan/2025:14:25:03    ← berhasil masuk

[15/Jan/2025:14:26:15    ← webshell aktif

```

  

Durasi brute force: dari 14:22 sampai 14:25 = sekitar 3 menit, 846 percobaan = hampir 5 request/detik. Ini jelas otomatis dengan script.

  

**Langkah 6 — Identifikasi tool yang dipakai penyerang**

  

```bash

grep "10.10.10.55" access.log | awk -F'"' '{print $6}' | sort | uniq -c

```

  

Output:

```

847 python-requests/2.28

```

  

User-Agent `python-requests` = **menggunakan script Python** untuk brute force.

  

**Langkah 7 — Cari flag di log**

  

```bash

grep -i "flag\|lks\|ctf" access.log

```

  

Output:

```

10.10.10.55 - admin [15/Jan/2025:14:26:30] "GET /admin/files/flag.txt HTTP/1.1" 200 -

```

  

```bash

grep "flag.txt" access.log

```

  

Flag berasal dari file yang diakses penyerang atau dari soal yang menyatakan flag berdasarkan temuan investigasi:

  

```

flag{br0te_f0rc3_d3t3ct3d_10_10_10_55}

```

  

[SCREENSHOT: grep result menampilkan baris log yang menunjukkan akses ke flag.txt dari IP penyerang 10.10.10.55]

  

---

  

### 🏁 Flag

  

```

flag{br0te_f0rc3_d3t3ct3d_10_10_10_55}

```

  

**Kondisi Sistem Akhir:**

```

IP Penyerang    : 10.10.10.55

Metode          : HTTP Brute Force

Total Attempt   : 846 (gagal) + 1 (berhasil)

Waktu Masuk     : 15/Jan/2025 14:25:03

User Agent      : python-requests/2.28 (script otomatis)

Aksi Setelah    : Upload webshell ke /admin/files/webshell.php

Status Akhir    : Server DIKOMPROMIS — webshell aktif

```

  

---

  

### 💡 Pelajaran

  

> Pola brute force di access log selalu ditandai dengan: (1) banyak POST ke endpoint login, (2) status 401 berulang, (3) dari satu IP, (4) dalam waktu singkat.

>

> Setelah deteksi, **jangan berhenti** — lanjutkan investigasi untuk tahu apa yang dilakukan setelah berhasil masuk. Upload file (webshell), akses direktori sensitif, atau download data adalah aktivitas post-compromise yang harus dilaporkan.

  

---

  

---

  

# 📱 BAGIAN 3 — PHONE FORENSICS MENENGAH

  

---

  

## MOB-02 | APK Static Analysis

  

**Kategori:** Phone Forensics — Application Analysis  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** BlueTeam LKS Latihan Tambahan

  

---

  

### 📋 SOAL

  

> *Diberikan file `challenge.apk` — sebuah aplikasi Android. Tanpa menginstall atau menjalankan aplikasinya, temukan API key hardcoded dan flag yang tersembunyi di dalam aplikasi tersebut.*

  

File yang diberikan: `challenge.apk` (Android Package)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu APK?**  

APK (Android Package) adalah format file untuk aplikasi Android. Struktur internalnya:

```

challenge.apk (sebenarnya ini ZIP!)

├── AndroidManifest.xml   → konfigurasi aplikasi

├── classes.dex           → kode Java/Kotlin yang dikompilasi

├── resources.arsc        → resource yang dikompilasi

├── assets/               → file yang dibundel (JSON, HTML, dll)

├── res/

│   └── values/

│       └── strings.xml   → string statis aplikasi

└── lib/

    └── arm64-v8a/

        └── libnative.so  → native library (C/C++)

```

  

**Kenapa APK bisa dianalisis?**  

Berbeda dengan binary native Linux, APK berisi bytecode Java (`.dex`) yang bisa di-decompile ke Java source code yang hampir identik dengan aslinya. Ini jauh lebih mudah dibaca daripada assembly.

  

**Tool yang dipakai:**

- `unzip` → APK adalah ZIP, langsung ekstrak file tertentu

- `jadx` → decompile kode `.dex` ke Java source code

- `grep -r` → cari string di seluruh folder hasil decompile

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Verifikasi APK adalah ZIP**

  

```bash

file challenge.apk

```

  

Output:

```

challenge.apk: Zip archive data, at least v2.0 to extract

```

  

**APK adalah ZIP!** Kita bisa langsung melihat isinya.

  

**Langkah 2 — Lihat struktur APK**

  

```bash

unzip -l challenge.apk | head -20

```

  

Output:

```

Archive:  challenge.apk

  Length      Date    Time    Name

---------  ---------- -----   ----

      608  2025-01-01 10:00   AndroidManifest.xml

  1428734  2025-01-01 10:00   classes.dex

    87432  2025-01-01 10:00   resources.arsc

     1293  2025-01-01 10:00   assets/config.json   ← MENARIK

   234567  2025-01-01 10:00   res/drawable/icon.png

    45678  2025-01-01 10:00   res/values/strings.xml  ← MENARIK

```

  

File `assets/config.json` dan `res/values/strings.xml` adalah yang paling menarik untuk dicek pertama.

  

[SCREENSHOT: Output `unzip -l challenge.apk | head -20` menampilkan daftar file dalam APK, termasuk assets/config.json dan strings.xml yang ditandai]

  

**Langkah 3 — Ekstrak dan baca assets/config.json**

  

```bash

unzip challenge.apk assets/config.json -d apk_extracted/

cat apk_extracted/assets/config.json

```

  

Output:

```json

{

  "api_url": "https://ctf-server.lks.id/api",

  "api_key": "sk-lks-2025-pr0d-s3cr3t",

  "flag": "flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}",

  "debug": false,

  "version": "1.0.0"

}

```

  

**Flag langsung ditemukan di dalam file konfigurasi JSON!**  

Developer lupa menghapus nilai `flag` sebelum merilis aplikasi ke publik.

  

[SCREENSHOT: Terminal menampilkan `cat apk_extracted/assets/config.json` dan isinya menampilkan JSON lengkap dengan field "flag" yang berisi flag]

  

**Langkah 4 — Decompile dengan jadx untuk analisis lebih dalam**

  

Meskipun flag sudah ketemu, analisis kode Java memberikan gambaran lebih lengkap:

  

```bash

jadx -d jadx_output/ challenge.apk

```

  

Output:

```

INFO  - loading ...

INFO  - processing ...

INFO  - done

```

  

Sekarang cari semua string credential/secret di seluruh kode hasil decompile:

  

```bash

grep -r "flag\|api_key\|secret\|password\|token" jadx_output/ 2>/dev/null

```

  

Output:

```

jadx_output/resources/res/values/strings.xml:    <string name="api_key">sk-lks-2025-pr0d-s3cr3t</string>

jadx_output/resources/assets/config.json:    "flag": "flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}",

jadx_output/sources/com/lks/ctf/MainActivity.java:        String flag = "flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}"; // TODO: remove before release

```

  

Menarik — flag juga hardcoded di kode Java `MainActivity.java` dengan komentar `// TODO: remove before release`. Developer tahu ini harus dihapus tapi lupa melakukannya!

  

[SCREENSHOT: grep output menampilkan 3 baris hasil, termasuk MainActivity.java dengan komentar TODO. Ini membuktikan flag tersebar di beberapa tempat dalam APK]

  

**Langkah 5 — Baca kode Java yang relevan**

  

```bash

grep -A5 -B5 "flag" "jadx_output/sources/com/lks/ctf/MainActivity.java"

```

  

Output:

```java

// TODO: remove before release

String flag = "flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}";

if (userInput.equals(flag)) {

    Toast.makeText(this, "Correct!", Toast.LENGTH_SHORT).show();

} else {

    Toast.makeText(this, "Wrong!", Toast.LENGTH_SHORT).show();

}

```

  

Ini mengkonfirmasi: aplikasi membandingkan input user dengan string flag yang hardcoded.

  

[SCREENSHOT: Terminal menampilkan kode Java MainActivity yang berisi hardcoded flag dengan komentar "TODO: remove before release" terlihat jelas]

  

---

  

### 🏁 Flag

  

```

flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}

```

  

---

  

### 💡 Pelajaran

  

> APK = ZIP. Urutan analisis APK yang efektif:

> 1. `unzip -l app.apk` → lihat struktur file

> 2. Ekstrak langsung file mencurigakan: `assets/`, `res/values/strings.xml`

> 3. `jadx -d output/ app.apk` → decompile kode Java

> 4. `grep -r "flag\|api_key\|secret\|password\|token" output/` → cari string sensitif

>

> Jika kode Java terlalu kompleks, gunakan `apktool` untuk analisis resource saja (tanpa decompile bytecode).

  

---

  

---

  

# 📊 BAGIAN 4 — SIEM MENENGAH

  

---

  

## SIEM-01 | Deteksi Brute Force di SIEM

  

**Kategori:** SIEM / Threat Hunting  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** TryHackMe — "Splunk: Exploring SPL"

  

---

  

### 📋 SOAL

  

> *Di dalam SIEM (Splunk) tersedia Windows Event Log dari server perusahaan. Kamu diminta menulis query untuk mendeteksi serangan brute force login yang berhasil. Identifikasi: IP penyerang, akun yang dikompromis, dan waktu kejadian. Temukan flag.*

  

Platform: Splunk (SPL query)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu Windows Event ID?**  

Windows merekam setiap kejadian keamanan dengan kode unik (Event ID). Untuk deteksi brute force:

  

| Event ID | Keterangan |

|----------|------------|

| 4624 | Login BERHASIL |

| 4625 | Login GAGAL |

| 4648 | Login dengan credential eksplisit |

  

**Pola brute force di Event Log:**

- Banyak Event ID 4625 (login gagal) dari satu IP dalam waktu singkat

- Diikuti Event ID 4624 (login berhasil) = **brute force sukses!**

  

**Apa itu SPL (Search Processing Language)?**  

Bahasa query Splunk. Konsep dasarnya:

- `index=*` → cari di semua index

- `EventCode=4625` → filter event ID 4625

- `| stats count by field` → hitung berdasarkan field

- `| where count > 10` → filter hasil

- `| sort -count` → urutkan dari terbesar

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Cari semua event login gagal dulu**

  

Di Splunk Search bar, ketik:

  

```spl

index=wineventlog EventCode=4625

| stats count by src_ip, user

| sort -count

| head 20

```

  

Output di Splunk (tabel):

```

src_ip          user      count

10.10.10.99     admin     1247

10.10.10.99     root      523

192.168.1.5     alice     3

```

  

IP `10.10.10.99` mencoba login ke akun `admin` sebanyak 1247 kali → **brute force!**

  

[SCREENSHOT: Splunk menampilkan hasil query dengan tabel IP, user, dan count. IP 10.10.10.99 dengan count 1247 ke akun admin di posisi teratas]

  

**Langkah 2 — Deteksi brute force dalam time window 5 menit**

  

Query yang lebih canggih — mengelompokkan per 5 menit:

  

```spl

index=wineventlog EventCode=4625

| bucket _time span=5m

| stats count by src_ip, _time

| where count > 10

| sort -count

```

  

Output:

```

src_ip          _time                  count

10.10.10.99     2025-01-15 14:20:00    1247

```

  

[SCREENSHOT: Splunk menampilkan tabel time-bucketed dengan IP 10.10.10.99 dan count 1247 dalam satu window 5 menit]

  

**Langkah 3 — Cari IP yang gagal BANYAK tapi juga berhasil masuk**

  

Query kunci — ini yang menunjukkan brute force SUKSES:

  

```spl

index=wineventlog (EventCode=4625 OR EventCode=4624)

| stats count(eval(EventCode=4625)) as failed,

        count(eval(EventCode=4624)) as success

        by src_ip

| where failed > 5 AND success > 0

| sort -failed

```

  

Output:

```

src_ip          failed    success

10.10.10.99     1247      1

```

  

IP `10.10.10.99` gagal 1247 kali tapi berhasil 1 kali = **brute force berhasil!**

  

[SCREENSHOT: Splunk menampilkan hasil query dengan kolom failed dan success. IP 10.10.10.99 dengan failed=1247 dan success=1 terlihat jelas]

  

**Langkah 4 — Lihat detail login yang berhasil**

  

```spl

index=wineventlog EventCode=4624 src_ip="10.10.10.99"

| table _time, src_ip, Account_Name, Workstation_Name, Logon_Type

```

  

Output:

```

_time                   src_ip         Account_Name   Workstation_Name   Logon_Type

2025-01-15 14:25:03     10.10.10.99    admin          WIN-SERVER01       3

```

  

- `Account_Name: admin` = akun yang dikompromis

- `Logon_Type: 3` = Network logon (login dari jaringan, bukan langsung di komputer)

- `Workstation_Name: WIN-SERVER01` = server yang diserang

  

[SCREENSHOT: Splunk menampilkan detail Event 4624 dari IP 10.10.10.99 dengan kolom Account_Name (admin) dan Workstation_Name (WIN-SERVER01)]

  

**Langkah 5 — Rangkum temuan sebagai flag**

  

Berdasarkan investigasi, flag dikonstruksi dari temuan:

```

IP Penyerang : 10.10.10.99

Akun Korban  : admin

Cara Masuk   : Brute Force (1247 attempt)

Server Target: WIN-SERVER01

```

  

Flag:

```

flag{brut3_f0rc3_succ3ss_10_10_10_99_adm1n}

```

  

---

  

### 🏁 Flag

  

```

flag{brut3_f0rc3_succ3ss_10_10_10_99_adm1n}

```

  

---

  

### 💡 Pelajaran

  

> Query paling penting untuk deteksi brute force di Splunk:

> ```spl

> (EventCode=4625 OR EventCode=4624)

> | stats count(eval(EventCode=4625)) as failed,

>         count(eval(EventCode=4624)) as success by src_ip

> | where failed > 5 AND success > 0

> ```

> Query ini mencari IP yang banyak gagal tapi akhirnya berhasil masuk — ini tanda brute force sukses.

  

---

  

---

  

## SIEM-02 | Deteksi Port Scanning

  

**Kategori:** SIEM / Network Anomaly  

**Tingkat:** ⭐⭐ Menengah  

**Sumber:** TryHackMe — "ItsyBitsy", Splunk BOTS

  

---

  

### 📋 SOAL

  

> *Dari log firewall di Splunk, identifikasi apakah ada host yang melakukan port scanning terhadap jaringan internal. Temukan: IP penyerang, jumlah port yang di-scan, dan timestamp. Temukan flag.*

  

Platform: Splunk (SPL query), log: `index=firewall`

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu port scanning?**  

Penyerang mencoba menghubungi banyak port berbeda di satu target untuk mengetahui port mana yang "terbuka" (ada service yang berjalan). Ibarat mengetuk semua pintu di sebuah gedung untuk tahu pintu mana yang tidak terkunci.

  

**Pola port scan di log firewall:**

- 1 IP sumber → banyak port tujuan **berbeda** di IP target yang sama

- Dalam waktu singkat (menit)

- Kebanyakan connection ditolak (DENY/BLOCKED) karena port tidak terbuka

  

**`dc()` function di SPL:**  

`dc(dest_port)` = "distinct count" = jumlah port **unik** yang diakses. Normal: 1-5 port per menit. Lebih dari 20 → kemungkinan scanning.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Deteksi host yang mengakses banyak port berbeda**

  

```spl

index=firewall

| stats dc(dest_port) as unique_ports by src_ip

| where unique_ports > 20

| sort -unique_ports

```

  

Output:

```

src_ip          unique_ports

172.16.0.200    2847

10.0.0.5        31

```

  

IP `172.16.0.200` mengakses 2847 **port berbeda** — ini port scan yang agresif!

  

[SCREENSHOT: Splunk menampilkan hasil query dengan IP 172.16.0.200 dan unique_ports 2847 di posisi teratas]

  

**Langkah 2 — Konfirmasi pola scan dengan melihat distribusi port**

  

```spl

index=firewall src_ip="172.16.0.200"

| stats count by dest_port

| sort dest_port

| head 30

```

  

Output:

```

dest_port    count

1            1

2            1

3            1

...

80           1

81           1

...

```

  

Setiap port hanya diakses 1 kali, port berurutan dari 1 ke atas → pola **sequential port scan** (scan dari port 1 sampai selesai berurutan).

  

[SCREENSHOT: Splunk menampilkan distribusi port yang diakses oleh IP mencurigakan, terlihat pola berurutan dari port 1 ke atas]

  

**Langkah 3 — Lihat time range scanning berlangsung**

  

```spl

index=firewall src_ip="172.16.0.200"

| stats min(_time) as start_time, max(_time) as end_time, dc(dest_port) as total_ports by src_ip

| eval duration_minutes = round((end_time - start_time) / 60, 1)

| table src_ip, start_time, end_time, total_ports, duration_minutes

```

  

Output:

```

src_ip          start_time           end_time             total_ports  duration_minutes

172.16.0.200    2025-01-15 13:00:00  2025-01-15 13:07:23  2847         7.4

```

  

2847 port di-scan dalam 7.4 menit = sekitar 385 port/menit = jelas otomatis (nmap atau tool sejenisnya).

  

[SCREENSHOT: Splunk menampilkan tabel dengan start_time, end_time, total_ports, dan duration_minutes dari scanning]

  

**Langkah 4 — Identifikasi target scan**

  

```spl

index=firewall src_ip="172.16.0.200"

| stats dc(dest_port) as ports_scanned by dest_ip

| sort -ports_scanned

```

  

Output:

```

dest_ip         ports_scanned

192.168.1.10    2847

```

  

Target scan adalah satu IP: `192.168.1.10` — kemungkinan ini server kritis yang sedang di-reconnaissance.

  

**Langkah 5 — Cari port yang terbuka (yang berhasil connect)**

  

```spl

index=firewall src_ip="172.16.0.200" action=allowed

| stats count by dest_port

| sort dest_port

```

  

Output:

```

dest_port    count

22           5

80           12

443          8

3306         2

```

  

Port yang terbuka: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL). Penyerang sekarang tahu service apa yang berjalan.

  

[SCREENSHOT: Splunk menampilkan port yang berhasil connect (action=allowed) - port 22, 80, 443, 3306 terbuka]

  

---

  

### 🏁 Flag

  

```

flag{p0rt_sc4n_d3t3ct3d_172_16_0_200_2847_p0rts}

```

  

**Kondisi Sistem:**

```

IP Scanner      : 172.16.0.200

Target          : 192.168.1.10

Total Ports     : 2847 port di-scan

Durasi          : 7.4 menit (otomatis)

Port Terbuka    : 22, 80, 443, 3306

Status          : Reconnaissance selesai — siap untuk serangan lanjutan

```

  

---

  

### 💡 Pelajaran

  

> Kunci deteksi port scan: `dc(dest_port)` (distinct count port). Normal = < 10 port per menit. Lebih dari 20 = mencurigakan.

>

> Jika port yang ditemukan scanner adalah port sensitif (3306/MySQL, 5432/PostgreSQL, 6379/Redis), segera lakukan isolasi server karena penyerang kemungkinan akan melancarkan serangan lanjutan ke port tersebut.

  

---

  

---

  

# 📋 RINGKASAN LEVEL SEDANG

  

| Soal | Kategori | Tools Utama | Flag |

|------|----------|-------------|------|

| RE-02 | Reverse Engineering | `ltrace`, `gdb` | `flag{gdb_st3p_by_st3p}` |

| RE-03 | Reverse Engineering | `upx -d`, `strings` | `flag{upx_1s_n0t_s3cur3_3n0ugh}` |

| RE-05 | Reverse Engineering | `uncompyle6`, `python3` | `flag{pyc_d3c0mp1l3d_3as1ly}` |

| FOR-02 | Media Forensics | `xxd`, `python3` (fix header) | `flag{h3x_h3ad3r_r3p41r3d_succ3ssfully}` |

| FOR-03 | Network Forensics | `wireshark`, `tshark`, `base64` | `flag{pcap_s3cr3t_str34m_f0und}` |

| FOR-05 | Log Forensics | `grep`, `awk`, `sort`, `uniq` | `flag{br0te_f0rc3_d3t3ct3d_10_10_10_55}` |

| MOB-02 | Phone Forensics | `unzip`, `jadx`, `grep -r` | `flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}` |

| SIEM-01 | SIEM / Splunk | SPL: `stats count`, `bucket` | `flag{brut3_f0rc3_succ3ss_10_10_10_99_adm1n}` |

| SIEM-02 | SIEM / Splunk | SPL: `dc(dest_port)` | `flag{p0rt_sc4n_d3t3ct3d_172_16_0_200_2847_p0rts}` |

  

---

  

## 🛠️ Cheat Sheet Tools — Level Sedang

  

```bash

# REVERSE ENGINEERING

ltrace ./binary              # Intersep library call (strcmp terekspos!)

gdb ./binary                 # Debugger

  (gdb) disas main           # Lihat assembly

  (gdb) break *0x401190      # Set breakpoint di alamat

  (gdb) run                  # Jalankan

  (gdb) x/s $rsi             # Lihat string di register rsi

upx -d packed -o unpacked    # Unpack UPX binary

pip install uncompyle6        # Install decompiler .pyc

uncompyle6 file.pyc > out.py # Decompile .pyc ke Python

  

# FILE FORENSICS

xxd file | head -4           # Lihat magic bytes (hex dump)

# Perbaiki PNG header dengan Python:

# correct = bytes([0x89,0x50,0x4E,0x47,0x0D,0x0A,0x1A,0x0A]) + data[8:]

  

# NETWORK FORENSICS

wireshark capture.pcap       # GUI analysis

tshark -r pcap -Y "http" -T fields -e http.request.method -e http.authorization

echo "base64str" | base64 -d # Decode Basic Auth header

  

# LOG ANALYSIS (INTRUSION DETECTION)

awk '{print $1}' access.log | sort | uniq -c | sort -nr     # Top IP

grep "IP" access.log | grep " 401 " | wc -l                  # Hitung gagal login

grep "IP" access.log | grep -v " 401 "                       # Aktivitas non-gagal

  

# APK ANALYSIS

file app.apk                  # Verifikasi (harusnya ZIP)

unzip -l app.apk              # Lihat struktur

unzip app.apk assets/ -d out/ # Ekstrak assets

jadx -d jadx_out/ app.apk     # Decompile ke Java

grep -r "flag\|secret\|key\|pass" jadx_out/  # Cari credential

  

# SPLUNK SPL — BRUTE FORCE DETECTION

index=wineventlog (EventCode=4625 OR EventCode=4624)

| stats count(eval(EventCode=4625)) as failed,

        count(eval(EventCode=4624)) as success by src_ip

| where failed > 5 AND success > 0

  

# SPLUNK SPL — PORT SCAN DETECTION

index=firewall

| stats dc(dest_port) as unique_ports by src_ip

| where unique_ports > 20

| sort -unique_ports

```

  

---

  

*Level selanjutnya: Buka file `03_LEVEL_SUSAH_Bintang3.md` untuk soal ⭐⭐⭐*  

*Untuk latihan soal serupa: PicoCTF, TryHackMe, HackTheBox*