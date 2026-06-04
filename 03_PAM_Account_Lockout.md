# 03 — PAM: Account Lockout & Sudo Hardening
#lks #cyber-security #linux #hardening #pam #sudo #account-lockout

> **Peran:** Defender | **OS:** Debian/Ubuntu-based | **Konteks:** LKS Cyber Security 2026
> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Privileged Access Management (PAM)

---

## 🎯 Tujuan

- Mengunci akun otomatis setelah beberapa kali login gagal (anti brute-force)
- Mengontrol siapa yang boleh pakai `sudo` dan sejauh mana hak aksesnya
- Memastikan tidak ada user berbahaya atau misconfiguration di sistem

---

> 🧠 **Bedakan perbedaan ini:**
>
> | Alat | Yang Diblokir | Di Mana |
> |------|--------------|---------|
> | **Fail2ban** (File 01) | **IP address** attacker | Di firewall iptables |
> | **pam_faillock** (File ini) | **Akun user** di sistem | Di autentikasi PAM |
>
> Keduanya saling melengkapi. Fail2ban blokir dari sisi jaringan, pam_faillock blokir dari sisi sistem.

---

## Bagian A — Account Lockout dengan pam_faillock

### Konsep

```
Skenario tanpa pam_faillock:
Attacker coba password → salah → coba lagi → salah → coba lagi → ... → (unlimited!)

Skenario dengan pam_faillock:
Attacker coba password → salah (1/3)
                       → salah (2/3)
                       → salah (3/3)
                       → AKUN DIKUNCI 15 MENIT!
                       → Meskipun attacker tahu password yang benar → TETAP TIDAK BISA MASUK!
```

---

### Step 1: Konfigurasi faillock.conf

```bash
sudo nano /etc/security/faillock.conf
```

```ini
# ============================================================
# KONFIGURASI UTAMA
# ============================================================

deny = 3
# Kunci akun setelah 3 kali percobaan login gagal
# Rekomendasi: 3-5 (jangan terlalu kecil, user biasa juga bisa salah)

unlock_time = 900
# Berapa lama akun dikunci: 900 detik = 15 menit
# Setelah 15 menit → akun otomatis terbuka (tanpa intervensi admin)
# Untuk keamanan maksimal → set ke 0 (tidak pernah otomatis terbuka, harus manual)

fail_interval = 900
# Hitung percobaan gagal dalam rentang waktu ini: 900 detik = 15 menit
# Artinya: 3 kali gagal dalam 15 menit = akun dikunci
# Kalau gagal di menit 1, lalu gagal lagi di menit 20 → tidak dihitung berturut-turut

# ============================================================
# KEAMANAN TAMBAHAN
# ============================================================

audit
# Catat semua event ke syslog (untuk keperluan audit dan monitoring)

silent
# Sembunyikan pesan error spesifik
# Tanpa ini: "Account locked" → attacker tahu akun terkunci
# Dengan ini: tampil pesan generik → attacker tidak tahu apa yang terjadi

# even_root
# Aktifkan lockout bahkan untuk root (HATI-HATI! Bisa lock dirimu sendiri!)
# Di lomba: aktifkan ini hanya jika soal meminta
```

---

### Step 2: Aktifkan di PAM

Ini langkah yang paling penting dan paling sering salah. Posisi baris SANGAT berpengaruh!

```bash
sudo nano /etc/pam.d/common-auth
```

Isi file harus terlihat seperti ini (edit agar sesuai):

```
# BARIS 1: Cek SEBELUM autentikasi (preauth)
# Sebelum cek password, tanya dulu: "apakah akun ini sudah terkunci?"
# Jika terkunci → LANGSUNG tolak, tidak perlu cek password lagi
auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900

# BARIS 2: Cek password (baris ini biasanya sudah ada, jangan dihapus!)
auth [success=1 default=ignore] pam_unix.so nullok

# BARIS 3: Catat kegagalan SETELAH autentikasi (authfail)
# Kalau password salah → catat +1 kegagalan untuk user ini
auth [default=die] pam_faillock.so authfail audit deny=3 unlock_time=900

# BARIS 4 & 5: Baris default (biasanya sudah ada, jangan dihapus!)
auth requisite pam_deny.so
auth required pam_permit.so
```

**Visualisasi alur:**
```
User login → [preauth: cek apakah akun terkunci?]
                    ↓ YA (terkunci) → TOLAK langsung
                    ↓ TIDAK (belum terkunci)
             [pam_unix: cek password]
                    ↓ BENAR → izinkan masuk ✅
                    ↓ SALAH
             [authfail: catat kegagalan +1]
                    ↓ Kalau sudah 3x gagal → kunci akun
             [pam_deny: tolak] ❌
```

**Edit juga `/etc/pam.d/common-account`:**

```bash
sudo nano /etc/pam.d/common-account
```

Tambahkan baris ini (kalau belum ada):
```
account required pam_faillock.so
```

> Ini memastikan status lockout juga dicek di level account, bukan hanya di level auth.

---

### Step 3: Test Konfigurasi

```bash
# Test: coba login dengan password SALAH beberapa kali
ssh wrongpassword@localhost
# Ulangi 3 kali...

# Cek apakah akun terkunci
sudo faillock --user alice

# Output jika terkunci (ada catatan percobaan gagal):
# alice:
# When                Type  Source              Valid
# 2026-05-24 10:00:01 RHOST 192.168.1.5         V
# 2026-05-24 10:00:05 RHOST 192.168.1.5         V
# 2026-05-24 10:00:10 RHOST 192.168.1.5         V

# Output jika belum/tidak terkunci:
# alice:
# When                Type  Source              Valid
# (kosong)
```

---

### Step 4: Perintah Manajemen pam_faillock

```bash
# Lihat status lockout SEMUA user
sudo faillock

# Lihat status user tertentu
sudo faillock --user alice

# Reset / buka kunci akun yang terkunci (manual oleh admin)
sudo faillock --user alice --reset

# Cara alternatif via pam_faillock:
sudo pam_faillock --user alice --reset
```

---

## Bagian B — Sudo Hardening

`sudo` adalah pintu menuju hak root. Salah konfigurasi → privilege escalation!

> 🧠 **Prinsip Least Privilege:**
> "Berikan hak akses sekecil mungkin yang diperlukan untuk menjalankan tugas."
>
> Contoh BURUK: Kasih semua orang akses `sudo ALL` → semua bisa jadi root
> Contoh BAIK: Developer hanya bisa restart apache, tidak lebih dari itu

---

### WAJIB: Selalu Pakai `visudo`!

```bash
# SELALU gunakan ini untuk edit sudoers
sudo visudo

# JANGAN PERNAH pakai ini!
# sudo nano /etc/sudoers    ← BERBAHAYA!
```

> ⚠️ **Kenapa harus `visudo`?**
> `visudo` memvalidasi syntax sebelum menyimpan.
> Jika kamu salah syntax di `/etc/sudoers` tanpa `visudo` → kamu **tidak bisa menggunakan sudo sama sekali** → server bisa tidak bisa diakses!
> `visudo` akan menolak simpan jika ada error syntax.

---

### Format Aturan Sudoers

```
<user/group>    <host>=(<run_as>:<group>)    <perintah>
```

Kolom-kolomnya:
- **user/group** → siapa yang boleh (user atau `%grup`)
- **host** → dari host mana (biasanya `ALL`)
- **run_as** → menjalankan sebagai siapa (biasanya `ALL` atau `root`)
- **perintah** → perintah apa yang boleh (full path!)

---

### Contoh Aturan Sudoers

```bash
sudo visudo
```

```ini
# ============================================================
# CONTOH ATURAN
# ============================================================

# User "alice" bisa jalankan SEMUA perintah sebagai root
# (terlalu besar — hindari jika bisa)
alice ALL=(ALL:ALL) ALL

# User "bob" hanya bisa restart nginx, TANPA diminta password
bob ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx

# GRUP "webadmin" bisa restart service apapun (awali dengan %)
%webadmin ALL=(root) /usr/bin/systemctl restart *

# Batasi user "charlie" hanya untuk beberapa perintah tertentu
charlie ALL=(root) /usr/bin/apt update, /usr/bin/apt upgrade

# LARANG menjalankan bash/sh via sudo (cegah privilege escalation!)
# Ini penting! User bisa bypass via: sudo bash → dapat shell root
alice ALL=(ALL) ALL, !/bin/bash, !/bin/sh, !/bin/dash, !/usr/bin/python3
```

---

### Konfigurasi Defaults Penting

```bash
sudo visudo
```

Tambahkan di bagian atas (section Defaults):

```ini
# ============================================================
# DEFAULTS — berlaku untuk semua pengguna sudo
# ============================================================

# LOGGING: Catat semua perintah sudo ke file log
Defaults    logfile="/var/log/sudo.log"
Defaults    log_input, log_output
# log_input = log apa yang diketik user
# log_output = log output dari perintah

# SESSION TIMEOUT: Berapa menit sebelum sudo minta password lagi
Defaults    timestamp_timeout=5
# 5 menit idle → minta password lagi
# 0 = minta password setiap kali (paling aman)

# PATH AMAN: Cegah PATH hijacking
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
# Ini memastikan sudo hanya pakai program dari direktori yang terpercaya

# PERINGATAN: Tampilkan pesan peringatan saat pertama kali pakai sudo
Defaults    lecture=always

# EMAIL: Kirim email jika ada percobaan sudo yang tidak valid
Defaults    mail_badpass

# REQUIRETTY: Hanya izinkan sudo dari terminal nyata (bukan script)
Defaults    requiretty
```

Defaults    lecture=alwaysDefaults   
timestamp_timeout=5

---

### File Sudoers Terpisah (Praktik Terbaik)

Daripada edit `/etc/sudoers` langsung, buat file terpisah di `/etc/sudoers.d/`:

```bash
# Buat file khusus untuk grup webadmin
sudo visudo -f /etc/sudoers.d/webadmin
```

Isi:
```
%webadmin ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx
```

```bash
# Set permissions yang benar (WAJIB!)
sudo chmod 440 /etc/sudoers.d/webadmin

# Verifikasi
ls -la /etc/sudoers.d/
```

> **Keuntungan file terpisah:**
> - Lebih rapi dan mudah dikelola
> - Tidak mengotori file `/etc/sudoers` utama
> - Mudah dihapus jika tidak diperlukan lagi

---

### Verifikasi Sudo

```bash
# Cek konfigurasi sudoers (tidak ada error?)
sudo visudo -c
# Output: /etc/sudoers: parsed OK

# Lihat privilege sudo user tertentu
sudo -l -U alice
# Output menunjukkan apa yang boleh dilakukan alice via sudo

# Cek log sudo
sudo cat /var/log/sudo.log
```

---

## Bagian C — User Management (Keamanan Akun)

### Prinsip Keamanan User

```
1. Satu fungsi = satu akun (jangan sharing akun)
2. Nonaktifkan akun yang tidak digunakan
3. Akun sistem (daemon) TIDAK boleh bisa login
4. Tidak ada user dengan password kosong (sangat berbahaya!)
5. Hanya root yang boleh punya UID 0
```

### Audit User yang Ada

```bash
# Cek SEMUA user yang bisa login (tidak pakai shell nologin/false)
grep -v "nologin\|false\|sync\|halt\|shutdown" /etc/passwd

# Cek UID 0 — HANYA root yang boleh!
awk -F: '($3 == 0) {print $1}' /etc/passwd
# Jika ada user lain selain root → BAHAYA! Hapus atau perbaiki

# Cek user tanpa password (SANGAT BERBAHAYA!)
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
# Jika ada output → segera set password atau kunci akun

# Cek akun yang sudah expired
sudo awk -F: '{print $1, $8}' /etc/shadow | grep -v "^root"
```

### Kunci Akun yang Tidak Diperlukan

```bash
# Kunci akun service yang tidak perlu bisa login
for user in daemon bin sys games man lp mail news uucp www-data; do
    # Kunci akun
    sudo usermod -L $user
    # Set shell ke nologin
    sudo usermod -s /usr/sbin/nologin $user
    echo "Akun $user dikunci"
done

# Verifikasi
grep "nologin" /etc/passwd | head -10
```

### Perintah Manajemen User

```bash
# Kunci akun (tidak bisa login, akun tidak dihapus)
sudo usermod -L username          # L = Lock

# Buka kunci akun
sudo usermod -U username          # U = Unlock

# Set shell ke nologin (tidak bisa login)
sudo usermod -s /usr/sbin/nologin username

# Set akun expired (tidak bisa login setelah tanggal ini)
sudo usermod --expiredate 2026-12-31 username

# Cek informasi lengkap user
sudo chage -l username            # Status password & expiry
id username                       # UID, GID, grup yang diikuti
groups username                   # Semua grup user

# Hapus user dari grup tertentu
sudo gpasswd -d username sudo     # Hapus dari grup sudo
sudo gpasswd -d username admin    # Hapus dari grup admin
```

---

## ✅ Checklist Account Lockout & Sudo (Untuk Lomba!)

### pam_faillock
- [ ] `deny = 3` di `/etc/security/faillock.conf`
- [ ] `unlock_time = 900` diset
- [ ] `fail_interval = 900` diset
- [ ] `audit` diaktifkan
- [ ] `silent` diaktifkan
- [ ] Baris `preauth` ada di `/etc/pam.d/common-auth`
- [ ] Baris `authfail` ada di `/etc/pam.d/common-auth`
- [ ] `pam_faillock.so` ada di `/etc/pam.d/common-account`
- [ ] Sudah ditest: login salah 3x → `sudo faillock --user alice` menunjukkan akun terkunci

### Sudo
- [ ] Selalu gunakan `visudo` untuk edit sudoers
- [ ] Tidak ada user non-root dengan akses `ALL=(ALL:ALL) ALL` tanpa alasan
- [ ] `timestamp_timeout=5` diset di Defaults
- [ ] `secure_path` diset di Defaults
- [ ] `logfile="/var/log/sudo.log"` diset
- [ ] `log_input, log_output` diaktifkan
- [ ] `sudo visudo -c` tidak ada error

### User Management
- [ ] Tidak ada user dengan UID 0 selain root → `awk -F: '($3==0)' /etc/passwd`
- [ ] Tidak ada user dengan password kosong → `awk -F: '($2=="")' /etc/shadow`
- [ ] Service account tidak bisa login (shell = nologin)
- [ ] Akun yang tidak digunakan dikunci → `usermod -L`

---

## 🔗 Navigasi

← `02_PAM_Password_Complexity`
→ `04_Dangerous_Exposed_Services`
