````
# 📘 LKS Cyber Security 2026: Forensik Digital & File Carving

## 🎯 STRATEGI LOMBA (SOP)
1. **Identifikasi Awal:** Gunakan perintah `file` dan `strings` untuk menebak jenis file.
2. **PLAN A (Tools Otomatis):** Gunakan `binwalk` atau `foremost` [2]. Ini sangat cepat.
3. **PLAN B (Hex Editor & Python):** Jika Plan A gagal (karena header dirusak/dihapus pembuat soal) [4], buka Hex Editor, perbaiki *header*-nya, atau gunakan skrip Python untuk memotong file secara manual.

---

## 🔑 DAFTAR MAGIC BYTES (CONTEKAN WAJIB)
*Magic Bytes (File Signature)* adalah identitas asli file yang terletak di *offset* 0 (awal file) [5]. Jangan percaya pada ekstensi (seperti `.jpg` atau `.pdf`), periksa *Magic Bytes*-nya!

### 1. File Gambar / Media
*   **JPEG / JPG:**
    *   **Header:** `FF D8 FF E0` atau `FF D8 FF` [6, 7]
    *   **Footer:** `FF D9` [7]
*   **PNG:**
    *   **Header:** `89 50 4E 47 0D 0A 1A 0A` (ASCII: `.PNG....`) [7, 8]
    *   **Footer:** `49 45 4E 44 AE 42 60 82` (ASCII: `IEND`) [9]
*   **GIF:**
    *   **Header:** `47 49 46 38 37 61` atau `47 49 46 38 39 61` (ASCII: `GIF87a` / `GIF89a`) [7, 8]
*   **BMP:**
    *   **Header:** `42 4D` (ASCII: `BM`) [7, 10]. *(Catatan: BMP tidak punya footer tetap karena ukurannya dideklarasikan di header [7, 10])*

### 2. File Arsip & Dokumen
*   **PDF:**
    *   **Header:** `25 50 44 46` (ASCII: `%PDF`) [7, 8]
    *   **Footer:** `25 25 45 4F 46` (ASCII: `%%EOF`) [7, 11]
*   **ZIP / DOCX / XLSX / APK:**
    *   **Header (Local File):** `50 4B 03 04` (ASCII: `PK..`) [7, 8]
    *   **Penanda Akhir (EOCD):** `50 4B 05 06` [9, 12]
*   **GZIP:**
    *   **Header:** `1F 8B` [7, 8]

### 3. Executable & Malware (Penting untuk Analisis Malware)
*   **Windows EXE (DOS MZ):** `4D 5A` (ASCII: `MZ`) [7, 8]
*   **Linux Executable (ELF):** `7F 45 4C 46` (ASCII: `.ELF`) [7, 8]

### 4. Jaringan & Database
*   **PCAP (Wireshark):** `D4 C3 B2 A1` atau `A1 B2 C3 D4` [13]
*   **PCAPNG:** `0A 0D 0D 0A` [13]
*   **SQLite (.db):** `53 51 4C 69 74 65 20 66 6F 72 6D 61 74 20 33 00` (ASCII: `SQLite format 3\0`) [7]

---

## 🛠️ CHEAT SHEET TOOLS OTOMATIS (PLAN A)
Jalankan perintah ini di terminal Linux (Kali Linux / Parrot OS). Tools ini bekerja dengan mencari pola *header-footer* (*Magic Bytes*) di tumpukan data mentah tanpa peduli pada sistem file [1, 14].

### 1. `strings`
**Fungsi:** Mengeluarkan semua teks manusia yang bisa dibaca dari dalam file biner. Sangat berguna untuk mencari Flag LKS secara cepat atau melihat jenis file.
*   **Perintah:** `strings nama_file_soal.bin | grep "LKS{"`
*   **Perintah:** `strings nama_file_soal.bin | head -n 20` (Melihat bagian atas file)

### 2. `binwalk`
**Fungsi:** Alat paling canggih untuk CTF. Mencari file yang disembunyikan di dalam file lain (seperti *firmware* atau *Steganography*) [2].
*   **Perintah Analisis:** `binwalk nama_file_soal.bin` (Hanya melihat daftar file yang ada di dalamnya).
*   **Perintah Ekstrak:** `binwalk -e nama_file_soal.bin` (Mengekstrak otomatis semua file yang ditemukan ke dalam folder baru `_nama_file_soal.extracted`).

### 3. `foremost`
**Fungsi:** Spesialis *file carving* klasik menggunakan pencocokan *header* dan *footer* [1, 2].
*   **Perintah Ekstrak:** `foremost -i nama_file_soal.bin -o folder_hasil`
*   **Perintah Ekstrak Spesifik:** `foremost -t jpeg,pdf -i nama_file_soal.bin` (Hanya mengekstrak JPG dan PDF).

### 4. `photorec`
**Fungsi:** Alat *Smart Carving*. Sangat tangguh untuk me- *recover* file media dari disk/flashdisk yang sistem filenya rusak (corrupt) atau terfragmentasi [1, 2]. Dioperasikan menggunakan menu interaktif di terminal [2].
*   **Perintah:** `photorec nama_file_soal.img`

---

## 🐍 PYTHON SCRIPTS (PLAN B - BYPASS TOOLS GAGAL)
Tools otomatis seperti `binwalk` akan GAGAL jika file tidak memiliki *footer* yang jelas, file terfragmentasi, atau pembuat soal sengaja mengubah *Magic Bytes*-nya [3, 4]. Gunakan skrip di bawah untuk memanipulasi byte secara paksa.

### Script 1: Manual Slicer (Jika Binwalk Gagal Ekstrak)
**Kasus:** Binwalk melihat ada file ZIP di offset `15000`, tapi gagal mengekstraknya karena *junk bytes* (sampah).
**Solusi:** Potong paksa file tersebut dari offset `15000` sampai habis menggunakan Python.

```python
def potong_paksa_file(file_input, offset_awal, file_output):
    try:
        with open(file_input, 'rb') as f:
            # Pindahkan kursor baca langsung ke titik offset yang diinginkan
            f.seek(offset_awal)
            # Baca dari titik tersebut sampai akhir file
            data_ditemukan = f.read()
            
        with open(file_output, 'wb') as f_out:
            f_out.write(data_ditemukan)
            
        print(f"[+] Sukses memotong! File disimpan sebagai {file_output}")
    except Exception as e:
        print(f"[-] Terjadi kesalahan: {e}")

# Cara pakai: Ganti offset 15000 dengan angka yang kamu temukan dari analisa Hex Editor
potong_paksa_file('soal.bin', 15000, 'hasil_paksa.zip')
````

Script 2: Header Patcher (Reparasi Magic Bytes)

**Kasus:** Pembuat soal jahil. Mereka mengubah header JPEG `FF D8 FF` menjadi `00 00 00`. Binwalk buta dan tidak menemukan apa-apa. **Solusi:** Gunakan Python untuk mencari pola _footer_ JPEG (`FF D9`), lalu timpa bagian depannya dengan _header_ yang benar secara artifisial.

```
def perbaiki_header_jpeg(file_input, file_output):
    # Header JPEG asli yang seharusnya
    header_asli = b'\xff\xd8\xff\xe0'
    
    with open(file_input, 'rb') as f:
        data = f.read()
        
    # Cari posisi footer JPEG yang mungkin masih utuh
    footer_pos = data.find(b'\xff\xd9')
    
    if footer_pos != -1:
        print(f"[!] Footer JPEG ditemukan di offset {footer_pos}")
        
        # Asumsi: Gambar mungkin ukurannya sekitar 5MB ke belakang dari footer
        # Kita ambil datanya, dan paksa tempelkan Header Asli di depannya
        start_pos = max(0, footer_pos - 5000000)
        data_gambar = data[start_pos:footer_pos + 2]
        
        # Timpa 4 byte pertama dengan Header JPEG yang benar
        data_diperbaiki = header_asli + data_gambar[4:]
        
        with open(file_output, 'wb') as f_out:
            f_out.write(data_diperbaiki)
        print(f"[+] Header diperbaiki! Coba buka {file_output}")
    else:
        print("[-] Footer tidak ditemukan.")

# perbaiki_header_jpeg('file_rusak.bin', 'gambar_fix.jpg')
```

Script 3: Memory-Safe Advanced Carver (Untuk File Raksasa)

**Kasus:** Diberikan file _disk image_ ukuran 5 GB. Jika menggunakan script biasa `f.read()`, RAM laptop langsung _crash_ / penuh. **Solusi:** Membaca secara terpotong (_chunking_) menggunakan arsitektur _Sliding Window_ dengan _Overlap_. Overlap mencegah _Magic Bytes_ terpotong di tengah-tengah antar blok memori.

```
def carver_aman_memori(filepath, target_header, target_footer):
    chunk_size = 65536  # Baca per 64KB
    overlap_size = max(len(target_header), len(target_footer)) - 1 # Cegah signature robek [17]
    
    is_recording = False
    ekstrak_buffer = bytearray()
    
    with open(filepath, 'rb') as f:
        buffer = b''
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
                
            # Sliding Window: Gabung sisa buffer sebelumnya dengan chunk baru [17, 19]
            window = buffer + chunk
            
            if not is_recording:
                pos_awal = window.find(target_header)
                if pos_awal != -1:
                    is_recording = True
                    ekstrak_buffer.extend(window[pos_awal:])
            else:
                pos_akhir = window.find(target_footer)
                if pos_akhir != -1:
                    ekstrak_buffer.extend(window[:pos_akhir + len(target_footer)])
                    
                    # Simpan file yang berhasil di-carving
                    with open("hasil_carve_aman.jpg", 'wb') as f_out:
                        f_out.write(ekstrak_buffer)
                    print("[+] File raksasa berhasil di-carving tanpa RAM crash!")
                    return
                else:
                    ekstrak_buffer.extend(chunk)
            
            buffer = chunk[-overlap_size:]

# Cara Pakai: 
# carver_aman_memori('disk_5GB.img', b'\xff\xd8\xff', b'\xff\xd9')
```

```
*** 

**Penjelasan Singkat Cara Menerapkan SOP Saat Lomba Nanti:**
1. Masuk ke soal forensik, download *evidence*. 
2. Pertama, langsung tembak pakai **`binwalk -e`** atau **`foremost`**. Sambil tools ini berjalan (biasanya butuh beberapa menit untuk file besar), Anda periksa file tersebut pakai Hex Editor atau perintah `strings`.
3. Jika alat otomatis mengeluarkan folder berisi gambar/dokumen utuh, langsung cari letak tulisan "Flag" (misal: `LKS{...}`).
4. Jika alat tersebut selesai tapi hasilnya nihil (gagal/rusak), buka Obsidian ini, pelajari *Magic bytes*-nya, cek di Hex Editor bagian mana yang dirusak oleh pembuat soal, lalu jalankan **Python Skrip Manual (Script 1 atau Script 2)** untuk memperbaikinya secara bedah mesin.
```