# WGUI-2 — Group Policy & Security Baseline

  

> Modul belajar GUI-first untuk persiapan LKS Cyber Security (Blue Team)

> Bagian dari seri **WGUI — Windows GUI Hardening Learning System**

  

---

  

## WGUI-2.0 — Posisi Modul

  

Seri WGUI dibangun berlapis. Setiap modul menjadi pondasi untuk modul berikutnya.

  

| Modul | Fokus | Status |

|---|---|---|

| WGUI-0 | Navigasi Windows Server & fondasi anti-lockout | Selesai |

| WGUI-1 | Local Account, Privilege, PAM, Local Security Policy | Selesai |

| **WGUI-2** | **Group Policy & Security Baseline** | **Modul ini** |

| WGUI-3 | Active Directory Hardening | Selanjutnya |

  

Di WGUI-1, kamu belajar mengeraskan **satu komputer** lewat `secpol.msc`. Settingnya efektif hanya di komputer itu.

  

Group Policy mengubah cara kerja: satu konfigurasi bisa diterapkan ke **banyak** user dan komputer sekaligus, secara terpusat.

  

> [!MEMORY]

> WGUI-1 = mengeraskan satu mesin secara manual.

> WGUI-2 = mengeraskan banyak mesin secara terpusat dan konsisten.

  

Mengapa Group Policy harus dikuasai sebelum Active Directory Hardening (WGUI-3)? Karena hampir semua hardening AD di dunia nyata **dieksekusi lewat GPO**. Kalau kamu belum paham cara GPO dibuat, di-link, diterapkan, dan diverifikasi, materi AD hardening di WGUI-3 akan terasa seperti hafalan tanpa pemahaman.

  

---

  

## WGUI-2.1 — Gambaran Besar Group Policy

  

**Apa itu policy?**

Policy adalah aturan konfigurasi yang mengontrol perilaku sistem operasi atau user — misalnya siapa yang boleh login, aplikasi apa yang boleh berjalan, atau bagaimana password harus dibuat.

  

**Apa itu Group Policy Object (GPO)?**

GPO adalah "wadah" berisi kumpulan setting policy. Satu GPO bisa berisi puluhan setting sekaligus. GPO ini kemudian **di-link** (dikaitkan) ke lokasi tertentu di struktur domain, sehingga semua objek di lokasi itu mewarisi settingnya.

  

**Fungsi GPO:**

- Menerapkan konfigurasi keamanan secara konsisten ke banyak target

- Mengurangi kesalahan konfigurasi manual satu-per-satu

- Memusatkan kontrol administrator terhadap seluruh jaringan

- Mempercepat proses hardening skala besar saat lomba

  

**Mengapa GPO penting dalam hardening?**

Bayangkan ada 50 komputer di jaringan lomba. Mengatur password policy satu-per-satu lewat `secpol.msc` di setiap mesin sangat lambat dan rawan salah. Dengan satu GPO yang di-link ke OU yang tepat, 50 komputer itu bisa menerima setting yang sama dalam sekali update.

  

**Konfigurasi manual vs konfigurasi terpusat:**

  

| Aspek | Manual (Local Policy) | Terpusat (Group Policy Domain) |

|---|---|---|

| Skala | Satu komputer | Banyak komputer/user |

| Konsistensi | Rawan berbeda-beda | Seragam |

| Kecepatan | Lambat untuk banyak target | Cepat sekali link |

| Titik kegagalan | Tersebar di tiap mesin | Terpusat di satu GPO |

| Risiko salah target | Kecil | Besar jika salah link/scope |

  

**Peta konsep sederhana:**

  

```

Administrator

     │

     ▼

  Buat GPO  ──────────────►  Berisi kumpulan Setting

     │

     ▼

  Link GPO ke lokasi (OU / Domain / Site)

     │

     ▼

  Semua User/Computer di lokasi itu → menerima policy

     │

     ▼

  Policy diproses saat startup / login / refresh interval

     │

     ▼

  Effective Policy pada target

```

  

> [!MEMORY]

> GPO tidak otomatis berlaku ke seluruh domain. GPO baru aktif setelah **dibuat DAN di-link** ke lokasi yang benar.

  

---

  

## WGUI-2.2 — Local Policy vs Domain Policy

  

Ada tiga tool utama yang sering tertukar oleh pemula. Ketiganya terlihat mirip tapi fungsinya berbeda jauh.

  

| Aspek | `secpol.msc` | `gpedit.msc` | `gpmc.msc` |

|---|---|---|---|

| Nama | Local Security Policy | Local Group Policy Editor | Group Policy Management Console |

| Lokasi penggunaan | Komputer lokal (non-domain atau standalone) | Komputer lokal | Domain Controller (server dengan AD DS) |

| Target | Hanya komputer itu sendiri | Hanya komputer itu sendiri | Domain, Site, OU, banyak komputer/user |

| Kapan digunakan | Server berdiri sendiri, belum join domain | Uji coba setting sebelum diterapkan ke domain | Hardening skala domain, tugas utama lomba GPO |

| Kelebihan | Cepat, langsung, sederhana | Cakupan lebih luas dari secpol (ada Administrative Templates) | Bisa mengatur banyak target sekaligus, ada backup/restore |

| Risiko | Tidak terpusat, harus diulang di tiap mesin | Tetap hanya lokal, sering disangka sudah "mendunia" | Kesalahan bisa berdampak ke banyak mesin sekaligus |

| Cara verifikasi | Buka ulang `secpol.msc`, cek nilai | Buka ulang `gpedit.msc`, cek nilai | `gpresult`, RSoP, atau tab "Settings" di GPMC |

  

> [!WARNING]

> `gpedit.msc` **tidak** mengubah domain. Ia hanya mengubah Local Group Policy di komputer tempat kamu membukanya. Banyak peserta lomba salah kira mengedit di sini berarti sudah berlaku ke seluruh domain.

  

> [!CHECK]

> Sebelum mengedit apa pun, pastikan kamu tahu: apakah target objective lomba ini **lokal** atau **domain**? Salah pilih tool = salah sasaran.

  

---

  

## WGUI-2.3 — Mengenal Struktur Group Policy

  

Setiap GPO memiliki dua cabang besar:

  

```

Group Policy Object

│

├── Computer Configuration   → berlaku saat komputer STARTUP, tidak peduli siapa yang login

│     ├── Policies

│     │     ├── Windows Settings   (Security Settings, Scripts, dll)

│     │     └── Administrative Templates

│     └── Preferences

│

└── User Configuration       → berlaku saat USER LOGIN, mengikuti akun user

      ├── Policies

      │     ├── Windows Settings

      │     └── Administrative Templates

      └── Preferences

```

  

**Computer Configuration** dipakai untuk setting yang harus berlaku di mesin itu **tanpa memandang siapa yang login** — contoh: password policy, firewall, service startup, security options.

  

**User Configuration** dipakai untuk setting yang mengikuti **akun user ke mana pun user login** — contoh: pembatasan Control Panel, desktop wallpaper korporat, pemetaan drive.

  

**Contoh kapan memilih yang mana:**

  

| Kebutuhan | Pilih |

|---|---|

| Wajibkan panjang password minimal 10 karakter | Computer Configuration |

| Sembunyikan Control Panel dari user tertentu | User Configuration |

| Nonaktifkan akun Guest | Computer Configuration |

| Terapkan wallpaper khusus untuk semua staf finance | User Configuration |

| Rename akun Administrator lokal | Computer Configuration |

  

> [!DANGER]

> Menaruh setting di cabang yang salah adalah penyebab paling umum "GPO ter-link tapi tidak berlaku". Setting login screen dan account policy WAJIB dicek dulu apakah dia ada di Computer atau User Configuration sebelum dikonfigurasi.

  

---

  

## WGUI-2.4 — Tools Map

  

| Tool | Perintah Run | Fungsi | Dipakai kapan |

|---|---|---|---|

| Local Security Policy | `secpol.msc` | Atur security policy komputer lokal (password, lockout, user rights) | Server standalone atau uji cepat lokal |

| Local Group Policy Editor | `gpedit.msc` | Atur seluruh Local Group Policy (lebih luas dari secpol) | Uji setting sebelum dipakai di domain |

| Group Policy Management | `gpmc.msc` | Kelola GPO domain: buat, edit, link, backup, restore | Tugas utama hardening domain saat lomba |

| Resultant Set of Policy | `rsop.msc` | Tampilkan hasil akhir (effective policy) dari gabungan semua GPO yang berlaku pada target saat ini | Verifikasi setelah update policy |

| Group Policy Results | (dalam GPMC → klik kanan node "Group Policy Results") | Wizard untuk generate laporan effective policy dari target user/computer tertentu | Troubleshooting dan evidence lomba |

| Event Viewer (bagian Group Policy) | `eventvwr.msc` → Applications and Services Logs → Microsoft → Windows → GroupPolicy → **Operational** | Melihat proses penerapan GPO, menemukan error pemrosesan, melihat waktu refresh policy, dan membantu membedakan masalah link, permission, jaringan, atau processing | Saat GPO gagal diproses tanpa sebab jelas di GUI verifikasi lain |

  

> [!MEMORY]

> Tiga tool inti: **secpol** (lokal), **gpmc** (domain, buat & link), **rsop/gpresult** (verifikasi hasil).

  

---

  

## WGUI-2.5 — Scope dan Urutan Penerapan (LSDOU)

  

Urutan resmi penerapan Group Policy:

  

```

Local  →  Site  →  Domain  →  OU

  (L)       (S)       (D)       (O)

```

  

Prinsip dasar: kebijakan yang **lebih dekat ke objek** (paling akhir diproses, yaitu OU) biasanya **menang** jika terjadi konflik nilai setting yang sama — ini berlaku untuk mayoritas setting Group Policy biasa.

  

Namun hasil akhir bisa berubah karena:

  

- **Enforced** — GPO di level lebih atas dipaksa tidak bisa ditimpa oleh level di bawahnya

- **Block Inheritance** — OU menolak mewarisi GPO dari level di atasnya (kecuali yang Enforced)

- **Security Filtering** — GPO hanya berlaku ke user/group/computer tertentu meski sudah di-link

- **Konflik konfigurasi** — dua GPO berbeda mengatur setting yang sama dengan nilai berbeda

  

**Contoh sederhana (bukan password policy):**

  

Domain punya GPO "Domain Baseline" yang mengatur pesan judul saat logon (legal notice) menjadi "Selamat Datang". OU "Workstation-Lomba" punya GPO "SEC-Baseline-Workstation-v1" yang mengubah pesan logon menjadi "Akses Terbatas — Hanya Personel Berwenang".

  

Karena OU diproses paling akhir (lebih dekat ke objek), komputer di OU tersebut akan menampilkan pesan **"Akses Terbatas..."**, kecuali GPO Domain Baseline diset **Enforced** — jika Enforced, nilai dari domain yang menang.

  

> [!WARNING]

> Contoh di atas berlaku untuk kebanyakan setting Computer/User Configuration biasa. **Domain Password Policy, Account Lockout Policy, dan Kerberos Policy untuk akun domain adalah pengecualian penting** — tidak mengikuti pola "OU selalu menang" seperti contoh di atas. Lihat kotak khusus di bawah.

  

**Kasus khusus: Password Policy untuk akun domain**

  

```text

Password akun domain:

→ diproses dari kebijakan yang berlaku pada root domain (biasanya Default Domain

  Policy, atau GPO lain yang di-link ke root domain dengan precedence yang

  direncanakan).

  

Password akun lokal komputer:

→ dapat dipengaruhi GPO Account Policies yang di-link ke OU tempat computer

  object berada. Ini mengubah password policy akun LOKAL di komputer itu —

  BUKAN password akun domain milik user yang login ke komputer tersebut.

  

Password berbeda untuk kelompok user domain tertentu:

→ GPO Account Policies biasa yang di-link ke OU user TIDAK BISA melakukan ini.

  Gunakan Fine-Grained Password Policy / Password Settings Object (PSO).

```

  

> [!MEMORY]

> Satu domain (mode native) hanya punya **satu** Domain Password Policy efektif untuk akun domain, kecuali menggunakan Fine-Grained Password Policy. Meng-link GPO Account Policies ke OU user **tidak** menciptakan password policy domain yang berbeda untuk user di OU itu — setting tersebut tidak berpengaruh ke atribut password akun domain sama sekali.

  

> [!MEMORY]

> LSDOU: yang terakhir diproses biasanya menang — **kecuali** ada Enforced di level atas.

  

**Link Order — precedence di level/lokasi yang sama**

  

LSDOU menjelaskan urutan **antarlevel** (Local, Site, Domain, OU). Tapi bagaimana jika ada **beberapa GPO di-link pada lokasi yang sama**, misalnya tiga GPO sama-sama di-link ke satu OU?

  

Di sinilah **Link Order** berperan:

  

- Setiap GPO yang di-link ke satu lokasi memiliki nomor urut (Link Order) yang terlihat di tab **Linked Group Policy Objects** pada OU/Domain/Site tersebut.

- GPO dengan **Link Order 1** memiliki **precedence tertinggi** di lokasi itu — nilai settingnya yang menang jika terjadi konflik dengan GPO lain di lokasi yang sama.

- Semakin besar angka Link Order, semakin rendah precedence-nya di lokasi tersebut.

- Urutan Link Order dapat diubah lewat tombol panah naik/turun pada tab tersebut di GPMC.

  

> [!WARNING]

> Link Order 1 = precedence **tertinggi**, bukan terendah. Jangan tertukar dengan asumsi "angka besar berarti menang".

  

**Rumus hafalan:**

  

```text

Beda level/lokasi → lihat LSDOU

Lokasi sama       → lihat Link Order

Link Order 1      → precedence tertinggi di lokasi itu

Enforced          → tidak mudah ditimpa dari bawah, TAPI bukan solusi otomatis

                     setiap kali GPO gagal — gunakan hanya jika desain

                     kebijakan memang mengharuskannya

```

  

---

  

## WGUI-2.5A — Security Filtering, Scope, dan Effective Policy

  

Empat istilah ini sering tertukar oleh pemula. Pahami perbedaannya sebelum lanjut ke workflow.

  

```text

Link

→ lokasi tempat GPO dikaitkan. GPO hanya bisa di-link ke Site, Domain, atau OU —

  TIDAK BISA di-link langsung ke user, computer individual, atau security group.

  

Scope

→ area objek yang secara struktur berada di bawah lokasi link tersebut (termasuk

  OU-OU di bawahnya, kecuali ada Block Inheritance).

  

Security Filtering

→ dari seluruh objek yang ada di dalam scope, filter ini menentukan objek MANA

  yang benar-benar diizinkan menerapkan GPO. Security group boleh dipakai di sini

  sebagai filter — tapi security group tetap bukan lokasi link.

  

Effective Policy

→ hasil akhir setelah inheritance (LSDOU + Link Order), filtering, permission,

  konflik antar-GPO, dan processing di target selesai semua.

```

  

> [!MEMORY]

> Link menentukan **di mana** GPO dikaitkan. Security Filtering menentukan **siapa** di dalam lokasi itu yang benar-benar menerima. Effective Policy adalah **hasil akhirnya** setelah semua faktor digabung.

  

**Permission yang dibutuhkan target**

  

Agar sebuah user/computer yang berada dalam scope benar-benar menerapkan GPO, target butuh dua permission pada GPO tersebut:

  

```text

Read

Apply Group Policy

```

  

Secara default, grup **Authenticated Users** sudah memiliki kedua permission ini, sehingga secara default semua objek dalam scope menerapkan GPO. Security Filtering bekerja dengan mengganti atau menambah entri permission ini — misalnya mengganti Authenticated Users dengan security group tertentu yang sudah diberi Read + Apply Group Policy, sementara objek lain di luar grup itu tidak menerima GPO tersebut.

  

**Cara memeriksa lewat GUI:**

  

```text

GPMC

→ pilih GPO

→ tab Scope → lihat daftar Security Filtering

→ tab Delegation → Advanced

→ pilih objek/grup target

→ periksa centang Read dan Apply Group Policy

```

  

> [!CHECK]

> Kalau GPO sudah di-link dan target sudah berada di OU yang benar, tapi setting tetap tidak berlaku — jangan buru-buru curiga ke LSDOU. Cek dulu Security Filtering dan permission Apply Group Policy pada tab Delegation → Advanced. Penyebab ini sering terlewat karena tidak langsung terlihat sekilas di tab Scope.

  

---

  

## WGUI-2.6 — Workflow Membuat GPO dengan Aman

  

Ikuti alur ini setiap kali membuat GPO baru saat lomba. Jangan melompati langkah.

  

```

OBJECTIVE → SCOPE → BASELINE → RECOVERY → TEST TARGET →

CHANGE ONE → UPDATE → VERIFY → LOGIN TEST → EVIDENCE

```

  

**Langkah click-by-click:**

  

1. **Identifikasi objective** — tulis ulang objective lomba dengan kata-katamu sendiri. Apa yang benar-benar diminta?

2. **Tentukan target** — apakah objective menyasar user, computer, atau keduanya?

3. **Periksa baseline** — buka `rsop.msc` atau `gpresult /r` di target untuk tahu kondisi awal sebelum diubah.

4. **Pastikan recovery access** — pastikan ada akun/akses cadangan yang tidak terkena dampak perubahan (lihat prinsip anti-lockout WGUI-0).

5. **Buat test OU** — buka `dsa.msc` (**Active Directory Users and Computers**), klik kanan domain atau OU induk → **New** → **Organizational Unit** → beri nama jelas, misalnya `OU-Test-Hardening`.

6. **Siapkan test user/computer** — masih di `dsa.msc`, buat akun uji baru atau pindahkan akun uji (klik kanan objek → **Move**, atau drag-drop) ke dalam test OU tersebut. Jangan langsung memakai akun produksi.

7. **Buat GPO baru** — buka `gpmc.msc`, klik kanan **Group Policy Objects** → **New** → beri nama deskriptif.

8. **Edit hanya setting yang diperlukan** — klik kanan GPO → **Edit** → masuk ke Computer/User Configuration sesuai kebutuhan → ubah **satu kelompok setting kecil saja**.

9. **Link ke test OU** — klik kanan test OU → **Link an Existing GPO** → pilih GPO yang baru dibuat.

10. **Update policy** — di target, jalankan `gpupdate /force`.

11. **Verifikasi effective policy** — buka `rsop.msc` di target atau gunakan **Group Policy Results** di GPMC.

12. **Lakukan login test** — logoff/login ulang (untuk User Configuration) atau restart (untuk Computer Configuration), lalu coba login dengan akun uji.

13. **Ambil evidence** — screenshot hasil effective policy dan hasil login test.

14. **Baru perluas scope** — jika semua langkah di atas berhasil tanpa masalah, baru link GPO ke OU produksi yang sesuai objective.

  

> [!CHECK]

> Pembagian fungsi dua tool ini sering tertukar oleh pemula:

> - `dsa.msc` (Active Directory Users and Computers) → mengelola **objek** Active Directory: OU, user, group, computer.

> - `gpmc.msc` (Group Policy Management Console) → mengelola **GPO**: membuat, mengedit, me-link, membackup, dan menganalisis kebijakan.

>

> OU dan akun uji dibuat lewat `dsa.msc`. GPO dibuat, di-edit, dan di-link lewat `gpmc.msc`.

  

**Contoh penamaan GPO:**

  

```

SEC-Baseline-Workstation-v1

```

  

Format ini menjelaskan: kategori (SEC = Security), tujuan (Baseline), target (Workstation), dan versi (v1) — memudahkan tracking saat rollback.

  

> [!DANGER]

> **Jangan menaruh seluruh hardening** (workstation, firewall, Defender, browser, RDP, audit, pembatasan user, dan sejenisnya) ke dalam **Default Domain Policy**. GPO ini otomatis di-link ke seluruh domain sejak awal, sehingga kesalahan kecil di sini bisa berdampak ke **semua** user dan komputer sekaligus, termasuk domain controller. Untuk hardening tambahan, selalu buat GPO terpisah dengan nama dan scope yang jelas.

>

> Ini **bukan berarti** Default Domain Policy sama sekali tidak boleh disentuh. Domain Password Policy, Account Lockout Policy, dan Kerberos Policy untuk akun domain memang harus berasal dari kebijakan yang berlaku pada **root domain** — dan kebijakan itu bisa saja berada di Default Domain Policy, atau di GPO khusus lain yang di-link ke root domain dengan precedence yang sudah direncanakan (lihat WGUI-2.5). Yang harus dihindari adalah menumpuk hardening yang tidak berkaitan dengan Account Policies ke GPO ini, dan membuat dua sumber Account Policy domain yang saling bertentangan tanpa memahami Link Order.

  

> [!MEMORY]

> Default Domain Policy bukan tempat semua hardening. Namun Account Policies akun domain (password, lockout, Kerberos) tetap harus berasal dari kebijakan di root domain.

  

---

  

## WGUI-2.7 — Praktik Mini-Baseline untuk Memahami Workflow GPO

  

**Objective:** Ini **bukan** baseline Windows yang lengkap. Praktik ini adalah **mini-baseline** untuk melatih mekanisme GPO secara utuh: membuat GPO, mengedit setting, melakukan link, melakukan update, memverifikasi effective policy, melakukan login test, mengambil evidence, dan melakukan rollback. Setting yang dipakai sengaja sedikit (dari WGUI-1) agar fokus tetap pada workflow, bukan pada menghafal banyak setting.

  

> [!CHECK]

> Baseline Windows yang sesungguhnya jauh lebih luas: mencakup Account dan logon security, User Rights Assignment, Security Options, Windows Defender Firewall, Microsoft Defender, Advanced Audit Policy, Remote Desktop, SMB dan protokol lama, Services, hingga Administrative Templates. WGUI-2 tidak membahas semuanya secara mendalam — modul ini fokus pada **mekanisme Group Policy dan deployment yang aman**. Cakupan setting yang lebih luas dipelajari pada praktik hardening lanjutan berikutnya.

  

**Target:** Computer Configuration, di-link ke `OU-Test-Hardening`.

  

**Setting yang digunakan (contoh, bukan pembahasan ulang teori WGUI-1):**

- Guest account status: Disabled

- Interactive logon: Do not display last signed-in user name → Enabled

- Rename built-in Administrator account

- Setting UAC yang aman (mode default: notify saat aplikasi mencoba melakukan perubahan)

  

**Risiko:**

> [!WARNING]

> Rename Administrator sebaiknya selalu didokumentasikan agar tidak lupa nama akun baru saat butuh akses darurat. Tapi ini termasuk **risiko rendah sampai sedang** — bukan penyebab utama lockout. Tetap pastikan ada akun admin uji lain yang aktif sebelum mulai, sebagai kebiasaan baik. Risiko lockout yang sesungguhnya besar datang dari kategori setting lain — lihat WGUI-2.7A.

  

**Jalur GUI:**

`gpmc.msc` → klik kanan GPO → **Edit** → **Computer Configuration** → **Policies** → **Windows Settings** → **Security Settings** → **Local Policies** → **Security Options**

  

**Langkah konfigurasi:**

1. Buka GPMC, buat GPO baru bernama `SEC-Baseline-Workstation-v1`.

2. Klik kanan → **Edit**.

3. Masuk ke path di atas.

4. Cari **Accounts: Guest account status** → set **Disabled**.

5. Cari **Interactive logon: Do not display last signed-in user name** → set **Enabled**.

6. Cari **Accounts: Rename administrator account** → isi nama baru sesuai konvensi tim.

7. Cari **User Account Control: Behavior of the elevation prompt for administrators** → pastikan tidak diset ke **Elevate without prompting**.

8. Tutup editor.

  

**Update:**

Di komputer target, jalankan:

```

gpupdate /force

```

  

**Verifikasi:**

Buka `rsop.msc` di target, arahkan ke path yang sama, pastikan nilai sudah sesuai konfigurasi.

  

**Login test:**

Coba login dengan akun uji biasa (bukan admin) untuk memastikan tidak ada lockout, dan cek nama user terakhir tidak tampil di layar login.

  

**Evidence:**

- Screenshot GPO dan setting di editor

- Screenshot hasil `rsop.msc` yang menunjukkan nilai efektif

- Screenshot layar login yang tidak menampilkan nama user terakhir

  

**Rollback:**

Jika bermasalah, set kembali setting ke **Not Configured** di GPO, lalu `gpupdate /force` ulang di target, atau unlink GPO dari OU.

  

> [!CHECK]

> Fokus praktik ini adalah **memahami alur workflow**, bukan menghafal banyak setting sekaligus.

  

---

  

## WGUI-2.7A — Tingkat Risiko Lockout

  

Tidak semua setting hardening punya risiko lockout yang sama besar. Kenali tingkatannya supaya kehati-hatian difokuskan ke tempat yang tepat, bukan tersebar rata ke semua setting.

  

**Risiko rendah sampai sedang**

- Rename Administrator tanpa dokumentasi

- Lupa atau salah memahami nama akun baru hasil rename

- Salah target Security Filtering

- Salah cabang Computer/User Configuration

  

**Risiko tinggi**

- Deny log on locally

- Deny log on through Remote Desktop Services

- Allow log on locally / Allow log on through Remote Desktop Services yang salah dikonfigurasi hingga menghapus akses yang valid

- Perubahan membership grup Administrators

- Account lockout policy yang ekstrem (threshold terlalu rendah, durasi terlalu lama)

- Firewall yang memutus akses RDP

- Menonaktifkan service atau koneksi jaringan yang diperlukan untuk manajemen

- Menerapkan GPO berisiko langsung ke Domain Controllers

- Salah link GPO ke domain atau OU produksi (bukan OU uji)

  

> [!DANGER]

> Rename Administrator harus tetap didokumentasikan. Namun, risiko lockout terbesar biasanya berasal dari **User Rights Assignment, pembatasan RDP, firewall, account lockout policy ekstrem, dan perubahan grup Administrators** — bukan sekadar dari disable Guest atau rename Administrator.

  

---

  

## WGUI-2.8 — Policy Update dan Effective Policy

  

Lima kondisi ini sering disamakan padahal berbeda:

  

| Kondisi | Artinya |

|---|---|

| **Configured** | Setting sudah diklik dan disimpan di editor GPO |

| **Linked** | GPO sudah dikaitkan ke lokasi (Site/Domain/OU) |

| **Applied** | Target sudah menerima dan memproses GPO tersebut (butuh update/refresh) |

| **Effective** | Setting menjadi nilai akhir yang berlaku, hasil dari inheritance, filtering, permission, dan konflik antar-GPO |

| **Verified** | Administrator sudah membuktikannya lewat RSoP, Group Policy Results, `gpresult`, log Event Viewer, atau pengujian langsung (login test) |

  

> [!MEMORY]

> **Configured ≠ Linked ≠ Applied ≠ Effective ≠ Verified.**

> Editor yang terlihat benar bukan bukti bahwa policy sudah benar-benar berjalan di target — dan effective policy yang terlihat di RSoP juga baru benar-benar "verified" setelah dibuktikan lewat pengujian nyata.

  

GUI tetap jadi metode utama untuk memverifikasi (lewat `rsop.msc` atau Group Policy Results di GPMC). Command line hanya sebagai alternatif validasi cepat:

  

```

gpupdate /force

gpresult /r

```

  

`gpupdate /force` memaksa refresh policy tanpa menunggu interval otomatis. `gpresult /r` menampilkan ringkasan GPO apa saja yang berlaku pada sesi saat ini.

  

> [!WARNING]

> Sebagian setting Computer Configuration butuh **restart**, bukan hanya `gpupdate`, untuk benar-benar aktif (contoh: beberapa security options). Sebagian setting User Configuration butuh **logoff/login ulang**.

  

---

  

## WGUI-2.9 — Verification Matrix

  

| Yang diperiksa | Metode GUI | Alternatif command | Hasil yang diharapkan |

|---|---|---|---|

| Link GPO | GPMC → klik OU → tab "Linked Group Policy Objects" | — | GPO tampil dalam daftar link |

| Scope | GPMC → klik GPO → tab "Scope" | — | Security Filtering dan target sesuai rencana |

| Security filtering | GPMC → tab "Scope" → bagian Security Filtering | `gpresult /r` | Hanya user/group/computer yang dimaksud tercantum |

| Effective setting | `rsop.msc` di target | `gpresult /h report.html` | Nilai setting sesuai konfigurasi GPO |

| User target | GPMC → Group Policy Results Wizard | `gpresult /r /user <namauser>` | GPO yang relevan muncul di hasil |

| Computer target | GPMC → Group Policy Results Wizard | `gpresult /r` | GPO computer-level muncul di hasil |

| Waktu update | `eventvwr.msc` → GroupPolicy → **Operational** | `gpupdate /force` lalu cek ulang | Timestamp terbaru sesuai waktu update |

| Hasil login test | Layar login langsung | — | Perilaku sistem sesuai objective (mis. tidak lockout) |

  

---

  

## WGUI-2.10 — Backup dan Rollback

  

**Backup GPO:**

GPMC → klik kanan **Group Policy Objects** → **Back Up All**, atau klik kanan satu GPO → **Back Up**. Simpan ke folder yang jelas namanya.

  

**Restore GPO:**

GPMC → klik kanan **Group Policy Objects** → **Manage Backups** → pilih backup → **Restore**.

  

**Unlink GPO:**

Klik kanan link GPO di OU/domain → **Delete** (ini hanya menghapus link, GPO aslinya tetap ada di **Group Policy Objects**).

  

**Disable link:**

Klik kanan link GPO → centang/hilangkan centang **Link Enabled**. GPO tetap ada dan tetap ter-link, tapi tidak diproses.

  

**Disable User/Computer Configuration:**

Klik kanan GPO → **GPO Status** → pilih **User Configuration Settings Disabled** atau **Computer Configuration Settings Disabled** sesuai kebutuhan, agar hanya satu cabang yang aktif dan proses login/startup lebih cepat.

  

**Mengembalikan setting ke Not Configured:**

Buka editor GPO → cari setting → set kembali ke **Not Configured**, bukan sekadar "Disabled" (karena Disabled tetap sebuah nilai aktif, berbeda dari tidak diatur sama sekali).

  

**Kapan rollback harus dilakukan:**

- Login test gagal

- Ada gejala lockout

- Setting salah target (User vs Computer, OU salah)

- Objective ternyata salah dipahami

  

> [!DANGER]

> Menghapus GPO **bukan** langkah rollback pertama. Urutan yang benar: **unlink atau disable link** dulu → verifikasi masalah hilang → baru pertimbangkan hapus GPO jika memang sudah tidak diperlukan sama sekali.

  

---

  

## WGUI-2.11 — Troubleshooting

  

| Gejala | Penyebab umum | Pemeriksaan | Solusi |

|---|---|---|---|

| GPO tidak muncul di daftar | Salah lokasi GPMC, atau GPO ada di domain lain | Cek node **Group Policy Objects** di GPMC | Pastikan berada di domain/forest yang benar |

| GPO ter-link tapi tidak diterapkan | Link Enabled tidak dicentang, atau security filtering salah | Tab "Scope" pada GPO | Aktifkan link, perbaiki filtering |

| Setting salah ditaruh di User Configuration | Setting seharusnya di Computer Configuration | Baca ulang deskripsi setting di editor | Pindahkan ke cabang yang benar |

| Setting salah ditaruh di Computer Configuration | Kebalikan dari di atas | Sama seperti di atas | Pindahkan ke cabang yang benar |

| Target berada di OU yang salah | Objek belum dipindahkan ke OU yang di-link GPO | Cek lokasi objek di Active Directory Users and Computers | Pindahkan objek ke OU yang tepat |

| Security filtering tidak sesuai | Default filtering "Authenticated Users" tidak diubah sesuai kebutuhan | Tab "Scope" → Security Filtering | Tambahkan/kurangi grup sesuai target |

| Policy belum diperbarui | Belum `gpupdate` atau interval refresh belum tercapai | Jalankan `gpupdate /force` | Tunggu proses selesai, cek ulang |

| Konflik dengan GPO lain | Dua GPO mengatur setting sama dengan nilai beda | `gpresult /r` untuk lihat GPO yang menang | Cek urutan link (link order) atau gunakan Enforced |

| Block Inheritance aktif | OU menolak warisan dari atas | Ikon OU di GPMC (ada tanda seru biru) | Evaluasi apakah Block Inheritance memang diperlukan |

| Enforced aktif di GPO lain | GPO atas dipaksa menang meski OU punya setting berbeda | Cek atribut "Enforced" pada link GPO | Sesuaikan mana yang seharusnya menang |

| User belum logoff/login | Setting User Configuration butuh sesi baru | Tanyakan status sesi user saat ini | Minta logoff lalu login ulang |

| Computer belum restart | Setting Computer Configuration tertentu butuh restart | Cek jenis setting di editor | Restart komputer target |

| Setting masih Not Configured | Lupa mengubah dari default | Buka ulang editor GPO | Ubah ke nilai yang dimaksud |

| Membuka editor yang salah | Memakai `gpedit.msc` padahal maksudnya domain | Cek judul jendela editor | Gunakan `gpmc.msc` untuk domain |

| GPO sudah di-link, target sudah di OU yang benar, tapi tetap tidak berlaku | Target tidak punya permission **Apply Group Policy** (Security Filtering sudah terlanjur diubah dari default) | GPMC → GPO → tab Delegation → **Advanced** → cek permission target | Tambahkan **Read** + **Apply Group Policy** untuk target, langsung atau lewat security group yang sesuai |

| GPO gagal diproses tanpa sebab yang terlihat di RSoP/gpresult | Kegagalan tingkat rendah (network, permission, file GPO corrupt, dll) yang tidak selalu tampil di GUI verifikasi lain | `eventvwr.msc` → Applications and Services Logs → Microsoft → Windows → GroupPolicy → **Operational** | Baca detail pesan error di log, sesuaikan solusi dengan penyebab yang tertulis |

  

---

  

## WGUI-2.12 — Skenario Lomba

  

### Skenario 1 — Setting lokal ditimpa domain policy

**Situasi:** Peserta sudah set password policy lewat `secpol.msc`, tapi saat dicek ulang nilainya kembali berubah.

**Risiko:** Waktu terbuang mengulang setting yang percuma.

**Langkah berpikir:** Apakah komputer ini join domain? Jika ya, kemungkinan besar ada GPO domain yang menimpa local policy.

**Solusi:** Cek `gpresult /r`, temukan GPO domain yang mengatur password policy, sesuaikan di sana, bukan di local policy.

**Evidence akhir:** Screenshot `gpresult` menunjukkan GPO domain sebagai sumber setting yang aktif.

  

### Skenario 2 — GPO dibuat tapi belum di-link

**Situasi:** GPO sudah lengkap dikonfigurasi tapi target tidak menerima perubahan apa pun.

**Risiko:** Objective dianggap gagal padahal tinggal satu langkah.

**Langkah berpikir:** Apakah GPO sudah muncul di tab "Linked Group Policy Objects" pada OU target?

**Solusi:** Link GPO ke OU yang benar, lalu `gpupdate /force`.

**Evidence akhir:** Screenshot tab Scope menunjukkan link aktif dan `rsop.msc` menunjukkan setting efektif.

  

### Skenario 3 — GPO di-link ke OU yang salah

**Situasi:** GPO sudah ter-link tapi target uji tetap tidak berubah.

**Risiko:** Peserta mengira GPO rusak padahal hanya salah target.

**Langkah berpikir:** Di OU mana sebenarnya objek target berada?

**Solusi:** Pindahkan objek ke OU yang benar, atau ubah link GPO ke OU yang sesuai.

**Evidence akhir:** Screenshot lokasi objek di Active Directory Users and Computers dan hasil verifikasi setelahnya.

  

### Skenario 4 — User policy ditempatkan pada Computer Configuration

**Situasi:** Setting pembatasan Control Panel tidak muncul untuk user manapun.

**Risiko:** Waktu habis mengecek link dan filtering padahal akar masalah di cabang konfigurasi.

**Langkah berpikir:** Apakah setting ini seharusnya mengikuti user atau komputer?

**Solusi:** Pindahkan setting ke User Configuration.

**Evidence akhir:** Screenshot setting baru di cabang yang benar dan hasil efektif di `rsop.msc` mode User.

  

### Skenario 5 — Kebijakan menyebabkan user test tidak bisa login

**Situasi:** Setelah menerapkan GPO, akun uji tidak bisa login sama sekali.

**Risiko:** Lockout pada target uji, berpotensi menular jika scope diperluas.

**Langkah berpikir:** Setting apa yang baru saja diubah? Apakah berkaitan dengan logon rights atau account lockout?

**Solusi:** Rollback segera — disable link GPO, verifikasi login kembali normal, baru telusuri setting yang bermasalah satu per satu.

**Evidence akhir:** Screenshot sebelum dan sesudah rollback, serta login berhasil kembali.

  

### Skenario 6 — Membuktikan setting efektif dengan evidence

**Situasi:** Juri meminta bukti bahwa objective benar-benar tercapai, bukan hanya terlihat di editor.

**Risiko:** Poin hilang jika hanya screenshot editor tanpa bukti efektif.

**Langkah berpikir:** Apakah aku sudah menunjukkan hasil `rsop.msc`/`gpresult`, bukan cuma tampilan GPO Edit?

**Solusi:** Kumpulkan evidence lengkap sesuai standar di WGUI-2.9 (bagian Verification Matrix).

**Evidence akhir:** Kombinasi screenshot: GPO, link, effective policy, dan hasil login test.

  

### Skenario 7 — Target tidak memiliki permission Apply Group Policy

**Situasi:** GPO sudah dibuat dengan benar, sudah di-link ke OU yang tepat, dan objek target memang berada di OU tersebut — tapi setting tetap tidak muncul di `rsop.msc` target.

**Risiko:** Waktu habis mengecek ulang link dan lokasi OU yang sebenarnya sudah benar, padahal akar masalah ada di tempat lain.

**Langkah berpikir:** Apakah target memiliki permission Read dan Apply Group Policy pada GPO ini? Cek tab Scope → Security Filtering, dan tab Delegation → Advanced.

**Solusi:** Tambahkan Read + Apply Group Policy untuk target (langsung atau lewat security group), atau perbaiki Security Filtering yang sudah terlanjur diubah dari default Authenticated Users.

**Evidence akhir:** Screenshot tab Delegation → Advanced menunjukkan permission sebelum dan sesudah diperbaiki, serta hasil `rsop.msc` yang sudah menunjukkan setting efektif.

  

---

  

## WGUI-2.13 — Kesalahan Fatal

  

Kesalahan yang paling sering dilakukan peserta lomba pemula:

  

- Mengubah banyak setting sekaligus dalam satu GPO tanpa pengujian bertahap

- Mengedit **Default Domain Policy** tanpa alasan kuat

- Tidak membuat test OU sebelum deploy ke produksi

- Tidak mengetahui dengan pasti target objective: user atau computer

- Mengira setting yang terlihat benar di editor pasti sudah efektif

- Mengira GPO Account Policies yang di-link ke OU user bisa membuat password policy domain berbeda, padahal harus dari root domain atau Fine-Grained Password Policy (lihat WGUI-2.5)

- Menganggap disable Guest atau rename Administrator sebagai penyebab utama lockout, padahal risiko terbesar ada di User Rights Assignment, RDP, firewall, dan account lockout policy (lihat WGUI-2.7A)

- Tidak melakukan login test setelah perubahan

- Tidak menyiapkan rencana rollback sebelum mengubah setting berisiko

- Mengambil screenshot evidence sebelum konfigurasi benar-benar diterapkan (misalnya lupa `gpupdate` dulu)

  

> [!DANGER]

> Kombinasi "ubah banyak sekaligus" + "tidak ada rollback" adalah penyebab paling umum kegagalan total di sesi lomba Group Policy.

  

---

  

## WGUI-2.14 — Memory Guide

  

**Empat tool utama:**

- `dsa.msc` → kelola objek AD: OU, user, group, computer

- `secpol.msc` → lokal, satu mesin

- `gpmc.msc` → domain, buat, edit, & link GPO

- `rsop.msc` / `gpresult` → verifikasi hasil efektif

  

**Local vs Domain:**

Local = cepat tapi tidak terpusat. Domain = terpusat tapi berisiko lebih luas jika salah.

  

**Computer vs User:**

Computer Configuration → berlaku saat startup, tanpa peduli siapa yang login.

User Configuration → berlaku saat login, mengikuti akun.

  

**Urutan LSDOU:**

Local → Site → Domain → OU. Yang terakhir diproses biasanya menang, kecuali ada **Enforced**.

  

**Link Order:**

Di lokasi (OU/Domain/Site) yang sama, GPO dengan Link Order **1** punya precedence tertinggi.

  

**Password akun domain:**

Selalu dari kebijakan di root domain (Default Domain Policy atau GPO lain yang di-link ke root). GPO di OU user tidak bisa membuat password domain berbeda — untuk itu gunakan Fine-Grained Password Policy.

  

**Link vs Scope vs Security Filtering vs Effective Policy:**

Link = di mana GPO dikaitkan (Site/Domain/OU saja). Scope = area yang tercakup secara struktur. Security Filtering = siapa di dalam scope yang benar-benar menerima (butuh permission Read + Apply Group Policy). Effective Policy = hasil akhir sesudah semuanya digabung.

  

**Tingkat risiko lockout:**

Risiko tinggi ada di User Rights Assignment, pembatasan RDP, firewall, account lockout ekstrem, dan perubahan grup Administrators — bukan sekadar disable Guest atau rename Administrator.

  

**Workflow aman:**

```

OBJECTIVE → SCOPE → BASELINE → RECOVERY → TEST TARGET →

CHANGE ONE → UPDATE → VERIFY → LOGIN TEST → EVIDENCE

```

  

**Verification checklist:**

- [ ] GPO ter-link ke lokasi benar

- [ ] Scope dan security filtering sesuai target

- [ ] Target punya permission Read + Apply Group Policy

- [ ] `gpupdate /force` sudah dijalankan

- [ ] `rsop.msc` menunjukkan setting efektif

- [ ] Login test dilakukan dan berhasil

  

**Rollback checklist:**

- [ ] Disable link atau unlink GPO

- [ ] Verifikasi target kembali normal

- [ ] Set setting bermasalah ke Not Configured

- [ ] Dokumentasikan penyebab sebelum mencoba ulang

  

**Enam penyebab umum GPO gagal:**

1. Belum di-link

2. Salah cabang (User vs Computer)

3. Salah OU target

4. Belum `gpupdate`/belum restart/logoff-login

5. Ditimpa GPO lain (Enforced/Block Inheritance/Link Order)

6. Target tidak punya permission Apply Group Policy (Security Filtering)

  

---

  

## WGUI-2.15 — Combat Card Satu Halaman

  

```

══════════════════════════════════════════

   WGUI-2 COMBAT CARD — GROUP POLICY

══════════════════════════════════════════

  

IDENTIFY

  → Baca objective ulang dengan kata sendiri

  → Tentukan: User atau Computer?

  

TARGET

  → Cari lokasi OU objek yang dimaksud

  → Pastikan bukan Default Domain Policy

  

TEST

  → Buat/pakai test OU

  → Siapkan akun uji, bukan akun produksi

  

CONFIGURE

  → Buat GPO baru, nama jelas (SEC-xxx-v1)

  → Ubah HANYA setting yang diperlukan

  → Pastikan cabang benar: Computer / User

  

UPDATE

  → Link GPO ke test OU

  → gpupdate /force di target

  

VERIFY

  → rsop.msc atau Group Policy Results

  → Bandingkan nilai dengan objective

  

LOGIN TEST

  → Logoff/login (User) atau restart (Computer)

  → Coba login dengan akun uji

  

EVIDENCE

  → Screenshot: GPO, Link, Effective Policy, Login

  

ROLLBACK

  → Disable link dulu, bukan hapus GPO

  → Verifikasi target kembali normal

  → Baru investigasi ulang

  

══════════════════════════════════════════

INGAT: Configured ≠ Linked ≠ Applied ≠ Effective ≠ Verified

══════════════════════════════════════════

```

  

---

  

## WGUI-2.16 — Quiz dan Latihan

  

### A. Pilihan Ganda

  

1. Tool mana yang digunakan untuk mengatur policy di domain, bukan komputer lokal?

   a) secpol.msc  b) gpedit.msc  c) gpmc.msc  d) taskschd.msc

  

2. Setting yang berlaku saat komputer startup tanpa memandang siapa yang login berada di cabang:

   a) User Configuration  b) Computer Configuration  c) Local Preferences  d) Site Configuration

  

3. Urutan LSDOU yang benar adalah:

   a) Site → Local → OU → Domain

   b) Local → Site → Domain → OU

   c) Domain → OU → Site → Local

   d) OU → Domain → Site → Local

  

4. Fungsi utama `rsop.msc` adalah:

   a) Membuat GPO baru

   b) Menampilkan effective policy pada target

   c) Menghapus GPO

   d) Menautkan GPO ke OU

  

5. GPO yang otomatis di-link ke seluruh domain sejak awal dan berisiko tinggi bila diedit sembarangan adalah:

   a) SEC-Baseline-Workstation-v1

   b) Default Domain Policy

   c) Local Group Policy

   d) OU-Test-Hardening

  

6. Jika sebuah GPO di level Domain diset **Enforced**, maka:

   a) OU di bawahnya selalu menang

   b) Domain tetap menang meski OU punya setting berbeda

   c) Enforced tidak berpengaruh apa pun

   d) GPO otomatis terhapus

  

7. Cara cepat memaksa refresh policy tanpa menunggu interval otomatis:

   a) gpresult /r

   b) gpupdate /force

   c) rsop.msc

   d) secpol.msc

  

8. Langkah pertama saat rollback GPO yang bermasalah adalah:

   a) Menghapus GPO

   b) Restart domain controller

   c) Disable link atau unlink GPO

   d) Membuat GPO baru lagi

  

9. Setting "Do not display last signed-in user name" sebaiknya diverifikasi lewat:

   a) Membaca deskripsi setting saja

   b) Login test langsung di layar logon

   c) Menghapus GPO

   d) Membuka Event Viewer saja

  

10. Security Filtering pada GPO berguna untuk:

    a) Mengganti nama GPO

    b) Membatasi target penerima GPO meski sudah di-link ke OU

    c) Membackup GPO

    d) Mengubah urutan LSDOU

  

11. Password Policy untuk akun domain (bukan akun lokal) berasal dari:

    a) GPO apa pun yang di-link ke OU tempat user berada

    b) GPO yang di-link ke root domain (atau Fine-Grained Password Policy untuk kelompok tertentu)

    c) `secpol.msc` di setiap komputer

    d) Local Group Policy Editor

  

12. Sebuah Organizational Unit (OU) baru sebaiknya dibuat melalui:

    a) `gpmc.msc`

    b) `rsop.msc`

    c) `dsa.msc` (Active Directory Users and Computers)

    d) `eventvwr.msc`

  

13. Jika ada tiga GPO di-link pada OU yang sama, GPO dengan Link Order berapa yang punya precedence tertinggi di OU tersebut?

    a) Link Order dengan angka terbesar

    b) Link Order 1

    c) Semua GPO punya precedence yang sama

    d) Tergantung urutan alfabet nama GPO

  

14. Agar target yang berada dalam scope benar-benar menerapkan sebuah GPO, target minimal harus memiliki permission:

    a) Write dan Modify

    b) Read dan Apply Group Policy

    c) Full Control saja

    d) Delete dan Create

  

**Kunci Jawaban:** 1-c, 2-b, 3-b, 4-b, 5-b, 6-b, 7-b, 8-c, 9-b, 10-b, 11-b, 12-c, 13-b, 14-b

  

### B. Benar atau Salah

  

1. `gpedit.msc` dapat langsung mengubah policy seluruh domain. **(Salah — hanya lokal)**

2. Computer Configuration berlaku saat startup, User Configuration berlaku saat login. **(Benar)**

3. Menghapus GPO adalah langkah rollback pertama yang dianjurkan. **(Salah — unlink/disable dulu)**

4. Setting yang terlihat benar di editor sudah pasti efektif di target. **(Salah — perlu verifikasi rsop/gpresult)**

5. Block Inheritance dapat menghalangi OU mewarisi GPO dari level di atasnya, kecuali GPO tersebut Enforced. **(Benar)**

6. Default Domain Policy sama sekali tidak boleh disentuh, bahkan untuk Account Policies. **(Salah — Account Policies akun domain memang harus berasal dari kebijakan root domain, yang boleh berada di Default Domain Policy)**

  

### C. Studi Kasus Singkat

  

1. Setting password policy sudah dikonfigurasi di GPO dan sudah di-link, tapi target masih memakai nilai lama. Apa dua hal pertama yang harus dicek?

   *(Jawaban inti: apakah sudah `gpupdate /force` dijalankan, dan — khusus untuk password policy akun domain — apakah GPO tersebut benar-benar di-link ke root domain, bukan ke OU user, atau apakah ada GPO lain dengan Link Order lebih tinggi di lokasi yang sama yang menimpanya.)*

  

2. Peserta ingin membatasi Control Panel untuk staf tertentu, tapi settingnya ditaruh di Computer Configuration dan tidak berjalan. Apa penyebabnya dan solusinya?

   *(Jawaban inti: setting ini seharusnya mengikuti user, pindahkan ke User Configuration.)*

  

3. Setelah menerapkan GPO baru, akun uji tidak bisa login. Apa langkah pertama yang harus diambil?

   *(Jawaban inti: rollback segera dengan disable link GPO, verifikasi login normal kembali, baru investigasi.)*

  

4. Juri meminta bukti bahwa policy benar-benar efektif, bukan hanya terkonfigurasi. Evidence apa saja yang minimal harus disiapkan?

   *(Jawaban inti: nama GPO, lokasi link, target OU, setting yang dikonfigurasi, hasil effective policy, identitas user/computer uji, timestamp, dan screenshot hasil akhir — bukan hanya screenshot editor.)*

  

5. GPO sudah dibuat dengan setting yang benar tapi tidak muncul efeknya sama sekali di target manapun. Apa kemungkinan penyebab paling dasar yang harus dicek lebih dulu?

   *(Jawaban inti: apakah GPO sudah benar-benar di-link ke suatu lokasi — GPO yang belum di-link tidak berlaku ke mana pun.)*

  

6. GPO sudah di-link ke OU yang benar dan target berada di OU tersebut, tapi setting tetap tidak berlaku sama sekali di target. Link dan lokasi sudah dipastikan benar. Apa yang harus dicek berikutnya?

   *(Jawaban inti: cek Security Filtering pada GPO — pastikan target atau grup yang menaunginya memiliki permission Read dan Apply Group Policy, bukan hanya Read saja.)*

  

---

  

**Selesai — WGUI-2_Group_Policy_Security_Baseline.md**