# 🪟 Windows Hardening — Master Index (LKS Cyber Security 2026)

#lks #cyber-security #windows #hardening #index

  

---

  

## 🧠 Konteks Lomba

  

> **Hari ke-1 (H1)** — durasi 3 jam, bobot **25%** dari total nilai.

> Kamu diberikan OS Windows (Server/Client) dan diminta konfigurasi sesuai objektif soal.

> Penilaian: hasil konfigurasi berhasil atau tidak (measurement = biner 0/1).

  

**ARTINYA:** Kamu HARUS bisa langsung eksekusi, tidak ada waktu berpikir panjang.

Hafal perintah dan lokasi settingnya! Latihan berkali-kali sampai jadi otomatis.

  

---

  

## 📊 Daftar Modul (Urutan Belajar)

  

| No | File | Topik | Tingkat | Waktu | Status |

|----|------|-------|---------|-------|--------|

| W1 | `W1_PAM_Local_Security.md` | PAM — Local Security Policy | 🟢 Mudah | 1 malam | - |

| W2 | `W2_Active_Directory.md` | Active Directory Basic Security | 🔴 Susah | 3 malam | - |

| W3 | `W3_GPO_Policy.md` | GPO — Local & AD Policy | 🟡 Sedang | 2 malam | - |

| W4 | `W4_Network_Service_Security.md` | Network Service Security | 🟢 Mudah | 1 malam | - |

| W5 | `W5_Windows_Defender.md` | AV — Windows Defender | 🟢 Mudah | 0.5 malam | - |

| W6 | `W6_Logging_Audit.md` | Logging & Audit Policy | 🟡 Sedang | 1 malam | - |

| W7 | `W7_LAPS.md` | LAPS — Local Admin Password Solution | 🟡 Sedang | 1 malam | - |

| W8 | `W8_Credential_Guard.md` | Credential Guard & LSA Protection | 🔴 Susah | 1 malam | - |

| W9 | `W9_Windows_Update.md` | Windows Update & Patch Management | 🟢 Mudah | 0.5 malam | - |

| W10 | `W10_BitLocker.md` | BitLocker — Disk Encryption | 🟡 Sedang | 1 malam | - |

  

> **Total waktu belajar: ~25 jam**

> Dengan 2-3 jam/malam selama 10 hari = bisa terkejar, tapi harus fokus!

  

---

  

## 🗓️ Jadwal Belajar 10 Hari (Rekomendasi)

  

| Hari | Materi | Target |

|------|--------|--------|

| Hari 1 | W1 — PAM Local Security Policy | Hafal secpol.msc + lusrmgr.msc + semua perintah net |

| Hari 2 | W4 + W5 — Network + AV | Hafal PowerShell Defender + wf.msc + SMB hardening |

| Hari 3 | W6 — Logging & Audit | Hafal auditpol + Event IDs penting + PowerShell logging |

| Hari 4 | W3 — GPO Local | Hafal gpedit.msc semua kategori security options |

| Hari 5 | W2 — AD basics | Install AD DS + create users/OUs + hardening akun |

| Hari 6 | W2 lanjutan + W3 domain | gpmc.msc + PSO + AdminSDHolder |

| Hari 7 | W7 — LAPS | Install LAPS + konfigurasi + verifikasi |

| Hari 8 | W8 — Credential Guard | LSA Protection + Credential Guard via GPO |

| Hari 9 | W9 + W10 — Update + BitLocker | WSUS/Windows Update policy + BitLocker setup |

| Hari 10 | Review + Simulasi Soal | Praktek ulang dari awal seperti kondisi lomba |

  

---

  

## 🔑 Quick Command Cheatsheet (HAFAL INI!)

  

```powershell

# === BUKA TOOLS PENTING ===

secpol.msc          # Local Security Policy

lusrmgr.msc         # Local Users and Groups

gpedit.msc          # Local Group Policy Editor

gpmc.msc            # Group Policy Management (butuh AD)

eventvwr.msc        # Event Viewer

wf.msc              # Windows Firewall Advanced

services.msc        # Services manager

dsa.msc             # Active Directory Users and Computers

adsiedit.msc        # ADSI Edit (advanced AD editing)

  

# === GPO ===

gpupdate /force             # Paksa apply GPO sekarang

gpresult /r                 # Lihat GPO yang berlaku

gpresult /h C:\report.html  # Buat laporan GPO HTML

  

# === ACTIVE DIRECTORY ===

Get-ADUser -Filter *

Get-ADGroupMember "Domain Admins"

Disable-ADAccount -Identity "username"

Unlock-ADAccount -Identity "username"

Get-ADUser -Filter {PasswordNeverExpires -eq $true}

  

# === WINDOWS DEFENDER ===

Get-MpComputerStatus

Set-MpPreference -DisableRealtimeMonitoring $false

Update-MpSignature

  

# === NETWORK ===

netstat -ano

Get-SmbServerConfiguration | Select EnableSMB1Protocol

Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

  

# === AUDIT POLICY ===

auditpol /get /category:*

auditpol /set /subcategory:"Logon" /success:enable /failure:enable

  

# === USER MANAGEMENT ===

net user

net user administrator /active:no

net localgroup administrators

  

# === WINDOWS UPDATE ===

Get-WindowsUpdateLog

Install-Module PSWindowsUpdate -Force

Get-WindowsUpdate

Install-WindowsUpdate -AcceptAll -AutoReboot

```

  

---

  

## 💡 Tips Lomba Windows Hardening

  

1. **Baca soal dulu!** Identifikasi: apakah soalnya AD atau standalone (local)?

2. **Jika ada AD** → semua setting harus via GPO domain, bukan local

3. **Selalu jalankan `gpupdate /force`** setelah edit GPO

4. **Jika ada error** → cek Event Viewer dulu sebelum panik

5. **Screenshot/note setiap perubahan** — juri mungkin minta dokumentasi

6. **Backup konfigurasi awal** sebelum mengubah apapun

7. **Cek dulu status sebelum konfigurasi** — jangan langsung eksekusi buta

8. **Verifikasi setelah setiap perubahan** — pastikan setting benar-benar berlaku

  

---

  

## 🗺️ Peta Konsep Windows Hardening

  

```

Windows Hardening (LKS 2026)

├── Akun & Password

│   ├── W1. PAM Local Security Policy    ← Mulai sini!

│   ├── W2. Active Directory Security    ← Terberat

│   └── W7. LAPS                         ← Admin password otomatis

│

├── Policy & Kontrol

│   ├── W3. GPO Local & Domain           ← Sangat sering diuji

│   └── W8. Credential Guard             ← Cegah credential theft

│

├── Jaringan & Layanan

│   └── W4. Network Service Security     ← Firewall + SMB + RDP

│

├── Perlindungan Aktif

│   ├── W5. Windows Defender (AV)        ← Paling cepat dipelajari

│   └── W10. BitLocker                   ← Enkripsi disk

│

├── Monitoring & Log

│   └── W6. Logging & Audit Policy       ← Deteksi serangan

│

└── Patch & Update

    └── W9. Windows Update               ← Tutup celah keamanan

```

  

---

  

## 🔗 Navigasi

  

→ [[W1_PAM_Local_Security]]

→ [[W2_Active_Directory]]

→ [[W3_GPO_Policy]]

→ [[W4_Network_Service_Security]]

→ [[W5_Windows_Defender]]

→ [[W6_Logging_Audit]]

→ [[W7_LAPS]]

→ [[W8_Credential_Guard]]

→ [[W9_Windows_Update]]

→ [[W10_BitLocker]]