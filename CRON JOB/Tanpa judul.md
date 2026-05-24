### Tahap 1: Instalasi dan Modifikasi Web Server

1. **Update repositori dan instal Apache2:**
    
    Bash
    
    ```
    apt update && apt install apache2 -y
    ```
    
2. **Ubah tampilan halaman web:** Ketik `nano /var/www/html/index.html`. Hapus isinya dan ganti dengan HTML berisi identitasmu:
    
    HTML
    
    ```
    <html>
    <body>
        <h1>Selamat Datang di Web Server Praktikum</h1>
        <p>Nama Siswa: [Nama Kamu]</p>
        <p>Kelas: [Kelas Kamu]</p>
        <p>Status: Web Server Berhasil Dikonfigurasi</p>
    </body>
    </html>
    ```
    
    _(Simpan: `Ctrl+O` -> `Enter` -> `Ctrl+X`)_
    

---

### Tahap 2: Pembuatan Script Backup

1. **Buat folder khusus untuk menyimpan hasil cadangan:**
    
    Bash
    
    ```
    mkdir /home/backup_server
    ```
    
2. **Buat file script-nya:**
    
    Bash
    
    ```
    nano /home/cadangkan_web.sh
    ```
    
3. **Masukkan kode script (untuk kompresi dan pencatatan log):**
    
    Bash
    
    ```
    #!/bin/bash
    TANGGAL=$(date +%Y-%m-%d_%H%M)
    tar -czvf /home/backup_server/backup_web_$TANGGAL.tar.gz /var/www/html
    echo "Backup selesai pada $TANGGAL" >> /home/backup_server/log_history.txt
    ```
    
    _(Simpan: `Ctrl+O` -> `Enter` -> `Ctrl+X`)_
    
4. **Beri izin eksekusi agar script bisa berjalan:**
    
    Bash
    
    ```
    chmod +x /home/cadangkan_web.sh
    ```
    

---

### Tahap 3: Otomatisasi dengan Cronjob

1. **Buka pengaturan jadwal sistem:**
    
    Bash
    
    ```
    crontab -e
    ```
    
2. **Tambahkan jadwal pengujian di baris paling bawah:**
    
    Bash
    
    ```
    */2 * * * * /home/cadangkan_web.sh
    ```
    
    _(Ini memerintahkan sistem mengeksekusi script setiap 2 menit)._ _(Simpan: `Ctrl+O` -> `Enter` -> `Ctrl+X`)_
    
3. **Tunggu sekitar 4-6 menit** agar sistem sempat melakukan backup 2-3 kali secara otomatis.
    

---

### Tahap 4: Pengumpulan Bukti (Screenshot)

Setelah menunggu, ambil 4 _screenshot_ (SS) berikut:

- **SS 1 (Web):** Buka _browser_ di Windows/Client, ketik IP Address Debian kamu. Foto halaman web yang menampilkan nama dan kelasmu.
    
- **SS 2 (Jadwal):** Ketik `crontab -e`, foto tampilan baris `*/2 * * * *` di paling bawah.
    
- **SS 3 (Hasil Backup):** Ketik `ls -l /home/backup_server/`, foto daftar file `.tar.gz` yang berhasil terbuat.
    
- **SS 4 (Log):** Ketik `cat /home/backup_server/log_history.txt`, foto catatan waktu backupnya.
    

---

### Tahap 5: Penyusunan Laporan & Pembersihan

1. **Buat Laporan PDF:** Buka Word/Google Docs, masukkan identitas tugasmu, tempel keempat screenshot tadi secara berurutan, beri keterangan singkat di bawahnya, lalu _Save As / Export_ sebagai **PDF**.
    
2. **Matikan Cronjob (SANGAT PENTING):** Kembali ke terminal Debian, buka `crontab -e`, lalu tambahkan tanda pagar `#` di depan baris jadwal tadi (menjadi `# */2 * * * * /home/cadangkan_web.sh`). Simpan dan keluar. _(Ini dilakukan agar memori VM kamu tidak penuh karena sistem terus-terusan melakukan backup setiap 2 menit saat tidak dibutuhkan)._