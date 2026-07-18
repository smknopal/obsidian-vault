# WGUI-5 — MICROSOFT DEFENDER & ATTACK SURFACE REDUCTION

  

### Windows GUI Hardening Learning System

  

**Modul Persiapan Kompetisi Cyber Security LKS Tingkat Nasional — Blue Team Track**

  

Modul ini merupakan kelanjutan dari:

  

* WGUI-0 — Windows Hardening Foundation

* WGUI-1 — Local Account, PAM, dan Local Security Policy

* WGUI-2 — Group Policy Hardening

* WGUI-3 — Active Directory Hardening

* WGUI-4 — Windows Network & Firewall Hardening

  

---

  

## WGUI-5.0 Posisi dan Tujuan Modul

  

WGUI-4 mengendalikan jalur masuk dan keluar trafik jaringan pada level firewall dan network layer. WGUI-5 melangkah satu lapis lebih dalam, yaitu mengendalikan apa yang boleh terjadi **di dalam** endpoint setelah sebuah proses, file, script, atau koneksi berhasil melewati jalur jaringan. WGUI-5 mengontrol:

  

* File yang masuk, disimpan, dan dieksekusi

* Process dan child process yang dibuat oleh aplikasi

* Script (PowerShell, VBScript, JavaScript) yang dijalankan

* Application behavior yang mencurigakan

* Exploit mitigation pada level proses

* Ransomware protection pada folder kritikal

* Threat remediation setelah deteksi terjadi

  

Hasil belajar yang harus dikuasai peserta setelah menyelesaikan modul ini:

  

* Mampu mengidentifikasi state Microsoft Defender Antivirus secara cepat melalui GUI pada berbagai edisi Windows Server.

* Mampu membedakan *configured setting* dan *effective setting*.

* Mampu melakukan baseline, perubahan satu langkah, verifikasi, dan rollback tanpa mengganggu layanan.

* Mampu mengaudit exclusion, quarantine, dan Protection History secara aman.

* Mampu menerapkan Attack Surface Reduction rules dengan alur Audit-first.

* Mampu membaca Event Viewer untuk memvalidasi effective protection.

* Mampu menyusun evidence yang lengkap untuk keperluan penilaian lomba.

  

---

  

## WGUI-5.1 Mental Model Defender

  

```text

PREVENT

   ↓

DETECT

   ↓

BLOCK

   ↓

QUARANTINE

   ↓

INVESTIGATE

   ↓

REMEDIATE

   ↓

VERIFY

```

  

Setiap komponen Defender berperan pada tahap berbeda dalam alur di atas:

  

| Komponen | Peran utama | Tahap pada mental model |

|---|---|---|

| Antivirus (signature-based) | Mendeteksi file dikenal berbahaya | DETECT, BLOCK |

| Real-time protection | Memantau file dan proses saat diakses | PREVENT, DETECT |

| Behavior monitoring | Mendeteksi pola perilaku mencurigakan | DETECT |

| Cloud-delivered protection | Mempercepat verifikasi reputasi file via cloud | DETECT |

| Security intelligence | Basis data signature dan model deteksi | DETECT |

| Attack Surface Reduction (ASR) | Membatasi perilaku berisiko pada level proses/script | PREVENT, BLOCK |

| Controlled Folder Access (CFA) | Melindungi folder dari penulisan oleh aplikasi tidak dipercaya | PREVENT, BLOCK |

| Exploit Protection | Mitigasi teknik eksploitasi pada level proses | PREVENT |

| Network Protection | Memblokir koneksi keluar ke domain/IP berbahaya | PREVENT, BLOCK |

| Tamper Protection | Melindungi konfigurasi Defender dari perubahan tidak sah | PREVENT (terhadap Defender sendiri) |

  

> Catatan: ASR, CFA, Exploit Protection, dan Network Protection bekerja sebelum dan di luar deteksi signature murni. Komponen ini menekan permukaan serangan meskipun file belum dikenali sebagai malware.

  

---

  

## WGUI-5.2 GUI Tools dan Availability Check

  

| Tool | Lokasi | Fungsi utama | Keterbatasan |

|---|---|---|---|

| Windows Security | Start Menu atau `windowsdefender:` | Melihat status antivirus, menjalankan scan, membaca Protection History, mengelola exclusions, Controlled Folder Access, protected folders, allowed apps, dan Exploit Protection jika fitur tersebut tersedia | Tidak tersedia penuh pada Server Core; sebagian tersembunyi jika dikelola GPO; tidak digunakan untuk mengonfigurasi daftar individual Attack Surface Reduction rules |

| Virus & threat protection | Dalam Windows Security | Real-time protection, scan, Protection History, quarantine | Toggle greyed out jika dikunci Tamper Protection/GPO |

| App & browser control | Dalam Windows Security | Exploit protection settings | Hanya tersedia jika Desktop Experience terpasang |

| Server Manager | Start Menu (Server) | Menambah/melihat fitur Windows Defender Antivirus | Tidak menampilkan detail protection state |

| Add Roles and Features | Server Manager | Memastikan role/feature Windows Defender Antivirus terpasang | Hanya instalasi, bukan konfigurasi harian |

| `gpedit.msc` | Run/MMC | Local Group Policy Editor untuk setting Defender lokal, termasuk ASR rule configuration dan CFA mode | Tetap dapat tersedia pada domain-joined Windows Server, tetapi local policy dapat ditimpa oleh Domain GPO; ketersediaan tetap bergantung pada edisi, installation type, dan komponen GUI yang terpasang; local policy bukan bukti effective policy |

| `gpmc.msc` | Domain-joined server | Domain GPO untuk Defender terpusat | Memerlukan hak akses AD yang sesuai |

| `eventvwr.msc` | Run/MMC | Membaca log Operational Windows Defender | Hanya membaca, tidak mengubah konfigurasi |

| `services.msc` | Run/MMC | Memeriksa status service `WinDefend`, `SecurityHealthService` | Tidak menampilkan detail policy |

| Task Scheduler | Run/MMC | Memeriksa scheduled scan Defender | Terbatas untuk audit jadwal, bukan konfigurasi protection |

| Group Policy Results | `rsop.msc` atau GPMC | Melihat effective policy yang benar-benar diterapkan | Memerlukan waktu refresh, tidak real-time |

  

> Windows Security tidak digunakan untuk mengonfigurasi daftar individual Attack Surface Reduction rules. Jalur GUI-first utama untuk ASR rules adalah `gpedit.msc` untuk Local Group Policy, `gpmc.msc` untuk Domain Group Policy, dan Group Policy Results atau `rsop.msc` untuk memeriksa effective policy.

  

Domain membership tidak otomatis menghilangkan `gpedit.msc`; local policy tetap dapat dikonfigurasi, tetapi local policy bukan bukti effective policy.

  

```text

LOCAL POLICY

      ≠

EFFECTIVE POLICY

```

  

```text

CONFIGURE LOCAL POLICY

        ↓

CHECK DOMAIN GPO

        ↓

CHECK POLICY PRECEDENCE

        ↓

CHECK GROUP POLICY RESULTS

        ↓

VERIFY EFFECTIVE SETTING

```

  

### Ringkasan Peran Setiap GUI Tool

  

**Windows Security**

  

Digunakan untuk:

  

* Antivirus status.

* Scan.

* Protection History.

* Threat actions.

* Exclusions.

* Cloud protection jika tersedia.

* Controlled Folder Access jika tersedia.

* Exploit Protection.

  

Tidak digunakan untuk:

  

* Mengonfigurasi daftar individual ASR rules.

* Menentukan CFA Audit mode.

* Menentukan seluruh effective policy.

* Mengelola Tamper Protection organisasi melalui GPO.

  

**gpedit.msc**

  

Digunakan untuk:

  

* Local Defender policy.

* ASR rule configuration.

* CFA mode.

* Network Protection policy.

* Core protection settings yang tersedia.

  

Keterbatasan:

  

* Local policy dapat ditimpa Domain GPO.

* Tidak membuktikan effective state.

  

**gpmc.msc**

  

Digunakan untuk:

  

* Defender policy pada domain.

* ASR, CFA, Network Protection, dan policy lain secara terpusat.

* Harus digunakan hanya jika objective dan hak akses mengizinkan.

  

**Group Policy Results atau rsop.msc**

  

Digunakan untuk:

  

* Menentukan policy yang benar-benar menang.

* Menelusuri sumber effective setting.

  

**PowerShell**

  

Gunakan hanya untuk verifikasi opsional:

  

```text

Get-MpComputerStatus

Get-MpPreference

Get-MpThreatDetection

```

  

Jangan menjadikannya jalur konfigurasi utama.

  

Prosedur awal wajib sebelum melakukan perubahan apa pun:

  

1. Identifikasi versi Windows Server yang digunakan.

2. Identifikasi apakah instalasi berupa Server Core atau Desktop Experience.

3. Periksa apakah Microsoft Defender Antivirus terpasang sebagai feature.

4. Periksa apakah Windows Security GUI tersedia dan dapat dibuka.

5. Periksa mode Defender: Active, Passive, atau tidak aktif.

6. Periksa keberadaan antivirus pihak ketiga yang mungkin menjadi primary protection.

7. Identifikasi sumber konfigurasi yang berlaku: local policy, domain GPO, atau centralized management lain.

  

> Peringatan: Server Core tidak memiliki Windows Security GUI lengkap. Pada kondisi ini, verifikasi dan sebagian konfigurasi harus dilakukan melalui `gpedit.msc`/`gpmc.msc` (jika tersedia) atau PowerShell sebagai alat bantu verifikasi, bukan sebagai metode konfigurasi utama. Jangan mengarang lokasi menu yang tidak tersedia pada edisi yang sedang diaudit.

  

---

  

## WGUI-5.3 Defender Baseline

  

Checklist baseline yang wajib direkam sebelum melakukan perubahan apa pun:

  

- [ ] Defender feature terpasang

- [ ] Service `WinDefend` berjalan (`services.msc`)

- [ ] Mode Active/Passive teridentifikasi

- [ ] Ada/tidaknya antivirus pihak ketiga

- [ ] Status Real-time protection

- [ ] Status Behavior monitoring

- [ ] Status Cloud-delivered protection

- [ ] Status Automatic sample submission

- [ ] Security intelligence version dan tanggal

- [ ] Platform dan engine version/status

- [ ] Status PUA protection

- [ ] Riwayat scan terakhir

- [ ] Recent threats (jika ada)

- [ ] Daftar exclusions (file, folder, extension, process)

- [ ] State setiap ASR rule

- [ ] State Controlled Folder Access

- [ ] State Network Protection

- [ ] Custom setting Exploit Protection

- [ ] Status Tamper Protection

- [ ] Sumber policy yang efektif (local/GPO/terpusat)

- [ ] Jalur recovery/rollback yang tersedia

  

> Baseline wajib dicatat sebagai bagian dari evidence sebelum perubahan pertama dilakukan. Tanpa baseline, klaim perbaikan tidak dapat dibuktikan.

  

---

  

## WGUI-5.4 Active, Passive, dan Policy-Managed State

  

> Pada Windows Server, keberadaan antivirus pihak ketiga tidak selalu otomatis membuat Microsoft Defender Antivirus masuk Passive mode.

  

```text

THIRD-PARTY ANTIVIRUS

          ≠

AUTOMATIC PASSIVE MODE

ON EVERY WINDOWS SERVER

```

  

Status efektif Defender pada sebuah Windows Server dipengaruhi oleh kombinasi faktor berikut, bukan oleh satu faktor tunggal:

  

* Versi Windows Server yang digunakan.

* Keberadaan antivirus pihak ketiga yang aktif.

* Status onboarding server ke Microsoft Defender for Endpoint.

* Konfigurasi `ForceDefenderPassiveMode`.

* Status Tamper Protection.

* Policy atau mekanisme pengelolaan lain yang berlaku pada server tersebut.

  

| Kondisi | Penjelasan |

|---|---|

| Active mode | Defender berjalan sebagai primary antivirus dan melakukan remediation |

| Passive mode | Defender berjalan tetapi tidak melakukan remediation; dapat dipicu oleh kombinasi antivirus pihak ketiga aktif, status onboarding Defender for Endpoint, dan/atau `ForceDefenderPassiveMode`; Defender tetap dapat memberi telemetry |

| Tidak tersedia/tidak berjalan | Defender dinonaktifkan, tidak terpasang, atau service berhenti |

| Antivirus pihak ketiga terdeteksi | Tidak boleh langsung disimpulkan sebagai Passive mode; periksa versi Windows Server, status onboarding Defender for Endpoint, dan `ForceDefenderPassiveMode` sebelum mengambil kesimpulan; menonaktifkan antivirus pihak ketiga tanpa arahan objective berisiko mengubah kondisi lomba |

| Setting lokal terkunci GPO | Toggle pada Windows Security tampak ada namun tidak dapat diubah (greyed out) |

| Configured vs effective setting | Nilai yang tampil di GUI lokal belum tentu nilai yang benar-benar diberlakukan sistem |

  

```text

VISIBLE SETTING

       ≠

EFFECTIVE PROTECTION

```

  

Alur pemeriksaan status Defender yang wajib diikuti sebelum mengambil kesimpulan atau tindakan apa pun:

  

```text

CHECK SERVER VERSION

        ↓

CHECK THIRD-PARTY AV

        ↓

CHECK DEFENDER FOR ENDPOINT ONBOARDING

        ↓

CHECK ForceDefenderPassiveMode

        ↓

CHECK AMRunningMode

        ↓

IDENTIFY EFFECTIVE STATE

```

  

> Larangan: jangan berasumsi Defender sudah Passive hanya karena antivirus pihak ketiga terpasang. Jangan pula memaksa Active mode sebelum primary antivirus dan objective lomba teridentifikasi. Mengubah primary antivirus tanpa instruksi dapat dianggap sebagai perubahan di luar scope penilaian.

  

Verifikasi mode dilakukan melalui Windows Security bila tersedia, dan `Get-MpComputerStatus` sebagai alat konfirmasi — fokuskan pemeriksaan pada field atau status `AMRunningMode` bila tersedia sebagai penentu status efektif.

  

---

  

## WGUI-5.5 Core Protection Settings

  

Untuk setiap setting berikut, evaluasi dilakukan melalui Windows Security → Virus & threat protection → Manage settings, atau melalui GPO bila terkunci.

  

* Real-time protection biasanya dapat dilihat melalui Windows Security jika GUI tersedia.

* Cloud-delivered protection biasanya dapat dilihat melalui Windows Security jika GUI tersedia.

* Behavior monitoring tidak selalu memiliki toggle tersendiri pada Windows Security. Statusnya harus diperiksa melalui Local/Domain Group Policy dan diverifikasi melalui effective policy atau cmdlet verifikasi.

* Jangan mengarang toggle Behavior Monitoring pada Windows Security jika tidak tampil pada GUI yang sedang diaudit.

  

| Setting | Fungsi | Risiko bila dimatikan | Potensi compatibility issue |

|---|---|---|---|

| Real-time protection | Memindai file saat diakses/dieksekusi | Malware dapat berjalan tanpa terdeteksi | Rendah; jarang konflik pada kondisi normal |

| Behavior monitoring | Mendeteksi pola perilaku mencurigakan proses | Serangan fileless/living-off-the-land lebih sulit terdeteksi | Rendah |

| Cloud-delivered protection | Verifikasi reputasi file secara cepat via cloud | Deteksi terhadap ancaman baru melambat | Memerlukan konektivitas keluar |

| Automatic sample submission | Mengirim sample mencurigakan untuk analisis | Deteksi ancaman baru kurang optimal secara global | Pertimbangan kebijakan data pada lingkungan tertentu |

| Scan downloaded files and attachments | Memindai file hasil unduhan/lampiran | File berbahaya dari unduhan tidak segera diperiksa | Rendah |

| Script scanning | Memindai eksekusi script melalui AMSI | Script berbahaya (PowerShell dsb.) lolos deteksi | Dapat memperlambat script legitimate yang kompleks |

| Archive scanning | Memindai isi file arsip | Payload dalam arsip tidak terdeteksi | Menambah waktu scan |

| Removable drive scanning | Memindai media USB/removable | Ancaman dari USB tidak terdeteksi | Menambah waktu saat drive dipasang |

| Email scanning awareness | Kesadaran bahwa email client/webmail turut diawasi jalur eksekusinya | Lampiran berbahaya lebih mudah dieksekusi | Terkait juga dengan ASR rule email |

| PUA protection | Memblokir Potentially Unwanted Application | Aplikasi bundel/adware tidak terblokir | Dapat memblokir installer bundling yang dianggap sah oleh user |

  

Untuk setiap perubahan setting pada tabel di atas, ikuti pola berikut:

  

* Lokasi GUI/GPO: dicatat sesuai versi Windows Server yang diaudit.

* Verification: Windows Security status, `Get-MpPreference`, atau Group Policy Results.

* Rollback: kembalikan nilai ke kondisi baseline yang telah dicatat.

* Evidence: screenshot before/after, hasil scan, dan entri Event Viewer terkait bila ada.

  

---

  

## WGUI-5.6 Security Intelligence dan Updates

  

| Istilah | Penjelasan |

|---|---|

| Security intelligence | Basis data signature/model deteksi yang diperbarui berkala |

| Antivirus engine | Komponen yang menjalankan proses scanning dan deteksi |

| Defender platform | Kerangka aplikasi Defender itu sendiri (terpisah dari intelligence dan engine) |

  

Versi dan tanggal update dapat dilihat melalui Windows Security → Virus & threat protection → Virus & threat protection updates.

  

Poin penting:

  

* Defender yang aktif dengan security intelligence lama tetap berisiko tinggi, karena deteksi terhadap ancaman terbaru tidak akan berjalan optimal.

* Status "No current threats" tidak boleh diartikan sebagai kondisi aman jika intelligence version belum diverifikasi.

* Kegagalan update harus ditelusuri secara berurutan: konektivitas jaringan, status service `WinDefend`, kebijakan (GPO yang mungkin membatasi sumber update), dan Event Viewer untuk error spesifik.

  

---

  

## WGUI-5.7 Scan Strategy

  

```text

QUICK = AREA BERISIKO TINGGI

FULL = SELURUH AREA YANG DIDUKUNG

CUSTOM = TARGET SPESIFIK

```

  

| Jenis scan | Kapan digunakan | Dampak performa |

|---|---|---|

| Quick scan | Pemeriksaan rutin harian, verifikasi cepat setelah perubahan setting | Rendah |

| Full scan | Setelah insiden signifikan atau sebelum menyerahkan hasil audit | Tinggi, dapat memakan waktu lama |

| Custom scan | Menargetkan folder/drive tertentu yang dicurigai | Sedang, tergantung ukuran target |

| Microsoft Defender Offline scan | Awareness saja; digunakan bila ancaman persisten tidak dapat dibersihkan dalam kondisi normal, hanya jika didukung lingkungan | Tinggi, memerlukan restart |

| Scheduled scan | Verifikasi melalui Task Scheduler bahwa scan rutin berjalan sesuai jadwal | Tergantung jadwal dan beban server |

  

Scan sebelum dan sesudah remediation digunakan untuk memastikan tindakan (quarantine/remove) benar-benar menghilangkan ancaman dan tidak meninggalkan sisa komponen berbahaya.

  

---

  

## WGUI-5.8 Protection History, Quarantine, dan Remediation

  

```text

READ DETECTION

   ↓

RECORD PATH + TIME + SEVERITY

   ↓

CHECK DEPENDENCY

   ↓

QUARANTINE

   ↓

SCAN

   ↓

VERIFY

```

  

| Istilah | Penjelasan |

|---|---|

| Current threats | Ancaman yang sedang terdeteksi aktif pada saat ini |

| Protection History | Riwayat seluruh deteksi dan tindakan Defender |

| Detection | Identifikasi awal terhadap objek mencurigakan |

| Quarantine | Mengisolasi file agar tidak dapat dieksekusi tanpa menghapus permanen |

| Remove | Menghapus objek terdeteksi secara permanen |

| Allow on device | Mengizinkan objek tertentu berjalan meskipun terdeteksi |

| Restore | Mengembalikan file dari quarantine ke lokasi asal |

| Remediation failed | Tindakan otomatis gagal dilakukan dan memerlukan investigasi manual |

  

> Restore atau Allow on device hanya boleh dilakukan untuk file yang telah terbukti sah (false positive yang terverifikasi) dan sesuai dengan objective lomba. Quarantine selalu lebih aman dibandingkan Allow atau Restore ketika status file belum jelas.

  

Lokasi Event Viewer terkait:

  

```text

Applications and Services Logs

→ Microsoft

→ Windows

→ Windows Defender

→ Operational

```

  

| Event ID | Arti |

|---|---|

| 1116 | Deteksi ancaman (malware detected) |

| 1117 | Tindakan/remediation dilakukan |

| 1119 | Remediation gagal (critical failure) |

  

---

  

## WGUI-5.9 Exclusion Audit

  

Jenis exclusion yang perlu diaudit:

  

* File

* Folder

* File extension

* Process

* Awareness terhadap file yang dibuka oleh process yang dikecualikan

  

Kriteria audit untuk setiap exclusion:

  

| Kriteria | Pertanyaan audit |

|---|---|

| Authorized purpose | Apakah exclusion ini memiliki alasan bisnis/teknis yang tercatat? |

| Path terlalu luas | Apakah exclusion mencakup folder yang jauh lebih luas dari kebutuhan? |

| Root drive exclusion | Apakah exclusion mencakup seluruh drive (misalnya `C:\`)? |

| Temporary/download folder | Apakah exclusion menyasar folder yang sering menjadi titik masuk file berbahaya? |

| User-writable folder | Apakah folder tersebut dapat ditulis oleh user biasa? |

| Service dependency | Apakah ada service yang bergantung pada exclusion ini? |

| Application vendor requirement | Apakah exclusion direkomendasikan resmi oleh vendor aplikasi? |

| Source local atau GPO | Apakah exclusion berasal dari local policy atau domain GPO? |

  

```text

EXCLUSION =

LESS SCANNING

+

MORE RISK

```

  

Prosedur audit yang aman: nonaktifkan atau hapus exclusion satu per satu, kemudian uji fungsi aplikasi terkait sebelum melanjutkan ke exclusion berikutnya. Jangan menghapus banyak exclusion sekaligus karena akan menyulitkan identifikasi penyebab bila terjadi gangguan aplikasi.

  

---

  

## WGUI-5.10 Tamper Protection

  

* **Tujuan**: mencegah perubahan tidak sah terhadap setting Defender inti (real-time protection, cloud protection, dan lainnya) baik oleh malware maupun oleh perubahan manual yang tidak melalui jalur resmi.

* **Sumber pengelolaan**: Tamper Protection tidak dikelola langsung melalui Group Policy biasa.

  * Pada perangkat individual yang tidak dikelola organisasi, statusnya dapat tersedia melalui Windows Security jika didukung.

  * Pada lingkungan organisasi, Tamper Protection dapat dikelola melalui Microsoft Defender portal, Intune, atau Configuration Manager.

  * Jangan mengasumsikan Microsoft Defender portal, Intune, atau Configuration Manager tersedia dalam lingkungan lomba.

* **Hubungan dengan GPO**: GPO tetap dapat mengatur banyak setting Defender lain, tetapi perubahan terhadap setting yang dilindungi oleh Tamper Protection dapat diabaikan meskipun GPO terlihat sudah dikonfigurasi dengan benar.

* **Toggle greyed out**: toggle yang greyed out pada Windows Security tidak selalu berarti error; dapat berarti setting dikunci oleh Tamper Protection atau oleh mekanisme pengelolaan lain.

* **Keterbatasan visibilitas**: pada beberapa versi Windows Server, status Tamper Protection mungkin tidak terlihat lengkap melalui Windows Security.

  

```text

GPO CONFIGURATION

       +

TAMPER PROTECTION ACTIVE

       =

PROTECTED CHANGES MAY BE IGNORED

```

  

* **Larangan**: Tamper Protection tidak boleh dimatikan sekadar untuk mempermudah perubahan setting lain. Tindakan ini mengurangi postur keamanan endpoint secara keseluruhan dan bertentangan dengan prinsip minimal-attack-surface yang dinilai dalam kompetisi.

  

> Jangan men-disable Tamper Protection hanya agar setting lain dapat diubah.

  

* **Dokumentasi status**: catat status Tamper Protection (Windows Security bila tersedia, atau `Get-MpComputerStatus`) sebagai bagian dari baseline dan evidence.

* **Verifikasi opsional**:

  

```text

Get-MpComputerStatus

```

  

Nilai `IsTamperProtected` pada hasil cmdlet ini dapat digunakan sebagai konfirmasi status Tamper Protection jika GUI tidak memberikan informasi yang jelas.

  

* **Troubleshooting saat policy tidak dapat diedit**: periksa sumber policy melalui Group Policy Results, periksa apakah setting yang relevan dilindungi Tamper Protection, dan jangan mencoba memaksa perubahan melalui penonaktifan Tamper Protection.

  

---

  

## WGUI-5.11 Attack Surface Reduction Rules

  

ASR rules mengendalikan **perilaku berisiko** pada level proses, script, dan aplikasi office/email, bukan sekadar mengenali file berdasarkan signature semata.

  

> Dukungan ASR rule dapat berbeda berdasarkan versi Windows Server, platform Defender, aplikasi yang terpasang, dan jenis workload server.

  

```text

RULE EXISTS IN TEMPLATE

          ≠

RULE SUPPORTED AND EFFECTIVE

```

  

Checklist wajib sebelum menerapkan sebuah rule:

  

- [ ] Nama rule resmi sudah diverifikasi.

- [ ] Rule didukung pada versi Windows Server yang digunakan.

- [ ] Defender platform memenuhi kebutuhan rule.

- [ ] Workload server sudah diidentifikasi.

- [ ] Aplikasi yang mungkin terdampak sudah dicatat.

- [ ] Rule diuji dalam Audit mode.

- [ ] Event Audit sudah ditinjau.

- [ ] Rollback tersedia.

  

Mode yang tersedia untuk setiap rule:

  

| Mode | Penjelasan |

|---|---|

| Not configured | Rule tidak diatur; mengikuti default Defender |

| Disabled | Rule dimatikan secara eksplisit |

| Audit | Perilaku dicatat ke Event Viewer tanpa diblokir |

| Warn | User diberi peringatan dan dapat memilih melanjutkan (bila rule mendukung Warn) |

| Block | Perilaku benar-benar diblokir |

  

```text

AUDIT

  ↓

REVIEW EVENTS

  ↓

CHECK FALSE POSITIVE

  ↓

ADD MINIMAL EXCLUSION IF REQUIRED

  ↓

BLOCK

```

  

> Tidak semua ASR rule mendukung Warn mode. Beberapa rule (misalnya rule terkait LSASS dan code injection Office) tidak memiliki dukungan Warn mode dan langsung berpindah dari Audit ke Block.

  

### Tabel Rule Prioritas (Standard Protection Rules)

  

Rule berikut adalah kelompok *standard protection rules* yang direkomendasikan sebagai prioritas awal karena cakupan proteksinya tinggi dengan risiko compatibility yang secara umum terkendali.

  

| Official rule name | Perilaku yang dibatasi | Risiko compatibility | Initial recommended testing mode | Evidence event |

|---|---|---|---|---|

| Block abuse of exploited vulnerable signed drivers | Mencegah aplikasi menyimpan driver bertanda tangan yang rentan | Rendah | Audit | 1122 (Audit) / 1121 (Block) |

| Block credential stealing from the Windows local security authority subsystem (lsass.exe) | Mencegah proses mengakses memori LSASS untuk mencuri kredensial | Sedang; dapat menghasilkan noise dari proses yang mengakses LSASS tanpa niat jahat | Audit (tidak mendukung Warn) | 1122 (Audit) / 1121 (Block) |

| Block persistence through WMI event subscription | Mencegah malware memakai WMI event subscription untuk persistence | Sedang; berdampak pada tooling manajemen berbasis WMI | Audit | 1122 (Audit) / 1121 (Block) |

  

### Tabel Rule Prioritas (Other ASR Rules — dipilih yang relevan untuk Windows Server)

  

| Official rule name | Perilaku yang dibatasi | Risiko compatibility | Initial recommended testing mode | Evidence event |

|---|---|---|---|---|

| Block executable files from running unless they meet a prevalence, age, or trusted list criterion | Mencegah eksekusi file yang belum dikenal/tidak lazim | Tinggi; dapat memblokir installer atau tool internal yang belum lazim | Audit | 1122 (Audit) / 1121 (Block) |

| Block execution of potentially obfuscated scripts | Mendeteksi script dengan properti obfuscation mencurigakan | Sedang; script legitimate yang kompleks dapat terpicu | Audit | 1122 (Audit) / 1121 (Block) |

| Block process creations originating from PSExec and WMI commands | Mencegah proses yang dibuat melalui PsExec dan WMI | Tinggi pada lingkungan yang bergantung pada Configuration Manager/WMI tooling | Audit | 1122 (Audit) / 1121 (Block) |

| Block untrusted and unsigned processes that run from USB | Mencegah eksekusi file tidak bertanda tangan dari USB/removable drive | Rendah pada server (jarang menggunakan USB) | Audit | 1122 (Audit) / 1121 (Block) |

| Block use of copied or impersonated system tools | Mencegah penggunaan file yang meniru system tool Windows | Rendah hingga sedang | Audit | 1122 (Audit) / 1121 (Block) |

| Block Webshell creation for Servers | Mencegah pembuatan web shell script pada server | Tergantung workload; lihat catatan di bawah tabel | Audit | 1122 (Audit) / 1121 (Block) |

| Use advanced protection against ransomware | Menggunakan heuristik client dan cloud untuk mendeteksi perilaku menyerupai ransomware | Sedang; file baru/belum dikenal dapat sementara diblokir | Audit | 1122 (Audit) / 1121 (Block) |

  

> **Block Webshell Creation for Servers**: rule ini ditujukan terutama untuk Windows Server yang menjalankan Microsoft Exchange. Jangan menjadikannya prioritas pada server non-Exchange tanpa objective dan workload yang relevan.

  

> Untuk rule lain: jangan membuat klaim dukungan universal pada seluruh versi Windows Server atau seluruh workload. Jangan mengarang GUID rule; jika GUID tidak dapat dipastikan, jangan dicantumkan. Audit mode tetap wajib dilalui sebelum Block pada setiap rule. Event 1121 atau 1122 hanya akan muncul jika rule tersebut didukung, efektif pada sistem yang diaudit, dan aktivitas yang sesuai benar-benar terjadi — event yang tidak muncul tidak boleh langsung disimpulkan sebagai rule gagal.

  

> Rule terkait Office (child process, executable content, process injection, Win32 API dari macro) dan rule terkait email/JavaScript-VBScript tersedia pada Windows Server namun relevansinya lebih tinggi pada workstation dengan Office terpasang. Cantumkan hanya bila lingkungan lomba benar-benar menjalankan aplikasi tersebut, dan verifikasi nama serta GUID resmi sebelum diterapkan; jika GUID tidak dapat dipastikan, jangan dicantumkan.

  

Event penting terkait ASR:

  

| Event ID | Arti |

|---|---|

| 1121 | ASR memblokir aktivitas (Block mode) |

| 1122 | ASR mencatat aktivitas dalam Audit mode |

| 1129 | User melakukan override pada Warn mode |

| 5007 | Konfigurasi ASR/Defender berubah |

  

> Larangan: jangan mengaktifkan seluruh ASR rule dalam Block mode secara sekaligus. Terapkan satu rule pada satu waktu melalui alur Audit terlebih dahulu, tinjau event yang dihasilkan, tambahkan exclusion minimal bila diperlukan, baru kemudian pindah ke Block.

  

---

  

## WGUI-5.12 Controlled Folder Access

  

Controlled Folder Access (CFA) melindungi folder terpilih (protected folders) dari perubahan yang dilakukan oleh aplikasi yang tidak berada dalam allowed apps list.

  

**Windows Security** dapat digunakan, jika tersedia, untuk:

  

* Mengaktifkan atau menonaktifkan Controlled Folder Access.

* Melihat atau menambah protected folders.

* Melihat atau menambah allowed apps.

  

**Group Policy** digunakan untuk memilih mode yang lebih spesifik, terutama:

  

* Disabled.

* Enabled atau Block.

* Audit Mode.

  

Mode Audit secara khusus dikonfigurasi melalui Local/Domain Group Policy, bukan melalui toggle sederhana pada Windows Security.

  

Poin pembahasan:

  

* Protected folders mencakup folder sistem umum dan dapat ditambah folder kustom.

* Allowed apps adalah daftar aplikasi yang diizinkan menulis ke protected folders.

* Terapkan Audit sebelum Enable/Block untuk memahami dampak terhadap aplikasi bisnis yang berjalan.

* Risiko utama: aplikasi bisnis yang sah dapat gagal menulis file jika belum ditambahkan ke allowed apps.

* Jangan memasukkan aplikasi ke allowed apps list tanpa memverifikasi path lengkap dan publisher aplikasi tersebut.

  

Event terkait CFA:

  

| Event ID | Arti |

|---|---|

| 1123 | CFA memblokir aktivitas penulisan (Block) |

| 1124 | CFA mencatat aktivitas dalam Audit mode |

| 5007 | Konfigurasi CFA/Defender berubah |

  

---

  

## WGUI-5.13 Network Protection

  

Network Protection berbeda dari Windows Firewall (WGUI-4): Firewall mengendalikan port dan arah koneksi berdasarkan aturan jaringan, sedangkan Network Protection mengendalikan koneksi keluar berdasarkan **reputasi domain atau resource** yang dituju, memanfaatkan intelligence dari cloud protection.

  

Poin ringkas:

  

* Mode yang tersedia: Audit dan Block.

* Network Protection bergantung pada cloud-delivered protection untuk evaluasi reputasi; tanpa cloud protection aktif, efektivitasnya menurun.

* Dapat dikelola secara local maupun melalui GPO.

* Event 1125 untuk Audit, Event 1126 untuk Block.

  

Materi detail konfigurasi firewall tidak diulang di modul ini; rujuk WGUI-4 untuk pengaturan port dan aturan jaringan.

  

---

  

## WGUI-5.14 Exploit Protection

  

Diakses melalui:

  

```text

Windows Security

→ App & browser control

→ Exploit protection settings

```

  

| Level | Penjelasan |

|---|---|

| System settings | Mitigasi default yang berlaku untuk seluruh sistem |

| Program settings | Mitigasi yang diatur per aplikasi/proses |

| Use default | Program mengikuti mitigasi pada System settings |

| Override system settings | Program menggunakan mitigasi kustom yang berbeda dari default sistem |

  

Poin penting:

  

* Pertahankan konfigurasi default apabila objective lomba tidak secara eksplisit memerintahkan perubahan.

* Custom mitigation yang tidak tepat dapat menyebabkan aplikasi crash atau gagal berjalan.

* Export konfigurasi ke XML sebelum melakukan perubahan besar, sebagai titik rollback.

* Import/deployment configuration dapat memengaruhi banyak setting sekaligus sehingga berisiko lebih tinggi dibanding perubahan per aplikasi.

* Untuk perubahan pada satu aplikasi saja, rollback manual (mengembalikan ke Use default) lebih aman dan lebih mudah dilacak dibandingkan import ulang konfigurasi penuh.

  

---

  

## WGUI-5.15 Verification, Evidence, dan Rollback

  

```text

BEFORE SCREENSHOT

      ↓

CHANGE

      ↓

CHECK EFFECTIVE POLICY

      ↓

SAFE FUNCTION TEST

      ↓

CHECK EVENT LOG

      ↓

AFTER SCREENSHOT

```

  

Evidence minimal yang wajib dikumpulkan untuk setiap perubahan:

  

- [ ] Baseline Defender sebelum perubahan

- [ ] Sumber policy yang berlaku (local/GPO/terpusat)

- [ ] Perbandingan setting before/after

- [ ] Versi security intelligence pada saat pengujian

- [ ] Hasil scan terkait (jika relevan)

- [ ] Entri Protection History terkait (jika relevan)

- [ ] Entri Event Viewer yang relevan

- [ ] Bukti fungsi aplikasi tetap berjalan normal setelah perubahan

- [ ] Event Audit atau Block dari ASR/CFA bila perubahan menyangkut kedua fitur tersebut

  

---

  

## WGUI-5.16 Troubleshooting Decision Tree

  

```text

DEFENDER ISSUE

   ↓

CHECK OS + GUI AVAILABILITY

   ↓

CHECK ACTIVE/PASSIVE MODE

   ↓

CHECK THIRD-PARTY AV

   ↓

CHECK SERVICE

   ↓

CHECK LOCAL VS GPO POLICY

   ↓

CHECK TAMPER PROTECTION

   ↓

CHECK UPDATE STATUS

   ↓

CHECK EXCLUSIONS

   ↓

CHECK EVENT LOG

```

  

Penerapan decision tree pada kasus umum:

  

| Kasus | Langkah investigasi utama |

|---|---|

| Real-time protection tidak bisa diaktifkan | Cek Tamper Protection, cek GPO yang mungkin menonaktifkan, cek antivirus pihak ketiga |

| Setting greyed out | Cek sumber policy (local vs GPO), cek Tamper Protection, cek Group Policy Results |

| Defender aktif tetapi tidak mendeteksi | Cek mode Active/Passive, cek intelligence version, cek exclusion yang terlalu luas |

| Intelligence tidak update | Cek konektivitas, cek service `WinDefend`, cek kebijakan sumber update, cek Event Viewer |

| Scan gagal | Cek service, cek disk space, cek Event Viewer untuk error spesifik |

| File terus terdeteksi | Cek apakah file benar-benar berbahaya, cek exclusion yang salah arah, cek apakah remediation gagal (Event 1119) |

| Aplikasi sah diblokir ASR | Cek Event 1121/1122, verifikasi aplikasi, uji ulang dalam Audit mode, tambahkan exclusion minimal bila perlu |

| Aplikasi tidak bisa menulis karena CFA | Cek Event 1123/1124, verifikasi path dan publisher aplikasi sebelum menambahkan ke allowed apps |

| Setting berubah kembali setelah diedit | Cek apakah setting dikelola GPO yang menimpa perubahan lokal, cek jadwal refresh Group Policy |

  

### Defender Passive atau Tidak Aktif

  

```text

CHECK WINDOWS SERVER VERSION

   ↓

CHECK THIRD-PARTY AV

   ↓

CHECK DEFENDER FOR ENDPOINT ONBOARDING

   ↓

CHECK ForceDefenderPassiveMode

   ↓

CHECK AMRunningMode

   ↓

CHECK TAMPER PROTECTION

```

  

### Setting Berubah tetapi Tidak Efektif

  

```text

CHECK CONFIGURED VALUE

   ↓

CHECK LOCAL POLICY

   ↓

CHECK DOMAIN GPO

   ↓

CHECK GROUP POLICY RESULTS

   ↓

CHECK TAMPER PROTECTION

   ↓

CHECK EFFECTIVE DEFENDER STATE

```

  

### ASR Tidak Menghasilkan Event

  

```text

CHECK SERVER VERSION

   ↓

CHECK RULE SUPPORT

   ↓

CHECK DEFENDER PLATFORM

   ↓

CHECK RULE MODE

   ↓

CHECK EFFECTIVE POLICY

   ↓

CHECK TEST ACTIVITY

   ↓

CHECK DEFENDER OPERATIONAL LOG

```

  

### CFA Audit Tidak Tercatat

  

```text

CHECK CFA POLICY MODE

   ↓

CHECK EFFECTIVE POLICY

   ↓

CHECK PROTECTED FOLDER

   ↓

CHECK TEST APPLICATION

   ↓

CHECK EVENT 1124

```

  

---

  

## WGUI-5.17 Closed-Book Training

  

### Skenario 1 — Real-time protection Off

* Apa yang dicek: status Real-time protection di Windows Security, sumber policy, status Tamper Protection.

* Risiko: file berbahaya dapat berjalan tanpa terdeteksi.

* Perubahan aman: aktifkan kembali melalui GUI/GPO sesuai sumber policy yang berlaku, satu langkah.

* Verifikasi: `Get-MpComputerStatus`, Windows Security status.

* Rollback: tidak diperlukan bila hasil sesuai baseline aman; jika berasal dari GPO, kembalikan ke policy yang benar.

* Evidence: before/after status, Event Viewer terkait perubahan konfigurasi.

  

### Skenario 2 — Defender berada di Passive mode

* Apa yang dicek: versi Windows Server, keberadaan antivirus pihak ketiga, status onboarding Microsoft Defender for Endpoint, konfigurasi `ForceDefenderPassiveMode`, status `AMRunningMode`, dan status Tamper Protection; objective lomba terkait primary antivirus.

* Risiko: remediation otomatis Defender tidak berjalan.

* Perubahan aman: hanya mengubah primary antivirus bila memang diperintahkan objective; jika tidak, dokumentasikan kondisi sebagai temuan. Jangan berasumsi Passive mode hanya karena antivirus pihak ketiga terpasang.

* Verifikasi: `Get-MpComputerStatus`, fokus pada `AMRunningMode`.

* Rollback: kembalikan ke kondisi awal bila perubahan ternyata di luar scope.

* Evidence: versi Windows Server, daftar antivirus terpasang, status onboarding Defender for Endpoint, status mode Defender.

  

### Skenario 3 — Security intelligence sangat lama

* Apa yang dicek: tanggal dan versi intelligence, status konektivitas, status service update.

* Risiko: deteksi terhadap ancaman baru tidak optimal.

* Perubahan aman: jalankan update melalui Windows Security, periksa hasil.

* Verifikasi: tanggal/versi intelligence setelah update.

* Rollback: tidak relevan untuk proses update yang berhasil.

* Evidence: before/after versi intelligence, log update.

  

### Skenario 4 — Exclusion seluruh drive ditemukan

* Apa yang dicek: alasan/authorized purpose exclusion, dampak bila dihapus, dependency aplikasi.

* Risiko: hampir seluruh isi drive tidak dipindai.

* Perubahan aman: hapus exclusion secara bertahap, uji aplikasi terkait setelah setiap perubahan.

* Verifikasi: `Get-MpPreference` untuk daftar exclusion terkini.

* Rollback: kembalikan exclusion sempit bila terbukti dibutuhkan aplikasi tertentu, bukan mengembalikan exclusion seluruh drive.

* Evidence: daftar exclusion before/after, hasil scan setelah exclusion dipersempit.

  

### Skenario 5 — File dikarantina tetapi ternyata dibutuhkan aplikasi

* Apa yang dicek: path file, hasil analisis (apakah benar false positive), dependency aplikasi.

* Risiko: Restore sembarangan dapat mengembalikan ancaman nyata.

* Perubahan aman: Restore hanya setelah file terverifikasi sah dan sesuai objective; jika tidak yakin, biarkan tetap quarantine.

* Verifikasi: Protection History, hasil scan ulang setelah Restore.

* Rollback: quarantine kembali bila setelah Restore ditemukan indikasi mencurigakan.

* Evidence: detail deteksi, alasan Restore, hasil scan pasca-Restore.

  

### Skenario 6 — ASR Block menyebabkan aplikasi bisnis gagal

* Apa yang dicek: Event 1121, rule spesifik yang memblokir, path aplikasi.

* Risiko: gangguan operasional aplikasi bisnis.

* Perubahan aman: turunkan sementara rule terkait ke Audit untuk konfirmasi, tambahkan exclusion minimal dan spesifik bila memang diperlukan, baru kembalikan ke Block.

* Verifikasi: Event 1122 (Audit) menunjukkan aktivitas tanpa diblokir, aplikasi berjalan normal.

* Rollback: kembalikan rule ke Block setelah exclusion minimal diterapkan dan diverifikasi.

* Evidence: Event 1121 sebelum perubahan, Event 1122 setelah Audit, konfigurasi exclusion, konfirmasi aplikasi berjalan.

  

### Skenario 7 — CFA memblokir aplikasi sah

* Apa yang dicek: Event 1123, path dan publisher aplikasi.

* Risiko: aplikasi tidak dapat menulis file yang dibutuhkan.

* Perubahan aman: verifikasi path dan publisher, tambahkan ke allowed apps hanya setelah verifikasi.

* Verifikasi: aplikasi dapat menulis normal setelah ditambahkan, Event 1124/1123 sesuai kondisi.

* Rollback: hapus dari allowed apps bila ternyata tidak sesuai objective atau mencurigakan.

* Evidence: detail aplikasi yang ditambahkan, hasil pengujian fungsi tulis.

  

### Skenario 8 — Setting lokal tidak berlaku karena GPO

* Apa yang dicek: Group Policy Results/`rsop.msc`, sumber policy yang menang.

* Risiko: perubahan lokal dianggap berhasil padahal tidak efektif.

* Perubahan aman: ubah pada level GPO yang sesuai, bukan memaksa setting lokal.

* Verifikasi: effective setting melalui Group Policy Results setelah refresh policy.

* Rollback: kembalikan GPO ke nilai semula bila perubahan di luar scope.

* Evidence: hasil Group Policy Results before/after, identifikasi GPO yang mengatur setting tersebut.

  

### Skenario 9 — Remediation failure muncul

* Apa yang dicek: Event 1119, path file, alasan kegagalan (permission, file locked, dan sejenisnya).

* Risiko: ancaman tetap berada di sistem meski tampak tertangani.

* Perubahan aman: investigasi penyebab kegagalan, jalankan scan ulang, lakukan remediation manual dengan hati-hati bila diperlukan.

* Verifikasi: scan ulang menunjukkan status bersih, Event 1117 pada percobaan berikutnya.

* Rollback: tidak relevan; fokus pada penyelesaian remediation.

* Evidence: Event 1119 awal, langkah investigasi, Event 1117 setelah remediation berhasil.

  

### Skenario 10 — Custom Exploit Protection tidak diketahui tujuannya

* Apa yang dicek: apakah setting merupakan Override system settings, aplikasi yang terdampak, apakah ada dokumentasi tujuan perubahan.

* Risiko: mitigasi kustom yang tidak dipahami dapat menyebabkan crash atau justru melemahkan proteksi.

* Perubahan aman: export konfigurasi saat ini ke XML sebagai baseline, kembalikan ke Use default bila tujuan tidak dapat diverifikasi dan objective tidak memerintahkan konfigurasi tersebut.

* Verifikasi: aplikasi tetap berjalan normal setelah dikembalikan ke default.

* Rollback: import kembali XML baseline bila pengembalian ke default ternyata tidak sesuai objective.

* Evidence: XML export sebelum perubahan, hasil pengujian aplikasi setelah perubahan.

  

### Skenario 11 — ASR Rule Tidak Menghasilkan Event

* Apa yang dicek: dukungan rule pada versi Windows Server yang digunakan, effective policy, Defender platform, mode rule (Audit atau Block), aktivitas uji aman yang relevan, dan Event Viewer Operational log.

* Risiko: rule dapat tampak tidak berfungsi padahal sebenarnya tidak didukung pada versi tersebut, tidak efektif karena tertimpa policy lain, atau aktivitas uji belum benar-benar memicu kondisi rule.

* Perubahan aman: verifikasi seluruh rantai compatibility dan effective policy sebelum mengubah konfigurasi apa pun; jangan langsung menyimpulkan rule gagal atau sistem sudah aman.

* Verifikasi: Group Policy Results untuk effective policy, Event Viewer Operational log untuk Event 1121/1122.

* Rollback: tidak relevan bila rule ternyata memang belum didukung atau belum efektif; dokumentasikan temuan sebagai bagian dari evidence.

* Evidence: versi Windows Server, hasil compatibility check, hasil effective policy, aktivitas uji yang dilakukan, hasil Event Viewer.

  

### Skenario 12 — Tamper Protection Mengabaikan Perubahan

* Apa yang dicek: apakah GPO terlihat sudah dikonfigurasi dengan benar, apakah setting yang dilindungi tetap tidak berubah, status `IsTamperProtected`.

* Risiko: perubahan dianggap berhasil diterapkan padahal Tamper Protection mengabaikan perubahan tersebut, sehingga postur keamanan sebenarnya tidak sesuai laporan.

* Perubahan aman: dokumentasikan policy source dan effective state; jangan men-disable Tamper Protection untuk memaksa perubahan diterapkan.

* Verifikasi: `Get-MpComputerStatus` (`IsTamperProtected`), Group Policy Results.

* Rollback: tidak relevan; fokus pada dokumentasi policy source dan effective state yang benar.

* Evidence: konfigurasi GPO yang terlihat, hasil `IsTamperProtected`, effective state Defender.

  

---

  

## WGUI-5.18 GUI Practical Lab

  

### Lab 1 — Audit Defender Baseline

* Scenario: Server belum pernah diaudit status Defender-nya.

* Objective: mendapatkan baseline lengkap sesuai checklist WGUI-5.3, dengan membedakan secara eksplisit antara feature terpasang, service berjalan, Active/Passive mode, effective state, dan keberadaan antivirus pihak ketiga.

* Safety Check: pastikan hanya melakukan pembacaan, tidak ada perubahan setting.

* GUI Steps: periksa Server Manager/Add Roles and Features untuk memastikan feature Windows Defender Antivirus terpasang; buka `services.msc` untuk status service `WinDefend`; buka Windows Security → Virus & threat protection untuk mode Active/Passive dan status lain yang tersedia; catat antivirus pihak ketiga yang terdeteksi.

* Verification: `Get-MpComputerStatus` untuk cross-check, dengan perhatian khusus pada `AMRunningMode` sebagai penentu effective state.

* Rollback: tidak diperlukan, lab bersifat observasi.

* Evidence: screenshot seluruh status baseline, dengan pemisahan jelas antara feature terpasang, service berjalan, mode Active/Passive, effective state, dan antivirus pihak ketiga.

  

### Lab 2 — Verifikasi Core Protection dan Intelligence

* Scenario: memastikan komponen inti protection dan security intelligence dalam kondisi aktif dan terkini.

* Objective: mengonfirmasi status Real-time protection, Cloud-delivered protection, Behavior Monitoring, dan intelligence melalui jalur yang benar.

* Safety Check: catat kondisi awal sebelum menyentuh toggle atau setting apa pun.

* GUI Steps:

  1. Buka Windows Security → Virus & threat protection → Manage settings.

  2. Catat status Real-time protection.

  3. Catat status Cloud-delivered protection.

  4. Buka Local Group Policy atau Domain GPO.

  5. Masuk ke policy Microsoft Defender Antivirus → Real-time Protection.

  6. Periksa konfigurasi Behavior Monitoring.

  7. Buka Virus & threat protection updates.

  8. Catat versi dan tanggal security intelligence.

* Verification: gunakan Group Policy Results untuk menentukan effective policy; gunakan `Get-MpComputerStatus` atau `Get-MpPreference` hanya sebagai konfirmasi; jangan menganggap tampilan local policy otomatis efektif.

* Rollback: kembalikan setting ke baseline sesuai sumber policy yang benar.

* Evidence: status before/after, versi intelligence, hasil Group Policy Results.

  

### Lab 3 — Menjalankan Quick Scan dan Membaca Hasil

* Scenario: memverifikasi bahwa scan berjalan dan hasilnya dapat dibaca dengan benar.

* Objective: menjalankan Quick scan dan menginterpretasikan hasilnya.

* Safety Check: pastikan tidak ada proses kritikal yang terganggu oleh beban scan.

* GUI Steps: Windows Security → Virus & threat protection → Quick scan; tunggu hasil.

* Verification: hasil scan pada Protection History.

* Rollback: tidak diperlukan.

* Evidence: hasil scan, waktu eksekusi, status akhir.

  

### Lab 4 — Audit Protection History dan Quarantine Secara Aman

* Scenario: terdapat entri lama pada Protection History yang perlu ditinjau.

* Objective: membedakan entri yang aman diabaikan dan yang memerlukan tindakan lanjut.

* Safety Check: jangan melakukan Restore tanpa verifikasi menyeluruh.

* GUI Steps: Windows Security → Virus & threat protection → Protection history; tinjau setiap entri, path, dan severity.

* Verification: Event Viewer 1116/1117/1119 sesuai entri terkait.

* Rollback: tidak relevan untuk aktivitas audit murni.

* Evidence: daftar entri Protection History beserta status tindakan.

  

### Lab 5 — Audit serta Mempersempit Exclusion

* Scenario: ditemukan beberapa exclusion yang tidak terdokumentasi.

* Objective: mempersempit atau menghapus exclusion yang tidak dapat dijustifikasi.

* Safety Check: uji aplikasi terkait sebelum dan sesudah setiap perubahan exclusion.

* GUI Steps: Windows Security → Virus & threat protection → Manage settings → Exclusions; tinjau, hapus/persempit satu per satu.

* Verification: `Get-MpPreference`, uji fungsi aplikasi.

* Rollback: kembalikan exclusion spesifik bila aplikasi terbukti bergantung padanya.

* Evidence: daftar exclusion before/after, hasil uji aplikasi.

  

### Lab 6 — Mengaktifkan Satu ASR Rule dalam Audit Mode dan Membaca Event Viewer

* Scenario: rule ASR prioritas belum pernah diuji pada lingkungan lomba.

* Objective: mengaktifkan satu rule dalam Audit mode dan membaca event yang dihasilkan.

* Safety Check: pastikan Defender dalam Active mode dan cloud protection aktif untuk rule yang membutuhkannya.

* GUI Steps:

  1. Identifikasi versi Windows Server yang digunakan.

  2. Verifikasi rule yang dipilih didukung pada versi tersebut.

  3. Catat workload server yang berjalan.

  4. Aktifkan satu rule ASR melalui GPO/Local Group Policy dalam mode Audit.

  5. Verifikasi effective policy melalui Group Policy Results.

  6. Jalankan aktivitas uji aman yang disediakan lingkungan lab.

  7. Cari Event 1122 pada Event Viewer Operational log.

* Verification: Event Viewer Operational log, Event 1122; effective policy melalui Group Policy Results.

* Rollback: kembalikan rule ke Not configured bila lab bersifat latihan murni.

* Evidence: versi Windows Server, hasil compatibility check, konfigurasi rule, entri Event 1122, kesimpulan false positive/true positive.

  

### Lab 7 — Controlled Folder Access Audit

* Scenario: memastikan dampak CFA terhadap aplikasi bisnis sebelum diaktifkan penuh.

* Objective: mengaktifkan CFA dalam mode Audit melalui Group Policy dan meninjau event yang tercatat.

* Safety Check: identifikasi aplikasi yang biasa menulis ke protected folders sebelum pengujian.

* GUI Steps:

  1. Buka `gpedit.msc` atau `gpmc.msc`.

  2. Masuk ke jalur kebijakan Microsoft Defender Antivirus.

  3. Buka: `Microsoft Defender Exploit Guard → Controlled Folder Access`.

  4. Buka setting **Configure Controlled folder access**.

  5. Set ke **Enabled**.

  6. Pilih **Audit Mode**.

  7. Terapkan policy.

  8. Jalankan aktivitas uji aman menggunakan aplikasi sah yang disediakan lingkungan lab.

  9. Periksa Event Viewer pada log Microsoft Defender Operational.

* Verification: cari Event ID 1124; cocokkan waktu, aplikasi, dan folder dengan aktivitas uji; pastikan aktivitas hanya dicatat dan belum diblokir.

* Rollback: kembalikan setting ke baseline atau `Not Configured` jika lab bersifat latihan; jangan mengubah ke Block sebelum event Audit ditinjau.

* Evidence: daftar protected folders, konfigurasi Group Policy yang diterapkan, entri Event 1124, aplikasi yang teridentifikasi terdampak.

  

### Lab 8 — Export Baseline Exploit Protection

* Scenario: sebelum melakukan eksperimen mitigasi, baseline harus diamankan.

* Objective: mengekspor konfigurasi Exploit Protection saat ini ke file XML sebagai titik rollback.

* Safety Check: pastikan tidak ada perubahan setting dilakukan sebelum export selesai.

* GUI Steps: Windows Security → App & browser control → Exploit protection settings → Export settings.

* Verification: file XML berhasil dibuat dan dapat dibuka/dibaca.

* Rollback: gunakan file XML ini untuk import kembali bila diperlukan.

* Evidence: file XML baseline, waktu export, ringkasan isi konfigurasi.

  

> Peringatan: Import konfigurasi Exploit Protection dapat mengubah banyak mitigasi sekaligus. Untuk satu perubahan kecil, rollback manual lebih aman.

  

> Seluruh lab hanya menggunakan file atau aktivitas uji aman yang telah disediakan oleh lingkungan lab. Modul ini tidak menyertakan pengunduhan malware, pembuatan payload, atau pencetakan test string berbahaya dalam bentuk apa pun.

  

---

  

## MEMORY SECTION — DEFENDER COMBAT MEMORY

  

```text

MODE

 ↓

UPDATE

 ↓

REAL-TIME

 ↓

CLOUD

 ↓

EXCLUSION

 ↓

SCAN

 ↓

HISTORY

 ↓

ASR

 ↓

CFA

 ↓

EVENT

 ↓

VERIFY

```

  

```text

DEFENDER EFFECTIVE =

ACTIVE MODE

+ CURRENT INTELLIGENCE

+ REAL-TIME PROTECTION

+ MINIMAL EXCLUSIONS

+ EFFECTIVE POLICY

```

  

```text

ASR DEPLOYMENT =

AUDIT

+ REVIEW

+ MINIMAL EXCLUSION

+ BLOCK

```

  

```text

THIRD-PARTY AV

      ≠

AUTOMATIC PASSIVE MODE

```

  

```text

CONFIGURED POLICY

        ≠

EFFECTIVE PROTECTION

```

  

```text

ASR TEMPLATE AVAILABLE

          ≠

RULE SUPPORTED

```

  

```text

TAMPER PROTECTION ACTIVE

          =

PROTECTED CHANGES MAY BE IGNORED

```

  

### 10 Langkah Tempur

  

1. Identifikasi versi Windows Server dan installation type.

2. Periksa Defender feature, service, Active/Passive mode, serta antivirus pihak ketiga.

3. Jangan mengasumsikan third-party antivirus otomatis membuat Passive mode.

4. Periksa policy source dan effective policy.

5. Verifikasi Real-time, Cloud, Behavior Monitoring, dan intelligence melalui jalur yang benar.

6. Audit exclusion satu per satu.

7. Terapkan ASR hanya setelah compatibility check dan Audit mode.

8. Terapkan CFA Audit melalui Group Policy sebelum Block.

9. Jangan men-disable Tamper Protection untuk memaksa perubahan.

10. Tutup dengan safe function test, Event Viewer, rollback, dan evidence.

  

---

  

## FINAL QUALITY CONTROL

  

- [x] Output tetap Markdown.

- [x] Seluruh materi lama yang benar dipertahankan.

- [x] Tidak ada HTML, React, TSX, CSS, atau JavaScript.

- [x] GUI tetap jalur konfigurasi utama.

- [x] CLI hanya digunakan sebagai verifikasi.

- [x] Tidak ada asumsi third-party antivirus otomatis membuat Passive mode pada semua Windows Server.

- [x] Defender for Endpoint onboarding dibedakan dari Defender Antivirus biasa.

- [x] `ForceDefenderPassiveMode` diperkenalkan secara ringkas.

- [x] `AMRunningMode` digunakan untuk verifikasi mode.

- [x] Tamper Protection tidak diklaim dikelola langsung melalui GPO.

- [x] `IsTamperProtected` disebut sebagai verifikasi opsional.

- [x] Windows Security tidak diklaim mengelola individual ASR rules.

- [x] CFA Audit dikonfigurasi melalui Group Policy.

- [x] Domain membership tidak diklaim menghilangkan `gpedit.msc`.

- [x] Local policy dan effective policy dibedakan.

- [x] Behavior Monitoring tidak diklaim selalu memiliki toggle Windows Security.

- [x] Compatibility check ASR tersedia.

- [x] Block Webshell Creation for Servers dijelaskan terutama relevan untuk Exchange Server.

- [x] ASR Audit-first tetap dipertahankan.

- [x] Quarantine tetap diprioritaskan dibanding Restore atau Allow.

- [x] Verification, rollback, dan evidence tetap lengkap.

- [x] Closed-book training sudah diperbaiki.

- [x] Practical lab sudah diperbaiki.

- [x] Memory section sudah diperbarui.

- [x] Tidak ada teknik bypass, evasion, malware, payload, atau materi ofensif.

- [x] Tidak ada saran revisi lanjutan.

  

**STATUS: FINAL COMPETITION READY — LOCKED**