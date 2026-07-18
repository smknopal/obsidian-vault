# WGUI-3 — Active Directory Hardening

  

Seri: **WGUI — Windows GUI Hardening Learning System**

  

---

  

## WGUI-3.0 — Posisi dan Batas Modul

  

```text

WGUI-1 = mengamankan akun pada satu mesin

WGUI-2 = menerapkan policy secara terpusat

WGUI-3 = mengamankan identitas dan struktur domain

```

  

> [!NOTE]

> WGUI-3 tidak mengulang teori GPO, LSDOU, atau precedence dari WGUI-2. Fokus di sini: user, group, OU, delegasi, dan policy password domain.

  

---

  

## WGUI-3.1 — Gambaran Besar Active Directory

  

Analogi: domain = gedung perusahaan.

  

| Istilah | Analogi |

|---|---|

| Forest | Kompleks gedung |

| Domain | Satu gedung |

| Domain Controller (DC) | Kantor keamanan/resepsionis pusat |

| OU | Lantai/departemen |

| Object | Barang/orang terdaftar |

| User | Karyawan |

| Group | Divisi/kartu akses |

| Computer | Ruangan kerja |

| Security Principal | Apapun yang bisa punya izin akses |

| SID | Nomor identitas unik permanen |

  

---

  

## WGUI-3.2 — Peta GUI Active Directory

  

| Tool | Cara buka | Fungsi |

|---|---|---|

| Server Manager | Otomatis saat login | Pusat kelola role |

| ADUC | `dsa.msc` | Kelola user/group/OU (klasik) |

| AD Administrative Center | `dsac.exe` | Kelola AD + FGPP (modern) |

| GPMC | `gpmc.msc` | Hanya penghubung ke WGUI-2 |

| AD Sites and Services | `dssite.msc` | Topologi replikasi |

| DNS Manager | `dnsmgmt.msc` | DNS domain |

  

---

  

## WGUI-3.3 — Pemeriksaan Awal Domain Controller

  

Checklist IDENTIFY/BASELINE:

  

- [ ] Nama domain & hostname DC

- [ ] Role server terpasang

- [ ] Waktu sistem sesuai

- [ ] DNS yang digunakan DC

- [ ] Jumlah DC dalam domain

- [ ] Status dasar replikasi (jika >1 DC)

- [ ] Akun recovery yang sah tercatat

- [ ] Akses console/recovery tersedia

  

> [!WARNING]

> Kesalahan DNS, waktu sistem, atau replikasi bisa **terlihat** seperti kesalahan akun. Periksa dasar ini dulu sebelum menuduh user/group.

  

---

  

## WGUI-3.4 — Authorized Account Baseline

  

```text

AUTHORIZED LIST vs ACTUAL USERS vs ACTUAL PRIVILEGES vs SERVICE DEPENDENCIES

```

  

> [!NOTE]

> Unknown account **bukan otomatis malicious**. Cocokkan dulu dengan authorized list dan objective sebelum bertindak.

  

---

  

## WGUI-3.5 — Audit Domain Users

  

**Baseline yang diperiksa** (ADUC → Properties user):

- Enabled/disabled

- Password expired / never expires

- User cannot change password

- Must change password at next logon

- Account expiration date

- Logon hours/workstation restriction

- Stale account (lama tidak login)

- Duplikat atau akun asing

  

**Lokasi GUI:** `dsa.msc` → klik kanan user → *Properties* (tab *Account*, *General*)

  

**Langkah aman:** catat kondisi awal → bandingkan dengan authorized list → ubah satu atribut → verifikasi → evidence.

  

> [!TIP]

> Gunakan kolom tambahan di ADUC (*View → Add/Remove Columns*) untuk melihat banyak akun sekaligus tanpa membuka satu per satu.

  

---

  

## WGUI-3.6 — Audit Privileged Groups

  

| Grup | Fungsi Singkat | Risiko |

|---|---|---|

| Domain Admins | Kontrol penuh domain | Sangat tinggi |

| Enterprise Admins | Kontrol penuh forest | Sangat tinggi |

| Schema Admins | Ubah schema AD | Sangat tinggi |

| Administrators (domain local) | Admin builtin domain | Tinggi |

| Account Operators | Kelola user/group non-admin | Sedang-tinggi |

| Server Operators | Kelola server anggota domain | Sedang-tinggi |

| Backup Operators | Backup/restore data | Sedang-tinggi (bisa bypass ACL) |

| Group Policy Creator Owners | Buat GPO baru | Sedang |

| DnsAdmins | Kelola DNS domain | Sedang-tinggi |

  

> [!DANGER]

> **Nested membership** dapat memberikan privilege tersembunyi. Grup A anggota Grup B (privileged) tetap mewarisi privilege meski tidak terlihat langsung. Selalu cek *Member Of* secara berlapis.

  

---

  

## WGUI-3.7 — Prosedur Aman Mengurangi Privilege

  

```text

READ OBJECTIVE → RECORD MEMBERSHIP → IDENTIFY ACCOUNT

→ CHECK DEPENDENCY → REMOVE ONE MEMBERSHIP

→ VERIFY → NEW LOGIN TEST → EVIDENCE

```

  

**Risiko:** salah keluarkan akun sah dari grup kritikal dapat menghentikan operasional.

**Verifikasi:** `whoami /groups` dari sesi login baru (bukan sesi lama).

**Rollback:** tambahkan kembali membership yang sama persis.

  

> [!DANGER]

> Jangan melakukan penghapusan massal. Satu perubahan, satu verifikasi.

  

---

  

## WGUI-3.8 — Administrator dan Guest Domain

  

Administrator **domain** ≠ Administrator **lokal** (WGUI-1). Administrator domain berlaku di seluruh domain.

  

**Jangan:**

- Disable Administrator sebelum admin sah lain teruji bisa login

- Membuat admin baru tanpa izin objective

- Menganggap rename akun sudah menghilangkan seluruh risiko

- Mengaktifkan Guest tanpa kebutuhan jelas

  

---

  

## WGUI-3.9 — Service Account Awareness

  

Kenali akun yang dipakai: Windows service, scheduled task, application pool, database, backup, sinkronisasi/aplikasi domain.

  

**Langkah aman:** identifikasi pemakai akun → cek dependency (service apa yang login pakai akun ini) → baru reset password/disable/kurangi privilege → verifikasi service tetap jalan.

  

> [!DANGER]

> Perubahan password service account dapat mematikan service terkait secara langsung.

  

---

  

## WGUI-3.10 — OU dan Penempatan Objek

  

- OU ≠ default container (`Users`, `Computers` bawaan tidak bisa di-link GPO)

- Pisahkan: user, workstation, server, admin ke OU masing-masing

- Gunakan test OU untuk uji coba

- Memindahkan objek antar-OU **mengubah GPO efektif** objek tersebut

- Aktifkan *Protect object from accidental deletion* pada OU penting

  

> [!WARNING]

> Jangan memindahkan Domain Controller keluar dari `Domain Controllers OU`.

  

---

  

## WGUI-3.11 — Delegation of Control

  

**Konsep:** delegasi tugas spesifik lebih aman daripada memberi Domain Admins.

  

Delegasi **bukan** aksi sekali-jalan yang bisa "dibatalkan" dengan menjalankan wizard ulang. Wizard *Delegate Control* hanya **menambahkan** Access Control Entry (ACE) baru ke ACL objek — wizard tidak punya mode pencabutan. Mencabut delegasi berarti mengelola ACE itu langsung di *Security tab*.

  

**Siklus delegasi yang benar:**

  

```text

CREATE → VERIFY → AUDIT → REMOVE ACE

```

  

| Tahap | Isi |

|---|---|

| Create | Jalankan wizard *Delegate Control* pada OU target, beri tugas spesifik |

| Verify | Cek ACE baru muncul di *Security tab* OU, uji dengan akun delegasi |

| Audit | Tinjau berkala — ACE ini masih dibutuhkan atau tidak |

| Remove ACE | Hapus ACE delegasi secara langsung saat sudah tidak diperlukan |

  

**Langkah GUI — Create:**

klik kanan OU → *Delegate Control...* → tentukan siapa (user/group) → tentukan tugas spesifik (reset password, kelola grup, dsb.) → jangan delegasikan di root domain tanpa alasan.

  

**Langkah GUI — Rollback (Remove ACE):**

  

```text

ADUC

→ View

→ Advanced Features

→ OU Properties

→ Security

→ Advanced

→ Identifikasi ACE delegasi

→ Remove ACE yang sesuai

→ Apply

→ Test ulang akun delegasi

```

  

**Verifikasi:** buka *Advanced Features* → tab *Security* pada OU → cek ACE yang ditambahkan/dihapus, lalu uji ulang akses akun delegasi (harus **gagal** melakukan tugas yang tadi diberikan).

  

> [!DANGER]

> Jangan menghapus permission bawaan Active Directory. Hapus hanya ACE yang dibuat saat proses delegasi.

  

---

  

## WGUI-3.12 — Domain Password Policy

  

Berbeda dari local password policy (WGUI-1).

  

Satu domain memiliki **satu** Account Policy efektif. Biasanya dikonfigurasi melalui *Default Domain Policy*. Namun GPO lain pada root domain dengan precedence lebih tinggi dapat memengaruhi hasil — jadi jangan berasumsi *Default Domain Policy* selalu jadi sumber yang berlaku.

  

**Lokasi:** GPMC → *Default Domain Policy* → *Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies*

  

**Risiko:** mengubah nilai memengaruhi **semua** user domain sekaligus.

  

**Verifikasi GUI (utama):**

  

```text

GPMC

→ Domain

→ Linked Group Policy Objects

→ cek precedence

→ edit GPO efektif

→ Account Policies

```

  

`gpresult /r` hanya pendukung, bukan satu-satunya bukti — hasil `gpresult` bisa memuat cache atau perlu di-refresh. Bukti utama tetap urutan precedence GPO yang ter-link di root domain, dicek langsung lewat GPMC.

  

> [!MEMORY]

> Angka dalam objective lomba mengalahkan angka contoh latihan.

  

---

  

## WGUI-3.13 — Fine-Grained Password Policy (FGPP)

  

**Kapan dipakai:** saat sekelompok user butuh aturan password berbeda dari domain policy umum.

  

**Lokasi GUI (Create):** `dsac.exe` → *System → Password Settings Container* → *New → Password Settings*

  

- Target: user atau **global security group** (bukan OU langsung)

- Precedence: ditentukan oleh nilai *Precedence* terkecil yang menang

- Risiko: konflik antar-PSO jika precedence tidak direncanakan

  

**Verifikasi GUI — cek konfigurasi PSO:**

  

```text

AD Administrative Center

→ System

→ Password Settings Container

→ Password Settings

→ Properties

```

  

**Verifikasi GUI — cek hasil efektif pada user:**

  

```text

AD Administrative Center

→ User

→ Tasks

→ View Resultant Password Settings

```

  

(Alternatif: tab *Attribute Editor* → atribut `msDS-ResultantPSO` pada user.)

  

**Rollback FGPP (bukan "disable PSO" — PSO tidak punya mode disable sederhana):**

  

```text

1. Identifikasi PSO yang dibuat

2. Lepaskan target user/group

3. Kembalikan konfigurasi baseline

4. Hapus PSO jika sudah tidak diperlukan

```

  

> [!WARNING]

> Melepas target lebih aman dilakukan lebih dulu daripada langsung menghapus PSO — pastikan tidak ada dependency lain sebelum PSO dihapus permanen.

  

---

  

## WGUI-3.14 — Domain Controller Safety

  

> [!DANGER]

> - Jangan gunakan `lusrmgr.msc` di DC seperti member server biasa

> - Jangan hapus akun, role, service, DNS, SYSVOL, atau share penting secara buta

> - Jangan edit *Default Domain Controllers Policy* sembarangan

> - AD bergantung pada DNS, waktu, SYSVOL, dan replikasi — periksa semua sebelum menuduh akun

  

**DSRM** = mode recovery darurat DC, bukan akun login domain harian.

  

---

  

## WGUI-3.15 — Protected Accounts dan AdminSDHolder

  

- Akun privileged tertentu (mis. Domain Admins) mendapat perlindungan permission khusus

- Permission pada akun ini dapat **kembali sendiri** karena mekanisme proteksi berkala

- Jangan terus mengubah permission tanpa tahu sumber masalahnya

- Verifikasi dulu apakah objek termasuk protected account sebelum heran permission "kembali"

  

---

  

## WGUI-3.16 — Verification, Rollback, dan Evidence

  

Bukti wajib per perubahan: screenshot kondisi awal, objek yang diubah, membership sebelum/sesudah, lokasi OU, effective password policy, hasil login baru, kondisi service terkait, alasan perubahan.

  

Command verifikasi opsional:

```text

whoami

whoami /groups

net user username /domain

gpresult /r

```

GUI tetap cara utama; command hanya pelengkap.

  

---

  

## WGUI-3.17 — Troubleshooting Decision Tree

  

```text

SYMPTOM

→ CHECK ACCOUNT → CHECK GROUP → CHECK POLICY

→ CHECK OU → CHECK DEPENDENCY → CHECK REPLICATION

→ ROLLBACK

```

  

Masalah umum: user tidak bisa login • privilege masih ada setelah dikeluarkan dari grup (cek sesi lama vs baru) • akun terkunci • password policy tidak sesuai • OU berubah memengaruhi GPO • service account berhenti • hasil beda antar-DC (replikasi) • objek tak bisa diedit (permission/AdminSDHolder) • permission kembali sendiri (protected account).

  

**Studi kasus — "User Tidak Bisa Login":**

  

```text

USER TIDAK BISA LOGIN

↓

Check Account Status

↓

Check Group Membership

↓

Check Password Policy

↓

Check OU/GPO

↓

Check Replication

↓

Rollback

```

  

Setiap panah = satu pemeriksaan GUI di ADUC/dsac/GPMC/AD Sites and Services sebelum lanjut ke pemeriksaan berikutnya. Jangan lompat langsung ke rollback tanpa melewati urutan ini.

  

---

  

## WGUI-3.18 — Closed-Book Training

  

Untuk tiap skenario, jawab: apa diperiksa dulu, risiko terbesar, perubahan paling aman, verifikasi, rollback, evidence.

  

1. Akun asing di Domain Admins

2. User biasa punya privilege terlalu tinggi

3. Akun service terlihat mencurigakan

4. Password policy domain tidak sesuai objective

5. Delegasi OU terlalu luas

6. Perubahan menyebabkan user tidak bisa login

  

---

  

## WGUI-3.19 — GUI Practical Lab

  

Empat mini-lab berikut mensimulasikan situasi lomba: temukan masalah → ambil tindakan aman → verifikasi → evidence.

  

### LAB 1 — Audit Domain Admin Membership

  

**Skenario:** Ada akun yang tidak dikenal dalam Domain Admins.

**Tujuan:** Menentukan apakah akun valid atau risiko.

  

**Langkah GUI:**

```text

ADUC

→ Users

→ Domain Admins

→ Members

```

  

**Periksa:**

- Nama akun

- Nested group

- Authorized list

  

**Tindakan aman:**

```text

Record → Check dependency → Remove membership jika valid risiko

→ Login ulang → Verify

```

  

**Verifikasi:** `whoami /groups` dari sesi login baru.

  

**Evidence:** screenshot *Before*, *After*, dan daftar *Membership*.

  

---

  

### LAB 2 — Audit User Account

  

**Skenario:** User memiliki konfigurasi tidak sesuai.

  

**Periksa:**

```text

ADUC → User Properties

```

  

**Tab Account:**

- Password never expires

- Account disabled

- Expiration

- Logon restriction

  

**Tab Member Of:**

- Group membership

  

> [!WARNING]

> Jangan langsung *delete*. *Disable* lebih aman.

  

---

  

### LAB 3 — Delegation OU

  

**Skenario:** Helpdesk boleh reset password user, tetapi tidak boleh menjadi Domain Admin.

  

**Langkah:**

1. Buat test OU

2. OU → *Delegate Control*

3. Berikan tugas: *Reset user password*

  

**Verifikasi:**

- Login menggunakan akun helpdesk

- Tes reset password (harus berhasil)

- Tes akses Domain Admins (harus gagal)

  

**Rollback:** Remove ACE delegasi (lihat WGUI-3.11).

  

---

  

### LAB 4 — FGPP

  

**Skenario:** Administrator membutuhkan password policy lebih kuat untuk sekelompok user tertentu.

  

**Langkah:**

```text

AD Administrative Center

→ System

→ Password Settings Container

→ New Password Settings

```

  

**Target:** Global Security Group

  

**Atur:**

- Minimum password length

- Complexity

- Lockout

  

**Verifikasi:** *View Resultant Password Settings* (lihat WGUI-3.13).

  

---

  

## Golden Warnings

  

> [!DANGER]

> - Unknown bukan otomatis malicious

> - Disable biasanya lebih aman daripada delete

> - Jangan hapus akun sebelum dependency diperiksa

> - Jangan kurangi privilege secara massal

> - Jangan buat Domain Admin baru tanpa izin

> - Nested membership bisa sembunyikan privilege

> - Session aktif bukan bukti login berikutnya berhasil

> - Reset password service account bisa mematikan service

> - Pindah OU bisa mengubah GPO efektif

> - Jangan ubah Default Domain Policy sembarangan

> - Jangan pindahkan DC keluar dari OU resminya

> - Jangan perlakukan DC seperti standalone server

> - Apply berhasil ≠ konfigurasi aman

> - Screenshot setting saja tidak cukup; verifikasi hasil efektif

> - Delegasi tidak dicabut lewat wizard ulang — cabut lewat Remove ACE

> - PSO tidak punya "disable" — rollback lewat lepas target lalu hapus PSO

> - Jangan reset **KRBTGT** tanpa objective, alasan insiden, dan prosedur terkontrol

  

---

  

## Memory Cheat Sheet

  

```text

AD = IDENTITAS + GRUP + OU + DELEGASI + POLICY

  

1. KENALI DOMAIN DAN DC

2. BACA AUTHORIZED LIST

3. AUDIT USER

4. AUDIT PRIVILEGED GROUP

5. CEK NESTED MEMBERSHIP

6. CEK SERVICE DEPENDENCY

7. CEK OU DAN DELEGASI

8. UBAH SATU HAL

9. VERIFY DAN LOGIN BARU

10. EVIDENCE

```

Mnemonic: **"Diam Dulu, Grup Diperiksa"** (Domain-DC → Grup/OU → Delegasi/Policy).

  

> [!MEMORY]

> **AD Combat Memory**

> ```text

> AD = USER + GROUP + OU + DELEGATION + POLICY

> ```

> Urutan lomba:

> 1. Kenali Domain Controller

> 2. Audit User

> 3. Audit Privileged Group

> 4. Cek Nested Membership

> 5. Cek Service Account

> 6. Cek OU

> 7. Cek Delegation

> 8. Cek Password Policy

> 9. Change One Thing

> 10. Verify + Evidence

  

---

  

## Final Combat Card

  

```text

ALUR: IDENTIFY → BASELINE → RECOVERY → CHECK RISK → CHANGE → VERIFY → RECONNECT → EVIDENCE

  

5 GUI UTAMA:

dsa.msc | dsac.exe | gpmc.msc | dssite.msc | dnsmgmt.msc

  

PRIVILEGED GROUPS PRIORITAS:

Domain Admins • Enterprise Admins • Schema Admins • Account Operators

Server Operators • Backup Operators • GPO Creator Owners • DnsAdmins

  

GOLDEN WARNINGS: lihat bagian di atas — baca ulang sebelum ubah apa pun

  

VERIFIKASI WAJIB: sesi login baru, whoami /groups, gpresult /r (pendukung),

GPMC precedence check, resultant PSO

  

ROLLBACK CEPAT:

- Membership → kembalikan ke baseline tercatat

- Delegasi → Remove ACE (bukan wizard ulang)

- FGPP → lepas target → kembalikan baseline → hapus PSO jika perlu

  

EVIDENCE: screenshot before/after + alasan perubahan

```

  

### Kuis 10 Soal

  

1. Apa bedanya OU dengan default container? → *OU bisa di-link GPO, container tidak*

2. Kenapa unknown account tidak boleh langsung dihapus? → *Belum tentu malicious, cek dependency dulu*

3. Grup apa yang setara kontrol penuh forest? → *Enterprise Admins*

4. Apa risiko Backup Operators? → *Bisa bypass ACL lewat backup/restore*

5. Kenapa nested membership berbahaya? → *Privilege tersembunyi lewat grup dalam grup*

6. Apa beda domain password policy dan FGPP? → *Domain policy global; FGPP untuk grup/user tertentu dengan precedence*

7. Kenapa tidak boleh pindahkan DC dari OU-nya? → *Mengubah Default Domain Controllers Policy yang berlaku*

8. Apa itu DSRM? → *Mode recovery darurat DC, bukan akun harian*

9. Kenapa permission bisa "kembali sendiri"? → *Protected account/AdminSDHolder*

10. Bagaimana cara mencabut delegasi yang benar? → *Remove ACE lewat Security → Advanced, bukan jalankan wizard ulang*

  

### Status Kesiapan

  

```text

BELUM PAHAM

PAHAM TEORI

BISA DENGAN PANDUAN

BISA TANPA PANDUAN

SIAP LOMBA

```

  

---

  

## Final Quality Check

  

```text

✓ Fokus Active Directory

✓ GUI-first

✓ Bisa dipraktikkan tanpa CLI

✓ Ada langkah perubahan aman

✓ Ada rollback (delegasi & FGPP diperbaiki)

✓ Ada evidence

✓ Mudah dihafal (Memory Cheat Sheet + AD Combat Memory)

✓ Tidak terlalu panjang

```

  

WGUI-3 ACTIVE DIRECTORY HARDENING FINAL COMPETITION READY

  

```text

FILE:

WGUI-3_Active_Directory_Hardening.md

  

STATUS:

READY — FINAL REVISION

  

SCOPE CHECK:

Fokus AD hardening (user, group, OU, delegasi, password policy);

teori GPO tidak diulang; tidak ada teknik red team/exploit/Kerberos attack/BloodHound/trust attack

  

MAIN SAFETY CHECK:

Setiap perubahan berisiko memiliki verifikasi, rollback, evidence;

golden warnings lengkap; delegasi & FGPP rollback diperbaiki sesuai mekanisme aslinya

```