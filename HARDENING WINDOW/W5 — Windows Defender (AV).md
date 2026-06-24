# W5 — Windows Defender (AV)

#lks #cyber-security #windows #hardening #antivirus #defender

  

> **Tingkat:** 🟢 Mudah | **Waktu:** 0.5 malam (1–1.5 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → AV

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

Windows Defender seperti **satpam + sistem keamanan gedung** sekaligus:

- **Real-time protection** = satpam yang berjaga 24/7 di pintu masuk

- **Cloud protection** = satpam yang terhubung ke database penjahat nasional

- **Tamper Protection** = sistem yang mencegah seseorang "menyuap" satpam agar tidur

- **ASR Rules** = peraturan ketat: "tidak boleh bawa senjata ke dalam gedung, bahkan kalau tidak ada yang melapor"

  

Tugasmu: pastikan semua sistem keamanan ini AKTIF dan tidak bisa dimatikan!

  

---

  

## 🎯 Tujuan

  

Memastikan Windows Defender aktif, dikonfigurasi optimal, dan tidak bisa dimatikan oleh attacker atau malware.

  

---

  

## BAGIAN 1 — Cek Status Defender (LANGKAH PERTAMA!)

  

```powershell

# Cek status lengkap Windows Defender

Get-MpComputerStatus

  

# Yang perlu diperhatikan dari output:

# AntivirusEnabled              : True   ← HARUS True

# RealTimeProtectionEnabled     : True   ← HARUS True

# IoavProtectionEnabled         : True   ← HARUS True (scan downloads)

# AntispywareEnabled            : True   ← HARUS True

# BehaviorMonitorEnabled        : True   ← HARUS True

# NISEnabled                    : True   ← HARUS True (Network Inspection System)

# AntivirusSignatureLastUpdated : [tanggal recent] ← Cek tanggalnya!

# AntivirusSignatureAge         : [harus < 7 hari]

  

# Quick summary — hanya tampilkan yang penting:

Get-MpComputerStatus | Select-Object `

  AntivirusEnabled, RealTimeProtectionEnabled, BehaviorMonitorEnabled, `

  IoavProtectionEnabled, NISEnabled, AntispywareEnabled, `

  AntivirusSignatureLastUpdated, AntivirusSignatureAge

```

  

> 🔴 **Jika ada yang False:** Segera perbaiki! Defender yang mati = sistem tidak terlindungi.

  

---

  

## BAGIAN 2 — Aktifkan Semua Fitur Defender

  

```powershell

# Aktifkan real-time protection

Set-MpPreference -DisableRealtimeMonitoring $false

  

# Aktifkan cloud protection (Basic=1, Advanced=2)

Set-MpPreference -MAPSReporting Advanced

  

# Aktifkan automatic sample submission

Set-MpPreference -SubmitSamplesConsent SendAllSamples

  

# Aktifkan behavior monitoring

Set-MpPreference -DisableBehaviorMonitoring $false

  

# Aktifkan script scanning (scan PowerShell dan script lain)

Set-MpPreference -DisableScriptScanning $false

  

# Aktifkan archive scanning (scan file zip, rar, dll.)

Set-MpPreference -DisableArchiveScanning $false

  

# Aktifkan Network Inspection System

Set-MpPreference -DisableIntrusionPreventionSystem $false

  

# Aktifkan scan email

Set-MpPreference -DisableEmailScanning $false

  

# Update signature/definisi (SELALU lakukan ini di awal!)

Write-Host "Mengupdate signature Defender..."

Update-MpSignature

Write-Host "Update selesai!"

  

# Verifikasi semua setting:

Get-MpPreference | Select-Object `

  DisableRealtimeMonitoring, MAPSReporting, SubmitSamplesConsent, `

  DisableBehaviorMonitoring, DisableScriptScanning, DisableArchiveScanning, `

  DisableIntrusionPreventionSystem, DisableEmailScanning

# Semua Disable* harus: False

```

  

---

  

## BAGIAN 3 — Tamper Protection (Cegah Defender Dimatikan!)

  

Tamper Protection mencegah malware atau user biasa mematikan Defender.

  

**Cara aktifkan via GUI (harus via GUI, tidak bisa via PowerShell biasa):**

```

1. Klik Start → cari "Windows Security" → buka

2. Pilih "Virus & threat protection"

3. Scroll ke "Virus & threat protection settings" → klik "Manage settings"

4. Scroll ke "Tamper Protection"

5. Toggle ke ON (warna biru)

```

  

```powershell

# Cek status Tamper Protection:

Get-MpComputerStatus | Select-Object TamperProtectionSource, IsTamperProtected

# TamperProtectionSource: "Signatures" atau "ATP" = aktif

# TamperProtectionSource: "None" = perlu diaktifkan via GUI

  

# Alternatif cek via registry:

Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows Defender\Features" `

  -Name "TamperProtection" -ErrorAction SilentlyContinue

# Nilai 5 = aktif, 4 = tidak aktif

```

  

> ⚠️ **Tamper Protection TIDAK BISA diaktifkan via PowerShell secara langsung.**

> Harus via GUI Windows Security atau Microsoft Intune (MDM).

  

---

  

## BAGIAN 4 — Attack Surface Reduction (ASR) Rules

  

ASR rules memblokir perilaku yang biasanya dipakai malware, bahkan kalau malwarenya belum punya signature di database!

  

Contoh: "Blokir Office dari menjalankan PowerShell" → meskipun malware baru, behavior ini langsung diblokir.

  

```powershell

# Mode: 0=Off, 1=Block, 2=Audit (log saja, tidak blokir), 6=Warn

# Di lomba: langsung pakai mode 1 (Block) untuk semua rule

  

$ASRRules = @{

  # Blokir content executable dari email

  "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550" = 1

  

  # Blokir semua Office child processes (Word tidak boleh buka cmd!)

  "D4F940AB-401B-4EFC-AADC-AD5F3C50688A" = 1

  

  # Blokir Office dari membuat executable content

  "3B576869-A4EC-4529-8536-B80A7769E899" = 1

  

  # Blokir Office dari inject code ke process lain

  "75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84" = 1

  

  # Blokir JavaScript/VBScript menjalankan downloaded content

  "D3E037E1-3EB8-44C8-A917-57927947596D" = 1

  

  # Blokir eksekusi script yang di-obfuscate

  "5BEB7EFE-FD9A-4556-801D-275E5FFC04CC" = 1

  

  # Blokir Win32 API calls dari Office macros

  "92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B" = 1

  

  # Blokir executable files dari email client dan webmail

  "01443614-CD74-433A-B99E-2ECDC07BFC25" = 1

  

  # Advanced protection against ransomware

  "C1DB55AB-C21A-4637-BB3F-A12568109D35" = 1

  

  # Blokir credential stealing dari LSASS (cegah mimikatz!)

  "9E6C4E1F-7D60-472F-BA1A-A39EF669E4B0" = 1

  

  # Blokir persistence through WMI event subscription

  "E6DB77E5-3DF2-4CF1-B95A-636979351E5B" = 1

  

  # Blokir proses dari PSExec dan WMI commands (lateral movement)

  "D1E49AAC-8F56-4280-B9BA-993A6D77406C" = 1

}

  

Write-Host "Mengaktifkan ASR Rules..."

foreach ($rule in $ASRRules.GetEnumerator()) {

  Add-MpPreference `

    -AttackSurfaceReductionRules_Ids $rule.Key `

    -AttackSurfaceReductionRules_Actions $rule.Value

  Write-Host "ASR Rule aktif: $($rule.Key)"

}

  

# Verifikasi ASR rules aktif:

Write-Host "`nVerifikasi ASR Rules:"

$prefs = Get-MpPreference

for ($i = 0; $i -lt $prefs.AttackSurfaceReductionRules_Ids.Count; $i++) {

  $id     = $prefs.AttackSurfaceReductionRules_Ids[$i]

  $action = $prefs.AttackSurfaceReductionRules_Actions[$i]

  $status = switch ($action) {

    0 { "Off" }

    1 { "Block" }

    2 { "Audit" }

    6 { "Warn" }

  }

  Write-Host "$id : $status"

}

```

  

---

  

## BAGIAN 5 — Jadwalkan Scan Otomatis

  

```powershell

# Set scan terjadwal (setiap hari jam 2 pagi)

Set-MpPreference -ScanScheduleDay Everyday

Set-MpPreference -ScanScheduleTime "02:00:00"

Set-MpPreference -ScanParameters 2   # 1=Quick scan, 2=Full scan

  

# Aktifkan juga scan saat ada signature update baru:

Set-MpPreference -CheckForSignaturesBeforeRunningScan $true

  

# Jalankan quick scan sekarang (untuk cek langsung):

Write-Host "Menjalankan Quick Scan..."

Start-MpScan -ScanType QuickScan

  

# Lihat hasil scan terakhir:

Get-MpComputerStatus | Select-Object `

  QuickScanStartTime, QuickScanEndTime, `

  FullScanStartTime, FullScanEndTime, `

  LastQuickScanSource

```

  

---

  

## BAGIAN 6 — Periksa dan Hapus Exclusions Mencurigakan!

  

Exclusion = folder/file yang dikecualikan dari scanning. Ini celah besar yang sering dipakai attacker untuk menyembunyikan malware!

  

```powershell

# WAJIB CEK: Lihat semua exclusion yang ada

Write-Host "=== EXCLUSION YANG ADA ==="

$prefs = Get-MpPreference

  

Write-Host "Path Exclusions:"

$prefs.ExclusionPath | ForEach-Object { Write-Host "  - $_" }

  

Write-Host "Process Exclusions:"

$prefs.ExclusionProcess | ForEach-Object { Write-Host "  - $_" }

  

Write-Host "Extension Exclusions:"

$prefs.ExclusionExtension | ForEach-Object { Write-Host "  - $_" }

  

# Kalau ada exclusion yang mencurigakan → hapus!

Remove-MpPreference -ExclusionPath "C:\path\yang\mencurigakan"

Remove-MpPreference -ExclusionProcess "namaproses.exe"

Remove-MpPreference -ExclusionExtension ".ekstensi"

```

  

**Exclusion yang normal:** folder database (SQL Server data files), backup tool, antivirus lain.

  

**Exclusion yang mencurigakan:**

- `C:\Temp` — seluruh folder Temp dikecualikan!

- `C:\Users\*\AppData` — seluruh AppData!

- `C:\` — seluruh drive C!

- Folder dengan nama acak di lokasi tidak wajar

  

---

  

## BAGIAN 7 — Controlled Folder Access (Anti-Ransomware!)

  

Controlled Folder Access mencegah program yang tidak diizinkan memodifikasi file di folder penting (Documents, Pictures, dll.). Ini sangat efektif melawan ransomware!

  

```powershell

# Aktifkan Controlled Folder Access

Set-MpPreference -EnableControlledFolderAccess Enabled

  

# Cek status:

Get-MpPreference | Select-Object EnableControlledFolderAccess

# Harus: Enabled

  

# Lihat folder yang dilindungi (default sudah mencakup Desktop, Documents, dll.):

Get-MpPreference | Select-Object -ExpandProperty ControlledFolderAccessProtectedFolders

  

# Tambah folder yang ingin dilindungi:

Add-MpPreference -ControlledFolderAccessProtectedFolders "D:\DataPenting"

  

# Tambah aplikasi yang diizinkan mengakses folder terlindungi:

# (hanya kalau ada false positive — aplikasi legit yang diblokir)

Add-MpPreference -ControlledFolderAccessAllowedApplications "C:\Program Files\AppSaya\app.exe"

```

  

---

  

## BAGIAN 8 — Cek Malware yang Sudah Terdeteksi

  

```powershell

# Lihat semua threat yang pernah terdeteksi:

Get-MpThreatDetection | Select-Object `

  ThreatID, ThreatName, ActionSuccess, DetectionTime, `

  InitialDetectionTime, LastDetectedTime |

  Sort-Object DetectionTime -Descending |

  Format-Table -AutoSize

  

# Lihat detail threat spesifik:

Get-MpThreat | Select-Object ThreatID, ThreatName, SeverityID, CategoryID, IsActive

  

# Cek apakah ada threat yang belum dibersihkan:

Get-MpThreat | Where-Object { $_.IsActive -eq $true }

# Kalau ada hasil → ada malware yang masih aktif!

  

# Hapus semua threat yang terdeteksi:

Remove-MpThreat

```

  

---

  

## ✅ Checklist Windows Defender

  

**Status Dasar**

- [ ] `Get-MpComputerStatus` → semua status True/aktif

- [ ] Real-time protection aktif (`DisableRealtimeMonitoring = False`)

- [ ] Behavior monitoring aktif

- [ ] Script scanning aktif

- [ ] Archive scanning aktif

- [ ] Network Inspection System aktif

  

**Update & Cloud**

- [ ] Cloud protection = Advanced (`MAPSReporting = Advanced`)

- [ ] Signature definitions up-to-date (cek `AntivirusSignatureAge` < 7 hari)

- [ ] `Update-MpSignature` sudah dijalankan

  

**Tamper Protection**

- [ ] Tamper Protection aktif (cek via Windows Security GUI)

- [ ] `TamperProtectionSource` bukan "None"

  

**ASR Rules**

- [ ] ASR rules aktif (semua rule penting dalam mode Block = 1)

- [ ] Verifikasi `AttackSurfaceReductionRules_Ids` sudah terisi

  

**Controlled Folder Access**

- [ ] `EnableControlledFolderAccess = Enabled`

  

**Exclusions**

- [ ] Tidak ada exclusion yang mencurigakan (cek `ExclusionPath`, `ExclusionProcess`, `ExclusionExtension`)

  

**Scan**

- [ ] Scan terjadwal dikonfigurasi (setiap hari jam 2 pagi)

- [ ] Quick Scan sudah dijalankan dan hasilnya bersih

  

---

  

## 🔗 Navigasi

  

← [[W4_Network_Service_Security]]

→ [[W6_Logging_Audit]]