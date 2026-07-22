# BT-H1 — Windows Event Log, SIEM Triage, dan Timeline Investigation

  

> Modul fondasi investigasi log Windows untuk persiapan LKS Cyber Security.

  

**STATUS: README H1 — FINAL TAHAP 2**

  

> README ini adalah pusat navigasi. Materi teknis lengkap berada pada file 01–07. Kunci jawaban hanya berada pada file 99 dan tidak boleh dibuka sebelum latihan selesai.

  

---

  

## 1. Tentang H1

  

H1 adalah hari pertama pembelajaran Blue Team, bukan Hari 1 lomba.

  

Fokus H1 adalah Windows Event Log, SIEM triage, dan timeline investigation.

  

H1 menjadi fondasi untuk seluruh topik forensic berikutnya. Kemampuan membaca log adalah dasar yang dibutuhkan untuk kerja SIEM, incident response, dan write-up.

  

H1 dirancang dengan pendekatan **GUI-first**. Command-line tetap disediakan sebagai jalur cadangan, bukan jalur utama.

  

---

  

## 2. Tujuan Akhir H1

  

Alur kemampuan yang ditargetkan:

  

```

Diberi Windows Event Log atau data SIEM

→ memahami objective

→ menentukan evidence

→ mengatur time range

→ memfilter event

→ membaca field

→ melakukan pivot

→ menghubungkan beberapa event

→ memverifikasi temuan

→ menyimpan bukti

→ menjawab soal

→ membuat write-up

```

  

**Kompetensi akhir:**

  

- [ ] Bisa membuka EVTX.

- [ ] Bisa memproses EVTX menjadi CSV.

- [ ] Bisa memfilter Event ID.

- [ ] Bisa mencari user, host, IP, process, dan timestamp.

- [ ] Bisa membedakan Subject dan Target.

- [ ] Bisa membaca Logon Type.

- [ ] Bisa membuat timeline.

- [ ] Bisa mengorelasikan minimal dua atau tiga event.

- [ ] Bisa membedakan indikasi dan kesimpulan.

- [ ] Bisa menyimpan screenshot bukti.

- [ ] Bisa membuat findings dan write-up.

- [ ] Bisa menggunakan Combat Card saat lomba.

  

---

  

## 3. Batas Cakupan H1

  

### Termasuk dalam H1

  

Dasar investigasi Blue Team, evidence handling, EVTX, Windows Security Log, Windows System Log, Windows Application Log, operational logs, authentication, account modification, group modification, process execution, PowerShell logging, scheduled task, service installation, event log clearing, Defender event dasar, Sysmon event dasar (jika tersedia), timeline, correlation, SIEM triage, query logic, screenshot evidence, findings, dan write-up.

  

### Di luar cakupan H1

  

Registry forensics mendalam, Prefetch, Amcache, Shimcache, LNK, Jump Lists, ShellBags, disk image, NTFS internals, PCAP, memory forensics, Volatility, malware reverse engineering, assembly, FTK Imager, Autopsy, Wireshark, NetworkMiner, Ghidra, IDA.

  

Topik-topik di atas akan dibahas pada modul hari berikutnya, bukan pada H1.

  

---

  

## 4. Tools yang Digunakan

  

| Tool | Fungsi | Jalur penggunaan | Status |

|---|---|---|---|

| Windows Event Viewer | Membuka satu EVTX secara manual, membaca tab General dan XML | Jalur utama (GUI) | [ ] Siap / [ ] Belum diuji |

| EvtxECmd | Parsing EVTX menjadi CSV | Jalur parser | [ ] Siap / [ ] Belum diuji |

| Timeline Explorer | Filter, sort, pencarian, dan timeline atas CSV | Jalur utama (GUI) | [ ] Siap / [ ] Belum diuji |

| CyberChef | Decode Base64, URL, hex, timestamp, atau PowerShell encoded command | Alat bantu | [ ] Siap / [ ] Belum diuji |

| CMD | Menjalankan command sederhana | Jalur cadangan | [ ] Siap / [ ] Belum diuji |

| PowerShell | Export log, hash, folder handling, otomasi sederhana | Jalur cadangan | [ ] Siap / [ ] Belum diuji |

  

> **PERINGATAN:** Jangan menandai tool sebagai siap sebelum benar-benar diverifikasi berjalan di perangkat yang akan digunakan.

  

Event Viewer dan Timeline Explorer adalah jalur analisis utama. EvtxECmd berperan sebagai parser. CyberChef hanya alat bantu decode. CMD dan PowerShell adalah jalur cadangan atau otomasi sederhana. SIEM (misalnya Wazuh atau Elastic) hanya dibahas dari sisi filtering, query, time range, pivoting, correlation, field inspection, timeline, dan evidence collection — bukan instalasi.

  

---

  

## 5. Checklist Kesiapan Sebelum Belajar

  

- [x] Event Viewer dapat dibuka.

- [x] EvtxECmd tersedia.

- [ ] Timeline Explorer dapat dibuka.

- [ ] CyberChef lokal dapat dibuka.

- [ ] CMD dapat digunakan.

- [ ] PowerShell dapat digunakan.

- [ ] Folder H1 tersedia.

- [ ] Folder evidence lengkap.

- [ ] Ruang disk cukup.

- [ ] Timezone Windows sudah benar.

- [ ] Answer Key belum dibuka.

- [ ] Folder screenshot siap.

- [ ] Aplikasi berat yang tidak dibutuhkan sudah ditutup.

  

> **COMBAT RULE:** Jangan mulai Guided Lab sebelum tool utama berhasil dibuka.

  

---

  

## 6. Struktur Folder dan Fungsi

  

```

BT-H1_EVENT_LOG_AND_SIEM/

│

├── 00_README_H1.md

├── 01_H1_INVESTIGATION_FOUNDATION.md

├── 02_H1_WINDOWS_EVENT_LOG.md

├── 03_H1_TOOL_EXECUTION.md

├── 04_H1_GUIDED_LAB.md

├── 05_H1_COMPETITION_DRILL.md

├── 06_H1_COMBAT_CARD.md

├── 07_H1_WRITEUP_AND_WORKSHEET.md

├── 99_H1_ANSWER_KEY.md

│

└── EVIDENCE/

    ├── INBOX/

    ├── ORIGINAL/

    ├── WORKING/

    └── OUTPUT/

```

  

**Fungsi setiap file:**

  

- **00_README_H1.md** — Pusat navigasi dan progres.

- [Investigation Foundation](01_H1_INVESTIGATION_FOUNDATION.md) — Cara berpikir investigasi, evidence integrity, timestamp, timezone, hash, false positive, dan batas kesimpulan.

- [Windows Event Log](02_H1_WINDOWS_EVENT_LOG.md) — Materi teknis Windows Event Log: field, Event ID, Logon Type, timeline, correlation, dan SIEM logic.

- [Tool Execution](03_H1_TOOL_EXECUTION.md) — Panduan penggunaan Event Viewer, EvtxECmd, Timeline Explorer, CyberChef, CMD, dan PowerShell.

- [Guided Lab](04_H1_GUIDED_LAB.md) — Latihan dengan panduan penuh.

- [Competition Drill](05_H1_COMPETITION_DRILL.md) — Simulasi dengan panduan minimal dan batas waktu.

- [Combat Card](06_H1_COMBAT_CARD.md) — Cheatsheet utama saat lomba.

- [Write-up and Worksheet](07_H1_WRITEUP_AND_WORKSHEET.md) — Worksheet investigasi dan template write-up.

- **99_H1_ANSWER_KEY.md** — Jawaban dan evaluasi latihan.

  

> **ANTI-SPOILER:** Berkas `99_H1_ANSWER_KEY.md` berisi jawaban. Jangan dibuka sebelum latihan pada file 04 dan 05 selesai dikerjakan.

  

---

  

## 7. Fungsi Folder Evidence

  

- **INBOX** — Tempat file yang baru diterima atau diunduh.

- **ORIGINAL** — Salinan evidence asli yang tidak boleh diedit atau dianalisis langsung.

- **WORKING** — Salinan evidence untuk analisis.

- **OUTPUT** — CSV, screenshot, findings, timeline, hasil ekspor, dan write-up.

  

**Alur evidence:**

  

```

INBOX → ORIGINAL → hash → WORKING → analisis → OUTPUT

```

  

> **COMBAT RULE:** Jangan menganalisis langsung dari ORIGINAL.

  

---

  

## 8. Urutan Belajar H1

  

**Fase 1 — Orientasi**

Buka: `00_README_H1.md`

Target: memahami struktur, memeriksa tools, memeriksa folder.

  

**Fase 2 — Fondasi Investigasi**

Buka: [Investigation Foundation](01_H1_INVESTIGATION_FOUNDATION.md)

Target: memahami alur QUESTION → EVIDENCE → FILTER → PIVOT → CORRELATE → VERIFY → ANSWER → DOCUMENT; memahami WHO, WHAT, WHEN, WHERE, HOW, dan PROOF; memahami perbedaan ORIGINAL dan WORKING; memahami timezone dan hash.

  

**Fase 3 — Windows Event Log**

Buka: [Windows Event Log](02_H1_WINDOWS_EVENT_LOG.md)

Target: memahami log, event, field, Event ID, Logon Type, dan correlation.

  

**Fase 4 — Tool Execution**

Buka: [Tool Execution](03_H1_TOOL_EXECUTION.md)

Target: bisa menjalankan seluruh workflow tools.

  

**Fase 5 — Guided Lab**

Buka: [Guided Lab](04_H1_GUIDED_LAB.md), [Write-up and Worksheet](07_H1_WRITEUP_AND_WORKSHEET.md)

Target: menyelesaikan latihan dengan panduan, menyimpan screenshot, membuat findings.

  

**Fase 6 — Competition Drill**

Buka: [Competition Drill](05_H1_COMPETITION_DRILL.md), [Combat Card](06_H1_COMBAT_CARD.md), [Write-up and Worksheet](07_H1_WRITEUP_AND_WORKSHEET.md)

Target: menyelesaikan investigasi dalam batas waktu, mengandalkan cheatsheet, membuat write-up.

  

**Fase 7 — Evaluasi**

Buka: `99_H1_ANSWER_KEY.md` (hanya setelah latihan selesai)

Target: membandingkan temuan, memperbaiki interpretasi, memperbaiki Combat Card bila perlu.

  

---

  

## 9. Estimasi Waktu Belajar

  

| Bagian | Estimasi |

|---|---:|

| Orientasi dan tool check | 20–30 menit |

| Fondasi investigasi | 45–60 menit |

| Windows Event Log | 90–120 menit |

| Tool execution | 60–90 menit |

| Guided Lab | 90–120 menit |

| Competition Drill | 60 menit |

| Review dan Combat Card | 30–45 menit |

  

Total perkiraan: sekitar 6–8 jam efektif. Waktu ini bukan durasi pasti dan dapat dibagi menjadi dua sesi jika terlalu berat dalam satu waktu.

  

---

  

## 10. Metode Belajar

  

```

PAHAMI → PRAKTIK → VERIFIKASI → INTERPRETASI → DOKUMENTASI → ULANG TANPA PANDUAN

```

  

**WAJIB PAHAM:**

  

- Jangan hanya membaca.

- Jangan hanya menyalin command.

- Setiap langkah harus diverifikasi.

- Setiap temuan harus dijelaskan maknanya.

- Setiap latihan harus menghasilkan output.

- Setiap kesalahan harus dicatat.

  

---

  

## 11. Level Penguasaan

  

| Level | Arti |

|---|---|

| 0 | Belum pernah melihat |

| 1 | Mengenal konsep |

| 2 | Bisa dengan panduan penuh |

| 3 | Bisa dengan cheatsheet |

| 4 | Bisa tanpa panduan |

| 5 | Siap kompetisi |

  

**Target H1:**

  

- Investigation workflow: level 4.

- Event Log filtering: level 4.

- Subject vs Target: level 4.

- Logon Type: level 4.

- Timeline correlation: minimal level 3.

- Tool execution: minimal level 3.

- Write-up: minimal level 3.

- SIEM logic: minimal level 3.

  

---

  

## 12. Output Wajib H1

  

**OUTPUT WAJIB** setelah H1 selesai:

  

- [ ] Minimal satu folder kasus.

- [ ] Minimal satu salinan evidence ORIGINAL.

- [ ] Minimal satu salinan WORKING.

- [ ] Minimal satu hash SHA-256.

- [ ] Minimal satu CSV hasil EvtxECmd.

- [ ] Minimal tiga screenshot evidence.

- [ ] Minimal satu file findings.

- [ ] Minimal satu timeline.

- [ ] Minimal satu write-up.

- [ ] Combat Card yang dapat dicari cepat.

- [ ] Catatan kesalahan pribadi.

  

---

  

## 13. Naming Convention

  

**Case ID:** `H1-LAB-01`, `H1-LAB-02`, `H1-LAB-03`

  

**Drill ID:** `H1-DRILL-01`, `H1-DRILL-02`

  

**Screenshot:**

`H1-LAB-01_01_EVENT_OVERVIEW.png`

`H1-LAB-01_02_SUBJECT_TARGET.png`

`H1-LAB-01_03_TIMELINE.png`

  

**Output:**

`H1-LAB-01_EVTX.csv`

`H1-LAB-01_FINDINGS.md`

`H1-LAB-01_WRITEUP.md`

`H1-DRILL-01_FINDINGS.md`

`H1-DRILL-01_WRITEUP.md`

  

Nama harus konsisten agar write-up mudah dibuat dan ditelusuri kembali.

  

---

  

## 14. Aturan Anti-Spoiler

  

> **ANTI-SPOILER**

  

- Jangan membuka file 99 sebelum latihan selesai.

- Jangan mencari jawaban sampel sebelum investigasi.

- Guided Lab memberi langkah, bukan jawaban.

- Drill tidak memberi langkah detail.

- Combat Card tidak berisi jawaban latihan.

- Setelah evaluasi, tutup kembali Answer Key.

- Catat alasan jawaban salah, bukan hanya jawaban benar.

  

---

  

## 15. Aturan Evidence dan Anti-Fabrikasi

  

> **PERINGATAN**

  

- Jangan mengarang event.

- Jangan mengarang timestamp.

- Jangan mengarang hash.

- Jangan mengarang user atau IP.

- Jangan mengarang hasil tool.

- Jangan mengarang screenshot.

- Jangan mengarang flag.

- Jangan mengklaim beberapa sampel terpisah sebagai satu insiden nyata.

- Bedakan fakta, indikasi, dugaan, dan kesimpulan.

  

**Format klasifikasi temuan:**

  

- **FAKTA** — Data yang terlihat langsung pada evidence.

- **INDIKASI** — Pola yang patut diperiksa.

- **DUGAAN** — Hipotesis yang belum terverifikasi.

- **KESIMPULAN** — Pernyataan yang sudah didukung beberapa bukti.

  

---

  

## 16. Cara Menggunakan Materi Saat Lomba

  

Urutan file yang dibuka saat mengerjakan challenge nyata:

  

1. [Combat Card](06_H1_COMBAT_CARD.md)

2. [Write-up and Worksheet](07_H1_WRITEUP_AND_WORKSHEET.md)

3. [Tool Execution](03_H1_TOOL_EXECUTION.md)

4. [Windows Event Log](02_H1_WINDOWS_EVENT_LOG.md)

5. [Investigation Foundation](01_H1_INVESTIGATION_FOUNDATION.md) — jika perlu memahami ulang konsep.

  

**Jangan membuka** saat sedang mengerjakan challenge nyata:

  

- [Guided Lab](04_H1_GUIDED_LAB.md)

- [Competition Drill](05_H1_COMPETITION_DRILL.md)

- `99_H1_ANSWER_KEY.md`

  

**Workflow kompetisi:**

  

```

BACA OBJECTIVE

→ CATAT FORMAT JAWABAN

→ IDENTIFIKASI EVIDENCE

→ ATUR TIME RANGE

→ TRIAGE

→ FILTER

→ PIVOT

→ CORRELATE

→ SCREENSHOT

→ CATAT FINDINGS

→ SUBMIT

→ LANJUTKAN WRITE-UP

```

  

> **COMBAT RULE:** Simpan bukti sebelum pindah ke soal lain.

  

---

  

## 17. Emergency Quick Start

  

### Jika tidak tahu mulai dari mana

  

1. Baca ulang objective.

2. Tentukan user, IP, host, process, atau waktu yang diminta.

3. Identifikasi jenis evidence.

4. Atur time range.

5. Cari Event ID atau keyword kasar.

6. Buka satu event.

7. Lihat field aktual.

8. Pivot dari field tersebut.

9. Korelasikan dengan event lain.

10. Simpan bukti.

  

### Jika query tidak menghasilkan data

  

**TROUBLESHOOTING**

  

1. Periksa time range.

2. Periksa timezone.

3. Kurangi filter.

4. Cari hanya Event ID.

5. Cari keyword tanpa field.

6. Buka raw event.

7. Periksa nama field aktual.

8. Pastikan index atau log source benar.

9. Pastikan evidence sudah diproses.

10. Catat kegagalan dan pindah sementara jika waktu habis.

  

### Jika lupa fungsi tool

  

- **Event Viewer** — verifikasi satu EVTX.

- **EvtxECmd** — EVTX menjadi CSV.

- **Timeline Explorer** — filter dan timeline CSV.

- **CyberChef** — decode.

- **PowerShell/CMD** — export, hash, dan otomasi.

  

---

  

## 18. Checklist Mulai Sesi

  

- [ ] Tujuan sesi ditentukan.

- [ ] File materi yang sesuai dibuka.

- [ ] Tool yang diperlukan siap.

- [ ] Evidence berada di WORKING.

- [ ] Timezone dicatat.

- [ ] Worksheet dibuat.

- [ ] Folder OUTPUT siap.

- [ ] Answer Key tertutup.

- [ ] Timer latihan aktif jika sedang drill.

  

---

  

## 19. Checklist Akhir Sesi

  

- [ ] Evidence ORIGINAL tetap utuh.

- [ ] Output tersimpan.

- [ ] Screenshot diberi nama.

- [ ] Findings diperbarui.

- [ ] Write-up diperbarui.

- [ ] Kesalahan dicatat.

- [ ] Level penguasaan dinilai.

- [ ] Pertanyaan yang belum dipahami dicatat.

- [ ] File Answer Key ditutup.

- [ ] Folder kasus dirapikan.

  

---

  

## 20. Progress Tracker

  

| Bagian | Belum | Teori | Dengan panduan | Dengan cheatsheet | Mandiri | Siap lomba |

|---|---:|---:|---:|---:|---:|---:|

| Investigation workflow | | | | | | |

| Evidence handling | | | | | | |

| Event anatomy | | | | | | |

| Subject vs Target | | | | | | |

| Logon Type | | | | | | |

| Authentication events | | | | | | |

| Account changes | | | | | | |

| Process and PowerShell | | | | | | |

| Persistence | | | | | | |

| Log clearing | | | | | | |

| Timeline correlation | | | | | | |

| Event Viewer | | | | | | |

| EvtxECmd | | | | | | |

| Timeline Explorer | | | | | | |

| SIEM logic | | | | | | |

| Findings and write-up | | | | | | |

  

Isi setiap sel dengan tanda ✔ atau tanggal saat bagian tersebut dikuasai pada level itu.

  

---

  

## 21. Kriteria H1 Selesai

  

H1 dinyatakan selesai hanya jika:

  

- [ ] Semua tools utama dapat digunakan.

- [ ] Materi fondasi telah dibaca.

- [ ] Materi Event Log telah dipahami.

- [ ] Guided Lab selesai.

- [ ] Competition Drill selesai.

- [ ] Minimal satu timeline dibuat.

- [ ] Minimal satu write-up dibuat.

- [ ] Combat Card diuji.

- [ ] Informasi Combat Card dapat ditemukan dalam maksimal sekitar 30 detik.

- [ ] Kesalahan latihan sudah dievaluasi.

- [ ] Pengguna minimal mencapai level 3 pada mayoritas kompetensi.

- [ ] Authentication dan Event Log filtering minimal mencapai level 4.

  

---

  

## 22. Navigasi Berikutnya

  

Langkah setelah README ini:

  

Buka: [Investigation Foundation](01_H1_INVESTIGATION_FOUNDATION.md)

  

Tujuan: membangun cara berpikir investigasi sebelum mempelajari Event ID dan penggunaan tool.

  

Jangan melompat langsung ke Guided Lab kecuali fondasi dan tool sudah siap.

  

---

  

## Kontrak Global (dipertahankan dari Tahap 1)

  

Bagian ini merangkum kontrak yang telah dikunci pada Tahap 1 dan berlaku untuk seluruh file H1.

  

- **Prinsip GUI-first** — Event Viewer dan Timeline Explorer adalah jalur analisis utama; CMD dan PowerShell adalah jalur cadangan.

- **Batas cakupan** — sesuai bagian 3 di atas; topik di luar cakupan dipindahkan ke modul hari berikutnya.

- **Workflow evidence** — sesuai bagian 7 di atas: INBOX → ORIGINAL → hash → WORKING → analisis → OUTPUT.

- **Prioritas sumber** — evidence aktual dan hasil tool langsung selalu diutamakan di atas asumsi atau ingatan pribadi; setiap klaim harus dapat ditelusuri kembali ke field atau output yang benar-benar terlihat.

- **Naming convention** — sesuai bagian 13 di atas; wajib konsisten di seluruh file dan output.

- **Sinkronisasi** — istilah, nama file, dan struktur folder yang dipakai pada file 01–07 dan 99 harus konsisten dengan yang tercantum pada README ini.

- **Anti-spoiler** — sesuai bagian 14 di atas.

- **Anti-fabrikasi** — sesuai bagian 15 di atas.

- **Competition-ready assumptions** — materi disusun dengan asumsi kondisi lomba: waktu terbatas, tanpa bantuan AI, dan hanya mengandalkan Combat Card serta worksheet.

- **Audit Tahap 1** — struktur proyek, kontrak global, workflow evidence, naming convention, sinkronisasi, aturan sumber, dan anti-spoiler dinyatakan telah dikunci pada Tahap 1.

  

---

  

## Audit Tahap 2

  

- [x] README menjadi pusat navigasi

- [x] Tujuan H1 dijelaskan

- [x] Batas cakupan dijelaskan

- [x] Tools dan fungsi dijelaskan

- [x] Tool readiness checklist tersedia

- [x] Struktur folder dijelaskan

- [x] Urutan belajar tersedia

- [x] Estimasi waktu tersedia

- [x] Metode belajar tersedia

- [x] Level penguasaan tersedia

- [x] Output wajib tersedia

- [x] Naming convention tersedia

- [x] Aturan anti-spoiler tersedia

- [x] Aturan anti-fabrikasi tersedia

- [x] Workflow lomba tersedia

- [x] Emergency Quick Start tersedia

- [x] Checklist mulai dan akhir sesi tersedia

- [x] Progress tracker tersedia

- [x] Kriteria H1 selesai tersedia

- [x] Kontrak global Tahap 1 tetap dipertahankan

- [x] Materi teknis file 01–07 belum ditulis

- [x] Tahap 3 belum dikerjakan

  

**Status akhir:**

  

```

TAHAP 2: SELESAI

README: FINAL

NAVIGASI: SIAP

PROGRESS TRACKER: SIAP

ANTI-SPOILER: AKTIF

LAYAK LANJUT KE TAHAP 3

```