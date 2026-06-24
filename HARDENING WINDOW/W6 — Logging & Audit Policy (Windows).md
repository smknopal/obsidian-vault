# W6 — Logging & Audit Policy (Windows)

#lks #cyber-security #windows #hardening #logging #audit #event-viewer #sysmon

  

> **Tingkat:** 🟡 Sedang | **Waktu:** 1 malam (2–3 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → **Logging** ← INI DI KISI-KISI!

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

Bayangkan Windows seperti **gedung kantor dengan kamera CCTV**:

- Tanpa logging = CCTV tidak merekam → kalau ada maling, tidak bisa dilacak

- Dengan logging biasa = CCTV merekam tapi hanya di pintu utama

- Dengan **Sysmon** = CCTV di SETIAP sudut gedung + rekaman detail siapa sentuh apa

  

**Logging di Windows = merekam semua kejadian penting:**

- Siapa yang login (berhasil atau gagal)

- Siapa yang membuat/menghapus user

- Program apa yang dieksekusi (Sysmon)

- Koneksi jaringan apa yang dibuat (Sysmon)

  

---

  

## 🎯 Tujuan

  

- Mengaktifkan Audit Policy untuk mencatat semua event penting

- Membaca Event Log via PowerShell dan GUI

- Mengatur ukuran log agar tidak kehabisan kapasitas

- Mengaktifkan PowerShell logging untuk deteksi serangan

- Install dan konfigurasi Sysmon untuk logging lebih detail

  

---

  

## BAGIAN 1 — Mengenal Event Viewer

  

### Cara Membuka Event Viewer

  

```

Cara 1 (Tercepat): Win + R → eventvwr.msc → Enter

Cara 2: Win + R → eventvwr → Enter

```

  

### Struktur Event Viewer

  

```

Event Viewer (Local)

├── Windows Logs                ← FOKUS DI SINI!

│   ├── Application             ← Log aplikasi (error program)

│   ├── Security                ← 🔴 PALING PENTING! Login, audit

│   ├── Setup                   ← Log instalasi Windows

│   └── System                  ← Log sistem (service start/stop)

│

└── Applications and Services Logs

    └── Microsoft → Windows

        ├── PowerShell → Operational    ← Log PowerShell (penting!)

        ├── Sysmon → Operational        ← Log Sysmon (sangat detail!)

        └── Windows Defender → Operational

```

  

---

  

## BAGIAN 2 — Event ID yang WAJIB Dihafal

  

> 🧠 Setiap kejadian di Windows punya nomor ID unik. Hafalkan ini!

  

### Event ID Kritis — Security Log

  

| Event ID | Artinya | Bahaya? |

|----------|---------|---------:|

| **4624** | Login BERHASIL | Normal |

| **4625** | Login GAGAL | ⚠️ Brute force! |

| **4648** | Login dengan explicit credentials | ⚠️ Curigai |

| **4672** | Special privileges assigned (admin login) | Monitor |

| **4720** | User account DIBUAT | ⚠️ Monitor |

| **4722** | User account DIAKTIFKAN | ⚠️ Monitor |

| **4725** | User account DINONAKTIFKAN | Monitor |

| **4726** | User account DIHAPUS | ⚠️ Monitor |

| **4728** | User DITAMBAHKAN ke grup Security (domain) | 🔴 BAHAYA |

| **4732** | User DITAMBAHKAN ke grup Administrators (lokal) | 🔴 SANGAT BAHAYA |

| **4740** | User account DIKUNCI (lockout) | ⚠️ Brute force? |

| **4767** | User account di-UNLOCK | Monitor |

| **4771** | Kerberos pre-authentication GAGAL | ⚠️ AS-REP Roasting? |

| **4776** | NTLM authentication attempt | Monitor |

| **4797** | Attempt to query blank password | 🔴 BAHAYA |

| **1102** | Security log DIHAPUS | 🔴 SANGAT BAHAYA! Attacker tutup jejak! |

  

### Event ID Penting — System Log

  

| Event ID | Artinya | Bahaya? |

|----------|---------|---------:|

| **7045** | Service baru DIINSTALL | ⚠️ Mungkin malware! |

| **7036** | Service start/stop | Monitor |

| **7040** | Service startup type berubah | ⚠️ Monitor |

| **6005** | Event Log service started (reboot) | Normal |

| **6006** | Event Log service stopped (shutdown) | Normal |

  

### Event ID Penting — Sysmon Log

  

| Event ID | Artinya |

|----------|---------|

| **1** | Process creation (program dieksekusi) |

| **3** | Network connection (koneksi jaringan dibuat) |

| **7** | Image loaded (DLL dimuat) |

| **8** | CreateRemoteThread (injeksi kode — bahaya!) |

| **11** | File created |

| **12/13** | Registry key dibuat/diubah |

| **22** | DNS query |

  

---

  

## BAGIAN 3 — Mengaktifkan Audit Policy (WAJIB!)

  

### Via auditpol (TERCEPAT — Pakai ini saat lomba!)

  

```powershell

# Cek kondisi saat ini:

auditpol /get /category:*

  

# ===== SET SEMUA SEKALIGUS (COPY-PASTE INI!) =====

  

# Account Logon (login ke domain/lokal)

auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable

auditpol /set /subcategory:"Kerberos Authentication Service" /success:enable /failure:enable

auditpol /set /subcategory:"Kerberos Service Ticket Operations" /success:enable /failure:enable

  

# Account Management (buat/hapus/ubah user)

auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable

auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable

auditpol /set /subcategory:"Computer Account Management" /success:enable /failure:enable

  

# Logon/Logoff

auditpol /set /subcategory:"Logon" /success:enable /failure:enable

auditpol /set /subcategory:"Logoff" /success:enable

auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable

auditpol /set /subcategory:"Special Logon" /success:enable

  

# Object Access

auditpol /set /subcategory:"File System" /failure:enable

auditpol /set /subcategory:"Registry" /failure:enable

  

# Policy Change

auditpol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable

auditpol /set /subcategory:"Authentication Policy Change" /success:enable /failure:enable

  

# Privilege Use

auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable

  

# Detailed Tracking

auditpol /set /subcategory:"Process Creation" /success:enable

  

# System

auditpol /set /subcategory:"Security State Change" /success:enable /failure:enable

auditpol /set /subcategory:"Security System Extension" /success:enable /failure:enable

auditpol /set /subcategory:"System Integrity" /success:enable /failure:enable

  

# ===== VERIFIKASI SEMUA SUDAH AKTIF =====

auditpol /get /category:*

# Cari yang masih "No Auditing" di kategori penting!

```

  

### Via GPO (untuk domain environment):

  

```

gpedit.msc → Computer Configuration → Windows Settings

→ Security Settings → Local Policies → Audit Policy

  

Atur semua ke "Success, Failure":

✅ Audit account logon events

✅ Audit account management

✅ Audit logon events

✅ Audit policy change

✅ Audit privilege use

✅ Audit system events

✅ Audit process tracking  ← Set ke Success saja (Failure terlalu banyak log)

  

Untuk kontrol lebih granular:

Security Settings → Advanced Audit Policy Configuration → Audit Policies

(ini lebih detail dan disarankan)

```

  

---

  

## BAGIAN 4 — Membaca Log via PowerShell

  

```powershell

# ===== CEK LOG CEPAT =====

  

# Login gagal (Event 4625) — cek brute force:

Get-WinEvent -LogName Security -MaxEvents 200 |

  Where-Object { $_.Id -eq 4625 } |

  Select-Object TimeCreated, @{N='Message'; E={$_.Message}} -First 10

  

# User baru dibuat (Event 4720):

Get-WinEvent -LogName Security |

  Where-Object { $_.Id -eq 4720 } |

  Select-Object TimeCreated, Message

  

# Ditambahkan ke Administrators (Event 4732):

Get-WinEvent -LogName Security |

  Where-Object { $_.Id -eq 4732 } |

  Select-Object TimeCreated, Message

  

# Log dihapus (Event 1102) — BAHAYA!:

Get-WinEvent -LogName Security |

  Where-Object { $_.Id -eq 1102 } |

  Select-Object TimeCreated, Message

  

# Service baru (Event 7045) — kemungkinan malware:

Get-WinEvent -LogName System -MaxEvents 200 |

  Where-Object { $_.Id -eq 7045 } |

  Select-Object TimeCreated, Message

  

# ===== CEK LOG DALAM RENTANG WAKTU =====

$mulai = (Get-Date).AddHours(-24)  # 24 jam terakhir

Get-WinEvent -FilterHashtable @{

  LogName   = 'Security'

  Id        = 4625

  StartTime = $mulai

} | Select-Object TimeCreated, Message

  

# ===== EXPORT LOG KE FILE =====

# Export ke CSV untuk analisis lebih lanjut:

Get-WinEvent -LogName Security -MaxEvents 1000 |

  Where-Object { $_.Id -in @(4625, 4624, 4720, 4732, 1102) } |

  Select-Object TimeCreated, Id, Message |

  Export-Csv -Path "C:\security-audit.csv" -NoTypeInformation

Write-Host "Log diekspor ke C:\security-audit.csv"

```

  

---

  

## BAGIAN 5 — Atur Ukuran Log (WAJIB!)

  

> Log yang penuh akan menimpa log lama → bukti hilang!

  

```powershell

# Set ukuran log (dalam bytes):

wevtutil sl Security /ms:1073741824    # 1 GB = 1,073,741,824 bytes

wevtutil sl System /ms:524288000       # 500 MB

wevtutil sl Application /ms:524288000  # 500 MB

  

# Set behavior: archive (simpan log lama) bukan overwrite

wevtutil sl Security /rt:false         # false = do not overwrite (archive)

wevtutil sl System /rt:false

wevtutil sl Application /rt:false

  

# Verifikasi ukuran log:

Write-Host "=== Ukuran Log Saat Ini ==="

wevtutil gl Security | Select-String "maxSize|logSize"

wevtutil gl System | Select-String "maxSize|logSize"

  

# Atau via PowerShell:

Get-WinEvent -ListLog Security, System, Application |

  Select-Object LogName, MaximumSizeInBytes, FileSize, RecordCount |

  Format-Table -AutoSize

```

  

> 💡 **Di lomba:** Pastikan log Security minimal 500 MB, idealnya 1 GB.

> Log yang kecil = bukti forensik hilang lebih cepat!

  

---

  

## BAGIAN 6 — PowerShell Logging

  

> Attacker sering pakai PowerShell — dengan logging ini, semua perintah mereka terekam!

  

```powershell

# ===== AKTIFKAN SEMUA POWERSHELL LOGGING =====

  

# 1. Script Block Logging — catat ISI semua script (termasuk yang di-decode/decrypt!)

$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"

if (-not (Test-Path $path)) { New-Item -Path $path -Force | Out-Null }

Set-ItemProperty -Path $path -Name "EnableScriptBlockLogging" -Value 1

Set-ItemProperty -Path $path -Name "EnableScriptBlockInvocationLogging" -Value 1

Write-Host "✅ Script Block Logging aktif"

  

# 2. Module Logging — catat semua module yang dipanggil

$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"

if (-not (Test-Path $path)) { New-Item -Path $path -Force | Out-Null }

Set-ItemProperty -Path $path -Name "EnableModuleLogging" -Value 1

$path2 = "$path\ModuleNames"

if (-not (Test-Path $path2)) { New-Item -Path $path2 -Force | Out-Null }

Set-ItemProperty -Path $path2 -Name "*" -Value "*"

Write-Host "✅ Module Logging aktif"

  

# 3. Transcription — simpan semua output ke file teks

$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"

if (-not (Test-Path $path)) { New-Item -Path $path -Force | Out-Null }

Set-ItemProperty -Path $path -Name "EnableTranscripting" -Value 1

Set-ItemProperty -Path $path -Name "EnableInvocationHeader" -Value 1

Set-ItemProperty -Path $path -Name "OutputDirectory" -Value "C:\PSTranscripts"

  

# Buat folder transcript:

if (-not (Test-Path "C:\PSTranscripts")) {

  New-Item -Path "C:\PSTranscripts" -ItemType Directory -Force | Out-Null

}

Write-Host "✅ Transcription aktif → C:\PSTranscripts"

  

# ===== VERIFIKASI =====

Write-Host "`n=== Verifikasi PowerShell Logging ==="

$sbPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"

$sb = Get-ItemProperty $sbPath -ErrorAction SilentlyContinue

Write-Host "Script Block Logging: $($sb.EnableScriptBlockLogging)"

  

# Baca log PowerShell (Event 4104 = script block executed):

Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 20 |

  Where-Object { $_.Id -eq 4104 } |

  Select-Object TimeCreated, Message

```

  

---

  

## BAGIAN 7 — SYSMON: Logging Level Expert! ⭐

  

> 🧠 **Apa itu Sysmon?**

> Sysmon (System Monitor) adalah tool dari Microsoft Sysinternals yang memberikan logging JAUH lebih detail dari Event Viewer biasa.

>

> Contoh: Event Viewer hanya catat "ada login gagal". Sysmon catat "program PowerShell.exe tiba-tiba membuat koneksi ke IP 1.2.3.4 di luar negeri pada jam 3 pagi".

  

### Install Sysmon

  

```powershell

# LANGKAH 1: Download Sysmon dari Microsoft Sysinternals

# URL: https://docs.microsoft.com/sysinternals/downloads/sysmon

# File: Sysmon64.exe

  

# LANGKAH 2: Download konfigurasi Sysmon (SwiftOnSecurity — paling umum dipakai)

# URL: https://github.com/SwiftOnSecurity/sysmon-config

# File: sysmonconfig-export.xml

  

# LANGKAH 3: Install Sysmon dengan konfigurasi

# Buka PowerShell sebagai Administrator, masuk ke direktori download:

cd C:\Tools

  

.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml

  

# LANGKAH 4: Verifikasi Sysmon berjalan

Get-Service Sysmon64

# Status harus: Running

  

# Cek apakah log sudah masuk:

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |

  Select-Object TimeCreated, Id, Message

```

  

### Konfigurasi Sysmon Minimal (Jika Tidak Ada File Config)

  

```xml

<!-- Simpan sebagai: sysmon-minimal.xml -->

<Sysmon schemaversion="4.82">

  <HashAlgorithms>md5,sha256,IMPHASH</HashAlgorithms>

  <EventFiltering>

  

    <!-- Event 1: Catat semua process creation (SANGAT PENTING!) -->

    <RuleGroup name="" groupRelation="or">

      <ProcessCreate onmatch="exclude">

        <!-- Exclude proses noise yang tidak penting -->

        <Image condition="is">C:\Windows\System32\wermgr.exe</Image>

        <Image condition="is">C:\Windows\System32\svchost.exe</Image>

      </ProcessCreate>

    </RuleGroup>

  

    <!-- Event 3: Catat koneksi jaringan dari proses mencurigakan -->

    <RuleGroup name="" groupRelation="or">

      <NetworkConnect onmatch="include">

        <Image condition="contains">powershell</Image>

        <Image condition="is">C:\Windows\System32\cmd.exe</Image>

        <Image condition="contains">wscript</Image>

        <Image condition="contains">cscript</Image>

      </NetworkConnect>

    </RuleGroup>

  

    <!-- Event 8: CreateRemoteThread (injeksi kode — selalu catat!) -->

    <RuleGroup name="" groupRelation="or">

      <CreateRemoteThread onmatch="exclude">

        <!-- Exclude yang normal: -->

        <SourceImage condition="is">C:\Windows\System32\werfault.exe</SourceImage>

      </CreateRemoteThread>

    </RuleGroup>

  

    <!-- Event 11: File dibuat di lokasi mencurigakan -->

    <RuleGroup name="" groupRelation="or">

      <FileCreate onmatch="include">

        <TargetFilename condition="contains">\Temp\</TargetFilename>

        <TargetFilename condition="contains">\AppData\</TargetFilename>

        <TargetFilename condition="contains">\Downloads\</TargetFilename>

      </FileCreate>

    </RuleGroup>

  

  </EventFiltering>

</Sysmon>

```

  

```powershell

# Simpan config di atas ke file, lalu install:

.\Sysmon64.exe -accepteula -i sysmon-minimal.xml

  

# Update config (tanpa reinstall):

.\Sysmon64.exe -c sysmon-minimal.xml

  

# Uninstall Sysmon (kalau perlu):

.\Sysmon64.exe -u force

```

  

### Membaca Log Sysmon

  

```powershell

# Lihat proses yang baru dieksekusi (Event 1 = Process Creation):

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |

  Where-Object { $_.Id -eq 1 } |

  Select-Object TimeCreated, Message -First 20

  

# Cari koneksi jaringan dari PowerShell (Event 3):

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |

  Where-Object { $_.Id -eq 3 -and $_.Message -like "*powershell*" } |

  Select-Object TimeCreated, Message -First 10

  

# Cari CreateRemoteThread (injeksi kode — sangat berbahaya!):

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |

  Where-Object { $_.Id -eq 8 } |

  Select-Object TimeCreated, Message

```

  

---

  

## BAGIAN 8 — Skenario Deteksi Serangan dari Log

  

### Skenario 1: Deteksi Brute Force

  

```powershell

# Cari banyak Event 4625 dalam waktu singkat:

$waktu = (Get-Date).AddMinutes(-10)

$loginGagal = Get-WinEvent -FilterHashtable @{

  LogName   = 'Security'

  Id        = 4625

  StartTime = $waktu

} -ErrorAction SilentlyContinue

  

Write-Host "Login gagal 10 menit terakhir: $($loginGagal.Count)"

if ($loginGagal.Count -gt 10) {

  Write-Host "⚠️ PERINGATAN: Kemungkinan brute force!" -ForegroundColor Red

}

  

# Hitung per user yang dicoba:

$loginGagal | ForEach-Object {

  # Property[5] = Target User Name di Event 4625

  $_.Properties[5].Value

} | Group-Object | Sort-Object Count -Descending | Select-Object -First 10

```

  

### Skenario 2: Deteksi Privilege Escalation

  

```powershell

# User tiba-tiba ditambahkan ke admin (Event 4732):

Get-WinEvent -LogName Security |

  Where-Object { $_.Id -eq 4732 } |

  Select-Object TimeCreated, Message

  

# Lihat semua Event 4672 (admin login):

Get-WinEvent -LogName Security -MaxEvents 100 |

  Where-Object { $_.Id -eq 4672 } |

  Select-Object TimeCreated, Message -First 10

```

  

### Skenario 3: Deteksi Malware Service

  

```powershell

# Service baru yang mencurigakan (Event 7045):

Get-WinEvent -LogName System -MaxEvents 500 |

  Where-Object { $_.Id -eq 7045 } |

  Select-Object TimeCreated, Message

```

  

### Skenario 4: Deteksi Log Tampering

  

```powershell

# Attacker hapus log (Event 1102) — SANGAT SERIUS:

$logDeleted = Get-WinEvent -LogName Security |

  Where-Object { $_.Id -eq 1102 } |

  Select-Object TimeCreated, Message

  

if ($logDeleted) {

  Write-Host "🔴 BAHAYA: Security Log pernah dihapus!" -ForegroundColor Red

  $logDeleted

} else {

  Write-Host "✅ Tidak ada penghapusan log" -ForegroundColor Green

}

```

  

---

  

## BAGIAN 9 — Script Audit Logging Cepat

  

```powershell

# ===== JALANKAN INI DI AWAL LOMBA UNTUK CEK KONDISI LOGGING =====

  

Write-Host "========================================" -ForegroundColor Cyan

Write-Host "        AUDIT LOGGING STATUS            " -ForegroundColor Cyan

Write-Host "========================================" -ForegroundColor Cyan

  

Write-Host "`n[1] Status Audit Policy:" -ForegroundColor Yellow

auditpol /get /category:*

  

Write-Host "`n[2] Ukuran Event Log:" -ForegroundColor Yellow

Get-WinEvent -ListLog Security, System, Application |

  Select-Object LogName, MaximumSizeInBytes, FileSize |

  Format-Table -AutoSize

  

Write-Host "`n[3] PowerShell Script Block Logging:" -ForegroundColor Yellow

$sbLogging = Get-ItemProperty `

  "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `

  -ErrorAction SilentlyContinue

if ($sbLogging.EnableScriptBlockLogging -eq 1) {

  Write-Host "✅ AKTIF" -ForegroundColor Green

} else {

  Write-Host "❌ TIDAK AKTIF — perlu diaktifkan!" -ForegroundColor Red

}

  

Write-Host "`n[4] Status Sysmon:" -ForegroundColor Yellow

$sysmon = Get-Service Sysmon64 -ErrorAction SilentlyContinue

if ($sysmon) {

  Write-Host "Sysmon64: $($sysmon.Status)"

} else {

  Write-Host "❌ Sysmon tidak terinstall" -ForegroundColor Red

}

  

Write-Host "`n[5] 10 Login Gagal Terakhir (Event 4625):" -ForegroundColor Yellow

Get-WinEvent -LogName Security -MaxEvents 200 -ErrorAction SilentlyContinue |

  Where-Object { $_.Id -eq 4625 } |

  Select-Object TimeCreated -First 10

  

Write-Host "`n[6] Cek Log Dihapus (Event 1102):" -ForegroundColor Yellow

$logDel = Get-WinEvent -LogName Security -ErrorAction SilentlyContinue |

  Where-Object { $_.Id -eq 1102 }

if ($logDel) {

  Write-Host "🔴 LOG PERNAH DIHAPUS!" -ForegroundColor Red

} else {

  Write-Host "✅ Tidak ada penghapusan log" -ForegroundColor Green

}

  

Write-Host "`n[7] Service Baru (Event 7045):" -ForegroundColor Yellow

Get-WinEvent -LogName System -MaxEvents 200 -ErrorAction SilentlyContinue |

  Where-Object { $_.Id -eq 7045 } |

  Select-Object TimeCreated, Message -First 5

  

Write-Host "`n========================================" -ForegroundColor Cyan

Write-Host "          AUDIT SELESAI                 " -ForegroundColor Cyan

Write-Host "========================================" -ForegroundColor Cyan

```

  

---

  

## ✅ Checklist Logging & Audit

  

**Audit Policy**

- [ ] `auditpol /get /category:*` sudah diperiksa

- [ ] Account Logon: Success + Failure aktif

- [ ] Account Management: Success + Failure aktif

- [ ] Logon/Logoff: Success + Failure aktif

- [ ] Policy Change: Success + Failure aktif

- [ ] System: Success + Failure aktif

- [ ] Process Creation: Success aktif

  

**Event Log Size**

- [ ] Security log: minimal 500 MB (idealnya 1 GB)

- [ ] System log: minimal 500 MB

- [ ] Log behavior: Archive when full (bukan overwrite!)

  

**PowerShell Logging**

- [ ] Script Block Logging aktif (registry `EnableScriptBlockLogging = 1`)

- [ ] Module Logging aktif

- [ ] Transcription aktif → folder `C:\PSTranscripts` ada

- [ ] Verifikasi: ada Event 4104 di PowerShell/Operational setelah jalankan script

  

**Sysmon**

- [ ] Sysmon64.exe terdownload dan terinstall

- [ ] Config file tersedia (SwiftOnSecurity atau custom)

- [ ] `Get-Service Sysmon64` → Running

- [ ] Log Sysmon ada: Event ID 1 muncul setelah ada process baru

  

**Kemampuan Baca Log**

- [ ] Bisa filter Event 4625 (login gagal) dan hitung per user

- [ ] Bisa filter Event 4732 (ditambah ke admin)

- [ ] Bisa filter Event 7045 (service baru)

- [ ] Bisa filter Event 1102 (log dihapus)

- [ ] Bisa filter berdasarkan rentang waktu dengan `StartTime`

  

---

  

## 🔗 Navigasi

  

← [[W5_Windows_Defender]]

→ [[W7_LAPS]]