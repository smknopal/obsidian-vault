# WGUI-0: Windows Server Navigation & Anti-Lockout Foundation

  

### Competition Ready Final

  

---

  

## Posisi Modul

  

WGUI-0 adalah fondasi paling awal sebelum peserta mempelajari hardening Windows Server secara teknis. Urutan pembelajaran yang benar adalah:

  

```text

WGUI-0 (Navigation & Anti-Lockout Foundation)

    ↓

Windows PAM / Local Security Policy

    ↓

Group Policy Hardening

    ↓

Active Directory Hardening

    ↓

Network Service Security

    ↓

Defender dan Logging

```

  

WGUI-0 **tidak** membahas hardening mendalam. Modul ini hanya membentuk lima kemampuan dasar yang wajib dikuasai peserta sebelum menyentuh konfigurasi keamanan apa pun:

  

1. Mengenali jenis dan kondisi Windows Server.

2. Menemukan lokasi konfigurasi penting melalui GUI.

3. Menghindari account lockout dan connection lockout.

4. Menyiapkan recovery yang sah dan sesuai aturan lomba.

5. Melakukan perubahan secara bertahap, memverifikasi hasil, menguji koneksi baru, dan menyimpan evidence yang diperlukan.

  

Peserta yang langsung melakukan hardening tanpa fondasi ini berisiko besar mengalami lockout, kehilangan akses RDP, atau melanggar rules lomba.

  

**Catatan penting: modul ini wajib dihafal, bukan dibaca ulang saat hari-H.** Hari hardening LKSN Cybersecurity 2026 bersifat closed book total: dilarang googling, dilarang menggunakan AI dalam bentuk apa pun, dilarang membuka cheatsheet baik fisik maupun digital, dan dilarang menggunakan script atau tools otomatis apa pun — seluruh konfigurasi wajib dilakukan manual. Pelanggaran terhadap aturan ini berakibat diskualifikasi langsung, bukan pengurangan poin bertahap. Karena peserta tidak bisa membuka modul ini, mencari referensi, atau bertanya ke AI saat hari-H, seluruh pola "checklist hafalan" yang berulang di sepanjang dokumen ini bukan sekadar gaya penulisan — checklist tersebut adalah latihan agar alur berpikir sudah otomatis di kepala peserta jauh sebelum waktu pengerjaan dimulai.

  

### Alur Mental Utama

  

Seluruh isi modul berputar pada satu alur berpikir yang harus dihafal:

  

```text

IDENTIFY

    ↓

BASELINE

    ↓

RECOVERY

    ↓

CHECK RISK

    ↓

CHANGE

    ↓

VERIFY

    ↓

RECONNECT

    ↓

EVIDENCE

```

  

| Langkah    | Pertanyaan utama                                    |

| ---------- | ---------------------------------------------------- |

| IDENTIFY   | Server apa ini dan apa objective-nya?                |

| BASELINE   | Apa kondisi awalnya?                                 |

| RECOVERY   | Apa jalan kembali yang sah dan sudah teruji?         |

| CHECK RISK | Perubahan ini menyentuh akun, koneksi, atau policy?  |

| CHANGE     | Apa satu perubahan yang akan dilakukan?              |

| VERIFY     | Apakah konfigurasi benar-benar berlaku?              |

| RECONNECT  | Apakah koneksi atau login baru masih berhasil?       |

| EVIDENCE   | Bukti apa yang perlu disimpan?                       |

  

Alur ini digunakan untuk setiap perubahan penting. Screenshot tidak harus diambil untuk setiap klik kecil — lihat WGUI-0.9 untuk aturan lengkapnya.

  

---

  

## WGUI-0.1 — Prinsip Dasar Sebelum Hardening

  

**Konsep**

  

Sebelum menyentuh satu pun konfigurasi keamanan, peserta wajib memahami bahwa hardening yang dilakukan tanpa persiapan justru menjadi sumber kegagalan paling umum di kompetisi: lockout akun sendiri, RDP terputus, atau perubahan yang melanggar objective.

  

**Kenapa penting**

  

Poin dalam lomba tidak hanya diberikan untuk konfigurasi yang benar, tetapi juga hilang apabila peserta kehilangan akses ke server yang sedang dikerjakan. Prinsip dasar ini mencegah kesalahan fatal di awal waktu pengerjaan.

  

**Lokasi GUI**

  

Tidak ada lokasi GUI khusus pada tahap ini — ini adalah tahap membaca dan berpikir sebelum membuka satu pun snap-in.

  

**Risiko lomba**

  

* Langsung mengubah firewall atau policy tanpa mengenali server.

* Membuat akun administratif baru sebagai kebiasaan otomatis.

* Melompat ke perubahan tanpa membaca objective dan rules.

* Menggunakan script atau automation untuk mempercepat konfigurasi — dilarang total di hari hardening, seluruh konfigurasi wajib dilakukan manual; pelanggaran berakibat diskualifikasi langsung.

  

**Cara aman**

  

1. Baca objective dan rules lomba secara menyeluruh sebelum membuka snap-in apa pun.

2. Kenali dahulu jenis server (lihat WGUI-0.5).

3. Susun baseline (lihat WGUI-0.6) sebelum melakukan perubahan pertama.

4. Validasi recovery yang sah (lihat WGUI-0.2) sebelum menyentuh akun, koneksi, atau policy.

5. Gunakan alur IDENTIFY → BASELINE → RECOVERY → CHECK RISK → CHANGE → VERIFY → RECONNECT → EVIDENCE untuk setiap perubahan penting.

  

**Checklist hafalan**

  

```text

[ ] Objective dan rules sudah dibaca

[ ] Jenis server sudah dikenali

[ ] Baseline belum dan akan disusun

[ ] Recovery belum divalidasi

[ ] Belum melakukan perubahan apa pun

[ ] Tidak menggunakan script/automation apa pun — semua konfigurasi manual

```

  

> “Jangan hardening sebelum mengenali sistem.”

  

---

  

## WGUI-0.2 — Recovery yang Sah: Account Recovery dan Access Channel Recovery

  

**Konsep**

  

Recovery bukan berarti selalu membuat akun administratif baru. Recovery berarti memastikan ada jalan kembali yang sah apabila sebuah perubahan menyebabkan kegagalan akses. Ada tiga lapisan recovery yang harus dipahami secara terpisah.

  

### A. Account Recovery

  

Melindungi ketika masalah berada pada identitas atau hak akun, misalnya:

  

* akun utama disabled;

* akun terkunci (locked);

* password bermasalah;

* administrative group membership hilang;

* akun utama terkena pembatasan login tertentu.

  

Account recovery hanya berguna apabila akun recovery:

  

* sah (authorized);

* aktif;

* password-nya diketahui;

* memiliki hak yang sesuai;

* tidak ikut terkena deny policy;

* dapat melakukan login baru.

  

### B. Access Channel Recovery

  

Diperlukan ketika masalah berada pada **jalur akses**, bukan pada identitas akun, misalnya:

  

* firewall memblokir RDP;

* network adapter salah konfigurasi;

* IP, gateway, atau DNS berubah salah;

* Remote Desktop Services berhenti;

* RDP dinonaktifkan;

* NLA atau jalur autentikasi bermasalah.

  

Dua akun berbeda yang sama-sama memakai RDP tetap menggunakan jalur network, firewall, dan service yang sama.

  

> **Dua akun RDP bukan dua jalur akses yang berbeda.**

  

**Realita akses di kompetisi ini**: peserta **tidak** diberikan akses console, hypervisor, maupun KVM. Satu-satunya jalur akses adalah VPN WireGuard menuju SSH dan/atau RDP yang sudah disiapkan panitia. Karena itu, access channel recovery yang sah dan realistis pada kompetisi ini **bukan** console/hypervisor, melainkan **melapor ke panitia/juri untuk me-reset mesin** apabila seluruh jalur RDP/SSH benar-benar hilang.

  

**Prosedur lapor-reset (access channel recovery utama di kompetisi ini)**

  

1. Pastikan seluruh jalur RDP dan SSH benar-benar tidak dapat diakses — bukan sekadar satu session yang terputus.

2. Informasikan ke juri/teknisi: asal sekolah dan nama tim (IP tim bersifat opsional).

3. Panitia akan me-reset mesin, dengan durasi sekitar 5–8 menit.

4. Reset menghapus **seluruh progres pengerjaan sebelumnya** pada mesin tersebut.

5. **Tidak ada penambahan waktu pengerjaan** akibat reset ini.

6. Jika beberapa tim reset bersamaan, berlaku sistem antrian (queue) — waktu tunggu nyata bisa lebih lama dari 5–8 menit.

  

> Karena konsekuensi reset berat — progres hilang total, waktu tidak bertambah, dan bisa mengantre — **pencegahan jauh lebih penting daripada mengandalkan recovery ini**. Satu perubahan connection risk dalam satu waktu dan reconnect test sebelum melanjutkan (lihat WGUI-0.4 dan WGUI-0.9) adalah pertahanan utama, bukan reset oleh panitia.

  

### C. AD DS Emergency Recovery (khusus Domain Controller)

  

* Domain administrative account digunakan untuk pemulihan operasional normal.

* Console atau hypervisor digunakan untuk pemulihan jalur akses.

* Directory Services Restore Mode (DSRM) digunakan khusus untuk kondisi darurat AD DS.

  

> DSRM bukan akun domain dan bukan akun untuk login RDP harian. DSRM digunakan melalui console dalam mode pemulihan Directory Services.

  

> “DC normal memakai akun domain. DC darurat memakai console dan DSRM.”

  

Prosedur repair database Active Directory secara mendalam berada di luar lingkup WGUI-0.

  

> **Catatan konteks kompetisi ini**: karena peserta tidak memiliki akses console/hypervisor (lihat Bagian B), materi DSRM di atas bersifat pengetahuan konsep umum Windows Server, bukan prosedur yang bisa benar-benar dijalankan mandiri di kompetisi ini. Jika kondisi darurat AD DS benar-benar terjadi pada hari-H, jalur realistis yang tersedia tetap prosedur lapor-reset ke panitia/juri pada Bagian B, bukan masuk DSRM sendiri.

  

**Kenapa penting**

  

Kesalahan paling umum peserta pemula adalah menyamakan “punya recovery” dengan “sudah membuat akun admin baru”. Kesalahan ini justru dapat melanggar objective, menambah attack surface, atau dianggap unauthorized account oleh juri.

  

**Lokasi GUI**

  

* Computer Management / `lusrmgr.msc` — untuk memeriksa local account pada standalone atau domain member.

* Active Directory Users and Computers / `dsa.msc` — untuk memeriksa domain account pada Domain Controller.

* Local Security Policy / `secpol.msc` — untuk memeriksa hak logon.

* Tidak ada console VM/hypervisor bagi peserta di kompetisi ini — access channel recovery dilakukan lewat prosedur lapor-reset ke panitia/juri (lihat Bagian B).

  

**Risiko lomba**

  

* Membuat akun administratif baru tanpa izin objective/rules.

* Menganggap akun RDP kedua sebagai jalur akses cadangan.

* Menyamakan domain admin account dengan DSRM.

* Mengasumsikan ada console/hypervisor pribadi yang bisa diakses kapan saja, padahal tidak tersedia di kompetisi ini.

  

**Cara aman — urutan recovery yang benar**

  

1. Baca objective dan rules.

2. Periksa daftar authorized users.

3. Identifikasi akun administratif sah yang sudah tersedia.

4. Validasi status akun, password, group membership, hak login, dan metode akses.

5. Gunakan akun resmi tersebut sebagai account recovery apabila sesuai.

6. Buat akun baru **hanya** apabila:

   * diminta objective;

   * diperbolehkan rules;

   * termasuk tindakan recovery yang sah;

   * tidak melanggar daftar authorized users.

7. Jika pembuatan akun tidak diizinkan, jangan membuat akun baru hanya demi kenyamanan.

  

Gunakan istilah **authorized recovery account**, **akun administratif yang diizinkan**, atau **validasi/siapkan recovery** — bukan selalu “buat recovery account”.

  

**Checklist hafalan**

  

```text

[ ] Authorized recovery account sudah diidentifikasi, bukan dibuat sembarangan

[ ] Access channel recovery sudah dikenali: tidak ada console/hypervisor bagi peserta; jalur utama adalah lapor-reset ke panitia/juri

[ ] Konsekuensi reset (progres hilang, 5–8 menit, tanpa tambahan waktu, ada antrian) sudah dipahami

[ ] DC: DSRM dipahami sebagai emergency-only, bukan akun harian, dan tidak realistis dijalankan mandiri tanpa console

[ ] Dua akun RDP tidak dianggap dua jalur akses

```

  

> “Validasi akun resmi lebih dahulu. Membuat akun baru bukan tindakan default.”

  

---

  

## WGUI-0.3 — Klasifikasi Risiko Anti-Lockout

  

**Konsep**

  

Setiap perubahan konfigurasi Windows Server dapat dikelompokkan ke dalam tiga kategori risiko utama: **Account Risk**, **Connection Risk**, dan **Policy Risk**. Mengenali kategori risiko sebelum melakukan perubahan membantu peserta memilih mitigasi yang tepat.

  

### Account Risk

  

Contoh: disable account, reset password, account lockout, menghapus administrative group membership, mengubah keanggotaan Remote Desktop Users, mengubah Allow/Deny logon rights.

  

Mitigasi: authorized recovery account, cek effective policy, login baru, pertahankan session atau console yang sedang aktif.

  

### Connection Risk

  

Contoh: firewall, RDP setting, Remote Desktop Services, network adapter, IP, gateway, DNS, NLA, perubahan jalur remote management.

  

Mitigasi: console atau hypervisor, perubahan satu per satu, pertahankan jalur lama, uji koneksi baru sebelum melanjutkan.

  

> **Catatan status kompetisi ini**: status SSH pada sisi Windows masih **tentatif** ("direncanakan wajib menyala", belum final) — wajib dikonfirmasi ulang oleh peserta mendekati hari-H. Jika saat hari-H dipastikan SSH Windows wajib menyala, maka service SSH tersebut wajib diperlakukan sama seperti RDP: dilarang dimatikan atau diubah konfigurasinya tanpa reconnect test terlebih dahulu (lihat WGUI-0.4).

  

### Policy Risk

  

Contoh: Local Security Policy, Local Group Policy, Domain GPO, deny policy, policy precedence, setting lokal yang di-override oleh domain.

  

Mitigasi: periksa `rsop.msc`, gunakan Group Policy Results jika tersedia, bedakan configured policy dan effective policy, verifikasi setelah refresh atau login ulang.

  

**Kenapa penting**

  

Ketiga risiko ini adalah tiga pintu berbeda yang dapat mengunci akses peserta terhadap server yang sedang dikerjakan. Salah menilai kategori risiko berarti salah memilih mitigasi.

  

**Lokasi GUI**

  

* `secpol.msc` — Local Security Policy.

* `rsop.msc` — Resultant Set of Policy (effective policy).

* `gpmc.msc` — Group Policy Management (jika relevan dan tersedia).

* `wf.msc` — Windows Defender Firewall with Advanced Security.

* `services.msc` — status Remote Desktop Services.

* Computer Management / `lusrmgr.msc` atau `dsa.msc` — administrative group membership.

  

**Risiko lomba**

  

* Mengubah account, connection, dan policy secara bersamaan dalam satu langkah.

* Tidak menyadari bahwa configured policy berbeda dari effective policy.

  

**Cara aman**

  

1. Sebelum tiap perubahan, tanyakan: ini menyentuh akun, koneksi, atau policy?

2. Tangani satu kategori risiko dalam satu waktu.

3. Terapkan mitigasi sesuai kategori.

4. Verifikasi hasil sebelum melanjutkan ke perubahan berikutnya.

  

**Checklist hafalan**

  

```text

[ ] Kategori risiko dari perubahan ini sudah diidentifikasi

[ ] Mitigasi yang sesuai kategori sudah disiapkan

[ ] Tidak mengubah lebih dari satu kategori risiko sekaligus

```

  

> “Akun, koneksi, policy — tiga pintu yang dapat mengunci akses.”

  

---

  

## WGUI-0.4 — Waktu Efek Perubahan dan Reconnect Test

  

**Konsep**

  

Tidak semua perubahan konfigurasi berlaku pada saat yang sama. Sebagian berlaku seketika, sebagian pada login berikutnya, dan sebagian lagi memerlukan refresh policy atau restart service. Peserta wajib memahami kapan sebuah perubahan benar-benar aktif sebelum menganggap pekerjaan selesai.

  

**Kenapa penting**

  

Salah memperkirakan waktu efek dapat membuat peserta menganggap suatu setting sudah berlaku, padahal belum diuji dengan benar — atau sebaliknya, membuat peserta panik karena mengira perubahan gagal padahal hanya belum waktunya berlaku.

  

**Lokasi GUI**

  

* `rsop.msc` — memeriksa effective policy.

* `gpmc.msc` — Group Policy Results (jika tersedia).

* `services.msc` — restart service terkait.

* Sign-out/login ulang — untuk memperbarui token akses.

  

**Risiko lomba**

  

Tabel waktu efek perubahan (bersifat umum, bergantung pada konfigurasi environment):

  

| Jenis perubahan                                                      | Waktu atau bentuk efek umum                                                     |

| --------------------------------------------------------------------- | -------------------------------------------------------------------------------- |

| Firewall, network adapter, RDP setting, atau Remote Desktop Services  | Dapat memutus sesi aktif atau membuat koneksi baru gagal                        |

| Password length dan complexity                                       | Diperiksa ketika password berikutnya dibuat atau diubah                        |

| Account lockout threshold                                            | Berlaku pada percobaan login gagal berikutnya                                  |

| Account lockout duration                                             | Berlaku setelah akun mencapai kondisi locked                                    |

| Maximum password age                                                 | Berkaitan dengan usia dan masa berlaku password                                |

| User Rights Assignment                                                | Umumnya terlihat pada login atau session baru                                  |

| Administrative group membership                                      | Dapat memerlukan sign-out dan login baru agar token akses diperbarui           |

| Local atau domain GPO                                                 | Berlaku setelah policy refresh; sebagian setting memerlukan logoff atau restart |

| Service configuration                                                 | Dapat berlaku langsung, setelah service restart, atau setelah boot berikutnya  |

  

**Cara aman — Reconnect Test**

  

> Session yang masih hidup belum membuktikan koneksi baru masih dapat dibuat.

  

Windows Firewall atau perubahan policy tertentu dapat membuat session lama tetap aktif, tetapi login atau koneksi baru gagal.

  

Prosedur aman:

  

1. Pertahankan Session A tetap terbuka.

2. Lakukan satu perubahan melalui Session A.

3. Gunakan Session B atau koneksi baru untuk melakukan login ulang.

4. Pastikan koneksi baru benar-benar berhasil.

5. Pastikan akun dan desktop dapat digunakan.

6. Baru lanjut ke perubahan berikutnya.

7. Jangan menutup jalur lama sebelum koneksi baru berhasil.

  

**Checklist hafalan**

  

```text

[ ] Waktu efek perubahan sudah diperkirakan dengan benar

[ ] Session lama (Session A) belum ditutup

[ ] Reconnect test menggunakan Session B/koneksi baru sudah dilakukan

[ ] Login baru dan desktop terbukti berfungsi

```

  

> “Session masih hidup belum berarti reconnect berhasil.”

  

---

  

## WGUI-0.5 — Strategi Recovery Berdasarkan Jenis Server

  

**Konsep**

  

Strategi recovery berbeda tergantung jenis server yang dihadapi: Standalone Server, Domain Member, atau Domain Controller.

  

| Jenis server              | Pengelolaan akun                       | Recovery operasional                                          |

| -------------------------- | ---------------------------------------- | ----------------------------------------------------------------- |

| Standalone Server          | Local users dan local groups            | Authorized local administrative account                          |

| Domain Member              | Local accounts serta domain accounts    | Authorized local admin atau domain account sesuai objective       |

| Domain Controller          | Active Directory users dan groups       | Authorized domain administrative account                         |

| Domain Controller darurat  | DSRM melalui console                     | Pemulihan darurat AD DS                                           |

  

Peserta **tidak diwajibkan** selalu membuat akun-akun tersebut. Yang wajib dilakukan adalah **identifikasi, validasi, atau siapkan** akun recovery yang diizinkan — lihat WGUI-0.2.

  

**Kenapa penting**

  

Setiap jenis server memiliki model pengelolaan akun yang berbeda. Menyamaratakan strategi recovery antar jenis server adalah kesalahan umum, terutama pada Domain Controller.

  

**Lokasi GUI**

  

* Standalone/Domain Member: Computer Management → Local Users and Groups, atau `lusrmgr.msc`.

* Domain Controller: Active Directory Users and Computers, atau `dsa.msc`.

* DC darurat: console atau hypervisor, masuk ke mode DSRM.

  

**Risiko lomba**

  

* Mencari `lusrmgr.msc` seperti biasa pada Domain Controller (lihat WGUI-0.6 untuk penjelasan bercabang).

* Menyarankan pembuatan Domain Administrator baru sebagai tindakan default.

  

**Cara aman**

  

1. Tentukan dahulu jenis server (lihat baseline pada WGUI-0.6).

2. Sesuaikan lokasi pengecekan akun dengan tabel di atas.

3. Validasi akun resmi yang tersedia sebagai recovery, bukan membuat akun baru sebagai kebiasaan.

4. Untuk Domain Controller, kenali dua jalur: operasional normal (domain admin account) dan darurat (console + DSRM).

  

**Checklist hafalan**

  

```text

[ ] Jenis server sudah dipastikan sebelum menentukan strategi recovery

[ ] Lokasi pengelolaan akun sudah sesuai jenis server

[ ] Tidak membuat akun admin baru sebagai default

```

  

> “Standalone → local account. Domain Member → local/domain account. Domain Controller → domain account. DC darurat → console + DSRM.”

  

---

  

## WGUI-0.6 — Baseline dan GUI Location Map

  

**Konsep**

  

Baseline adalah kondisi awal server yang wajib dicatat sebelum perubahan pertama dilakukan. GUI Location Map adalah peta lokasi konfigurasi penting yang bercabang sesuai jenis server.

  

**Kenapa penting**

  

Tanpa baseline, peserta tidak memiliki titik pembanding untuk mendeteksi efek samping perubahan. Tanpa GUI Location Map yang benar, peserta dapat salah mencari tool pada jenis server yang tidak sesuai — misalnya mencari `lusrmgr.msc` pada Domain Controller.

  

**Minimum baseline yang wajib dicatat**

  

1. Server identity

2. OS edition atau version

3. Hostname

4. Domain atau workgroup

5. Jenis server: standalone / domain member / Domain Controller

6. Network: adapter, IP, subnet, gateway, DNS

7. Authorized administrative accounts

8. Administrative group membership

9. RDP status

10. Remote Desktop Services status

11. Active firewall profile

12. RDP firewall rule

13. Local policy

14. Effective policy

15. Jalur console atau access channel recovery

16. Hasil login baru menggunakan recovery yang diizinkan, jika rules memungkinkan

  

**Lokasi GUI**

  

* Server Manager → Local Server (identity, OS, domain/workgroup)

* System Properties (hostname, domain/workgroup)

* Network Connections / Adapter Properties (IP, gateway, DNS)

* Computer Management

* Local Users and Groups (`lusrmgr.msc`) — untuk standalone/domain member

* Active Directory Users and Computers (`dsa.msc`) — untuk Domain Controller

* Windows Defender Firewall with Advanced Security (`wf.msc`)

* Services (`services.msc`)

* Remote Desktop settings (System Properties → Remote)

* Local Security Policy (`secpol.msc`)

* Resultant Set of Policy (`rsop.msc`)

* Group Policy Management (`gpmc.msc`) — jika relevan dan tersedia

  

**GUI bercabang: Standalone/Domain Member vs Domain Controller**

  

| Informasi                  | Standalone/Domain Member              | Domain Controller                       |

| ---------------------------- | ---------------------------------------- | ------------------------------------------- |

| Local users dan groups      | Computer Management / `lusrmgr.msc`     | Tidak berlaku seperti server biasa         |

| Domain users dan groups     | ADUC jika tersedia dan berwenang        | `dsa.msc`                                  |

| Administrative membership   | Local Administrators                    | Domain/Builtin administrative groups       |

| Effective policy            | `rsop.msc`                              | `rsop.msc` atau Group Policy Results       |

| Recovery darurat             | Console dan akun lokal sah              | Console dan DSRM                           |

  

> “Tidak ada Local Users and Groups seperti biasa di DC.”

  

`lusrmgr.msc` relevan untuk standalone server dan domain member. Domain Controller tidak menggunakan model local users dan local groups seperti member server; pengelolaan akun dan grup domain dilakukan melalui `dsa.msc`. Grup administratif bawaan domain diperiksa melalui container atau OU yang sesuai di Active Directory. Jika `lusrmgr.msc` tidak tersedia atau tidak dapat digunakan sebagaimana biasanya, peserta harus memeriksa apakah server tersebut merupakan Domain Controller — jangan langsung menganggap tool rusak.

  

**Administrative group membership, bukan “current privilege”**

  

Gunakan istilah **administrative group membership**, bukan “current user privilege”. Menjadi anggota Administrators tidak selalu berarti setiap aplikasi sedang berjalan elevated; UAC masih dapat membatasi access token. Pengecekan group membership berbeda dari menjalankan program dengan elevation. Pembahasan UAC teknis mendalam berada di luar lingkup modul ini.

  

**Risiko lomba**

  

* Tidak mencatat baseline sebelum perubahan pertama.

* Menggunakan GUI Location Map yang sama untuk semua jenis server.

* Menyamakan administrative group membership dengan elevation proses saat ini.

  

**Cara aman**

  

1. Tentukan jenis server terlebih dahulu.

2. Catat baseline minimum sesuai daftar di atas, sesuai cabang GUI yang relevan.

3. Simpan evidence baseline (lihat WGUI-0.9 untuk aturan evidence).

4. Jangan membuka seluruh tool jika tidak relevan dengan jenis server yang dihadapi.

  

**Checklist hafalan**

  

```text

[ ] Baseline 16 poin sudah dicatat

[ ] GUI Location Map yang dipakai sudah sesuai jenis server

[ ] Administrative group membership sudah diperiksa, bukan disamakan dengan elevation

```

  

---

  

## WGUI-0.7 — Emergency Rollback saat Akses RDP Hilang

  

**Konsep**

  

Bagian ini menjelaskan urutan berpikir yang tenang dan sistematis ketika akses RDP tiba-tiba gagal atau hilang di tengah pengerjaan.

  

**Kenapa penting**

  

Kepanikan saat kehilangan akses sering membuat peserta mengambil tindakan acak yang justru memperburuk situasi. Urutan berpikir yang jelas mempercepat pemulihan tanpa memperbesar risiko. Di kompetisi ini, konsekuensi kehilangan akses juga tidak ringan: satu-satunya jalan kembali saat semua akses benar-benar hilang adalah reset oleh panitia, yang menghapus seluruh progres dan tidak menambah waktu pengerjaan. Karena itu, urutan berpikir di bawah ini sengaja menempatkan pencegahan dan verifikasi bertahap di atas segalanya, dan menjadikan lapor-reset sebagai langkah terakhir, bukan langkah pertama.

  

**Lokasi GUI**

  

* Session lama yang masih terbuka (jika ada).

* Tidak ada console VM/hypervisor/KVM bagi peserta di kompetisi ini — jalur alternatif satu-satunya saat semua akses hilang adalah melapor ke panitia/juri untuk reset (lihat prosedur lapor-reset pada WGUI-0.2).

* `wf.msc`, `services.msc`, Network Connections untuk memeriksa firewall, service, dan network — hanya dapat dilakukan selama masih ada session lama yang bisa dipakai.

  

**Risiko lomba**

  

* Langsung mengasumsikan password salah sebagai penyebab.

* Menutup session lama sebelum jalur baru terbukti berhasil.

* Melakukan rollback lebih dari satu area sekaligus sehingga sulit dilacak.

* Menunggu terlalu lama sebelum melapor ke panitia saat akses benar-benar sudah hilang, sehingga membuang waktu pengerjaan yang hanya 3 jam.

* Mengira ada jalur console/hypervisor pribadi yang bisa dipakai, padahal tidak tersedia di kompetisi ini.

  

**Cara aman — urutan berpikir**

  

1. Jangan langsung mengasumsikan password salah.

2. Tentukan lapisan masalah: account, policy, service, firewall, atau network.

3. Jika session lama masih ada, jangan ditutup.

4. Jika koneksi baru gagal tetapi session lama masih ada, gunakan session lama untuk rollback perubahan terakhir.

5. Jika seluruh RDP (dan SSH, apabila berlaku di Windows) benar-benar mati dan tidak ada session lama yang bisa dipakai, segera laporkan ke panitia/juri untuk reset — jangan mencari-cari console atau hypervisor karena tidak tersedia bagi peserta di kompetisi ini.

6. Sebelum melapor, ingat atau catat perubahan terakhir yang dilakukan agar tidak diulang persis sama setelah mesin di-reset.

7. Sampaikan ke panitia/juri: asal sekolah dan nama tim (IP tim opsional), lalu tunggu proses reset (5–8 menit, bisa lebih lama jika ada antrian).

8. Setelah mesin di-reset, lanjutkan kembali dengan lebih hati-hati: satu perubahan dalam satu waktu, dan uji koneksi baru (reconnect test) setelah tiap perubahan sebelum melanjutkan.

9. Simpan evidence incident apabila relevan, sebagai catatan pribadi — bukan syarat penilaian resmi (lihat WGUI-0.9).

  

**Checklist hafalan**

  

```text

[ ] Lapisan masalah sudah diidentifikasi (account/policy/service/firewall/network)

[ ] Session lama tidak ditutup sebelum jalur baru terbukti berhasil

[ ] Tidak ada asumsi console/hypervisor tersedia — lapor ke panitia/juri adalah jalur utama saat semua akses hilang

[ ] Konsekuensi reset sudah dipahami: progres hilang, 5–8 menit, tanpa tambahan waktu, bisa mengantre

[ ] Pencegahan (satu perubahan per waktu, reconnect test) diutamakan dibanding mengandalkan reset

```

  

> “Reset panitia adalah jalan terakhir, bukan rencana utama — mencegah jauh lebih murah daripada mengulang dari nol.”

  

---

  

## WGUI-0.8 — First 5–10 Minutes Checklist

  

**Konsep**

  

Pembagian waktu lima sampai sepuluh menit di awal pengerjaan adalah target orientasi, **bukan** batas waktu mutlak. Jangan mengorbankan validasi hanya untuk mengejar angka lima atau sepuluh menit.

  

**Kenapa penting**

  

Awal pengerjaan adalah momen paling rawan kesalahan karena peserta ingin segera mulai mengubah konfigurasi. Checklist ini memastikan orientasi dilakukan lebih dulu.

  

**Lokasi GUI**

  

Sama seperti WGUI-0.6 (Baseline dan GUI Location Map) — rujukan lokasi tidak diulang di sini.

  

**Risiko lomba**

  

* Melompat langsung ke perubahan konfigurasi tanpa orientasi.

* Memburu-buru validasi recovery demi mengejar target waktu.

  

**Cara aman**

  

### Tahap 1 — Baca sebelum klik

  

* Baca objective.

* Baca rules.

* Baca daftar authorized users.

* Kenali batasan perubahan akun.

* Jangan langsung mengubah konfigurasi.

  

### Tahap 2 — Identifikasi server

  

* Hostname

* OS

* IP

* Domain atau workgroup

* Standalone, domain member, atau Domain Controller

* Akun yang sedang digunakan

  

### Tahap 3 — Periksa jalur akses

  

* RDP status

* Remote Desktop Services

* Active firewall profile

* Rule RDP

* Network adapter

* Jalur console atau hypervisor

  

### Tahap 4 — Validasi recovery yang sah

  

* Cari akun administratif resmi yang sudah tersedia.

* Periksa administrative group membership.

* Periksa hak login efektif.

* Uji login baru apabila aman dan diizinkan.

* Jangan membuat akun tambahan tanpa izin.

  

### Tahap 5 — Simpan baseline evidence

  

Ambil evidence minimum yang relevan (lihat WGUI-0.9) sebelum perubahan pertama.

  

**Checklist hafalan**

  

```text

[ ] Tahap 1 — Baca objective, rules, authorized users

[ ] Tahap 2 — Identifikasi server

[ ] Tahap 3 — Periksa jalur akses

[ ] Tahap 4 — Validasi recovery

[ ] Tahap 5 — Simpan baseline evidence

```

  

> “Baca rules, kenali server, cek akses, validasi recovery, baru ubah.”

  

---

  

## WGUI-0.9 — Safe Change Memory Pattern

  

**Konsep**

  

Pola ini menyatukan seluruh disiplin perubahan aman: diagram delapan langkah, empat aturan perubahan aman, dan perbedaan antara verifikasi, reconnect test, dan evidence.

  

**Kenapa penting**

  

Sebagian besar kegagalan lomba terjadi bukan karena peserta tidak tahu konfigurasi yang benar, tetapi karena mengubah terlalu banyak hal sekaligus tanpa verifikasi bertahap.

  

**Lokasi GUI**

  

Tidak ada lokasi GUI tunggal — pola ini berlaku lintas seluruh snap-in yang telah disebutkan pada WGUI-0.6.

  

**Risiko lomba**

  

* Mengubah account, connection, dan policy secara bersamaan.

* Melanjutkan ke perubahan berikutnya tanpa verifikasi.

* Mengambil screenshot untuk setiap klik kecil sehingga menghabiskan waktu, atau sebaliknya tidak menyimpan evidence sama sekali di titik penting.

  

**Cara aman**

  

### 1. Diagram delapan langkah

  

```text

IDENTIFY → BASELINE → RECOVERY → CHECK RISK → CHANGE → VERIFY → RECONNECT → EVIDENCE

```

  

### 2. Pertanyaan pada setiap langkah

  

| Langkah    | Pertanyaan utama                                    |

| ---------- | ---------------------------------------------------- |

| IDENTIFY   | Server apa ini dan apa objective-nya?                |

| BASELINE   | Apa kondisi awalnya?                                 |

| RECOVERY   | Apa jalan kembali yang sah dan sudah teruji?         |

| CHECK RISK | Perubahan ini menyentuh akun, koneksi, atau policy?  |

| CHANGE     | Apa satu perubahan yang akan dilakukan?              |

| VERIFY     | Apakah konfigurasi benar-benar berlaku?              |

| RECONNECT  | Apakah koneksi atau login baru masih berhasil?       |

| EVIDENCE   | Bukti apa yang perlu disimpan?                       |

  

### 3. Empat aturan perubahan aman

  

1. Satu perubahan penting dalam satu waktu.

2. Jangan mengubah account, connection, dan policy secara bersamaan.

3. Verifikasi sebelum melanjutkan.

4. Simpan evidence pada titik yang bernilai atau berisiko.

  

### 4. Perbedaan verify, reconnect, dan evidence

  

* **Verify setting** — memastikan konfigurasi benar-benar tersimpan dan effective policy sesuai (lihat WGUI-0.4).

* **Reconnect test** — memastikan koneksi atau login baru masih berhasil, bukan sekadar melihat session lama masih hidup (lihat WGUI-0.4).

* **Screenshot evidence** — dokumentasi yang disimpan pada titik bernilai, bukan pada setiap klik kecil.

  

Semua perubahan penting wajib diverifikasi secara mental dan teknis: apakah setting benar-benar berlaku, apakah akses masih tersedia, apakah terjadi efek samping, apakah effective policy sesuai, apakah login atau koneksi baru berhasil.

  

Screenshot evidence diprioritaskan untuk:

  

* baseline;

* objective bernilai poin;

* before dan after perubahan penting;

* perubahan berisiko tinggi;

* hasil final;

* recovery test;

* anomaly atau incident;

* bukti yang diminta juri.

  

**Catatan penting soal status evidence di kompetisi ini**

  

Modul A **tidak mewajibkan** pengumpulan write-up atau dokumentasi apa pun. Penilaian dilakukan oleh juri/teknisi dengan memeriksa langsung kondisi mesin peserta setelah waktu habis, secara biner per objektif (0/1). Artinya, seluruh evidence/screenshot pada bagian ini **bukan syarat penilaian resmi Modul A** — fungsinya murni untuk **self-tracking atau audit pribadi peserta** selama pengerjaan, misalnya untuk membandingkan baseline sendiri atau membantu proses rollback. Karena waktu pengerjaan Modul A hanya 3 jam, jangan membuang waktu kompetisi untuk dokumentasi berlebihan — ambil evidence secukupnya, hanya pada titik yang benar-benar berguna bagi peserta sendiri.

  

**Format nama evidence**

  

Gunakan tujuh contoh nama evidence berikut:

  

```text

W0_01_Server_Identity.png

W0_02_Network_Baseline.png

W0_03_Administrative_Membership.png

W0_04_RDP_Baseline.png

W0_05_Firewall_Baseline.png

W0_06_Effective_Policy.png

W0_07_Recovery_Login_Test.png

```

  

Evidence recovery login (`W0_07_Recovery_Login_Test.png`) hanya dibuat apabila pengujian tersebut diizinkan, aman, tidak melanggar objective, dan tidak mengharuskan membuat akun ilegal.

  

Format lebih lengkap tersedia sebagai pilihan, bukan kewajiban hafalan utama:

  

```text

W0_SRV01_01_Server_Identity_YYYYMMDD-HHMM.png

```

  

**Checklist hafalan**

  

```text

[ ] Satu perubahan penting dalam satu waktu

[ ] Account, connection, policy tidak diubah bersamaan

[ ] Verify setting sudah dilakukan

[ ] Reconnect test sudah dilakukan bila menyentuh akses

[ ] Evidence disimpan pada titik bernilai, bukan setiap klik

[ ] Ingat: evidence di sini bukan syarat penilaian resmi — murni self-tracking pribadi

```

  

> “Satu perubahan, satu verifikasi, satu reconnect bila menyentuh akses.”

  

---

  

## WGUI-0.10 — Mini Scenario Practice

  

**Konsep**

  

Enam skenario ringkas berikut melatih peserta menerapkan seluruh prinsip WGUI-0.1 sampai WGUI-0.9 dalam situasi operasional.

  

**Kenapa penting**

  

Skenario memberi konteks nyata sehingga prinsip yang bersifat abstrak menjadi lebih mudah diingat saat lomba berlangsung.

  

**Lokasi GUI**

  

Lihat WGUI-0.6 untuk rujukan lokasi GUI setiap skenario.

  

**Risiko lomba**

  

Lihat pembahasan tiap skenario di bawah.

  

**Cara aman**

  

### Scenario 1 — Membuat akun admin tanpa izin

  

* **Masalah**: peserta membuat akun administrator tambahan di awal pengerjaan tanpa memeriksa rules.

* **Penyebab**: kebiasaan menganggap “punya recovery” berarti “harus buat akun baru”.

* **Kesalahan pemula**: membuat unauthorized account yang tidak tercantum dalam daftar authorized users.

* **Langkah aman**: validasi akun resmi yang sudah tersedia terlebih dahulu; buat akun baru hanya jika diizinkan objective/rules.

* **Pelajaran**: Validasi akun resmi lebih dahulu. Membuat akun baru bukan tindakan default.

  

### Scenario 2 — Menonaktifkan atau mengubah built-in Administrator

  

* **Masalah**: peserta men-disable, me-rename, atau mereset password built-in Administrator sebagai kebiasaan otomatis.

* **Penyebab**: anggapan bahwa built-in Administrator selalu harus “diamankan” di awal.

* **Kesalahan pemula**: melakukan perubahan sensitif tanpa recovery yang teruji.

* **Langkah aman**: tindakan hanya dilakukan apabila diminta objective, sesuai baseline, ada recovery sah, recovery telah diuji, dan dampaknya dipahami.

* **Pelajaran**: rename atau disable built-in Administrator bukan tindakan default.

  

### Scenario 3 — Firewall RDP dan session lama masih hidup

  

* **Masalah**: Session A masih aktif, rule firewall sudah salah, session lama terlihat aman, tetapi login baru gagal.

* **Penyebab**: peserta menganggap session yang masih hidup sebagai bukti bahwa perubahan firewall aman.

* **Kesalahan pemula**: menutup Session A karena merasa perubahan sudah benar.

* **Langkah aman**: pertahankan Session A, gunakan Session B sebagai koneksi baru, uji login ulang, baru lanjut atau rollback.

* **Pelajaran**: Session aktif bukan reconnect test.

  

### Scenario 4 — Domain Controller dan `lusrmgr.msc`

  

* **Masalah**: peserta mencari local administrator pada Domain Controller menggunakan `lusrmgr.msc` seperti pada server biasa.

* **Penyebab**: menyamaratakan model akun DC dengan standalone/domain member.

* **Kesalahan pemula**: menganggap `lusrmgr.msc` rusak atau tidak berfungsi.

* **Langkah aman**: gunakan `dsa.msc` untuk pengelolaan akun; recovery operasional memakai authorized domain administrative account; emergency AD DS recovery mengenal DSRM melalui console.

* **Pelajaran**: `lusrmgr.msc` tidak berlaku seperti server biasa di Domain Controller.

  

### Scenario 5 — Local policy di-override GPO

  

* **Masalah**: peserta mengubah `secpol.msc`, tetapi hasil akhir tidak sesuai harapan.

* **Penyebab**: domain GPO menjadi effective policy dan meng-override setting lokal.

* **Kesalahan pemula**: hanya percaya tampilan local policy tanpa memeriksa effective policy.

* **Langkah aman**: periksa `rsop.msc`, gunakan Group Policy Results jika relevan, jangan hanya percaya tampilan local policy.

* **Pelajaran**: configured policy dapat berbeda dari effective policy.

  

### Scenario 6 — Semua akses RDP hilang

  

* **Masalah**: perubahan firewall, service, dan network dilakukan bersamaan sehingga seluruh akses RDP hilang.

* **Penyebab**: mengubah lebih dari satu kategori risiko (connection) sekaligus tanpa verifikasi bertahap.

* **Kesalahan pemula**: mengandalkan account recovery untuk mengatasi masalah jalur akses.

* **Langkah aman**: account recovery tidak cukup untuk masalah jalur akses; di kompetisi ini peserta tidak memiliki akses console/hypervisor/KVM, sehingga jalur recovery yang realistis adalah segera melapor ke panitia/juri untuk reset mesin (lihat prosedur lapor-reset pada WGUI-0.2) — progres akan hilang, prosesnya 5–8 menit, tanpa tambahan waktu, dan bisa mengantre; karena konsekuensi itu berat, pencegahan (satu perubahan per waktu, reconnect test sebelum lanjut) jauh lebih penting daripada mengandalkan reset.

* **Pelajaran**: account recovery tidak menyelamatkan firewall, service, atau network failure.

  

**Checklist hafalan**

  

```text

[ ] Enam skenario sudah dipahami polanya, bukan dihafal per kata

[ ] Setiap skenario dikaitkan dengan prinsip WGUI-0.1–0.9 yang relevan

```

  

---

  

## Target Hasil Akhir

  

Pada akhir modul, peserta harus mampu menjawab pertanyaan:

  

> “Apa yang saya lakukan sebelum Windows hardening?”

  

Jawaban refleks yang diharapkan:

  

> “Saya baca objective dan rules, identifikasi jenis server, catat baseline, validasi recovery yang diizinkan, klasifikasikan risiko akun, koneksi, atau policy, ubah satu per satu, verifikasi hasil, uji login atau koneksi baru, lalu simpan evidence yang diperlukan.”

  

Peserta juga harus memahami:

  

* recovery bukan selalu membuat akun baru;

* dua RDP session bukan dua jalur akses;

* session aktif bukan bukti reconnect berhasil;

* Domain Controller tidak mempunyai local users seperti member server;

* DSRM hanya untuk emergency recovery AD DS melalui console;

* effective policy dapat berbeda dari local policy;

* account recovery tidak menyelamatkan firewall, service, atau network failure.

  

---

  

## W0 Memory Cheat Sheet

  

### 1. Alur utama

  

```text

IDENTIFY → BASELINE → RECOVERY → RISK → CHANGE → VERIFY → RECONNECT → EVIDENCE

```

  

### 2. Tiga risiko

  

```text

ACCOUNT — CONNECTION — POLICY

```

  

### 3. Tiga tipe server

  

```text

Standalone → Local account

Domain Member → Local atau domain account

Domain Controller → Domain account

DC emergency → Console + DSRM

```

  

> Semua akun recovery harus authorized dan teruji.

  

### 4. Lima langkah orientasi

  

```text

Baca rules

Kenali server

Cek jalur akses

Validasi recovery

Simpan baseline

```

  

### 5. Tiga peringatan utama

  

```text

Akun kedua bukan jalur kedua.

Session hidup bukan reconnect test.

Jangan membuat akun admin tanpa izin.

```

  

### 6. Kalimat disiplin

  

> “Jangan hardening sebelum mengenali sistem.

> Jangan mengubah sebelum punya jalan kembali yang sah.

> Jangan lanjut sebelum verifikasi dan reconnect.”