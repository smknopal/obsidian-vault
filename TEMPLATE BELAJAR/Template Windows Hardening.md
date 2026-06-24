# 🪟 Template Windows Hardening — Modul A

  

> **Cara pakai:** Buka PowerShell sebagai **Administrator**, lalu copy-paste blok perintah langsung.

> Untuk CMD, gunakan Command Prompt (Admin).

  

---

  

## ✅ CHECKLIST URUTAN EKSEKUSI

  

```

[ ] 1. Audit Policy (auditpol)

[ ] 2. Password & Account Policy

[ ] 3. Disable SMBv1 & legacy protocols

[ ] 4. Windows Defender / Firewall

[ ] 5. Registry Hardening (PowerShell)

[ ] 6. Script Block Logging & PowerShell Logging

[ ] 7. Disable layanan tidak perlu

[ ] 8. User & Group management

[ ] 9. Windows Update policy

[ ] 10. Verifikasi akhir

```

  

---

  

## 1. AUDIT POLICY — SAPU JAGAT

  

```cmd

REM Jalankan di CMD (Admin) — copy-paste seluruh blok ini sekaligus

  

auditpol /set /category:"Account Logon" /success:enable /failure:enable

auditpol /set /category:"Account Management" /success:enable /failure:enable

auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable

auditpol /set /category:"Object Access" /success:enable /failure:enable

auditpol /set /category:"Policy Change" /success:enable /failure:enable

auditpol /set /category:"Privilege Use" /success:enable /failure:enable

auditpol /set /category:"System" /success:enable /failure:enable

auditpol /set /category:"Detailed Tracking" /success:enable /failure:enable

auditpol /set /category:"DS Access" /success:enable /failure:enable

  

REM Verifikasi:

auditpol /get /category:*

```

  

---

  

## 2. PASSWORD & ACCOUNT POLICY

  

```cmd

REM Di CMD (Admin):

  

REM Password Policy

net accounts /minpwlen:14 /maxpwage:90 /minpwage:7 /uniquepw:5

  

REM Account Lockout Policy

net accounts /lockoutthreshold:5 /lockoutduration:30 /lockoutwindow:30

  

REM Verifikasi:

net accounts

```

  

```powershell

# Di PowerShell (Admin) — Untuk domain controller:

Import-Module ActiveDirectory

  

Set-ADDefaultDomainPasswordPolicy -Identity "yourdomain.local" `

    -MinPasswordLength 14 `

    -MaxPasswordAge (New-TimeSpan -Days 90) `

    -MinPasswordAge (New-TimeSpan -Days 7) `

    -PasswordHistoryCount 5 `

    -ComplexityEnabled $true `

    -LockoutThreshold 5 `

    -LockoutDuration (New-TimeSpan -Minutes 30) `

    -LockoutObservationWindow (New-TimeSpan -Minutes 30)

```

  

---

  

## 3. DISABLE SMBv1 & LEGACY PROTOCOLS

  

```powershell

# Disable SMBv1 (WAJIB!)

Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

Set-SmbClientConfiguration -EnableBandwidthThrottling $false -Force

  

# Verifikasi SMB:

Get-SmbServerConfiguration | Select EnableSMB1Protocol, EnableSMB2Protocol

  

# Disable LLMNR via Registry:

New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Force

Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" `

    -Name "EnableMulticast" -Value 0 -Type DWord

  

# Disable NetBIOS over TCP/IP (via registry):

$adapters = Get-WmiObject Win32_NetworkAdapterConfiguration | Where-Object {$_.IPEnabled -eq $true}

foreach ($adapter in $adapters) {

    $adapter.SetTcpipNetbios(2)  # 2 = Disable NetBIOS

}

  

# Disable WDigest (mencegah plaintext password di memory):

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" `

    -Name "UseLogonCredential" -Value 0 -Type DWord

```

  

---

  

## 4. WINDOWS FIREWALL

  

```powershell

# Aktifkan semua profile:

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

  

# Default policy:

Set-NetFirewallProfile -DefaultInboundAction Block -DefaultOutboundAction Allow -Profile All

  

# Allow RDP (jika diperlukan soal):

# New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow

  

# Block telnet:

New-NetFirewallRule -DisplayName "Block Telnet" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block

  

# Verifikasi:

Get-NetFirewallProfile | Select Name, Enabled, DefaultInboundAction

```

  

---

  

## 5. REGISTRY HARDENING — POWERSHELL SAPU JAGAT

  

```powershell

# Copy-paste seluruh blok ini sekaligus ke PowerShell Admin

  

# ==========================================

# DISABLE AUTORUN/AUTOPLAY

# ==========================================

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" `

    -Name "NoDriveTypeAutoRun" -Value 255 -Type DWord -Force

  

# ==========================================

# DISABLE WINDOWS SCRIPT HOST

# ==========================================

New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings" -Force

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows Script Host\Settings" `

    -Name "Enabled" -Value 0 -Type DWord

  

# ==========================================

# DISABLE ANONYMOUS SID ENUMERATION

# ==========================================

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `

    -Name "RestrictAnonymousSAM" -Value 1 -Type DWord

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `

    -Name "RestrictAnonymous" -Value 1 -Type DWord

  

# ==========================================

# DISABLE NTLM v1

# ==========================================

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `

    -Name "LmCompatibilityLevel" -Value 5 -Type DWord

  

# ==========================================

# DISABLE REMOTE REGISTRY

# ==========================================

Stop-Service RemoteRegistry -Force

Set-Service RemoteRegistry -StartupType Disabled

  

# ==========================================

# ENABLE UAC

# ==========================================

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `

    -Name "EnableLUA" -Value 1 -Type DWord

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `

    -Name "ConsentPromptBehaviorAdmin" -Value 2 -Type DWord

  

# ==========================================

# DISABLE WINDOWS ERROR REPORTING

# ==========================================

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting" `

    -Name "Disabled" -Value 1 -Type DWord

  

Write-Host "[+] Registry hardening selesai!" -ForegroundColor Green

```

  

---

  

## 6. POWERSHELL LOGGING — WAJIB!

  

```powershell

# Script Block Logging:

$PSLoggingPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"

New-Item -Path $PSLoggingPath -Force

Set-ItemProperty -Path $PSLoggingPath -Name "EnableScriptBlockLogging" -Value 1 -Type DWord

Set-ItemProperty -Path $PSLoggingPath -Name "EnableScriptBlockInvocationLogging" -Value 1 -Type DWord

  

# Module Logging:

$ModulePath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"

New-Item -Path $ModulePath -Force

Set-ItemProperty -Path $ModulePath -Name "EnableModuleLogging" -Value 1 -Type DWord

New-Item -Path "$ModulePath\ModuleNames" -Force

Set-ItemProperty -Path "$ModulePath\ModuleNames" -Name "*" -Value "*"

  

# Transcription (simpan semua output PS):

$TransPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"

New-Item -Path $TransPath -Force

Set-ItemProperty -Path $TransPath -Name "EnableTranscripting" -Value 1 -Type DWord

Set-ItemProperty -Path $TransPath -Name "OutputDirectory" -Value "C:\PSTranscripts"

Set-ItemProperty -Path $TransPath -Name "EnableInvocationHeader" -Value 1 -Type DWord

  

Write-Host "[+] PowerShell logging aktif!" -ForegroundColor Green

```

  

---

  

## 7. DISABLE LAYANAN TIDAK PERLU

  

```powershell

# Daftar service yang umumnya perlu di-disable:

$services = @(

    "Telnet",

    "TlntSvr",

    "FTPSVC",

    "W3SVC",       # IIS - disable jika tidak dipakai

    "RemoteRegistry",

    "WinRM",       # Hati-hati, bisa dipakai untuk remote management

    "SNMP",

    "SSDPSRV",     # SSDP Discovery

    "upnphost",    # UPnP

    "XblGameSave",

    "XblAuthManager"

)

  

foreach ($svc in $services) {

    $service = Get-Service -Name $svc -ErrorAction SilentlyContinue

    if ($service) {

        Stop-Service -Name $svc -Force -ErrorAction SilentlyContinue

        Set-Service -Name $svc -StartupType Disabled -ErrorAction SilentlyContinue

        Write-Host "[+] Disabled: $svc" -ForegroundColor Yellow

    }

}

```

  

---

  

## 8. USER & GROUP MANAGEMENT

  

```cmd

REM Rename akun Administrator default:

wmic useraccount where name='Administrator' call rename name='Adm1n_Secure'

  

REM Disable akun Guest:

net user guest /active:no

  

REM Cek user yang ada:

net user

  

REM Cek member Administrators group:

net localgroup Administrators

```

  

```powershell

# PowerShell — cek user dengan password tidak kedaluwarsa:

Get-LocalUser | Where-Object {$_.PasswordExpires -eq $null -and $_.Enabled -eq $true} | Select Name

  

# Set password harus di-reset:

# Set-LocalUser -Name "username" -PasswordNeverExpires $false

```

  

---

  

## 9. WINDOWS DEFENDER

  

```powershell

# Pastikan Defender aktif:

Set-MpPreference -DisableRealtimeMonitoring $false

Set-MpPreference -DisableBehaviorMonitoring $false

Set-MpPreference -DisableOnAccessProtection $false

Set-MpPreference -DisableIOAVProtection $false

Set-MpPreference -DisableScriptScanning $false

Set-MpPreference -EnableNetworkProtection Enabled

Set-MpPreference -PUAProtection Enabled

  

# Update definisi:

Update-MpSignature

  

# Scan cepat:

Start-MpScan -ScanType QuickScan

```

  

---

  

## 10. VERIFIKASI AKHIR

  

```powershell

# Cek SMBv1:

Get-SmbServerConfiguration | Select EnableSMB1Protocol

  

# Cek Audit Policy:

auditpol /get /category:* | Select-String "Success and Failure"

  

# Cek Firewall:

Get-NetFirewallProfile | Select Name, Enabled, DefaultInboundAction

  

# Cek service yang berjalan (port terbuka):

netstat -ano | findstr LISTENING

  

# Cek WDigest:

Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" -Name UseLogonCredential

  

# Cek PowerShell logging:

Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"

```

  

---

  

## BONUS: Active Directory (AD) Hardening

  

```powershell

# Cari akun dengan Kerberoasting risk (SPN set):

Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName | Select Name, SamAccountName, ServicePrincipalName

  

# Cari akun dengan "Password Never Expires":

Get-ADUser -Filter {PasswordNeverExpires -eq $true -and Enabled -eq $true} | Select Name, SamAccountName

  

# Cari akun dengan "Password Not Required":

Get-ADUser -Filter {PasswordNotRequired -eq $true} | Select Name, SamAccountName

  

# Enable Fine-Grained Password Policy (PSO):

# (Buat dulu PSO, lalu apply ke grup)

New-ADFineGrainedPasswordPolicy -Name "AdminPSO" -Precedence 10 `

    -MinPasswordLength 16 -PasswordHistoryCount 10 `

    -ComplexityEnabled $true -MaxPasswordAge "45.00:00:00" `

    -LockoutThreshold 3 -LockoutDuration "00:30:00"

  

# Disable Unconstrained Delegation:

Get-ADComputer -Filter {TrustedForDelegation -eq $true} | Select Name

# Set-ADComputer -Identity "computername" -TrustedForDelegation $false

```

  

---

  

> 💡 **Tips Lomba:** Di Windows, selalu buka PowerShell/CMD sebagai **Run as Administrator**. Gunakan `Write-Host "[+]"` untuk konfirmasi visual setiap langkah berhasil.