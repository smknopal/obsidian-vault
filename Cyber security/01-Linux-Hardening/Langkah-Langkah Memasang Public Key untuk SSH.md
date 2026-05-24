### 

#### **1. Membuat Public Key dan Private Key di Client**

- Jalankan perintah berikut di terminal client (komputer kamu):
    
    ```text
    ssh-keygen -t rsa -b 4096
    ```
    
    - **Penjelasan:**
        - `-t rsa`: Jenis algoritma kunci (RSA).
        - `-b 4096`: Panjang kunci (4096 bit untuk keamanan tinggi).
    - Kamu akan diminta untuk memilih lokasi penyimpanan kunci. Tekan `Enter` untuk menyimpan di lokasi default (`~/.ssh/id_rsa`).
    - Jika diminta, masukkan **passphrase** (opsional). Passphrase menambah lapisan keamanan, tetapi jika tidak ingin repot, cukup tekan `Enter`.

- **Hasil:**
    - Private key: `~/.ssh/id_rsa` (jangan pernah dibagikan!).
    - Public key: `~/.ssh/id_rsa.pub` (ini yang akan dipasang di server).

---

#### **2. Menyalin Public Key ke Server**

- Gunakan perintah berikut untuk menyalin public key ke server:
    
    ```text
    ssh-copy-id -i ~/.ssh/id_rsa.pub user@server_ip
    ```
    
    - **Penjelasan:**
        - `-i ~/.ssh/id_rsa.pub`: Menentukan file public key yang akan disalin.
        - `user@server_ip`: Ganti dengan username dan IP server tujuan.
    - Kamu akan diminta memasukkan password server untuk pertama kali.

- **Alternatif Manual:** Jika `ssh-copy-id` tidak tersedia, salin isi public key secara manual:
    
    ```text
    cat ~/.ssh/id_rsa.pub
    ```
    
    - Salin outputnya, lalu login ke server:
        
        ```text
        ssh user@server_ip
        ```
        
    - Tambahkan public key ke file `~/.ssh/authorized_keys` di server:
        
        ```text
        echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
        chmod 600 ~/.ssh/authorized_keys
        ```
        

---

#### **3. Menguji Koneksi SSH dengan Public Key**

- Setelah public key dipasang, coba login ke server tanpa password:
    
    ```text
    ssh user@server_ip
    ```
    
    - Jika berhasil, kamu tidak akan diminta memasukkan password lagi.

---

### **Hal-Hal Penting Agar Public Key Berfungsi**

1. **Pastikan File dan Folder Memiliki Izin yang Benar**
    - Di server:
        
        ```text
        chmod 700 ~/.ssh
        chmod 600 ~/.ssh/authorized_keys
        ```
        
    - Di client:
        
        ```text
        chmod 700 ~/.ssh
        chmod 600 ~/.ssh/id_rsa
        ```
        

2. **Pastikan SSH Server Mengizinkan Login dengan Public Key**
    - Edit file konfigurasi SSH di server:
        
        ```text
        sudo nano /etc/ssh/sshd_config
        ```
        
    - Pastikan opsi berikut diatur:
        
        ```text
        PubkeyAuthentication yes
        PasswordAuthentication no
        AuthorizedKeysFile .ssh/authorized_keys
        ```
        
    - Restart SSH server:
        
        ```text
        sudo systemctl restart sshd
        ```
        

3. **Pastikan Tidak Ada Masalah Jaringan**
    - Jika server berada di belakang firewall, pastikan port SSH (default 22) terbuka:
        
        ```text
        sudo ufw allow 22/tcp
        ```
        

4. **Gunakan Passphrase untuk Keamanan Tambahan**
    - Jika kamu memilih menggunakan passphrase saat membuat kunci, setiap kali login kamu akan diminta memasukkan passphrase. Ini menambah lapisan keamanan.

---

### **Troubleshooting Jika Public Key Tidak Berfungsi**

1. **Cek Log SSH di Server**
    - Jika login gagal, periksa log SSH di server:
        
        ```text
        sudo tail -f /var/log/auth.log
        ```
        
    - Cari pesan error seperti "Permission denied" atau "Authentication refused".

2. **Pastikan Public Key Sudah Tersalin dengan Benar**
    - Di server, cek isi file `~/.ssh/authorized_keys`:
        
        ```text
        cat ~/.ssh/authorized_keys
        ```
        
    - Pastikan public key sesuai dengan file `~/.ssh/id_rsa.pub` di client.

3. **Pastikan SSH Server Aktif**
    - Cek status SSH server:
        
        ```text
        sudo systemctl status sshd
        ```
        

4. **Pastikan Tidak Ada Masalah Izin**
    - Periksa izin file dan folder `.ssh` di server dan client (lihat langkah sebelumnya).

---

### **Kesimpulan**

- **Langkah utama:**
    1. Buat public dan private key di client.
    2. Salin public key ke server.
    3. Uji koneksi SSH tanpa password.
- **Hal penting:** Pastikan izin file benar, konfigurasi SSH server mendukung public key, dan tidak ada masalah jaringan.