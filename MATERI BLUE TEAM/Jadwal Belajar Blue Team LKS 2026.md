# 📅 Jadwal Belajar Blue Team LKS 2026

### Versi Terstruktur — Waktu Spesifik · Rotasi Materi · 27 Mei – 27 Juli 2026

  

---

  

## 📌 Prinsip Jadwal Ini

  

| Parameter | Detail |

|---|---|

| Mulai | Rabu, 27 Mei 2026 |

| Selesai materi baru | Minggu, 26 Juli 2026 |

| Sisa waktu sebelum lomba | Agustus → full murojaah |

| Waktu belajar harian | **19:30–22:00** (2,5 jam aktif) |

| Waktu praktik Sabtu | **09:00–12:00** (3 jam CTF/lab) |

| Minggu | Istirahat / review ringan 1 jam saja |

| Pendekatan | **Rotasi** — setiap hari topik berbeda, satu topik besar dibagi 2–3 sesi tersebar |

  

> **Kenapa rotasi?** Otak menyimpan memori lebih kuat ketika ada jeda antar sesi (spaced repetition).

> Hardening Linux tidak harus selesai hari ini — yang penting tiap sesi ada kemajuan konkret.

  

---

  

## 🗓️ MINGGU 1 — 27 Mei – 1 Juni 2026

**Tema: Fondasi + File Carving + Awal Stego**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Rabu, 27 Mei** | 19:30–22:00 | Mindset Defender · CIA Triad · MITRE ATT&CK overview · Siklus IR (NIST vs SANS) | Bisa jelaskan 5 fase IR + mapping ke soal CTF |

| **Kamis, 28 Mei** | 19:30–22:00 | Linux CLI intensif part 1: `grep`, `awk`, `find`, `pipe`, `redirect` | Bisa filter log dengan grep+awk tanpa googling |

| **Jumat, 29 Mei** | 19:30–22:00 | Python CTF: `bytes`, `struct`, `base64`, `hashlib` · File formats: hex, magic bytes, `xxd` | Bisa decode file aneh pakai Python 10 baris |

| **Sabtu, 30 Mei** | 09:00–12:00 | File carving praktik: `binwalk`, `foremost`, `photorec` — pakai file embedded nyata dari PicoCTF | Extract minimal 2 file tersembunyi dan dokumentasikan |

| **Minggu, 31 Mei** | 20:00–21:00 | Review singkat: ulangi 3 command Linux yang paling sering salah kemarin | — |

  

---

  

## 🗓️ MINGGU 2 — 2–7 Juni 2026

**Tema: Stego + Awal Network + Linux CLI lanjut**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 2 Jun** | 19:30–22:00 | Image stego: LSB theory · `steghide` · `zsteg` · `stegsolve` | Solve 1 soal stego PicoCTF end-to-end |

| **Selasa, 3 Jun** | 19:30–22:00 | Linux CLI intensif part 2: `sort`, `uniq`, `wc`, `cut`, `sed`, `tr`, `tee` | Bisa proses CSV log dengan pipeline 1 baris |

| **Rabu, 4 Jun** | 19:30–22:00 | Audio stego: Sonic Visualizer (spectrogram) · `stegsnow` · DTMF decoder | Deteksi hidden message di file .wav |

| **Kamis, 5 Jun** | 19:30–22:00 | Network basics: OSI, TCP 3-way handshake, protokol wajib (HTTP, FTP, DNS, SMB, ICMP) | Bisa gambar flow TCP handshake dari memory |

| **Jumat, 6 Jun** | 19:30–22:00 | Document forensics: `olevba`, `oledump`, `peepdf` — analisis macro Word/PDF berbahaya | Ekstrak macro dari .doc dan identifikasi payload |

| **Sabtu, 7 Jun** | 09:00–12:00 | CTF mix: 1 soal file carving + 1 soal stego + 1 soal document forensics | Selesaikan 3 soal + tulis mini-writeup tiap soal |

| **Minggu, 8 Jun** | 20:00–21:00 | Flashcard review: magic bytes penting (PNG, ZIP, PDF, ELF, PE) + OSI layer | — |

  

---

  

## 🗓️ MINGGU 3 — 9–14 Juni 2026

**Tema: Wireshark + tshark + Awal Log Analysis**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 9 Jun** | 19:30–22:00 | Wireshark dasar: filter syntax `ip.addr`, `tcp.port`, `http`, `dns`, `frame contains` | Bisa isolasi traffic HTTP dari pcap 10 menit |

| **Selasa, 10 Jun** | 19:30–22:00 | Wireshark advanced: follow TCP stream, export HTTP objects, credential di cleartext (FTP/HTTP) | Extract username+password dari pcap nyata |

| **Rabu, 11 Jun** | 19:30–22:00 | Linux log analysis part 1: `auth.log`, `syslog`, Apache access log — pola brute force via `grep/awk` | Tulis 1-liner grep untuk deteksi 5+ failed login |

| **Kamis, 12 Jun** | 19:30–22:00 | Deteksi anomali di Wireshark: port scan (banyak SYN), DNS tunneling, C2 beacon pattern | Identifikasi 3 jenis anomali dari 1 file pcap |

| **Jumat, 13 Jun** | 19:30–22:00 | `tshark` CLI: extract fields via terminal · Python scripting untuk parse pcap (`scapy`/`pyshark`) | Bisa extract semua DNS query dari pcap via CLI |

| **Sabtu, 14 Jun** | 09:00–12:00 | CTF: HTB Academy Network Traffic Analysis + 1 soal Wireshark dari PicoCTF | Selesaikan modul + writeup teknis |

| **Minggu, 15 Jun** | 20:00–21:00 | Review: hafal filter Wireshark paling penting + pola anomali | — |

  

---

  

## 🗓️ MINGGU 4 — 16–21 Juni 2026

**Tema: Windows Event IDs + Splunk SPL + Awal Hardening Linux**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 16 Jun** | 19:30–22:00 | Windows Event IDs: 4624/4625/4648/4688/4720/4732/7045/1102 + Logon Types | Hafal 10 Event ID kritis + artinya tanpa lihat catatan |

| **Selasa, 17 Jun** | 19:30–22:00 | Splunk SPL part 1: `search`, `stats`, `timechart`, `table`, `where`, `dedup` | Bisa query event login gagal per IP dalam 1 jam |

| **Rabu, 18 Jun** | 19:30–22:00 | Linux hardening part 1: PAM (`pam_pwquality`, `pam_faillock`), disable services berbahaya | Konfigurasi password policy & account lockout di VM |

| **Kamis, 19 Jun** | 19:30–22:00 | Splunk SPL part 2: `eval`, `rex`, `lookup`, `alert logic` · Wazuh install + dashboard overview | Buat 1 alert Splunk untuk deteksi brute force |

| **Jumat, 20 Jun** | 19:30–22:00 | Linux log analysis part 2: `/var/log/secure`, `journalctl`, `auditd` — lateral movement pattern | Tulis query auditd untuk deteksi `sudo` abuse |

| **Sabtu, 21 Jun** | 09:00–12:00 | Threat hunting: Splunk Boss of the SOC (BOTS) dataset — cari 1 attack chain lengkap | Dokumentasi: siapa attacker, kapan, apa yang dilakukan |

| **Minggu, 22 Jun** | 20:00–21:00 | Flashcard: Event IDs + SPL cheatsheet — uji diri sendiri tanpa melihat | — |

  

---

  

## 🗓️ MINGGU 5 — 23–28 Juni 2026

**Tema: Sigma Rules + Hardening Linux Lanjut + Awal Windows Hardening**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 23 Jun** | 19:30–22:00 | Sigma rules: format YAML, struktur rule, tulis dari IOC, konversi ke SPL/Elastic | Tulis 2 Sigma rule valid dari scratch |

| **Selasa, 24 Jun** | 19:30–22:00 | Linux hardening part 2: SUID audit, `find / -perm -4000`, disable USB, Lynis scan | Jalankan Lynis dan fix 5 temuan hardening |

| **Rabu, 25 Jun** | 19:30–22:00 | Linux network security: `ufw`/`iptables`, SSH hardening (`/etc/ssh/sshd_config`), `fail2ban` | Konfigurasi SSH hanya key-based, block port tidak perlu |

| **Kamis, 26 Jun** | 19:30–22:00 | AppArmor + `sysctl` kernel hardening (ASLR, disable IPv6, core dump disable) | Enable AppArmor profile untuk 1 service + apply sysctl |

| **Jumat, 27 Jun** | 19:30–22:00 | Windows hardening part 1: `secpol.msc` (password+lockout policy), User Rights Assignment, Protected Users group | Konfigurasi Windows policy sesuai CIS benchmark dasar |

| **Sabtu, 28 Jun** | 09:00–12:00 | Lab hardening: VM Linux + Windows dari kondisi "fresh install" → hardening sampai Lynis score naik | Dokumentasi before/after score |

| **Minggu, 29 Jun** | 20:00–21:00 | Review Sigma + Linux hardening checklist | — |

  

---

  

## 🗓️ MINGGU 6 — 30 Juni – 5 Juli 2026

**Tema: Active Directory + GPO + Windows Defender/Sysmon**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 30 Jun** | 19:30–22:00 | Active Directory security: privileged groups (DA, EA, Schema Admin), Kerberoasting, LDAP signing | Bisa identify misconfigured AD group dari output `net group` |

| **Selasa, 1 Jul** | 19:30–22:00 | Kerberos delegation + disable NTLMv1 + LDAP signing enforce | Konfigurasi di lab AD: disable NTLMv1 + enforce signing |

| **Rabu, 2 Jul** | 19:30–22:00 | GPO: `gpedit.msc` + `gpmc`, AppLocker rules, PowerShell logging (ScriptBlock + Module logging) | Buat GPO AppLocker block .exe dari Downloads folder |

| **Kamis, 3 Jul** | 19:30–22:00 | GPO lanjut: disable WDigest, SMBv1, LLMNR via registry/GPO | Verifikasi dengan `Get-SmbServerConfiguration` |

| **Jumat, 4 Jul** | 19:30–22:00 | Windows Defender: AMSI, ASR rules (enable 13 rules), Tamper Protection | Enable semua ASR rules dan test detection |

| **Sabtu, 5 Jul** | 09:00–12:00 | Sysmon install + config XML (SwiftOnSecurity) + Windows Logging review | Sysmon running, Event ID 1/3/7/10/11 tercapture di Event Viewer |

| **Minggu, 6 Jul** | 20:00–21:00 | Review AD + GPO checklist — rekap dari senin sampai jumat | — |

  

---

  

## 🗓️ MINGGU 7 — 7–12 Juli 2026

**Tema: OS Forensic Windows + Linux + Timeline Analysis**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 7 Jul** | 19:30–22:00 | Windows artifacts part 1: Registry (`NTUSER.DAT`, Run key, `UserAssist`), `reg query` CLI | Extract 3 persistence artefak dari registry dump |

| **Selasa, 8 Jul** | 19:30–22:00 | Windows artifacts part 2: Prefetch (`.pf` files, `PECmd`), LNK files (`LECmd`), `$MFT` dasar | Analisis Prefetch untuk identifikasi program yang pernah jalan |

| **Rabu, 9 Jul** | 19:30–22:00 | Browser forensic: `hindsight` (Chrome history), Jump Lists, Recycle Bin (`$I` + `$R` files) | Extract history browser + temukan file yang dihapus |

| **Kamis, 10 Jul** | 19:30–22:00 | Linux artifacts: `bash_history`, crontab (`/var/spool/cron`), `/tmp`, `/etc/passwd`, `/etc/shadow` | Identifikasi backdoor dari cron + shadow file dump |

| **Jumat, 11 Jul** | 19:30–22:00 | Timeline analysis: `log2timeline` + `plaso` → `psort` → CSV export · timestomping detection | Buat super-timeline dari disk image sederhana |

| **Sabtu, 12 Jul** | 09:00–12:00 | Autopsy full session: mount disk image, artifact extraction, generate report | Report Autopsy dengan minimal 5 artefak terdokumentasi |

| **Minggu, 13 Jul** | 20:00–21:00 | Review: Windows artifact checklist — hafal nama file + lokasi path | — |

  

---

  

## 🗓️ MINGGU 8 — 14–19 Juli 2026

**Tema: Memory Forensic + Malware Static Analysis**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 14 Jul** | 19:30–22:00 | Volatility 3 setup · `pslist`, `pstree`, `cmdline`, `dlllist` — analisis proses mencurigakan | Identifikasi 2 proses injected dari memory dump Windows |

| **Selasa, 15 Jul** | 19:30–22:00 | Volatility 3 lanjut: `netscan`, `filescan`, `malfind`, `hashdump`, `registry.printkey` | Extract hash NTLM + list koneksi aktif dari memory |

| **Rabu, 16 Jul** | 19:30–22:00 | Malware static: `strings`, PEStudio, `capa` (Mandiant), entropy check, packer ID via DIE | Analisis 1 malware sample: output strings + capabilities |

| **Kamis, 17 Jul** | 19:30–22:00 | YARA rules: tulis rule dari strings malware, scan dengan `yara`, IOC extraction dari PEStudio | Tulis 2 YARA rule yang detect malware sample spesifik |

| **Jumat, 18 Jul** | 19:30–22:00 | Malware dynamic: Any.run + Hybrid Analysis — behavioral analysis, persistence, C2 identification | Submit 1 sample ke Any.run + baca laporan: persistence + network |

| **Sabtu, 19 Jul** | 09:00–12:00 | CTF memory forensic + malware challenge · Integrasi IOC → Sigma rule + YARA rule | Selesaikan 1 challenge + writeup lengkap |

| **Minggu, 20 Jul** | 20:00–21:00 | Review Volatility plugin cheatsheet + YARA syntax | — |

  

---

  

## 🗓️ MINGGU 9 — 21–26 Juli 2026

**Tema: Reverse Engineering Fondasi + Static + Dynamic**

  

| Hari & Tanggal | Waktu | Materi | Target Konkret |

|---|---|---|---|

| **Senin, 21 Jul** | 19:30–22:00 | Assembly x86_64 part 1: registers (`rax`–`rsp`), `mov/add/sub/cmp/jmp/call` — baca, bukan tulis | Trace 10 instruksi assembly dan tebak output program |

| **Selasa, 22 Jul** | 19:30–22:00 | Assembly x86_64 part 2: stack frame, calling convention, loop pattern, function prologue/epilogue | Rekonstruksi pseudocode dari assembly sederhana |

| **Rabu, 23 Jul** | 19:30–22:00 | ARM basics (r0–r15, Thumb mode, BL) + MIPS basics (delay slot, $a0–$a3, $ra) | Bisa baca output `objdump` ARM dan identifikasi fungsi main |

| **Kamis, 24 Jul** | 19:30–22:00 | ELF + PE format + Ghidra intro: import binary, navigate, rename variable, retype | Analyze 1 binary CTF easy di Ghidra + label semua fungsi |

| **Jumat, 25 Jul** | 19:30–22:00 | GDB + pwndbg: breakpoint, step, `info registers`, `x/s` examine memory — trace live binary | Set breakpoint di `main`, trace input validation |

| **Sabtu, 26 Jul** | 09:00–12:00 | CTF RE easy tier: PicoCTF reversing + Detect-it-Easy packer identification | Solve 2 soal RE PicoCTF + identifikasi packer 3 binary |

| **Minggu, 27 Jul** | — | **🎉 Materi baru selesai. Mulai fase murojaah.** | — |

  

---

  

## 🔁 FASE MUROJAAH — 27 Juli ke Lomba

**Tidak ada materi baru. Hanya drill, repetisi, dan perkuat yang lemah.**

  

> Ghidra advanced, z3 solver, Binary patching, Anti-RE, Languages in binary, Mobile RE —

> semua ini bisa dipelajari di fase murojaah sebagai **bonus** jika ada waktu lebih.

  

### Rotasi Murojaah Mingguan:

  

| Minggu | Fokus Utama | Fokus Kedua |

|---|---|---|

| 28 Jul – 3 Agt | Volatility 3 (memory forensic) | Wireshark + tshark |

| 4–10 Agt | Splunk SPL (BOTS dataset) | Sigma rules |

| 11–17 Agt | Ghidra static analysis | Assembly reading |

| 18–24 Agt | Hardening review (Linux + Windows) | Windows artifacts |

| 25 Agt+ | Simulasi penuh H3 (5 jam) | Writeup quality |

  

### Pola Harian Murojaah:

- **19:30–21:30**: 1–2 soal CTF dari kategori rotasi + writeup singkat (max 30 menit per soal)

- **21:30–22:00**: Baca catatan/cheatsheet topik besok

  

### Alarm Peringatan:

- Jika >3 hari tidak sentuh 1 topik → langsung 1 soal CTF untuk topik itu

- Jika simulasi full menunjukkan nilai rendah di 1 kategori → geser murojaah ke sana 3 hari

  

---

  

## 📊 Ringkasan Blok Waktu

  

| Blok | Tanggal | Tema |

|---|---|---|

| Minggu 1 | 27 Mei – 1 Jun | Fondasi + File Carving + Awal Stego |

| Minggu 2 | 2–7 Jun | Stego lanjut + Network + Linux CLI |

| Minggu 3 | 9–14 Jun | Wireshark + tshark + Log Analysis |

| Minggu 4 | 16–21 Jun | Event IDs + Splunk + Hardening Linux awal |

| Minggu 5 | 23–28 Jun | Sigma + Hardening Linux lanjut + Windows awal |

| Minggu 6 | 30 Jun – 5 Jul | Active Directory + GPO + Defender/Sysmon |

| Minggu 7 | 7–12 Jul | OS Forensic Windows + Linux + Timeline |

| Minggu 8 | 14–19 Jul | Memory Forensic + Malware Analysis |

| Minggu 9 | 21–26 Jul | Reverse Engineering (fondasi) |

| Murojaah | 27 Jul → Lomba | Drill semua kategori, simulasi penuh |

  

---

  

## 🔴 Prioritas Tidak Boleh Terlewat

  

1. **Volatility 3** — memory forensic Windows + Linux

2. **Wireshark + tshark** — PCAP analysis

3. **Ghidra** — static reverse engineering *(dipelajari fase murojaah jika belum)*

4. **Splunk SPL** — SIEM threat hunting

5. **`binwalk` + `steghide`** — file carving + stego

6. **Linux log analysis** — grep/awk pola serangan

7. **Windows Event IDs** — hafal 10 ID kritis

8. **Linux + Windows Hardening** — GPO, Sysmon, PAM

  

---

  

> **Ingat:** Jadwal ini bukan kontrak — ini kompas. Kalau satu hari terlewat, tidak perlu kejar dua materi besoknya.

> Cukup lanjut dari titik berhenti, dan konsisten. Satu jam malam ini lebih berharga dari rencana 10 jam yang tidak pernah dimulai.