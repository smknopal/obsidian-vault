# 02 — PAM: Password Complexity
#lks #cyber-security #linux #hardening #pam #password

> **Peran:** Defender | **OS:** Debian/Ubuntu-based | **Konteks:** LKS Cyber Security 2026
> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Privileged Access Management (PAM)

---

## 🎯 Tujuan

Memaksa semua user di sistem menggunakan password yang kuat, sehingga tidak mudah di-crack via dictionary attack atau brute-force.

---

## Apa itu PAM?

**PAM (Pluggable Authentication Modules)** adalah kerangka kerja di Linux yang mengatur bagaimana autentikasi bekerja. **SEMUA** proses login melewati PAM — SSH, `su`, `sudo`, login lokal, semuanya!

```
User ketik password
        ↓
      [PAM Stack]
        ↓
   Cek modul-modul secara berurutan:
   ┌─────────────────────────────────────┐
   │ pam_faillock.so  → akun dikunci?    │ (File 03)
   │ pam_pwquality.so → password kuat?  │ (File ini)
   │ pam_unix.so      → password benar? │
   │ pam_pwhistory.so → pernah dipakai? │ (File ini)
   └─────────────────────────────────────┘
        ↓
   Izinkan / Tolak
```

### File Konfigurasi PAM

| File | Fungsi |
|------|--------|
| `/etc/pam.d/common-auth` | Konfigurasi autentikasi umum (cek password) |
| `/etc/pam.d/common-password` | Konfigurasi saat GANTI password |
| `/etc/pam.d/common-account` | Konfigurasi manajemen akun |
| `/etc/pam.d/sshd` | Konfigurasi khusus untuk SSH |
| `/etc/security/pwquality.conf` | Aturan kualitas password |
| `/etc/security/faillock.conf` | Aturan lockout akun |

---

## 1. Install Modul pwquality

```bash
# Update dulu, kemudian install
sudo apt update
sudo apt install libpam-pwquality -y

# Verifikasi berhasil terinstall
dpkg -l | grep libpam-pwquality
```

Output verifikasi yang benar:
```
ii  libpam-pwquality:amd64   1.4.4-1   amd64   PAM module to check password strength
```
> `ii` di depan = terinstall dengan benar ✅

---

## 2. Konfigurasi Aturan Password

```bash
sudo nano /etc/security/pwquality.conf
```

Isi dengan konfigurasi berikut (hapus semua tanda `#` di depan aturan yang kamu aktifkan):

```ini
# ============================================================
# ATURAN PANJANG PASSWORD
# ============================================================
minlen = 12
# Password minimal 12 karakter.
# Contoh DITOLAK: "Pass1!" (hanya 6 karakter)
# Contoh DITERIMA: "MyPassw0rd!@" (12 karakter) ✅

# ============================================================
# JENIS KARAKTER WAJIB
# PENTING: Nilai NEGATIF = WAJIB ada (required)
#          Nilai POSITIF = bonus kredit (bukan wajib)
# ============================================================
dcredit = -1
# Wajib ada minimal 1 angka (digit: 0-9)
# Contoh DITOLAK: "MyPassword!!" (tidak ada angka)
# Contoh DITERIMA: "MyPassw0rd!!" (ada "0") ✅

ucredit = -1
# Wajib ada minimal 1 huruf BESAR (A-Z)
# Contoh DITOLAK: "mypassw0rd!!" (tidak ada huruf besar)
# Contoh DITERIMA: "MyPassw0rd!!" (ada "M", "P") ✅

lcredit = -1
# Wajib ada minimal 1 huruf kecil (a-z)
# Contoh DITOLAK: "MYPASSW0RD!!" (tidak ada huruf kecil)
# Contoh DITERIMA: "MyPassw0rd!!" (ada "y", "assw", "rd") ✅

ocredit = -1
# Wajib ada minimal 1 simbol (!@#$%^&*_-=+., dll)
# Contoh DITOLAK: "MyPassw0rd12" (tidak ada simbol)
# Contoh DITERIMA: "MyPassw0rd!!" (ada "!", "!") ✅

# ============================================================
# ATURAN TAMBAHAN
# ============================================================
minclass = 4
# Password harus mengandung semua 4 jenis karakter sekaligus:
# angka, huruf besar, huruf kecil, simbol

maxrepeat = 3
# Maksimal 3 karakter SAMA berturut-turut
# Contoh DITOLAK: "MyPaaaard!1" (ada "aaaa" = 4 karakter sama berturut-turut)
# Contoh DITERIMA: "MyPaard!1X" (ada "aa" = hanya 2 berturut-turut) ✅

maxsequence = 3
# Maksimal 3 karakter BERURUTAN
# Contoh DITOLAK: "MyP@1234ssw" (ada "1234" = 4 karakter berurutan)
# Contoh DITERIMA: "MyP@ssw123!" (ada "123" = tepat 3, masih boleh) ✅

difok = 5
# Password baru harus beda minimal 5 karakter dari password lama
# Mencegah user yang iseng ganti password sedikit-sedikit

# ============================================================
# CEGAH PASSWORD UMUM
# ============================================================
badwords = password admin root linux server
# Kata-kata yang tidak boleh ada dalam password

dictcheck = 1
# Cek apakah password ada di kamus kata umum
# Password seperti "Qwerty123!" akan ditolak

usercheck = 1
# Cek apakah password mengandung username
# User "alice" tidak bisa pakai password "Alice123!"
```

---

## 3. Aktifkan Modul di PAM

```bash
# Cek apakah sudah ada
grep "pwquality" /etc/pam.d/common-password
```

Jika ada output → lanjut ke langkah 4.
Jika TIDAK ada output → tambahkan manual:

```bash
sudo nano /etc/pam.d/common-password
```

Cari baris pertama yang dimulai dengan `password` dan tambahkan baris ini **DI ATAS** baris itu:
```
password requisite pam_pwquality.so retry=3
```

> **Penjelasan:**
> - `requisite` → Jika gagal → LANGSUNG ditolak, tidak cek modul lain
> - `retry=3` → User dapat 3 kesempatan menginput password baru

---

## 4. pam_pwhistory — Cegah Pakai Password Lama

Modul ini mencegah user memakai kembali password yang sudah pernah dipakai.

> 🧠 **Kenapa penting?**
> Tanpa ini: User ganti password baru → lalu langsung ganti balik ke password lama = percuma!

```bash
sudo nano /etc/pam.d/common-password
```

Tambahkan baris ini:
```
password required pam_pwhistory.so remember=5 use_authok
```

> **Penjelasan:**
> - `remember=5` → Simpan 5 password terakhir. User tidak boleh pakai 5 password sebelumnya.
> - `use_authok` → Pakai token password dari modul sebelumnya (jangan minta password lagi)

### Contoh Urutan `/etc/pam.d/common-password` yang Benar

```
# 1. Cek kualitas password
password requisite pam_pwquality.so retry=3

# 2. Cek histori (jangan pakai password lama)
password required pam_pwhistory.so remember=5 use_authok

# 3. Simpan password baru ke sistem (baris ini biasanya sudah ada)
password [success=1 default=ignore] pam_unix.so obscure use_authtok try_first_pass yescrypt

# 4 & 5. Baris default (biasanya sudah ada, jangan dihapus)
password requisite pam_deny.so
password required pam_permit.so
```

---

## 5. Password Expiry — Atur Masa Berlaku Password

Password yang tidak pernah diganti = risiko keamanan besar!

### A. Untuk User BARU (edit /etc/login.defs)

```bash
sudo nano /etc/login.defs
```

Cari dan ubah baris-baris ini:
```ini
PASS_MAX_DAYS   90
# Password harus diganti setelah maksimal 90 hari
# Jika tidak diganti → akun terkunci (user dipaksa ganti)

PASS_MIN_DAYS   1
# Minimal 1 hari setelah ganti password baru sebelum boleh ganti lagi
# Mencegah user yang iseng cepat-cepat ganti balik ke password lama

PASS_WARN_AGE   7
# Kirim peringatan 7 hari sebelum password expired
```

> ⚠️ Pengaturan ini HANYA berlaku untuk user yang dibuat SETELAH perubahan ini.
> Untuk user yang sudah ada, gunakan `chage` di bawah.

### B. Untuk User yang SUDAH ADA (gunakan chage)

```bash
# Set expired 90 hari
sudo chage -M 90 alice

# Set minimal 1 hari sebelum bisa ganti lagi
sudo chage -m 1 alice

# Set peringatan 7 hari sebelum expired
sudo chage -W 7 alice

# Paksa user ganti password saat login berikutnya
sudo chage -d 0 alice
# Penjelasan: -d 0 = set tanggal ganti terakhir ke "epoch" (sangat lama yang lalu)
# Efek: sistem langsung anggap password sudah expired

# Cek status lengkap
sudo chage -l alice
```

Output `chage -l alice`:
```
Last password change                    : May 24, 2026
Password expires                        : Aug 22, 2026   ← 90 hari dari sekarang
Password inactive                       : never
Account expires                         : never
Minimum number of days between changes  : 1
Maximum number of days between changes  : 90
Number of days of warning before expiry : 7
```

### C. Terapkan Massal untuk Semua User

```bash
# Terapkan expiry untuk semua user biasa sekaligus
for user in $(awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd); do
    sudo chage -M 90 -m 1 -W 7 "$user"
    echo "Set expiry untuk: $user"
done
```

---

## 6. Uji Konfigurasi Password (WAJIB Dilakukan!)

Setelah konfigurasi, WAJIB ditest untuk memastikan benar-benar bekerja:

```bash
# Coba ganti password sebagai user biasa
passwd
# atau:
sudo passwd alice
```

**Test dengan password yang HARUS DITOLAK:**

| Password | Alasan Ditolak |
|----------|---------------|
| `abc` | Terlalu pendek |
| `password123` | Tidak ada huruf besar, tidak ada simbol |
| `PASSWORD123!` | Tidak ada huruf kecil |
| `MyPass!` | Terlalu pendek (kurang dari 12 karakter) |
| `aaaaaA1!bbbb` | maxrepeat dilanggar (aaaaa = 5 karakter sama berturut) |

**Test dengan password yang HARUS DITERIMA:**

| Password | Keterangan |
|----------|-----------|
| `MyS3cur3P@ss!` | ✅ 13 karakter, ada A-Z, a-z, 0-9, simbol |
| `Jamb1_B1s@Win!` | ✅ 15 karakter, lengkap semua jenis |

Jika password lemah ditolak dengan pesan seperti:
```
BAD PASSWORD: The password is shorter than 12 characters
BAD PASSWORD: The password fails the dictionary check
BAD PASSWORD: The password does not contain enough character classes
```
→ Konfigurasi sudah benar! ✅

---

## 7. Verifikasi Konfigurasi

```bash
# Lihat konfigurasi pwquality yang aktif (tanpa komentar)
grep -v "^#" /etc/security/pwquality.conf | grep -v "^$"

# Cek modul pwquality terdaftar di PAM
grep "pwquality" /etc/pam.d/common-password

# Cek modul pwhistory terdaftar di PAM
grep "pwhistory" /etc/pam.d/common-password

# Lihat seluruh isi common-password
cat /etc/pam.d/common-password

# Cek setting login.defs
grep "PASS_" /etc/login.defs
```

---

## ✅ Checklist PAM Password Complexity (Untuk Lomba!)

### Instalasi
- [ ] `libpam-pwquality` terinstall → `dpkg -l | grep libpam-pwquality`

### pwquality.conf
- [ ] `minlen = 12`
- [ ] `dcredit = -1` (wajib 1 angka)
- [ ] `ucredit = -1` (wajib 1 huruf besar)
- [ ] `lcredit = -1` (wajib 1 huruf kecil)
- [ ] `ocredit = -1` (wajib 1 simbol)
- [ ] `minclass = 4` (wajib semua 4 jenis)
- [ ] `maxrepeat = 3`
- [ ] `maxsequence = 3`
- [ ] `difok = 5`
- [ ] `dictcheck = 1`
- [ ] `usercheck = 1`

### PAM Stack
- [ ] `pam_pwquality.so retry=3` ada di `/etc/pam.d/common-password`
- [ ] `pam_pwhistory.so remember=5` ada di `/etc/pam.d/common-password`
- [ ] Urutan modul sudah benar (pwquality → pwhistory → pam_unix)

### login.defs
- [ ] `PASS_MAX_DAYS 90`
- [ ] `PASS_MIN_DAYS 1`
- [ ] `PASS_WARN_AGE 7`

### Existing Users
- [ ] `chage -M 90 username` sudah dilakukan untuk semua user
- [ ] `chage -d 0 username` untuk user yang perlu segera ganti password

### Pengujian
- [ ] Password pendek ditolak
- [ ] Password tanpa simbol ditolak
- [ ] Password tanpa huruf besar ditolak
- [ ] Password kuat berhasil diterima

---

## 🔗 Navigasi

← `01_Network_Service_Security`
→ `03_PAM_Account_Lockout`
