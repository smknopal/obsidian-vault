# 📅 Jadwal Belajar Blue Team LKS 2026 — REVISI

### Versi Pemula · Strategi Catatan · 5 Juni – 27 Juli 2026

  

---

  

## 🔄 Apa yang Berubah dan Kenapa

  

| Aspek | Jadwal Asal | Jadwal Revisi | Alasan |

|---|---|---|---|

| **Kepadatan per sesi** | 2–3 topik sekaligus | 1 topik fokus per sesi | Pemula butuh kedalaman, bukan lebar |

| **Target belajar** | "Bisa X tanpa googling" | "Punya catatan X yang siap pakai" | Boleh lihat catatan saat lomba = catatan adalah senjata |

| **Cara ukur kemajuan** | Hafal syntax | Selesaikan 1 soal CTF + tulis writeup | Praktik lebih jujur dari hafalan |

| **Python session** | 1 malam, semua konsep | 2 sesi ringkas, template saja | base64 + hex sudah cukup untuk Blue Team awal |

| **Alur belajar** | Belajar → Hafal | Pahami → Praktik → Catat | Catatan yang bagus = persiapan lomba yang nyata |

| **Topik per minggu** | 5–7 topik berbeda | 2–4 topik yang saling berkaitan | Rotasi terlalu cepat untuk pemula |

  

---

  

## 📌 Filosofi Jadwal Baru

  

| Parameter | Detail |

|---|---|

| **Mulai** | Kamis, 5 Juni 2026 |

| **Selesai materi baru** | Minggu, 26 Juli 2026 |

| **Waktu belajar harian** | 19:30–22:00 (2,5 jam aktif) |

| **Waktu praktik Sabtu** | 09:00–12:00 (3 jam lab + CTF) |

| **Minggu** | Istirahat atau review catatan max 1 jam |

| **Output wajib tiap sesi** | 1 halaman catatan siap pakai di Obsidian |

  

> **Prinsip utama:** Karena boleh lihat catatan saat lomba, menghafal syntax bukan tujuan.

> Yang perlu dikuasai adalah: **tahu jenis soal apa ini → tahu tool apa yang dipakai → punya template tinggal eksekusi.**

>

> Satu soal CTF yang benar-benar diselesaikan sendiri nilainya lebih dari 10 halaman materi yang hanya dibaca.

  

---

  

## 🗓️ MINGGU 1 — 5–8 Juni 2026

**Tema: Selesaikan Fondasi + Pahami Bahasa File**

  

> Tujuan minggu ini: punya catatan template decode file aneh. Tidak perlu hafal Python dari kepala.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Kamis, 5 Jun** | 19:30–22:00 | Python CTF essentials: `base64`, bytes, `binascii` — decode saja, tidak perlu semua modul | Decode 3 string base64 berbeda tanpa lihat referensi | Template Python: decode base64, hex ke bytes, baca bytes file |

| **Jumat, 6 Jun** | 19:30–22:00 | Magic bytes + `file` command + `xxd` — identifikasi tipe file dari hex dump | Identifikasi 5 tipe file hanya dari baris pertama hex dump | Tabel magic bytes: PNG, ZIP, PDF, ELF, PE, GIF, JPEG + cara bacanya |

| **Sabtu, 7 Jun** | 09:00–12:00 | File carving: `binwalk` + `foremost` — ekstrak file tersembunyi dari file gabungan | Extract min. 2 file embedded dari 1 file CTF nyata + tulis mini-writeup | Workflow carving: `file → xxd → binwalk -e → foremost` |

| **Minggu, 8 Jun** | 20:00–21:00 | Review + rapikan catatan minggu ini di Obsidian | Semua catatan tersusun dan bisa dibaca ulang dalam 5 menit | — |

  

---

  

## 🗓️ MINGGU 2 — 9–14 Juni 2026

**Tema: Steganografi — Pesan Tersembunyi di File**

  

> Tujuan: tahu jenis stego apa yang dihadapi → tahu tool yang tepat. Decision tree, bukan hafalan.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 9 Jun** | 19:30–22:00 | Image stego theory: cara kerja LSB, bit planes, kenapa tidak terlihat mata | Jelaskan cara LSB menyembunyikan data tanpa lihat catatan | Decision tree: "dapat gambar mencurigakan → cek ini dulu → lalu ini" |

| **Selasa, 10 Jun** | 19:30–22:00 | `steghide` + `zsteg` — ekstrak data tersembunyi dari image | Solve 1 soal stego image PicoCTF end-to-end | Cheatsheet: `steghide extract -sf file.jpg`, `zsteg -a file.png` + output contoh |

| **Rabu, 11 Jun** | 19:30–22:00 | `stegsolve` — analisis colour channel, bit plane, visual clue | Temukan hidden data di gambar yang tidak bisa di-extract `steghide`/`zsteg` | Workflow stegsolve: saluran mana yang dicek duluan + kenapa |

| **Kamis, 12 Jun** | 19:30–22:00 | Audio stego: Sonic Visualizer + spectrogram + DTMF decoder | Deteksi dan baca pesan tersembunyi di file `.wav` dari CTF | Checklist audio stego: tools + urutan pengecekan |

| **Jumat, 13 Jun** | 19:30–22:00 | Document forensics: `olevba` + `peepdf` — analisis macro Word/PDF berbahaya | Ekstrak macro dari `.doc` dan identifikasi payload-nya | Command template: `olevba file.doc`, `peepdf -i file.pdf` + contoh output |

| **Sabtu, 14 Jun** | 09:00–12:00 | CTF mix: 1 soal file carving + 1 soal stego image + 1 soal audio/document | Selesaikan 3 soal + mini-writeup tiap soal (apa masalahnya, tool apa, langkah apa) | — |

| **Minggu, 15 Jun** | 20:00–21:00 | Review: flashcard magic bytes + decision tree stego | — | — |

  

---

  

## 🗓️ MINGGU 3 — 16–21 Juni 2026

**Tema: Wireshark + Analisis Lalu Lintas Jaringan**

  

> Tujuan: buka PCAP → isolasi traffic mencurigakan → baca apa yang terjadi → temukan flag.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 16 Jun** | 19:30–22:00 | Wireshark dasar: filter syntax — `ip.addr`, `tcp.port`, `http`, `dns`, `frame contains` | Isolasi semua HTTP request dari PCAP contoh dalam 5 menit | Cheatsheet filter Wireshark: 15 filter paling sering muncul di CTF |

| **Selasa, 17 Jun** | 19:30–22:00 | Follow TCP stream + export HTTP objects + credential cleartext di FTP/HTTP | Extract username + password dari PCAP nyata (lab simulasi) | Workflow: "cari credential di PCAP → langkah 1 sampai 5" |

| **Rabu, 18 Jun** | 19:30–22:00 | Deteksi anomali: port scan (banyak SYN), DNS tunneling, C2 beacon pattern | Identifikasi 3 jenis anomali dari 1 file PCAP | Tabel: pattern anomali → cara identifikasi di Wireshark → apa artinya |

| **Kamis, 19 Jun** | 19:30–22:00 | Protokol wajib: HTTP, FTP, DNS, SMB, ICMP — apa yang bisa dilihat di tiap protokol | Rekonstruksi 1 skenario serangan dari PCAP multi-protokol | Notes: "kalau ada protokol X → cek field Y → artinya Z" |

| **Jumat, 20 Jun** | 19:30–22:00 | `tshark` CLI: extract fields via terminal — DNS query, HTTP header | Extract semua DNS query dari PCAP hanya dengan 1 command | Template `tshark`: 5 use case paling sering di CTF + command lengkap |

| **Sabtu, 21 Jun** | 09:00–12:00 | CTF: 2 soal Wireshark dari PicoCTF atau HTB Academy | Selesaikan 2 soal + writeup teknis (filter yang dipakai + kenapa) | — |

| **Minggu, 22 Jun** | 20:00–21:00 | Review catatan Wireshark — pastikan filter cheatsheet sudah lengkap dan jelas | — | — |

  

---

  

## 🗓️ MINGGU 4 — 23–28 Juni 2026

**Tema: Log Analysis + Splunk SPL + Windows Event IDs**

  

> Tujuan: bisa baca log serangan dan tulis query SIEM untuk deteksi ancaman.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 23 Jun** | 19:30–22:00 | Linux log analysis: `auth.log`, `syslog`, Apache access log + pola brute force via `grep`/`awk` | Tulis 1-liner grep untuk deteksi 5+ failed login dari `auth.log` | Template: 5 grep/awk query paling berguna untuk analisis log Linux |

| **Selasa, 24 Jun** | 19:30–22:00 | Windows Event IDs: 4624/4625/4648/4688/4720/4732/7045/1102 + Logon Types | Bisa jelaskan arti 10 Event ID kritis dan kapan dicurigai | Tabel Event ID: ID → Artinya → Tanda serangan apa |

| **Rabu, 25 Jun** | 19:30–22:00 | Splunk SPL part 1: `search`, `stats`, `timechart`, `table`, `where`, `dedup` | Query event login gagal per IP dalam 1 jam terakhir | Cheatsheet SPL: perintah dasar + contoh query dengan output |

| **Kamis, 26 Jun** | 19:30–22:00 | Splunk SPL part 2: `eval`, `rex`, `lookup`, alert logic | Buat 1 alert Splunk untuk deteksi brute force (threshold 5x gagal/menit) | Template alert Splunk: kondisi + threshold + tindakan |

| **Jumat, 27 Jun** | 19:30–22:00 | `auditd` + `journalctl` + `/var/log/secure` — lateral movement pattern | Tulis query auditd untuk deteksi `sudo` abuse dan privilege escalation | Workflow: "dapat log → cari ini → artinya serangan apa" |

| **Sabtu, 28 Jun** | 09:00–12:00 | Threat hunting: Splunk BOTS dataset — cari 1 attack chain lengkap | Dokumentasikan: siapa attacker, kapan, apa yang dilakukan, bukti di log | — |

| **Minggu, 29 Jun** | 20:00–21:00 | Flashcard: Event IDs + SPL cheatsheet — uji diri tanpa melihat | — | — |

  

---

  

## 🗓️ MINGGU 5 — 30 Juni – 5 Juli 2026

**Tema: Windows Forensics + Sigma Rules + Hardening Dasar**

  

> Tujuan: bisa extract artefak Windows, tulis Sigma rule, dan pahami konfigurasi keamanan dasar.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 30 Jun** | 19:30–22:00 | Registry artifacts: `NTUSER.DAT`, Run key, `UserAssist` — `reg query` CLI | Extract 3 artefak persistence dari registry dump | Lokasi registry kritis: path → artinya → tanda serangan |

| **Selasa, 1 Jul** | 19:30–22:00 | Prefetch (`.pf` files, `PECmd`) + LNK files (`LECmd`) | Identifikasi program yang pernah jalan dari Prefetch | Template command: `PECmd.exe -f file.pf` + cara baca output |

| **Rabu, 2 Jul** | 19:30–22:00 | Browser forensics: `hindsight` (Chrome history) + Recycle Bin (`$I` + `$R` files) | Extract browser history + file yang dihapus dari Recycle Bin | Lokasi artefak browser di Windows + cara baca `$I`/`$R` |

| **Kamis, 3 Jul** | 19:30–22:00 | Windows hardening: `secpol.msc`, GPO, AppLocker, Sysmon — konsep + konfigurasi dasar | Konfigurasi password policy + lockout sesuai CIS benchmark | Checklist hardening Windows (untuk H1 lomba) |

| **Jumat, 4 Jul** | 19:30–22:00 | Sigma rules: format YAML, struktur rule, tulis dari IOC, konversi ke SPL | Tulis 2 Sigma rule valid dari scratch | Template Sigma rule: struktur YAML + contoh nyata |

| **Sabtu, 5 Jul** | 09:00–12:00 | Autopsy: mount disk image, artifact extraction, generate report | Report Autopsy dengan min. 5 artefak terdokumentasi | — |

| **Minggu, 6 Jul** | 20:00–21:00 | Review Windows artifact checklist — nama file + lokasi path + artinya | — | — |

  

---

  

## 🗓️ MINGGU 6 — 7–12 Juli 2026

**Tema: Memory Forensics + Timeline Analysis**

  

> Tujuan: bisa analisis RAM dump dan buat timeline kejadian dari disk image.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 7 Jul** | 19:30–22:00 | Volatility 3 setup + `pslist`, `pstree`, `cmdline`, `dlllist` — proses mencurigakan | Identifikasi 2 proses injected dari memory dump Windows | Cheatsheet plugin Volatility: plugin → fungsi → kapan dipakai |

| **Selasa, 8 Jul** | 19:30–22:00 | Volatility lanjut: `netscan`, `filescan`, `malfind`, `hashdump`, `registry.printkey` | Extract hash NTLM + list koneksi aktif dari memory dump | Template command per kasus: "mau cari X → pakai plugin Y dengan flag Z" |

| **Rabu, 9 Jul** | 19:30–22:00 | Linux memory artifacts: `bash_history`, crontab, `/tmp`, `/etc/passwd`, `/etc/shadow` | Identifikasi backdoor dari cron + analisis shadow file dump | Checklist Linux artifact: file → lokasi → apa yang dicurigai |

| **Kamis, 10 Jul** | 19:30–22:00 | Timeline analysis: `log2timeline` + `plaso` → `psort` → CSV export | Buat super-timeline dari disk image sederhana | Workflow timeline: langkah lengkap + command dengan flag yang benar |

| **Jumat, 11 Jul** | 19:30–22:00 | Malware static basics: `strings`, PEStudio, entropy check, identifikasi packer | Analisis 1 malware sample: strings penting + capabilities overview | Checklist analisis statis: langkah → tool → yang dicari |

| **Sabtu, 12 Jul** | 09:00–12:00 | CTF memory forensic challenge + malware static | Selesaikan 1 challenge memory forensics + writeup lengkap | — |

| **Minggu, 13 Jul** | 20:00–21:00 | Review Volatility plugin cheatsheet | — | — |

  

---

  

## 🗓️ MINGGU 7 — 14–19 Juli 2026

**Tema: YARA Rules + Reverse Engineering Fondasi**

  

> Tujuan: tulis YARA rule sederhana dan bisa baca assembly dasar untuk CTF easy tier.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 14 Jul** | 19:30–22:00 | YARA rules: tulis dari strings malware, scan dengan `yara`, IOC extraction | Tulis 2 YARA rule yang berhasil detect malware sample | Template YARA rule: struktur + kondisi + contoh nyata |

| **Selasa, 15 Jul** | 19:30–22:00 | Assembly x86_64 part 1: registers (`rax`–`rsp`), `mov`, `add`, `cmp`, `jmp`, `call` | Trace 5 instruksi assembly dan tebak output program | Tabel register x86_64 + instruksi paling umum + artinya dalam pseudocode |

| **Rabu, 16 Jul** | 19:30–22:00 | Assembly x86_64 part 2: stack frame, loop pattern, function prologue/epilogue | Rekonstruksi pseudocode dari fungsi sederhana berisi loop | Pola assembly yang sering muncul di CTF easy tier |

| **Kamis, 17 Jul** | 19:30–22:00 | Ghidra intro: import binary, navigate, rename variable, retype | Analyze 1 binary CTF easy di Ghidra + label semua fungsi penting | Workflow Ghidra: langkah-langkah pertama saat dapat binary baru |

| **Jumat, 18 Jul** | 19:30–22:00 | GDB + pwndbg: breakpoint, step, `info registers`, `x/s` examine memory | Set breakpoint di `main`, trace alur input validation | Cheatsheet GDB: command paling berguna + contoh penggunaan |

| **Sabtu, 19 Jul** | 09:00–12:00 | CTF RE easy tier: 2 soal PicoCTF reversing + identifikasi packer dengan DIE | Solve 2 soal RE + identifikasi packer pada 2 binary berbeda | — |

| **Minggu, 20 Jul** | 20:00–21:00 | Review YARA syntax + pola assembly paling umum | — | — |

  

---

  

## 🗓️ MINGGU 8 — 21–26 Juli 2026

**Tema: Konsolidasi + Simulasi Kondisi Lomba**

  

> Tujuan: identifikasi celah, kuatkan yang lemah, rapikan semua catatan, simulasi nyata.

  

| Hari & Tanggal | Waktu | Topik | Target Konkret | Output Catatan |

|---|---|---|---|---|

| **Senin, 21 Jul** | 19:30–22:00 | Gap analysis: baca ulang semua catatan, tandai bagian yang masih ragu atau kosong | Daftar 5 topik paling lemah yang perlu di-drill | Prioritas murojaah: topik diurutkan dari yang paling lemah |

| **Selasa, 22 Jul** | 19:30–22:00 | Drill topik lemah #1 (dari hasil gap analysis Senin) | Selesaikan 1 soal CTF untuk topik tersebut | Perbaiki/lengkapi catatan topik ini |

| **Rabu, 23 Jul** | 19:30–22:00 | Drill topik lemah #2 | Selesaikan 1 soal CTF untuk topik tersebut | Perbaiki/lengkapi catatan topik ini |

| **Kamis, 24 Jul** | 19:30–22:00 | Organisasi catatan final — pastikan mudah dibaca dan bisa dinavigasi cepat saat lomba | Semua catatan terstruktur, ada index/TOC, bisa dicari | — |

| **Jumat, 25 Jul** | 19:30–22:00 | Explain ke diri sendiri tanpa buka catatan — verifikasi pemahaman workflow | Bisa jelaskan workflow tiap topik dari kepala tanpa syntax detail | — |

| **Sabtu, 26 Jul** | 09:00–12:00 | **Simulasi H3 penuh (5 jam)** — kondisi lomba sebenarnya, boleh lihat catatan | Selesaikan sebanyak mungkin + ukur di kategori mana paling lemah | — |

| **Minggu, 27 Jul** | — | **Materi baru selesai. Mulai fase murojaah.** | — | — |

  

---

  

## 🔁 FASE MUROJAAH — 27 Juli → Lomba

**Tidak ada materi baru. Drill, repetisi, perkuat yang lemah.**

  

### Rotasi Murojaah Mingguan

  

| Minggu | Fokus Utama | Fokus Kedua |

|---|---|---|

| 28 Jul – 3 Agt | Volatility 3 + Memory forensics | Wireshark + tshark |

| 4–10 Agt | Splunk SPL (BOTS dataset) | Sigma rules + log analysis |

| 11–17 Agt | Ghidra static + Assembly reading | YARA rules |

| 18–24 Agt | Windows forensics artifacts | Hardening checklist (H1) |

| 25 Agt+ | Simulasi penuh H3 (5 jam) | Writeup quality |

  

### Pola Harian Murojaah

- **19:30–21:30**: 1–2 soal CTF dari kategori rotasi + writeup singkat (max 30 menit per soal)

- **21:30–22:00**: Baca catatan topik besok — bukan belajar baru, hanya refresh

  

### Alarm Murojaah

- Jika >3 hari tidak sentuh 1 topik → langsung 1 soal CTF topik itu

- Jika simulasi penuh menunjukkan skor rendah di 1 kategori → geser fokus ke sana 3 hari

  

---

  

## 📊 Ringkasan Blok Waktu

  

| Blok | Tanggal | Tema |

|---|---|---|

| Minggu 1 | 5–8 Jun | Python Essentials + File Identification + File Carving |

| Minggu 2 | 9–14 Jun | Steganografi (image + audio) + Document Forensics |

| Minggu 3 | 16–21 Jun | Wireshark + tshark + Network Forensics |

| Minggu 4 | 23–28 Jun | Log Analysis + Splunk SPL + Windows Event IDs |

| Minggu 5 | 30 Jun–5 Jul | Windows Forensics + Sigma Rules + Hardening Dasar |

| Minggu 6 | 7–12 Jul | Memory Forensics (Volatility 3) + Timeline Analysis |

| Minggu 7 | 14–19 Jul | YARA Rules + Reverse Engineering Fondasi |

| Minggu 8 | 21–26 Jul | Konsolidasi + Gap Analysis + Simulasi Penuh |

| Murojaah | 27 Jul → Lomba | Drill semua kategori, simulasi penuh |

  

---

  

## 🔴 Prioritas Tidak Boleh Terlewat

  

1. **Wireshark + tshark** — PCAP analysis (sering muncul di H3)

2. **Splunk SPL** — SIEM threat hunting (porsi besar H3)

3. **Volatility 3** — memory forensics Windows

4. **File carving + stego** — `binwalk`, `steghide`, `zsteg`

5. **Linux log analysis** — grep/awk pola serangan

6. **Windows Event IDs** — hafal 10 ID kritis + artinya

7. **Ghidra** — static reverse engineering (dipelajari/diperdalam saat murojaah)

8. **Linux + Windows Hardening** — untuk H1

  

---

  

## 📝 Template Prompt Belajar

  

> Ini template yang bisa langsung dipakai saat belajar tiap topik.

> Prinsip: **satu prompt = satu tujuan konkret.**

  

---

  

### Template Standar (Pakai Ini Setiap Sesi)

  

```

[KONTEKS]

Aku siswa SMK, persiapan LKS 2026 Blue Team Cybersecurity.

Level: Pemula — baru mulai Blue Team dari awal.

Format lomba: boleh lihat catatan saat kompetisi.

Tujuan belajar: pahami workflow dan kapan pakai tool ini, bukan hafal syntax.

  

[TOPIK HARI INI — satu topik saja]

Topik spesifik: [isi dengan satu hal, contoh: "base64 decode di CTF"]

Yang sudah aku tahu tentang ini: [isi jujur, boleh "tidak tahu sama sekali"]

Yang belum aku mengerti: [isi spesifik, contoh: "tidak tahu bedanya encode vs encrypt"]

  

[YANG AKU BUTUHKAN — dalam urutan ini]

1. Apa ini? (max 3 kalimat, pakai analogi kalau bisa)

2. Kapan ini muncul di soal CTF? (1 contoh skenario nyata)

3. Workflow: kalau dapat soal tipe ini, langkah 1 sampai dapat flag

4. Template perintah/kode minimal untuk catatanku (dengan contoh output nyata, bukan placeholder)

5. Berikan 1 soal latihan sederhana — jangan kasih jawaban dulu

  

[ATURAN]

- Bahasa Indonesia

- Satu topik ini saja, jangan melebar ke topik lain

- Kalau ada istilah teknis, beri penjelasan singkat dalam kurung

- Output terminal/kode harus menunjukkan hasil nyata

```

  

---

  

### Prompt Follow-up (Gunakan Setelah Prompt Standar)

  

**Kalau tidak paham penjelasannya:**

```

Bagian [tulis bagian yang bingung] aku masih tidak mengerti.

Jelaskan ulang dengan analogi yang berbeda atau contoh yang lebih sederhana.

Jangan lanjut ke langkah berikutnya dulu.

```

  

**Kalau mau cek apakah sudah benar paham:**

```

Cek pemahaman aku:

[tulis apa yang kamu pahami dengan kata-katamu sendiri]

Apa ada yang salah atau kurang tepat?

```

  

**Kalau mau lanjut ke soal berikutnya:**

```

Aku sudah selesaikan soal latihan tadi. Jawabanku: [tulis langkah yang kamu ambil].

Apakah cara itu benar? Kalau iya, berikan soal berikutnya — sedikit lebih susah.

```

  

**Kalau mau buat catatan dari sesi ini:**

```

Berdasarkan yang sudah kita bahas, buatkan aku catatan ringkas dalam format ini:

- Kapan digunakan: [1–2 kalimat]

- Workflow: [numbered list]

- Command/template siap pakai: [code block]

- Jangan lupa: [2–3 hal yang sering salah]

Format untuk Obsidian, tanpa penjelasan panjang.

```

  

---

  

### Contoh Prompt yang Salah vs Benar

  

**❌ Prompt yang Salah (terlalu lebar, output tidak berguna):**

```

Topik malam ini: Python CTF: bytes, struct, base64, hashlib · File formats: hex, magic bytes, xxd

Buatkan konsep singkat + cheatsheet + workflow + soal latihan + dari 0 sampai bisa + desain keren untuk Obsidian

```

  

Masalahnya:

- 6 konsep sekaligus → hasilnya lebar tapi dangkal

- "Singkat" bertentangan dengan "dari 0 sampai bisa"

- "Desain keren" mengalihkan fokus AI dari kedalaman materi

- Tidak ada info level → AI tidak tahu harus mulai dari mana

  

**✅ Prompt yang Benar (fokus, satu tujuan):**

```

Konteks: LKS Blue Team, pemula, boleh lihat catatan saat lomba.

Yang sudah aku tahu: encoding dan decoding secara umum.

Yang belum aku tahu: apa itu base64 dan kapan muncul di soal CTF.

  

Tolong jelaskan:

1. Apa itu base64 dalam 3 kalimat (pakai analogi)

2. Satu contoh skenario CTF dimana base64 muncul

3. Command paling sederhana untuk decode di terminal Linux

4. Satu soal latihan — jangan kasih jawaban dulu

```

  

---

  

### Prompt Khusus per Topik

  

**File Carving & Magic Bytes:**

```

Topik: binwalk untuk file carving di CTF forensics

Level: sudah tahu xxd dan file command, belum pernah pakai binwalk

Yang ingin dipahami: cara kerja binwalk dan kapan dipakai vs foremost

[lanjutkan dengan template standar di atas]

```

  

**Wireshark:**

```

Topik: filter Wireshark untuk isolasi satu jenis traffic

Level: baru pertama buka Wireshark, tahu konsep HTTP dan DNS

Yang ingin dipahami: cara tulis filter yang benar, bukan klik-klik menu

[lanjutkan dengan template standar di atas]

```

  

**Splunk SPL:**

```

Topik: query Splunk untuk deteksi brute force login

Level: pernah lihat Splunk tapi belum pernah tulis SPL

Yang ingin dipahami: sintaks dasar SPL dan cara ukur kejadian dalam rentang waktu

[lanjutkan dengan template standar di atas]

```

  

**Volatility 3:**

```

Topik: plugin pslist dan pstree di Volatility 3

Level: belum pernah pakai Volatility, tahu konsep process di OS

Yang ingin dipahami: cara install, cara run, dan cara baca outputnya

[lanjutkan dengan template standar di atas]

```

  

**Assembly x86_64:**

```

Topik: instruksi mov dan cmp di assembly x86_64

Level: tahu bahasa Python, belum pernah baca assembly sama sekali

Yang ingin dipahami: cara baca instruksi ini dan apa artinya dalam pseudocode

[lanjutkan dengan template standar di atas]

```

  

---

  

## ⚠️ Prinsip Saat Belajar

  

1. **Jangan lanjut jika masih bingung** — minta penjelasan ulang dengan contoh berbeda

2. **Tulis catatan selama sesi berlangsung** — jangan tunggu setelah selesai

3. **Praktik soal sebelum pindah topik** — minimal 1 soal CTF per topik

4. **Catatan = produk akhir sesi** — kalau tidak ada catatan, sesi belum selesai

5. **Satu malam, satu topik** — tidak ada kejar-kejaran, yang penting dalam

  

> Jadwal ini bukan kontrak — ini kompas.

> Kalau satu hari terlewat, cukup lanjut dari titik berhenti.

> Satu jam fokus malam ini lebih berharga dari rencana 10 jam yang tidak dimulai.