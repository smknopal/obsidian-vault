# W1 — Privileged Access Management (PAM)

## Local Security Policy Hardening

  

*Modul Persiapan LKSN Cyber Security*

  

Modul ini membahas cara mengaudit dan mengamankan akun lokal Windows melalui Local Security Policy — dengan fokus pada pemahaman konsep, kemampuan praktik, dan kecepatan mengingat saat lomba.

  

---

  

## 1. Tujuan Pembelajaran

  

Setelah mempelajari modul ini, peserta mampu:

  

- Memahami konsep dasar PAM (Privileged Access Management) pada Windows.

- Melakukan audit user dan grup pada sistem Windows.

- Mengamankan akun bawaan (built-in accounts) seperti Administrator dan Guest.

- Mengonfigurasi Password Policy sesuai standar hardening.

- Mengonfigurasi Account Lockout Policy secara realistis.

- Memahami dan mengonfigurasi Security Options yang paling berdampak.

- Melakukan verifikasi setelah setiap perubahan konfigurasi.

  

---

  

## 2. Konsep Dasar PAM Windows

  

**Apa itu Privileged Access Management (PAM)?**

  

PAM adalah pendekatan untuk mengontrol, membatasi, dan mengawasi akun-akun yang punya hak akses tinggi (privileged accounts). Semakin sedikit akun yang berkuasa penuh, dan semakin ketat kekuasaan itu diawasi, semakin kecil ruang gerak attacker jika kredensial bocor.

  

**Mengapa akun privilege tinggi jadi target utama attacker?**

  

Satu akun Administrator yang berhasil dibobol = kontrol penuh atas sistem. Attacker tidak perlu membobol banyak akun kecil — cukup satu akun admin untuk membuat user baru, mematikan antivirus, menghapus log, atau memasang backdoor.

  

**Administrator vs Standard User**

  

| Aspek | Administrator | Standard User |

|---|---|---|

| Install/uninstall software | Bisa | Terbatas |

| Ubah system settings | Bisa | Tidak bisa |

| Akses file user lain | Bisa | Tidak bisa |

| Ubah security policy | Bisa | Tidak bisa |

  

**Least Privilege Principle**

  

Setiap akun/proses hanya diberi hak akses minimum yang benar-benar dibutuhkan — tidak lebih. Dalam praktik hardening:

- User biasa tidak perlu jadi Administrator.

- Service tidak perlu berjalan dengan akun SYSTEM kalau tidak dibutuhkan.

- Semakin sedikit orang yang tahu password admin, semakin baik.

  

---

  

## 3. Decision Flow Saat Hardening

  

Sebelum mengubah satu pun konfigurasi, ikuti alur ini:

  

```

CEK KONDISI AWAL

        ↓

PASTIKAN ADA AKUN ADMIN CADANGAN

        ↓

UBAH KONFIGURASI

        ↓

VERIFY

        ↓

CATAT HASIL

```

  

**Kenapa urutan ini penting:**

  

1. **Cek kondisi awal** — Anda harus tahu kondisi sebelum diubah (baseline), supaya bisa membandingkan dan tahu apa yang sudah/belum sesuai standar.

2. **Pastikan ada akun admin cadangan** — langkah yang paling sering dilewatkan dan paling sering menyebabkan lockout total. Sebelum mengubah apa pun terkait akun/akses, pastikan ada minimal satu jalan masuk aman jika perubahan salah.

3. **Ubah konfigurasi** — satu per satu, jangan sekaligus banyak, supaya mudah dilacak kalau ada yang salah.

4. **Verify** — setiap perubahan WAJIB diverifikasi. Policy yang "terlihat sudah diubah" di GUI belum tentu benar-benar aktif.

5. **Catat hasil** — untuk dokumentasi, dan supaya Anda sendiri tidak lupa apa yang sudah dikerjakan (penting untuk penilaian juri).

  

---

  

## 4. Initial Assessment

  

Sebelum hardening, kumpulkan dulu gambaran kondisi sistem.

  

### User Audit

**Command:** `net user` | **PowerShell:** `Get-LocalUser`

**Fungsi:** Menampilkan semua akun lokal yang ada di sistem.

**Informasi yang dicari:** Akun tidak dikenal, akun yang seharusnya nonaktif tapi masih aktif.

**Indikator risiko:** Akun asing yang tidak pernah dibuat admin, nama akun mencurigakan.

  

### Administrator Group Audit

**Command:** `net localgroup administrators`

**Fungsi:** Menampilkan semua anggota grup Administrators.

**Informasi yang dicari:** Siapa saja yang punya privilege admin di mesin ini.

**Indikator risiko:** Anggota yang seharusnya tidak punya privilege admin, jumlah admin berlebih.

  

### Privilege Check

**Command:** `whoami /priv`

**Fungsi:** Menampilkan privilege yang dimiliki user yang sedang login.

**Informasi yang dicari:** Privilege sensitif yang aktif (`SeDebugPrivilege`, `SeBackupPrivilege`, `SeTakeOwnershipPrivilege`, dll).

**Indikator risiko:** Privilege tinggi dimiliki akun yang seharusnya tidak membutuhkannya.

  

### Password Policy Check

**Command:** `net accounts`

**Fungsi:** Menampilkan konfigurasi password policy & lockout policy yang berlaku saat ini.

**Informasi yang dicari:** Minimum password length, password age, lockout threshold saat ini.

**Indikator risiko:** Minimum length terlalu pendek (< 8), lockout threshold 0 (tidak pernah lockout).

  

---

  

## 5. Password Policy Hardening

  

Setiap setting mengikuti pola: **Konsep → Risiko → Konfigurasi → Verifikasi.**

  

**Minimum Password Length**

- Konsep: panjang minimum karakter password.

- Risiko: password pendek jauh lebih cepat ditebak lewat brute force.

- Konfigurasi: `net accounts /minpwlen:8`

- Verifikasi: `net accounts` → cek "Minimum password length".

  

**Complexity**

- Konsep: mewajibkan kombinasi huruf besar, huruf kecil, angka, dan simbol.

- Risiko: password seperti "password123" mudah ditebak dictionary attack.

- Konfigurasi: `secpol.msc → Account Policies → Password Policy → "Password must meet complexity requirements" → Enabled`.

- Verifikasi: cek ulang di secpol.msc, atau coba buat password lemah — harus ditolak sistem.

  

**Password History**

- Konsep: jumlah password lama yang diingat sistem agar tidak dipakai ulang.

- Risiko: user mengganti password lalu langsung kembali ke password lama yang sama.

- Konfigurasi: `net accounts /uniquepw:5`

- Verifikasi: `net accounts` → cek "Length of password history maintained".

  

**Maximum Password Age**

- Konsep: batas waktu maksimum sebelum password wajib diganti.

- Risiko: password yang bocor tetap valid dalam waktu lama.

- Konfigurasi: `net accounts /maxpwage:90`

- Verifikasi: `net accounts` → cek "Maximum password age".

  

**Minimum Password Age**

- Konsep: batas waktu minimum sebelum password boleh diganti lagi.

- Risiko jika 0: user bisa ganti password berkali-kali secara instan untuk "menghabiskan" history dan kembali ke password favorit.

- Konfigurasi: `net accounts /minpwage:1`

- Verifikasi: `net accounts` → cek "Minimum password age".

  

**Reversible Encryption**

- Konsep: opsi menyimpan password dalam bentuk yang bisa didekripsi kembali (bukan hash satu arah).

- Risiko: jika aktif, password setara plaintext — sangat berbahaya.

- Konfigurasi: pastikan **Disabled** (default). Jangan pernah diaktifkan kecuali ada aplikasi legacy yang benar-benar mewajibkan.

- Verifikasi: `secpol.msc → Password Policy → "Store password using reversible encryption"` harus **Disabled**.

  

**Tabel Default vs Hardening:**

  

| Setting | Default Windows | Rekomendasi Hardening |

|---|---|---|

| Minimum Password Length | 0 | 8–12+ |

| Complexity | Bervariasi | Enabled |

| Password History | 0 | 5–24 |

| Maximum Password Age | 42 hari | 60–90 hari |

| Minimum Password Age | 0 | 1 hari |

| Reversible Encryption | Disabled | Tetap Disabled |

  

---

  

## 6. Account Lockout Policy

  

- **Threshold** — jumlah maksimal percobaan login gagal sebelum akun dikunci.

- **Duration** — berapa lama akun tetap terkunci sebelum otomatis terbuka lagi.

- **Reset Counter** — setelah berapa lama hitungan percobaan gagal kembali ke nol jika tidak ada percobaan gagal lagi.

  

**Risiko terlalu longgar** (threshold tinggi/tidak ada lockout): sistem rentan brute force karena attacker bisa mencoba password berkali-kali tanpa batas.

  

**Risiko terlalu ketat** (threshold 2–3 kali): berisiko mengunci diri sendiri — salah ketik password 2-3 kali itu wajar, dan bisa membuat akun sendiri terkunci saat lomba.

  

**Rekomendasi realistis:**

  

| Setting | Rekomendasi |

|---|---|

| Threshold | 5–10 percobaan |

| Duration | 15–30 menit |

| Reset Counter | 15–30 menit |

  

---

  

## 7. User Account Management

  

**Guest Account**

- Risiko: Guest adalah akun anonim tanpa password yang bisa dipakai siapa saja dengan privilege minimal — pintu masuk yang tidak perlu ada.

- Disable: `net user guest /active:no`

- Verifikasi: `net user guest` → pastikan "Account active" = **No**.

  

**Administrator Account**

  

> Jangan langsung disable akun Administrator.

  

Akun ini sering jadi satu-satunya jalan masuk kalau akun lain bermasalah. Mematikannya tanpa persiapan = risiko lockout total.

  

Urutan aman:

1. Cek administrator lain — pastikan ada akun admin lain yang aktif dan bisa login.

2. Buat akun cadangan jika perlu — kalau tidak ada admin lain, buat satu akun admin baru dengan password kuat.

3. Test login dengan akun cadangan tersebut — SEBELUM mengubah akun Administrator asli.

4. Baru disable atau rename akun Administrator bawaan.

  

**Risiko lockout:** melewati salah satu dari 4 langkah di atas adalah penyebab paling umum peserta "terkunci" dari sistem yang sedang mereka kerjakan sendiri.

  

---

  

## 8. Security Options

  

Diakses lewat `secpol.msc → Local Policies → Security Options`.

  

**Prioritas utama (wajib):**

  

| Setting | Fungsi |

|---|---|

| Accounts: Rename administrator account | Mempersulit attacker menebak nama akun admin |

| Accounts: Guest account status | Memastikan Guest nonaktif |

| Interactive logon: Don't display last signed-in user name | Mencegah attacker tahu username valid dari layar login |

| Network security: LAN Manager authentication level | Set ke "Send NTLMv2 response only. Refuse LM & NTLM" — menghindari protokol lama yang lemah |

| User Account Control (UAC) settings | Memastikan proses privilege tinggi tetap butuh konfirmasi |

  

**Mana yang wajib vs situasional:**

- **Wajib** (hampir selalu diterapkan): Guest disable, NTLMv2, UAC aktif.

- **Situasional** (tergantung skenario lomba): Rename Administrator (cek instruksi soal — kadang akun tertentu harus tetap bernama Administrator untuk keperluan penilaian), Hide Last Username.

  

---

  

## 9. User Rights Assignment

  

**Apa itu User Rights Assignment?**

Bagian Local Security Policy yang mengatur hak sistem tingkat rendah (bukan izin ke file/folder tertentu) — misalnya siapa yang boleh login lokal, siapa yang boleh shutdown remote, siapa yang boleh take ownership file.

  

**Bedanya privilege dan permission:**

- **Privilege (User Right):** hak melekat pada akun/grup untuk aksi tingkat sistem (mis. *Log on locally*, *Shut down the system*).

- **Permission:** hak akses ke objek spesifik (file, folder, registry key) — mis. Read, Write, Full Control.

  

> **PERINGATAN:** Jangan menghapus hak akses secara sembarangan.

  

Menghapus satu privilege yang salah (mis. "Log on locally" dari grup yang sedang Anda pakai) bisa membuat Anda sendiri tidak bisa login.

  

- **Risiko:** lockout, service berhenti berfungsi, akun tidak bisa dipakai untuk tugas yang seharusnya bisa.

- **Verifikasi:** logout-login ulang dengan akun yang haknya baru diubah — jangan hanya percaya tampilan GUI.

- **Rollback:** catat kondisi awal sebelum mengubah (lihat Decision Flow #3), sehingga privilege yang terlanjur dihapus bisa ditambahkan kembali lewat secpol.msc.

  

---

  

## 10. Threat Mapping

  

| Setting | Risiko | Perlindungan |

|---|---|---|

| Password Policy | Brute Force | Password kuat & kompleks |

| Account Lockout | Password Attack (percobaan berulang) | Account lockout aktif |

| Guest Disable | Anonymous Access | Disable Guest |

| NTLMv2 | Credential Theft / Pass-the-Hash | Strong Authentication (NTLMv2) |

  

---

  

## 11. Implementasi GUI

  

Buka `secpol.msc` (Run → ketik `secpol.msc` → Enter).

  

**Path yang perlu diketahui:**

1. **Account Policies** — `secpol.msc → Account Policies → Password Policy / Account Lockout Policy`

2. **Security Options** — `secpol.msc → Local Policies → Security Options`

3. **User Rights Assignment** — `secpol.msc → Local Policies → User Rights Assignment`

  

**Langkah umum mengubah satu setting:**

1. Buka `secpol.msc`.

2. Navigasi ke path yang sesuai.

3. Double-click setting yang ingin diubah.

4. Ubah nilai/opsi.

5. Klik **Apply → OK**.

6. Tutup dan buka ulang secpol.msc untuk konfirmasi perubahan tersimpan.

  

---

  

## 12. Command dan PowerShell

  

**`net user`**

Fungsi: menampilkan daftar semua akun lokal. | Kapan: initial assessment. | Output: daftar username. | Risiko: read-only, tidak ada.

  

**`net user [username]`**

Fungsi: menampilkan detail satu akun (status aktif, kapan password terakhir diganti). | Kapan: cek kondisi akun sebelum diubah. | Output: detail status akun. | Risiko: read-only, tidak ada.

  

**`net user guest /active:no`**

Fungsi: menonaktifkan akun Guest. | Kapan: setelah memastikan tidak ada aplikasi butuh Guest aktif. | Output: "The command completed successfully." | Risiko: rendah — tetap verifikasi setelahnya.

  

**`net localgroup administrators`**

Fungsi: menampilkan anggota grup Administrators. | Kapan: audit privilege, sebelum & sesudah perubahan akun admin. | Output: daftar member. | Risiko: read-only, tidak ada.

  

**`net accounts`**

Fungsi: menampilkan/mengatur password & lockout policy lewat CLI. | Kapan: assessment awal & konfigurasi lewat CLI. | Output: ringkasan policy (tanpa parameter) atau konfirmasi perubahan (dengan parameter). | Risiko: perubahan langsung berlaku — pastikan tahu efeknya sebelum Enter.

  

**`whoami /priv`**

Fungsi: menampilkan privilege akun aktif. | Kapan: verifikasi setelah perubahan User Rights Assignment. | Output: daftar privilege & status. | Risiko: read-only, tidak ada.

  

**`Get-LocalUser`** (PowerShell)

Fungsi: versi PowerShell dari `net user`, output lebih terstruktur. | Kapan: alternatif audit user. | Output: tabel Name, Enabled, Description. | Risiko: read-only, tidak ada.

  

---

  

## 13. Verification Checklist

  

**ACCOUNT:**

- [ ] Guest aman (disabled)

- [ ] Administrator aman (password kuat / sesuai instruksi soal)

- [ ] Tidak ada admin asing di `net localgroup administrators`

  

**PASSWORD:**

- [ ] Complexity aktif

- [ ] Minimum length sesuai standar

  

**LOCKOUT:**

- [ ] Threshold sesuai (tidak 0, tidak terlalu ketat)

  

**SECURITY:**

- [ ] UAC aktif

- [ ] NTLM authentication level sesuai (NTLMv2)

  

---

  

## 14. Common Mistakes Peserta

  

1. Disable akun Administrator tanpa akun cadangan yang sudah teruji.

2. Mengubah firewall/policy tanpa cek dulu akses yang sedang dipakai (mis. remote session sendiri terputus).

3. Memberikan privilege lebih tinggi dari yang dibutuhkan pada akun/servis.

4. Tidak verifikasi setelah perubahan — asumsi "sudah klik Apply berarti sudah aktif".

5. Set lockout threshold terlalu rendah (2–3 kali) sehingga mengunci akun sendiri.

6. Lupa mencatat kondisi awal (baseline) sehingga tidak bisa rollback.

7. Menghapus privilege "Log on locally" dari akun/grup yang sedang aktif dipakai.

8. Mengaktifkan Reversible Encryption karena mengira ini bagian dari hardening.

9. Hanya mengubah lewat GUI tanpa verifikasi lewat command (atau sebaliknya).

10. Tidak membaca instruksi soal dengan teliti sebelum rename/disable akun tertentu.

11. Mengerjakan banyak perubahan sekaligus tanpa urutan, sehingga sulit troubleshooting.

12. Lupa logout-login ulang untuk menguji efek nyata perubahan.

  

---

  

## 15. Troubleshooting

  

**Masalah: Tidak bisa login setelah perubahan konfigurasi.**

Kemungkinan penyebab: account lockout aktif, password policy baru membuat password lama tidak valid, akun ter-disable saat proses hardening.

Solusi:

1. Login dengan akun cadangan.

2. Cek status akun: `net user [username]`.

3. Jika lockout: tunggu durasi selesai, atau reset lewat akun cadangan.

4. Jika disabled: `net user [username] /active:yes` lewat akun cadangan.

5. Jika password bermasalah: reset lewat akun cadangan, catat perubahan yang dilakukan.

  

**Masalah: Perubahan di secpol.msc terlihat "tidak tersimpan".**

Kemungkinan penyebab: ada Group Policy (jika sistem bagian domain) yang menimpa Local Security Policy, atau lupa klik Apply.

Solusi:

1. Tutup dan buka ulang secpol.msc, cek ulang settingnya.

2. Jalankan `gpresult /r` untuk melihat apakah ada GPO yang menimpa setting lokal.

  

---

  

## 16. W1 MEMORY GUIDE (WAJIB)

  

*Untuk hafalan cepat saat lomba — tidak perlu membaca seluruh modul lagi.*

  

### A. PAM Golden Rule

**"CEK → AMANKAN → UBAH → CEK LAGI"**

  

### B. 5 Command Wajib Hafal

```

net user

net localgroup administrators

net accounts

whoami /priv

secpol.msc

```

  

### C. Password Memory

**12 - 5 - 90**

12 = minimum password length | 5 = password history | 90 = maximum password age (hari)

  

### D. Account Rule

**"Jangan matikan admin sebelum punya kunci cadangan."**

  

### E. Security Options Priority (Top 5)

1. Guest account — disable

2. NTLM authentication level — NTLMv2

3. UAC — aktif

4. Rename administrator account

5. Don't display last signed-in user name

  

### F. 1 Menit Checklist Sebelum Submit

- [ ] User aman

- [ ] Admin aman (+ cadangan sudah teruji)

- [ ] Password policy aman

- [ ] Lockout policy aman (tidak 0, tidak terlalu ketat)

- [ ] Security options sudah dicek

- [ ] Verifikasi selesai (login ulang tanpa masalah)

  

### G. Kesalahan Fatal Yang Harus Diingat

1. Disable admin tanpa cadangan.

2. Lockout threshold terlalu rendah — mengunci diri sendiri.

3. Tidak verifikasi setelah perubahan.

4. Menghapus privilege login sendiri.