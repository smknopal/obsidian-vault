### 

|Perintah|Kegunaan Singkat|
|---|---|
|`cat /var/log/syslog`|Tampilkan seluruh isi log sistem|
|`less /var/log/syslog`|Baca log sistem dengan scroll dan navigasi|
|`tail /var/log/syslog`|Lihat 10 baris terakhir log sistem|
|`tail -f /var/log/syslog`|Pantau log sistem secara real-time|
|`grep "keyword" /var/log/syslog`|Cari baris yang mengandung kata kunci tertentu|
|`journalctl`|Lihat log systemd (modern Linux)|
|`journalctl -xe`|Lihat log systemd dengan detail error dan event terbaru|
|`journalctl -u <service>`|Lihat log untuk service tertentu (misal sshd)|
|`dmesg`|Lihat pesan kernel saat boot dan hardware events|
|`last`|Lihat riwayat login pengguna|
|`lastb`|Lihat riwayat login gagal|
|`ausearch`|Cari log audit (SELinux atau auditd)|
|`grep "Failed password" /var/log/auth.log`|Cari percobaan login gagal SSH|
|`grep "sudo" /var/log/auth.log`|Cari aktivitas sudo|

Export as CSV

---

### Catatan Kegunaan

- **`cat` dan `less`** digunakan untuk membaca file log secara keseluruhan atau dengan navigasi.
- **`tail` dan `tail -f`** sangat berguna untuk memantau log secara real-time saat troubleshooting.
- **`grep`** membantu mencari pola atau kejadian tertentu dalam log yang sangat panjang.
- **`journalctl`** adalah tool modern untuk melihat log systemd, menggantikan banyak file log tradisional.
- **`dmesg`** berguna untuk melihat pesan kernel, terutama saat debugging hardware atau boot issues.
- **`last` dan `lastb`** membantu melihat aktivitas login dan login gagal, penting untuk audit keamanan.
- **`ausearch`** digunakan untuk audit keamanan dan compliance, terutama jika SELinux atau auditd aktif


### Cheatsheet Melihat dan Menganalisis Log di Linux

|Perintah|Kegunaan & Fungsi|Contoh Penggunaan & Penjelasan|
|---|---|---|
|`cat /var/log/syslog`|Menampilkan seluruh isi log sistem secara penuh|`cat /var/log/syslog` → Menampilkan semua pesan log sistem, berguna untuk melihat semua aktivitas sistem tanpa filter.|
|`less /var/log/syslog`|Membaca log panjang dengan navigasi scroll dan pencarian|`less /var/log/syslog` → Mudah membaca log besar dengan kemampuan scroll dan pencarian kata kunci.|
|`tail /var/log/syslog`|Menampilkan 10 baris terakhir log sistem|`tail /var/log/syslog` → Melihat aktivitas terbaru pada sistem.|
|`tail -f /var/log/syslog`|Memantau log secara real-time, menampilkan baris baru saat muncul|`tail -f /var/log/syslog` → Berguna untuk troubleshooting saat ingin memonitor log saat kejadian berlangsung.|
|`grep "keyword" /var/log/syslog`|Mencari baris yang mengandung kata kunci tertentu|`grep "error" /var/log/syslog` → Menemukan pesan error dalam log sistem.|
|`journalctl`|Melihat log systemd (untuk sistem yang menggunakan systemd)|`journalctl` → Menampilkan semua log yang dikelola systemd.|
|`journalctl -xe`|Melihat log systemd dengan detail error dan event terbaru|`journalctl -xe` → Berguna untuk debugging error sistem secara lebih mendalam.|
|`journalctl -u sshd`|Melihat log untuk service tertentu, misal sshd|`journalctl -u sshd` → Memantau log terkait SSH daemon.|
|`dmesg`|Menampilkan pesan kernel dan hardware events saat booting|`dmesg` → Berguna untuk memeriksa masalah hardware atau driver saat boot.|
|`last`|Menampilkan riwayat login pengguna|`last` → Melihat siapa saja yang login ke sistem dan kapan.|
|`lastb`|Menampilkan riwayat login gagal|`lastb` → Mengidentifikasi percobaan login yang gagal, penting untuk audit keamanan.|
|`ausearch`|Mencari log audit (SELinux atau auditd)|`ausearch -m avc` → Mencari pesan audit SELinux terkait akses yang dibatasi.|
|`grep "Failed password" /var/log/auth.log`|Mencari percobaan login SSH yang gagal|`grep "Failed password" /var/log/auth.log` → Menemukan upaya login SSH yang gagal, indikasi serangan brute force.|
|`grep "sudo" /var/log/auth.log`|Mencari aktivitas sudo|`grep "sudo" /var/log/auth.log` → Melihat aktivitas penggunaan sudo, penting untuk audit aktivitas admin.|

Export as CSV

---

### Penjelasan dan Fungsi Perintah

- **`cat` & `less`**: Untuk membaca file log secara langsung, `less` cocok untuk file besar karena bisa scroll dan cari kata.  
    _Contoh:_ `less /var/log/syslog` untuk membaca log sistem dengan mudah.

- **`tail` & `tail -f`**: Melihat baris terakhir atau memantau file log secara real-time.  
    _Contoh:_ `tail -f /var/log/syslog` untuk memantau aktivitas terbaru saat troubleshooting.

- **`grep`**: Memfilter log berdasarkan kata kunci penting untuk menemukan kejadian spesifik.  
    _Contoh:_ `grep "error" /var/log/syslog` untuk mencari pesan error.

- **`journalctl`**: Melihat log dari systemd, menggantikan log tradisional di banyak distro modern.  
    _Contoh:_ `journalctl -u sshd` untuk melihat log SSH.

- **`dmesg`**: Melihat pesan kernel, sangat penting untuk masalah hardware dan booting.  
    _Contoh:_ `dmesg | grep usb` untuk melihat pesan terkait USB.

- **`last` & `lastb`**: Melacak login dan login gagal, membantu audit keamanan.  
    _Contoh:_ `lastb` untuk melihat siapa yang gagal login.

- **`ausearch`**: Untuk audit keamanan lebih dalam, seperti SELinux.  
    _Contoh:_ `ausearch -m avc` untuk melihat akses yang diblokir SELinux.

---

### Contoh Kasus Penggunaan

- **Mendeteksi serangan brute force SSH:**
    
    ```text
    grep "Failed password" /var/log/auth.log | awk '{print $1, $2, $3, $9, $11}'
    ```
    
    → Menampilkan waktu, user, dan IP yang mencoba login gagal.

- **Memantau log realtime saat melakukan percobaan login:**
    
    ```text
    sudo tail -f /var/log/auth.log
    ```
    
    → Melihat langsung log saat percobaan login terjadi.

- **Melihat log error terbaru systemd:**
    
    ```text
    journalctl -xe
    ```
    
    → Membantu menemukan penyebab error sistem terbaru.