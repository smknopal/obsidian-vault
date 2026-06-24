# W4 — Network Service Security (Windows)

#lks #cyber-security #windows #hardening #firewall #smb #rdp #network

  

> **Tingkat:** 🟢 Mudah | **Waktu:** 1 malam (2–3 jam)

> **Topik Kisi-kisi:** Infrastructure Hardening → Windows → Network Service Security

  

---

  

## 🧠 Analogi Buat yang Baru Mulai

  

Bayangkan Windows seperti **gedung kantor**. Ada banyak pintu (port jaringan) di gedung itu:

- Beberapa pintu **harus ada** (misal: pintu utama = HTTP port 80)

- Beberapa pintu **tidak perlu ada** dan justru berbahaya (misal: Telnet port 23 = pintu tanpa kunci!)

- Beberapa pintu **perlu dikunci lebih kuat** (misal: RDP = pintu yang perlu kode + sidik jari)

  

Tugasmu: **Tutup pintu yang tidak perlu, perkuat yang harus tetap ada.**

  

---

  

## BAGIAN 1 — Cek Port dan Service yang Terbuka

  

**Langkah pertama sebelum apapun:** Audit dulu apa yang sedang berjalan!

  

```powershell

# Cara 1: netstat — lihat semua port yang listening

netstat -ano

# Kolom: Proto | Local Address | Foreign Address | State | PID

  

# Cari tahu nama aplikasi dari PID:

tasklist | findstr "1234"   # Ganti 1234 dengan PID yang mau dicek

  

# Cara 2: PowerShell lebih lengkap (port + nama aplikasi sekaligus):

Get-NetTCPConnection -State Listen |

  Select-Object LocalAddress, LocalPort, OwningProcess |

  ForEach-Object {

    $proc = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue

    [PSCustomObject]@{

      Port        = $_.LocalPort

      Address     = $_.LocalAddress

      ProcessName = $proc.Name

      PID         = $_.OwningProcess

    }

  } | Sort-Object Port | Format-Table -AutoSize

  

# Cara 3: Lihat service yang berjalan sekarang

Get-Service | Where-Object { $_.Status -eq 'Running' } |

  Select-Object DisplayName, Name, StartType |

  Sort-Object DisplayName

```

  

### Port Berbahaya yang Sering Muncul:

  

| Port | Service | Tindakan |

|------|---------|----------|

| 23 | Telnet | ❌ Matikan segera! (unencrypted) |

| 21 | FTP | ❌ Matikan jika tidak perlu (unencrypted) |

| 139/445 | SMB | ⚠️ Disable SMBv1, aktifkan signing |

| 3389 | RDP | ⚠️ Aktifkan NLA, batasi akses dengan firewall |

| 5985/5986 | WinRM (HTTP/HTTPS) | ⚠️ Matikan jika tidak pakai, atau amankan |

| 3306 | MySQL | ⚠️ Bind ke localhost saja jika tidak perlu akses remote |

| 5900 | VNC | ❌ Matikan atau ganti dengan RDP yang aman |

| 53 | DNS | ⚠️ Hanya buka di server DNS, tutup di workstation |

  

---

  

## BAGIAN 2 — Windows Firewall (wf.msc)

  

**Buka:**

```

Win + R → wf.msc

ATAU: Control Panel → System and Security → Windows Defender Firewall → Advanced Settings

```

  

### Aktifkan Firewall untuk Semua Profile:

  

```powershell

# Aktifkan firewall untuk semua profile (Domain, Public, Private)

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

  

# Set default: block semua inbound yang tidak ada rule-nya

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultInboundAction Block

  

# Outbound: allow semua (atau block untuk lebih ketat)

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultOutboundAction Allow

  

# Aktifkan logging firewall (WAJIB untuk forensik!)

Set-NetFirewallProfile -Profile Domain,Public,Private `

  -LogAllowed True `

  -LogBlocked True `

  -LogFileName "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `

  -LogMaxSizeKilobytes 32767

  

# Cek status firewall:

Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction

```

  

### Buat Rule Firewall:

  

```powershell

# Blokir port 23 (Telnet) - inbound

New-NetFirewallRule -DisplayName "Block Telnet Inbound" `

  -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block -Enabled True

  

# Blokir port 21 (FTP) - inbound

New-NetFirewallRule -DisplayName "Block FTP Inbound" `

  -Direction Inbound -Protocol TCP -LocalPort 21 -Action Block -Enabled True

  

# Allow RDP hanya dari subnet tertentu (bukan dari internet):

New-NetFirewallRule -DisplayName "Allow RDP from LAN Only" `

  -Direction Inbound -Protocol TCP -LocalPort 3389 `

  -RemoteAddress "192.168.1.0/24" -Action Allow -Enabled True

  

# Blokir ICMP ping dari luar (opsional, tapi kadang diminta):

New-NetFirewallRule -DisplayName "Block ICMP Echo Inbound" `

  -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -Action Block -Enabled True

  

# Hapus rule yang tidak perlu:

Remove-NetFirewallRule -DisplayName "Nama Rule yang Ingin Dihapus"

  

# Lihat semua rule yang aktif:

Get-NetFirewallRule | Where-Object { $_.Enabled -eq 'True' } |

  Select-Object DisplayName, Direction, Action | Sort-Object Direction | Format-Table -AutoSize

```

  

---

  

## BAGIAN 3 — Disable SMBv1 (WAJIB! Ini Sering Diuji!)

  

**Kenapa penting?**

SMBv1 adalah protokol lawas yang digunakan malware WannaCry dan NotPetya untuk menyebar. Di tahun 2017, WannaCry menginfeksi ratusan ribu komputer di seluruh dunia hanya karena SMBv1 aktif. **HARUS DIMATIKAN!**

  

```powershell

# LANGKAH 1: Cek apakah SMBv1 aktif

Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol

# Jika True → harus dimatikan!

  

# LANGKAH 2: Disable SMBv1 di server

Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

  

# LANGKAH 3: Disable SMBv1 di client

Set-SmbClientConfiguration -EnableSMB1Protocol $false -Force

  

# LANGKAH 4: Uninstall feature SMBv1 (untuk Windows Server — lebih permanen):

Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# ATAU di Windows Server:

# Remove-WindowsFeature -Name FS-SMB1

  

# LANGKAH 5: Verifikasi SMBv1 sudah mati

Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol

# Harus: False

  

# LANGKAH 6: Pastikan SMBv2 dan v3 masih aktif (JANGAN disable ini!)

Get-SmbServerConfiguration | Select-Object EnableSMB2Protocol

# Harus: True

  

# Restart mungkin diperlukan setelah uninstall feature

```

  

> ⚠️ **Jangan disable SMBv2!** Itu akan memutus file sharing normal.

> SMBv2 dan v3 aman dan perlu tetap aktif.

  

### Aktifkan SMB Signing (Cegah MITM Attack):

  

```powershell

# Server harus menandatangani — REQUIRED (bukan hanya "enabled")

Set-SmbServerConfiguration -RequireSecuritySignature $true -Force

  

# Client juga harus

Set-SmbClientConfiguration -RequireSecuritySignature $true -Force

  

# Verifikasi:

Write-Host "Server Signing:"

Get-SmbServerConfiguration | Select-Object RequireSecuritySignature, EnableSecuritySignature

Write-Host "Client Signing:"

Get-SmbClientConfiguration | Select-Object RequireSecuritySignature, EnableSecuritySignature

# RequireSecuritySignature harus: True

```

  

---

  

## BAGIAN 4 — RDP Hardening

  

RDP (Remote Desktop Protocol) adalah target favorit attacker karena ada di hampir semua Windows dan sering lupa diamankan.

  

```powershell

# CEK status RDP saat ini:

$rdpStatus = Get-ItemProperty `

  -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" `

  -Name "fDenyTSConnections" -ErrorAction SilentlyContinue

if ($rdpStatus.fDenyTSConnections -eq 0) {

  Write-Host "RDP: AKTIF" -ForegroundColor Yellow

} else {

  Write-Host "RDP: Tidak aktif" -ForegroundColor Green

}

  

# Nonaktifkan RDP (jika tidak diperlukan — ini lebih aman!):

Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" `

  -Name "fDenyTSConnections" -Value 1

Write-Host "RDP sudah dinonaktifkan"

  

# Aktifkan RDP (jika diperlukan):

Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" `

  -Name "fDenyTSConnections" -Value 0

# Nilai: 0 = RDP enabled, 1 = RDP disabled

```

  

### WAJIB: Aktifkan Network Level Authentication (NLA)

  

NLA = user harus autentikasi SEBELUM sesi RDP terbuka. Tanpa NLA, attacker bisa membuka layar login RDP tanpa autentikasi → rentan DoS dan brute-force lebih mudah.

  

```powershell

# Aktifkan NLA

Set-ItemProperty `

  -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `

  -Name "UserAuthentication" -Value 1

# 1 = NLA enabled, 0 = NLA disabled

  

# Verifikasi:

$nla = Get-ItemProperty `

  -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `

  -Name "UserAuthentication"

Write-Host "NLA UserAuthentication: $($nla.UserAuthentication)"

# Harus: 1

  

# Set minimum enkripsi RDP ke High

Set-ItemProperty `

  -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `

  -Name "MinEncryptionLevel" -Value 3

# 1=Low, 2=Client Compatible, 3=High, 4=FIPS Compliant

```

  

---

  

## BAGIAN 5 — Disable Service yang Tidak Diperlukan

  

```powershell

# Service yang WAJIB dimatikan di environment hardened:

$services_to_disable = @(

  @{ Name="TlntSvr";        DisplayName="Telnet" },

  @{ Name="FTPSVC";         DisplayName="FTP" },

  @{ Name="WinRM";          DisplayName="Windows Remote Management" },

  @{ Name="RemoteRegistry"; DisplayName="Remote Registry" },

  @{ Name="Browser";        DisplayName="Computer Browser" },

  @{ Name="Spooler";        DisplayName="Print Spooler (jika bukan print server)" }

)

  

foreach ($svc in $services_to_disable) {

  $service = Get-Service -Name $svc.Name -ErrorAction SilentlyContinue

  if ($service) {

    if ($service.Status -eq 'Running') {

      Stop-Service -Name $svc.Name -Force -ErrorAction SilentlyContinue

    }

    Set-Service -Name $svc.Name -StartupType Disabled -ErrorAction SilentlyContinue

    Write-Host "✅ Disabled: $($svc.DisplayName) ($($svc.Name))"

  } else {

    Write-Host "⏭️ Not found (skip): $($svc.DisplayName)"

  }

}

  

# Verifikasi service sudah disabled:

foreach ($svc in $services_to_disable) {

  $service = Get-Service -Name $svc.Name -ErrorAction SilentlyContinue

  if ($service) {

    Write-Host "$($svc.Name): Status=$($service.Status), StartType=$($service.StartType)"

  }

}

```

  

> ⚠️ **Print Spooler:** Matikan kecuali ini adalah print server.

> Rentan terhadap PrintNightmare (CVE-2021-34527) exploit!

  

### Service yang JANGAN Dimatikan:

```

Workstation          ← Dibutuhkan untuk koneksi jaringan

Server               ← Dibutuhkan untuk file sharing (SMBv2/v3)

DNS Client           ← Dibutuhkan untuk resolusi nama

DHCP Client          ← Dibutuhkan untuk dapat IP otomatis

Windows Update (wuauserv) ← Dibutuhkan untuk patch security

Windows Defender     ← Antivirus bawaan

Netlogon             ← Dibutuhkan untuk autentikasi domain

CryptSvc             ← Dibutuhkan untuk kriptografi

```

  

---

  

## BAGIAN 6 — Disable LLMNR dan NetBIOS

  

**Kenapa berbahaya?**

Kedua protokol ini bisa dieksploitasi dengan tool **Responder** untuk mencuri hash password. Caranya: ketika komputer mencari nama yang tidak ada di DNS, ia akan "broadcast" bertanya ke jaringan. Responder menjawab broadcast ini dan "menipu" komputer untuk mengirimkan hash NTLM.

  

```powershell

# === DISABLE LLMNR ===

$dnsClientPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient"

if (-not (Test-Path $dnsClientPath)) {

  New-Item -Path $dnsClientPath -Force | Out-Null

}

Set-ItemProperty -Path $dnsClientPath -Name "EnableMulticast" -Value 0

  

# Verifikasi:

$llmnr = Get-ItemProperty -Path $dnsClientPath -Name "EnableMulticast" -ErrorAction SilentlyContinue

Write-Host "LLMNR EnableMulticast: $($llmnr.EnableMulticast)"

# Harus: 0

  

# === DISABLE NetBIOS ===

$adapters = Get-WmiObject Win32_NetworkAdapterConfiguration |

  Where-Object { $_.IPEnabled -eq $true }

foreach ($adapter in $adapters) {

  $result = $adapter.SetTcpipNetbios(2)   # 2 = Disable NetBIOS over TCP/IP

  if ($result.ReturnValue -eq 0) {

    Write-Host "✅ NetBIOS disabled on: $($adapter.Description)"

  } else {

    Write-Host "❌ GAGAL disable NetBIOS pada: $($adapter.Description)"

  }

}

```

  

---

  

## BAGIAN 7 — WinRM Hardening

  

WinRM (Windows Remote Management) memungkinkan remote management via PowerShell. Kalau tidak diperlukan, matikan. Kalau diperlukan, amankan.

  

```powershell

# Cek status WinRM:

Get-Service WinRM | Select-Object Name, Status, StartType

  

# Kalau tidak perlu — matikan:

Stop-Service WinRM -Force

Set-Service WinRM -StartupType Disabled

Write-Host "WinRM dinonaktifkan"

  

# Kalau perlu tapi ingin diamankan:

# 1. Aktifkan hanya HTTPS (bukan HTTP):

winrm set winrm/config/listener?Address=*+Transport=HTTP '@{Enabled="false"}'

  

# 2. Batasi siapa yang bisa terkoneksi (hanya admin):

Set-PSSessionConfiguration -Name "Microsoft.PowerShell" `

  -ShowSecurityDescriptorUI

  

# 3. Set idle timeout (disconnect setelah 15 menit idle):

Set-PSSessionConfiguration -Name "Microsoft.PowerShell" `

  -IdleTimeoutSec 900

```

  

---

  

## BAGIAN 8 — Cek Koneksi Aktif yang Mencurigakan

  

```powershell

# Lihat semua koneksi aktif (bukan hanya listening):

Get-NetTCPConnection -State Established |

  ForEach-Object {

    $proc = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue

    [PSCustomObject]@{

      LocalPort   = $_.LocalPort

      RemoteAddr  = $_.RemoteAddress

      RemotePort  = $_.RemotePort

      ProcessName = $proc.Name

      PID         = $_.OwningProcess

    }

  } | Format-Table -AutoSize

  

# Cek koneksi ke IP asing (bukan localhost atau LAN):

Get-NetTCPConnection -State Established |

  Where-Object {

    $_.RemoteAddress -ne "127.0.0.1" -and

    $_.RemoteAddress -notlike "192.168.*" -and

    $_.RemoteAddress -notlike "10.*" -and

    $_.RemoteAddress -notlike "172.16.*"

  } |

  ForEach-Object {

    $proc = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue

    [PSCustomObject]@{

      RemoteAddress = $_.RemoteAddress

      RemotePort    = $_.RemotePort

      ProcessName   = $proc.Name

      PID           = $_.OwningProcess

    }

  }

```

  

---

  

## ✅ Checklist Network Service Security

  

**Audit Awal**

- [ ] `Get-NetTCPConnection -State Listen` sudah diperiksa — tidak ada port aneh yang listening

- [ ] Semua service yang berjalan sudah diaudit

  

**Windows Firewall**

- [ ] Firewall aktif: Domain, Public, Private semua enabled

- [ ] Default inbound action = Block

- [ ] Firewall logging aktif (log ke pfirewall.log)

- [ ] Tidak ada rule yang terlalu permissive (misal: Allow all inbound)

  

**SMB**

- [ ] SMBv1 dinonaktifkan (`EnableSMB1Protocol = False`)

- [ ] SMBv1 feature sudah di-uninstall (Windows Server)

- [ ] SMB signing diaktifkan dan required (client dan server)

- [ ] SMBv2 masih aktif (jangan dimatikan!)

  

**RDP**

- [ ] RDP dinonaktifkan jika tidak diperlukan

- [ ] Jika RDP aktif: NLA diaktifkan (UserAuthentication = 1)

- [ ] Jika RDP aktif: hanya bisa diakses dari IP/subnet tertentu (via firewall)

- [ ] RDP encryption level = High (MinEncryptionLevel = 3)

  

**Service**

- [ ] Telnet service dinonaktifkan

- [ ] Remote Registry dinonaktifkan

- [ ] Print Spooler dinonaktifkan (jika bukan print server)

- [ ] WinRM dinonaktifkan (jika tidak dipakai)

- [ ] Computer Browser dinonaktifkan

  

**Protokol Berbahaya**

- [ ] LLMNR dinonaktifkan (EnableMulticast = 0)

- [ ] NetBIOS dinonaktifkan di semua adapter

  

---

  

## 🔗 Navigasi

  

← [[W3_GPO_Policy]]

→ [[W5_Windows_Defender]]