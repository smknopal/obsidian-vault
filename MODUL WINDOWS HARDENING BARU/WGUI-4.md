# WGUI-4 — WINDOWS NETWORK & FIREWALL HARDENING

  

### Windows GUI Hardening Learning System

  

**Modul Persiapan Kompetisi Cyber Security LKS Tingkat Nasional — Blue Team Track**

  

> Modul ini merupakan kelanjutan dari WGUI-0 (Windows Hardening Foundation), WGUI-1 (Local Account, PAM, Local Security Policy), WGUI-2 (Group Policy Hardening), dan WGUI-3 (Active Directory Hardening). WGUI-4 fokus pada hardening jaringan Windows Server melalui GUI: Windows Defender Firewall with Advanced Security, konfigurasi jaringan, pembatasan service exposure, RDP, SMB, DNS, network profile, serta proses verification, rollback, troubleshooting, dan evidence.

  

---

  

## WGUI-4.0 Posisi dan Tujuan Modul

  

WGUI-4 berada pada lapisan **jaringan** dari keseluruhan seri hardening. Modul sebelumnya menutup identitas (akun, GPO, AD). Modul ini menutup jalur komunikasi menuju dan dari server.

  

Empat area yang dibedakan dalam modul ini:

  

| Area | Definisi | Contoh |

|---|---|---|

| Network configuration | Pengaturan adapter, IP, DNS, gateway | `ncpa.cpl`, TCP/IPv4 properties |

| Service exposure | Service apa yang mendengarkan di jaringan | `services.msc`, `netstat -ano` |

| Firewall enforcement | Aturan yang mengizinkan/menolak traffic | `wf.msc` |

| Network monitoring | Pencatatan dan verifikasi aktivitas jaringan | Firewall log, Event Viewer |

  

Keempatnya saling bergantung. Salah konfigurasi pada satu lapisan dapat merusak lapisan lain.

  

### Mengapa salah konfigurasi jaringan berbahaya

  

| Kesalahan | Akibat |

|---|---|

| Mengubah/menghapus rule RDP yang aktif | RDP lockout, kehilangan akses remote |

| Mengganti DNS adapter domain tanpa alasan jelas | DNS failure, resolusi nama domain gagal |

| Mengubah gateway/IP tanpa baseline | Domain communication failure |

| Menonaktifkan service yang masih dipakai | Service outage |

| Memblokir port administrasi tanpa recovery path | Loss of remote administration |

  

> Modul ini tidak mengulang teori GPO dan Active Directory dari WGUI-2 dan WGUI-3. Fokus murni pada jaringan dan firewall.

  

---

  

## WGUI-4.1 Mental Model Jaringan Windows

  

| Istilah | Penjelasan singkat |

|---|---|

| Network adapter | Perangkat/interface yang menghubungkan server ke jaringan |

| IPv4 / IPv6 | Alamat logis perangkat pada jaringan |

| IP address | Alamat unik host pada jaringan |

| Subnet mask / prefix | Menentukan batas jaringan lokal |

| Default gateway | Jalur keluar menuju jaringan lain |

| DNS server | Penerjemah nama host/domain menjadi IP |

| Protocol | Aturan komunikasi (TCP, UDP, ICMP) |

| Port | Titik akhir komunikasi pada sebuah host |

| Listening service | Proses yang menunggu koneksi masuk pada port tertentu |

| Inbound traffic | Traffic yang masuk menuju server |

| Outbound traffic | Traffic yang keluar dari server |

| Stateful firewall | Firewall yang mengingat status koneksi sehingga balasan dari koneksi yang diizinkan otomatis diperbolehkan |

  

### Analogi Gedung

  

```text

IP ADDRESS        = alamat gedung

PORT               = pintu masuk tertentu

SERVICE            = petugas yang berjaga di balik pintu

FIREWALL RULE      = aturan penjaga tentang siapa boleh lewat pintu mana

REMOTE IP SCOPE    = daftar tamu yang diizinkan masuk

```

  

Firewall yang baik tidak menutup semua pintu — firewall yang baik memastikan **pintu yang terbuka hanya dijaga oleh petugas yang tepat, untuk tamu yang tepat**.

  

---

  

## WGUI-4.2 GUI Tools Network Hardening

  

| Tool | Run Command | Fungsi Utama |

|---|---|---|

| Network Connections | `ncpa.cpl` | Melihat/mengatur adapter, IP, DNS, status koneksi |

| Windows Defender Firewall with Advanced Security | `wf.msc` | Mengelola inbound/outbound rule, connection security, logging |

| Windows Security | (Start Menu) | Melihat status firewall per profile secara ringkas |

| Network and Sharing Center | (Control Panel) | Melihat profile jaringan aktif dan status koneksi |

| Server Manager | (Start Menu) | Melihat role/feature yang terpasang dan status server |

| Services | `services.msc` | Melihat dan mengatur status service (running/stopped) |

| Event Viewer | `eventvwr.msc` | Membaca log sistem, keamanan, dan firewall |

| Computer Management | `compmgmt.msc` | Melihat service, event log, shared folder dalam satu console |

| Group Policy Management Console | `gpmc.msc` | Digunakan bila hardening jaringan diterapkan melalui GPO |

  

### Kapan digunakan

  

- Gunakan `ncpa.cpl` saat perlu memverifikasi adapter, IP, atau DNS sebelum melakukan perubahan apa pun.

- Gunakan `wf.msc` sebagai pusat kerja utama modul ini: audit rule, membuat rule, mengatur logging.

- Gunakan Windows Security untuk pengecekan cepat status firewall per profile tanpa membuka detail rule.

- Gunakan `services.msc` untuk memastikan service yang terkait rule firewall benar-benar berjalan atau memang sengaja dihentikan.

- Gunakan `eventvwr.msc` untuk membaca hasil firewall logging dan event keamanan terkait koneksi.

- Gunakan `compmgmt.msc` ketika ingin melihat service dan shared folder dalam satu layar saat audit SMB.

- Gunakan `gpmc.msc` hanya jika objective lomba secara eksplisit meminta penerapan melalui Group Policy.

  

---

  

## WGUI-4.3 Network Baseline

  

Sebelum melakukan **perubahan apa pun**, catat kondisi berikut melalui GUI (`ncpa.cpl` → Properties/Status, Windows Security, `wf.msc`):

  

- [ ] Nama adapter

- [ ] Status adapter (connected/disconnected)

- [ ] Alamat IPv4 dan IPv6

- [ ] Mode DHCP atau static

- [ ] Subnet mask/prefix

- [ ] Default gateway

- [ ] DNS server yang digunakan

- [ ] Network profile aktif (Domain/Private/Public)

- [ ] Status domain connectivity

- [ ] Port RDP yang digunakan

- [ ] Port lain yang sedang listening

- [ ] Service yang wajib tetap berjalan

- [ ] Status firewall saat ini (on/off per profile)

- [ ] Daftar allow rule dan block rule yang relevan

- [ ] Recovery path yang tersedia (console, snapshot, sesi kedua)

  

> Baseline bukan formalitas. Baseline adalah bukti kondisi awal sekaligus rujukan rollback.

  

**Larangan pada tahap ini:** jangan mengganti IP, DNS, atau gateway hanya karena "terlihat tidak standar". Perubahan pada tahap baseline hanya berupa pencatatan, bukan modifikasi.

  

---

  

## WGUI-4.4 Network Profile

  

| Profile | Karakteristik | Tingkat Rule Default |

|---|---|---|

| Domain | Aktif otomatis saat server terhubung ke domain controller dan dapat mengautentikasi | Lebih permisif untuk kebutuhan domain (AD, GPO, file sharing) |

| Private | Dipilih manual untuk jaringan yang dipercaya (bukan domain) | Menengah, tergantung rule yang dibuat administrator |

| Public | Default untuk jaringan tidak dikenal | Paling ketat secara default |

  

### Cara melihat profile aktif melalui GUI

  

1. Buka **Network and Sharing Center**, lihat label profile di bawah nama koneksi.

2. Atau buka **Windows Security → Firewall & network protection**, profile aktif ditandai "(active)".

3. Atau buka `wf.msc`, panel utama menampilkan status firewall untuk ketiga profile sekaligus.

  

### Risiko salah klasifikasi

  

- Server yang seharusnya berada di profile **Domain** tetapi terbaca sebagai **Public** akan kehilangan akses ke rule yang dibutuhkan untuk komunikasi domain.

- Menandai jaringan tidak dikenal sebagai **Private** memperluas exposure rule yang seharusnya hanya untuk jaringan tepercaya.

- **"Apply to all profiles"** pada sebuah rule berarti rule tersebut aktif di Domain, Private, maupun Public sekaligus — ini adalah bentuk overexposure jika rule tersebut sebenarnya hanya dibutuhkan pada satu profile saja.

  

> Prinsip: rule seharusnya seketat mungkin pada profile yang benar-benar relevan, bukan otomatis "all profiles".

  

---

  

## WGUI-4.5 Windows Defender Firewall Architecture

  

Struktur utama pada `wf.msc`:

  

| Komponen | Fungsi |

|---|---|

| Inbound Rules | Mengatur traffic yang masuk ke server |

| Outbound Rules | Mengatur traffic yang keluar dari server |

| Connection Security Rules | Mengatur autentikasi/enkripsi koneksi (IPsec) |

| Monitoring | Menampilkan rule aktif, firewall state, dan security association saat ini |

| Firewall Properties | Pengaturan default behavior dan logging per profile |

  

### Default behavior

  

- **Default inbound**: block, kecuali ada rule allow yang cocok.

- **Default outbound**: allow, kecuali ada rule block yang cocok.

  

### Prinsip precedence praktis

  

> Specific allow tidak mengalahkan explicit block yang cocok.

  

Artinya, jika ada rule block yang secara spesifik menolak traffic tertentu, rule allow yang juga cocok tidak otomatis membuat traffic tersebut lolos. Block eksplisit yang lebih spesifik tetap menang atas allow yang lebih umum dalam praktik evaluasi Windows Firewall.

  

### Atribut yang wajib dibaca saat audit rule

  

- [ ] Enabled status

- [ ] Action (Allow/Block)

- [ ] Profile (Domain/Private/Public)

- [ ] Program

- [ ] Service

- [ ] Protocol

- [ ] Local port

- [ ] Remote port

- [ ] Local IP

- [ ] Remote IP

- [ ] Interface type

- [ ] Edge traversal

  

Semua atribut ini dapat dilihat/diubah melalui tab **General, Programs and Services, Protocols and Ports, Scope, Advanced** pada Properties setiap rule di `wf.msc`.

  

---

  

## WGUI-4.6 Audit Inbound Rules

  

### Prosedur GUI

  

1. Buka `wf.msc` → **Inbound Rules**.

2. Tambahkan kolom Profile, Enabled, Action, Program melalui **View → Add/Remove Columns** agar audit lebih cepat.

3. Urutkan berdasarkan Enabled untuk melihat semua rule aktif terlebih dahulu.

4. Periksa setiap rule aktif satu per satu melalui Properties.

  

### Kategori yang diperiksa

  

| Kategori | Tanda bahaya |

|---|---|

| Allow Any | Action Allow tanpa pembatasan program/port/scope |

| Any program | Program diset ke "All programs" |

| Any port | Local port diset ke "All Ports" |

| Any remote IP | Remote IP scope diset ke "Any IP address" |

| All profiles | Rule aktif di Domain, Private, dan Public sekaligus tanpa alasan jelas |

| Duplicate rule | Dua rule atau lebih dengan fungsi yang sama |

| Disabled rule | Perlu dicatat, tidak perlu diubah kecuali relevan dengan objective |

| Rule dengan nama aneh | Nama tidak sesuai konvensi Windows, mencurigakan tapi belum tentu malicious |

| Rule aplikasi tidak digunakan | Program yang sudah tidak terpasang di server |

| Rule bawaan Windows | Predefined rule group dari Windows/Server roles |

| Custom rule | Dibuat manual, wajib diverifikasi tujuannya |

  

### Aturan aman

  

- Jangan menghapus built-in rule secara sembarangan; gunakan **Disable** terlebih dahulu.

- Disable lebih aman daripada delete karena masih dapat dikembalikan tanpa membuat ulang rule.

- Catat rule (nama, action, profile, port) sebelum mengubahnya sebagai bagian dari evidence.

- Periksa service dependency melalui `services.msc` sebelum menonaktifkan rule yang terkait sebuah service.

  

---

  

## WGUI-4.7 Audit Outbound Rules

  

Secara default, Windows Firewall jauh lebih permisif untuk outbound dibanding inbound. Hal ini memudahkan operasional tetapi membuka risiko tertentu.

  

### Risiko

  

- Outbound yang terlalu terbuka dapat menjadi jalur **command-and-control** bila ada proses berbahaya pada server.

- Sebaliknya, memblokir outbound secara serampangan berisiko memutus **Windows Update, resolusi DNS, autentikasi domain**, atau aplikasi sah lain.

  

### Kapan outbound restriction diperlukan

  

- Ketika objective lomba secara eksplisit meminta pembatasan outbound.

- Ketika ada proses/service yang diketahui tidak sah dan perlu diputus komunikasinya.

  

### Mengapa jangan langsung default outbound block

  

Mengubah default outbound menjadi block pada seluruh server akan memutus banyak fungsi dasar sistem operasi sekaligus (update, DNS, NTP, autentikasi) dan sangat sulit dipulihkan dalam kondisi closed-book tanpa dokumentasi lengkap semua dependency.

  

### Metode aman

  

Gunakan pembatasan **terbatas per program atau service**:

  

1. Identifikasi program/service yang ingin dibatasi.

2. Buat outbound rule Block khusus untuk program tersebut saja.

3. Jangan mengubah default outbound behavior secara global.

4. Verifikasi bahwa fungsi lain tetap berjalan setelah rule diterapkan.

  

---

  

## WGUI-4.8 Membuat Firewall Rule Aman

  

### Empat jenis dasar rule pada New Inbound/Outbound Rule Wizard

  

| Jenis | Kapan digunakan |

|---|---|

| Program | Ketika ingin mengizinkan/memblokir berdasarkan file executable tertentu |

| Port | Ketika ingin mengatur berdasarkan port dan protokol tertentu, tanpa terikat program |

| Predefined | Ketika mengatur fitur Windows/role yang sudah punya rule group siap pakai (contoh: Remote Desktop, File and Printer Sharing) |

| Custom | Ketika kombinasi program, port, service, dan scope perlu diatur sekaligus secara spesifik |

  

### Formula rule aman

  

```text

ALLOW ONLY:

REQUIRED SERVICE

+ REQUIRED PORT

+ REQUIRED PROFILE

+ REQUIRED REMOTE IP

```

  

### Remote IP Scope

  

| Scope | Penjelasan |

|---|---|

| Any IP | Semua alamat IP diizinkan — exposure paling luas |

| Local subnet | Hanya IP pada subnet yang sama dengan server |

| Specific IP | Hanya satu alamat IP tertentu |

| Specific subnet | Hanya rentang subnet tertentu |

  

> Remote IP scope adalah salah satu cara **paling efektif** mengurangi exposure tanpa mengubah service atau port sama sekali. Rule yang sudah benar dari sisi port dan program tetap berisiko tinggi jika scope-nya "Any IP" padahal akses seharusnya terbatas.

  

Langkah umum (Scope tab pada rule Properties):

  

1. Buka rule → tab **Scope**.

2. Pada bagian **Remote IP address**, pilih "These IP addresses".

3. Tambahkan IP/subnet yang sah sesuai objective.

4. Simpan dan verifikasi dari sesi/koneksi kedua.

  

---

  

## WGUI-4.9 RDP Hardening

  

### Fakta dasar

  

| Aspek | Keterangan |

|---|---|

| Default port | TCP 3389 |

| UDP RDP awareness | RDP modern juga dapat menggunakan UDP 3389 untuk performa; tetap perhatikan saat audit |

| Network Level Authentication (NLA) | Mengharuskan autentikasi sebelum sesi desktop penuh terbentuk — kurangi exposure terhadap percobaan koneksi anonim |

| Authorized user/group | Hanya user/group tertentu yang boleh terhubung via RDP (diatur di System Properties → Remote, atau Local/Group Policy) |

| Firewall rule | Predefined rule group "Remote Desktop" |

| Remote IP scope | Dapat dibatasi ke subnet/IP tertentu |

| Profile | Perhatikan rule aktif di profile mana (Domain/Private/Public) |

| Sesi baru sebagai verifikasi | Perubahan RDP wajib diverifikasi dari sesi/koneksi baru, bukan sesi yang sudah terhubung |

  

### Prosedur anti-lockout wajib sebelum mengubah firewall/rule RDP

  

```text

CHECK CURRENT RDP SESSION

        ↓

IDENTIFY RDP PORT

        ↓

IDENTIFY ACTIVE FIREWALL PROFILE

        ↓

VERIFY ALLOW RULE

        ↓

PREPARE ROLLBACK

        ↓

CHANGE ONE THING

        ↓

TEST FROM SECOND SESSION

```

  

### Larangan

  

- Jangan memblokir RDP sebelum console access (VirtualBox console, snapshot, out-of-band access) tersedia.

- Jangan menghapus seluruh rule group Remote Desktop; gunakan disable pada bagian spesifik bila perlu.

- Jangan hanya mengandalkan perubahan port sebagai hardening — perubahan port tanpa NLA dan scope yang benar tidak memberi perlindungan berarti.

- Jangan membuka RDP ke Any IP jika objective lomba meminta pembatasan.

- Sesi RDP yang sedang aktif **bukan bukti** bahwa koneksi baru tetap bisa dilakukan setelah perubahan — koneksi lama tetap "menempel" meski rule baru sudah lebih ketat.

  

---

  

## WGUI-4.10 SMB dan File Sharing Hardening

  

### Fakta dasar

  

| Aspek | Keterangan |

|---|---|

| SMB | TCP 445 |

| NetBIOS ports awareness | TCP/UDP 137-139, masih relevan pada sebagian environment lama |

| File and Printer Sharing | Predefined rule group untuk SMB, NetBIOS, spooler |

| SMBv1 awareness | Versi lama dan rentan; periksa apakah masih diperlukan sebelum mengambil keputusan |

| Share permission | Izin pada level shared folder |

| NTFS permission | Izin pada level file system |

| Firewall scope | Membatasi siapa yang boleh mengakses port SMB dari jaringan |

| Domain/administrative shares | Beberapa share (contoh administrative share) dibutuhkan oleh proses domain tertentu — jangan dihapus tanpa memastikan dampaknya |

  

### Hubungan akses efektif

  

```text

SHARE PERMISSION

        +

NTFS PERMISSION

        +

FIREWALL ACCESS

        =

EFFECTIVE FILE ACCESS

```

  

Ketiganya harus benar secara bersamaan. Memperbaiki firewall saja tidak akan membuka akses jika share permission atau NTFS permission masih membatasi, dan sebaliknya.

  

### Risiko menonaktifkan SMB secara buta

  

Menonaktifkan seluruh rule File and Printer Sharing tanpa memeriksa dependency dapat memutus proses domain tertentu, printer bersama, atau kebutuhan operasional lain yang justru menjadi bagian dari objective lomba (misalnya file server yang harus tetap berfungsi).

  

---

  

## WGUI-4.11 Network Discovery dan Remote Management

  

| Fitur | Fungsi | Rule Group Terkait |

|---|---|---|

| Network Discovery | Melihat perangkat lain di jaringan lokal | Network Discovery |

| File and Printer Sharing | Berbagi file/printer | File and Printer Sharing |

| Remote Service Management | Mengelola service dari komputer lain | Remote Service Management |

| Remote Event Log Management | Membaca event log dari komputer lain | Remote Event Log Management |

| WMI | Query manajemen jarak jauh berbasis WMI | Windows Management Instrumentation (WMI) |

| WinRM | Manajemen jarak jauh berbasis web services | Windows Remote Management |

| RPC | Komunikasi antar proses, dasar banyak layanan manajemen jarak jauh | Terkait beberapa rule group di atas |

| ICMP Echo | Ping request/reply | File and Printer Sharing (Echo Request) |

  

### Cara kerja audit

  

1. Periksa apakah fitur benar-benar dibutuhkan sesuai objective (contoh: apakah server memang harus bisa di-manage dari komputer lain via WinRM).

2. Jika dibutuhkan, batasi rule group tersebut hanya ke profile yang relevan (biasanya Domain atau Private, bukan Public).

3. Batasi remote scope ke subnet/IP administratif saja.

4. Jika tidak dibutuhkan, disable group rule tersebut secara hati-hati, satu group pada satu waktu, lalu verifikasi dampaknya.

  

---

  

## WGUI-4.12 DNS dan Domain Connectivity Safety

  

### Prinsip dasar

  

- Domain member harus menggunakan **DNS internal domain**, bukan DNS publik/eksternal langsung pada adapter yang sama dengan domain.

- Mengarahkan adapter domain langsung ke DNS eksternal dapat merusak pencarian resource domain (SRV record, domain controller, dsb).

- **Kerberos** dan Active Directory sangat bergantung pada resolusi DNS yang benar.

- Jangan mengganti DNS hanya karena internet terasa tidak berjalan — periksa dulu penyebab sebenarnya.

- Periksa DNS suffix, DNS server yang terpasang, dan connectivity sebelum melakukan perubahan apa pun pada pengaturan DNS.

  

### Troubleshooting berurutan

  

```text

CHECK ADAPTER

   ↓

CHECK IP

   ↓

CHECK GATEWAY

   ↓

CHECK DNS

   ↓

CHECK NAME RESOLUTION

   ↓

CHECK DOMAIN PROFILE

   ↓

CHECK FIREWALL

   ↓

CHECK SERVICE

```

  

Setiap tahap diperiksa melalui GUI (`ncpa.cpl` untuk adapter/IP/gateway/DNS, Network and Sharing Center untuk profile) dan CLI verifikasi (`ipconfig /all`, `nslookup`) hanya sebagai konfirmasi, bukan cara mengubah konfigurasi.

  

---

  

## WGUI-4.13 Firewall Logging dan Monitoring

  

### Langkah GUI mengaktifkan logging

  

1. Buka `wf.msc`.

2. Klik kanan **Windows Defender Firewall with Advanced Security on Local Computer** → **Properties**.

3. Pilih tab profile yang relevan (Domain/Private/Public).

4. Pada bagian **Logging**, klik **Customize**.

5. Atur:

   - **Log dropped packets**: Yes

   - **Log successful connections**: Yes (bila dibutuhkan untuk evidence, perhatikan ukuran file)

   - **Name**: lokasi file log (default `%systemroot%\system32\LogFiles\Firewall\pfirewall.log`)

   - **Size limit (KB)**: sesuaikan agar tidak terlalu cepat penuh

6. Ulangi untuk profile lain yang relevan.

  

### Fungsi logging

  

| Fungsi | Penjelasan |

|---|---|

| Evidence | Bukti bahwa perubahan rule benar-benar berdampak pada traffic |

| Troubleshooting | Melihat traffic yang ditolak untuk memahami penyebab kegagalan koneksi |

| Identifikasi scanning | Pola banyak port dicoba dalam waktu singkat dari satu sumber |

| Mendeteksi koneksi diblokir | Konfirmasi bahwa rule block bekerja sesuai harapan |

| Validasi rule | Memastikan rule allow/block baru benar-benar diterapkan sistem |

  

> Pembahasan SIEM dan korelasi log tingkat lanjut akan diperdalam pada WGUI-6. Modul ini hanya menekankan logging dasar sebagai evidence.

  

---

  

## WGUI-4.14 Verification, Evidence, dan Rollback

  

### Alur setiap perubahan

  

```text

BEFORE SCREENSHOT

      ↓

CHANGE

      ↓

NEW CONNECTION TEST

      ↓

CHECK LOG

      ↓

AFTER SCREENSHOT

```

  

### Evidence minimal per perubahan

  

- [ ] Firewall profile aktif saat perubahan dilakukan

- [ ] Rule sebelum perubahan (screenshot/catatan)

- [ ] Rule setelah perubahan (screenshot/catatan)

- [ ] Port/service yang relevan dengan perubahan

- [ ] Hasil koneksi (berhasil/gagal) sesuai objective

- [ ] Firewall log jika relevan dengan perubahan tersebut

  

### Rollback

  

| Item | Cara rollback |

|---|---|

| Rule yang di-disable | Re-enable rule |

| Scope yang diubah | Restore scope ke kondisi awal |

| Profile yang diubah | Restore ke profile sebelumnya |

| IP/DNS yang diubah | Restore ke IP/DNS baseline |

| Service yang dihentikan | Restore service state (start/automatic sesuai baseline) |

| Kumpulan rule kompleks | Gunakan backup/export policy (`wf.msc` → Export Policy) sebelum perubahan besar, import kembali bila perlu |

  

> Rollback hanya efektif jika baseline (WGUI-4.3) dicatat dengan lengkap sejak awal.

  

---

  

## WGUI-4.15 Troubleshooting Decision Tree

  

Urutan pemeriksaan utama untuk seluruh kasus jaringan:

  

```text

PHYSICAL/ADAPTER

   ↓

IP CONFIGURATION

   ↓

ROUTE/GATEWAY

   ↓

DNS

   ↓

NETWORK PROFILE

   ↓

SERVICE STATUS

   ↓

LISTENING PORT

   ↓

FIREWALL RULE

   ↓

REMOTE SCOPE

   ↓

LOG

```

  

### Penerapan pada kasus umum

  

| Kasus | Fokus pemeriksaan |

|---|---|

| RDP tidak bisa terhubung | Service status Remote Desktop Services → listening port 3389 → firewall rule Remote Desktop → remote scope → firewall log |

| Server tidak bisa mengakses internet | IP configuration → gateway → DNS → outbound rule yang mungkin memblokir |

| Domain tidak terdeteksi | DNS → network profile → firewall rule terkait AD/Kerberos |

| DNS tidak resolve | DNS server terpasang → name resolution → firewall (port 53) |

| File sharing gagal | Service Server (LanmanServer) → listening port 445 → firewall rule File and Printer Sharing → share/NTFS permission |

| Ping gagal | Firewall rule ICMP (Echo Request) → profile aktif → bukan berarti service lain juga bermasalah |

| Service listening tetapi tetap tidak bisa diakses | Firewall rule untuk port tersebut → remote scope → profile aktif saat koneksi dicoba |

  

---

  

## WGUI-4.16 Closed-Book Training

  

Format jawaban wajib setiap skenario:

  

- Apa yang dicek?

- Risiko?

- Perubahan aman?

- Verifikasi?

- Rollback?

- Evidence?

  

### Skenario 1 — RDP terbuka untuk Any IP

  

- **Apa yang dicek?** Rule Remote Desktop pada `wf.msc`, tab Scope, Remote IP address.

- **Risiko?** Siapa pun dari jaringan mana pun dapat mencoba terhubung ke RDP.

- **Perubahan aman?** Ubah Remote IP address menjadi subnet/IP administratif yang sah, satu perubahan pada satu waktu.

- **Verifikasi?** Coba koneksi RDP baru dari IP yang diizinkan dan dari IP di luar scope (jika memungkinkan) untuk memastikan hasil sesuai harapan.

- **Rollback?** Kembalikan scope ke "Any IP" bila perubahan menyebabkan lockout dari sesi administratif yang sah.

- **Evidence?** Screenshot scope sebelum/sesudah, hasil test koneksi.

  

### Skenario 2 — Firewall dimatikan pada Domain Profile

  

- **Apa yang dicek?** Status firewall per profile di `wf.msc` atau Windows Security.

- **Risiko?** Seluruh traffic pada profile Domain tidak lagi difilter, exposure penuh.

- **Perubahan aman?** Aktifkan kembali firewall pada Domain Profile setelah memastikan rule allow yang dibutuhkan (termasuk RDP) sudah benar dan aktif.

- **Verifikasi?** Test koneksi RDP baru dan konektivitas domain setelah firewall diaktifkan kembali.

- **Rollback?** Jika mengaktifkan firewall menyebabkan service penting gagal diakses, periksa rule yang hilang, bukan mematikan kembali firewall.

- **Evidence?** Screenshot status firewall sebelum/sesudah, hasil verifikasi RDP dan domain.

  

### Skenario 3 — Rule custom mencurigakan membuka semua port

  

- **Apa yang dicek?** Detail rule: program, port, protocol, scope, profile, siapa yang membuatnya jika ada catatan.

- **Risiko?** Exposure sangat luas, berpotensi disalahgunakan.

- **Perubahan aman?** Disable rule terlebih dahulu (bukan langsung delete), catat detail lengkap sebagai evidence.

- **Verifikasi?** Pastikan service yang sah tidak bergantung pada rule tersebut sebelum benar-benar menonaktifkannya secara permanen.

- **Rollback?** Re-enable rule jika ternyata dibutuhkan oleh proses sah yang belum teridentifikasi.

- **Evidence?** Screenshot detail rule (semua tab) sebelum diubah.

  

### Skenario 4 — SMB tidak dapat diakses setelah hardening

  

- **Apa yang dicek?** Status service Server (LanmanServer), rule File and Printer Sharing, share permission, NTFS permission.

- **Risiko?** File sharing yang dibutuhkan operasional/objective menjadi tidak berfungsi.

- **Perubahan aman?** Aktifkan kembali rule spesifik yang diperlukan (bukan seluruh group jika tidak semuanya relevan), sesuaikan scope agar tetap terbatas.

- **Verifikasi?** Akses share dari komputer/sesi lain setelah rule diaktifkan kembali.

- **Rollback?** Kembalikan ke kondisi rule sebelum hardening jika akses tetap gagal setelah rule diaktifkan.

- **Evidence?** Screenshot rule sebelum/sesudah, hasil akses share.

  

### Skenario 5 — Server gagal bergabung/berkomunikasi dengan domain

  

- **Apa yang dicek?** Konfigurasi DNS pada adapter, network profile aktif, firewall rule terkait AD/Kerberos.

- **Risiko?** Domain authentication failure, GPO tidak diterapkan, replikasi AD terganggu.

- **Perubahan aman?** Perbaiki DNS server ke DNS internal domain terlebih dahulu sebelum menyentuh firewall.

- **Verifikasi?** Uji resolusi nama domain dan komunikasi ke domain controller setelah perbaikan.

- **Rollback?** Kembalikan DNS ke kondisi baseline bila perbaikan tidak menyelesaikan masalah, lanjutkan investigasi pada lapisan lain.

- **Evidence?** Screenshot pengaturan DNS sebelum/sesudah, hasil name resolution.

  

### Skenario 6 — Service listening tetapi koneksi tetap gagal

  

- **Apa yang dicek?** Firewall rule untuk port terkait, remote scope, profile aktif saat koneksi dicoba.

- **Risiko?** Kesalahan diagnosis (dikira service mati padahal service berjalan normal).

- **Perubahan aman?** Sesuaikan scope/profile rule agar mencakup sumber koneksi yang sah, tanpa membuka ke Any IP.

- **Verifikasi?** Ulangi test koneksi dari sesi/mesin sumber setelah rule disesuaikan.

- **Rollback?** Kembalikan scope/profile ke kondisi sebelumnya jika perubahan tidak menyelesaikan masalah.

- **Evidence?** Screenshot rule dan hasil test sebelum/sesudah.

  

### Skenario 7 — Public Profile memiliki terlalu banyak allow rule

  

- **Apa yang dicek?** Daftar rule aktif pada profile Public di `wf.msc`.

- **Risiko?** Jaringan yang seharusnya tidak dipercaya justru memiliki banyak akses masuk.

- **Perubahan aman?** Disable rule yang tidak seharusnya aktif di Public, pertahankan hanya yang benar-benar diperlukan.

- **Verifikasi?** Pastikan fungsi yang memang dibutuhkan pada jaringan publik (jika ada) tetap berjalan.

- **Rollback?** Re-enable rule spesifik bila ternyata dibutuhkan.

- **Evidence?** Daftar rule sebelum/sesudah pada profile Public.

  

### Skenario 8 — Outbound rule memblokir aplikasi penting

  

- **Apa yang dicek?** Rule outbound yang baru dibuat/diubah, program/service yang terdampak.

- **Risiko?** Aplikasi/service sah tidak dapat berkomunikasi keluar, termasuk kemungkinan update atau autentikasi.

- **Perubahan aman?** Persempit rule block outbound agar hanya menyasar program/service yang benar-benar dimaksud, bukan pada level yang lebih luas.

- **Verifikasi?** Pastikan aplikasi penting kembali berfungsi setelah rule disesuaikan.

- **Rollback?** Disable rule outbound yang bermasalah jika penyesuaian scope tidak cukup.

- **Evidence?** Screenshot rule outbound sebelum/sesudah, status aplikasi terdampak.

  

---

  

## WGUI-4.17 GUI Practical Lab

  

### LAB 1 — Audit Firewall Profile

  

- **Scenario**: Server belum pernah diaudit status firewallnya secara menyeluruh.

- **Objective**: Memastikan status Domain, Private, dan Public Profile diketahui dan terdokumentasi.

- **Safety Check**: Pastikan sesi RDP aktif sebelum memulai, siapkan akses console cadangan.

- **GUI Steps**:

  1. Buka `wf.msc`.

  2. Pada halaman utama, catat status (On/Off) untuk ketiga profile.

  3. Klik kanan root node → Properties untuk melihat detail setting tiap profile (inbound/outbound default, logging).

  4. Catat semuanya sebagai baseline.

- **Verification**: Bandingkan hasil catatan dengan tampilan Windows Security untuk memastikan konsistensi.

- **Rollback**: Tidak ada perubahan pada lab ini, hanya audit.

- **Evidence**: Screenshot halaman utama `wf.msc` dan Properties tiap profile.

  

### LAB 2 — Audit dan Persempit RDP Rule

  

- **Scenario**: Rule Remote Desktop saat ini terbuka untuk Any IP pada semua profile.

- **Objective**: Membatasi RDP ke profile dan remote IP tertentu tanpa menyebabkan lockout.

- **Safety Check**: Ikuti prosedur anti-lockout WGUI-4.9 secara penuh sebelum mengubah apa pun; pastikan sesi RDP aktif tetap terbuka selama proses, dan siapkan akses console cadangan.

- **GUI Steps**:

  1. Buka `wf.msc` → Inbound Rules, cari grup "Remote Desktop".

  2. Buka Properties rule yang enabled, catat kondisi awal (profile, scope).

  3. Pada tab **Scope**, ubah Remote IP address menjadi subnet/IP administratif yang ditentukan objective.

  4. Pada tab **Advanced**, pastikan hanya profile yang relevan yang dicentang.

  5. Simpan perubahan.

- **Verification**: Buka sesi RDP **baru** (bukan sesi lama) dari IP yang diizinkan untuk memastikan koneksi tetap berhasil.

- **Rollback**: Jika sesi baru gagal terhubung, kembalikan Remote IP address ke kondisi awal menggunakan sesi lama yang masih terbuka.

- **Evidence**: Screenshot Scope sebelum/sesudah, hasil koneksi sesi baru.

  

### LAB 3 — Membuat Custom Inbound Rule

  

- **Scenario**: Dibutuhkan satu port test untuk keperluan verifikasi layanan tertentu sesuai objective lomba.

- **Objective**: Membuat inbound rule untuk satu port test, dengan profile dan remote subnet tertentu.

- **Safety Check**: Pastikan port dan tujuan rule benar-benar sesuai objective, bukan port acak.

- **GUI Steps**:

  1. Buka `wf.msc` → Inbound Rules → **New Rule**.

  2. Pilih jenis **Port**.

  3. Pilih protokol (TCP/UDP) dan masukkan nomor port sesuai objective.

  4. Pilih **Allow the connection**.

  5. Centang hanya profile yang relevan.

  6. Beri nama rule yang jelas dan deskriptif.

  7. Buka Properties rule yang baru dibuat → tab Scope → atur Remote IP address ke subnet yang ditentukan.

- **Verification**: Test koneksi ke port tersebut dari IP dalam scope dan (bila memungkinkan) dari IP di luar scope.

- **Rollback**: Disable atau hapus rule yang baru dibuat bila ternyata tidak sesuai objective.

- **Evidence**: Screenshot seluruh tab rule, hasil test koneksi.

  

### LAB 4 — Audit File and Printer Sharing

  

- **Scenario**: Server memiliki beberapa rule File and Printer Sharing yang aktif tanpa kejelasan kebutuhan.

- **Objective**: Memastikan hanya rule yang benar-benar dibutuhkan yang aktif.

- **Safety Check**: Periksa service Server (LanmanServer) dan kebutuhan share sebelum menonaktifkan rule apa pun.

- **GUI Steps**:

  1. Buka `wf.msc` → Inbound Rules, filter/cari grup "File and Printer Sharing".

  2. Catat seluruh rule pada grup ini beserta status enabled dan profile-nya.

  3. Bandingkan dengan kebutuhan aktual (apakah share benar-benar digunakan, apakah printer sharing diperlukan).

  4. Disable rule spesifik yang tidak diperlukan, satu per satu.

- **Verification**: Setelah setiap disable, uji akses share yang seharusnya tetap berfungsi dari komputer/sesi lain.

- **Rollback**: Re-enable rule spesifik bila akses yang seharusnya berfungsi justru terganggu.

- **Evidence**: Daftar rule sebelum/sesudah, hasil test akses share.

  

### LAB 5 — Firewall Logging

  

- **Scenario**: Belum ada catatan dropped packet yang dapat digunakan sebagai evidence audit.

- **Objective**: Mengaktifkan dropped packet logging dan menggunakan hasilnya sebagai evidence.

- **Safety Check**: Pastikan lokasi dan ukuran file log wajar agar tidak memenuhi disk.

- **GUI Steps**:

  1. Buka `wf.msc` → klik kanan root node → **Properties**.

  2. Pilih tab profile yang relevan (misalnya Domain Profile).

  3. Pada bagian Logging, klik **Customize**.

  4. Set **Log dropped packets** ke **Yes**.

  5. Catat lokasi file log yang ditampilkan.

  6. Ulangi untuk profile lain sesuai kebutuhan.

  7. Coba satu koneksi yang seharusnya diblokir (sesuai objective) untuk menghasilkan entry log.

- **Verification**: Buka file log (melalui Notepad atau Event Viewer bila diarahkan ke sana) dan pastikan entry dropped packet muncul sesuai percobaan koneksi.

- **Rollback**: Set kembali logging ke pengaturan awal bila diperlukan (misalnya jika ukuran log menjadi masalah).

- **Evidence**: Screenshot pengaturan logging dan cuplikan isi file log yang menunjukkan dropped packet.

  

---

  

## MEMORY SECTION — NETWORK COMBAT MEMORY

  

```text

ADAPTER

  ↓

IP

  ↓

GATEWAY

  ↓

DNS

  ↓

PROFILE

  ↓

SERVICE

  ↓

PORT

  ↓

RULE

  ↓

SCOPE

  ↓

VERIFY

```

  

### Formula Akses Jaringan

  

```text

NETWORK ACCESS =

SERVICE RUNNING

+ PORT LISTENING

+ FIREWALL ALLOW

+ CORRECT SCOPE

+ CORRECT ROUTE

```

  

### 10 Langkah Tempur

  

1. Cek sesi RDP aktif sebelum menyentuh apa pun.

2. Catat baseline adapter, IP, gateway, DNS.

3. Kenali profile jaringan aktif saat ini.

4. Audit inbound rule: cari Allow Any, Any port, Any IP, All profiles.

5. Audit outbound rule hanya jika objective memintanya.

6. Hubungkan setiap rule mencurigakan ke service dan process nyatanya.

7. Persempit scope sebelum mempersempit port atau menutup rule.

8. Ubah satu hal dalam satu waktu.

9. Verifikasi selalu dari sesi/koneksi baru, bukan sesi lama.

10. Simpan evidence sebelum dan sesudah, siapkan rollback setiap saat.

  

---

  

## RINGKASAN VERIFIKASI MODUL

  

- [x] GUI-first

- [x] Anti-lockout tersedia

- [x] Network baseline tersedia

- [x] Firewall profiles dibahas

- [x] Inbound dan outbound rules dibahas

- [x] RDP hardening tersedia

- [x] SMB hardening tersedia

- [x] DNS/domain safety tersedia

- [x] Firewall logging tersedia

- [x] Verification tersedia

- [x] Rollback tersedia

- [x] Evidence tersedia

- [x] Closed-book training tersedia

- [x] Practical lab tersedia

- [x] Memory section tersedia

  

**STATUS: FINAL COMPETITION READY**