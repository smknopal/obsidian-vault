Markdown

````
# 🛡️ Linux Hardening: PAM & Exposed Services
**Peran:** Defender
**Konteks:** LKS Cyber Security 
**OS:** Debian/Ubuntu-based

---

## 1. Privileged Access Management (PAM)
Fokus: Mencegah eksploitasi via *password* lemah dan serangan *brute-force* internal.

### A. Memaksa Kompleksitas Password
Gunakan `pwquality` untuk memaksa standar *password* yang kuat.

**1. Install Modul**
```bash
sudo apt update
sudo apt install libpam-pwquality -y
````

**2. Konfigurasi Aturan** Buka file konfigurasi:

Bash

```
sudo nano /etc/security/pwquality.conf
```

Tambahkan/sesuaikan baris berikut:

Ini, TOML

```
minlen = 12
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```

> **Penjelasan Parameter:**
> 
> - `minlen`: Minimal panjang password 12 karakter.
>     
> - `dcredit`: Wajib mengandung minimal 1 angka.
>     
> - `ucredit`: Wajib mengandung minimal 1 huruf besar.
>     
> - `lcredit`: Wajib mengandung minimal 1 huruf kecil.
>     
> - `ocredit`: Wajib mengandung minimal 1 simbol.
>     

### B. Account Lockout (pam_faillock)

Gunakan ini sebagai pelapis `fail2ban` untuk mengunci akun spesifik yang diserang.

**1. Edit Konfigurasi Auth** Buka file `common-auth`:

Bash

```
sudo nano /etc/pam.d/common-auth
```

**2. Tambahkan Aturan Lockout** Tambahkan baris berikut **tepat di bawah** baris `auth required pam_env.so`:

Plaintext

```
auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900
```

> **Penjelasan Parameter:** Akun akan otomatis terkunci selama 15 menit (`900` detik) setelah mengalami kegagalan login sebanyak 3 kali (`deny=3`).

---

## 2. Dangerous / Exposed Services

Fokus: Prinsip _Least Privilege_. Matikan port/layanan yang tidak memiliki urgensi atau rentan dieksploitasi.

### A. Identifikasi Layanan yang Terbuka (Listening)

Cek port apa saja yang terbuka ke jaringan publik:

Bash

```
sudo ss -tulnp
```

> **Flags:**
> 
> - `t`: TCP
>     
> - `u`: UDP
>     
> - `l`: Listening ports
>     
> - `n`: Numeric (tidak resolve IP ke nama domain, lebih cepat)
>     
> - `p`: Tampilkan nama program/proses
>     

### B. Investigasi Target Berbahaya

Cari layanan berikut pada _output_ perintah `ss` di atas:

- **Port 21 (FTP) & Port 23 (Telnet):** Sangat rentan! Data dan _password_ dikirim dalam format _plain text_. Harus dimatikan kecuali soal secara spesifik memintanya.
    
- **Port 3306 (MySQL) & Port 5432 (PostgreSQL):** Cek IP yang _listening_. Jika `0.0.0.0`, artinya terbuka untuk publik. Seharusnya dikonfigurasi untuk _listen_ hanya di `127.0.0.1` (localhost).
    

### C. Eksekusi: Stop & Disable Layanan

Jika menemukan layanan yang tidak diperlukan (misal: `vsftpd`, `telnetd`, `rpcbind`), segera matikan.

**1. Hentikan layanan saat ini juga:**

Bash

```
sudo systemctl stop <nama_layanan>
```

**2. Cegah layanan hidup kembali saat server restart (PENTING!):**

Bash

```
sudo systemctl disable <nama_layanan>
```