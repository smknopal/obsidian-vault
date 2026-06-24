# W2 — Active Directory Basic Security

#lks #cyber-security #windows #hardening #active-directory #ad #domain

  

> **Tingkat:** 🔴 Susah | **Waktu:** 3 malam (6–8 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → Basic Security Configurations on Active Directory

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

Active Directory (AD) seperti **buku direktori perusahaan yang super canggih**:

- **Domain** = nama perusahaan (contoh: `perusahaan.local`)

- **OU (Organizational Unit)** = departemen (IT, HR, Finance)

- **User** = karyawan

- **Group** = jabatan/tim (semua anggota tim IT masuk grup yang sama)

- **GPO** = peraturan kantor yang berlaku untuk satu departemen

- **DC (Domain Controller)** = server pusat yang mengatur semua ini

  

Kalau perusahaan = AD, maka tugasmu adalah **memastikan tidak ada karyawan yang punya akses lebih dari yang mereka butuhkan**.

  

---

  

## 🎯 Tujuan

  

- Memahami struktur dasar AD (Domain, OU, User, Group)

- Mengamankan akun-akun privileged di AD

- Menerapkan password policy yang berbeda untuk group berbeda (PSO)

- Membersihkan konfigurasi AD yang berbahaya

  

---

  

## BAGIAN 1 — Konsep Dasar AD (WAJIB PAHAM!)

  

```

Domain: perusahaan.local

├── OU: IT

│   └── User: admin_it

├── OU: Staff

│   ├── User: alice

│   └── User: bob

├── OU: Computers

│   └── PC: WORKSTATION01

└── OU: Servers

    └── PC: SERVER01

```

  

### Komponen Penting:

  

| Komponen | Penjelasan |

|----------|------------|

| Domain | Wadah utama, misal `perusahaan.local` |

| DC (Domain Controller) | Server yang mengelola AD |

| OU (Organizational Unit) | Folder untuk mengelompokkan objek |

| User | Akun pengguna |

| Group | Kumpulan user dengan hak akses sama |

| GPO | Group Policy yang di-link ke OU atau Domain |

  

### Grup Bawaan yang WAJIB Diketahui:

  

| Grup | Fungsi | Harus Diperhatikan? |

|------|--------|---------------------|

| Domain Admins | Admin domain, akses ke semua | 🔴 YES — jaga jumlahnya! |

| Enterprise Admins | Admin seluruh forest | 🔴 YES — harusnya kosong! |

| Schema Admins | Modifikasi schema AD | 🔴 YES — harusnya kosong! |

| Domain Users | Semua user domain otomatis masuk | 🟡 Normal |

| Protected Users | Perlindungan ekstra (no NTLM, dll.) | ✅ Masukkan admin di sini |

| Administrators | Admin lokal di DC | 🔴 YES — pantau |

  

---

  

## BAGIAN 2 — Install Active Directory Domain Services (AD DS)

  

> 💡 Bagian ini untuk kasus kamu diminta setup AD dari awal di lomba.

  

### Cara 1: Via Server Manager (GUI) — Untuk Pemula

  

```

1. Buka Server Manager (otomatis terbuka di Windows Server)

2. Klik "Add roles and features"

3. Pilih "Role-based or feature-based installation" → Next

4. Pilih server lokal → Next

5. Centang "Active Directory Domain Services"

6. Klik "Add Features" jika ada popup → Next → Next → Install

7. Tunggu selesai (5-10 menit)

8. Klik "Promote this server to a domain controller" (muncul di notifikasi kuning)

9. Pilih "Add a new forest"

10. Isi Root domain name: contoh "perusahaan.local"

11. Set Directory Services Restore Mode (DSRM) password

    → Ini password darurat kalau AD rusak, simpan baik-baik!

    → Gunakan password kuat: minimal 12 karakter, kompleks

12. Next → Next → Install → Server akan restart otomatis

```

  

### Cara 2: Via PowerShell (LEBIH CEPAT — Disarankan saat lomba!)

  

```powershell

# LANGKAH 1: Install role AD DS

Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

  

# LANGKAH 2: Promote server jadi Domain Controller

Import-Module ADDSDeployment

  

$DSRMPassword = ConvertTo-SecureString "P@ssw0rd2026!" -AsPlainText -Force

  

Install-ADDSForest `

  -CreateDnsDelegation:$false `

  -DatabasePath "C:\Windows\NTDS" `

  -DomainMode "WinThreshold" `

  -DomainName "perusahaan.local" `

  -DomainNetbiosName "PERUSAHAAN" `

  -ForestMode "WinThreshold" `

  -InstallDns:$true `

  -LogPath "C:\Windows\NTDS" `

  -NoRebootOnCompletion:$false `

  -SysvolPath "C:\Windows\SYSVOL" `

  -SafeModeAdministratorPassword $DSRMPassword `

  -Force:$true

  

# Server akan restart otomatis setelah selesai

```

  

> 💡 **WinThreshold** = Windows Server 2016 functional level. Gunakan ini untuk kompatibilitas terbaik.

  

---

  

## BAGIAN 3 — Buat Struktur OU, User, Group

  

**Buka Active Directory Users and Computers:**

```

Win + R → dsa.msc

```

  

### Buat OU (Organizational Unit):

  

```powershell

# Via PowerShell (lebih cepat!):

New-ADOrganizationalUnit -Name "IT" -Path "DC=perusahaan,DC=local" -ProtectedFromAccidentalDeletion $true

New-ADOrganizationalUnit -Name "Staff" -Path "DC=perusahaan,DC=local" -ProtectedFromAccidentalDeletion $true

New-ADOrganizationalUnit -Name "Computers" -Path "DC=perusahaan,DC=local" -ProtectedFromAccidentalDeletion $true

New-ADOrganizationalUnit -Name "Servers" -Path "DC=perusahaan,DC=local" -ProtectedFromAccidentalDeletion $true

  

# Verifikasi OU sudah dibuat:

Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

```

  

> 💡 `-ProtectedFromAccidentalDeletion $true` mencegah OU terhapus tidak sengaja.

  

### Buat User:

  

```powershell

# Buat 1 user:

New-ADUser `

  -Name "Alice Smith" `

  -GivenName "Alice" `

  -Surname "Smith" `

  -SamAccountName "alice.smith" `

  -UserPrincipalName "alice.smith@perusahaan.local" `

  -Path "OU=Staff,DC=perusahaan,DC=local" `

  -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `

  -Enabled $true `

  -ChangePasswordAtLogon $true   # Paksa ganti password saat login pertama

  

# Buat beberapa user sekaligus (contoh batch):

$users = @("alice", "bob", "charlie")

foreach ($user in $users) {

  New-ADUser `

    -Name $user `

    -SamAccountName $user `

    -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `

    -Enabled $true `

    -Path "OU=Staff,DC=perusahaan,DC=local" `

    -ChangePasswordAtLogon $true

  Write-Host "Created user: $user"

}

  

# Verifikasi user sudah dibuat:

Get-ADUser -Filter * -SearchBase "OU=Staff,DC=perusahaan,DC=local" | Select-Object Name, SamAccountName, Enabled

```

  

### Buat Group dan Tambahkan Member:

  

```powershell

# Buat grup

New-ADGroup `

  -Name "IT_Admins" `

  -GroupScope Global `

  -GroupCategory Security `

  -Path "OU=IT,DC=perusahaan,DC=local" `

  -Description "IT Administrator Group"

  

# Tambahkan user ke grup

Add-ADGroupMember -Identity "IT_Admins" -Members "alice.smith"

  

# Cek member grup

Get-ADGroupMember -Identity "IT_Admins" | Select-Object Name, SamAccountName

  

# Cek user ada di grup apa saja

Get-ADPrincipalGroupMembership -Identity "alice.smith" | Select-Object Name

```

  

---

  

## BAGIAN 4 — Hardening Akun Privileged AD

  

### 4A. Amankan Domain Admins (LANGKAH PERTAMA!)

  

```powershell

# WAJIB CEK PERTAMA: Siapa saja di Domain Admins?

Get-ADGroupMember -Identity "Domain Admins" | Select-Object Name, SamAccountName

  

# Kalau ada user yang seharusnya tidak ada → hapus!

Remove-ADGroupMember -Identity "Domain Admins" -Members "namauser" -Confirm:$false

Write-Host "Removed namauser from Domain Admins"

  

# Prinsip: Domain Admins HANYA berisi 1-2 akun admin yang benar-benar diperlukan!

# Verifikasi setelah hapus:

Get-ADGroupMember -Identity "Domain Admins" | Select-Object Name, SamAccountName

```

  

### 4B. Cek dan Kosongkan Enterprise Admins & Schema Admins

  

```powershell

# Harusnya kosong (tidak ada anggota), kecuali saat dibutuhkan:

Write-Host "=== Enterprise Admins ==="

Get-ADGroupMember -Identity "Enterprise Admins"

  

Write-Host "=== Schema Admins ==="

Get-ADGroupMember -Identity "Schema Admins"

  

# Kalau ada anggota yang tidak perlu → hapus:

Remove-ADGroupMember -Identity "Enterprise Admins" -Members "namauser" -Confirm:$false

Remove-ADGroupMember -Identity "Schema Admins" -Members "namauser" -Confirm:$false

```

  

### 4C. Tambahkan Admin ke Protected Users Group

  

Protected Users Group memberikan perlindungan ekstra pada akun admin:

- Tidak bisa menggunakan NTLM (hanya Kerberos) → cegah Pass-the-Hash

- Tidak bisa cache credentials → proteksi lebih kuat

- Ticket Kerberos lebih pendek → window serangan lebih kecil

  

```powershell

# Tambahkan domain admin ke Protected Users

Add-ADGroupMember -Identity "Protected Users" -Members "nama_admin"

  

# Verifikasi:

Get-ADGroupMember -Identity "Protected Users" | Select-Object Name, SamAccountName

  

# ⚠️ PERHATIAN: Jangan masukkan service account ke Protected Users!

# Service account yang pakai NTLM akan gagal autentikasi setelah ini.

```

  

### 4D. Nonaktifkan Akun Default yang Berbahaya

  

```powershell

# Nonaktifkan built-in Guest di AD

Disable-ADAccount -Identity "Guest"

Write-Host "Guest account disabled"

  

# Nonaktifkan user yang tidak aktif > 90 hari

$cutoff = (Get-Date).AddDays(-90)

Write-Host "=== User tidak aktif >90 hari ==="

Get-ADUser -Filter * -Properties LastLogonDate |

  Where-Object { $_.LastLogonDate -lt $cutoff -and $_.Enabled -eq $true } |

  Select-Object Name, LastLogonDate |

  Sort-Object LastLogonDate

  

# Setelah verifikasi, nonaktifkan:

Get-ADUser -Filter * -Properties LastLogonDate |

  Where-Object { $_.LastLogonDate -lt $cutoff -and $_.Enabled -eq $true } |

  ForEach-Object {

    Disable-ADAccount -Identity $_

    Write-Host "Disabled: $($_.Name) (Last logon: $($_.LastLogonDate))"

  }

```

  

> ⚠️ **Hati-hati!** Beberapa service account mungkin tidak pernah login interaktif tapi tetap aktif.

> Selalu verifikasi manual dulu sebelum mass-disable!

  

### 4E. Perbaiki User dengan PasswordNeverExpires (BAHAYA!)

  

```powershell

# Lihat user yang passwordnya tidak pernah expired

Write-Host "=== User dengan PasswordNeverExpires ==="

Get-ADUser -Filter { PasswordNeverExpires -eq $true } |

  Select-Object Name, SamAccountName, PasswordNeverExpires

  

# Perbaiki semua sekaligus (kecuali service account yang memang perlu):

Get-ADUser -Filter { PasswordNeverExpires -eq $true } |

  ForEach-Object {

    Set-ADUser $_ -PasswordNeverExpires $false

    Write-Host "Fixed: $($_.Name)"

  }

  

# Verifikasi:

Get-ADUser -Filter { PasswordNeverExpires -eq $true } | Select-Object Name

# Harusnya kosong (atau hanya service account yang terdokumentasi)

```

  

### 4F. Perbaiki User dengan PasswordNotRequired (SANGAT BAHAYA!)

  

```powershell

# User yang tidak perlu password = celah besar!

Write-Host "=== User dengan PasswordNotRequired ==="

Get-ADUser -Filter { PasswordNotRequired -eq $true } |

  Select-Object Name, SamAccountName

  

# Perbaiki:

Get-ADUser -Filter { PasswordNotRequired -eq $true } |

  ForEach-Object {

    Set-ADUser $_ -PasswordNotRequired $false

    Write-Host "Fixed: $($_.Name)"

  }

  

# Verifikasi:

Get-ADUser -Filter { PasswordNotRequired -eq $true } | Select-Object Name

# Harusnya kosong!

```

  

---

  

## BAGIAN 5 — Fine-Grained Password Policy (PSO)

  

> PSO = Password Policy berbeda untuk grup berbeda.

> Contoh: Domain Admins harus pakai password minimal 16 karakter,

> sedangkan user biasa cukup 12 karakter.

>

> **Syarat:** Domain Functional Level minimal Windows Server 2008.

  

```powershell

# Cek Domain Functional Level dulu:

(Get-ADDomain).DomainMode

  

# Buat PSO untuk admin (password lebih ketat)

New-ADFineGrainedPasswordPolicy `

  -Name "AdminPasswordPolicy" `

  -Precedence 1 `

  -MinPasswordLength 16 `

  -PasswordHistoryCount 10 `

  -ComplexityEnabled $true `

  -MaxPasswordAge "30.00:00:00" `

  -MinPasswordAge "1.00:00:00" `

  -LockoutThreshold 3 `

  -LockoutObservationWindow "0.00:15:00" `

  -LockoutDuration "1.00:00:00" `

  -ReversibleEncryptionEnabled $false

  

# Terapkan PSO ke grup Domain Admins

Add-ADFineGrainedPasswordPolicySubject `

  -Identity "AdminPasswordPolicy" `

  -Subjects "Domain Admins"

  

# Verifikasi PSO berlaku untuk user tertentu

Get-ADUserResultantPasswordPolicy -Identity "namaadmin"

  

# Lihat semua PSO yang ada

Get-ADFineGrainedPasswordPolicy -Filter * |

  Select-Object Name, Precedence, MinPasswordLength, LockoutThreshold

```

  

---

  

## BAGIAN 6 — Audit AD Configuration

  

```powershell

# Lihat semua user yang Enabled

Write-Host "=== User Aktif ==="

Get-ADUser -Filter { Enabled -eq $true } |

  Select-Object Name, SamAccountName, LastLogonDate |

  Sort-Object LastLogonDate

  

# Lihat semua user yang Disabled

Write-Host "=== User Non-aktif ==="

Get-ADUser -Filter { Enabled -eq $false } | Select-Object Name, SamAccountName

  

# Lihat semua user yang BELUM PERNAH login

Write-Host "=== User Belum Pernah Login ==="

Get-ADUser -Filter * -Properties LastLogonDate |

  Where-Object { $_.LastLogonDate -eq $null } |

  Select-Object Name, SamAccountName

  

# Cek semua computer yang join domain

Write-Host "=== Computer di Domain ==="

Get-ADComputer -Filter * -Properties LastLogonDate |

  Select-Object Name, OperatingSystem, LastLogonDate |

  Sort-Object LastLogonDate -Descending

```

  

---

  

## BAGIAN 7 — Protect AdminSDHolder

  

AdminSDHolder adalah objek AD yang melindungi akun-akun admin dari modifikasi tidak sah. Setiap 60 menit, Windows menyalin permission dari AdminSDHolder ke semua akun yang dilindungi.

  

```powershell

# Cek user mana yang dilindungi AdminSDHolder (adminCount=1):

Write-Host "=== User dengan adminCount=1 ==="

Get-ADUser -LDAPFilter "(adminCount=1)" | Select-Object Name, SamAccountName

  

# ⚠️ BAHAYA: Kalau ada user biasa (bukan admin) yang punya adminCount=1

# → artinya user itu pernah jadi admin dan adminCount belum di-reset

# → permissions admin masih menempel, berbahaya!

  

# Fix: reset adminCount untuk user yang sudah tidak perlu:

Set-ADUser -Identity "namauser" -Replace @{adminCount=0}

  

# Juga perbaiki inheritance permission via GUI:

# dsa.msc → View → Advanced Features → cari user → Properties

# → Security → Advanced → centang "Enable inheritance"

```

  

---

  

## BAGIAN 8 — Kerberoasting Prevention

  

Kerberoasting adalah teknik serangan di mana attacker meminta Kerberos ticket untuk service account, lalu crack passwordnya offline. Solusinya: pastikan service account pakai password panjang dan dikelola dengan baik.

  

```powershell

# Cek service account yang rentan Kerberoasting

# (punya ServicePrincipalName = SPN)

Write-Host "=== Service Account dengan SPN (rentan Kerberoasting) ==="

Get-ADUser -Filter { ServicePrincipalName -ne "$null" } -Properties ServicePrincipalName |

  Select-Object Name, SamAccountName, ServicePrincipalName

  

# Pastikan service account punya password yang sangat panjang (minimal 25 karakter!)

# Solusi terbaik: Gunakan Group Managed Service Account (gMSA):

  

# Buat KDS Root Key (hanya perlu dilakukan sekali per forest):

Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))

  

# Buat gMSA:

New-ADServiceAccount `

  -Name "svc_webapp" `

  -DNSHostName "webapp.perusahaan.local" `

  -PrincipalsAllowedToRetrieveManagedPassword "Domain Computers"

  

# Install gMSA di server yang butuh:

Install-ADServiceAccount -Identity "svc_webapp"

Test-ADServiceAccount -Identity "svc_webapp"

```

  

---

  

## BAGIAN 9 — AS-REP Roasting Prevention

  

AS-REP Roasting menyerang akun yang tidak memerlukan pre-authentication Kerberos. Ini setting yang lemah dan harus dimatikan.

  

```powershell

# Cek user yang tidak perlu pre-authentication (rentan AS-REP Roasting!):

Write-Host "=== User rentan AS-REP Roasting ==="

Get-ADUser -Filter { DoesNotRequirePreAuth -eq $true } |

  Select-Object Name, SamAccountName

  

# Perbaiki - aktifkan pre-authentication untuk semua user:

Get-ADUser -Filter { DoesNotRequirePreAuth -eq $true } |

  ForEach-Object {

    Set-ADUser $_ -DoesNotRequirePreAuth $false

    Write-Host "Fixed: $($_.Name)"

  }

  

# Verifikasi:

Get-ADUser -Filter { DoesNotRequirePreAuth -eq $true } | Select-Object Name

# Harusnya kosong!

```

  

---

  

## BAGIAN 10 — Script Audit AD Menyeluruh (Jalankan di Awal Lomba!)

  

```powershell

# ===== SCRIPT AUDIT AD — JALANKAN INI PERTAMA KALI =====

  

Write-Host "========================================" -ForegroundColor Cyan

Write-Host "    AUDIT ACTIVE DIRECTORY SECURITY     " -ForegroundColor Cyan

Write-Host "========================================" -ForegroundColor Cyan

  

Write-Host "`n[1] Domain Admins:" -ForegroundColor Yellow

Get-ADGroupMember -Identity "Domain Admins" | Select-Object Name, SamAccountName

  

Write-Host "`n[2] Enterprise Admins:" -ForegroundColor Yellow

Get-ADGroupMember -Identity "Enterprise Admins" | Select-Object Name, SamAccountName

  

Write-Host "`n[3] Schema Admins:" -ForegroundColor Yellow

Get-ADGroupMember -Identity "Schema Admins" | Select-Object Name, SamAccountName

  

Write-Host "`n[4] User PasswordNeverExpires:" -ForegroundColor Yellow

Get-ADUser -Filter { PasswordNeverExpires -eq $true } | Select-Object Name, SamAccountName

  

Write-Host "`n[5] User PasswordNotRequired:" -ForegroundColor Yellow

Get-ADUser -Filter { PasswordNotRequired -eq $true } | Select-Object Name, SamAccountName

  

Write-Host "`n[6] User rentan AS-REP Roasting:" -ForegroundColor Yellow

Get-ADUser -Filter { DoesNotRequirePreAuth -eq $true } | Select-Object Name, SamAccountName

  

Write-Host "`n[7] Service Account dengan SPN (Kerberoasting):" -ForegroundColor Yellow

Get-ADUser -Filter { ServicePrincipalName -ne "$null" } -Properties ServicePrincipalName |

  Select-Object Name, SamAccountName

  

Write-Host "`n[8] User dengan adminCount=1 (cek AdminSDHolder):" -ForegroundColor Yellow

Get-ADUser -LDAPFilter "(adminCount=1)" | Select-Object Name, SamAccountName

  

Write-Host "`n[9] User tidak aktif >90 hari:" -ForegroundColor Yellow

$cutoff = (Get-Date).AddDays(-90)

Get-ADUser -Filter * -Properties LastLogonDate |

  Where-Object { $_.LastLogonDate -lt $cutoff -and $_.Enabled -eq $true } |

  Select-Object Name, LastLogonDate | Sort-Object LastLogonDate

  

Write-Host "`n========================================" -ForegroundColor Cyan

Write-Host "            AUDIT SELESAI               " -ForegroundColor Cyan

Write-Host "========================================" -ForegroundColor Cyan

```

  

---

  

## ✅ Checklist Active Directory Security

  

**Struktur AD**

- [ ] Struktur OU dibuat dengan benar (IT, Staff, Computers, Servers)

- [ ] User ditempatkan di OU yang tepat (bukan di default Users/Computers container)

- [ ] OU dilindungi dari accidental deletion

  

**Privileged Groups**

- [ ] Domain Admins hanya berisi akun yang benar-benar diperlukan (max 2)

- [ ] Enterprise Admins kosong (kecuali saat dibutuhkan)

- [ ] Schema Admins kosong (kecuali saat dibutuhkan)

- [ ] Protected Users group berisi akun admin

  

**User Account Security**

- [ ] Guest account dinonaktifkan

- [ ] Tidak ada user dengan PasswordNeverExpires = True (kecuali service account)

- [ ] Tidak ada user dengan PasswordNotRequired = True

- [ ] Tidak ada user dengan DoesNotRequirePreAuth = True

- [ ] User tidak aktif >90 hari sudah dinonaktifkan

- [ ] adminCount di-reset untuk user yang sudah tidak perlu perlindungan admin

  

**Password Policy**

- [ ] PSO diterapkan untuk akun privileged (lebih ketat, minimal 16 karakter)

  

**Serangan Prevention**

- [ ] Service account menggunakan gMSA atau password sangat panjang (>25 karakter)

- [ ] Script audit AD sudah dijalankan dan semua temuan sudah diperbaiki

  

---

  

## 🔗 Navigasi

  

← [[W1_PAM_Local_Security]]

→ [[W3_GPO_Policy]]