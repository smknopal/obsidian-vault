# W7 — LAPS: Local Administrator Password Solution

#lks #cyber-security #windows #hardening #laps #active-directory #password

  

> **Tingkat:** 🟡 Sedang | **Waktu:** 1 malam (2–3 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → PAM / Active Directory

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

Bayangkan kamu punya 100 komputer di kantor, dan setiap komputer punya akun admin lokal bernama "Administrator". Masalahnya: **semua komputer pakai password admin lokal yang SAMA!**

  

Kalau attacker berhasil tau password 1 komputer → dia bisa login ke SEMUA 100 komputer (teknik ini disebut **Pass-the-Hash** atau **lateral movement**).

  

LAPS (Local Administrator Password Solution) solusinya:

- Setiap komputer punya password admin lokal yang **BERBEDA**

- Password **diganti otomatis secara berkala** (misal: tiap 30 hari)

- Password **disimpan di AD** dan hanya bisa dilihat admin yang berwenang

  

Hasilnya: kalau 1 komputer dikompromis, attacker tidak bisa pakai password yang sama untuk komputer lain!

  

---

  

## 🎯 Tujuan

  

- Memahami apa itu LAPS dan kenapa penting

- Install dan konfigurasi LAPS di AD

- Memahami cara membaca password LAPS

- Memverifikasi LAPS bekerja dengan benar

  

---

  

## BAGIAN 1 — Ada 2 Versi LAPS

  

| | LAPS Lama (Legacy) | Windows LAPS (Baru) |

|--|---------------------|---------------------------------------|

| Nama | Microsoft LAPS | Windows LAPS |

| Cara install | Download MSI terpisah | Sudah built-in di OS |

| Penyimpanan | Atribut AD biasa (plaintext di AD) | Atribut AD yang lebih aman (encrypted) |

| Enkripsi password | Tidak | Ya (opsional) |

| Tersedia di | Windows 7+ | Windows Server 2019+ / Win 11 22H2+ |

| Module | `AdmPwd.PS` | `LAPS` |

  

Di lomba, kemungkinan besar akan menggunakan **Legacy LAPS** karena masih umum. Tapi pelajari keduanya!

  

> 💡 **Perbedaan penting:** Di legacy LAPS, password tersimpan di atribut AD `ms-MCS-AdmPwd`

> dan bisa dibaca siapa saja yang punya akses read ke atribut tersebut.

> Windows LAPS mendukup enkripsi sehingga lebih aman.

  

---

  

## BAGIAN 2 — Install LAPS (Legacy)

  

### Langkah 1: Download LAPS

  

```

Download dari: https://www.microsoft.com/en-us/download/details.aspx?id=46899

File: LAPS.x64.msi (untuk 64-bit) atau LAPS.x86.msi (untuk 32-bit)

```

  

### Langkah 2: Install di Domain Controller (semua komponen)

  

```powershell

# Via command line (Silent install dengan semua komponen):

msiexec /i LAPS.x64.msi ADDLOCAL=ALL /quiet /l*v C:\LAPS-install.log

  

# Komponen yang diinstall (ADDLOCAL=ALL):

# - AdmPwd.PS        → PowerShell module

# - CSE              → Client-side extension (untuk komputer yang dikelola)

# - Management.UI    → GUI tool LAPS UI

# - Management.PS    → PowerShell management tools

# - Management.ADMX  → GPO template

  

# Verifikasi install berhasil:

Get-Module AdmPwd.PS -ListAvailable

```

  

### Langkah 3: Extend AD Schema (Tambah Atribut Baru ke AD)

  

> ⚠️ Langkah ini hanya perlu dilakukan SEKALI per forest!

  

```powershell

# Import modul LAPS

Import-Module AdmPwd.PS

  

# Update AD schema (tambah atribut ms-MCS-AdmPwd dan ms-MCS-AdmPwdExpirationTime)

Update-AdmPwdADSchema

  

# Output yang benar:

# Operation                                         Status

# -------------------------                         ------

# Checking domain controller                        Success

# Adding attribute ms-MCS-AdmPwd                   Success

# Adding attribute ms-MCS-AdmPwdExpirationTime     Success

# ...

  

# Verifikasi atribut sudah ada:

Get-ADObject -SearchBase (Get-ADRootDSE).schemaNamingContext `

  -Filter { name -like "*AdmPwd*" } | Select-Object Name

```

  

### Langkah 4: Beri Permission ke Computer Object untuk Update Password-nya Sendiri

  

```powershell

# Komputer perlu permission untuk MENULIS password ke atribut AD-nya sendiri

# Terapkan ke OU tempat komputer berada:

Set-AdmPwdComputerSelfPermission -OrgUnit "OU=Computers,DC=perusahaan,DC=local"

  

# Verifikasi:

Find-AdmPwdExtendedRights -Identity "OU=Computers,DC=perusahaan,DC=local"

```

  

### Langkah 5: Tentukan Siapa yang Boleh MEMBACA Password

  

```powershell

# Hanya grup IT_Admins yang boleh baca password:

Set-AdmPwdReadPasswordPermission `

  -OrgUnit "OU=Computers,DC=perusahaan,DC=local" `

  -AllowedPrincipals "IT_Admins"

  

# Hanya grup IT_Admins yang boleh reset password:

Set-AdmPwdResetPasswordPermission `

  -OrgUnit "OU=Computers,DC=perusahaan,DC=local" `

  -AllowedPrincipals "IT_Admins"

  

# Verifikasi siapa yang boleh baca:

Find-AdmPwdExtendedRights -Identity "OU=Computers,DC=perusahaan,DC=local"

```

  

> ⚠️ **PENTING:** Pastikan user biasa (non-admin) TIDAK punya hak baca atribut `ms-MCS-AdmPwd`!

> Jika bisa dibaca semua orang, LAPS tidak berguna.

  

### Langkah 6: Deploy Policy LAPS via GPO

  

```

1. Buka gpmc.msc

2. Buat GPO baru: "LAPS Policy"

3. Edit GPO → Computer Configuration

   → Administrative Templates → LAPS

   (ADMX template otomatis tersedia setelah install LAPS dengan ADDLOCAL=ALL)

4. Konfigurasi setting:

```

  

| Setting | Nilai |

|---------|-------|

| Enable local admin password management | **Enabled** |

| Password Settings → Complexity | Large letters + small letters + numbers + specials |

| Password Settings → Length | **20** (minimal!) |

| Password Settings → Age (Days) | **30** |

| Do not allow password expiration longer than policy | **Enabled** |

  

```

5. Link GPO ke OU Computers

6. gpupdate /force di komputer target

```

  

---

  

## BAGIAN 3 — Install LAPS di Client Computers

  

```powershell

# Di setiap komputer yang ingin dikelola LAPS:

# Install hanya CSE (Client Side Extension) — bukan Management Tools

msiexec /i LAPS.x64.msi ADDLOCAL=CSE /quiet

  

# Via GPO (cara yang benar untuk banyak komputer):

# Deploy MSI via GPO Software Installation:

# Computer Config → Policies → Software Settings → Software installation

# → Add LAPS.x64.msi dari share

  

# Verifikasi CSE terinstall di client:

Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\GPExtensions" |

  Where-Object { $_ -like "*AdmPwd*" }

```

  

---

  

## BAGIAN 4 — Membaca Password LAPS

  

### Via PowerShell:

  

```powershell

# Import modul:

Import-Module AdmPwd.PS

  

# Lihat password LAPS komputer tertentu:

Get-AdmPwdPassword -ComputerName "WORKSTATION01"

# Output:

# ComputerName       Password    ExpirationTimestamp

# ------------       --------    -------------------

# WORKSTATION01      xK9#mP2!    01/15/2026 02:00:00

  

# Atau via AD Attribute langsung:

Get-ADComputer -Identity "WORKSTATION01" `

  -Properties "ms-MCS-AdmPwd", "ms-MCS-AdmPwdExpirationTime" |

  Select-Object Name, "ms-MCS-AdmPwd", "ms-MCS-AdmPwdExpirationTime"

```

  

### Via GUI (LAPS UI):

  

```

1. Buka "LAPS UI" (terinstall di DC bersama Management Tools)

2. Masukkan nama komputer → klik Get

3. Password dan expiration time akan muncul

```

  

### Reset Password Lebih Awal:

  

```powershell

# Paksa reset password LAPS sekarang (berguna kalau password mungkin bocor):

Reset-AdmPwdPassword -ComputerName "WORKSTATION01"

Write-Host "Password reset dijadwalkan. Akan berubah saat komputer apply policy berikutnya."

  

# Paksa gpupdate di komputer target agar segera berubah:

# (Jalankan di komputer WORKSTATION01)

gpupdate /force

  

# Verifikasi password sudah berubah:

Get-AdmPwdPassword -ComputerName "WORKSTATION01"

```

  

---

  

## BAGIAN 5 — Windows LAPS (Versi Baru — Built-in)

  

Jika menggunakan Windows Server 2022 atau Windows 11 22H2+, bisa menggunakan Windows LAPS yang sudah built-in.

  

```powershell

# CEK: Verifikasi Windows LAPS tersedia di OS

$os = Get-WmiObject Win32_OperatingSystem

Write-Host "OS: $($os.Caption) Build $($os.BuildNumber)"

# Build 25145+ = Windows LAPS tersedia

  

# Update AD Schema untuk Windows LAPS:

Update-LapsADSchema

  

# Beri permission ke OU:

Set-LapsADComputerSelfPermission -Identity "OU=Computers,DC=perusahaan,DC=local"

  

# Konfigurasi siapa yang boleh baca:

Set-LapsADReadPasswordPermission `

  -Identity "OU=Computers,DC=perusahaan,DC=local" `

  -AllowedPrincipals "IT_Admins"

  

# Buat GPO untuk Windows LAPS:

# Computer Config → Administrative Templates → System → LAPS

# → Backup directory: Active Directory

# → Password age days: 30

# → Password length: 20

# → Password complexity: Large letters + small letters + numbers + special

  

# Baca password:

Get-LapsADPassword -Identity "WORKSTATION01" -AsPlainText

  

# Reset password:

Reset-LapsPassword -ComputerName "WORKSTATION01"

```

  

---

  

## BAGIAN 6 — Verifikasi LAPS Berjalan

  

```powershell

# Di DC: cek semua komputer dan status LAPS-nya

Write-Host "=== Status LAPS di semua komputer ==="

Get-ADComputer -Filter * -Properties "ms-MCS-AdmPwd", "ms-MCS-AdmPwdExpirationTime" |

  Select-Object Name, "ms-MCS-AdmPwd", "ms-MCS-AdmPwdExpirationTime" |

  Sort-Object Name | Format-Table -AutoSize

  

# Kalau kolom ms-MCS-AdmPwd terisi → LAPS berjalan di komputer itu!

# Kalau kosong → LAPS belum berjalan (cek CSE terinstall, GPO berlaku)

  

# Cek Event Log LAPS di komputer client (Event ID 10018 = password berhasil di-set):

Get-EventLog -LogName "Application" -Source "AdmPwd" -Newest 10 -ErrorAction SilentlyContinue

  

# Di komputer client: cek apakah GPO LAPS berlaku:

gpresult /r | Select-String "LAPS"

```

  

---

  

## ✅ Checklist LAPS

  

**Install & Setup (DC)**

- [ ] LAPS MSI terinstall di DC dengan Management Tools (`ADDLOCAL=ALL`)

- [ ] AD Schema sudah di-extend (`Update-AdmPwdADSchema`)

- [ ] Computer self-permission sudah di-set untuk OU Computers

- [ ] Read permission dibatasi hanya untuk group admin (`Set-AdmPwdReadPasswordPermission`)

- [ ] Reset permission dibatasi hanya untuk group admin

  

**GPO LAPS**

- [ ] GPO LAPS sudah dibuat dan di-link ke OU Computers

- [ ] GPO LAPS: Enable local admin password management = Enabled

- [ ] GPO LAPS: Password length minimal 20 karakter

- [ ] GPO LAPS: Kompleksitas tinggi (semua karakter)

- [ ] GPO LAPS: Password expire 30 hari

  

**Client**

- [ ] LAPS CSE terinstall di semua komputer yang dikelola

- [ ] `gpupdate /force` sudah dijalankan di client

  

**Verifikasi**

- [ ] `Get-AdmPwdPassword -ComputerName "..."` berhasil menampilkan password

- [ ] Kolom `ms-MCS-AdmPwd` terisi di AD (cek 2-3 komputer)

- [ ] Setiap komputer punya password yang **BERBEDA** dari komputer lain

  

---

  

## 🔗 Navigasi

  

← [[W6_Logging_Audit]]

→ [[W8_Credential_Guard]]