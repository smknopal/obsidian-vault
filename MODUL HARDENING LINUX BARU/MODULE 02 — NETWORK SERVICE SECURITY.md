# MODULE 02 — NETWORK SERVICE SECURITY

## LKS Cyber Security Field Manual — Linux Hardening

  

> **Mode:** Field Manual (Competition-Ready) · **Peran:** Defender · **OS:** Debian/Ubuntu-based

> **Topik Kisi-kisi:** Infrastructure Hardening → Linux → Network Service Security

> **Prasyarat:** Module 01 (Baseline & Recon) sudah selesai — snapshot dan evidence port/service sudah ada sebagai pembanding before/after.

> **Kompatibilitas:** SSH (OpenSSH), UFW, dan Fail2ban stabil di Ubuntu 22.04, 24.04, dan 26.04 — tidak ada perbedaan command. Catatan: 24.04+ memakai backend nftables di balik `ufw`, tapi seluruh perintah `ufw` di modul ini tidak berubah.

  

---

  

## 0. QUICK START — 15 MENIT PERTAMA

  

> Kerjakan Level 1 (Critical, lihat Bagian 3.1) untuk SEMUA komponen dulu sebelum masuk ke Level 2/3 di satu komponen — jangan habiskan waktu menyempurnakan SSH sementara UFW/Fail2ban belum tersentuh sama sekali.

  

```

[ ] 1. Generate key pair di client                         → 4.1 Langkah 1

[ ] 2. Copy public key ke server                            → 4.1 Langkah 2

[ ] 3. Test login pakai key di TERMINAL BARU (jangan tutup yang lama) → 4.1 Urutan Eksekusi

[ ] 4. PasswordAuthentication no + PermitRootLogin no        → 4.1 Langkah 3

[ ] 5. sudo sshd -t (wajib tanpa error) lalu restart sshd    → 4.1 VERIFY

[ ] 6. ufw default deny incoming/allow outgoing              → 4.2

[ ] 7. ufw allow <port SSH> SEBELUM ufw enable                → 4.2

[ ] 8. ufw enable                                             → 4.2

[ ] 9. Install fail2ban, copy jail.conf → jail.local          → 4.3

[ ] 10. Set port jail sshd = port SSH aktif, restart fail2ban → 4.3

[ ] 11. Verifikasi dari sesi terpisah: key login berhasil,   → 5

        password login ditolak, ufw status aktif,

        fail2ban-client status sshd jalan

```

  

**Kalau ada langkah gagal di tengah:** jangan lanjut ke langkah berikutnya — lihat SSH Recovery Procedure (4.5) atau Firewall Recovery Procedure (4.6) sesuai gejala.

  

---

  

## 1. Objective

  

**Tujuan modul:** mengamankan jalur komunikasi jaringan — terutama SSH sebagai pintu masuk utama — dari akses tidak sah via password lemah, serangan brute-force, dan port terbuka yang tidak perlu.

  

**Kaitan dengan kompetisi:** SSH, UFW, dan Fail2ban adalah tiga item yang hampir selalu masuk rubrik penilaian hardening jaringan LKS. Juri mengecek konfigurasi aktual (`sshd_config`, `ufw status`, `jail.local`) dan menguji perilaku sistem (percobaan login gagal → apakah benar ter-block). Modul ini menuntut eksekusi presisi karena satu kesalahan urutan (mis. mematikan password auth sebelum key terpasang) bisa mengunci peserta sendiri dari server.

  

**Risiko jika tidak dilakukan:**

- SSH dengan password default/lemah → brute-force berhasil dalam hitungan menit oleh scanner otomatis.

- Tanpa firewall (default allow) → seluruh port pada sistem terekspos ke jaringan lomba/internet.

- Tanpa rate-limiting/Fail2ban → serangan brute-force berulang tidak pernah dihentikan otomatis, membebani log dan berpotensi berhasil di percobaan ke-1000.

  

---

  

## 2. Concept Foundation

  

Analogi rumah untuk tiga lapisan pertahanan modul ini:

  

```

SSH       = pintu utama rumah

UFW       = pagar + satpam di depan gerbang

Fail2ban  = CCTV yang otomatis mengunci tamu mencurigakan

```

  

**Hubungan antar komponen:**

  

```

Internet ──▶ [UFW: pagar]           deny incoming by default,

                  │                  hanya port yang diizinkan lewat

                  ▼

            [SSH: pintu utama]       autentikasi berbasis key,

                  │                  bukan password

                  ▼

            [Fail2ban: CCTV]         memantau /var/log/auth.log,

                                     auto-block IP yang gagal login berkali-kali

```

  

**Kenapa key lebih kuat dari password:**

  

```

TANPA KEY (berbahaya):

Attacker → coba "admin123" → coba "password" → coba "root123" → ...

Server hanya menolak jika Fail2ban aktif (lapisan kedua)

  

DENGAN KEY (aman):

Attacker → tidak punya private key → LANGSUNG DITOLAK (lapisan pertama sudah menutup jalur)

```

  

SSH Key Auth memakai pasangan kunci asymmetric: **Private Key** disimpan di client (analogi: kunci gembok — rahasia mutlak), **Public Key** dipasang di server (analogi: gembok — boleh diketahui umum). Anda memasang gembok di server; hanya Anda yang punya kuncinya.

  

---

  

## 3. Competition Scenario

  

> **Kondisi lomba:** Server diberikan ke peserta dengan SSH masih memakai password authentication di port default 22, tanpa firewall aktif, dan tanpa mekanisme anti-brute-force. Panitia akan mensimulasikan serangan brute-force ke SSH sebagai bagian penilaian, dan peserta lain dalam tim mungkin perlu tetap bisa masuk selama proses hardening berlangsung.

  

**Cara berpikir defender:**

  

1. **Urutan eksekusi menentukan hidup-mati akses.** Defender yang terburu-buru mematikan password authentication sebelum public key terpasang dan teruji akan mengunci diri sendiri dari server — di lomba, ini berarti kehilangan waktu berharga untuk recovery via console/hypervisor.

2. **Uji di jalur baru, jangan tutup jalur lama dulu.** Selalu buka terminal/sesi baru untuk menguji perubahan sebelum menutup sesi yang sedang berjalan.

3. **Firewall dan SSH harus sinkron.** Mengaktifkan UFW dengan default-deny sebelum meng-allow port SSH yang sedang dipakai = mengunci diri sendiri dari sisi jaringan, sama fatalnya dengan kesalahan di poin 1.

4. **Lapisan pertahanan itu berlapis, bukan pengganti satu sama lain.** Fail2ban bukan pengganti disabling password auth; UFW bukan pengganti hardening `sshd_config`. Ketiganya dinilai terpisah oleh juri.

  

---

  

### 3.1 Hardening Priority Level

  

> Kalau waktu lomba habis di tengah jalan, item LEVEL 1 di semua komponen (SSH+UFW+Fail2ban) harus sudah selesai lebih dulu — baru kejar LEVEL 2, baru LEVEL 3. Jangan sempurnakan satu komponen ke Level 3 sementara komponen lain belum Level 1.

  

**LEVEL 1 — CRITICAL (WAJIB, urutan sesuai anti-lockout di atas)**

- SSH: `PermitRootLogin no`, `PasswordAuthentication no` (setelah key teruji), `PermitEmptyPasswords no`

- UFW: aktif dengan `default deny incoming`, port SSH ter-allow

- Fail2ban: `active`, jail `sshd` enabled

  

**LEVEL 2 — RECOMMENDED**

- SSH: `AllowUsers`/`AllowGroups`, `ClientAliveInterval`+`ClientAliveCountMax` (session timeout), `X11Forwarding no`, `AllowTcpForwarding no`, `MaxAuthTries 3`

- UFW: rate limiting (`ufw limit`) pada port SSH, deny eksplisit port berbahaya (21/23/3306/6379)

- Fail2ban: `bantime`/`findtime`/`maxretry` disesuaikan kebijakan tim, `ignoreip` diisi IP tim sendiri

  

**LEVEL 3 — OPTIONAL**

- SSH: port non-default (mis. 2222), banner login, `UseDNS no`, `LogLevel VERBOSE`

- UFW: allow scoped ke IP/subnet spesifik alih-alih global

- TCP Wrappers (`hosts.allow`/`hosts.deny`) — deprecated tapi masih sering muncul di soal lomba

  

---

  

## 4. Recon / Detection Phase

  

Sebelum mengubah apa pun, konfirmasi kondisi saat ini (data lebih lengkap sudah ada di evidence Module 01, bagian 4.9–4.10).

  

**CHECK — status SSH saat ini**

  

Command:

```bash

sudo systemctl status sshd

sudo sshd -T | grep -Ei 'passwordauthentication|permitrootlogin|port '

```

  

Expected output: service `active (running)`; baris konfigurasi efektif untuk password auth, root login, dan port.

  

Interpretasi: `sshd -T` menampilkan konfigurasi **efektif** (setelah semua include diproses), lebih dapat diandalkan daripada membaca `sshd_config` mentah yang mungkin punya baris ter-comment atau ter-override.

  

**CHECK — status UFW saat ini**

  

Command:

```bash

sudo ufw status verbose

```

  

Interpretasi: jika `Status: inactive`, seluruh port pada sistem saat ini terekspos tanpa filter — ini kondisi awal yang wajib dicatat sebagai baseline sebelum diaktifkan.

  

**CHECK — apakah Fail2ban sudah terpasang**

  

Command:

```bash

dpkg -l | grep fail2ban

sudo systemctl status fail2ban 2>/dev/null

```

  

Interpretasi: kosong = belum terpasang, lanjut ke instalasi di bagian 4.4. Sudah terpasang tapi tidak `active` = kandidat konfigurasi ulang, bukan asumsi sudah aman.

  

---

  

### 4.1 HARDEN — SSH Public Key Authentication

  

**Langkah 1: Buat key pair di client**

  

```bash

ssh-keygen -t rsa -b 4096

```

`-t rsa` = algoritma RSA. `-b 4096` = panjang kunci 4096 bit (makin panjang makin aman). Tekan Enter untuk lokasi default; passphrase boleh diisi untuk keamanan tambahan.

  

Hasil: `~/.ssh/id_rsa` (private, **rahasia mutlak**) dan `~/.ssh/id_rsa.pub` (public, dipasang di server).

  

**Langkah 2: Kirim public key ke server**

  

Otomatis:

```bash

ssh-copy-id -i ~/.ssh/id_rsa.pub user@server_ip

```

  

Manual (jika `ssh-copy-id` tidak tersedia):

```bash

# Di server:

mkdir -p ~/.ssh

chmod 700 ~/.ssh

nano ~/.ssh/authorized_keys   # paste isi id_rsa.pub dari client

chmod 600 ~/.ssh/authorized_keys

```

  

**Langkah 3: Konfigurasi `/etc/ssh/sshd_config`**

  

```bash

sudo sshd -t          # test config SEBELUM edit apa pun, pastikan baseline valid

sudo nano /etc/ssh/sshd_config

```

  

Konfigurasi target (terapkan bertahap, uji tiap tahap penting):

  

```ini

# ========== PORT ==========

Port 2222

# Ganti dari default 22 — mengurangi noise scanner otomatis, bukan keamanan utama.

  

# ========== AUTENTIKASI ==========

PubkeyAuthentication yes

PasswordAuthentication no

# PERINGATAN: pastikan key sudah terpasang & teruji SEBELUM baris ini di-set 'no'.

PermitEmptyPasswords no

  

# ========== AKSES ROOT ==========

PermitRootLogin no

  

# ========== PEMBATASAN PERCOBAAN ==========

MaxAuthTries 3

MaxSessions 5

LoginGraceTime 20

  

# ========== PEMBATASAN USER ==========

AllowUsers alice bob

# atau: AllowGroups sshusers

  

# ========== KEAMANAN TAMBAHAN ==========

X11Forwarding no

AllowTcpForwarding no

GatewayPorts no

PermitUserEnvironment no

StrictModes yes

UseDNS no

LogLevel VERBOSE

AuthorizedKeysFile .ssh/authorized_keys

  

# ========== SESSION TIMEOUT ==========

ClientAliveInterval 300

ClientAliveCountMax 2

# Efek: sesi idle > 10 menit (300 x 2 detik) otomatis diputus.

  

# ========== BANNER ==========

Banner /etc/ssh/banner.txt

```

  

**Urutan eksekusi WAJIB (jangan dibalik):**

  

```

1. Pasang public key di server

2. Test login dengan key di terminal BARU (jangan tutup terminal lama!)

3. Baru set PasswordAuthentication no

4. Restart SSH

5. Test lagi dari terminal baru

```

  

Melanggar urutan ini = terkunci dari server tanpa jalan masuk lewat SSH.

  

**VERIFY setelah edit:**

```bash

sudo sshd -t                        # harus tanpa output = tanpa error

sudo systemctl restart sshd

sudo systemctl status sshd          # harus active (running)

```

  

**Buat banner login (aspek legal):**

```bash

sudo nano /etc/ssh/banner.txt

```

```

============================================================

  SISTEM INI HANYA UNTUK PENGGUNA YANG BERWENANG.

  Semua aktivitas dicatat dan dipantau.

  Akses tanpa izin akan ditindaklanjuti secara hukum.

============================================================

```

  

**Permission file SSH:**

  

| File/Folder | Harus | Kalau Salah |

|---|---|---|

| `~/.ssh/` | `700` | SSH tidak mau pakai key di dalamnya |

| `~/.ssh/authorized_keys` | `600` | SSH tidak mau baca file ini |

| `~/.ssh/id_rsa` (private) | `600` | SSH tidak mau pakai key ini |

| `~/.ssh/id_rsa.pub` (public) | `644` | Tidak bermasalah, standar saja |

  

```bash

chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys        # server

chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_rsa && chmod 644 ~/.ssh/id_rsa.pub   # client

```

  

**Troubleshooting SSH:**

  

| Masalah | Kemungkinan Penyebab | Solusi |

|---|---|---|

| `Permission denied (publickey)` | Permission `~/.ssh` salah | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |

| Key tidak dikenali | Isi `authorized_keys` tidak cocok | `cat ~/.ssh/authorized_keys` |

| Tidak bisa konek sama sekali | SSH mati / port salah | `sudo systemctl status sshd` |

| Koneksi ditolak | UFW memblok | `sudo ufw status` |

| Koneksi lambat | DNS lookup | Tambah `UseDNS no` |

  

Debug: `ssh -vvv user@server_ip` (dari client); `sudo tail -f /var/log/auth.log` (di server).

  

---

  

### 4.2 HARDEN — UFW Firewall

  

**Prinsip default-deny:**

```

TANPA firewall: Internet → [semua port terbuka] → Server

DENGAN firewall: Internet → [UFW] → hanya port yang di-allow → sisanya otomatis diblokir

```

  

**Setup — urutan WAJIB (allow SSH dulu, baru enable):**

  

```bash

# 1. Cek status sekarang

sudo ufw status verbose

  

# 2. Set default policy DULU

sudo ufw default deny incoming

sudo ufw default allow outgoing

  

# 3. Allow SSH SEBELUM enable — atau langsung terkunci!

sudo ufw allow 2222/tcp     # sesuaikan dengan Port di sshd_config

  

# 4. Baru aktifkan

sudo ufw enable

  

# 5. Verifikasi

sudo ufw status verbose

```

  

**Allow/deny port lain:**

```bash

sudo ufw allow 80/tcp

sudo ufw allow 443/tcp

sudo ufw allow from 192.168.1.100 to any port 22      # 1 IP spesifik saja

sudo ufw allow from 192.168.1.0/24 to any port 443    # 1 subnet

  

sudo ufw deny 21/tcp        # FTP

sudo ufw deny 23/tcp        # Telnet

sudo ufw deny 3306/tcp      # MySQL dari luar

sudo ufw deny 6379/tcp      # Redis dari luar

  

sudo ufw limit 22/tcp       # rate-limit: auto-block IP > 6 koneksi/30 detik

```

  

**Kelola aturan:**

```bash

sudo ufw status numbered

sudo ufw delete 4                # hapus berdasarkan nomor

sudo ufw delete deny 21/tcp      # hapus berdasarkan definisi

sudo ufw reload

sudo ufw reset                   # hati-hati: kembali ke kosong

```

  

---

  

### 4.3 HARDEN — Fail2ban

  

**Cara kerja:**

```

/var/log/auth.log: 3x "Failed password for root from 1.2.3.4"

        ↓

   [fail2ban] deteksi ambang gagal terlampaui

        ↓

   tambahkan aturan "DROP semua paket dari 1.2.3.4"

        ↓

   IP diblokir selama durasi bantime

```

  

**Install:**

```bash

sudo apt install fail2ban -y

```

  

**Konfigurasi — WAJIB copy ke `.local`, jangan edit `.conf` langsung:**

```bash

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

sudo nano /etc/fail2ban/jail.local

```

  

```ini

[DEFAULT]

ignoreip = 127.0.0.1/8 ::1

findtime = 600

maxretry = 3

bantime = 3600

# bantime = -1 untuk permanent ban

  

[sshd]

enabled = true

port = 2222          # sesuaikan dengan Port di sshd_config

filter = sshd

logpath = /var/log/auth.log

maxretry = 3

bantime = 3600

```

  

**Aktifkan:**

```bash

sudo systemctl enable fail2ban

sudo systemctl start fail2ban

sudo systemctl status fail2ban

sudo systemctl restart fail2ban   # setiap ada perubahan konfigurasi

```

  

**Monitoring:**

```bash

sudo fail2ban-client status

sudo fail2ban-client status sshd

sudo tail -f /var/log/fail2ban.log

sudo fail2ban-client set sshd unbanip 192.168.1.100

sudo fail2ban-client set sshd banip 192.168.1.200

```

  

---

  

### 4.4 Lapisan Tambahan — TCP Wrappers

  

> Deprecated sejak Ubuntu 22+, tapi masih sering muncul di soal lomba. Urutan pemrosesan: cek `hosts.allow` dulu (ada izin → masuk) → cek `hosts.deny` (ada larangan → ditolak) → tidak ada di keduanya → diizinkan (karena itu `hosts.deny` diisi `ALL : ALL`).

  

```bash

sudo nano /etc/hosts.allow

```

```

sshd : 192.168.1.0/24

ALL  : 127.0.0.1

```

```bash

sudo nano /etc/hosts.deny

```

```

ALL : ALL

```

  

---

  

### 4.5 SSH Recovery Procedure

  

> Halaman khusus — dipakai saat SSH terkunci akibat kesalahan urutan hardening (lihat Jebakan #1 dan #5). Prasyarat: sesi lama masih terbuka, ATAU akses console VM lewat panel hypervisor tersedia.

  

**Skenario:** "SSH gagal setelah hardening"

  

```

GEJALA

  Tidak bisa login SSH — "Permission denied" atau connection refused/timeout,

  sesi lama (kalau masih terbuka) tetap bisa dipakai tapi sesi baru gagal.

  

CHECK

  # Kalau masih ada sesi lama yang terbuka, jalankan dari situ:

  systemctl status sshd

  sudo sshd -t

  sudo journalctl -u sshd -n 50 --no-pager

  

  # Kalau tidak ada sesi terbuka sama sekali, masuk lewat CONSOLE VM

  # (panel hypervisor), bukan lewat jaringan.

  

RECOVERY

  # Restore config dari backup terarah Module 01:

  BACKUP_DIR=~/lksn-hardening/config-backup/pre-hardening_<timestamp>

  sudo cp "$BACKUP_DIR/etc/ssh/sshd_config" /etc/ssh/sshd_config

  sudo sshd -t                      # wajib tanpa error sebelum restart

  sudo systemctl restart sshd

  

  # Kalau backup Module 01 tidak tersedia, minimal set manual:

  #   PasswordAuthentication yes

  #   PermitRootLogin yes (sementara, untuk recovery saja)

  # lalu ulangi 4.1 dari Langkah 1 dengan lebih hati-hati.

  

VALIDATE

  sudo systemctl status sshd        # harus active (running)

  ssh -p <port> user@server_ip      # login berhasil dari terminal baru

  # Setelah berhasil, ulangi 4.1 langkah demi langkah — jangan lompat

  # langsung ke PasswordAuthentication no sebelum key benar-benar teruji.

```

  

---

  

### 4.6 Firewall Recovery Procedure

  

> Halaman khusus — dipakai saat UFW mengunci akses jaringan (lihat Jebakan #2). Karena SSH sendiri diblokir firewall, perbaikan **tidak bisa** lewat SSH — wajib lewat console VM.

  

**Skenario:** "UFW mengunci akses"

  

```

GEJALA

  Koneksi SSH timeout/refused padahal sshd aktif dan config benar

  (bedakan dari 4.5: di sini sshd sehat, masalahnya di jalur jaringan).

  

CHECK

  # WAJIB masuk lewat console VM (panel hypervisor), bukan SSH —

  # karena kemungkinan besar port SSH sendiri yang terblokir.

  sudo ufw status verbose

  sudo ufw status numbered

  

RECOVERY

  # Opsi cepat: allow ulang port SSH yang aktif

  sudo ufw allow <port_ssh>/tcp

  sudo ufw reload

  

  # Kalau rule lama membingungkan / salah urutan, reset dan bangun ulang

  # SESUAI urutan wajib (allow SSH dulu, baru enable):

  sudo ufw disable

  sudo ufw default deny incoming

  sudo ufw default allow outgoing

  sudo ufw allow <port_ssh>/tcp

  sudo ufw enable

  

VALIDATE

  sudo ufw status verbose           # port SSH tampil ALLOW

  # Test dari MESIN LAIN/sesi baru (bukan console yang sama):

  ssh -p <port> user@server_ip      # harus berhasil

  # Setelah SSH terkonfirmasi jalan, lanjutkan allow/deny port lain

  # satu per satu sambil ufw tetap aktif — jangan ufw disable lagi

  # kecuali darurat.

```

  

---

  

## 5. Verification Phase

  

Jalankan setelah SSH, UFW, dan Fail2ban dikonfigurasi — dari **sesi/terminal terpisah**, jangan menutup sesi yang sedang aktif sampai semua verifikasi lulus.

  

**VERIFY — SSH**

```bash

sudo sshd -t                         # tanpa output = tanpa error

ssh -p 2222 user@server_ip           # login dengan key dari terminal baru harus berhasil

ssh -o PreferredAuthentications=password -p 2222 user@server_ip   # harus DITOLAK

```

  

**VERIFY — UFW**

```bash

sudo ufw status verbose              # default deny incoming, port SSH ter-allow

```

  

**VERIFY — Fail2ban**

```bash

sudo fail2ban-client status sshd     # jail aktif, siap mencatat ban

```

  

---

  

## 6. Checklist (Scoring-Oriented)

  

**SSH**

- [ ] Key pair dibuat, public key terpasang di server, login key **teruji berhasil** sebelum password auth dimatikan.

- [ ] `PasswordAuthentication no`, `PermitRootLogin no`, `PermitEmptyPasswords no`.

- [ ] `MaxAuthTries 3`, `X11Forwarding no`, `AllowTcpForwarding no`, `PermitUserEnvironment no`, `StrictModes yes`.

- [ ] `ClientAliveInterval 300` + `ClientAliveCountMax 2`; `AllowUsers`/`AllowGroups` dikonfigurasi.

- [ ] Permission: `~/.ssh` 700, `authorized_keys` 600, private key 600.

- [ ] Port SSH bukan 22; banner aktif; `sudo sshd -t` tanpa error.

  

**UFW**

- [ ] Aktif, default `deny incoming` / `allow outgoing`.

- [ ] Port SSH di-allow **sebelum** `ufw enable` dijalankan.

- [ ] Port berbahaya (21, 23, 3306, dll.) diblokir; rate limiting aktif untuk SSH.

  

**Fail2ban**

- [ ] Terinstall dan `active`; `jail.local` dikonfigurasi (bukan `jail.conf`).

- [ ] Jail `sshd` aktif dengan `maxretry = 3`, `bantime` sesuai kebijakan tim.

- [ ] `fail2ban-client status sshd` terverifikasi menampilkan jail berjalan.

  

---

  

## 7. Jebakan Umum (Competition Pitfalls)

  

| # | Jebakan | Akibat |

|---|---|---|

| 1 | Set `PasswordAuthentication no` sebelum key teruji | Terkunci total dari server via SSH |

| 2 | `ufw enable` sebelum allow port SSH | Terkunci dari sisi jaringan |

| 3 | Edit `jail.conf` langsung | Perubahan tertimpa saat update paket, tidak konsisten dinilai |

| 4 | Port di `jail.local` tidak disesuaikan setelah ganti Port SSH | Fail2ban memantau port yang salah, tidak pernah trigger |

| 5 | Menutup sesi lama sebelum sesi baru teruji berhasil | Tidak ada jalur mundur kalau konfigurasi baru gagal |

| 6 | Lupa `sudo sshd -t` sebelum restart | Restart gagal, service down di tengah lomba |

  

---

  

**Sebelumnya:** Module 01 — Baseline & Recon.

**Selanjutnya:** Module 03 — PAM & Password Complexity (di luar cakupan dokumen ini).