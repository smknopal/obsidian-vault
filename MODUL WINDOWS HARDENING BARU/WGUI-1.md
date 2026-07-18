# WGUI-1 — Local Account & Privilege Hardening

  

**Privileged Access Management (PAM) — Single-Machine Windows Foundations**

Master Module Document (Markdown) · Revisi v4 · Sumber tunggal (single source of truth) untuk konversi ke HTML interaktif

  

Peran penyusun dokumen ini: Senior Windows Security Engineer, Blue Team Instructor, LKS Cybersecurity Competition Coach, dan Technical Writer.

  

Target: persiapan LKS Nasional Cyber Security 2026, kategori kisi-kisi resmi **Privileged Access Management (PAM)** pada cabang Infrastructure Hardening → Windows.

  

## Daftar Isi

  

- 1.0 — Posisi & Batas Modul

- 1.1 — Target Pembelajaran

- 1.2 — Mental Model PAM

- 1.3 — Aturan Utama Lomba

- 1.4 — Initial Assessment

- 1.5 — Klasifikasi Akun

- 1.6 — Audit Grup Berprivilege

- 1.7 — Built-in Administrator

- 1.8 — Guest Account

- 1.9 — Password Policy

- 1.10 — Account Lockout Policy

- 1.11 — Security Options

- 1.12 — UAC

- 1.13 — User Rights Assignment

- 1.14 — Effective Policy

- 1.15 — Safe Change Workflow

- 1.16 — GUI Procedures

- 1.17 — Change–Risk–Verification Matrix

- 1.18 — Verification Checklist

- 1.19 — Evidence Pack

- 1.20 — Troubleshooting & Rollback

- 1.21 — Kesalahan Umum

- 1.22 — Threat Mapping

- 1.23 — Memory Cheat Sheet

- 1.24 — Closed-Book Training

- 1.25 — Final Combat Card

- Lampiran — Standar Kualitas

  

---

  

## 1.0 — Posisi & Batas Modul

  

*Kategori: PENDAHULUAN*

  

**WGUI-1 — Local Account & Privilege Hardening**

  

*Revisi v3 — lima perbaikan [FIX-v3] diterapkan retroaktif: kutipan kisi-kisi verbatim & status tentatif (1.0), status Evidence Pack (1.19), fallback reset resmi (1.20), anti-lockout mencakup akses juri (1.3, 1.20), estimasi waktu eksekusi lomba (1.25).*

  

*Revisi v4 — enam perbaikan [FIX-v4] diterapkan setelah review teknis independen: koreksi klaim "Tab Account" pada local user + command-assisted exception untuk account expiration (1.4), penambahan "Allow Administrator account lockout" untuk RID 500 (1.7, 1.10), pemisahan dua setting anonymous SAM enumeration yang sebelumnya digabung (1.11), koreksi klaim penerapan struktur 17 poin (1.17), penggantian istilah "Baseline Latihan Terbaru" menjadi "Baseline Latihan Internal WGUI-1" agar tidak dikira baseline resmi Microsoft (1.9, 1.10), penambahan template Referensi Aturan Lomba (1.0).*

  

PAM Foundations melalui Local Users and Groups dan Local Security Policy. Modul teknis pertama setelah WGUI-0 Windows Hardening Foundations — persiapan LKS Nasional Cyber Security 2026, cabang Infrastructure Hardening → Windows.

  

> **🧠 [FIX-v3] Nama kategori kisi-kisi resmi (verbatim)**

>

> Dikutip persis dari Rangkuman Technical Meeting LKSN Cybersecurity 2026, daftar Kisi-kisi Windows Hardening (Hari 1 / Modul A):

>

> ```

> Privileged Access Management (PAM)

> Basic Security Configurations on Active Directory

> GPO Local/AD Policy

> Network Service Security

> AV (Windows Defender)

> Logging

> ```

>

> WGUI-1 hanya membahas **"Privileged Access Management (PAM)"** — satu dari enam kategori di atas, dibatasi pada fondasi single-machine (local account, local group, local policy). Lima kategori lain dibahas modul WGUI terpisah (lihat tabel Batas Materi di bawah).

  

> **⚠️ [FIX-v3] Konvensi status — 🔸 = tentatif**

>

> Simbol 🔸 dipakai konsisten di seluruh modul untuk menandai mekanisme yang menurut Technical Meeting *belum dikonfirmasi resmi*. Wajib dicek ulang mendekati hari-H. Status untuk modul ini:

>

> - Audit local account & group, Password Policy, Account Lockout Policy, Security Options akun, User Rights Assignment, UAC dasar — **sudah dikonfirmasi** sebagai bagian kisi-kisi PAM.

> - 🔸 Kewajiban SSH & RDP tetap menyala di mesin Windows selama sesi Hari 1 — juri sendiri menyatakan status ini *masih tentatif* ("kami sendiri agak bingung untuk Windows"). Lihat 1.3 prinsip 6 dan 1.20 untuk implikasinya terhadap anti-lockout.

  

### Cakupan Modul

  

```

AKUN

+ GRUP

+ PASSWORD

+ ACCOUNT LOCKOUT

+ SECURITY OPTIONS

+ USER RIGHTS ASSIGNMENT

+ UAC

+ VERIFIKASI

+ ROLLBACK

+ EVIDENCE

```

  

**WGUI-1 bukan sekadar modul Password Policy.** PAM di dunia industri jauh lebih luas — modul ini hanya membahas fondasi PAM pada satu mesin Windows.

  

> **ℹ️ Untuk siapa modul ini**

>

> Peserta yang masih pemula pada Windows Hardening, sudah paham dasar Linux Hardening, lebih mudah belajar lewat visual daripada teks panjang, lebih suka GUI daripada terminal, dan akan mengerjakan lomba lewat sesi RDP sambil sangat berhati-hati agar tidak kehilangan akses.

  

### Batas Materi

  

Termasuk dalam WGUI-1: audit local users & groups (termasuk status kedaluwarsa akun), least privilege, built-in Administrator, Guest, service account awareness, Password Policy (+ pengecualian per-akun), Account Lockout Policy (+ makna nilai 0), Security Options terkait akun/logon, User Rights Assignment, UAC dasar, effective policy, anti-lockout, reconnect test, verification, rollback, evidence, troubleshooting, threat mapping, closed-book training.

  

| Disebutkan, tidak dibahas mendalam | Modul tujuan |

|---|---|

| Active Directory Users and Computers | WGUI-2 |

| Domain users dan Domain Admins | WGUI-2 |

| Domain GPO dan precedence | WGUI-3 |

| Firewall | Network Service Security |

| RDP service dan port | Network Service Security |

| SMB dan WinRM | Network Service Security |

| Advanced Audit Policy | Logging |

| Windows Defender | Defender |

| Windows LAPS | Modul LAPS |

| Credential Guard | Modul lanjutan |

| BitLocker | Modul enkripsi |

  

*Tujuannya agar WGUI-1 tidak melebar dan tidak menduplikasi modul berikutnya.*

  

> **🧠 Sumber kebenaran (urutan prioritas)**

>

> 1) Objective / aturan lomba  →  2) Kisi-kisi resmi LKS Nasional 2026  →  3) Dokumentasi resmi Microsoft  →  4) Windows Security Baseline / baseline organisasi  →  5) Modul WGUI-1 ini  →  6) Baseline latihan modul.

  

> **📌 [FIX-v4] Referensi Aturan Lomba**

>

> Karena dokumen ini berstatus single source of truth, setiap klaim "resmi" (kutipan kisi-kisi, aturan penilaian, waktu, format Evidence, dll.) sebaiknya bisa ditelusuri ke sumbernya. Isi/lengkapi baris berikut setiap kali ada klaim resmi baru masuk ke modul:

>

> ```

> Dokumen                     :

> Tanggal                     :

> Versi                       :

> Halaman/bagian              :

> Timestamp Technical Meeting :

> Status                      : resmi / tentatif

> Terakhir diverifikasi       :

> ```

>

> Tanpa ini, pembaca berikutnya tidak bisa membedakan mana aturan resmi, catatan Technical Meeting, interpretasi penyusun modul, dan baseline latihan internal.

  

---

  

## 1.1 — Target Pembelajaran

  

*Kategori: PENDAHULUAN*

  

Setelah menyelesaikan WGUI-1, kamu harus mampu melakukan hal-hal berikut — tanpa membuka catatan.

  

1. Menentukan apakah mesin adalah standalone server, domain member, atau Domain Controller.

2. Membedakan local account, domain account, built-in account, service account, unknown account, dan stale account.

3. Membaca authorized user list dari objective lomba.

4. Mengaudit seluruh akun lokal lewat GUI, termasuk status kedaluwarsa akun.

5. Mengaudit grup Administrators dan grup berprivilege lainnya.

6. Menerapkan prinsip least privilege.

7. Menangani built-in Administrator dengan aman.

8. Menonaktifkan Guest jika memang diminta atau sesuai kondisi.

9. Mengatur Password Policy lewat GUI, termasuk memahami pengecualian per-akun.

10. Mengatur Account Lockout Policy lewat GUI, termasuk makna nilai 0.

11. Mengatur Security Options yang berkaitan dengan akun dan autentikasi.

12. Memahami dan menguji UAC.

13. Mengatur User Rights Assignment tanpa menyebabkan lockout.

14. Membedakan local setting dengan effective policy.

15. Memeriksa override dari Domain GPO.

16. Memverifikasi perubahan lewat login atau session baru.

17. Menyediakan rollback untuk setiap perubahan.

18. Mengambil evidence before dan after.

19. Menyelesaikan objective lomba tanpa mengandalkan terminal.

20. Mengingat seluruh alur lewat satu Combat Card, memakai satu Alur Kanonis yang sama di seluruh modul.

  

---

  

## 1.2 — Mental Model PAM

  

*Kategori: PENDAHULUAN*

  

Bayangkan sebuah gedung dengan kartu akses, ruangan, dan petugas keamanan. Analogi ini dipakai di seluruh modul.

  

| Komponen | Analogi gedung |

|---|---|

| User account | Identitas pemegang kartu |

| Password | Rahasia pembuktian identitas |

| Group membership | Daftar ruangan yang boleh dimasuki |

| Permission | Hak terhadap file atau objek tertentu |

| User Rights Assignment | Wewenang khusus tingkat sistem |

| UAC | Petugas yang meminta konfirmasi sebelum membuka ruang sensitif |

| Account Lockout | Sistem yang membekukan kartu setelah kegagalan berulang |

| Audit log | Rekaman aktivitas |

  

### Empat Pertanyaan Akun

  

- **01** — WHO?

- **02** — WHY?

- **03** — WHAT POWER?

- **04** — STILL NEEDED?

  

Tanyakan empat hal ini untuk setiap akun yang kamu temui: siapa pemiliknya, kenapa akun ini ada, kekuatan apa yang dimilikinya, dan apakah masih dibutuhkan.

  

> **⚠️ Istilah yang benar**

>

> Jangan gunakan istilah "user privilege" untuk menyebut keanggotaan grup Administrators. Istilah yang tepat: **Administrative group membership**. Perbedaan user, group, permission, privilege, administrative membership, dan elevation akan dijelaskan bertahap di bagian-bagian berikutnya.

  

---

  

## 1.3 — Aturan Utama Lomba

  

*Kategori: PENDAHULUAN*

  

Tujuh prinsip ini berlaku di setiap bagian modul. Kalau ragu di tengah lomba, kembali ke halaman ini.

  

### 1 — Objective Wins

  

Nilai atau tindakan yang tertulis pada soal selalu lebih tinggi daripada baseline hafalan.

  

```

Objective:

Set minimum password length to 12 characters.

```

  

Walaupun baseline latihan memakai 14 karakter, nilai yang benar tetap **12**.

  

> **🧠 Kalimat hafalan**

>

> Angka soal mengalahkan angka hafalan.

  

### 2 — Jangan Melakukan Hardening Secara Buta

  

Jangan otomatis: menghapus akun tak dikenal · membuat administrator baru · menonaktifkan built-in Administrator · mengganti nama Administrator · mengeluarkan semua anggota Administrators · menambahkan grup besar ke Deny logon · mengubah seluruh User Rights Assignment · menerapkan seluruh baseline tanpa membaca objective.

  

Setiap perubahan wajib punya: **ALASAN · RISIKO · VERIFIKASI · ROLLBACK · EVIDENCE.**

  

### 3 — Alur Kanonis (Authorized Account First)

  

> **🧠 Satu-satunya alur resmi WGUI-1**

>

> Alur 10 langkah ini dipakai kata-per-kata sama di 1.15 (Safe Change Workflow) dan 1.25 (Final Combat Card). Tidak ada versi lain.

  

```

1. BACA OBJECTIVE

2. KENALI JENIS SERVER

3. AUDIT AUTHORIZED USERS

4. CEK ADMINISTRATORS

5. CEK SERVICE DEPENDENCY

6. VALIDASI RECOVERY SAH

7. UBAH SATU HAL

8. VERIFY

9. LOGIN BARU

10. EVIDENCE

```

  

*Versi panah untuk diagram alur visual:*

  

```

OBJECTIVE → AUTHORIZED → BASELINE → RECOVERY → CHECK DEPENDENCY → CHANGE ONE → VERIFY SETTING → TEST FUNCTION / LOGIN BARU → EVIDENCE → CONTINUE

```

  

Saat mengklasifikasi akun (bagian dari langkah 3), gunakan sub-alur berikut:

  

```

IDENTIFY → CHECK DEPENDENCY → COMPARE AUTHORIZED LIST → DECIDE

```

  

### 4 — Jangan Otomatis Membuat Recovery Administrator

  

Prinsip yang benar: *identifikasi, validasi, atau gunakan akun administratif yang telah diotorisasi sebagai recovery. Buat akun baru hanya jika rules atau objective mengizinkan.*

  

Sebelum menyentuh built-in Administrator atau grup Administrators:

  

- [ ] Akun recovery sah tersedia

- [ ] Status akun aktif

- [ ] Password diketahui

- [ ] Privilege sesuai kebutuhan

- [ ] Tidak terkena Deny logon

- [ ] Memiliki hak RDP jika diperlukan

- [ ] Sudah diuji lewat login baru

- [ ] Session utama belum ditutup

  

*Akun yang hanya terlihat aktif tetapi belum diuji login bukan recovery yang valid.*

  

### 5 — Deny Mengalahkan Allow

  

Pada User Rights Assignment: akun bisa memperoleh Allow lewat user atau group, tetapi jika akun juga terkena Deny, login tetap ditolak.

  

> **🧠 Kalimat hafalan**

>

> Deny menang. Apply belum tentu aman. Uji login baru.

  

### 6 — Session Aktif Bukan Reconnect Test

  

Session RDP yang sudah aktif dapat tetap hidup walaupun koneksi baru sudah gagal.

  

1. Pertahankan Session A.

2. Terapkan satu perubahan.

3. Buka koneksi atau Session B.

4. Lakukan login baru.

5. Pastikan desktop benar-benar dapat digunakan.

6. Jangan tutup Session A sebelum tes berhasil.

7. Baru lanjut ke perubahan lain.

  

> **🧠 Kalimat hafalan**

>

> Session masih hidup belum berarti reconnect berhasil.

  

> **⚠️ [FIX-v3] Anti-lockout mencakup akses juri, bukan hanya akses kamu**

>

> Reconnect test/login baru bukan cuma untuk memastikan kamu sendiri tidak terkunci. Tujuannya ganda: (1) kamu masih bisa login, DAN (2) jalur yang dipakai juri/teknisi untuk menilai mesinmu tetap dapat diakses setelah perubahan. Mengunci akses juri sama fatalnya dengan mengunci akses sendiri — mesin yang tidak bisa dinilai berisiko dianggap gagal pada objektif itu.

>

> 🔸 Untuk mesin Windows, service mana yang wajib tetap menyala untuk akses juri (SSH, RDP, atau keduanya) *masih berstatus tentatif* menurut Technical Meeting — beda dengan mesin Linux yang sudah jelas: SSH wajib menyala sebagai satu-satunya jalur juri. Sebelum mengubah User Rights Assignment, Security Options, atau apa pun yang bisa mematikan/membatasi RDP atau SSH di Windows, anggap keduanya berpotensi menjadi jalur juri sampai ada kepastian resmi mendekati hari-H.

  

### 7 — Local Setting Belum Tentu Effective Policy

  

Pada domain member, setting dari `secpol.msc` dapat ditimpa Domain GPO.

  

```

SETTING YANG DIKLIK ≠ SELALU SETTING YANG BERLAKU

```

  

Cek dengan `rsop.msc`, atau pada lingkungan domain: `gpmc.msc → Group Policy Results`. Detail lengkap ada di 1.14 Effective Policy.

  

---

  

## 1.4 — Initial Assessment

  

*Kategori: PERSIAPAN*

  

Sebelum menyentuh satu setting pun, lakukan account map lengkap. Ini pondasi seluruh alur kanonis.

  

### Perbedaan Tipe Server

  

| Jenis Mesin | Pengelolaan Akun | Sumber Policy |

|---|---|---|

| Standalone / workgroup | Local Users and Groups | Local Policy |

| Domain member | Local Users and Groups + akun domain | Local Policy dapat ditimpa Domain GPO |

| Domain Controller | Active Directory Users and Computers | Domain Policy |

  

> **🛑 Peringatan Domain Controller**

>

> Pada Domain Controller tidak ada model local user seperti standalone server. `lusrmgr.msc` tidak dipakai untuk mengelola akun seperti pada member server — akun dikelola lewat `dsa.msc`, dan Password Policy domain dikelola lewat domain policy atau Fine-Grained Password Policy. Jangan menerapkan langkah local account secara buta. Jika `lusrmgr.msc` tidak tersedia atau tidak berperilaku seperti biasa, cek dulu apakah server ini Domain Controller.

  

> **⚠️ Cek versi OS sebelum mulai**

>

> Sebelum mencari nama-nama setting spesifik di `secpol.msc`, cek dulu versi Windows Server yang dihadapi (lewat `winver`, Server Manager, atau `systeminfo`). Beberapa nama security option berbeda tergantung versi OS — lihat 1.11 Security Options. Ini bagian resmi dari initial assessment, bukan langkah opsional.

  

### Informasi yang Dicatat

  

```

Hostname:

Domain atau Workgroup:

Jenis server:

Versi OS (untuk penamaan security option):

Current user:

Current privilege:

Daftar local users:

Anggota Administrators:

Anggota Remote Desktop Users:

Anggota Backup Operators:

Service accounts:

Authorized account list:

Password Policy awal:

Account Lockout Policy awal:

Sumber effective policy:

Recovery path:

```

  

### GUI Audit Akun

  

**GUI Path:** `Computer Management → System Tools → Local Users and Groups → Users`

  

Klik kanan akun → Properties. Local user hanya punya tiga tab: **General**, **Member Of**, dan **Profile** — tidak ada tab **Account** seperti pada Active Directory Users and Computers (ADUC). Ini beda penting yang sering bikin peserta salah cari.

  

#### Tab General

  

- Full name · Description

- User must change password at next logon

- User cannot change password

- Password never expires

- Account is disabled

- Account is locked out

  

#### Tab Member Of

  

Periksa seluruh grup yang memberi privilege pada akun ini.

  

#### Tab Profile

  

Path profil dan logon script jika dikonfigurasi. Jarang relevan untuk objective PAM lomba, tapi tetap sekilas dicek untuk dependency.

  

> **⚠️ [FIX-v4] Account Expiration — pengecualian command-assisted**

>

> **Local Users and Groups TIDAK punya field "Account expires" di GUI.** Ini berbeda dari domain account, yang expiration-nya dicek/diatur lewat ADUC → Properties → tab **Account**.

>

> Untuk local account, audit expiration lewat command — ini satu-satunya pengecualian resmi terhadap prinsip GUI-first modul ini, dicatat secara jujur karena modul mengklaim GUI-first, bukan GUI-only:

>

> ```

> net user <username>

> ```

> atau

> ```

> Get-LocalUser <username> | Select-Object AccountExpires

> ```

>

> Untuk mengatur tanggal kedaluwarsa: `net user <username> /expires:MM/DD/YYYY` atau `Set-LocalUser -Name <username> -AccountExpires <date>`.

>

> Field ini tetap wajib masuk initial assessment (lihat 1.5 Klasifikasi Akun) — akun dengan tanggal kedaluwarsa yang sudah lewat tapi masih *enabled* adalah temuan stale account yang wajib dicatat. Hanya jalur auditnya lewat command, bukan GUI.

  

### Command Verifikasi Opsional

  

```

net user

net user <username>

net localgroup administrators

net localgroup "Remote Desktop Users"

net accounts

whoami

whoami /groups

whoami /priv

Get-LocalUser

Get-LocalGroup

Get-LocalGroupMember -Group "Administrators"

```

  

*Command perubahan sengaja tidak ditonjolkan pada tahap awal — GUI tetap jalur utama (lihat 1.16).*

  

---

  

## 1.5 — Klasifikasi Akun

  

*Kategori: PERSIAPAN*

  

Setiap akun yang ditemukan harus masuk salah satu kategori berikut sebelum diputuskan tindakannya.

  

### Authorized Human Account

  

Akun manusia yang diizinkan dan dibutuhkan. Tindakan: pertahankan, sesuaikan privilege, pastikan status dan password aman.

  

### Authorized Administrative Account

  

Akun administratif yang sah. Tindakan: jumlah minimum, login harus teruji, jangan dipakai untuk aktivitas biasa jika tidak diperlukan.

  

### Built-in Account

  

Contoh: Administrator, Guest, DefaultAccount, WDAGUtilityAccount.

  

> **ℹ️ Ingat**

>

> Built-in account bukan otomatis malicious.

  

### Service Account

  

Digunakan oleh Windows service, IIS, database, backup, Scheduled Task, atau aplikasi tertentu. Sebelum disable atau mengganti password, periksa dulu:

  

**GUI Path:** `services.msc → Service → Properties → Log On`

  

*Periksa juga Scheduled Task jika relevan.*

  

### Unknown Account

  

> **🛑 Jangan buru-buru**

>

> Belum diketahui fungsinya — tapi unknown tidak sama dengan malicious. Ikuti sub-alur IDENTIFY → CHECK DEPENDENCY → COMPARE AUTHORIZED LIST → DECIDE (lihat 1.3) sebelum bertindak.

  

### Stale Account

  

Akun sah tetapi sudah tidak digunakan.

  

> **✅ Preferensi lomba**

>

> Disable biasanya lebih aman daripada delete karena mudah di-rollback.

  

Delete hanya dilakukan jika: objective meminta · akun sudah dipastikan tidak dibutuhkan · dependency sudah diperiksa · evidence sudah diambil.

  

> **⚠️ Sinyal tambahan stale account**

>

> Periksa field **Account expires** (lihat 1.4). Akun dengan tanggal kedaluwarsa yang sudah lewat tapi masih *enabled* adalah temuan yang wajib dicatat.

  

---

  

## 1.6 — Audit Grup Berprivilege

  

*Kategori: PERSIAPAN*

  

Grup memberi akses ke ruangan — audit grup sama pentingnya dengan audit akun individual.

  

### Prioritas Utama

  

**GUI Path:** `Computer Management → Local Users and Groups → Groups → Administrators`

  

#### Pertanyaan Audit

  

1. Apakah semua anggota terdaftar pada authorized list?

2. Apakah user biasa menjadi Administrator?

3. Apakah ada akun asing?

4. Apakah ada grup domain yang tidak semestinya?

5. Apakah akun recovery yang sah masih tersedia?

6. Apakah akun yang akan dikeluarkan sedang digunakan?

7. Apakah perubahan memerlukan sign-out/login untuk memperbarui token?

  

### Grup yang Harus Dikenali

  

| Grup | Dampak |

|---|---|

| Administrators | Kontrol penuh mesin |

| Backup Operators | Dapat melewati izin tertentu untuk backup/restore |

| Remote Desktop Users | Jalur akses RDP jika policy mengizinkan |

| Remote Management Users | Akses remote management |

| Hyper-V Administrators | Kontrol virtualisasi |

| Event Log Readers | Membaca log |

| Performance Log Users | Mengelola performance log |

  

> **🛑 Kesalahan yang harus dicegah**

>

> Akun yang membutuhkan RDP tidak otomatis harus masuk Administrators.

  

#### Checklist RDP

  

- [ ] Account enabled

- [ ] Password valid

- [ ] Remote Desktop Users atau Administrators

- [ ] Allow RDP logon

- [ ] Tidak terkena Deny

- [ ] RDP service berjalan

- [ ] Firewall mengizinkan

- [ ] NLA sesuai kondisi

- [ ] Effective GPO sudah dicek

  

---

  

## 1.7 — Built-in Administrator

  

*Kategori: HARDENING AKUN*

  

Akun paling sensitif di seluruh mesin — dan yang paling sering diperlakukan secara sembrono.

  

Built-in Administrator memiliki SID dengan **RID 500**. Rename tidak mengubah identitas SID dan hanya memberi hambatan kecil bagi penyerang.

  

### Perlindungan Utama

  

- Password kuat dan unik

- Least privilege pada akun lain

- Pembatasan logon

- Audit

- LAPS jika tersedia

- **[FIX-v4]** Cek status "Allow Administrator account lockout" jika tersedia di versi OS ini — lihat 1.10

- Disable hanya bila aman dan diperbolehkan

  

> **✅ Boleh dipertimbangkan untuk disable jika**

>

> Objective meminta · ada admin sah lain · admin lain berhasil login baru · dependency sudah diperiksa · rollback tersedia.

  

> **🛑 Jangan disable jika**

>

> Ini satu-satunya admin sah · objective mempertahankan akun · recovery belum diuji · jenis server belum diketahui · akun diperlukan untuk recovery.

  

### GUI

  

**GUI Path:** `Computer Management → Local Users and Groups → Users → Administrator → Properties`

  

**Rename:** Klik kanan Administrator → Rename.

  

**Alternatif via policy:**

  

**GUI Path:** `secpol.msc → Local Policies → Security Options → Accounts: Rename administrator account`

  

> **🧠 Kalimat hafalan**

>

> Nama bukan pertahanan utama. Jangan matikan RID 500 sebelum admin sah lain teruji.

  

---

  

## 1.8 — Guest Account

  

*Kategori: HARDENING AKUN*

  

Salah satu setting paling sederhana — tapi tetap butuh konfirmasi ganda.

  

### GUI Utama

  

**GUI Path:** `Computer Management → Local Users and Groups → Users → Guest → Properties → Account is disabled`

  

### Konfirmasi

  

**GUI Path:** `secpol.msc → Local Policies → Security Options → Accounts: Guest account status → Disabled`

  

> **⚠️ Cek dulu sebelum menyimpulkan**

>

> Guest biasanya disabled, tetapi beberapa konfigurasi legacy tertentu dapat menggunakan guest-only sharing. Cek kebutuhan legacy sebelum menonaktifkan tanpa pikir panjang.

  

> **🧠 Kalimat hafalan**

>

> Guest biasanya mati; cek kebutuhan legacy sebelum menyimpulkan.

  

---

  

## 1.9 — Password Policy

  

*Kategori: HARDENING AKUN*

  

**GUI Path:** `Win + R → secpol.msc → Account Policies → Password Policy`

  

### Baseline Latihan Internal WGUI-1

  

| Setting | Nilai Latihan |

|---|---|

| Minimum password length | 14 characters |

| Enforce password history | 24 passwords |

| Maximum password age | 60 days |

| Minimum password age | 1 day |

| Password must meet complexity requirements | Enabled |

| Store passwords using reversible encryption | Disabled |

  

- **Urutan hafalan** — 14 / 24 / 60 / 1 / ON / OFF

  

> **🧠 Kalimat hafalan**

>

> 14 panjang, 24 ingatan, 60 umur, 1 jeda, kompleks ON, reversible OFF.

  

*Nilai ini adalah baseline latihan, bukan hukum mutlak. Objective tetap menang (lihat 1.3).*

  

### Waktu Efek

  

- Minimum length dan complexity diperiksa ketika password baru dibuat atau diubah.

- Setting tidak otomatis membatalkan semua password lama.

- Maximum age berkaitan dengan usia password.

- Minimum age mencegah pergantian berulang untuk melewati history.

- Reversible encryption harus Disabled kecuali ada pengecualian resmi.

  

> **🛑 Trap penting — pengecualian per-akun**

>

> Password Policy di `secpol.msc` berlaku sebagai default untuk semua akun lokal, **tetapi bisa ditimpa per-akun** lewat checkbox pada Properties user.

>

> - Jika **"Password never expires"** dicentang pada akun tertentu, **Maximum password age tidak berlaku untuk akun itu** — walaupun policy sudah benar diset di secpol.msc.

> - Ini penyebab umum "kenapa password policy terlihat tidak jalan" — cek dulu checkbox per-akun sebelum menyimpulkan policy salah.

> - Saat audit, selalu cocokkan: apakah ada akun authorized yang seharusnya ikut aturan expiry tapi punya "Password never expires" aktif tanpa alasan sah?

  

Setiap setting sebaiknya dijelaskan dengan: pengertian, risiko terlalu lemah, risiko terlalu ketat, nilai latihan, GUI path, waktu berlaku, cara verifikasi, rollback, dan kemungkinan objective lomba — lihat template lengkap di 1.17.

  

---

  

## 1.10 — Account Lockout Policy

  

*Kategori: HARDENING AKUN*

  

**GUI Path:** `secpol.msc → Account Policies → Account Lockout Policy`

  

### Baseline Latihan Internal WGUI-1

  

```

Account lockout threshold: 10

Account lockout duration: 15 minutes

Reset account lockout counter after: 15 minutes

```

  

- **Hafalan** — 10 / 15 / 15

  

> **🧠 Kalimat hafalan**

>

> Sepuluh salah, lima belas terkunci, lima belas hitungan pulih.

  

*Threshold terlalu kecil dapat dipakai attacker untuk lockout denial-of-service.*

  

> **🛑 Trap penting — makna nilai 0**

>

> **Account lockout threshold = 0 berarti lockout dinonaktifkan** — akun tidak akan pernah terkunci berapa pun jumlah percobaan gagal. Ini bukan berarti "belum diatur" atau kesalahan input. Beberapa objective lomba secara sengaja meminta lockout dimatikan (misalnya untuk service account tertentu) — caranya adalah set threshold ke 0, bukan mengosongkan field lain.

  

> **⚠️ Jangan uji dengan akun utama**

>

> Jangan menguji lockout memakai akun utama atau satu-satunya administrator. Gunakan akun uji yang: diizinkan · bukan akun recovery utama · dapat di-unlock · tidak digunakan service.

  

Hubungan tiga setting ini harus dipahami sebagai satu kesatuan, bukan angka terpisah. Jika objective memberi nilai berbeda — misalnya `5 percobaan / 30 menit duration / 30 menit reset` — gunakan 5/30/30, bukan baseline latihan.

  

> **🛑 [FIX-v4] Version-aware — Allow Administrator account lockout (RID 500)**

>

> Sebelum update kumulatif Windows Oktober 2022 (KB5020282) dan setelahnya, built-in Administrator (RID 500) **secara default TIDAK terkena Account Lockout Policy sama sekali** — berapa pun percobaan gagal, akun ini tidak akan pernah locked out lewat threshold biasa. Ini celah brute-force yang nyata untuk akun paling sensitif di mesin.

>

> Pada OS yang sudah menerima update tersebut, tersedia setting baru sejajar dengan tiga setting di atas:

>

> **GUI Path:** `secpol.msc → Account Policies → Account Lockout Policy → Allow Administrator account lockout`

>

> - **Disabled** (perilaku lama) — RID 500 kebal lockout, rentan brute-force tanpa batas.

> - **Enabled** — RID 500 ikut threshold/duration/reset yang sama dengan akun lain.

>

> Jangan asumsikan baseline 10/15/15 otomatis berlaku untuk RID 500 di semua versi OS — cek dulu versi Windows (lihat 1.4) dan apakah setting ini tersedia, terutama kalau objective menyinggung proteksi built-in Administrator dari brute-force. Lihat juga 1.7 Built-in Administrator.

  

---

  

## 1.11 — Security Options

  

*Kategori: HARDENING AKUN*

  

**GUI Path:** `secpol.msc → Local Policies → Security Options`

  

### Prioritas A

  

#### Accounts: Guest account status

  

Disabled

  

#### Accounts: Limit local account use of blank passwords to console logon only

  

Enabled

  

*Setting ini membatasi remote logon dengan password kosong, tetapi bukan alasan membiarkan akun tanpa password.*

  

#### Interactive logon: sembunyikan nama user terakhir

  

> **🛑 Version-aware — cek versi OS dulu (lihat 1.4)**

>

> Nama setting ini **berbeda tergantung versi OS**:

  

| Versi OS | Nama setting resmi |

|---|---|

| Windows Server 2012 / 2016 dan lebih lama | "Interactive logon: Do not display last user name" |

| Windows Server 2019 ke atas / Windows 10 versi 1703+ | "Interactive logon: Don't display last signed-in" |

  

Nilai: Enabled

  

*Format login yang relevan: `.\username` atau `DOMAIN\username`.*

  

#### Network access: Do not allow anonymous enumeration of SAM accounts

  

Enabled

  

*Mencegah pihak tak terautentikasi mengenumerasi daftar akun lokal — relevan langsung dengan tema PAM karena membatasi reconnaissance akun.*

  

#### Network access: Do not allow anonymous enumeration of SAM accounts and shares

  

Enabled

  

> **⚠️ [FIX-v4] Ini DUA setting berbeda, bukan satu**

>

> Windows punya dua entri terpisah di Security Options — jangan mencari satu setting gabungan "(and shares)" karena itu tidak ada. Setting di atas hanya membatasi enumerasi akun; setting ini menambah pembatasan enumerasi share. Keduanya biasanya diset Enabled bersamaan untuk hardening PAM, tapi tetap dua baris konfigurasi yang harus dicek dan diaktifkan masing-masing.

  

#### Network security: LAN Manager authentication level

  

Nilai hardening umum: Send NTLMv2 response only. Refuse LM & NTLM

  

> **⚠️ Peringatan kompatibilitas**

>

> NAS lama, perangkat lama, aplikasi legacy, dan OS lama dapat gagal autentikasi. Jangan menerapkan tanpa uji kompatibilitas jika objective tidak jelas.

  

### Prioritas B atau Situasional

  

- **Accounts: Rename administrator account** — hanya jika objective atau baseline meminta.

- **Accounts: Administrator account status** — jangan otomatis Disabled.

- **Interactive logon: Machine inactivity limit** — contoh 900 seconds, bukan angka wajib universal.

- **Interactive logon: Message title/text** — hanya jika objective meminta banner. Jangan buat banner panjang yang mengganggu lomba.

  

---

  

## 1.12 — UAC

  

*Kategori: HARDENING AKUN*

  

**User Account Control (UAC)**

  

**GUI Path:** `secpol.msc → Local Policies → Security Options → cari awalan "User Account Control:"`

  

### Setting Inti

  

| Setting | Nilai |

|---|---|

| Run all administrators in Admin Approval Mode | Enabled |

| Behavior of elevation prompt for administrators | Sesuai objective — tetap minta consent/credentials |

| Detect application installations and prompt for elevation | Enabled |

| Switch to the secure desktop when prompting for elevation | Enabled |

  

### Konsep yang Wajib Dipahami

  

- Anggota Administrators belum tentu setiap prosesnya elevated.

- Membership dan active access token berbeda.

- UAC memisahkan aktivitas biasa dan aktivitas elevated.

- UAC bukan pengganti least privilege.

  

> **🛑 Jangan**

>

> Jangan mematikan UAC hanya agar prompt tidak muncul.

  

### Verifikasi

  

1. Buka aplikasi administratif tanpa Run as administrator.

2. Lakukan tindakan aman yang membutuhkan elevasi.

3. Pastikan prompt muncul.

  

*Jangan menggunakan perubahan berbahaya hanya untuk mengetes.*

  

---

  

## 1.13 — User Rights Assignment

  

*Kategori: HARDENING AKUN*

  

**GUI Path:** `secpol.msc → Local Policies → User Rights Assignment`

  

*User Rights Assignment bukan permission file — ini wewenang tingkat sistem, bukan hak atas objek tertentu.*

  

### Setting Prioritas

  

| Setting | Catatan risiko |

|---|---|

| Allow log on locally | Siapa yang boleh login lewat console |

| Deny log on locally | Deny mengalahkan Allow |

| Allow log on through Remote Desktop Services | Siapa yang boleh login RDP |

| Deny log on through Remote Desktop Services | Sangat berisiko menyebabkan self-lockout |

| Log on as a service | Diperlukan service account |

| Deny log on as a service | Dapat membuat service gagal start |

| Back up files and directories | Hak kuat untuk proses backup |

| Restore files and directories | Hak kuat untuk proses restore |

| Take ownership of files or other objects | Hak sangat tinggi, harus dibatasi |

  

### Aturan Keselamatan

  

> **🛑 BAHAYA**

>

> Jangan mengosongkan atau mengganti seluruh daftar tanpa mencatat isi awal.

  

Sebelum menghapus akun dari **Log on as a service**:

  

**GUI Path:** `services.msc → cari service → Properties → Log On`

  

Setelah perubahan User Rights:

  

- Apply bukan bukti aman.

- Efek biasanya terlihat pada session atau login baru.

- Lakukan reconnect test (lihat 1.3, prinsip 6).

- Pastikan recovery account tidak terkena Deny.

  

*Detail hak login RDP dan service account adalah fokus utama bagian ini karena paling sering menyebabkan self-lockout.*

  

---

  

## 1.14 — Effective Policy

  

*Kategori: VERIFIKASI & GOVERNANCE*

  

Setting yang kamu klik belum tentu setting yang benar-benar berlaku.

  

```

SETTING YANG DIKLIK ≠ SELALU SETTING YANG BERLAKU

```

  

Pada domain member, gunakan `rsop.msc`, atau pada lingkungan domain: `gpmc.msc → Group Policy Results`.

  

### Empat Lapisan yang Harus Dibedakan

  

- **Configured setting** — apa yang kamu atur di GUI.

- **Local policy** — nilai di secpol.msc pada mesin ini.

- **Domain policy** — nilai yang datang dari GPO domain.

- **Effective policy** — nilai yang benar-benar berlaku setelah semua sumber digabung.

  

### Waktu Efek Perubahan

  

| Perubahan | Waktu Efek Umum |

|---|---|

| Password length/complexity | Saat password berikutnya dibuat atau diubah |

| Account lockout threshold | Pada kegagalan login berikutnya |

| Account lockout duration | Setelah akun mencapai kondisi locked |

| Maximum password age | Berdasarkan usia password (kecuali "never expires" per-akun) |

| User Rights Assignment | Umumnya pada login atau session baru |

| Group membership | Dapat memerlukan sign-out/login untuk token baru |

| Local/domain GPO | Setelah policy refresh; sebagian perlu restart/logoff |

| Service account rights | Dapat terlihat saat service start/restart |

| Firewall/network/RDP | Dapat memutus session atau menggagalkan koneksi baru |

  

> **ℹ️ Dua jenis verifikasi berbeda**

>

> **VERIFY SETTING** — melihat value di secpol.msc. **VERIFY FUNCTION** — berhasil login lewat RDP atau menjalankan fungsi sungguhan. Keduanya wajib, jangan berhenti di salah satu.

  

---

  

## 1.15 — Safe Change Workflow

  

*Kategori: VERIFIKASI & GOVERNANCE*

  

Ini bukan alur baru — ini kutipan langsung dari Alur Kanonis di 1.3, dipakai identik kata-per-kata di sini dan di 1.25 Final Combat Card.

  

> **🧠 Mengutip 1.3 — Alur Kanonis**

>

> Tidak ada versi baru yang didefinisikan di sini.

  

```

1. BACA OBJECTIVE

2. KENALI JENIS SERVER

3. AUDIT AUTHORIZED USERS

4. CEK ADMINISTRATORS

5. CEK SERVICE DEPENDENCY

6. VALIDASI RECOVERY SAH

7. UBAH SATU HAL

8. VERIFY

9. LOGIN BARU

10. EVIDENCE

```

  

Setiap langkah memetakan ke bagian modul ini:

  

| Langkah | Rujukan bagian |

|---|---|

| 1. Baca objective | Objective Wins — 1.3 |

| 2. Kenali jenis server | 1.4 Initial Assessment |

| 3. Audit authorized users | 1.4 & 1.5 Klasifikasi Akun |

| 4. Cek Administrators | 1.6 Audit Grup Berprivilege |

| 5. Cek service dependency | 1.5 (Service Account) & 1.13 |

| 6. Validasi recovery sah | 1.3 prinsip 4, 1.7 Built-in Administrator |

| 7. Ubah satu hal | 1.9–1.13 (setting spesifik) |

| 8. Verify | 1.14 Effective Policy |

| 9. Login baru | 1.3 prinsip 6, Session Aktif Bukan Reconnect Test |

| 10. Evidence | 1.19 Evidence Pack |

  

### Sub-alur klasifikasi akun (bagian dari langkah 3)

  

```

IDENTIFY → CHECK DEPENDENCY → COMPARE AUTHORIZED LIST → DECIDE

```

  

---

  

## 1.16 — GUI Procedures

  

*Kategori: VERIFIKASI & GOVERNANCE*

  

**GUI Utama yang Harus Dihafal**

  

GUI adalah jalur utama modul ini. Command hanya alat bantu.

  

| Perintah | Fungsi |

|---|---|

| `compmgmt.msc` | Computer Management — akun dan grup lokal |

| `lusrmgr.msc` | Local Users and Groups |

| `secpol.msc` | Password, lockout, Security Options, User Rights |

| `rsop.msc` | Resultant Set of Policy / effective policy |

| `services.msc` | Dependency service account |

| Server Manager | Jenis server, domain/workgroup, installed roles |

| `dsa.msc` | Active Directory Users and Computers (domain) |

| `gpmc.msc` | Group Policy Management (domain) |

  

> **ℹ️ Peran command**

>

> Command hanya ditempatkan sebagai: verifikasi cepat · alternatif jika GUI tidak tersedia · alat diagnosis · referensi tambahan. Jangan menjadikan PowerShell sebagai jalur utama pembelajaran.

  

```

net user

net user <username>

net localgroup administrators

net localgroup "Remote Desktop Users"

net accounts

whoami

whoami /groups

whoami /priv

Get-LocalUser

Get-LocalGroupMember -Group "Administrators"

Get-LocalUser | Where-Object {$_.LockedOut -eq $true}

```

  

*Command perubahan (yang mengubah setting) harus diberi label risiko dan tidak ditempatkan sebelum kamu paham jalur GUI-nya. Jangan membuat script otomatis yang mengubah semua setting sekaligus — target latihan adalah hardening manual dan closed-book.*

  

---

  

## 1.17 — Change–Risk–Verification Matrix

  

*Kategori: VERIFIKASI & GOVERNANCE*

  

> **⚠️ [FIX-v4] Ini kerangka analisis, bukan format wajib di setiap submateri**

>

> Struktur 17 poin berikut dipakai sebagai kerangka analisis saat latihan, supaya kamu tidak perlu belajar pola baru di setiap submateri. Tidak semua setting di 1.9–1.13 ditulis ulang penuh dalam 17 poin di modul ini — itu akan membuat modul terlalu panjang dan justru sulit dihafal. Contoh terisi penuh di bawah hanya untuk Guest Account; gunakan template ini untuk melengkapi baris kosong pada setting lain yang kamu temui saat latihan atau lomba.

  

1. Apa ini?

2. Kenapa penting?

3. Ancaman yang dicegah

4. Kemungkinan objective lomba

5. Kondisi awal yang harus dicek

6. Lokasi GUI

7. Langkah GUI bernomor

8. Command verifikasi opsional

9. Risiko perubahan

10. Kapan efek berlaku

11. Cara verifikasi setting

12. Cara verifikasi fungsi

13. Reconnect atau login test

14. Evidence yang diambil

15. Cara rollback

16. Kesalahan umum

17. Kalimat hafalan

  

### Contoh terisi — Disable Guest Account

  

| Field | Detail |

|---|---|

| Apa ini? | Menonaktifkan akun bawaan Guest |

| Ancaman dicegah | Unnecessary/anonymous access |

| GUI path | Computer Management → Local Users and Groups → Users → Guest → Properties |

| Verifikasi setting | Guest Properties menunjukkan Account is disabled; Security Options menunjukkan Disabled |

| Rollback | Hapus centang Account is disabled jika ada dependency sah |

| Evidence | W1_05_Guest_After.png |

  

*Gunakan baris kosong berikut sebagai template untuk setting lain yang kamu temui di lomba.*

  

---

  

## 1.18 — Verification Checklist

  

*Kategori: VERIFIKASI & GOVERNANCE*

  

Jalankan checklist ini setelah setiap praktik — berbeda fungsi dari Safe Change Workflow (1.15) yang jalan sebelum dan selama perubahan.

  

- [ ] Setting sudah dicek nilainya di secpol.msc / lusrmgr.msc (VERIFY SETTING)

- [ ] Fungsi sungguhan sudah diuji, misalnya login RDP (VERIFY FUNCTION)

- [ ] Reconnect / login baru berhasil, bukan hanya session lama yang masih hidup

- [ ] Session A dipertahankan sampai Session B teruji

- [ ] Effective policy sudah dicek (rsop.msc / gpmc.msc bila domain member)

- [ ] Before state dan after state sudah difoto

- [ ] Rollback sudah dicatat

  

---

  

## 1.19 — Evidence Pack

  

*Kategori: VERIFIKASI & GOVERNANCE*

  

Format catatan objective yang dipakai untuk setiap perubahan.

  

> **ℹ️ [FIX-v3] Status Evidence Pack saat lomba sungguhan**

>

> Untuk **Hari 1 / Modul A (Hardening)**: Evidence Pack di bagian ini adalah **alat latihan/self-verification**, *bukan* deliverable wajib. Berdasarkan aturan penilaian resmi, tidak perlu membuat write-up atau dokumentasi saat lomba — juri/teknisi menilai langsung dari kondisi mesin peserta setelah waktu habis, per-objektif secara biner (0/1). Jangan habiskan waktu lomba menyusun berkas evidence rapi seperti contoh di bawah — screenshot cepat before/after untuk kebutuhanmu sendiri sudah cukup.

>

> Ini berbeda dengan **Modul B (Hari 2) dan Modul C (Hari 3)**, yang *mewajibkan* write-up (kronologis, flag, screenshot/snippet, POC, dikumpulkan lewat Google Form di hari yang sama) — bukan cakupan modul ini.

  

```

Objective:

Initial condition:

Risk:

Change:

GUI path:

Verification:

Reconnect/function test:

Evidence:

Rollback:

Result:

```

  

### Contoh Terisi

  

```

Objective:

Disable the Guest account.

  

Initial condition:

Guest account enabled.

  

Risk:

Legacy guest-only sharing may fail.

  

Change:

Guest disabled through Local Users and Groups.

Security Options confirmed.

  

GUI path:

Computer Management → Local Users and Groups → Users → Guest → Properties

  

Verification:

Guest Properties shows Account is disabled.

Accounts: Guest account status shows Disabled.

  

Evidence:

W1_05_Guest_After.png

  

Rollback:

Clear Account is disabled if authorized dependency requires it.

  

Result:

PASS

```

  

### Evidence Minimum

  

- [ ] Before state

- [ ] Perubahan

- [ ] After state

- [ ] Effective policy

- [ ] Login/reconnect atau function test

- [ ] Catatan rollback

  

### Penamaan File

  

```

W1_01_Server_Type.png

W1_02_User_Baseline.png

W1_03_Administrators_Before.png

W1_04_Administrators_After.png

W1_05_Guest_After.png

W1_06_Password_Policy.png

W1_07_Lockout_Policy.png

W1_08_User_Rights.png

W1_09_Effective_Policy.png

W1_10_Login_Test.png

```

  

*Boleh ditambah hostname dan waktu jika tidak membebani.*

  

---

  

## 1.20 — Troubleshooting & Rollback

  

*Kategori: PEMULIHAN & EVALUASI*

  

**Troubleshooting dan Rollback**

  

Decision tree cepat untuk enam masalah paling sering terjadi saat lomba.

  

### Akun Tidak Bisa RDP

  

1. Account enabled?

2. Account locked?

3. Password valid?

4. Member Administrators atau Remote Desktop Users?

5. Allow log on through Remote Desktop Services?

6. Deny log on through Remote Desktop Services?

7. Effective GPO?

8. RDP service?

9. Firewall?

10. NLA atau authentication compatibility?

  

### Local Security Policy Tidak Bisa Diedit

  

Kemungkinan: tidak dijalankan sebagai administrator · setting dikontrol GPO · mesin adalah Domain Controller · policy source bukan local.

  

### Service Gagal Setelah User Rights

  

Kemungkinan: kehilangan Log on as a service · akun disabled · password akun service berubah · terkena Deny log on as a service.

  

> **🛑 Semua Administrator Lokal Terhapus**

>

> Jika masih ada elevated session:

>

> 1. Jangan logout.

> 2. Buka Computer Management.

> 3. Tambahkan kembali authorized admin.

> 4. Lakukan login baru.

> 5. Simpan evidence.

  

### Account Lockout Terlalu Ketat

  

- Masuk memakai admin recovery sah

- Unlock akun

- Pulihkan nilai threshold/duration/reset

- Cari cached credential, service, Scheduled Task, atau session lama penyebab kegagalan berulang

  

### Password Policy Terlihat Tidak Bekerja

  

- Password lama atau password baru?

- Local account atau domain account?

- **Checkbox "Password never expires" pada akun individual** (lihat 1.9)

- Domain GPO override?

- Policy refresh sudah terjadi?

- Aplikasi menggunakan credential berbeda?

  

> **🛑 [FIX-v3] Langkah Terakhir yang Sah — Reset Mesin Resmi**

>

> Jika mesin sudah tidak bisa diakses sama sekali (bukan hanya satu akun yang terkunci) dan seluruh langkah di atas gagal, **berhenti mencoba memperbaiki sendiri** dan minta reset ke panitia — ini fallback resmi tingkat kompetisi, bukan aib:

>

> 1. Informasikan ke juri/teknisi: asal sekolah + nama tim (IP tim opsional).

> 2. Reset menghapus **seluruh progres sebelumnya** di mesin itu — sebelum minta reset, catat dulu perubahan/command penting yang sudah dilakukan di notepad terpisah agar bisa diulang cepat.

> 3. Durasi reset: sekitar 5–8 menit, **tidak ada penambahan waktu pengerjaan**.

> 4. Ada sistem antrian (queue) bila beberapa tim reset bersamaan — makin cepat minta, makin cepat giliran.

>

> Karena reset memotong waktu lomba tanpa kompensasi, ini murni pilihan darurat — pakai hanya setelah checklist troubleshooting di atas benar-benar buntu, bukan sebagai jalan pintas dari awal.

  

---

  

## 1.21 — Kesalahan Umum

  

*Kategori: PEMULIHAN & EVALUASI*

  

**Kesalahan Umum Peserta**

  

23 kesalahan yang paling sering terjadi — tiga terakhir adalah tambahan revisi v2.

  

1. Menganggap PAM hanya Password Policy.

2. Menghapus unknown account tanpa dependency check.

3. Membuat administrator baru tanpa izin objective.

4. Menonaktifkan Administrator sebelum recovery teruji.

5. Menganggap rename Administrator sudah cukup aman.

6. Memasukkan user RDP ke Administrators padahal tidak perlu.

7. Mengubah seluruh User Rights Assignment sekaligus.

8. Lupa bahwa Deny mengalahkan Allow.

9. Menguji lockout pada akun utama.

10. Mengira password policy berlaku retroaktif.

11. Memakai nilai hafalan walaupun objective memberi nilai berbeda.

12. Menganggap Apply sama dengan effective.

13. Menganggap session aktif sama dengan reconnect berhasil.

14. Mengabaikan Domain GPO.

15. Menghapus Log on as a service dari service account.

16. Mematikan UAC agar tidak mengganggu.

17. Mengaktifkan reversible encryption.

18. Tidak menyimpan before state.

19. Tidak menyiapkan rollback.

20. Tidak mengambil evidence.

21. **Mengabaikan checkbox "Password never expires" per-akun** saat menyimpulkan Maximum password age tidak jalan.

22. **Salah paham lockout threshold 0** — dikira error/belum diatur, padahal berarti lockout sengaja dinonaktifkan.

23. **Memakai nama security option dari versi OS yang salah** saat mencari setting di secpol.msc.

  

---

  

## 1.22 — Threat Mapping

  

*Kategori: PEMULIHAN & EVALUASI*

  

| Temuan | Ancaman | Mitigasi |

|---|---|---|

| Terlalu banyak admin | Privilege abuse | Least privilege |

| Akun asing aktif | Persistence | Investigate lalu disable/remove sesuai objective |

| Guest aktif | Unnecessary access | Disable Guest |

| Password pendek | Brute force | Length dan complexity |

| History 0 | Password reuse | Enforce history |

| Lockout 0 tanpa alasan objective | Unlimited guessing | Set threshold sesuai baseline/objective |

| Threshold terlalu kecil | Lockout DoS | Nilai proporsional |

| Reversible encryption aktif | Credential exposure | Disabled |

| LM/NTLM lama | Weak authentication | NTLMv2 dan compatibility test |

| UAC mati | Uncontrolled elevation | Enable UAC |

| User biasa menjadi admin | Dampak kompromi penuh | Remove admin membership |

| Deny RDP salah | Self-lockout | Recovery dan login test |

| Service account boleh interactive logon | Credential abuse | Batasi logon rights |

| Local policy ditimpa GPO | False confidence | Check effective policy |

| Anonymous SAM enumeration diizinkan | Account reconnaissance | Enable "Do not allow anonymous enumeration of SAM" |

  

---

  

## 1.23 — Memory Cheat Sheet

  

*Kategori: HAFALAN*

  

Bahan belajar untuk dihafal sebelum closed-book — bukan alat hafalan tercepat saat lomba (itu tugas 1.25 Combat Card).

  

### Alur Utama

  

```

OBJECTIVE → AUTHORIZED → BASELINE → RECOVERY → CHANGE ONE → VERIFY → LOGIN TEST → EVIDENCE

```

  

### Empat Pertanyaan Akun

  

```

WHO? → WHY? → WHAT POWER? → STILL NEEDED?

```

  

### Lima GUI

  

```

compmgmt.msc

lusrmgr.msc

secpol.msc

rsop.msc

services.msc

```

  

- **Password** — 14 / 24 / 60 / 1 / ON / OFF

- **Lockout** — 10 / 15 / 15

(0 = lockout dimatikan, bukan error)

  

### Golden Warnings

  

- Angka soal mengalahkan angka hafalan.

- Unknown bukan otomatis malicious.

- Jangan membuat admin baru tanpa izin.

- Jangan disable Administrator sebelum admin sah lain teruji.

- Deny mengalahkan Allow.

- Session aktif bukan reconnect test.

- Local policy belum tentu effective policy.

- Jangan ganggu service account tanpa dependency check.

- "Password never expires" per-akun mengalahkan Maximum password age.

- Cek versi OS sebelum cari nama security option.

  

---

  

## 1.24 — Closed-Book Training

  

*Kategori: HAFALAN*

  

Empat level latihan, dari berpandu penuh sampai simulasi lomba.

  

### Level 1 — Dengan Panduan

  

```

Objective:

Ensure that the Guest account is disabled.

```

  

Jawab: GUI path · risiko · verifikasi · rollback · evidence.

  

### Level 2 — Objective Saja

  

**Objective A** — `Ensure standard user analyst1 is not a local administrator but can still use Remote Desktop.` Temukan: keluarkan dari Administrators · pertahankan akun · cek Remote Desktop Users · cek Allow/Deny RDP · login baru · evidence.

  

**Objective B** — `Configure minimum password length to 12 and enforce password complexity.` Jebakan: objective minta 12, jangan pakai 14; efek diuji pada password baru; cek juga akun dengan "Password never expires".

  

**Objective C** — `Lock accounts after 5 invalid attempts for 30 minutes and reset the counter after 30 minutes.` Jebakan: jangan pakai 10/15/15; jangan tes pakai admin utama.

  

**Objective D** — `Disable the built-in Administrator account safely.` Tolak langsung disable sebelum: authorized list · recovery admin · login baru · dependency · rollback.

  

**Objective E** — `Allow helpdesk1 to read event logs without making it an administrator.` Target: gunakan Event Log Readers, least privilege, verifikasi fungsi.

  

> **⚠️ Objective F (baru)**

>

> `Disable account lockout for the svc_backup service account because it triggers false positives during scheduled backups.`

>

> Target: pahami bahwa ini butuh threshold = 0, bukan menghapus policy; pastikan scoped dengan tepat (per-akun via Fine-Grained Password Policy jika domain, atau catat sebagai pengecualian terdokumentasi jika local policy berlaku global); evidence dan alasan wajib dicatat karena ini melemahkan proteksi brute-force untuk akun tersebut.

  

### Level 3 — Tanpa Panduan

  

Mesin berisi: akun authorized · akun unknown · service account · user biasa di Administrators · Guest aktif · password policy lemah · Deny RDP salah · akun dengan Account expires sudah lewat tapi masih enabled. Buat rencana lengkap tanpa panduan.

  

### Level 4 — Simulasi Lomba

  

Diberikan: waktu · objective · jebakan · skor · penalti lockout · daftar evidence · penilaian akhir.

  

### Status Kemampuan

  

- BELUM PAHAM

- PAHAM TEORI

- BISA DENGAN PANDUAN

- BISA TANPA PANDUAN

- SIAP LOMBA

  

*Latihan closed-book dan objective bernilai berbeda dari baseline harus terus dipertahankan sampai level "SIAP LOMBA" tercapai.*

  

---

  

## 1.25 — Final Combat Card

  

*Kategori: HAFALAN — SATU LAYAR*

  

**W1 = AKUN + GRUP + PASSWORD + LOCKOUT + LOGON RIGHTS + UAC**

  

Alur di bawah identik kata-per-kata dengan 1.3 dan 1.15. Ini kartu terakhir yang kamu lihat sebelum masuk sesi RDP lomba.

  

```

1. BACA OBJECTIVE

2. KENALI JENIS SERVER

3. AUDIT AUTHORIZED USERS

4. CEK ADMINISTRATORS

5. CEK SERVICE DEPENDENCY

6. VALIDASI RECOVERY SAH

7. UBAH SATU HAL

8. VERIFY

9. LOGIN BARU

10. EVIDENCE

```

  

#### GUI

  

- compmgmt.msc

- lusrmgr.msc

- secpol.msc

- rsop.msc

- services.msc

  

#### Baseline Angka

  

- **PASSWORD:** 14 / 24 / 60 / 1 / ON / OFF

- **LOCKOUT:** 10 / 15 / 15

- (0 = sengaja dimatikan)

  

#### Ingat

  

- OBJECTIVE WINS

- DENY WINS

- LOGIN BARU

- EFFECTIVE POLICY

- EVIDENCE

- CEK VERSI OS UNTUK NAMA SETTING

  

> **⚠️ [FIX-v3] Estimasi Waktu Eksekusi Saat Lomba (bukan waktu belajar)**

>

> Modul ini sengaja dibuat sedalam mungkin untuk latihan — itu bukan cerminan waktu yang wajar dipakai saat lomba sungguhan. Hari 1 (Hardening) hanya **3 jam total (09.15–12.15)** untuk *kedua* mesin (Linux + Windows) dan *seluruh* kategori kisi-kisi sekaligus — PAM hanyalah satu dari enam kategori Windows (lihat 1.0), belum termasuk kisi-kisi Linux.

>

> Perkiraan kasar dan realistis (bukan angka resmi — bobot per objektif tidak di-disclose panitia): alokasikan sekitar **15–25 menit** untuk keseluruhan objective PAM Windows (audit akun/grup, built-in Administrator, Guest, Password Policy, Account Lockout, Security Options terkait, UAC, User Rights Assignment) dalam satu putaran alur kanonis 10-langkah — bukan mengulang seluruh detail 17-poin per setting seperti saat belajar. Sisakan waktu untuk lima kategori Windows lainnya dan seluruh kisi-kisi Linux.

>

> Kalau satu objective terasa buntu lebih dari beberapa menit: lompat dulu ke objective lain, kembali lagi nanti — jangan korbankan seluruh kategori lain demi satu setting yang macet.

  

---

  

## Lampiran — Standar Kualitas

  

*Kategori: QA MODUL*

  

**Standar Kualitas Akhir**

  

Checklist internal — modul ini dianggap selesai apabila semua baris berikut terpenuhi.

  

- [ ] GUI-first

- [ ] Ramah pemula

- [ ] Tidak melebar ke W2/W3

- [ ] Tidak menyebabkan instruksi self-lockout ATAU memutus akses juri

- [ ] Membedakan standalone/member/DC

- [ ] Membedakan local dan effective policy

- [ ] Memiliki dependency check

- [ ] Memiliki before/after

- [ ] Memiliki verification setting dan function

- [ ] Memiliki login baru/reconnect test

- [ ] Memiliki rollback

- [ ] Memiliki evidence

- [ ] Memiliki troubleshooting

- [ ] Memiliki closed-book training

- [ ] Memiliki Memory Cheat Sheet

- [ ] Memiliki Final Combat Card

- [ ] Tidak menggunakan baseline lama (12/5/90/1)

- [ ] Tidak mengandung instruksi membuat admin tanpa izin

- [ ] Tidak ada materi teknis yang bertentangan antarbagian

- [ ] Alur kerja 10-langkah sama persis di 1.3, 1.15, dan 1.25

- [ ] Nama security option version-aware dicantumkan di 1.11

- [ ] Account expires masuk checklist initial assessment (1.4)

- [ ] Trap "Password never expires" vs Maximum password age dijelaskan (1.9, 1.20, 1.21)

- [ ] Makna lockout threshold = 0 dijelaskan (1.10, 1.21, 1.23, 1.25)

- [ ] [FIX-v3] Evidence Pack menyatakan eksplisit statusnya (deliverable wajib atau alat latihan saja) sesuai aturan penilaian resmi hari yang bersangkutan (1.19)

- [ ] [FIX-v3] Troubleshooting & Rollback mencantumkan fallback reset resmi tingkat kompetisi sebagai langkah terakhir yang sah (1.20)

- [ ] [FIX-v3] Prinsip anti-lockout mencakup kontinuitas akses juri/penilai, tidak hanya akses peserta (1.3, 1.20)

- [ ] [FIX-v3] Modul menyertakan estimasi waktu eksekusi saat lomba, terpisah dari kedalaman materi belajar (1.25)

- [ ] [FIX-v3] Mekanisme yang statusnya belum dikonfirmasi resmi ditandai 🔸 dan diminta dicek ulang mendekati hari-H (1.0, 1.3, 1.20)

- [ ] [FIX-v3] Nama kategori kisi-kisi resmi dikutip verbatim di bagian pembuka modul (1.0)

- [ ] [FIX-v4] Local Users and Groups tidak diklaim punya tab "Account"; audit expiration lewat command-assisted exception dijelaskan jujur (1.4)

- [ ] [FIX-v4] "Allow Administrator account lockout" untuk RID 500 dijelaskan sebagai version-aware (1.7, 1.10)

- [ ] [FIX-v4] Dua setting anonymous SAM enumeration dicantumkan terpisah, tidak digabung "(and shares)" (1.11)

- [ ] [FIX-v4] Klaim penerapan struktur 17 poin tidak melebihi apa yang benar-benar ditulis di modul (1.17)

- [ ] [FIX-v4] Istilah baseline latihan tidak berpotensi dikira baseline resmi Microsoft (1.9, 1.10)

- [ ] [FIX-v4] Template Referensi Aturan Lomba tersedia untuk audit-trail klaim resmi (1.0)

  

*Target hasil akhir: Competition Ready, GUI-First, Anti-Lockout, Mudah Dipahami, dan Mudah Dihafalkan.*

  

---

  

## Catatan Produksi (Meta-QC Dokumen Ini)

  

Audit sebelum dokumen ini dianggap selesai sebagai Master Module:

  

- [x] Tidak ada tag HTML, CSS, atau JavaScript di dalam isi modul

- [x] Satu Alur Kanonis 10-langkah dipakai identik di 1.3, 1.15, dan 1.25 — tidak ada versi flow lain

- [x] Tidak ada instruksi membuat administrator baru tanpa izin objective

- [x] Tidak ada instruksi yang berpotensi menyebabkan self-lockout tanpa peringatan dan langkah recovery

- [x] Local account, domain account, dan Domain Controller dibedakan secara eksplisit (1.4)

- [x] Account expires dan status kedaluwarsa akun masuk checklist initial assessment (1.4, 1.5)

- [x] "Password never expires" per-akun dijelaskan sebagai pengecualian terhadap Maximum password age (1.9, 1.20, 1.21)

- [x] Makna lockout threshold = 0 dijelaskan sebagai kondisi sengaja, bukan error (1.10, 1.21, 1.22, 1.25)

- [x] Prinsip Objective Wins dan Deny Wins konsisten di seluruh bagian

- [x] GUI-first di seluruh modul; command/PowerShell hanya sebagai alat verifikasi (1.16)

- [x] Checklist QA lengkap tersedia di bagian Lampiran (36 butir, termasuk lima perbaikan [FIX-v3] dan enam perbaikan [FIX-v4])

- [x] [FIX-v4] Tidak ada klaim GUI yang tidak sesuai perilaku sebenarnya (Tab Account, penamaan setting SAM enumeration) diverifikasi ulang lewat review teknis independen

  

Status: **Competition Ready.**