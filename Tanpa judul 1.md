# Runbook 9.1A — Linux Hardening

## Safety, Snapshot, Baseline Assessment, dan Asset Inventory

  

> **Cakupan dokumen ini:** hanya keselamatan, snapshot, backup terarah, baseline assessment, dan inventaris aset — tidak melakukan perubahan konfigurasi aktif; penulisan hanya terjadi pada folder kerja, backup, evidence, dan metadata snapshot. Perubahan konfigurasi aktif **tidak** dikerjakan di sini — dikerjakan di runbook lain sesuai pembagian final berikut:

> - **9.1B** — User, group, password policy, PAM, sudo.

> - **9.1C** — Package, service, network, SSH, firewall.

> - **9.1D** — Permission, SUID/SGID, cron, persistence.

> - **9.1E** — Logging, auditd, validasi akhir, rollback lintas-runbook, checklist akhir seluruh rangkaian 9.1.

  

> **Batas legal:** seluruh perintah pada runbook ini hanya boleh dijalankan pada VM latihan pribadi, sistem lab resmi LKSN, atau lingkungan lomba resmi — tidak untuk sistem di luar konteks tersebut.

  

---

  

## 1. Tujuan

  

Setelah menyelesaikan bagian 9.1A, peserta mampu:

  

1. Memahami objektif dan batas perubahan sistem pada tahap ini.

2. Mengidentifikasi layanan yang wajib tetap aktif sebelum menyentuh sistem apa pun.

3. Membuat snapshot VM dan backup konfigurasi terarah sebelum melakukan perubahan apa pun di runbook berikutnya.

4. Mendokumentasikan kondisi awal sistem secara lengkap dan objektif.

5. Mengidentifikasi distro, versi OS, kernel, arsitektur, dan package manager.

6. Menginventarisasi user, group, privilege, service, process, port, interface, route, mount, cron, package, dan mekanisme logging yang benar-benar berjalan.

7. Menyimpan evidence kondisi awal dengan struktur folder rapi dan hash pembuktian.

8. Menyusun baseline yang bisa dibandingkan setelah hardening (before/after).

9. Mengidentifikasi kandidat quick win dan kandidat review/investigasi (tanpa mengeksekusi perubahan).

10. Membuat checkpoint aman sebelum masuk ke Bagian 9.1B.

  

---

  

## 2. Prioritas

  

**P0 — wajib mutlak.** Bagian ini adalah fondasi Modul A (Hardening, bobot 25%) dan menjadi acuan pembanding (before/after) untuk seluruh validasi hardening di runbook berikutnya. Tanpa baseline yang benar, skor *Judgement* (kejelasan POC/writeup) sulit dibuktikan.

  

---

  

## 3. Ruang Lingkup

  

| Termasuk (9.1A) | Tidak termasuk (dikerjakan di runbook lain) |

|---|---|

| Snapshot VM & backup konfigurasi terarah sebelum perubahan | User, group, password policy, PAM, sudo (**9.1B**) |

| Identifikasi sistem (distro, kernel, arsitektur, package manager) | Package & service hardening aktif, konfigurasi network aktif, SSH hardening, firewall (**9.1C**) |

| Inventaris user, group, privilege (read-only) | Permission hardening aktif, pembersihan SUID/SGID, hardening cron, deteksi & pembersihan persistence (**9.1D**) |

| Inventaris service, process, port, koneksi (read-only) | Instalasi/konfigurasi auditd aktif, validasi akhir & rollback lintas-runbook, checklist akhir seluruh rangkaian 9.1 (**9.1E**) |

| Inventaris package, cron, mount, dan mekanisme logging (read-only) | — |

| Evidence collection & hashing | — |

| Baseline summary & daftar kandidat review (quick win) | Troubleshooting (Runbook 9.2) |

  

---

  

## 4. Prasyarat dan Hak Akses

  

| Kebutuhan | Keterangan |

|---|---|

| Akses shell | Lokal (console VM) atau SSH ke VM target |

| Akun | User biasa **dan** akses sudo/root (sebagian perintah butuh root) |

| Dasar command line | `cd`, `ls`, `cat`, pipe (`\|`), redirect (`>`, `>>`), grouping `{ ...; }` |

| Akses hypervisor/konsol | Untuk membuat snapshot VM (VirtualBox/VMware/Proxmox/panel lomba) |

| Koneksi jaringan | Tidak wajib — seluruh langkah bisa dilakukan offline di VM |

  

---

  

## 5. Aturan Keselamatan

  

> - Jangan menjalankan perintah apa pun terhadap sistem di luar VM latihan pribadi, platform CTF resmi, atau lomba resmi.

> - Jangan menonaktifkan service apa pun pada tahap ini.

> - Jangan menghapus user, group, file, package, atau log apa pun.

> - Jangan mengubah password, konfigurasi SSH, atau firewall.

> - Jangan menjalankan `apt/dnf/yum update` atau `upgrade`.

> - Jangan menjalankan script otomatis/download dari internet yang belum diperiksa isinya.

> - Jangan mengasumsikan service/proses asing otomatis berbahaya — catat dulu, verifikasi nanti di runbook analisis.

> - Jangan menyalin `/etc/security/opasswd`, `/etc/shadow`, `/etc/gshadow`, private key, token, kredensial, atau secret apa pun ke folder evidence maupun config-backup — cukup catat metadata (permission/ownership) jika relevan.

> - Jika ragu apakah sebuah perintah aman, jangan jalankan — tandai di catatan dan lanjutkan ke perintah lain.

> - Simpan **setiap** output sebelum menutup terminal — kondisi VM bisa di-reset panitia di luar kendali peserta.

  

---

  

## 6. Struktur Folder Kerja

  

```bash

# [READ-ONLY→CHANGE pada workdir sendiri, TIDAK mengubah sistem] [SAFE]

mkdir -p ~/lksn-hardening/{evidence/{00_system_identity,01_network,02_users_groups,03_privileges,04_services_processes,05_ports_connections,06_packages,07_scheduled_tasks,08_storage_mounts,09_logs,10_hashes,baseline_summary},config-backup,notes,reports}

cd ~/lksn-hardening

```

  

* **Fungsi:** membuat direktori kerja pribadi di `$HOME` — folder `evidence/` (berisi 12 subfolder di atas), `config-backup/` (backup konfigurasi terarah, lihat Bagian 8.2), `notes/`, dan `reports/` — tidak menyentuh direktori sistem.

* **Hak akses:** user biasa (tidak perlu root).

* **Validasi:** `ls ~/lksn-hardening` menampilkan folder `evidence/`, `config-backup/`, `notes/`, dan `reports/`; `ls -R ~/lksn-hardening/evidence` menampilkan 12 subfolder di atas.

* **Jika `mkdir -p` tidak dikenali:** buat satu per satu dengan `mkdir nama_folder`.

  

Konvensi penamaan file evidence: `NN_topik_YYYYMMDD_HHMM.txt` (contoh: `01_ip_a_20260101_0930.txt`) agar tidak tertimpa saat dijalankan ulang.

  

---

  

## 7. Initial Assessment

  

**Tujuan:** memastikan siapa/di mana kita berada sebelum menyentuh apa pun, dan mencatat siapa saja yang sedang login (untuk menghindari mengganggu sesi orang lain di sistem shared/lab).

  

```bash

# [READ-ONLY][SAFE] seluruh output digabung dalam satu blok { ...; } lalu di-tee

# agar SEMUA perintah benar-benar tersimpan, bukan hanya perintah terakhir.

{

  echo "== whoami ==";        whoami

  echo "== id ==";            id

  echo "== hostname ==";      hostname

  echo "== date ==";          date

  echo "== uptime ==";        uptime

  echo "== who ==";           who

  echo "== last -a (20 terakhir) =="; last -a | head -n 20

} | tee "evidence/00_system_identity/00_initial_assessment_$(date +%Y%m%d_%H%M).txt"

```

  

* **Hak akses:** user biasa.

* **Hasil diharapkan:** identitas jelas, tidak ada sesi lain yang akan terganggu.

* **Kenapa pakai `{ ...; } | tee`:** menulis `cmd1; cmd2; cmd3 > file` hanya me-redirect output `cmd3` ke file — output `cmd1` dan `cmd2` akan tampil di layar lalu hilang. Grouping `{ ...; }` menyatukan seluruh stdout dari semua perintah di dalamnya menjadi satu stream, lalu `tee` menyimpannya ke file **sekaligus** menampilkannya di layar.

* **Error umum:** `last` kosong pada VM baru → normal, tapi perintah tidak otomatis menghasilkan keterangan apa pun untuk kondisi ini — peserta wajib mencatat secara manual di notes bahwa "belum ada histori login" pada VM tersebut.

  

---

  

## 8. Snapshot dan Backup Terarah

  

### 8.1 Snapshot VM (wajib sebelum ada perubahan apa pun di runbook manapun)

  

```bash

df -h    # [READ-ONLY][SAFE] cek kapasitas filesystem GUEST VM (bukan storage host/hypervisor)

```

  

* **Tujuan:** titik pulih penuh jika terjadi kesalahan fatal di runbook berikutnya.

* **Cara:** melalui panel hypervisor (VirtualBox: *Machine → Take Snapshot*; VMware: *VM → Snapshot → Take Snapshot*; Proxmox/panel lomba: sesuai menu masing-masing). **Bukan perintah shell** — dilakukan dari luar VM.

* **Label:** [SAFE] tidak mengubah isi disk saat pengambilan snapshot berlangsung.

* **Penamaan snapshot:** `pre-hardening-baseline-<tanggal>`.

* **Catatan cakupan `df -h`:** perintah di atas hanya melihat kapasitas filesystem **di dalam guest VM**. Kapasitas storage tempat snapshot benar-benar disimpan berada di sisi **host/hypervisor** dan tidak terlihat dari dalam VM — periksa itu dari **antarmuka/panel hypervisor** (VirtualBox/VMware/Proxmox/panel lomba), bukan dari command line guest.

* **Kapasitas penyimpanan:** snapshot bukan operasi "gratis" — setiap snapshot menyimpan delta perubahan disk dan **tetap membutuhkan ruang penyimpanan** di storage host/hypervisor, yang bisa terus bertambah seiring berjalannya perubahan di runbook berikutnya. Periksa sisa kapasitas storage host (bukan hanya disk di dalam VM, yang dicek dengan `df -h` di atas) melalui antarmuka hypervisor sebelum mengambil snapshot, dan pantau ulang jika mengambil snapshot berkali-kali sepanjang persiapan.

* **Verifikasi mekanisme restore (wajib, jangan diasumsikan):** snapshot yang berhasil dibuat **tidak otomatis berarti restore akan berhasil**. Sebelum mengandalkan snapshot ini sebagai jaring pengaman, konfirmasikan bahwa mekanisme restore benar-benar berfungsi — misalnya cek menu *Restore/Revert to Snapshot* tersedia dan dapat diakses, atau tanyakan ke proctor/panitia bagaimana prosedur restore resmi di platform lomba. Jika tidak sempat melakukan uji restore penuh, minimal pastikan prosedurnya terdokumentasi dan dapat diakses saat dibutuhkan.

* **Validasi:** snapshot muncul di daftar snapshot hypervisor dengan ukuran > 0.

* **Jika platform lomba tidak menyediakan snapshot manual:** tanyakan ke panitia/proctor apakah tersedia mekanisme restore resmi; jangan berasumsi sendiri.

  

### 8.2 Backup Konfigurasi Terarah (bukan backup penuh `/etc`)

  

> **Perubahan kebijakan:** runbook ini **tidak lagi** membackup seluruh `/etc`. Backup penuh `/etc` berisiko ikut menyalin `/etc/shadow`, kunci privat, dan file sensitif lain ke folder evidence, serta menghasilkan arsip besar yang sebagian besar isinya tidak relevan. Sebagai gantinya, backup hanya menyasar file konfigurasi yang **akan diubah** di Runbook 9.1B–9.1E.

  

```bash

# [READ-ONLY terhadap sistem asli][SAFE] menyalin (bukan memindahkan) file/folder target saja

BACKUP_DIR=~/lksn-hardening/config-backup/pre-hardening_$(date +%Y%m%d_%H%M)

sudo install -d -m 700 -o root -g root "$BACKUP_DIR"

  

# Daftar path yang relevan untuk 9.1B (user/PAM/sudo), 9.1C (package/service/network/SSH/firewall),

# 9.1D (permission/cron/persistence), 9.1E (logging/auditd). Path yang tidak ada di sistem ini akan dilewati.

TARGETS="

/etc/login.defs

/etc/pam.d

/etc/security/pwquality.conf

/etc/security/pwquality.conf.d

/etc/security/faillock.conf

/etc/security/limits.conf

/etc/security/limits.d

/etc/security/access.conf

/etc/sudoers

/etc/sudoers.d

/etc/ssh/sshd_config

/etc/ssh/sshd_config.d

/etc/crontab

/etc/cron.d

/etc/cron.daily

/etc/cron.hourly

/etc/cron.weekly

/etc/cron.monthly

/etc/rsyslog.conf

/etc/rsyslog.d

/etc/syslog-ng

/etc/audit

/etc/ufw

/etc/nftables.conf

/etc/sysconfig/iptables

/etc/firewalld

"

  

: > /tmp/targeted_backup_log.txt

for path in $TARGETS; do

  if sudo test -e "$path"; then

    sudo cp -a --parents "$path" "$BACKUP_DIR" 2>>/tmp/targeted_backup_log.txt \

      && echo "OK   : $path" >> /tmp/targeted_backup_log.txt

  else

    echo "SKIP (tidak ada di sistem ini): $path" >> /tmp/targeted_backup_log.txt

  fi

done

  

sudo cp /tmp/targeted_backup_log.txt "$BACKUP_DIR/_backup_log.txt"

  

# Backup TETAP root-owned (tidak dikembalikan ke user biasa); kunci permission agar

# tidak bisa diakses group/other — akses berikutnya ke $BACKUP_DIR wajib pakai sudo.

sudo chmod -R go-rwx "$BACKUP_DIR"

```

  

* **Fungsi:** salinan referensi konfigurasi awal **hanya** untuk file yang benar-benar akan disentuh di runbook berikutnya, sehingga before/after bisa dibandingkan tanpa membawa risiko kebocoran data sensitif.

* **Hak akses:** [ROOT] (sudo) untuk seluruh proses backup. Hasil backup tetap dimiliki `root` (tidak di-`chown` ke user biasa) dan permission dikunci dengan `chmod -R go-rwx` sehingga hanya root yang bisa membaca — akses/verifikasi berikutnya terhadap `$BACKUP_DIR` wajib memakai `sudo`.

* **Larangan tegas:** JANGAN menambahkan `/etc/security/opasswd`, `/etc/shadow`, `/etc/gshadow`, kunci privat (mis. `/etc/ssh/ssh_host_*_key`), token, kredensial, atau secret apa pun ke daftar `TARGETS` atau ke folder mana pun (`evidence/`, `config-backup/`, `notes/`, `reports/`).

* **Hasil diharapkan:** folder `config-backup/pre-hardening_<timestamp>/` berisi struktur path yang benar-benar ada di sistem ini (sebagian path di atas bersifat opsional tergantung distro/tool firewall yang dipakai — SKIP adalah hasil normal, bukan error).

* **Validasi:** `sudo find "$BACKUP_DIR" -type f | wc -l` menghasilkan jumlah > 0; isi `_backup_log.txt` (dibaca dengan `sudo cat`) menunjukkan setiap path berstatus `OK` atau `SKIP`. Karena eksistensi setiap path sudah dicek lebih dulu dengan `sudo test -e`, seharusnya **tidak ada** baris `Permission denied` pada path yang berstatus `OK` — jika muncul, itu bukan hal wajar, catat sebagai anomali untuk diperiksa lebih lanjut.

* **Kemungkinan error:** sebagian besar `SKIP` adalah normal (mis. sistem Debian tidak punya `/etc/sysconfig/iptables`, sistem RHEL tidak punya `/etc/ufw`) — cocokkan dengan hasil identifikasi distro di Bagian 9.

* **Jika `cp --parents` tidak dikenali (BSD-style cp/minimal system):** salin manual per path dengan `sudo mkdir -p "$BACKUP_DIR$(dirname <path>)" && sudo cp -a <path> "$BACKUP_DIR$(dirname <path>)/"`.

  

---

  

## 9. Identifikasi Sistem

  

**Tujuan:** memastikan seluruh perintah lanjutan disesuaikan dengan distro dan package manager yang benar.

  

```bash

cat /etc/os-release          # [READ-ONLY][SAFE] nama & versi distro (standar di semua distro modern)

uname -a                     # [READ-ONLY][SAFE] kernel, arsitektur, hostname

uname -m                     # [READ-ONLY][SAFE] arsitektur singkat (x86_64/aarch64)

hostnamectl 2>/dev/null      # [READ-ONLY][SAFE] ringkasan (jika systemd tersedia)

```

  

**Deteksi keluarga distro & package manager:**

  

| Keluarga | Penanda | Package manager | Perintah listing |

|---|---|---|---|

| Debian / Ubuntu | `/etc/debian_version` ada | `apt` / `dpkg` | `dpkg -l` |

| RHEL / Rocky / AlmaLinux | `/etc/redhat-release` ada | `dnf` (fallback `yum`) | `rpm -qa` |

  

* **Simpan:** setiap output → `evidence/00_system_identity/`, contoh `os_release_<timestamp>.txt`.

* **Validasi:** bandingkan `ID=` pada `/etc/os-release` dengan tabel keluarga distro di atas. Jika tidak cocok dengan satu pun (mis. distro turunan yang jarang dipakai), **jangan anggap gagal** — tandai sebagai kandidat review agar perintah-perintah selanjutnya (package manager, lokasi log) disesuaikan manual.

* **Kemungkinan error:** `hostnamectl` tidak ada di sistem minimal/container → gunakan `uname -a` + `cat /etc/os-release` sebagai gantinya.

* **Bonus (opsional, catat saja):** cek Mandatory Access Control — Debian/Ubuntu: `sudo aa-status 2>/dev/null`; RHEL/Rocky/Alma: `getenforce 2>/dev/null`. Ini murni observasi status, bukan perubahan.

  

---

  

## 10. Inventory Jaringan

  

```bash

ip a                              # [READ-ONLY][SAFE] daftar interface & IP

ip route                           # [READ-ONLY][SAFE] tabel routing

ip -4 route get 8.8.8.8            # [READ-ONLY][SAFE] cek jalur keluar default (hanya lookup tabel, tidak mengirim paket)

cat /etc/hosts                     # [READ-ONLY][SAFE] pemetaan host lokal

cat /etc/resolv.conf               # [READ-ONLY][SAFE] konfigurasi DNS

nmcli device show 2>/dev/null      # [READ-ONLY][SAFE] detail koneksi (jika NetworkManager terpasang)

```

  

* **Hak akses:** user biasa cukup untuk seluruh perintah di atas.

* **Hasil diharapkan:** daftar interface aktif, IP, gateway default, dan resolver DNS tercatat.

* **Simpan:** masing-masing ke `evidence/01_network/`.

* **Validasi:** interface loopback (`lo`) dan minimal satu interface fisik/virtual muncul di `ip a`.

* **Kemungkinan error:** `nmcli: command not found` → normal jika sistem memakai `systemd-networkd`/`ifupdown`; cukup andalkan `ip a` dan `ip route`.

  

---

  

## 11. Inventory User dan Group

  

```bash

getent passwd                       # [READ-ONLY][SAFE] semua user (termasuk sumber NSS/LDAP jika ada)

cat /etc/passwd                     # [READ-ONLY][SAFE] user lokal

cat /etc/group                      # [READ-ONLY][SAFE] semua group

grep -E '^UID_MIN' /etc/login.defs  # [READ-ONLY][SAFE] cek ambang UID_MIN yang berlaku di sistem ini

ls -l /etc/shadow                   # [READ-ONLY][ROOT][SAFE] cek PERMISSION saja, jangan cat isinya

```

  

**Enumerasi user non-sistem (menggantikan asumsi hardcode UID≥1000):**

  

```bash

# [READ-ONLY][SAFE] ambil UID_MIN aktual dari /etc/login.defs, fallback 1000 jika tidak ditemukan,

# dan kecualikan UID 65534 (konvensi umum untuk akun "nobody"/nfsnobody, bukan user manusia)

UID_MIN=$(awk '$1=="UID_MIN"{print $2; exit}' /etc/login.defs 2>/dev/null)

UID_MIN=${UID_MIN:-1000}

awk -F: -v uidmin="$UID_MIN" '($3 >= uidmin) && ($3 != 65534) {print $1, $3}' /etc/passwd

```

  

* **Fungsi:** memetakan siapa saja user yang ada, mana yang akun sistem vs akun manusia, memakai ambang yang benar-benar berlaku di sistem ini (bukan angka `1000` yang di-hardcode, karena beberapa distro/konfigurasi kustom bisa memakai `UID_MIN` berbeda).

* **Hak akses:** user biasa untuk `passwd`/`group`; root untuk membaca `/etc/shadow` (di sini hanya cek permission, bukan isi, agar hash password tidak tercatat di evidence).

* **Hasil diharapkan:** daftar lengkap user/group tersimpan; permission `/etc/shadow` tercatat (idealnya `600`/`640` milik `root:shadow`); daftar user non-sistem sesuai `UID_MIN` aktual, tanpa UID 65534.

* **Simpan:** ke `evidence/02_users_groups/`.

* **Validasi:** jumlah baris `/etc/passwd` masuk akal (tidak nol). Jika ditemukan user dengan UID `0` selain `root`, atau anomali lain pada daftar user non-sistem, **tandai sebagai kandidat investigasi lanjutan** — bukan otomatis disimpulkan sebagai kompromi/kesalahan, karena bisa juga berupa konfigurasi lab yang memang disengaja.

* **Kemungkinan error:** `getent` tidak dikenali di sistem sangat minimal → cukup gunakan `cat /etc/passwd` dan `cat /etc/group`; jika `/etc/login.defs` tidak memuat baris `UID_MIN` sama sekali, fallback `1000` pada script di atas akan otomatis dipakai.

* **Catatan:** JANGAN menyalin isi `/etc/shadow` ke evidence — cukup catat permission-nya untuk menjaga kerahasiaan hash password.

  

---

  

## 12. Inventory Privilege

  

```bash

getent group sudo wheel 2>/dev/null        # [READ-ONLY][SAFE] anggota grup admin (sudo=Debian/Ubuntu, wheel=RHEL family)

sudo cat /etc/sudoers                      # [READ-ONLY][ROOT][SAFE] isi aturan sudo utama

sudo ls -la /etc/sudoers.d/                # [READ-ONLY][ROOT][SAFE] file sudoers tambahan

sudo find / -xdev -perm -4000 -type f 2>/dev/null   # [READ-ONLY][ROOT][SAFE] daftar file SUID

sudo find / -xdev -perm -2000 -type f 2>/dev/null   # [READ-ONLY][ROOT][SAFE] daftar file SGID

```

  

* **Fungsi:** memetakan siapa yang punya hak admin dan file mana yang berjalan dengan privilege tinggi (SUID/SGID) — dasar untuk analisis risiko privesc, **bukan** untuk mengubah apa pun sekarang.

* **Hak akses:** [ROOT] untuk sebagian besar; `getent group` bisa tanpa root.

* **Catatan performa:** gunakan `-xdev` agar pencarian tidak menyeberang ke filesystem lain (mencegah proses menggantung lama di mount jaringan).

* **Hasil diharapkan:** daftar anggota grup admin dan daftar file SUID/SGID tersimpan lengkap.

* **Simpan:** ke `evidence/03_privileges/`.

* **Validasi:** jumlah baris hasil `find` dicatat apa adanya; entri yang tidak dikenal atau di luar dugaan **ditandai sebagai kandidat review** untuk dibandingkan dengan baseline resmi CIS jika tersedia offline — bukan disimpulkan berbahaya secara langsung.

* **Kemungkinan error:** `find` lambat pada disk besar → jalankan di background dengan `&` lalu cek dengan `jobs`, atau batasi `-maxdepth` bila waktu terbatas.

* **Jika `sudo` tidak tersedia untuk akun peserta:** laporkan ke proctor — akses root/sudo adalah prasyarat wajib kompetisi hardening.

  

---

  

## 13. Inventory Service dan Process

  

```bash

systemctl list-units --type=service --state=running   # [READ-ONLY][SAFE] service yang sedang jalan

systemctl list-unit-files --type=service               # [READ-ONLY][SAFE] semua service terdaftar + status enable/disable

ps aux                                                  # [READ-ONLY][SAFE] semua process berjalan

ps -ef --forest                                          # [READ-ONLY][SAFE] hierarki parent-child process

```

  

* **Hak akses:** user biasa cukup untuk melihat (root hanya diperlukan untuk melihat detail process milik user lain di beberapa sistem yang dikeraskan).

* **Hasil diharapkan:** daftar service aktif vs semua service terdaftar, plus snapshot process tree.

* **Simpan:** ke `evidence/04_services_processes/`.

* **Validasi:** jumlah service `running` dibandingkan dengan total `unit-files` sebagai konteks, bukan patokan pass/fail; process yang tidak dikenal **dicatat sebagai kandidat investigasi**, jangan langsung dimatikan.

* **Kemungkinan error:** `systemctl: command not found` (sistem non-systemd/lama) → fallback `service --status-all` (Debian family) atau `chkconfig --list` (RHEL family lama).

  

---

  

## 14. Inventory Port dan Koneksi

  

```bash

sudo ss -tulnp        # [READ-ONLY][ROOT][SAFE] port listening + nama proses

ss -tuln               # [READ-ONLY][SAFE] port listening tanpa nama proses (tanpa root)

sudo ss -tanp   # [READ-ONLY][ROOT][SAFE] koneksi TCP aktif (established, dll)

```

  

* **Fungsi:** memetakan port yang terbuka dan proses pemiliknya — dasar untuk membandingkan "port wajib" vs "port kandidat review" di analisis lanjutan.

* **Hasil diharapkan:** daftar port listening lengkap dengan proses (jika root).

* **Simpan:** ke `evidence/05_ports_connections/`.

* **Validasi:** port listening yang proses pemiliknya tidak teridentifikasi (karena tanpa root, atau proses sudah berhenti) **ditandai untuk dicek ulang**, bukan langsung dianggap mencurigakan.

* **Kemungkinan error:** `ss: command not found` (jarang) → fallback `sudo netstat -tulnp`; jika `netstat` juga tidak ada, gunakan `sudo lsof -i -P -n`.

  

---

  

## 15. Inventory Package dan Aplikasi

  

| Keluarga | Perintah utama | Alternatif |

|---|---|---|

| Debian/Ubuntu | `dpkg -l` | `apt list --installed 2>/dev/null` |

| RHEL/Rocky/AlmaLinux | `rpm -qa \| sort` | `dnf list installed` (fallback `yum list installed`) |

  

```bash

dpkg -l > evidence/06_packages/dpkg_list_$(date +%Y%m%d_%H%M).txt        # [READ-ONLY][SAFE] Debian/Ubuntu

rpm -qa | sort > evidence/06_packages/rpm_list_$(date +%Y%m%d_%H%M).txt  # [READ-ONLY][SAFE] RHEL/Rocky/Alma

```

  

* **Hak akses:** user biasa.

* **Hasil diharapkan:** daftar lengkap package terpasang, siap dibandingkan dengan baseline resmi/CIS.

* **Validasi:** `wc -l` pada file hasil > 0; jumlah dicatat sebagai konteks (ratusan–ribuan tergantung image), bukan patokan pass/fail tunggal.

* **Kemungkinan error:** perintah salah keluarga (mis. `rpm -qa` di Ubuntu) → akan gagal "command not found", gunakan tabel di atas untuk memilih perintah yang benar sesuai hasil Bagian 9.

  

---

  

## 16. Inventory Scheduled Task dan Cron

  

```bash

sudo crontab -l -u root 2>/dev/null                       # [READ-ONLY][ROOT][SAFE] cron milik root

for u in $(cut -f1 -d: /etc/passwd); do echo "== $u =="; sudo crontab -l -u "$u" 2>/dev/null; done   # [READ-ONLY][ROOT][SAFE] cron semua user

cat /etc/crontab                                            # [READ-ONLY][SAFE] cron sistem

ls -la /etc/cron.d /etc/cron.daily /etc/cron.hourly /etc/cron.weekly /etc/cron.monthly   # [READ-ONLY][SAFE]

systemctl list-timers --all                                 # [READ-ONLY][SAFE] scheduled task via systemd timer

atq 2>/dev/null                                              # [READ-ONLY][SAFE] job `at` terjadwal (jika ada)

```

  

**Lokasi spool cron per keluarga distro (untuk pemeriksaan manual bila perlu):**

  

| Keluarga | Lokasi spool cron user |

|---|---|

| Debian/Ubuntu | `/var/spool/cron/crontabs/` |

| RHEL/Rocky/AlmaLinux | `/var/spool/cron/` |

  

* **Hasil diharapkan:** daftar lengkap tugas terjadwal per user + sistem, tidak ada yang terlewat.

* **Simpan:** ke `evidence/07_scheduled_tasks/`.

* **Validasi:** entri cron yang tidak dikenal/tanpa keterangan sumber **dicatat sebagai kandidat investigasi** (bukan langsung dihapus) — penghapusan/hardening cron aktif dikerjakan di Runbook 9.1D.

* **Kemungkinan error:** `atq: command not found` → paket `at` memang tidak selalu terpasang default, catat "tidak tersedia" dan lanjut.

  

---

  

## 17. Inventory Mount, Storage, dan File System

  

```bash

df -hT           # [READ-ONLY][SAFE] penggunaan disk + tipe filesystem

mount             # [READ-ONLY][SAFE] semua mount point aktif

lsblk -f          # [READ-ONLY][SAFE] struktur block device + filesystem + label

cat /etc/fstab    # [READ-ONLY][SAFE] konfigurasi mount permanen

```

  

* **Hasil diharapkan:** peta lengkap disk, partisi, dan mount point, termasuk mana yang `noexec`/`nosuid`/`ro` (relevan untuk analisis risiko lanjutan, bukan diubah sekarang).

* **Simpan:** ke `evidence/08_storage_mounts/`.

* **Validasi:** bandingkan entri `/etc/fstab` dengan hasil `mount`. Entri yang ada di `fstab` tapi tidak muncul di `mount` dan bukan berlabel `noauto` **ditandai sebagai kandidat investigasi** (bisa jadi mount gagal, device lepas, atau memang sengaja belum di-mount) — bukan disimpulkan sebagai kesalahan pasti.

* **Kemungkinan error:** `lsblk: command not found` pada sistem sangat minimal → cukup andalkan `df -hT` dan `mount`.

  

---

  

## 18. Inventory Logging

  

**Tujuan:** memastikan mekanisme logging yang benar-benar aktif di sistem ini teridentifikasi — **tanpa berasumsi** bahwa sistem hanya memakai `journald` atau `rsyslog` saja. Sebagian image/distro memakai `syslog-ng`, kombinasi beberapa mekanisme sekaligus, forwarding ke syslog remote, atau konfigurasi khusus lomba.

  

```bash

ls -la /var/log                                                     # [READ-ONLY][SAFE] daftar file log

systemctl list-units --all --type=service \

  | grep -iE 'journal|syslog|rsyslog|syslog-ng'                     # [READ-ONLY][SAFE] cari SEMUA service terkait logging yang benar-benar ada di sistem ini

systemctl is-active systemd-journald 2>/dev/null                    # [READ-ONLY][SAFE]

systemctl is-active rsyslog 2>/dev/null                             # [READ-ONLY][SAFE]

systemctl is-active syslog-ng 2>/dev/null                           # [READ-ONLY][SAFE]

journalctl --disk-usage 2>/dev/null                                 # [READ-ONLY][SAFE] hanya relevan jika journald ada

ls -la /etc/rsyslog.conf /etc/rsyslog.d 2>/dev/null                 # [READ-ONLY][SAFE]

ls -la /etc/syslog-ng 2>/dev/null                                   # [READ-ONLY][SAFE]

```

  

**Lokasi log klasik per keluarga distro (jika rsyslog/syslog klasik aktif):**

  

| Keluarga | Log sistem umum | Log autentikasi |

|---|---|---|

| Debian/Ubuntu | `/var/log/syslog` | `/var/log/auth.log` |

| RHEL/Rocky/AlmaLinux | `/var/log/messages` | `/var/log/secure` |

  

* **Hasil diharapkan:** peserta tahu mekanisme logging apa saja yang **benar-benar berjalan** di sistem ini (bisa lebih dari satu, atau kombinasi), berdasarkan hasil pemeriksaan aktual di atas — bukan ditebak dari nama distro.

* **Simpan:** ke `evidence/09_logs/`.

* **Validasi:** minimal satu mekanisme logging teridentifikasi aktif dari hasil pemeriksaan di atas. Jika tidak ada satu pun yang aktif, atau hasilnya tidak sesuai ekspektasi (mis. rsyslog terpasang tapi tidak `active`), **tandai sebagai kandidat investigasi** — bisa jadi konfigurasi khusus lab, bukan otomatis kegagalan sistem.

* **Kemungkinan error:** file log klasik tidak ada meski rsyslog terpasang → cek `journalctl` dan `systemctl is-active` sebagai sumber kebenaran aktual pada distro modern (Ubuntu 20+/RHEL 8+ cenderung journald-sentris, tapi tidak selalu — selalu verifikasi, jangan asumsikan).

  

---

  

## 19. Evidence Collection

  

**Prinsip:** setiap perintah observasi → simpan outputnya, jangan hanya dibaca di layar.

  

```bash

# Pola simpan tunggal

<perintah> > evidence/<folder>/<nama>_$(date +%Y%m%d_%H%M).txt

  

# Pola simpan sekaligus tampil di layar

<perintah> | tee evidence/<folder>/<nama>_$(date +%Y%m%d_%H%M).txt

  

# Pola untuk beberapa perintah berurutan yang harus tersimpan SEMUA (lihat Bagian 7)

{ <perintah1>; <perintah2>; <perintah3>; } | tee evidence/<folder>/<nama>_$(date +%Y%m%d_%H%M).txt

  

# Hashing seluruh evidence (jalankan PALING TERAKHIR, setelah semua file lengkap)

find evidence -type f ! -name "SHA256SUMS.txt" -exec sha256sum {} \; > evidence/10_hashes/SHA256SUMS.txt

```

  

* **Fungsi hashing:** membuktikan evidence tidak diubah setelah dikumpulkan (integritas bukti untuk writeup).

* **Hak akses:** user biasa.

* **Validasi:** `sha256sum -c evidence/10_hashes/SHA256SUMS.txt` (dijalankan dari direktori yang sama) mengembalikan `OK` untuk semua baris.

* **Kemungkinan error:** path berubah/berpindah folder saat verifikasi → jalankan verifikasi dari direktori kerja yang sama persis saat hashing dibuat.

  

---

  

## 20. Baseline Summary

  

Buat `evidence/baseline_summary/baseline_summary.md` berisi ringkasan seperti templat berikut (isi manual dari hasil Bagian 9–18):

  

```markdown

# Baseline Summary — <hostname> — <tanggal>

  

| Item | Nilai |

|---|---|

| Distro & versi | |

| Kernel | |

| Arsitektur | |

| Package manager | |

| UID_MIN aktif (/etc/login.defs) | |

| Total user non-sistem (UID≥UID_MIN, tidak termasuk 65534) | |

| Total group | |

| Anggota grup admin (sudo/wheel) | |

| Total service running | |

| Total service terdaftar | |

| Total port listening | |

| Total package terpasang | |

| Total cron/scheduled task | |

| Total mount point | |

| Mekanisme logging aktif (hasil pemeriksaan aktual) | |

| Total file evidence | |

| Hash evidence dibuat? (Y/N) | |

| Backup konfigurasi terarah dibuat? (Y/N) | |

| Restore snapshot terverifikasi? (Y/N) | |

```

  

* **Tipe:** [CHANGE] tapi hanya pada file baru milik peserta, tidak menyentuh sistem.

* **Validasi:** semua baris tabel terisi, tidak ada sel kosong.

  

---

  

## 21. Kandidat Review (Quick Wins)

  

> Bagian ini hanya **mengidentifikasi kandidat**, bukan mengeksekusi. Eksekusi dilakukan di runbook terkait (9.1B dan seterusnya).

  

Kandidat umum yang sering muncul dari hasil inventory dan tergolong risiko rendah untuk diperbaiki nanti:

  

1. Service berjalan tapi tidak dikenal/tidak dibutuhkan tim → kandidat *review* (bukan langsung disable).

2. Banner login (`/etc/issue`, `/etc/motd`) belum memuat catatan legal/kepemilikan.

3. Cron job tanpa komentar/keterangan sumber.

4. Port listening yang tidak sesuai daftar service resmi kompetisi.

5. File SUID/SGID di luar daftar standar distro (perlu dibandingkan dengan baseline resmi).

  

---

  

## 22. Perubahan Berisiko Tinggi (belum dilakukan, sengaja ditunda)

  

| Area | Ditunda ke |

|---|---|

| User/group management aktif, password policy, PAM | Runbook **9.1B** |

| Konfigurasi sudoers | Runbook **9.1B** |

| Package & service hardening aktif (disable/uninstall) | Runbook **9.1C** |

| Konfigurasi network aktif | Runbook **9.1C** |

| Hardening SSH | Runbook **9.1C** |

| Aturan firewall | Runbook **9.1C** |

| Permission hardening aktif (chmod/chown massal) | Runbook **9.1D** |

| Pembersihan SUID/SGID mencurigakan | Runbook **9.1D** |

| Hardening cron (pembatasan akses, review job) | Runbook **9.1D** |

| Deteksi & pembersihan persistence mencurigakan | Runbook **9.1D** |

| Instalasi/konfigurasi auditd | Runbook **9.1E** |

| Validasi akhir & checklist akhir seluruh rangkaian 9.1 | Runbook **9.1E** |

  

---

  

## 23. Validasi

  

- [ ] Semua 12 subfolder `evidence/` terisi minimal satu file.

- [ ] `SHA256SUMS.txt` ada dan `sha256sum -c` mengembalikan `OK` untuk semua entri.

- [ ] `baseline_summary.md` terisi lengkap tanpa sel kosong.

- [ ] Snapshot VM `pre-hardening-baseline-<tanggal>` terkonfirmasi ada di hypervisor, dan mekanisme restore sudah diverifikasi (bukan hanya diasumsikan berhasil).

- [ ] Backup konfigurasi terarah (`config-backup/pre-hardening_<timestamp>/`) tersedia untuk file yang akan diubah di 9.1B–9.1E, **tanpa** `/etc/shadow`, private key, atau kredensial.

- [ ] Enumerasi user non-sistem memakai `UID_MIN` dari `/etc/login.defs` dan mengecualikan UID 65534.

- [ ] Mekanisme logging dicatat berdasarkan pemeriksaan aktual (bukan asumsi hanya journald/rsyslog).

- [ ] Tidak ada service yang dimatikan, user dihapus, atau konfigurasi diubah selama proses (bandingkan `uptime`/`who` sebelum-sesudah).

  

---

  

## 24. Jebakan Umum

  

1. Lupa `sudo` → banyak output kosong/`Permission denied`, dikira sistem "bersih" padahal hanya tidak terbaca.

2. Menjalankan `>` berulang tanpa timestamp → evidence lama tertimpa dan hilang.

3. Langsung menyimpulkan service asing = malware tanpa mencatat dulu untuk investigasi lanjutan.

4. Melewatkan snapshot VM, atau melewatkan verifikasi mekanisme restore-nya, sebelum mencoba perintah apa pun di runbook berikutnya.

5. Menjalankan `find /` tanpa `-xdev` di sistem dengan mount jaringan → proses menggantung lama.

6. Menyalin isi `/etc/shadow`, private key, atau kredensial ke evidence — cukup catat permission-nya saja, atau lewati sama sekali.

7. Melakukan backup penuh `/etc` (bukan lagi bagian dari runbook ini) — berisiko ikut menyalin file sensitif; gunakan hanya backup terarah Bagian 8.2.

8. Memakai angka `1000` yang di-hardcode untuk enumerasi user, padahal `UID_MIN` sistem bisa berbeda — selalu ambil dari `/etc/login.defs`.

9. Berasumsi sistem hanya memakai journald atau rsyslog tanpa memeriksa `systemctl is-active` secara langsung.

10. Hashing dijalankan sebelum semua evidence lengkap → hash tidak mewakili kondisi akhir baseline.

  

---

  

## 25. Rollback

  

Karena seluruh langkah di bagian ini **tidak melakukan perubahan konfigurasi aktif; perubahan hanya terjadi pada folder kerja, backup, evidence, laporan, hash, dan metadata snapshot**, secara normal **tidak ada yang perlu di-rollback** pada sistem.

  

Jika terjadi kesalahan tidak sengaja (mis. salah target redirect `>` menimpa file sistem, atau salah menjalankan perintah instalasi):

  

1. Hentikan aktivitas lanjutan segera.

2. Catat perintah persis apa yang salah dijalankan dan waktunya.

3. Restore VM dari snapshot `pre-hardening-baseline-<tanggal>` (Bagian 8.1) — **hanya jika** mekanisme restore sudah pernah diverifikasi berfungsi; jika belum pernah diverifikasi, koordinasikan dulu dengan proctor/mentor sebelum mencoba restore di tengah tekanan waktu.

4. Jika tidak ada snapshot tersedia atau restore gagal, laporkan ke proctor/mentor sebelum melanjutkan — jangan mencoba memperbaiki sendiri tanpa panduan resmi.

5. Setelah restore, ulangi Bagian 6–20 dari awal untuk memastikan baseline tetap valid.

  

---

  

## 26. Checklist Selesai

  

- [ ] Identitas sistem (distro, versi, kernel, arsitektur, package manager) tercatat.

- [ ] Konfigurasi jaringan awal (interface, IP, route, DNS) tercatat.

- [ ] Daftar user dan group tercatat, memakai `UID_MIN` aktual dan mengecualikan UID 65534.

- [ ] Daftar privilege (anggota admin, SUID/SGID, sudoers) tercatat.

- [ ] Daftar service (running & terdaftar) tercatat.

- [ ] Daftar process (snapshot `ps aux`) tercatat.

- [ ] Daftar port listening tercatat.

- [ ] Daftar koneksi aktif tercatat.

- [ ] Daftar package terpasang tercatat.

- [ ] Daftar cron dan scheduled task tercatat.

- [ ] Daftar mount dan storage tercatat.

- [ ] Mekanisme logging aktual teridentifikasi (bukan asumsi journald/rsyslog semata).

- [ ] Evidence lengkap dan ber-hash (`SHA256SUMS.txt`).

- [ ] `baseline_summary.md` terisi lengkap.

- [ ] Daftar kandidat review (quick wins) tercatat (belum dieksekusi).

- [ ] Daftar tindakan berisiko tinggi tercatat sebagai "belum dilakukan", dengan penunjuk runbook 9.1B/9.1C/9.1D/9.1E yang benar.

- [ ] Snapshot VM `pre-hardening-baseline` terkonfirmasi, dan mekanisme restore-nya sudah diverifikasi.

- [ ] Backup konfigurasi terarah tersedia untuk file yang akan diubah nanti, tanpa file sensitif.

  

---

  

## 27. SESSION_CHECKPOINT

  

**Dokumen ini:** `01A_Runbook_Linux_Hardening_Baseline_Inventory_FINAL.md` — revisi penuh dari 9.1A. Tidak melakukan perubahan konfigurasi aktif; perubahan hanya berupa pembuatan folder kerja, snapshot, backup, evidence, laporan, dan hash.: keselamatan, snapshot, backup terarah, baseline assessment, dan inventaris aset — tanpa hardening aktif.

  

**Perubahan utama pada revisi ini:**

1. Pembagian final 9.1 ditetapkan: **9.1A** (baseline & inventory — dokumen ini), **9.1B** (user/group/password policy/PAM/sudo), **9.1C** (package/service/network/SSH/firewall), **9.1D** (permission/SUID-SGID/cron/persistence), **9.1E** (logging/auditd/validasi akhir/rollback/checklist akhir). Pembagian 9.1F dan 9.1G **dihapus** — dilebur ke 9.1D dan 9.1E.

2. Command initial assessment (Bagian 7) diperbaiki memakai grouping `{ ...; }` + `tee` agar seluruh output benar-benar tersimpan.

3. Backup penuh `/etc` dihapus, diganti backup terarah (Bagian 8.2) — hanya file yang akan diubah di 9.1B–9.1E, tanpa `/etc/shadow`, private key, atau kredensial.

4. Ditambahkan penjelasan bahwa snapshot tetap butuh kapasitas penyimpanan dan mekanisme restore harus diverifikasi (Bagian 8.1).

5. Validasi yang terlalu absolut (UID 0 selain root, kecocokan distro, fstab vs mount, mekanisme logging) diubah menjadi kandidat review/investigasi.

6. Enumerasi user (Bagian 11) kini memakai `UID_MIN` dari `/etc/login.defs`, mengecualikan UID 65534.

7. Inventory logging (Bagian 18) memeriksa kondisi aktual, tidak berasumsi hanya journald/rsyslog.

  

**Tidak diubah:** struktur bab, label keamanan (READ-ONLY/CHANGE/ROOT/SAFE), batas legal, dan cakupan 9.1A yang tidak melakukan perubahan konfigurasi aktif

  

**Sengaja belum dikerjakan:** Runbook 9.1B belum dibuat pada sesi ini.

  

**Tugas berikutnya:** Runbook 9.1B — User, Group, Password Policy, PAM, dan Sudo, mengikuti format dan cakupan yang telah difinalkan di atas.