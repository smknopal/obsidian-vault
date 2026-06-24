# W1 — PAM: Local Security Policy (Windows)

#lks #cyber-security #windows #hardening #pam #password #account-lockout

  

> **Tingkat:** 🟢 Mudah | **Waktu:** 1 malam (2–3 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → PAM

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

Bayangkan kamu punya kantor. PAM (Privileged Access Management) itu seperti:

- **Password Policy** = aturan kunci pintu kantor (harus pakai kunci 12 digit, ganti tiap 90 hari)

- **Account Lockout** = sistem alarm (salah masukkan kunci 5 kali → pintu terkunci 15 menit)

- **User Accounts** = kartu akses karyawan (siapa boleh masuk ke ruang mana)

  

Tugasmu: pastikan semua aturan ini aktif dan benar!

  

---

  

## 🎯 Tujuan

  

- Memaksa user pakai password kuat

- Mengunci akun otomatis kalau ada brute-force

- Menonaktifkan akun bawaan Windows yang berbahaya (Guest, Administrator)

- Mengatur hak akses user

  

---

  

## BAGIAN 1 — Cara Buka Local Security Policy

  

> ⚠️ Ini hanya untuk komputer yang **BUKAN** domain member (standalone). Kalau sudah join domain, setting ini dikontrol oleh Domain GPO — perubahan di sini akan di-override oleh GPO.

  

**Ada 3 cara membuka:**

  

```

Cara 1 (Tercepat): Win + R → ketik "secpol.msc" → Enter

Cara 2: Win + R → ketik "gpedit.msc" → Computer Configuration

        → Windows Settings → Security Settings

Cara 3: Control Panel → Administrative Tools → Local Security Policy

```

  

> ⚠️ `secpol.msc` hanya tersedia di Windows Pro/Enterprise/Server.

> Tidak ada di Windows Home! Di lomba biasanya pakai Server atau Pro, jadi aman.

  

---

  

## BAGIAN 2 — Password Policy

  

**Lokasi di `secpol.msc`:**

```

Account Policies → Password Policy

```

  

### Nilai yang Harus Dikonfigurasi:

  

| Setting | Nilai Wajib | Kenapa? |

|---------|-------------|---------|

| Enforce password history | **5** | Cegah user pakai ulang 5 password lama |

| Maximum password age | **90 days** | Password wajib ganti tiap 90 hari |

| Minimum password age | **1 day** | Cegah user langsung ganti balik ke yang lama |

| Minimum password length | **12** | Minimal 12 karakter |

| Password must meet complexity | **Enabled** | WAJIB ON! |

| Store passwords using reversible encryption | **Disabled** | JANGAN diaktifkan! |

  

> 💡 **Catatan:** Di beberapa soal lomba, minimum password length bisa diminta 14 karakter. Selalu baca soal dengan teliti!

  

### Apa itu "Complexity Requirements"?

Kalau Enabled, Windows otomatis mewajibkan password harus mengandung minimal 3 dari 4 kategori ini:

- Huruf BESAR (A-Z)

- Huruf kecil (a-z)

- Angka (0-9)

- Simbol (!@#$%^&*)

  

Dan password **tidak boleh mengandung nama akun user** (username).

  

Contoh password yang memenuhi: `P@ssw0rd2026!`

  

### Cara Ubah via GUI (Klik-klik):

```

1. Buka secpol.msc

2. Klik Account Policies → Password Policy

3. Double-click setting yang ingin diubah

4. Ubah nilainya → klik OK

```

  

### Cara Ubah via PowerShell (LEBIH CEPAT saat lomba!):

```powershell

# Set password minimum length ke 12

net accounts /minpwlen:12

  

# Set maximum password age ke 90 hari

net accounts /maxpwage:90

  

# Set minimum password age ke 1 hari

net accounts /minpwage:1

  

# Set password history ke 5

net accounts /uniquepw:5

  

# Lihat semua setting saat ini (untuk verifikasi)

net accounts

```

  

> 💡 **PENTING:** Complexity requirements dan "Store passwords using reversible encryption"

> **HANYA bisa diubah via GUI (secpol.msc)**, tidak bisa via `net accounts`.

> Untuk complexity via PowerShell di domain, gunakan `Set-ADDefaultDomainPasswordPolicy`.

  

---

  

## BAGIAN 3 — Account Lockout Policy

  

Account lockout = sistem penguncian otomatis kalau ada yang coba login salah berkali-kali (anti brute-force).

  

**Lokasi di `secpol.msc`:**

```

Account Policies → Account Lockout Policy

```

  

### Nilai yang Harus Dikonfigurasi:

  

| Setting | Nilai Wajib | Kenapa? |

|---------|-------------|---------|

| Account lockout threshold | **5** | Kunci setelah 5 kali gagal login |

| Account lockout duration | **15 minutes** | Kunci selama 15 menit |

| Reset account lockout counter after | **15 minutes** | Reset hitungan setelah 15 menit |

  

> ⚠️ **Urutan penting!** Setting threshold dulu, baru duration dan observation window muncul.

  

### Cara Mengatur via GUI:

```

1. Buka secpol.msc

2. Account Policies → Account Lockout Policy

3. Double-click "Account lockout threshold"

4. Set ke 5 → klik OK

5. Windows akan muncul dialog → minta konfirmasi suggest untuk 2 setting lainnya

6. Klik OK untuk accept suggest (biasanya sudah 15 menit)

7. Verifikasi ketiga nilai sudah benar

```

  

### Cara via PowerShell:

```powershell

# Set lockout threshold ke 5 kali percobaan

net accounts /lockoutthreshold:5

  

# Set lockout duration ke 15 menit

net accounts /lockoutduration:15

  

# Set observation window ke 15 menit

net accounts /lockoutwindow:15

  

# Verifikasi semua setting:

net accounts

```

  

### Cara Reset Akun yang Terkunci:

```powershell

# Lihat semua user lokal dan statusnya (termasuk yang terkunci)

Get-LocalUser | Select-Object Name, Enabled, IsAccountLocked

  

# Via PowerShell (lokal) — unlock user tertentu:

$user = [ADSI]"WinNT://./namauser,user"

$user.IsAccountLocked = $false

$user.SetInfo()

Write-Host "Akun namauser sudah di-unlock"

  

# Via PowerShell jika di AD (domain):

Unlock-ADAccount -Identity "namauser"

  

# Verifikasi sudah di-unlock:

Get-LocalUser -Name "namauser" | Select-Object Name, Enabled, IsAccountLocked

```

  

---

  

## BAGIAN 4 — Manage User Accounts (lusrmgr.msc)

  

**Buka:** `Win + R → lusrmgr.msc`

  

Atau lebih cepat via PowerShell:

```powershell

# Lihat semua user lokal

Get-LocalUser | Select-Object Name, Enabled, PasswordNeverExpires, LastLogon

```

  

### 4A. Nonaktifkan Built-in Administrator

  

Built-in Administrator adalah akun bawaan Windows yang punya akses penuh. Bahayanya:

- Tidak bisa dikunci oleh Account Lockout Policy (kebal lockout!)

- Selalu ada di setiap instalasi Windows baru

- Attacker tahu nama "Administrator" dan langsung menyerang akun ini

  

**Yang harus dilakukan:** Nonaktifkan DAN ganti namanya!

  

```powershell

# OPSI 1: Nonaktifkan built-in Administrator

Disable-LocalUser -Name "Administrator"

# Atau:

net user administrator /active:no

  

# OPSI 2 (LEBIH AMAN): Ganti nama dulu, lalu nonaktifkan

# Rename via GUI:

# lusrmgr.msc → Users → klik kanan Administrator → Rename → ketik nama baru

# Contoh nama baru: "adm_sysops" atau "sysadmin2026"

  

# Verifikasi (pastikan "Account active" = No):

net user administrator

# atau:

Get-LocalUser -Name "Administrator" | Select-Object Name, Enabled

```

  

> ⚠️ **Penting:** Jangan nonaktifkan Administrator jika belum ada akun admin lain yang aktif,

> atau kamu bisa terkunci dari sistem!

  

### 4B. Nonaktifkan Guest Account

  

Guest account adalah akun tanpa password yang sangat berbahaya.

  

```powershell

# Nonaktifkan Guest

Disable-LocalUser -Name "Guest"

# Atau:

net user guest /active:no

  

# Verifikasi:

Get-LocalUser -Name "Guest" | Select-Object Name, Enabled

# Enabled harus: False

```

  

### 4C. Buat User Baru dengan Hak Minimal

  

Kalau diminta buat user baru, JANGAN langsung tambahkan ke Administrators!

  

```powershell

# Buat user baru dengan password aman

$password = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

New-LocalUser -Name "namauser" -Password $password -FullName "Nama Lengkap" `

  -Description "User biasa" -PasswordNeverExpires $false

  

# Tambahkan ke grup Users (bukan Administrators!)

Add-LocalGroupMember -Group "Users" -Member "namauser"

  

# Kalau perlu jadi admin, baru tambahkan:

Add-LocalGroupMember -Group "Administrators" -Member "namauser"

  

# Lihat semua user

Get-LocalUser

  

# Lihat member grup Administrators (pastikan tidak ada user mencurigakan!)

Get-LocalGroupMember -Group "Administrators"

```

  

### 4D. Cek dan Perbaiki Password Expiry

  

```powershell

# Lihat semua user lokal beserta statusnya

Get-LocalUser | Select-Object Name, Enabled, PasswordExpires, PasswordNeverExpires

  

# Set agar password user harus expired (tidak boleh never expires)

# Via GUI: lusrmgr.msc → klik kanan user → Properties

# → uncheck "Password never expires"

  

# Via PowerShell (satu per satu):

Set-LocalUser -Name "namauser" -PasswordNeverExpires $false

  

# Via PowerShell (semua user sekaligus, kecuali service account):

Get-LocalUser | Where-Object { $_.PasswordNeverExpires -eq $true } |

  ForEach-Object {

    Set-LocalUser -Name $_.Name -PasswordNeverExpires $false

    Write-Host "Fixed: $($_.Name)"

  }

  

# Verifikasi:

Get-LocalUser | Select-Object Name, PasswordNeverExpires

# Semua harus: False (kecuali service account yang terdokumentasi)

```

  

---

  

## BAGIAN 5 — Security Options Penting

  

**Lokasi di `secpol.msc`:**

```

Local Policies → Security Options

```

  

Ini sering diuji di lomba! Hafal setting-setting ini:

  

### Setting yang Harus Diubah:

  

```

✅ Accounts: Rename administrator account

   → Ganti nama dari "Administrator" ke nama lain

   → Contoh: "sysadmin2026"

  

✅ Accounts: Rename guest account

   → Ganti dari "Guest" ke nama lain

  

✅ Interactive logon: Do not display last user name

   → Value: Enabled

   → Supaya nama user terakhir login tidak muncul di layar login

  

✅ Interactive logon: Message text for users attempting to log on

   → Isi: "Authorized users only. All activity is monitored."

   → Banner peringatan sebelum login

  

✅ Interactive logon: Message title for users attempting to log on

   → Isi: "WARNING"

   → Judul banner (wajib diisi jika message text sudah diisi!)

  

✅ Interactive logon: Machine inactivity limit

   → Value: 900 (15 menit dalam detik)

   → Auto-lock setelah 15 menit idle

  

✅ Network access: Do not allow anonymous enumeration of SAM accounts

   → Value: Enabled

   → Cegah attacker enumerasi user via network

  

✅ Network access: Do not allow anonymous enumeration of SAM accounts and shares

   → Value: Enabled

  

✅ Network security: LAN Manager authentication level

   → Value: "Send NTLMv2 response only. Refuse LM & NTLM"

   → Matikan autentikasi lama yang lemah (LM dan NTLMv1)

  

✅ Accounts: Guest account status

   → Disabled

  

✅ Shutdown: Allow system to be shut down without having to log on

   → Value: Disabled

   → Cegah orang shutdown server tanpa login

```

  

---

  

## BAGIAN 6 — User Rights Assignment

  

**Lokasi di `secpol.msc`:**

```

Local Policies → User Rights Assignment

```

  

```

✅ "Deny access to this computer from the network"

   → Tambahkan: Guest, Anonymous Logon

  

✅ "Deny log on locally"

   → Tambahkan: Guest

  

✅ "Access this computer from the network"

   → Hapus "Everyone" kalau ada

   → Sisakan: Administrators, Authenticated Users

  

✅ "Take ownership of files or other objects"

   → Pastikan hanya Administrators

  

✅ "Debug programs"

   → Hapus dari semua user, atau hanya Administrators

   → Ini sering dipakai attacker (Mimikatz butuh hak ini!)

```

  

---

  

## BAGIAN 7 — Verifikasi Semua Setting

  

Setelah selesai setting, SELALU verifikasi!

  

```powershell

# Cek semua password dan lockout policy

net accounts

  

# Cek user aktif dan status password

Get-LocalUser | Select-Object Name, Enabled, PasswordNeverExpires, IsAccountLocked

  

# Cek member group Administrators

Get-LocalGroupMember -Group "Administrators"

  

# Cek member group Guests

Get-LocalGroupMember -Group "Guests"

  

# Export setting security policy ke file teks (untuk bukti):

secedit /export /cfg C:\security-export.txt

type C:\security-export.txt

  

# Cek status firewall (bonus check)

netsh advfirewall show allprofiles

```

  

---

  

## ✅ Checklist PAM Local Security Policy

  

Sebelum selesai, centang semua ini:

  

**Password Policy**

- [ ] Minimum password length = 12

- [ ] Password complexity requirements = Enabled

- [ ] Maximum password age = 90 days

- [ ] Minimum password age = 1 day

- [ ] Enforce password history = 5

- [ ] Store passwords using reversible encryption = Disabled

  

**Account Lockout Policy**

- [ ] Account lockout threshold = 5

- [ ] Account lockout duration = 15 minutes

- [ ] Reset lockout counter after = 15 minutes

  

**User Accounts**

- [ ] Built-in Administrator dinonaktifkan DAN diganti nama

- [ ] Guest account dinonaktifkan

- [ ] Tidak ada user yang tidak dikenal di grup Administrators

- [ ] Tidak ada user dengan PasswordNeverExpires = True (kecuali service account)

  

**Security Options**

- [ ] "Do not display last user name" = Enabled

- [ ] Login banner (message text & title) sudah diisi

- [ ] LAN Manager authentication level = NTLMv2 only

- [ ] Machine inactivity limit = 900 detik

- [ ] Anonymous enumeration SAM accounts = Disabled

  

**Verifikasi**

- [ ] `net accounts` dijalankan untuk verifikasi

- [ ] `Get-LocalUser` dijalankan untuk konfirmasi status user

- [ ] `Get-LocalGroupMember -Group "Administrators"` sudah diperiksa

  

---

  

## 🔗 Navigasi

  

← [[W0_INDEX_Windows_Hardening]]

→ [[W2_Active_Directory]]