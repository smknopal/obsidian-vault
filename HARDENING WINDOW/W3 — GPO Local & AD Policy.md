# W3 — GPO: Local & AD Policy

#lks #cyber-security #windows #hardening #gpo #group-policy

  

> **Tingkat:** 🟡 Sedang | **Waktu:** 2 malam (4–5 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → GPO Local/AD Policy

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

GPO (Group Policy Object) seperti **peraturan kantor resmi** yang berlaku otomatis:

- Kamu buat 1 aturan → berlaku untuk SEMUA komputer/user di bawahnya sekaligus

- Tidak perlu setting satu-satu di tiap komputer

- Berlaku saat komputer startup atau user login

  

Contoh: Kamu buat GPO "matikan Telnet di semua komputer" → semua 100 komputer langsung mematuhinya tanpa kamu perlu pergi ke masing-masing.

  

---

  

## 🎯 Tujuan

  

- Menerapkan security policy secara terpusat

- Mengontrol apa yang bisa/tidak bisa dilakukan user dan komputer

- Enforce password, audit, firewall, software restriction via GPO

  

---

  

## BAGIAN 1 — Perbedaan Local GPO vs Domain GPO

  

| | Local GPO (`gpedit.msc`) | Domain GPO (`gpmc.msc`) |

|--|--------------------------|------------------------|

| Berlaku di | 1 komputer saja | Semua komputer di domain |

| Butuh AD? | Tidak | Ya (harus join domain) |

| Cocok untuk | Standalone PC/Server | AD environment |

| Lokasi file | `C:\Windows\System32\GroupPolicy` | SYSVOL di DC |

  

### Aturan Prioritas GPO (Penting!)

```

Local → Site → Domain → OU (OU paling spesifik = menang!)

```

  

Artinya: GPO di OU mengalahkan GPO di Domain, yang mengalahkan GPO Local.

Kalau ada konflik, GPO yang lebih spesifik (dekat ke user/komputer) yang menang.

  

> 💡 **Enforcement:** Bisa override prioritas ini dengan **Enforced** di GPMC.

> GPO yang di-Enforced tidak bisa di-override oleh GPO di level lebih bawah.

  

---

  

## BAGIAN 2 — Local Group Policy (gpedit.msc)

  

**Buka:** `Win + R → gpedit.msc`

  

```

Computer Configuration    ← Setting untuk komputer (berlaku saat boot)

└── Windows Settings

    └── Security Settings

        ├── Account Policies     (Password + Lockout)

        ├── Local Policies

        │   ├── Audit Policy

        │   ├── User Rights Assignment

        │   └── Security Options

        └── Windows Firewall with Advanced Security

  

User Configuration        ← Setting untuk user (berlaku saat login)

```

  

---

  

## BAGIAN 3 — Domain GPO via GPMC (gpmc.msc)

  

**Buka:** `Win + R → gpmc.msc`

  

### Cara Buat GPO Baru (Step by Step):

  

```

LANGKAH 1: Buka gpmc.msc

LANGKAH 2: Expand Forest → Domains → perusahaan.local

LANGKAH 3: Klik kanan "Group Policy Objects" → New

LANGKAH 4: Beri nama GPO yang deskriptif, contoh: "Security Hardening Policy"

LANGKAH 5: Klik kanan GPO yang baru → Edit (buka Group Policy Editor)

LANGKAH 6: Edit settingnya sesuai kebutuhan

LANGKAH 7: Tutup Group Policy Editor

LANGKAH 8: Kembali ke GPMC

LANGKAH 9: Drag GPO ke OU yang ingin diterapkan

           ATAU: klik kanan OU → Link an Existing GPO → pilih GPO

LANGKAH 10: Jalankan gpupdate /force di komputer target

```

  

---

  

## BAGIAN 4 — Security Settings via GPO (Yang Sering Diuji!)

  

### 4A. Password & Account Lockout Policy

  

**Lokasi:** `Computer Config → Windows Settings → Security Settings → Account Policies`

  

```

Password Policy:

✅ Enforce password history:                     5

✅ Maximum password age:                         90 days

✅ Minimum password age:                         1 day

✅ Minimum password length:                      12

✅ Password must meet complexity requirements:   Enabled

✅ Store passwords using reversible encryption:  Disabled

  

Account Lockout Policy:

✅ Account lockout duration:                     15 minutes

✅ Account lockout threshold:                    5

✅ Reset account lockout counter after:          15 minutes

```

  

> ⚠️ **Penting:** Di AD, Password Policy di **Default Domain Policy** berlaku untuk semua user domain.

> GPO yang di-link ke OU **tidak** bisa override Password Policy (kecuali pakai PSO — lihat W2).

  

---

  

### 4B. Audit Policy (SANGAT Sering Diuji!)

  

**Lokasi:** `Computer Config → Windows Settings → Security Settings → Local Policies → Audit Policy`

  

```

Audit account logon events:      Success, Failure  ← Login ke domain

Audit account management:        Success, Failure  ← Buat/hapus/ubah user

Audit directory service access:  Failure           ← Akses objek AD

Audit logon events:              Success, Failure  ← Login lokal

Audit object access:             Failure           ← Akses file/folder

Audit policy change:             Success, Failure  ← Perubahan policy

Audit privilege use:             Failure           ← Penggunaan hak istimewa

Audit process tracking:          Success           ← Untuk deteksi malware

Audit system events:             Success, Failure  ← Startup/shutdown

```

  

> 💡 **Versi lebih detail:** Gunakan **Advanced Audit Policy Configuration** di GPO untuk

> kontrol yang lebih granular (subcategory). Lihat W6 untuk detail audit policy lengkap.

  

**Cara lebih cepat via PowerShell:**

```powershell

# Set audit policy sekaligus (jalankan sebagai Administrator):

auditpol /set /subcategory:"Logon" /success:enable /failure:enable

auditpol /set /subcategory:"Logoff" /success:enable

auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable

auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable

auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable

auditpol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable

auditpol /set /subcategory:"System Integrity" /success:enable /failure:enable

auditpol /set /subcategory:"Process Creation" /success:enable

  

# Verifikasi:

auditpol /get /category:*

```

  

---

  

### 4C. Security Options Penting

  

**Lokasi:** `Computer Config → Windows Settings → Security Settings → Local Policies → Security Options`

  

```

✅ Accounts: Administrator account status                        → Disabled

✅ Accounts: Guest account status                                → Disabled

✅ Accounts: Rename administrator account                        → [nama baru]

✅ Accounts: Rename guest account                               → [nama baru]

  

✅ Interactive logon: Do not display last user name             → Enabled

✅ Interactive logon: Message text for users attempting to log on

   → "Authorized users only. All activity is monitored."

✅ Interactive logon: Message title for users attempting to log on

   → "WARNING"

✅ Interactive logon: Machine inactivity limit                  → 900 seconds

  

✅ Network access: Do not allow anonymous enumeration of SAM accounts

   → Enabled

✅ Network access: Do not allow anonymous enumeration of SAM accounts and shares

   → Enabled

✅ Network access: Restrict anonymous access to Named Pipes and Shares

   → Enabled

  

✅ Network security: LAN Manager authentication level

   → "Send NTLMv2 response only. Refuse LM & NTLM"

✅ Network security: Minimum session security for NTLM SSP (for clients)

   → Require NTLMv2 session security

   → Require 128-bit encryption

✅ Network security: Minimum session security for NTLM SSP (for servers)

   → Require NTLMv2 session security

   → Require 128-bit encryption

  

✅ Microsoft network client: Digitally sign communications (always)  → Enabled

✅ Microsoft network server: Digitally sign communications (always)  → Enabled

```

  

---

  

### 4D. Windows Firewall via GPO

  

**Lokasi:** `Computer Config → Windows Settings → Security Settings → Windows Firewall with Advanced Security`

  

**Atau via PowerShell:**

```powershell

# Aktifkan firewall untuk semua profile

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

  

# Set default inbound: block semua yang tidak ada rule-nya

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultInboundAction Block

  

# Set default outbound: izinkan semua keluar

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultOutboundAction Allow

  

# Aktifkan logging firewall (penting untuk forensik!)

Set-NetFirewallProfile -Profile Domain,Public,Private `

  -LogAllowed True -LogBlocked True `

  -LogFileName "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `

  -LogMaxSizeKilobytes 32767

  

# Contoh tambah rule:

New-NetFirewallRule -DisplayName "Allow RDP from LAN" `

  -Direction Inbound -Protocol TCP -LocalPort 3389 `

  -RemoteAddress "192.168.1.0/24" -Action Allow

  

# Cek semua rule aktif:

Get-NetFirewallRule | Where-Object { $_.Enabled -eq 'True' } |

  Select-Object DisplayName, Direction, Action | Sort-Object Direction

```

  

---

  

### 4E. AppLocker (Whitelist Aplikasi)

  

AppLocker memungkinkan kamu menentukan HANYA aplikasi tertentu yang boleh dijalankan.

  

**Lokasi di GPO:**

```

Computer Config → Windows Settings → Security Settings

→ Application Control Policies → AppLocker

```

  

```

Cara setup AppLocker dasar:

1. Expand AppLocker → klik "Configure rule enforcement"

2. Centang "Configured" untuk Executable Rules → set ke "Enforce rules"

3. Klik kanan "Executable Rules" → Create Default Rules

   (rule default: admin boleh jalankan semua, user hanya dari Windows folder + Program Files)

4. Tambah rule khusus kalau ada aplikasi yang perlu di-allow/block

```

  

> ⚠️ **AppLocker hanya tersedia di Windows Enterprise/Education/Server!**

> Tidak bisa dipakai di Windows Pro.

  

**Via PowerShell:**

```powershell

# Aktifkan service AppLocker (wajib!)

Set-Service -Name "AppIDSvc" -StartupType Automatic

Start-Service -Name "AppIDSvc"

  

# Verifikasi service:

Get-Service AppIDSvc

# Status harus: Running

```

  

---

  

### 4F. USB/Removable Media Restriction (Sering Diuji!)

  

**Lokasi di GPO:**

```

Computer Config → Administrative Templates → System → Removable Storage Access

```

  

```

✅ All Removable Storage classes: Deny all access  → Enabled

   (atau lebih spesifik:)

✅ Removable Disks: Deny read access               → Enabled

✅ Removable Disks: Deny write access              → Enabled

```

  

**Via Registry/PowerShell:**

```powershell

# Blokir semua USB storage device

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\USBSTOR" `

  -Name "Start" -Value 4

# Nilai: 4 = disabled, 3 = enabled (default)

  

# Verifikasi:

$usbStart = Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\USBSTOR" -Name "Start"

Write-Host "USBSTOR Start value: $($usbStart.Start)"

# Harus: 4

```

  

---

  

### 4G. Screen Lock / Session Timeout

  

**User Config:**

```

User Config → Administrative Templates → Control Panel → Personalization

→ "Enable screen saver"                    → Enabled

→ "Screen saver timeout"                   → 600 (10 menit)

→ "Password protect the screen saver"      → Enabled

→ "Force specific screen saver"            → Enabled → ketik: scrnsave.scr

```

  

**Computer Config:**

```

Computer Config → Windows Settings → Security Settings → Local Policies → Security Options

→ "Interactive logon: Machine inactivity limit" → 900 (15 menit)

```

  

---

  

### 4H. Disable LLMNR dan NetBIOS via GPO

  

LLMNR dan NetBIOS bisa dieksploitasi dengan tool **Responder** untuk mencuri hash password (MITM attack).

  

**Disable LLMNR via GPO:**

```

Computer Config → Administrative Templates → Network → DNS Client

→ "Turn off multicast name resolution" → Enabled

```

  

**Disable NetBIOS via PowerShell:**

```powershell

# Disable NetBIOS over TCP/IP untuk semua adapter

$adapters = Get-WmiObject Win32_NetworkAdapterConfiguration |

  Where-Object { $_.IPEnabled -eq $true }

foreach ($adapter in $adapters) {

  $result = $adapter.SetTcpipNetbios(2)  # 2 = disable NetBIOS

  if ($result.ReturnValue -eq 0) {

    Write-Host "NetBIOS disabled on: $($adapter.Description)"

  } else {

    Write-Host "GAGAL disable NetBIOS pada: $($adapter.Description)"

  }

}

```

  

---

  

### 4I. PowerShell Execution Policy via GPO

  

```

Lokasi: Computer Config → Administrative Templates

        → Windows Components → Windows PowerShell

→ "Turn on Script Execution" → Enabled

→ Execution Policy: "Allow only signed scripts" atau "RemoteSigned"

```

  

Atau via PowerShell:

```powershell

# Set execution policy lebih ketat

Set-ExecutionPolicy RemoteSigned -Scope LocalMachine -Force

  

# Verifikasi:

Get-ExecutionPolicy -List

# LocalMachine harus: RemoteSigned (atau AllSigned untuk lebih ketat)

```

  

---

  

### 4J. Disable Legacy Protocols via GPO (TAMBAHAN PENTING!)

  

```powershell

# Disable TLS 1.0 dan 1.1 (protokol enkripsi lama yang lemah)

# Aktifkan hanya TLS 1.2 dan 1.3

  

$regBase = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols"

  

# Disable TLS 1.0

New-Item -Path "$regBase\TLS 1.0\Server" -Force

Set-ItemProperty -Path "$regBase\TLS 1.0\Server" -Name "Enabled" -Value 0

Set-ItemProperty -Path "$regBase\TLS 1.0\Server" -Name "DisabledByDefault" -Value 1

New-Item -Path "$regBase\TLS 1.0\Client" -Force

Set-ItemProperty -Path "$regBase\TLS 1.0\Client" -Name "Enabled" -Value 0

Set-ItemProperty -Path "$regBase\TLS 1.0\Client" -Name "DisabledByDefault" -Value 1

  

# Disable TLS 1.1

New-Item -Path "$regBase\TLS 1.1\Server" -Force

Set-ItemProperty -Path "$regBase\TLS 1.1\Server" -Name "Enabled" -Value 0

Set-ItemProperty -Path "$regBase\TLS 1.1\Server" -Name "DisabledByDefault" -Value 1

New-Item -Path "$regBase\TLS 1.1\Client" -Force

Set-ItemProperty -Path "$regBase\TLS 1.1\Client" -Name "Enabled" -Value 0

Set-ItemProperty -Path "$regBase\TLS 1.1\Client" -Name "DisabledByDefault" -Value 1

  

Write-Host "TLS 1.0 dan 1.1 sudah dinonaktifkan. Restart diperlukan."

```

  

---

  

## BAGIAN 5 — Verifikasi GPO Berlaku

  

```powershell

# SELALU jalankan ini setelah edit GPO:

gpupdate /force

  

# Cek GPO apa yang berlaku untuk komputer ini:

gpresult /r

  

# Buat laporan HTML lengkap (buka di browser):

gpresult /h C:\gpo-report.html

Start-Process C:\gpo-report.html

  

# Cek GPO untuk user saat ini:

gpresult /r /scope user

  

# Cek GPO untuk komputer:

gpresult /r /scope computer

  

# Cek apakah ada error saat apply GPO (cek Event Log):

Get-WinEvent -LogName "System" |

  Where-Object { $_.ProviderName -like "*GroupPolicy*" -and $_.Level -eq 2 } |

  Select-Object TimeCreated, Message -First 10

```

  

---

  

## ✅ Checklist GPO

  

**Password & Lockout**

- [ ] Password policy dikonfigurasi via GPO (min 12 char, complexity enabled)

- [ ] Account lockout: threshold 5, duration 15 menit

  

**Audit & Logging**

- [ ] Audit policy: Logon, Account Management, Policy Change, Process Creation semua aktif

  

**Security Options**

- [ ] Administrator dan Guest dinonaktifkan via Security Options

- [ ] "Do not display last user name" = Enabled

- [ ] Banner login (message text + title) sudah diisi

- [ ] LAN Manager authentication level = NTLMv2 only

- [ ] SMB signing diaktifkan (client dan server)

- [ ] NTLM session security = NTLMv2 + 128-bit

  

**Network & Services**

- [ ] Windows Firewall aktif untuk semua profile (Domain, Public, Private)

- [ ] Firewall logging diaktifkan

- [ ] LLMNR dinonaktifkan via GPO

- [ ] NetBIOS dinonaktifkan

  

**User Experience & Session**

- [ ] Screen lock timeout dikonfigurasi (900 detik)

- [ ] USB/removable media dibatasi (jika diminta soal)

  

**Application Control**

- [ ] AppLocker diaktifkan (jika diminta soal, Windows Enterprise saja)

- [ ] PowerShell execution policy = RemoteSigned atau AllSigned

  

**Verifikasi**

- [ ] `gpupdate /force` dijalankan setelah setiap perubahan

- [ ] `gpresult /r` dijalankan untuk verifikasi

- [ ] Tidak ada error GPO di Event Log

  

---

  

## 🔗 Navigasi

  

← [[W2_Active_Directory]]

→ [[W4_Network_Service_Security]]