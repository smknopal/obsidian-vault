### **1. Serangan Brute Force**

**Definisi:**  
Serangan ini mencoba menebak password dengan mencoba berbagai kombinasi secara sistematis.**Penanganan:**

- **Blokir IP yang mencoba login berulang kali:** Gunakan **Fail2ban** untuk memblokir IP setelah beberapa kali percobaan login gagal.
    
    ```text
    sudo apt install fail2ban
    sudo systemctl enable fail2ban
    sudo systemctl start fail2ban
    ```
    
    Konfigurasi di `/etc/fail2ban/jail.local`:
    
    ```text
    [sshd]
    enabled = true
    port = 22
    maxretry = 5
    bantime = 600
    ```
    

- **Cek percobaan login gagal di log:**
    
    ```text
    grep "Failed password" /var/log/auth.log
    ```
    

- **Gunakan autentikasi berbasis kunci:**
    
    ```text
    ssh-keygen -t rsa -b 4096
    ssh-copy-id user@server
    ```
    

---

### **2. Serangan Denial of Service (DoS)**

**Definisi:**  
Serangan ini membanjiri server dengan lalu lintas palsu sehingga layanan menjadi tidak tersedia.**Penanganan:**

- **Identifikasi lalu lintas mencurigakan:** Gunakan `netstat` atau `ss` untuk melihat koneksi aktif.
    
    ```text
    sudo netstat -tulnp
    sudo ss -tulnp
    ```
    

- **Blokir IP mencurigakan dengan UFW:**
    
    ```text
    sudo ufw deny from <IP_ADDRESS>
    ```
    

- **Batasi koneksi per IP dengan iptables:**
    
    ```text
    sudo iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 10 -j DROP
    ```
    

---

### **3. Serangan SQL Injection**

**Definisi:**  
Penyerang menyisipkan perintah SQL berbahaya melalui input pengguna untuk mencuri atau mengubah data di database 

[](https://www.deltadatamandiri.com/post/kenali-jenis-ancaman-cyber-security-dan-cara-mengatasinya?utm_source=you.com)

 

[](https://badr.co.id/it-infrastruktur/jenis-serangan-cyber/?utm_source=you.com)

.**Penanganan:**

- **Validasi input pengguna:**  
    Pastikan aplikasi memvalidasi input dan menggunakan parameterized queries.
- **Pantau log aplikasi untuk aktivitas mencurigakan:**
    
    ```text
    grep "SELECT" /var/log/apache2/access.log
    ```
    

- **Amankan database dengan firewall:**
    
    ```text
    sudo ufw allow from <trusted_IP> to any port 3306
    ```
    

---

### **4. Serangan Phishing**

**Definisi:**  
Penipuan yang mencoba mencuri data sensitif dengan menyamar sebagai entitas tepercaya 

[](https://ilmukomunikasi.uma.ac.id/2024/02/21/mengenal-jenis-jenis-serangan-terhadap-jaringan-dan-database-serta-cara-mengatasinya/?utm_source=you.com)

.**Penanganan:**

- **Edukasi pengguna:**  
    Ajarkan untuk tidak mengklik tautan mencurigakan.
- **Gunakan filter email:**  
    Terapkan filter spam di server email.
- **Pantau log email untuk aktivitas mencurigakan:**
    
    ```text
    grep "phishing" /var/log/mail.log
    ```
    

---

### **5. Serangan Malware**

**Definisi:**  
Malware adalah perangkat lunak berbahaya yang dirancang untuk merusak atau mencuri data.**Penanganan:**

- **Scan sistem dengan antivirus:**
    
    ```text
    sudo apt install clamav
    sudo clamscan -r /home
    ```
    

- **Hapus file mencurigakan:**
    
    ```text
    sudo rm -rf /path/to/malware
    ```
    

- **Pantau proses mencurigakan:**
    
    ```text
    ps aux | grep suspicious_process
    ```
    

---

### **6. Serangan Man-in-the-Middle (MITM)**

**Definisi:**  
Penyerang mencegat komunikasi antara dua pihak untuk mencuri atau mengubah data 

[](https://biztechacademy.id/23-jenis-serangan-cybersecurity-dan-cara-menghadapinya-lindungi-diri-anda-dari-ancaman-digital/?utm_source=you.com)

.**Penanganan:**

- **Gunakan HTTPS untuk komunikasi aman:**  
    Pasang sertifikat SSL/TLS di server web.
    
    ```text
    sudo apt install certbot
    sudo certbot --apache
    ```
    

- **Gunakan VPN untuk mengenkripsi lalu lintas:**  
    Pasang OpenVPN:
    
    ```text
    sudo apt install openvpn
    ```
    

---

### **7. Kebocoran Data**

**Definisi:**  
Data sensitif terekspos atau dicuri oleh pihak tidak berwenang 

[](https://ilmukomunikasi.uma.ac.id/2024/02/21/mengenal-jenis-jenis-serangan-terhadap-jaringan-dan-database-serta-cara-mengatasinya/?utm_source=you.com)

.**Penanganan:**

- **Audit log untuk mengetahui penyebab kebocoran:**
    
    ```text
    grep "download" /var/log/apache2/access.log
    ```
    

- **Enkripsi data sensitif:**
    
    ```text
    openssl enc -aes-256-cbc -salt -in file.txt -out file.txt.enc
    ```
    

- **Batasi akses ke file sensitif:**
    
    ```text
    chmod 600 sensitive_file
    ```
    

---

### **Langkah Umum Penanganan Serangan Cyber**

1. **Identifikasi serangan:**  
    Gunakan log sistem untuk mendeteksi aktivitas mencurigakan.
    
    ```text
    tail -f /var/log/auth.log
    ```
    

2. **Isolasi sistem terdampak:**  
    Cabut koneksi jaringan atau hentikan layanan yang diserang.
    
    ```text
    sudo systemctl stop apache2
    ```
    

3. **Perbaiki celah keamanan:**  
    Perbarui perangkat lunak dan tambal kerentanan.
    
    ```text
    sudo apt update && sudo apt upgrade
    ```
    

4. **Pantau aktivitas lanjutan:**  
    Gunakan tools seperti Wireshark untuk memantau lalu lintas jaringan.
    
    ```text
    sudo apt install wireshark
    sudo wireshark
    ```
    

5. **Dokumentasi dan evaluasi:**  
    Catat semua langkah yang diambil untuk mencegah serangan serupa di masa depan.