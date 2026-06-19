````
# 🛠️ Script File Carving LKS Cyber Security 2026
**Tujuan:** Mengekstrak file (gambar/dokumen) yang disembunyikan di dalam file soal yang rusak/mencurigakan.

## 📝 DAFTAR MAGIC BYTES (CONTEKAN)
Saat lomba, kamu hanya perlu **copy-paste** kode di bawah ini ke bagian `HEADER` dan `FOOTER` di dalam script Python.

**1. Gambar JPEG / JPG** (Paling sering keluar) [2, 3]
- **HEADER:** `b'\xff\xd8\xff'`
- **FOOTER:** `b'\xff\xd9'`

**2. Gambar PNG** [2, 3]
- **HEADER:** `b'\x89\x50\x4e\x47\x0d\x0a\x1a\x0a'`
- **FOOTER:** `b'\x49\x45\x4e\x44\xae\x42\x60\x82'`

**3. Dokumen PDF** [2, 4]
- **HEADER:** `b'\x25\x50\x44\x46'`
- **FOOTER:** `b'\x25\x25\x45\x4f\x46'`

---

## 💻 SCRIPT PYTHON UTAMA
*Simpan kode ini dengan nama `carver.py`. Jalankan di terminal dengan perintah `python carver.py`.*

```python
import os

# =====================================================================
# BAGIAN 1: YANG HARUS KAMU UBAH (SESUAIKAN DENGAN SOAL LOMBA)
# =====================================================================

# 1. GANTI dengan nama file soal yang kamu download dari portal CTFd panitia.
NAMA_FILE_SOAL = "evidence.bin"

# 2. GANTI dengan nama file hasil yang kamu inginkan (bebas, tapi ingat ekstensinya!)
NAMA_FILE_HASIL = "bendera_ketemu.jpg"

# 3. GANTI Header dan Footer di bawah ini menggunakan "Daftar Magic Bytes" di atas.
# (Contoh di bawah ini adalah untuk mencari gambar JPEG)
HEADER = b'\xff\xd8\xff'
FOOTER = b'\xff\xd9'


# =====================================================================
# BAGIAN 2: MESIN CARVING (JANGAN UBAH ATAU HAPUS KODE DI BAWAH INI!)
# =====================================================================
try:
    print(f"[*] Membuka file soal: {NAMA_FILE_SOAL} ...")
    # Membaca file sebagai biner mentah [2]
    with open(NAMA_FILE_SOAL, "rb") as file_mentah:
        data_biner = file_mentah.read()

    print("[*] Mencari letak Header di dalam file...")
    # Mencari posisi header di tumpukan data [1]
    posisi_awal = data_biner.find(HEADER)

    if posisi_awal != -1:
        print(f"[+] Hore! Header ditemukan di posisi: {posisi_awal}")
        
        print("[*] Mencari letak Footer...")
        # Mulai cari footer dari posisi header ditemukan [1]
        posisi_akhir = data_biner.find(FOOTER, posisi_awal)
        
        if posisi_akhir != -1:
            print(f"[+] Hore! Footer ditemukan di posisi: {posisi_akhir}")
            
            # MENGGUNTING DATA: Dari posisi awal sampai posisi akhir + panjang footer [1, 2]
            data_hasil = data_biner[posisi_awal : posisi_akhir + len(FOOTER)]
            
            print(f"[*] Menyimpan file hasil potongan menjadi {NAMA_FILE_HASIL} ...")
            # Menyimpan file baru [1]
            with open(NAMA_FILE_HASIL, "wb") as file_jadi:
                file_jadi.write(data_hasil)
                
            print("\n[+] ==========================================")
            print(f"[+] MISI BERHASIL! Buka file {NAMA_FILE_HASIL}")
            print("[+] ==========================================")
        else:
            print("[-] GAGAL: Header ketemu, tapi Footer file tidak ditemukan (file mungkin terpotong).")
    else:
        print("[-] GAGAL: File yang kamu cari tidak ada di dalam evidence ini.")

except FileNotFoundError:
    print(f"[-] ERROR BESAR: File '{NAMA_FILE_SOAL}' tidak ditemukan!")
    print("    Pastikan file soal berada di dalam folder yang sama dengan script Python ini.")

````

📖 CARA PENGGUNAAN SAAT LOMBA:

1. Saat panitia memberikan file soal (misalnya `misteri.bin`), pindahkan file tersebut ke folder yang sama dengan script `carver.py` milikmu.
2. Buka script `carver.py`, lalu **Ubah Bagian 1**:
    - Ganti `NAMA_FILE_SOAL` menjadi `"misteri.bin"`.
    - Jika kamu curiga itu adalah file gambar PNG, ubah `HEADER` dan `FOOTER` dengan kode hex PNG dari daftar di atas, lalu ubah nama hasil menjadi `"hasil.png"`.
3. Save, lalu jalankan script-nya.
4. Jika tertulis **MISI BERHASIL**, langsung buka gambar/dokumen yang dihasilkan dan cari teks Flag-nya!

```
***

### Ringkasan untuk Anda:
*   **Bagian 1 (Yang Diubah):** Itulah satu-satunya tempat Anda bekerja. Anda hanya mengganti nama file input, nama file output, dan copas dua baris *Magic Bytes* berdasarkan format apa yang ingin Anda cari [2, 3].
*   **Bagian 2 (Yang Tidak Boleh Dihapus):** Itu adalah "mesin" yang secara otomatis akan menghitung panjang *bytes*, melakukan pencarian (*find*), menggunting (*slicing*), dan menyimpan (menulis ulang) file tanpa Anda harus pusing memikirkan perhitungan ukurannya [1, 2]. 

Anda bisa langsung meng-copy seluruh blok di atas ke dalam Obsidian Anda! Semoga sukses dengan persiapan LKS-nya. Jika butuh format lain ditambahkan ke daftar, beri tahu saya!
```