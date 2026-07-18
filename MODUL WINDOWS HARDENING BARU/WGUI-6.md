# WGUI-6 — WINDOWS LOGGING, AUDITING, MONITORING & INCIDENT RESPONSE

  

## Windows GUI Hardening Learning System

### Modul Persiapan Kompetisi Cyber Security LKS Tingkat Nasional — Blue Team Track

  

> Modul ini adalah kelanjutan dan penutup dari WGUI-0 (Windows Hardening Foundation), WGUI-1 (Local Account, PAM, Local Security Policy), WGUI-2 (Group Policy Hardening), WGUI-3 (Active Directory Hardening), WGUI-4 (Windows Network & Firewall Hardening), dan WGUI-5 (Microsoft Defender & Attack Surface Reduction).

  

> Catatan versi: nama menu, path GUI, ketersediaan channel, dan detail field event dapat berbeda menurut versi Windows Server, edisi Server Core atau Desktop Experience, role server (Domain Controller atau member server), audit policy yang aktif, serta provider/channel yang tersedia. Semua langkah pada modul ini harus diverifikasi ulang pada mesin target sebelum dianggap berlaku.

  

---

  

## WGUI-6.0 Posisi dan Tujuan Modul

  

WGUI-6 adalah modul penutup seri Windows Hardening. Modul sebelumnya berfokus pada **mengonfigurasi** kontrol keamanan. Modul ini mengubah hasil konfigurasi tersebut menjadi sistem yang dapat:

  

- Merekam aktivitas yang terjadi pada sistem.

- Membuktikan bahwa suatu perubahan benar-benar terjadi.

- Mendeteksi penyimpangan dari kondisi baseline.

- Menyelidiki insiden melalui korelasi event.

- Menjaga evidence agar tetap dapat dipercaya.

- Melakukan response yang aman dan dapat di-rollback.

  

Perbedaan istilah kunci:

  

| Konsep | Fokus |

|---|---|

| Logging | Merekam kejadian |

| Auditing | Menentukan aktivitas apa yang harus dicatat |

| Monitoring | Mencari dan meninjau kejadian penting |

| Investigation | Menghubungkan beberapa event menjadi timeline |

| Incident Response | Mengendalikan dampak dan memulihkan kondisi |

| Evidence Preservation | Menjaga bukti tetap dapat dipercaya |

  

Modul ini murni defensif. Tidak ada teknik serangan, bypass, atau anti-forensics yang diajarkan di sini.

  

---

  

## WGUI-6.1 Mental Model Event

  

Setiap event Windows terdiri dari elemen-elemen berikut, yang dapat dilihat pada Event Viewer:

  

- Log Name

- Provider

- Event ID

- Level

- Time Created

- Computer

- User / Security ID

- Logon ID

- Process ID

- Source IP (bila relevan, tersedia pada beberapa event tertentu)

- Workstation Name (bila relevan)

- EventRecordID

- General tab

- Details/XML tab

  

Formula konteks event:

  

```text

WHO

 +

WHAT

 +

WHEN

 +

WHERE

 +

HOW

 =

EVENT CONTEXT

```

  

> Event ID tanpa field context (user, waktu, komputer, proses) tidak cukup untuk mengambil kesimpulan investigasi. Selalu baca tab Details/XML, bukan hanya General.

  

---

  

## WGUI-6.2 GUI Tools dan Availability Check

  

| Tool | Fungsi Utama | Keterbatasan |

|---|---|---|

| `eventvwr.msc` (Event Viewer) | Melihat, memfilter, menyimpan, dan membuat Custom View dari log | Membutuhkan hak akses administratif yang sesuai untuk Security log |

| `secpol.msc` (Local Security Policy) | Mengonfigurasi Basic Audit Policy dan Advanced Audit Policy lokal | Tidak tersedia pada Server Core; hanya menampilkan configured policy, bukan effective policy |

| `gpedit.msc` (Local Group Policy Editor) | Mengonfigurasi kebijakan lokal termasuk audit dan logging | Dapat ditimpa oleh GPO Domain jika server berada dalam domain |

| `gpmc.msc` (Group Policy Management Console) | Mengelola GPO pada level domain | Hanya relevan jika server berada dalam domain |

| Group Policy Results / `rsop.msc` | Menampilkan effective policy hasil gabungan local dan domain GPO | Tidak selalu menampilkan seluruh detail audit subcategory secara langsung |

| Task Scheduler | Melihat dan mengelola scheduled task, termasuk Operational log | Tidak semua task history aktif secara default |

| Services (`services.msc`) | Melihat status, startup type, dan account service | Tidak menampilkan riwayat perubahan; riwayat harus dicari di Event Viewer |

| Windows Security | Melihat status Microsoft Defender Antivirus, menjalankan scan, membaca Protection History, melihat Controlled Folder Access, dan Exploit Protection jika tersedia | Tidak digunakan untuk mengonfigurasi daftar individual Attack Surface Reduction rules; detail event tetap harus dicek di Event Viewer |

| Windows Defender Firewall with Advanced Security | Mengonfigurasi rule dan firewall logging | Log allowed/dropped connection berada di file `pfirewall.log` atau Security log tergantung konfigurasi |

  

Catatan penting:

  

- Server Core tidak memiliki seluruh GUI di atas; sebagian tugas mungkin harus dilakukan lewat CLI verifikasi terbatas.

- Local policy dapat ditimpa oleh Domain GPO kapan saja.

- Security log membutuhkan hak administratif yang sesuai untuk dibaca.

- Tidak semua Applications and Services Logs aktif secara default.

- Jangan mengasumsikan Sysmon, SIEM eksternal, Microsoft Defender for Endpoint, atau Windows Event Collector sudah tersedia di lingkungan lomba kecuali dinyatakan eksplisit oleh panitia.

- Windows Security tidak digunakan untuk mengonfigurasi daftar individual Attack Surface Reduction rules. Individual ASR rules dikonfigurasi melalui `gpedit.msc`, `gpmc.msc`, PowerShell bila diperlukan, atau management platform yang memang tersedia. Konfigurasi ASR mengikuti WGUI-5 dan tidak diulang secara rinci pada modul ini.

  

---

  

## WGUI-6.3 Logging Baseline

  

Checklist yang harus dicatat di awal, sebelum melakukan perubahan apa pun:

  

- [ ] Versi Windows Server

- [ ] Server Core atau Desktop Experience

- [ ] Role server: Domain Controller, member server, file server, application server, atau lainnya

- [ ] Nama host

- [ ] Domain atau workgroup

- [ ] Time zone

- [ ] Waktu sistem saat ini

- [ ] Sumber sinkronisasi waktu

- [ ] Ketersediaan log Security

- [ ] Ketersediaan log System

- [ ] Ketersediaan log Application

- [ ] Ketersediaan Applications and Services Logs yang relevan

- [ ] Maximum log size tiap log

- [ ] Retention method tiap log

- [ ] Status Advanced Audit Policy

- [ ] Sumber policy: local atau GPO

- [ ] Status PowerShell logging

- [ ] Status Defender Operational log

- [ ] Status Task Scheduler Operational log

- [ ] Status firewall/WFP auditing

- [ ] Kapasitas disk yang tersedia

- [ ] Path penyimpanan recovery dan evidence

  

> Waktu sistem dan zona waktu wajib dicatat sebelum membuat timeline apa pun. Timeline yang dibangun tanpa referensi waktu yang konsisten dapat menyesatkan investigasi.

  

---

  

## WGUI-6.4 Arsitektur Windows Event Logs

  

**Windows Logs:**

  

- Application

- Security

- Setup

- System

- Forwarded Events

  

**Applications and Services Logs** (minimal yang relevan untuk modul ini):

  

- Microsoft → Windows → Windows Defender → Operational

- Microsoft → Windows → PowerShell → Operational

- Microsoft → Windows → TaskScheduler → Operational

- Microsoft → Windows → GroupPolicy → Operational

- Microsoft → Windows → TerminalServices (channel terkait), jika tersedia

- Microsoft → Windows → Windows Firewall atau provider terkait, jika tersedia

  

**Tingkatan channel:**

  

| Tingkatan | Karakteristik |

|---|---|

| Administrative | Ditujukan untuk end user/administrator, pesan mudah dipahami |

| Operational | Digunakan untuk troubleshooting, lebih rinci dari Administrative |

| Analytic | Volume tinggi, biasanya nonaktif secara default |

| Debug | Volume sangat tinggi, untuk keperluan debugging developer |

  

> Jangan mengaktifkan channel Analytic atau Debug tanpa objective yang jelas. Channel ini dapat meningkatkan volume log dan overhead sistem secara signifikan.

  

---

  

## WGUI-6.5 Basic vs Advanced Audit Policy

  

- Basic Audit Policy berada di `Local Security Policy → Security Settings → Local Policies → Audit Policy`.

- Advanced Audit Policy berada di `Local Security Policy → Security Settings → Advanced Audit Policy Configuration`, menggunakan subcategory yang lebih presisi.

- **Jangan mengonfigurasi Basic dan Advanced Audit Policy secara bersamaan tanpa perhitungan**, karena keduanya dapat menghasilkan effective policy yang tidak terduga.

- Jika Advanced Audit Policy digunakan, aktifkan setting berikut agar subcategory tidak ditimpa kategori dasar. **Setting ini bukan berada di dalam Advanced Audit Policy Configuration**, melainkan pada Security Options:

  

```text

Computer Configuration

→ Windows Settings

→ Security Settings

→ Local Policies

→ Security Options

→ Audit: Force audit policy subcategory settings

   (Windows Vista or later) to override

   audit policy category settings

```

  

Sedangkan subcategory Advanced Audit Policy tetap berada pada jalur terpisah:

  

```text

Computer Configuration

→ Windows Settings

→ Security Settings

→ Advanced Audit Policy Configuration

→ Audit Policies

```

  

Gunakan formula:

  

```text

SECURITY OPTIONS

      +

ADVANCED AUDIT SUBCATEGORY

      =

CONSISTENT ADVANCED AUDIT POLICY

```

  

> Jangan mengaktifkan setting override tanpa memahami apakah environment masih menggunakan Basic Audit Policy.

  

- Periksa effective policy melalui Group Policy Results dan `auditpol /get /category:*` sebagai verifikasi tambahan.

- **Jangan menganggap tampilan `secpol.msc` sebagai bukti effective policy** — tampilan tersebut hanya menunjukkan configured policy pada level yang sedang dilihat, bukan hasil akhir setelah digabung dengan GPO domain.

  

```text

CONFIGURED AUDIT POLICY

          ≠

EFFECTIVE AUDIT POLICY

```

  

---

  

## WGUI-6.6 Advanced Audit Policy Categories

  

| Kategori | Aktivitas yang Dicatat | Role Relevan | Success/Failure | Volume | Risiko Bila Tidak Dicatat |

|---|---|---|---|---|---|

| Account Logon | Validasi kredensial, Kerberos ticket | DC terutama | Keduanya | Medium–High pada DC | Tidak dapat melacak asal autentikasi |

| Account Management | Perubahan user/group/computer account | Semua role | Success minimal, Failure bila relevan | Low–Medium | Perubahan akun tidak terlacak |

| Detailed Tracking | Process creation, process termination | Semua role | Success | Medium–High | Tidak ada visibilitas eksekusi proses |

| DS Access | Akses objek Active Directory | DC | Sesuai objective | Medium–High | Perubahan objek AD tidak terlacak |

| Logon/Logoff | Logon, logoff, lock/unlock, koneksi RDP | Semua role | Keduanya | Medium | Aktivitas sesi tidak terlacak |

| Object Access | Akses file, folder, registry, share | Sesuai SACL | Sesuai objective | Rendah jika SACL sempit, tinggi jika luas | Akses objek sensitif tidak terlacak |

| Policy Change | Perubahan audit policy dan security policy | Semua role | Success | Low | Perubahan policy tidak terdeteksi |

| Privilege Use | Penggunaan privilege khusus | Semua role | Sesuai objective | Medium–High jika Success diaktifkan luas | Penyalahgunaan privilege tidak terlacak |

| System | Startup, shutdown, integrity system | Semua role | Success | Low | Perubahan status sistem tidak terlacak |

| Global Object Access Auditing | SACL global tanpa konfigurasi per objek | Sesuai objective | Sesuai objective | Berpotensi sangat tinggi | Tidak ada baseline akses objek luas |

  

Panduan verifikasi, rollback, dan evidence berlaku sama untuk semua kategori:

  

- **Verification**: cek melalui Group Policy Results dan `auditpol /get` setelah konfigurasi diterapkan.

- **Rollback**: kembalikan subcategory ke kondisi semula melalui GUI yang sama, bukan dengan menghapus log.

- **Evidence**: dokumentasikan subcategory yang diubah, waktu perubahan, dan siapa yang melakukan (dapat dikorelasikan dengan Event 4719 pada WGUI-6.12).

  

> Jangan mengaktifkan seluruh subcategory Success dan Failure sekaligus tanpa mempertimbangkan volume log dan kapasitas penyimpanan.

  

---

  

## WGUI-6.7 Authentication dan Logon Monitoring

  

| Event ID | Makna Utama |

|---|---|

| 4624 | Successful logon |

| 4625 | Failed logon |

| 4634 | Logoff selesai |

| 4647 | User initiated logoff |

| 4648 | Logon menggunakan explicit credentials |

| 4672 | Special privileges diberikan pada logon |

| 4740 | Account locked out |

| 4767 | Account unlocked |

| 4776 | Credential validation menggunakan NTLM |

| 4768 | Kerberos TGT diminta (terutama pada Domain Controller) |

| 4769 | Kerberos service ticket diminta (terutama pada Domain Controller) |

| 4771 | Kerberos pre-authentication failed (terutama pada Domain Controller) |

  

**Logon Type penting:**

  

| Logon Type | Makna |

|---|---|

| 2 | Interactive |

| 3 | Network |

| 5 | Service |

| 7 | Unlock |

| 10 | RemoteInteractive/RDP |

| 11 | CachedInteractive |

  

Prinsip analisis:

  

- 4624 bukan otomatis berarti aktivitas aman.

- 4625 tunggal bukan otomatis berarti serangan.

- Banyak 4625 dari sumber yang sama dapat berasal dari brute force, service dengan password lama, scheduled task, mapped drive, atau kesalahan konfigurasi — bukan hanya serangan.

- Event Kerberos tertentu (4768, 4769, 4771) muncul pada Domain Controller, bukan selalu pada komputer target investigasi.

  

```text

4672

≠

OTOMATIS PRIVILEGE ESCALATION

```

  

Event 4672 menunjukkan special privileges diberikan pada logon. Event ini sering muncul untuk account bawaan seperti `SYSTEM`, `LOCAL SERVICE`, dan `NETWORK SERVICE`, serta pada administrator sah. Event ini harus dikorelasikan dengan Event 4624 melalui Logon ID, diperiksa privilege yang benar-benar tercantum, diperiksa apakah account termasuk authorized administrator, dan diperiksa apakah sebelumnya ada perubahan membership group.

  

```text

4672 FOUND

   ↓

CHECK ACCOUNT

   ↓

CHECK LOGON ID

   ↓

CORRELATE 4624

   ↓

CHECK PRIVILEGE LIST

   ↓

CHECK ADMIN BASELINE

   ↓

CHECK GROUP CHANGE EVENTS

```

  

> Event 4672 merupakan indikator privilege-bearing logon. Status sah atau tidak sah ditentukan melalui account identity, Logon ID, privilege list, baseline administrator, dan perubahan group sebelumnya.

  

---

  

## WGUI-6.8 Account dan Group Change Monitoring

  

| Event ID | Aktivitas |

|---|---|

| 4720 | User account dibuat |

| 4722 | User account diaktifkan |

| 4725 | User account dinonaktifkan |

| 4726 | User account dihapus |

| 4723 | Percobaan mengubah password |

| 4724 | Percobaan reset password |

| 4738 | User account diubah |

| 4728/4729 | Member ditambah/dihapus dari global security group |

| 4732/4733 | Member ditambah/dihapus dari local security group |

| 4756/4757 | Member ditambah/dihapus dari universal security group |

  

Alur korelasi:

  

```text

ACCOUNT CHANGE EVENT

        ↓

SUBJECT ACCOUNT

        ↓

TARGET ACCOUNT/GROUP

        ↓

TIME

        ↓

AUTHORIZED CHANGE LIST

        ↓

VERIFY CURRENT STATE

```

  

> Event menunjukkan bahwa perubahan terjadi. Authorized change list (daftar perubahan yang sah) yang menentukan apakah perubahan tersebut sah atau tidak — bukan event itu sendiri.

  

---

  

## WGUI-6.9 Process Creation Monitoring

  

| Event ID | Makna |

|---|---|

| 4688 | A new process has been created |

| 4689 | A process has exited (bila audit terkait diaktifkan) |

  

Agar command line muncul pada Event 4688, dua policy dibutuhkan:

  

1. Audit Process Creation (Advanced Audit Policy → Detailed Tracking).

2. `Include command line in process creation events` (Administrative Templates).

  

> Command line pada 4688 dicatat sebagai teks, sehingga dapat memuat informasi sensitif seperti password, token, atau connection string bila dimasukkan langsung sebagai argument. Perlakukan log ini sebagai data sensitif dan batasi akses.

  

Alur korelasi:

  

```text

USER

 ↓

LOGON ID

 ↓

PARENT PROCESS

 ↓

NEW PROCESS

 ↓

COMMAND LINE

 ↓

DESTINATION/FILE

```

  

Konversi Process ID dari hexadecimal ke decimal cukup dipahami sebagai awareness saja, bukan langkah wajib pada jalur GUI.

  

---

  

## WGUI-6.10 Services dan Scheduled Tasks

  

**Services:**

  

| Event ID | Makna |

|---|---|

| 4697 | Service installed (bila Security auditing relevan aktif) |

| 7045 | Service installed (System log) |

| 7040 | Service startup type berubah |

| 7036 | Service state berubah |

| 7000 | Service gagal start |

  

**Scheduled Tasks:**

  

| Event ID | Makna |

|---|---|

| 4698 | Scheduled task dibuat |

| 4702 | Scheduled task diperbarui |

| 4699 | Scheduled task dihapus |

  

TaskScheduler Operational log dapat digunakan sebagai supporting evidence tambahan.

  

Prinsip analisis:

  

- Service atau scheduled task baru tidak otomatis bersifat malicious.

- Bandingkan nama, path executable, account yang menjalankan, trigger, action, author, dan waktu pembuatan dengan authorized baseline.

- Path pada direktori yang dapat ditulis oleh user biasa (user-writable directory) memiliki risiko lebih tinggi.

- Jangan menghapus task atau service sebelum dependency dan objective diperiksa terlebih dahulu.

  

---

  

## WGUI-6.11 Object Access Auditing

  

File/folder auditing membutuhkan dua lapisan sekaligus:

  

```text

AUDIT POLICY

      +

OBJECT SACL

      =

OBJECT ACCESS EVENT

```

  

Subcategory relevan:

  

- Audit File System

- Audit Registry

- Audit File Share

- Audit Detailed File Share

  

SACL dikonfigurasi melalui `Properties → Security → Advanced → Auditing` pada objek yang dituju.

  

Prinsip:

  

- Mengaktifkan Audit Object Access saja tidak cukup tanpa SACL pada objek.

- Tanpa SACL, event akses pada objek tertentu tidak akan muncul meskipun subcategory sudah aktif.

- SACL yang terlalu luas dapat menghasilkan log flooding.

- Gunakan folder uji atau objek kritis yang jelas ruang lingkupnya.

- Audit hanya user/action yang benar-benar dibutuhkan sesuai objective.

  

---

  

## WGUI-6.12 Policy, Log Integrity, dan Time Change

  

| Event ID | Makna |

|---|---|

| 4719 | System audit policy changed |

| 1102 | Security audit log cleared |

| 1104 | Security log full |

| 1105 | Automatic backup of Security log |

| 4616 | System time changed |

  

> Event 1102 adalah temuan bernilai tinggi. Selalu periksa bersama Subject Account, waktu kejadian, Logon ID, serta aktivitas sebelum dan sesudah event tersebut.

  

**Jangan langsung menyimpulkan bahwa 1102 berasal dari attacker.** Administrator sah juga dapat membersihkan log sebagai bagian dari maintenance, tetapi tindakan tersebut tetap harus memiliki alasan dan dokumentasi yang jelas.

  

---

  

## WGUI-6.13 PowerShell Logging

  

Tiga lapisan visibilitas:

  

**Module Logging**

  

- Mencatat operasi module/cmdlet tertentu.

- Event 4103 pada `Microsoft-Windows-PowerShell/Operational`.

  

**Script Block Logging**

  

- Mencatat isi script block yang diproses.

- Event 4104.

- Script yang panjang dapat terpecah menjadi beberapa event.

- Field yang tersedia pada Event 4104 dapat berbeda berdasarkan versi PowerShell, provider, versi Windows Server, jenis host PowerShell, dan konfigurasi logging. **Jangan menjamin bahwa Logon ID selalu tersedia pada Event 4104.**

- Korelasi Event 4104 menggunakan timestamp, hostname, user atau SID jika tersedia, process ID atau process context jika tersedia, ScriptBlock ID, path jika tersedia, Event 4103, Event 4688, dan logon events pada waktu berdekatan.

  

```text

4104

  +

TIME

  +

HOST

  +

USER/SID IF AVAILABLE

  +

PROCESS CONTEXT

  +

4103 / 4688 / LOGON EVENTS

  =

POWERSHELL CONTEXT

```

  

**PowerShell Transcription**

  

- Mencatat input dan output sesi ke file teks.

- Bukan pengganti Event Viewer.

- Folder transcript harus dibatasi ACL-nya.

- Pertimbangkan ukuran disk dan retention.

- Transcript dapat mengandung data sensitif dan harus diperlakukan sebagai data terbatas.

  

```text

POWERSHELL VISIBILITY =

MODULE LOGGING

+ SCRIPT BLOCK LOGGING

+ SECURE TRANSCRIPTION

```

  

> Modul ini tidak membahas AMSI bypass, obfuscation, atau teknik apa pun untuk menghindari logging PowerShell.

  

---

  

## WGUI-6.14 Defender Event Correlation

  

Ringkasan lanjutan dari WGUI-5:

  

| Event ID | Makna |

|---|---|

| 1116 | Threat detected |

| 1117 | Defender action/remediation |

| 1119 | Remediation failure |

| 5007 | Defender configuration changed |

| 1121 | ASR Block |

| 1122 | ASR Audit |

| 1123 | CFA Block |

| 1124 | CFA Audit |

  

> Event 1117 harus dibaca bersama detail action dan result-nya, karena event ini tidak selalu berarti objek berhasil dibersihkan atau dihapus.

  

Alur korelasi:

  

```text

1116 DETECTION

      ↓

1117 ACTION

      ↓

1119 IF FAILURE

      ↓

SCAN RESULT

      ↓

CURRENT FILE/PROCESS STATE

```

  

---

  

## WGUI-6.15 Firewall dan Windows Filtering Platform Events

  

**`pfirewall.log`**

  

- File teks, digunakan untuk logging allowed/dropped connection.

- Dibahas lebih rinci pada WGUI-4.

  

**Security Log — Windows Filtering Platform**

  

| Event ID | Makna |

|---|---|

| 5152 | Packet blocked |

| 5157 | Connection blocked |

| 5156 | Connection allowed (berpotensi volume sangat tinggi) |

  

Prinsip:

  

- Jangan mengaktifkan audit allowed connection secara luas tanpa mengukur volume terlebih dahulu.

- Event firewall harus dikorelasikan dengan source IP, destination IP, protocol, port, process/application, dan active firewall profile.

- Tidak semua blocked connection merupakan indikasi serangan.

  

---

  

## WGUI-6.16 Event Filtering dan Custom Views

  

Langkah GUI:

  

1. Buka Event Viewer.

2. Pilih log yang relevan.

3. Klik **Filter Current Log**.

4. Pilih rentang waktu, level, source, dan Event ID.

5. Baca event melalui tab General dan Details/XML.

6. Buat **Custom View** untuk filter yang berulang digunakan.

7. Beri nama Custom View yang menjelaskan tujuannya secara jelas.

  

Contoh Custom Views yang berguna:

  

- Authentication Failures

- Privileged Logons

- Account Changes

- Process Creation

- Service and Task Changes

- Defender Detections

- Audit Policy and Log Integrity

- PowerShell Activity

  

> Filter hanya menyembunyikan event yang tidak cocok dengan kriteria. Filter tidak menghapus event apa pun dari log.

  

**Jenis penyimpanan event:**

  

| Aksi | Fungsi |

|---|---|

| Save All Events As | Menyimpan seluruh isi sebuah log, bukan hanya event yang sedang tampil setelah filter |

| Save Filtered Log File As | Menyimpan event yang sesuai dengan filter aktif pada log |

| Save All Events in Custom View As | Menyimpan event yang termasuk dalam Custom View |

  

> Filter Current Log hanya mengubah tampilan. Save All Events As tetap dapat menyimpan seluruh log, bukan hanya hasil filter.

  

---

  

## WGUI-6.17 Log Size, Retention, dan Availability

  

Diatur melalui:

  

```text

Event Viewer

→ klik kanan pada log

→ Properties

```

  

Opsi retention:

  

- Overwrite events as needed

- Archive the log when full

- Do not overwrite events

  

Risiko masing-masing:

  

- Log yang terlalu kecil menyebabkan event lama cepat tertimpa.

- **Do not overwrite** dapat menyebabkan log penuh dan logging berhenti mencatat.

- **Archive the log when full** membutuhkan kapasitas disk dan pengelolaan file archive.

- Security log yang penuh dapat menghasilkan Event 1104.

- Automatic archive dapat menghasilkan Event 1105.

  

> Jangan menetapkan satu angka ukuran log sebagai standar universal. Nilai yang tepat bergantung pada objective lomba, volume event, role server, kapasitas disk, dan durasi evidence yang dibutuhkan.

  

---

  

## WGUI-6.18 Evidence Preservation

  

Strategi evidence:

  

```text

FULL LOG EXPORT

      =

ORIGINAL EVIDENCE

  

FILTERED EXPORT

      =

WORKING SET

```

  

Alur:

  

```text

EXPORT FULL LOG

      ↓

HASH ORIGINAL FULL LOG

      ↓

CREATE WORKING COPY

      ↓

EXPORT FILTERED EVENTS

      ↓

ANALYZE FILTERED WORKING SET

```

  

Prosedur:

  

1. Catat hostname, waktu sistem, zona waktu, dan nama log.

2. Export seluruh log menggunakan **Save All Events As** jika ruang dan objective memungkinkan.

3. Simpan file full log sebagai original evidence.

4. Hitung hash SHA-256 pada original evidence.

5. Terapkan filter atau Custom View.

6. Export event terpilih sebagai working set menggunakan **Save Filtered Log File As** atau **Save All Events in Custom View As**.

7. Analisis working copy atau filtered export.

8. Jangan mengubah original evidence.

  

> Filter Current Log hanya mengubah tampilan. Save All Events As tetap dapat menyimpan seluruh log, bukan hanya hasil filter.

  

Format nama evidence:

  

```text

YYYYMMDD-HHMM_HOST_LOGNAME_CASE.evtx

```

  

Contoh pemisahan file:

  

```text

20260718-1030-SRV01-Security-CASE01-FULL.evtx

20260718-1030-SRV01-Security-CASE01-FILTERED.evtx

```

  

- `FULL` adalah export seluruh log.

- `FILTERED` adalah working set.

  

Hash untuk verifikasi integritas:

  

```text

Get-FileHash <file> -Algorithm SHA256

```

  

- Hash dicatat minimal untuk file `FULL`.

- Jika working copy identik byte-for-byte dengan original, hash harus sama.

- Jika working copy sudah difilter, disimpan ulang, atau diubah, hash akan berbeda dan itu normal — bukan indikasi kerusakan evidence.

- Hash masing-masing file dapat dicatat secara terpisah.

  

> Hash hanya digunakan sebagai verifikasi integritas evidence, bukan sebagai pengganti prosedur chain of custody.

  

Evidence minimal yang harus didokumentasikan:

  

- Hostname

- Waktu sistem dan zona waktu

- Nama log

- Rentang waktu yang diperiksa

- Filter/Event ID yang digunakan

- File `.evtx` FULL (original evidence)

- File `.evtx` FILTERED (working set)

- Screenshot event yang relevan

- Nilai SHA-256 masing-masing file

- Catatan siapa yang mengambil evidence

- Working copy yang terpisah dari file asli

  

Chain of custody sederhana minimal mencatat:

  

- Case ID

- Hostname

- Collector

- Waktu pengambilan

- Time zone

- File name

- File type

- Hash

- Storage location

- Tindakan yang dilakukan terhadap file

  

---

  

## WGUI-6.19 Timeline dan Event Correlation

  

Tabel kerja timeline:

  

| Time | Host | Log | Event ID | User | Source IP | Process/Object | Interpretation | Confidence |

|---|---|---|---|---|---|---|---|---|

  

Alur urutan investigasi:

  

```text

FIRST KNOWN EVENT

       ↓

EVENTS BEFORE

       ↓

TRIGGER EVENT

       ↓

FOLLOW-UP ACTIVITY

       ↓

CONFIGURATION CHANGE

       ↓

REMEDIATION

       ↓

CURRENT STATE

```

  

Dasar korelasi:

  

- Timestamp

- Computer

- User

- Logon ID

- Process ID

- Parent process

- Source IP

- Target account

- Task/service name

- File path

- Defender threat ID

- EventRecordID

  

Bedakan secara ketat empat tingkat kepastian:

  

| Tingkat | Definisi |

|---|---|

| Fact | Data yang tercatat langsung pada event |

| Observation | Pola yang teramati dari kumpulan event |

| Inference | Dugaan berdasarkan observation, belum terbukti penuh |

| Conclusion | Kesimpulan yang didukung oleh korelasi dan validasi memadai |

  

> Jangan menyatakan inference sebagai fakta dalam laporan atau write-up.

  

---

  

## WGUI-6.20 Basic Incident Triage

  

Alur triage:

  

```text

IDENTIFY ALERT

      ↓

VALIDATE EVENT

      ↓

DEFINE SCOPE

      ↓

PRESERVE EVIDENCE

      ↓

CHECK CURRENT STATE

      ↓

CONTAIN MINIMALLY

      ↓

VERIFY SERVICE

      ↓

DOCUMENT

```

  

Pertanyaan triage:

  

1. Apa yang terjadi?

2. Kapan mulai terjadi?

3. User mana yang terlibat?

4. Host mana yang terdampak?

5. Apakah aktivitas masih berlangsung?

6. Apakah account/service penting terdampak?

7. Evidence apa yang tersedia?

8. Perubahan minimal apa yang aman dilakukan?

9. Bagaimana rollback dilakukan bila diperlukan?

10. Bagaimana membuktikan insiden sudah terkendali?

  

Containment aman yang dapat dilakukan:

  

- Disable akun yang terverifikasi tidak sah.

- Remove membership group yang berlebihan.

- Disable rule firewall atau scheduled task yang terverifikasi tidak sah.

- Stop service tidak sah setelah dependency diperiksa.

- Quarantine melalui Defender.

- Batasi scope firewall.

- Reset credential sesuai objective yang ditentukan.

  

> Jangan menjadikan delete sebagai tindakan pertama dalam containment.

  

---

  

## WGUI-6.21 Troubleshooting Decision Tree

  

**Alur umum — Expected Event Not Found:**

  

```text

EXPECTED EVENT NOT FOUND

   ↓

CHECK CORRECT HOST

   ↓

CHECK TIME RANGE + TIME ZONE

   ↓

CHECK LOG/CHANNEL

   ↓

CHECK AUDIT SUBCATEGORY

   ↓

CHECK LOCAL VS DOMAIN GPO

   ↓

CHECK EFFECTIVE POLICY

   ↓

CHECK PROVIDER/CHANNEL STATUS

   ↓

CHECK LOG SIZE/RETENTION

   ↓

REPEAT SAFE TEST

```

  

**Security Log Cepat Penuh:**

  

```text

CHECK EVENT VOLUME

   ↓

IDENTIFY NOISY EVENT ID

   ↓

CHECK AUDIT SUBCATEGORY

   ↓

CHECK SACL SCOPE

   ↓

CHECK LOG MAXIMUM SIZE

   ↓

ADJUST PRECISELY

```

  

**Event 4688 Tidak Memiliki Command Line:**

  

```text

CHECK AUDIT PROCESS CREATION

   ↓

CHECK INCLUDE COMMAND LINE POLICY

   ↓

CHECK EFFECTIVE GPO

   ↓

GENERATE SAFE TEST PROCESS

   ↓

CHECK NEW EVENT 4688

```

  

**PowerShell Event Tidak Muncul:**

  

```text

CHECK POWERSHELL VERSION

   ↓

CHECK MODULE/SCRIPT BLOCK POLICY

   ↓

CHECK EFFECTIVE POLICY

   ↓

CHECK CORRECT OPERATIONAL LOG

   ↓

RUN SAFE TEST

```

  

**Object Access Event Tidak Muncul:**

  

```text

CHECK OBJECT ACCESS POLICY

   ↓

CHECK OBJECT SACL

   ↓

CHECK USER/ACTION IN SACL

   ↓

RUN SAFE ACCESS TEST

   ↓

CHECK SECURITY LOG

```

  

---

  

## WGUI-6.22 Windows Event Forwarding Awareness

  

- Windows Event Forwarding (WEF) meneruskan event dari sumber ke satu atau lebih collector.

- Windows Event Collector (WEC) menerima event yang diteruskan, ditampilkan pada log **Forwarded Events**.

- Terdapat dua model: source-initiated dan collector-initiated.

- Local event log tetap berfungsi sebagai buffer ketika proses forwarding terganggu, sehingga event tidak langsung hilang.

- Channel `Eventlog-ForwardingPlugin/Operational` dapat digunakan untuk troubleshooting status subscription.

- WEF dapat meneruskan event dari channel Operational maupun Administrative, dan memiliki channel tersendiri untuk memantau status subscription.

  

> Jangan membuat konfigurasi WEF menjadi wajib bila environment lomba hanya memiliki satu server. Bahas ini sebatas awareness konsep.

  

---

  

## BATASAN KONTEN

  

Modul ini murni defensif dan tidak memuat: log deletion, log tampering, anti-forensics, event suppression, audit bypass, PowerShell logging bypass, AMSI bypass, Defender bypass, firewall evasion, credential dumping, persistence procedure, malware, payload, reverse shell, exploit, lateral movement, atau instruksi serangan Red Team apa pun. Seluruh isi modul terbatas pada: audit, logging, monitoring, investigation, evidence preservation, containment, recovery, verification, dan reporting.

  

---

  

## CLOSED-BOOK TRAINING

  

Setiap skenario dijawab dengan format wajib:

  

- Apa yang dicek?

- Risiko?

- Event pendukung?

- Perubahan atau response aman?

- Verifikasi?

- Rollback?

- Evidence?

- Kesimpulan sementara?

  

> Jangan langsung memberi label malicious tanpa korelasi yang memadai.

  

### Skenario 1 — Banyak 4625 dari satu IP

  

- **Apa yang dicek?** Source IP/workstation, jumlah percobaan, rentang waktu, akun target yang dicoba, service/scheduled task/mapped drive yang menggunakan account terkait.

- **Risiko?** Brute-force attempt, password user yang salah, service menggunakan password lama, scheduled task menggunakan kredensial lama, mapped drive menggunakan saved credential, aplikasi lama yang melakukan authentication berulang, atau sistem monitoring/management tool yang salah konfigurasi — bukan hanya serangan.

- **Event pendukung?** 4625 berulang, cek juga 4740 bila akun sampai locked out.

- **Perubahan atau response aman?** Gunakan alur berikut, bukan langsung men-disable akun target:

  

```text

PRESERVE EVENTS

   ↓

IDENTIFY SOURCE

   ↓

CHECK TARGET ACCOUNTS

   ↓

CHECK SERVICE / TASK / MAPPED DRIVE

   ↓

CHECK ACCOUNT LOCKOUT

   ↓

VALIDATE AUTHORIZATION

   ↓

CONTAIN SOURCE OR ACCOUNT

ONLY IF CONFIRMED

```

  

  Preserve dan export event terlebih dahulu, identifikasi source IP/workstation, periksa account target, periksa lockout Event 4740, periksa service dan scheduled task yang berjalan dengan account terkait, lalu batasi source melalui firewall atau disable account hanya jika terbukti tidak sah dan objective mengizinkan.

  

> Men-disable account hanya karena menjadi target failed logon dapat menyebabkan denial-of-service terhadap user atau service sah.

  

- **Verifikasi?** Cocokkan waktu dengan aktivitas service/scheduled task/mapped drive yang sah pada baseline.

- **Rollback?** Re-enable akun jika disable ternyata keliru, restore firewall scope jika source ternyata sah, atau perbaiki credential service/task bila penyebabnya password lama.

- **Evidence?** Export event 4625 terkait beserta rentang waktu dan source.

- **Kesimpulan sementara?** Observation — pola percobaan berulang; belum conclusion sebelum sumber dan penyebab dipastikan.

  

### Skenario 2 — 4624 Logon Type 10 pada waktu tidak wajar

  

- **Apa yang dicek?** Waktu logon dibandingkan jam kerja normal, akun yang digunakan, source workstation/IP.

- **Risiko?** Akses RDP di luar jam kerja bisa sah (maintenance) atau tidak sah.

- **Event pendukung?** 4624 Logon Type 10, cek 4672 bila akun memiliki privilege khusus.

- **Perubahan atau response aman?** Verifikasi ke pemilik akun/pihak berwenang sebelum tindakan lanjutan.

- **Verifikasi?** Cocokkan dengan jadwal maintenance resmi bila ada.

- **Rollback?** Tidak diperlukan bila logon terverifikasi sah.

- **Evidence?** Export 4624 beserta context lengkap (Logon ID, source, waktu).

- **Kesimpulan sementara?** Observation hingga konfirmasi otorisasi didapat.

  

### Skenario 3 — 4672 muncul pada akun yang bukan admin resmi

  

- **Apa yang dicek?** Identitas account (built-in service account atau user biasa), Logon ID pada 4672, korelasi dengan Event 4624, privilege list yang tercantum, keanggotaan group akun tersebut saat ini, riwayat perubahan group (4728/4732/4756).

- **Risiko?** Privilege escalation tidak sah, atau akun memang baru ditambahkan secara sah namun belum terdokumentasi. Event 4672 tidak otomatis berarti privilege escalation.

- **Event pendukung?** 4672, dikorelasikan dengan 4624 melalui Logon ID, serta 4728/4732/4756 bila ada perubahan group sebelumnya.

- **Perubahan atau response aman?** Belum melakukan perubahan sebelum validasi; bila tidak sah, remove membership berlebihan setelah validasi.

- **Verifikasi?** Cocokkan account dengan authorized administrator baseline dan authorized change list.

- **Rollback?** Kembalikan keanggotaan group ke kondisi sebelumnya bila diperlukan.

- **Evidence?** Export 4672, 4624 terkait (Logon ID), dan event perubahan group terkait.

- **Kesimpulan sementara?** Event 4672 merupakan indikator privilege-bearing logon. Status sah atau tidak sah ditentukan melalui account identity, Logon ID, privilege list, baseline administrator, dan perubahan group sebelumnya — bukan disimpulkan langsung sebagai privilege escalation.

  

### Skenario 4 — User baru dibuat dengan Event 4720

  

- **Apa yang dicek?** Subject account (siapa yang membuat), waktu pembuatan, apakah tercatat pada authorized list.

- **Risiko?** Akun backdoor, atau pembuatan akun sah untuk keperluan operasional.

- **Event pendukung?** 4720, ikuti dengan 4722 (aktivasi) dan 4738 (perubahan atribut).

- **Perubahan atau response aman?** **Containment:** disable akun yang terverifikasi tidak sah, setelah divalidasi. **Remediation:** hapus akun hanya jika sudah dipastikan tidak dibutuhkan dan evidence telah diamankan — jangan menghapus akun sebagai tindakan pertama.

- **Verifikasi?** Bandingkan dengan daftar user resmi/berwenang.

- **Rollback?**

  

```text

Rollback:

Aktifkan kembali akun jika disable ternyata keliru.

  

Jangan menghapus akun sebagai tindakan pertama karena

pembuatan ulang akun dengan nama yang sama akan menghasilkan

SID yang berbeda.

```

  

```text

SAME USERNAME

      ≠

SAME SID

```

  

- **Evidence?** Export 4720 beserta subject account dan waktu.

- **Kesimpulan sementara?** Observation hingga otorisasi dikonfirmasi.

  

### Skenario 5 — User ditambahkan ke privileged group

  

- **Apa yang dicek?** Event 4728/4732/4756, subject account, target account, group tujuan.

- **Risiko?** Privilege escalation.

- **Event pendukung?** Event group membership change terkait, disusul kemungkinan 4672 pada logon berikutnya.

- **Perubahan atau response aman?** Remove membership setelah validasi.

- **Verifikasi?** Cocokkan dengan authorized change list dan approval yang ada.

- **Rollback?** Kembalikan ke group semula.

- **Evidence?** Export event perubahan group beserta context.

- **Kesimpulan sementara?** Inference; conclusion menunggu validasi otorisasi.

  

### Skenario 6 — Event 4688 muncul tetapi command line kosong

  

- **Apa yang dicek?** Status policy Include Command Line dan effective GPO.

- **Risiko?** Kehilangan visibilitas argument proses, bukan berarti proses tersebut berbahaya.

- **Event pendukung?** 4688 tanpa field command line terisi.

- **Perubahan atau response aman?** Aktifkan policy Include Command Line, lalu jalankan safe test process untuk verifikasi.

- **Verifikasi?** Cek Event 4688 baru setelah policy diaktifkan.

- **Rollback?** Tidak relevan; ini perbaikan visibilitas, bukan containment.

- **Evidence?** Dokumentasikan status policy sebelum dan sesudah perubahan.

- **Kesimpulan sementara?** Fact — keterbatasan konfigurasi, bukan indikasi insiden.

  

### Skenario 7 — Service baru muncul pada Event 7045

  

- **Apa yang dicek?** Nama service, path executable, account yang menjalankan, waktu instalasi.

- **Risiko?** Service tidak sah bisa menjadi mekanisme persistence; service sah adalah bagian operasional normal.

- **Event pendukung?** 7045, cek juga 4697 bila audit relevan aktif.

- **Perubahan atau response aman?** Stop service setelah dependency diperiksa, bila terverifikasi tidak sah.

- **Verifikasi?** Bandingkan path dan account dengan baseline resmi.

- **Rollback?** Kembalikan startup type/state sesuai kondisi sebelumnya bila ternyata false positive.

- **Evidence?** Export 7045 beserta detail service.

- **Kesimpulan sementara?** Observation; conclusion menunggu perbandingan baseline.

  

### Skenario 8 — Scheduled task baru tidak dikenal

  

- **Apa yang dicek?** Nama task, action, trigger, author, account eksekusi, path executable.

- **Risiko?** Task dapat digunakan sebagai mekanisme persistence, atau merupakan task operasional sah yang belum terdokumentasi.

- **Event pendukung?** 4698, cek TaskScheduler Operational log sebagai supporting evidence.

- **Perubahan atau response aman?** Disable task setelah dependency diperiksa, bila tidak sah.

- **Verifikasi?** Bandingkan dengan authorized task baseline.

- **Rollback?** Aktifkan kembali bila ternyata task sah.

- **Evidence?** Export 4698 dan detail task dari Task Scheduler.

- **Kesimpulan sementara?** Inference; perlu konfirmasi pemilik task.

  

### Skenario 9 — Event 1102 menunjukkan Security log dibersihkan

  

- **Apa yang dicek?** Subject account pelaku, waktu kejadian, Logon ID, aktivitas sebelum dan sesudah.

- **Risiko?** Anti-forensics oleh attacker, atau maintenance oleh administrator sah tanpa dokumentasi memadai.

- **Event pendukung?** 1102, periksa event pada log lain (System, Application) di sekitar waktu yang sama sebagai cross-reference.

- **Perubahan atau response aman?** Preserve seluruh log yang tersisa segera; jangan melakukan perubahan lain sebelum evidence diamankan.

- **Verifikasi?** Konfirmasi ke administrator terkait apakah tindakan ini terdokumentasi.

- **Rollback?** Tidak dapat mengembalikan log yang sudah dibersihkan; fokus pada preservasi log yang tersisa.

- **Evidence?** Export log tersisa segera, catat Subject Account dan waktu 1102.

- **Kesimpulan sementara?** Temuan bernilai tinggi; conclusion menunggu konfirmasi otorisasi.

  

### Skenario 10 — Event 4719 menunjukkan audit policy berubah

  

- **Apa yang dicek?** Subject account, subcategory yang diubah, waktu perubahan.

- **Risiko?** Attacker dapat mengurangi visibilitas dengan mengubah audit policy; atau ini perubahan sah oleh administrator.

- **Event pendukung?** 4719, bandingkan dengan effective policy saat ini.

- **Perubahan atau response aman?** Kembalikan subcategory ke baseline bila tidak sah, setelah validasi.

- **Verifikasi?** Cek Group Policy Results dan `auditpol /get` untuk memastikan effective policy saat ini.

- **Rollback?** Terapkan kembali subcategory yang benar melalui GUI.

- **Evidence?** Export 4719 beserta konfigurasi audit policy sebelum dan sesudah.

- **Kesimpulan sementara?** Observation; perlu konfirmasi otorisasi.

  

### Skenario 11 — Defender 1116 muncul tanpa remediation berhasil

  

- **Apa yang dicek?** Detail action dan result pada 1117, ada tidaknya 1119.

- **Risiko?** Ancaman masih aktif di sistem meskipun terdeteksi.

- **Event pendukung?** 1116 diikuti 1117 dan/atau 1119.

- **Perubahan atau response aman?** Lakukan scan ulang melalui Windows Security setelah evidence diamankan; quarantine bila memungkinkan.

- **Verifikasi?** Cek scan result dan status file/process terkait saat ini.

- **Rollback?** Tidak relevan; fokus pada verifikasi remediasi berhasil.

- **Evidence?** Export 1116, 1117, dan 1119 terkait beserta detail file/threat.

- **Kesimpulan sementara?** Fact — deteksi terjadi; conclusion soal status akhir menunggu verifikasi scan.

  

### Skenario 12 — PowerShell 4104 menunjukkan script tidak dikenal

  

- **Apa yang dicek?** Isi script block secara lengkap, timestamp, hostname, user/SID jika tersedia, process context jika tersedia, ScriptBlock ID, path jika tersedia. **Jangan menyatakan Logon ID pasti tersedia** — field yang tersedia dapat berbeda berdasarkan versi PowerShell, provider, versi Windows Server, jenis host PowerShell, dan konfigurasi logging.

- **Risiko?** Script tidak sah dapat mengindikasikan aktivitas berbahaya, atau merupakan automation sah yang belum terdokumentasi. Jangan menganggap script tidak dikenal otomatis malicious.

- **Event pendukung?** 4104 — script panjang dapat terpecah menjadi beberapa Event 4104, gunakan ScriptBlock ID untuk menyatukan bagian jika tersedia — cek juga 4103, 4688, dan logon events pada waktu berdekatan.

- **Perubahan atau response aman?** Preserve seluruh bagian script block, konfirmasi ke pemilik sistem sebelum tindakan lanjutan.

- **Verifikasi?** Cocokkan dengan automation resmi dan authorized script list.

- **Rollback?** Tidak relevan pada tahap investigasi awal.

- **Evidence?** Export seluruh bagian 4104 beserta context yang tersedia dan waktu.

- **Kesimpulan sementara?** Observation; conclusion menunggu identifikasi pemilik script.

  

### Skenario 13 — Object access event tidak muncul

  

- **Apa yang dicek?** Status Audit Object Access, keberadaan SACL pada objek terkait.

- **Risiko?** Bukan berarti tidak ada akses; bisa jadi SACL belum dikonfigurasi.

- **Event pendukung?** Tidak ada event sampai SACL dikonfigurasi dengan benar.

- **Perubahan atau response aman?** Konfigurasi SACL terbatas pada objek yang relevan, lalu jalankan safe access test.

- **Verifikasi?** Cek Security log setelah safe test dilakukan.

- **Rollback?** Kembalikan SACL ke kondisi semula setelah pengujian selesai bila hanya untuk keperluan uji.

- **Evidence?** Dokumentasikan status SACL sebelum dan sesudah, serta hasil safe test.

- **Kesimpulan sementara?** Fact — keterbatasan konfigurasi; bukan bukti tidak adanya aktivitas.

  

### Skenario 14 — Security log terlalu cepat penuh

  

- **Apa yang dicek?** Volume event, Event ID yang paling dominan (noisy), scope SACL bila Object Access aktif.

- **Risiko?** Log rotation terlalu cepat menyebabkan evidence lama hilang.

- **Event pendukung?** 1104 (log full), pola volume tinggi pada Event ID tertentu.

- **Perubahan atau response aman?** Sesuaikan subcategory yang terlalu luas, sempitkan SACL bila relevan.

- **Verifikasi?** Pantau volume setelah penyesuaian.

- **Rollback?** Kembalikan ke konfigurasi semula bila penyesuaian menimbulkan efek samping.

- **Evidence?** Catat ukuran log, Event ID dominan, dan perubahan yang dilakukan.

- **Kesimpulan sementara?** Fact — masalah kapasitas/konfigurasi, bukan otomatis insiden keamanan.

  

### Skenario 15 — Event yang dicari hanya muncul pada Domain Controller

  

- **Apa yang dicek?** Apakah host yang diperiksa adalah tempat yang tepat untuk event tersebut (misalnya Kerberos events pada DC).

- **Risiko?** Kesimpulan salah karena mencari event di host yang tidak seharusnya menghasilkan event tersebut.

- **Event pendukung?** 4768/4769/4771 sebagai contoh event yang relevan pada DC.

- **Perubahan atau response aman?** Alihkan pencarian ke Domain Controller yang sesuai.

- **Verifikasi?** Cocokkan waktu dan akun antara member server dan DC.

- **Rollback?** Tidak relevan.

- **Evidence?** Export event dari DC yang relevan.

- **Kesimpulan sementara?** Fact — event tidak ditemukan pada host yang salah bukan berarti aktivitas tidak terjadi.

  

### Skenario 16 — Timeline memiliki timestamp yang tampak tidak konsisten

  

- **Apa yang dicek?** Time zone masing-masing host, status sinkronisasi waktu, Event 4616 (system time changed).

- **Risiko?** Kesimpulan urutan kejadian keliru akibat perbedaan zona waktu atau clock drift.

- **Event pendukung?** 4616 bila ada perubahan waktu sistem.

- **Perubahan atau response aman?** Normalisasi seluruh timestamp ke satu zona waktu referensi sebelum menyusun timeline.

- **Verifikasi?** Bandingkan waktu event dengan sumber waktu yang dipercaya.

- **Rollback?** Tidak relevan.

- **Evidence?** Catat time zone tiap host dan proses normalisasi yang dilakukan.

- **Kesimpulan sementara?** Fact — perlu normalisasi waktu sebelum kesimpulan urutan kejadian dapat diambil.

  

---

  

## GUI PRACTICAL LAB

  

Setiap lab wajib memiliki: Scenario, Objective, Safety Check, GUI Steps, Expected Event, Verification, Rollback, Evidence, Interpretation.

  

### LAB 1 — Logging Baseline

  

- **Scenario:** Mesin baru diserahkan untuk hardening dan investigasi.

- **Objective:** Mencatat baseline logging sebelum melakukan perubahan apa pun.

- **Safety Check:** Tidak ada perubahan konfigurasi pada lab ini, hanya observasi.

- **GUI Steps:**

  1. Buka Settings/Control Panel untuk cek time zone dan waktu sistem.

  2. Buka Event Viewer, cek ketersediaan log Application, Security, System.

  3. Cek Applications and Services Logs yang relevan.

  4. Klik kanan tiap log → Properties, catat maximum size dan retention method.

  5. Buka `secpol.msc`, catat status Advanced Audit Policy.

- **Expected Event:** Tidak ada event baru; ini adalah observasi kondisi awal.

- **Verification:** Bandingkan hasil catatan dengan checklist WGUI-6.3.

- **Rollback:** Tidak diperlukan.

- **Evidence:** Dokumen checklist baseline yang terisi lengkap.

- **Interpretation:** Baseline menjadi acuan pembanding untuk semua investigasi berikutnya.

  

### LAB 2 — Advanced Audit Policy

  

- **Scenario:** Subcategory audit yang dibutuhkan belum aktif.

- **Objective:** Mengaktifkan satu subcategory yang dibutuhkan tanpa menimbulkan konflik dengan Basic Audit Policy.

- **Safety Check:** Jangan mengaktifkan seluruh subcategory Success dan Failure sekaligus. Jangan mengaktifkan setting override tanpa memahami apakah environment masih menggunakan Basic Audit Policy.

- **GUI Steps:**

  1. Buka `secpol.msc`, `gpedit.msc`, atau `gpmc.msc` sesuai sumber policy.

  2. Masuk ke: `Local Policies → Security Options`.

  3. Buka: `Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings`.

  4. Set ke **Enabled** jika Advanced Audit Policy digunakan.

  5. Masuk ke: `Advanced Audit Policy Configuration → Audit Policies`.

  6. Pilih satu subcategory yang sesuai objective (misalnya Logon/Logoff).

  7. Aktifkan Success atau Failure secara presisi.

  8. Terapkan policy.

  9. Periksa effective policy melalui Group Policy Results dan `auditpol /get`.

- **Expected Event:** Event sesuai subcategory yang diaktifkan akan mulai tercatat.

- **Verification:** Cek melalui Group Policy Results dan `auditpol /get /category:*`.

- **Rollback:** Kembalikan subcategory ke kondisi semula melalui GUI yang sama.

- **Evidence:** Screenshot konfigurasi sebelum dan sesudah, hasil verifikasi `auditpol`.

- **Interpretation:** Effective policy harus dikonfirmasi, bukan hanya configured policy.

  

### LAB 3 — Authentication Investigation

  

- **Scenario:** Perlu membedakan logon sah dan gagal pada host yang sama.

- **Objective:** Menghasilkan dan menganalisis Event 4624 dan 4625 melalui aktivitas login aman.

- **Safety Check:** Gunakan akun uji resmi, jangan menggunakan akun produksi. Hanya lakukan satu atau sedikit percobaan gagal. Jangan sengaja menyebabkan lockout. Jangan disable account hanya karena event muncul.

- **GUI Steps:**

  1. Lakukan login dengan kredensial benar (interactive).

  2. Lakukan satu percobaan login dengan password salah pada akun uji.

  3. Buka Event Viewer → Security log.

  4. Filter Event ID 4624 dan 4625.

- **Expected Event:** 4624 pada login berhasil, 4625 pada login gagal.

- **Verification:** Cocokkan Logon Type, akun, dan waktu pada masing-masing event.

- **Rollback:** Tidak diperlukan; ini aktivitas observasi.

- **Evidence:** Export kedua event beserta detail Logon Type dan source.

- **Interpretation:** Logon Type dan context menentukan makna sebenarnya, bukan Event ID saja.

  

### LAB 4 — Process Creation

  

- **Scenario:** Command line pada 4688 tidak terlihat.

- **Objective:** Mengaktifkan Audit Process Creation dan Include Command Line, lalu menganalisis Event 4688.

- **Safety Check:** Command-line logging dapat mencatat data sensitif seperti password, token, atau connection string sebagai teks biasa. Jangan memasukkan password atau data sensitif pada command line; jangan menjalankan payload atau script berbahaya.

- **GUI Steps:**

  1. Aktifkan Audit Process Creation melalui Advanced Audit Policy.

  2. Aktifkan Include command line in process creation events melalui Group Policy.

  3. Jalankan aplikasi aman seperti Notepad.

  4. Buka Event Viewer → Security log, filter Event ID 4688.

- **Expected Event:** 4688 dengan command line terisi sesuai aplikasi yang dijalankan.

- **Verification:** Pastikan field command line terisi dan sesuai proses yang dijalankan.

- **Rollback:** Nonaktifkan kembali policy bila hanya untuk keperluan pengujian singkat.

- **Evidence:** Export Event 4688 beserta parent process dan command line.

- **Interpretation:** Visibilitas command line bergantung pada dua policy yang harus aktif bersamaan.

  

### LAB 5 — Account atau Group Change

  

- **Scenario:** Perlu memverifikasi bahwa perubahan akun tercatat dengan benar.

- **Objective:** Melakukan satu perubahan aman pada akun uji, menemukan event terkait, lalu melakukan rollback.

- **Safety Check:** Gunakan akun uji resmi lab, bukan akun produksi atau akun kompetisi asli.

- **GUI Steps:**

  1. Buka Local Users and Groups atau Active Directory Users and Computers (sesuai role server).

  2. Tambahkan akun uji ke satu group tertentu.

  3. Buka Event Viewer → Security log, cari event group membership change terkait.

  4. Lakukan rollback dengan menghapus akun uji dari group tersebut.

- **Expected Event:** Event 4728/4732/4756 sesuai jenis group yang digunakan.

- **Verification:** Cocokkan Subject Account, Target Account, dan waktu pada event.

- **Rollback:** Hapus kembali membership yang ditambahkan untuk pengujian.

- **Evidence:** Export event perubahan group sebelum dan sesudah rollback.

- **Interpretation:** Event membuktikan perubahan terjadi; authorized list menentukan keabsahannya.

  

### LAB 6 — Service atau Scheduled Task Monitoring

  

- **Scenario:** Perlu memverifikasi visibilitas perubahan service/task.

- **Objective:** Membuat task uji aman melalui Task Scheduler, menganalisis event terkait, lalu melakukan rollback terkontrol.

- **Safety Check:** Gunakan action aman yang disediakan lab (misalnya menampilkan pesan atau menjalankan perintah aman), bukan payload berbahaya.

- **GUI Steps:**

  1. Buka Task Scheduler, buat task baru dengan trigger dan action aman.

  2. Buka Event Viewer → Security log, cari Event 4698.

  3. Cek juga TaskScheduler Operational log sebagai supporting evidence.

  4. Hapus task uji melalui Task Scheduler.

  5. Cek Event 4699 pada Security log.

- **Expected Event:** 4698 saat pembuatan, 4699 saat penghapusan.

- **Verification:** Bandingkan detail task (trigger, action, author) dengan yang dibuat.

- **Rollback:** Task sudah dihapus sebagai bagian dari lab; pastikan tidak ada task uji tersisa.

- **Evidence:** Export 4698 dan 4699 beserta detail task.

- **Interpretation:** Perubahan task selalu harus dibandingkan dengan baseline yang authorized.

  

### LAB 7 — PowerShell Logging

  

- **Scenario:** Aktivitas PowerShell tidak terlihat pada Event Viewer.

- **Objective:** Mengaktifkan Module Logging atau Script Block Logging, lalu mencari event yang dihasilkan.

- **Safety Check:** Gunakan perintah aman seperti `Get-Date`; jangan menjalankan script yang tidak diketahui asal-usulnya.

- **GUI Steps:**

  1. Buka `gpedit.msc`, aktifkan Turn on Module Logging dan/atau Turn on PowerShell Script Block Logging.

  2. Jalankan perintah aman seperti `Get-Date` pada PowerShell.

  3. Buka Event Viewer → Microsoft-Windows-PowerShell/Operational.

  4. Cari Event 4103 dan/atau 4104.

- **Expected Event:** 4103 untuk module logging, 4104 untuk script block logging. Event 4104 dapat terpecah menjadi beberapa bagian bila script panjang; gunakan ScriptBlock ID untuk menyatukan bagian jika tersedia.

- **Verification:** Pastikan isi event sesuai perintah yang dijalankan. Jangan menjamin Logon ID tersedia pada Event 4104 — field yang tersedia dapat berbeda menurut versi PowerShell dan konfigurasi logging.

- **Rollback:** Nonaktifkan kembali policy bila hanya untuk keperluan pengujian singkat.

- **Evidence:** Export 4103/4104 beserta waktu dan user/SID jika tersedia.

- **Interpretation:** Visibilitas PowerShell membutuhkan kombinasi module logging, script block logging, dan transcription yang aman.

  

### LAB 8 — Defender dan Firewall Correlation

  

- **Scenario:** Perlu menghubungkan event Defender dan firewall dengan aktivitas yang sama.

- **Objective:** Menggunakan event aman yang sudah tersedia dari WGUI-4/WGUI-5 untuk membangun korelasi.

- **Safety Check:** Gunakan event/artefak simulasi yang disediakan lingkungan lab, jangan membuat malware atau payload baru.

- **GUI Steps:**

  1. Buka Windows Security, cek riwayat deteksi Defender yang tersedia.

  2. Buka Event Viewer → Microsoft-Windows-Windows Defender/Operational, cari Event 1116/1117.

  3. Buka Windows Defender Firewall with Advanced Security atau Security log untuk event WFP terkait (5152/5156/5157) bila tersedia.

  4. Susun tabel korelasi waktu, process, dan source IP dari kedua sumber.

- **Expected Event:** 1116/1117 dari Defender, 5152/5156/5157 dari firewall/WFP bila tersedia.

- **Verification:** Pastikan waktu dan process/IP pada kedua event saling mendukung.

- **Rollback:** Tidak diperlukan; ini aktivitas observasi dan korelasi.

- **Evidence:** Export event dari kedua sumber beserta tabel korelasi yang disusun.

- **Interpretation:** Korelasi lintas sumber memberikan gambaran lebih lengkap dibanding satu event tunggal.

  

### LAB 9 — Object Access Audit

  

- **Scenario:** Akses pada folder kritis perlu tercatat.

- **Objective:** Menerapkan SACL terbatas pada folder uji, melakukan akses aman, membaca event, lalu melakukan rollback SACL.

- **Safety Check:** Gunakan folder uji yang dibuat khusus untuk lab ini, bukan folder sistem produksi.

- **GUI Steps:**

  1. Buat folder uji baru.

  2. Aktifkan subcategory Audit File System melalui Advanced Audit Policy.

  3. Buka Properties folder uji → Security → Advanced → Auditing, tambahkan SACL untuk user/action yang relevan.

  4. Akses folder tersebut menggunakan akun uji.

  5. Buka Event Viewer → Security log, cari event akses objek terkait.

  6. Hapus SACL yang ditambahkan setelah pengujian selesai.

- **Expected Event:** Event object access sesuai subcategory dan SACL yang dikonfigurasi.

- **Verification:** Pastikan event mencatat user, waktu, dan jenis akses yang sesuai dengan aktivitas uji.

- **Rollback:** Hapus SACL uji dan nonaktifkan subcategory bila hanya untuk keperluan pengujian.

- **Evidence:** Export event object access beserta detail SACL yang digunakan.

- **Interpretation:** Tanpa SACL, audit policy saja tidak akan menghasilkan event pada objek tersebut.

  

### LAB 10 — Evidence Export

  

- **Scenario:** Event penting perlu diamankan sebagai evidence formal.

- **Objective:** Mengekspor full log sebagai original evidence, membuat working copy, menerapkan filter untuk menghasilkan working set, menghitung SHA-256, dan mendokumentasikan chain of custody sederhana.

- **Safety Check:** Jangan mengubah file export asli (FULL) setelah dibuat.

- **GUI Steps:**

  1. Buka Event Viewer, pilih log yang relevan.

  2. **Save All Events As**, simpan seluruh log sebagai original evidence dengan format nama `YYYYMMDD-HHMM_HOST_LOGNAME_CASE-FULL.evtx`.

  3. Hitung `Get-FileHash <file> -Algorithm SHA256` pada file FULL.

  4. Salin file FULL menjadi working copy terpisah.

  5. Terapkan filter atau Custom View pada working copy sesuai objective (misalnya menggunakan Custom View dari lab sebelumnya).

  6. **Save Filtered Log File As** atau **Save All Events in Custom View As**, simpan sebagai working set dengan format nama `YYYYMMDD-HHMM_HOST_LOGNAME_CASE-FILTERED.evtx`.

  7. Dokumentasikan kedua file (FULL dan FILTERED) beserta hostname, waktu, zona waktu, rentang waktu, filter yang digunakan, dan siapa yang mengambil evidence.

  8. Analisis hanya pada working set/copy, bukan pada file FULL asli.

- **Expected Event:** Tidak ada event baru; ini adalah prosedur penanganan evidence.

- **Verification:** Bandingkan hash file FULL dengan salinannya bila dibuat byte-for-byte identik. Hash file FILTERED tidak perlu dan tidak akan sama dengan hash file FULL karena isinya sudah disaring.

- **Rollback:** Tidak relevan.

- **Evidence:** File `.evtx` FULL (original evidence), file `.evtx` FILTERED (working set), nilai SHA-256 masing-masing file, dan catatan chain of custody.

- **Interpretation:** Evidence yang tidak diverifikasi integritasnya berisiko diragukan validitasnya saat penilaian. Original evidence dan working set memiliki peran berbeda dan tidak boleh dicampur.

  

### LAB 11 — Mini Incident Timeline

  

- **Scenario:** Sejumlah event dari berbagai kategori tersedia dan perlu disusun menjadi satu timeline investigasi.

- **Objective:** Menyusun timeline dari event authentication, process, account change, Defender, serta service/task yang disediakan lab.

- **Safety Check:** Gunakan kumpulan event yang disediakan lingkungan lab; jangan membuat aktivitas ofensif untuk menghasilkan event tambahan.

- **GUI Steps:**

  1. Kumpulkan seluruh event terkait dari Event Viewer sesuai kategori yang tersedia.

  2. Susun tabel timeline sesuai format WGUI-6.19 (Time, Host, Log, Event ID, User, Source IP, Process/Object, Interpretation, Confidence).

  3. Urutkan berdasarkan waktu yang sudah dinormalisasi ke satu zona waktu referensi.

  4. Tandai setiap baris sebagai Fact, Observation, Inference, atau Conclusion.

- **Expected Event:** Kombinasi event dari beberapa kategori yang sudah dibahas pada modul ini.

- **Verification:** Pastikan setiap baris timeline memiliki dukungan event yang jelas, bukan asumsi.

- **Rollback:** Tidak relevan; ini aktivitas analisis.

- **Evidence:** Tabel timeline lengkap beserta event pendukung yang sudah diexport.

- **Interpretation:** Timeline yang baik menunjukkan hubungan sebab-akibat antar event, bukan sekadar daftar event terpisah.

  

---

  

## FINAL INTEGRATED SCENARIO

  

**Skenario:** Seorang user mencurigakan ditemukan telah memperoleh privilege tinggi. Login RDP terjadi pada waktu yang tidak biasa. Sebuah process baru dijalankan setelah logon tersebut. Firewall atau Defender menghasilkan event terkait aktivitas ini. Sebuah scheduled task atau service baru muncul tidak lama setelahnya. Audit policy kemudian berubah.

  

Gunakan seluruh event dan artefak simulasi yang telah disediakan lingkungan lab. Jangan mencoba mereproduksi rangkaian ini menggunakan teknik serangan nyata.

  

> Prinsip yang berlaku di seluruh skenario: Event 4672 tidak otomatis berarti privilege escalation, dan banyak percobaan failed logon (4625) tidak otomatis berarti account harus di-disable. Setiap kesimpulan harus melalui korelasi dan validasi.

  

Alur penyelesaian wajib:

  

```text

IDENTIFY

→ BASELINE

→ COLLECT

→ CORRELATE

→ PRESERVE

→ CONTAIN

→ VERIFY

→ EVIDENCE

```

  

Langkah kerja:

  

1. **IDENTIFY** — Tentukan alert awal: privilege tinggi pada akun mencurigakan (4672 — belum tentu privilege escalation), logon RDP tidak wajar (4624 Logon Type 10).

2. **BASELINE** — Bandingkan dengan authorized admin list, jadwal kerja normal, dan baseline service/task yang sah.

3. **COLLECT** — Kumpulkan seluruh event terkait: account/group change, process creation (4688), Defender (1116/1117), firewall/WFP, service/task (7045/4698), dan policy change (4719).

4. **CORRELATE** — Susun timeline berdasarkan Logon ID, user, process, waktu, dan object yang saling terhubung; bedakan fact, observation, inference, dan conclusion.

5. **PRESERVE** — Export full log terlebih dahulu sebagai original evidence ke `.evtx`, hitung SHA-256 pada original, baru buat working copy dan filtered export sebagai working set, serta dokumentasikan chain of custody.

6. **CONTAIN** — Lakukan containment minimal yang dapat di-rollback: disable akun tidak sah, disable service/task tidak sah, quarantine melalui Defender, atau batasi firewall scope — sesuai hasil validasi, bukan berdasarkan dugaan awal. Jangan menghapus user sebagai tindakan pertama. Jangan menghapus service/task sebelum dependency diperiksa.

7. **VERIFY** — Pastikan layanan penting tetap berjalan setelah containment, dan tidak ada dependency yang rusak.

8. **EVIDENCE** — Susun dokumentasi akhir: timeline lengkap, event pendukung, hasil containment, dan status verifikasi akhir.

  

> Skenario ini tidak menyediakan atau membutuhkan teknik untuk menghasilkan aktivitas tersebut. Peserta bekerja dari event dan artefak yang sudah tersedia di lingkungan lab.

  

---

  

## MEMORY SECTION — BLUE TEAM COMBAT MEMORY

  

Alur ingatan cepat:

  

```text

TIME

 ↓

HOST

 ↓

USER

 ↓

LOGON

 ↓

PROCESS

 ↓

ACCOUNT CHANGE

 ↓

SERVICE/TASK

 ↓

DEFENDER/FIREWALL

 ↓

PRESERVE

 ↓

CORRELATE

 ↓

RESPOND

```

  

Formula inti:

  

```text

TRUSTED FINDING =

EVENT

+ CONTEXT

+ CORRELATION

+ CURRENT STATE

```

  

```text

NO EVENT

≠

NO ACTIVITY

```

  

```text

AUDIT POLICY

+ PROVIDER

+ CAPACITY

+ CORRECT HOST

=

AVAILABLE EVIDENCE

```

  

```text

PRESERVE FIRST

ANALYZE COPY

RESPOND MINIMALLY

```

  

```text

4672

≠

AUTOMATIC PRIVILEGE ESCALATION

```

  

```text

FULL LOG

=

ORIGINAL EVIDENCE

```

  

```text

FILTERED LOG

=

WORKING SET

```

  

```text

FAILED LOGON TARGET

≠

AUTOMATIC ACCOUNT COMPROMISE

```

  

```text

SAME USERNAME

≠

SAME SID

```

  

**10 Langkah Tempur:**

  

1. Catat host, waktu, dan zona waktu.

2. Identifikasi log, provider, serta channel yang benar.

3. Periksa configured dan effective audit policy.

4. Baca Event ID bersama seluruh field context.

5. Jangan anggap 4672 otomatis privilege escalation.

6. Korelasikan user, Logon ID, process, IP, object, dan event pendukung.

7. Export full log sebagai original evidence.

8. Gunakan filtered log atau working copy untuk analisis.

9. Lakukan containment minimal hanya setelah validasi.

10. Verifikasi current state dan dokumentasikan fact, observation, inference, serta conclusion.

  

---

  

STATUS: FINAL COMPETITION READY — LOCKED