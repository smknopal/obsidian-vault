### 1. Navigasi & File Management

| Perintah               | Fungsi                         |
| ---------------------- | ------------------------------ |
| `ls`                   | Lihat isi folder               |
| `ls -l`                | Lihat isi folder dengan detail |
| `cd <folder>`          | Pindah ke folder tertentu      |
| `pwd`                  | Tampilkan direktori saat ini   |
| `mkdir <nama_folder>`  | Buat folder baru               |
| `rm <file>`            | Hapus file                     |
| `rm -r <folder>`       | Hapus folder beserta isinya    |
| `cp <sumber> <tujuan>` | Copy file/folder               |
| `mv <sumber> <tujuan>` | Pindah atau rename file/folder |

Export as CSV

---

### 2. Melihat & Mengedit File

| Perintah                       | Fungsi                             |
| ------------------------------ | ---------------------------------- |
| `cat <file>`                   | Tampilkan isi file                 |
| `less <file>`                  | Baca file panjang dengan scroll    |
| `head <file>`                  | Lihat 10 baris awal file           |
| `tail <file>`                  | Lihat 10 baris akhir file          |
| `tail -f <file>`               | Monitor file secara real-time      |
| `nano <file>` / `vim <file>`   | Editor teks                        |
| `grep <kata> <file>`           | Cari kata dalam file               |
| `grep -r <kata> <folder>`      | Cari kata dalam folder rekursif    |
| `awk '{print \$1,\$2}' <file>` | Ambil kolom tertentu dari file     |
| `cut -d ' ' -f 1 <file>`       | Potong dan ambil kolom (delimiter) |

Export as CSV

---

### 3. Proses & Jaringan

| Perintah                       | Fungsi                                |
| ------------------------------ | ------------------------------------- |
| `ps aux`                       | Lihat proses yang berjalan            |
| `top`                          | Monitor proses secara realtime        |
| `kill <PID>`                   | Hentikan proses berdasarkan PID       |
| `ss -tulnp` / `netstat -tulnp` | Lihat port aktif dan proses pendengar |
| `ping <ip/host>`               | Cek koneksi jaringan                  |
| `traceroute <host>`            | Lacak jalur koneksi jaringan          |
| `curl <url>` / `wget <url>`    | Ambil data dari internet              |
| `nmap <ip>`                    | Scan port jaringan                    |

Export as CSV

---

### 4. User & Permission

|Perintah|Fungsi|
|---|---|
|`whoami`|Tampilkan user saat ini|
|`id <user>`|Info user|
|`passwd`|Ganti password|
|`chmod <mode> <file>`|Ubah permission file|
|`chown <user>:<group> <file>`|Ubah pemilik file|
|`sudo <perintah>`|Jalankan perintah dengan hak root|

Export as CSV

---

### 5. Logging & Audit

|Perintah|Fungsi|
|---|---|
|`tail -f /var/log/auth.log`|Monitor log autentikasi secara real-time|
|`journalctl -xe`|Lihat log systemd|
|`last`|Lihat login terakhir|
|`lastb`|Lihat login gagal|
|`dmesg`|Lihat pesan kernel|

Export as CSV

---

### 6. Manipulasi File & Data

|Perintah|Fungsi|
|---|---|
|`file <file>`|Cek tipe file|
|`strings <file>`|Ekstrak teks dari file binary|
|`hexdump -C <file>`|Tampilkan isi file dalam hex|
|`tar -czvf file.tar.gz <folder/file>`|Kompres file/folder|
|`tar -xzvf file.tar.gz`|Ekstrak file archive|
|`base64 -d <file>`|Decode base64|
|`openssl enc -d -aes-256-cbc -in <file>`|Decrypt file (butuh key)|

Export as CSV

---

### 7. Lain-lain

| Perintah  | Fungsi                      |
| --------- | --------------------------- |
| `history` | Lihat riwayat perintah      |
| `alias`   | Buat alias perintah         |
| `date`    | Tampilkan tanggal dan waktu |
| `uptime`  | Lama sistem hidup           |
| `df -h`   | Lihat penggunaan disk       |
| `free -h` | Lihat penggunaan memori     |

Export as CSV

---

#### Tips:

- Sering latihan praktek di VM agar hafal dan cepat pakai.
- Tambahkan catatan khusus sesuai materi lomba yang kamu pelajari.
- Gunakan shortcut `Ctrl + R` di terminal untuk cari perintah di history.