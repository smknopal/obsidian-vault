# 🔥 CHEATSHEET WINDOWS — Bawa Ke Lomba!

#lks #cheatsheet #windows #hardening #quick-reference

  

> **PRINT atau simpan offline. Buka ini saat lomba dimulai.**

  

---

  

## ⚡ BUKA TOOLS INI PERTAMA (Win+R → ketik → Enter)

  

```

secpol.msc    → Local Security Policy (password + lockout + security options)

lusrmgr.msc   → Local Users and Groups (kelola user)

gpedit.msc    → Group Policy Editor

gpmc.msc      → Group Policy Management (butuh AD)

eventvwr.msc  → Event Viewer (baca log)

wf.msc        → Windows Firewall Advanced

services.msc  → Services Manager

dsa.msc       → AD Users and Computers (butuh AD)

tpm.msc       → TPM status (untuk BitLocker)

```

  

---

  

## 🔍 RECON — JALANKAN PERTAMA KALI

  

```powershell

net user                                           # Daftar semua user

net localgroup administrators                      # Cek siapa di admin group

net accounts                                       # Cek password + lockout policy

Get-LocalUser | Select Name, Enabled, PasswordNeverExpires  # Detail user

Get-SmbServerConfiguration | Select EnableSMB1Protocol      # Cek SMBv1

Get-MpComputerStatus | Select RealTimeProtectionEnabled      # Cek Defender

netstat -ano                                       # Port yang terbuka

auditpol /get /category:*                          # Cek audit policy

```

  

---

  

## 🔐 PASSWORD & LOCKOUT (net accounts)

  

```cmd

net accounts /minpwlen:12        # Minimal 12 karakter

net accounts /maxpwage:90        # Expired 90 hari

net accounts /minpwage:1         # Minimal 1 hari sebelum ganti

net accounts /uniquepw:5         # Simpan 5 history

net accounts /lockoutthreshold:5 # Kunci setelah 5x gagal

net accounts /lockoutduration:15 # Kunci 15 menit

net accounts /lockoutwindow:15   # Reset counter 15 menit

net accounts                     # Verifikasi semua

```

  

> ⚠️ **Password complexity HARUS diset via secpol.msc!**

> Tidak bisa via net accounts.

  

---

  

## 👤 USER MANAGEMENT

  

```cmd

net user guest /active:no            # Nonaktifkan Guest (WAJIB!)

net user administrator /active:no    # Nonaktifkan Admin bawaan

net user NAMA "Password123!" /add    # Buat user baru

net localgroup administrators NAMA /add    # Tambah ke admin

net localgroup administrators NAMA /delete # Hapus dari admin

net user NAMA /delete                # Hapus user

```

  

```powershell

# Perbaiki semua user yang PasswordNeverExpires:

Get-LocalUser | Where {$_.PasswordNeverExpires -eq $true} |

  Set-LocalUser -PasswordNeverExpires $false

```

  

---

  

## 🛡️ SECURITY OPTIONS (secpol.msc → Local Policies → Security Options)

  

```

✅ Do not display last user name          → ENABLED

✅ Message text for users...              → Isi pesan peringatan

✅ Machine inactivity limit               → 900 (15 menit)

✅ Anonymous enumeration of SAM accounts  → ENABLED (no enumeration)

✅ Anonymous enumeration SAM and shares   → ENABLED

✅ LAN Manager auth level                 → NTLMv2 only, Refuse LM & NTLM

✅ Shutdown without logon                 → DISABLED

✅ Guest account status                   → DISABLED

✅ UAC: Run administrators in AAM         → ENABLED

```

  

---

  

## 📡 DISABLE SMBv1 (WAJIB!)

  

```powershell

Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

Set-SmbClientConfiguration -EnableSMB1Protocol $false -Force

Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

Set-SmbServerConfiguration -RequireSecuritySignature $true -Force

Set-SmbClientConfiguration -RequireSecuritySignature $true -Force

  

# Verifikasi:

Get-SmbServerConfiguration | Select EnableSMB1Protocol  # Harus: False

```

  

---

  

## 🧱 FIREWALL

  

```powershell

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultInboundAction Block

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultOutboundAction Allow

  

# Block port berbahaya:

New-NetFirewallRule -DisplayName "Block Telnet" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block

New-NetFirewallRule -DisplayName "Block FTP" -Direction Inbound -Protocol TCP -LocalPort 21 -Action Block

  

# Verifikasi:

Get-NetFirewallProfile | Select Name, Enabled, DefaultInboundAction

```

  

---

  

## 🛑 MATIKAN SERVICE BERBAHAYA

  

```powershell

$svcs = @("TlntSvr","RemoteRegistry","SNMP","Spooler","WinRM")

foreach ($s in $svcs) {

  Stop-Service $s -Force -EA SilentlyContinue

  Set-Service $s -StartupType Disabled -EA SilentlyContinue

  Write-Host "Done: $s"

}

```

  

---

  

## 🚫 DISABLE LLMNR & NetBIOS

  

```powershell

# LLMNR:

New-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Force | Out-Null

Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -Value 0

  

# NetBIOS:

(Get-WmiObject Win32_NetworkAdapterConfiguration | Where {$_.IPEnabled}) |

  ForEach { $_.SetTcpipNetbios(2) }

```

  

---

  

## 🛡️ WINDOWS DEFENDER

  

```powershell

Update-MpSignature                                           # Update dulu!

Set-MpPreference -DisableRealtimeMonitoring $false

Set-MpPreference -DisableBehaviorMonitoring $false

Set-MpPreference -MAPSReporting Advanced

Set-MpPreference -EnableControlledFolderAccess Enabled

Get-MpPreference | Select ExclusionPath                     # Cek exclusion mencurigakan!

Get-MpComputerStatus | Select *Enabled*                     # Verifikasi semua True

```

  

---

  

## 📊 AUDIT POLICY (Copy-paste semua sekaligus!)

  

```powershell

auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable

auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable

auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable

auditpol /set /subcategory:"Logon" /success:enable /failure:enable

auditpol /set /subcategory:"Logoff" /success:enable

auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable

auditpol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable

auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable

auditpol /set /subcategory:"Process Creation" /success:enable

auditpol /set /subcategory:"System Integrity" /success:enable /failure:enable

auditpol /get /category:*    # Verifikasi

```

  

---

  

## 📋 PERBESAR LOG & POWERSHELL LOGGING

  

```powershell

# Perbesar ukuran log:

wevtutil sl Security /ms:1073741824   # 1 GB

wevtutil sl System /ms:524288000      # 500 MB

  

# PowerShell Script Block Logging:

$p = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"

New-Item $p -Force | Out-Null

Set-ItemProperty $p -Name "EnableScriptBlockLogging" -Value 1

  

# PowerShell Module Logging:

$p = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"

New-Item $p -Force | Out-Null

Set-ItemProperty $p -Name "EnableModuleLogging" -Value 1

```

  

---

  

## 🔑 WDIGEST — CEGAH PASSWORD PLAINTEXT

  

```powershell

Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" `

  -Name "UseLogonCredential" -Value 0

# Verifikasi:

Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" `

  -Name "UseLogonCredential"

# Harus: 0 ✅

```

  

---

  

## 📖 BACA LOG — Event ID Penting

  

```powershell

# Login gagal (brute force?):

Get-WinEvent -LogName Security | Where {$_.Id -eq 4625} | Select TimeCreated, Message -First 10

  

# User baru dibuat:

Get-WinEvent -LogName Security | Where {$_.Id -eq 4720} | Select TimeCreated, Message

  

# Ditambahkan ke admin:

Get-WinEvent -LogName Security | Where {$_.Id -eq 4732} | Select TimeCreated, Message

  

# Log dihapus (BAHAYA!):

Get-WinEvent -LogName Security | Where {$_.Id -eq 1102} | Select TimeCreated, Message

  

# Service baru (malware?):

Get-WinEvent -LogName System | Where {$_.Id -eq 7045} | Select TimeCreated, Message

```

  

---

  

## ❓ Q&A CEPAT WINDOWS

  

| Pertanyaan | Jawaban |

|-----------|---------|

| Cek password policy? | `net accounts` |

| Nonaktifkan Guest? | `net user guest /active:no` |

| Disable SMBv1? | `Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force` |

| Cek siapa di admin? | `net localgroup administrators` |

| Aktifkan firewall? | `Set-NetFirewallProfile -Profile * -Enabled True` |

| Baca login gagal? | `Get-WinEvent -LogName Security | Where {$_.Id -eq 4625}` |

| Update Defender? | `Update-MpSignature` |

| Aktifkan audit? | `auditpol /set /subcategory:"Logon" /success:enable /failure:enable` |

| Apply GPO? | `gpupdate /force` |

| Cek GPO berlaku? | `gpresult /r` |

| Disable WDigest? | Set registry `UseLogonCredential = 0` |

| Disable LLMNR? | Set registry `EnableMulticast = 0` |

| Disable NetBIOS? | `SetTcpipNetbios(2)` via WMI |

| Event ID login gagal? | `4625` |

| Event ID user dibuat? | `4720` |

| Event ID ditambah admin? | `4732` |

| Event ID log dihapus? | `1102` |

| Event ID service baru? | `7045` |