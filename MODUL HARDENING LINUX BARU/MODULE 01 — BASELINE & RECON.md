# MODULE 01 — BASELINE & RECON

## LKS Cyber Security Field Manual — Linux Hardening

  

> **Mode:** Field Manual (Competition-Ready) · **Peran:** Defender · **Bobot terkait:** Fondasi Modul Hardening (25%)

> **Prasyarat lomba:** akses shell (lokal/SSH) + sudo/root, akses panel hypervisor untuk snapshot.

> **Prinsip modul:** 30% pemahaman, 70% eksekusi lomba. Tidak ada perubahan konfigurasi aktif di modul ini — murni baseline read-only, snapshot, dan evidence.

> **Kompatibilitas:** seluruh command di modul ini teruji pada Ubuntu 22.04, 24.04, dan 26.04. Tidak ada command distro-specific yang butuh alternatif di modul ini.

  

---

  

## 0. QUICK START — 15 MENIT PERTAMA

  

> Gunakan bagian ini saat lomba baru dimulai. Jangan baca seluruh modul dari atas — checklist ini adalah jalur tercepat menuju evidence dasar yang lengkap. Detail command dan interpretasi tiap item ada di bagian 4, rujuk hanya kalau ada yang gagal/tidak jelas.

  

```

[ ] 1. Snapshot VM lewat panel hypervisor            → 4.3

[ ] 2. Buat struktur folder evidence                  → 4.1

[ ] 3. Catat identitas sesi (whoami/id/who)           → 4.2

[ ] 4. Backup konfigurasi terarah (bukan /etc penuh)  → 4.4

[ ] 5. Cek distro/kernel                              → 4.5

[ ] 6. Cek jaringan (ip a, route, resolv.conf)        → 4.6

[ ] 7. Cek user & group + UID_MIN                     → 4.7

[ ] 8. Cek privilege (sudoers, SUID/SGID)             → 4.8

[ ] 9. Cek service & process yang berjalan            → 4.9

[ ] 10. Cek port & koneksi listening                  → 4.10

[ ] 11. Cek package terpasang                         → 4.11

[ ] 12. Cek cron & scheduled task                     → 4.12

[ ] 13. Cek mount & storage                            → 4.13

[ ] 14. Cek mekanisme logging aktif                    → 4.14

[ ] 15. Hash seluruh evidence (PALING TERAKHIR)        → 4.15

[ ] 16. Isi baseline_summary.md                        → Bagian 5

```

  

**Urutan #1 dan #15 tidak bisa ditukar posisinya:** snapshot harus di awal (sebelum apa pun berubah), hashing harus di akhir (setelah semua evidence lengkap, lihat Jebakan #8). Item 2–14 boleh dikerjakan sedikit fleksibel urutannya asal semua tercentang sebelum lanjut ke Module 02.

  

---

  

## 1. Objective

  

**Tujuan modul:** membangun *ground truth* kondisi awal sistem — siapa user-nya, service apa yang jalan, port apa yang terbuka, package apa yang terpasang — sebelum satu perintah hardening pun dijalankan.

  

**Kaitan dengan kompetisi:** setiap juri LKS menilai *Judgement* lewat kejelasan before/after. Tanpa baseline yang tercatat dan ter-hash, klaim "sudah saya hardening" tidak punya pembanding — nilai POC/writeup jatuh meskipun hardening-nya benar secara teknis. Baseline juga satu-satunya cara membedakan "service ini memang harus ada dari panitia" vs "service ini kandidat serangan".

  

**Risiko jika modul ini dilewati:**

- Mematikan service yang ternyata dibutuhkan skenario lomba → server dianggap down, poin availability hilang.

- Tidak ada snapshot → satu perintah salah di modul berikutnya bisa mengunci diri sendiri dari server tanpa jalan pulang.

- Tidak ada evidence/hash → writeup tidak bisa dibuktikan, rentan didiskualifikasi klaim tanpa bukti.

  

---

  

## 2. Concept Foundation

  

Bayangkan Anda mengambil alih sebuah rumah kos yang baru saja ditinggalkan pemilik lama. Sebelum mengganti kunci atau memasang pagar, Anda **foto dulu setiap ruangan**: siapa yang punya kunci cadangan, jendela mana yang terbuka, listrik apa yang menyala. Kalau langsung mengganti kunci tanpa foto, Anda bisa saja mengunci penghuni sah yang memang berhak tinggal di sana.

  

Baseline = foto kondisi awal itu. Tiga lapisan yang dipetakan:

  

```

IDENTITAS SISTEM        →  distro, kernel, package manager

        │                  (menentukan perintah apa yang valid dipakai)

        ▼

INVENTARIS ASET         →  user/group, service, process, port, package, cron, mount

        │                  (menentukan apa yang "normal" vs "kandidat investigasi")

        ▼

EVIDENCE + SNAPSHOT      →  hash pembuktian + titik pulih VM

                            (jaring pengaman sebelum modul hardening aktif dimulai)

```

  

Hubungan penting: **snapshot VM** (Bagian 8.1) adalah jaring pengaman di level sistem operasi — jika modul berikutnya (user/service/firewall/permission) merusak sistem, snapshot inilah yang dipulihkan. **Backup konfigurasi terarah** (Bagian 8.2) berbeda: hanya menyalin file spesifik yang *akan* diubah, bukan seluruh `/etc`, supaya tidak ikut membawa `/etc/shadow` atau private key ke folder bukti.

  

---

  

## 3. Competition Scenario

  

> **Kondisi lomba:** Server diserahkan ke peserta dalam kondisi "sudah pernah dipakai" — ada beberapa service asing yang tidak dikenal berjalan, beberapa port terbuka tanpa keterangan, dan cron job tanpa komentar sumber. Waktu terbatas, dan panitia bisa me-reset VM sewaktu-waktu di luar kendali peserta.

  

**Cara berpikir defender di titik ini:**

  

1. **Jangan reaktif.** Insting pertama saat melihat service asing adalah langsung `systemctl stop` atau `kill`. Ini salah di fase baseline — service itu bisa saja bagian dari skenario resmi lomba (mis. web app yang harus tetap tersedia untuk dinilai). Defender yang baik **mencatat dulu, mengeksekusi nanti** di modul yang tepat.

2. **Asumsikan VM bisa hilang kapan saja.** Karena reset di luar kontrol peserta, setiap output observasi harus langsung disimpan ke file — bukan hanya dibaca di layar lalu ditutup.

3. **Pisahkan "tidak dikenal" dari "berbahaya".** Tidak dikenal = butuh investigasi. Berbahaya = sudah ada bukti kuat (mis. koneksi keluar ke IP asing + proses tersembunyi). Baseline hanya menghasilkan kandidat, bukan vonis.

4. **Availability adalah bagian dari defense.** Mematikan sesuatu yang salah bisa menjatuhkan skor sama seperti dibobol attacker.

  

---

  

## 4. Recon / Detection Phase

  

Seluruh perintah di bawah **read-only terhadap sistem** — hanya menulis ke folder kerja pribadi peserta.

  

### 4.1 Setup Folder Kerja

  

**CHECK — struktur folder evidence sudah siap**

  

Command:

```bash

mkdir -p ~/lksn-hardening/{evidence/{00_system_identity,01_network,02_users_groups,03_privileges,04_services_processes,05_ports_connections,06_packages,07_scheduled_tasks,08_storage_mounts,09_logs,10_hashes,baseline_summary},config-backup,notes,reports}

cd ~/lksn-hardening

ls ~/lksn-hardening && ls -R ~/lksn-hardening/evidence

```

  

Expected output: folder `evidence/` (12 subfolder), `config-backup/`, `notes/`, `reports/` semuanya muncul.

  

Interpretasi: kalau `mkdir -p` tidak dikenali (shell minimal), buat manual satu-satu dengan `mkdir <nama>`. Konvensi nama file evidence: `NN_topik_YYYYMMDD_HHMM.txt` agar tidak tertimpa saat perintah diulang.

  

---

  

### 4.2 Initial Assessment (Siapa & Di Mana Kita)

  

**CHECK — identitas sesi, hindari mengganggu sesi lain di sistem shared**

  

Command:

```bash

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

  

Expected output: identitas user jelas, daftar sesi login aktif.

  

Interpretasi: `{ ...; } | tee` wajib dipakai — kalau perintah dirangkai dengan `;` biasa lalu diakhiri `>`, hanya output perintah terakhir yang tersimpan. `last` kosong pada VM baru = normal, catat manual "belum ada histori login" di notes, jangan dibiarkan tanpa keterangan.

  

---

  

### 4.3 Snapshot VM (Wajib Sebelum Modul Berikutnya)

  

**CHECK — kapasitas filesystem guest sebelum snapshot**

  

Command:

```bash

df -h

```

  

Interpretasi: ini hanya melihat kapasitas **di dalam guest VM**. Kapasitas storage tempat snapshot benar-benar disimpan ada di sisi host/hypervisor — cek dari panel VirtualBox/VMware/Proxmox, bukan dari command line guest. Snapshot bukan operasi gratis: setiap snapshot menyimpan delta perubahan disk dan terus bertambah seiring modul berikutnya berjalan.

  

**Aksi (di luar shell):** ambil snapshot lewat panel hypervisor dengan nama `pre-hardening-baseline-<tanggal>`.

  

**Wajib diverifikasi, jangan diasumsikan:** snapshot yang berhasil dibuat **tidak otomatis berarti restore akan berhasil**. Konfirmasikan menu *Restore/Revert to Snapshot* benar-benar tersedia, atau tanyakan prosedur restore resmi ke proctor. Jika platform lomba tidak menyediakan snapshot manual, tanyakan ke panitia — jangan berasumsi sendiri.

  

---

  

### 4.4 Backup Konfigurasi Terarah

  

> Bukan backup penuh `/etc` — hanya file yang akan disentuh di modul-modul berikutnya (user/PAM/sudo, service/network/SSH/firewall, permission/cron, logging/auditd). Backup penuh `/etc` berisiko menyalin `/etc/shadow` dan kunci privat ke evidence.

  

Command:

```bash

BACKUP_DIR=~/lksn-hardening/config-backup/pre-hardening_$(date +%Y%m%d_%H%M)

sudo install -d -m 700 -o root -g root "$BACKUP_DIR"

  

TARGETS="

/etc/login.defs /etc/pam.d /etc/security/pwquality.conf /etc/security/pwquality.conf.d

/etc/security/faillock.conf /etc/security/limits.conf /etc/security/limits.d

/etc/security/access.conf /etc/sudoers /etc/sudoers.d

/etc/ssh/sshd_config /etc/ssh/sshd_config.d

/etc/crontab /etc/cron.d /etc/cron.daily /etc/cron.hourly /etc/cron.weekly /etc/cron.monthly

/etc/rsyslog.conf /etc/rsyslog.d /etc/syslog-ng /etc/audit

/etc/ufw /etc/nftables.conf /etc/sysconfig/iptables /etc/firewalld

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

sudo chmod -R go-rwx "$BACKUP_DIR"

```

  

Expected output: `sudo find "$BACKUP_DIR" -type f | wc -l` > 0; setiap baris di `_backup_log.txt` berstatus `OK` atau `SKIP`.

  

Interpretasi: `SKIP` untuk path yang tidak ada di distro ini adalah **normal** (mis. Debian tidak punya `/etc/sysconfig/iptables`), bukan error — cocokkan dengan hasil identifikasi distro di 4.5. `Permission denied` pada path berstatus `OK` **bukan hal wajar** karena eksistensi sudah dicek lebih dulu — catat sebagai anomali.

  

> ⚠️ **Larangan tegas:** jangan pernah menambahkan `/etc/shadow`, `/etc/gshadow`, `/etc/security/opasswd`, private key SSH host, token, atau kredensial apa pun ke `TARGETS` atau folder evidence manapun.

  

---

  

### 4.5 Identifikasi Sistem

  

**CHECK — distro, kernel, package manager**

  

Command:

```bash

cat /etc/os-release

uname -a

uname -m

hostnamectl 2>/dev/null

```

  

| Keluarga | Penanda | Package manager | Listing |

|---|---|---|---|

| Debian/Ubuntu | `/etc/debian_version` | `apt`/`dpkg` | `dpkg -l` |

| RHEL/Rocky/Alma | `/etc/redhat-release` | `dnf` (fallback `yum`) | `rpm -qa` |

  

Interpretasi: kalau `ID=` di `/etc/os-release` tidak cocok tabel di atas (distro turunan langka), jangan anggap gagal — tandai kandidat review agar perintah selanjutnya disesuaikan manual. `hostnamectl` tidak ada di sistem minimal → cukup pakai `uname -a`.

  

Simpan ke `evidence/00_system_identity/`.

  

---

  

### 4.6 Inventory Jaringan

  

Command:

```bash

ip a

ip route

ip -4 route get 8.8.8.8

cat /etc/hosts

cat /etc/resolv.conf

nmcli device show 2>/dev/null

```

  

Expected output: interface `lo` + minimal satu interface fisik/virtual, gateway default, resolver DNS. Simpan ke `evidence/01_network/`. `nmcli: command not found` normal jika sistem pakai `systemd-networkd`/`ifupdown`.

  

---

  

### 4.7 Inventory User & Group

  

Command:

```bash

getent passwd

cat /etc/passwd

cat /etc/group

grep -E '^UID_MIN' /etc/login.defs

ls -l /etc/shadow    # cek permission SAJA, jangan cat isinya

```

  

**Enumerasi user non-sistem (pakai UID_MIN aktual, bukan hardcode 1000):**

  

```bash

UID_MIN=$(awk '$1=="UID_MIN"{print $2; exit}' /etc/login.defs 2>/dev/null)

UID_MIN=${UID_MIN:-1000}

awk -F: -v uidmin="$UID_MIN" '($3 >= uidmin) && ($3 != 65534) {print $1, $3}' /etc/passwd

```

  

Interpretasi: UID 65534 dikecualikan (konvensi akun "nobody", bukan user manusia). User dengan UID `0` selain `root` = kandidat investigasi lanjutan, bukan langsung divonis kompromi. **Jangan pernah** menyalin isi `/etc/shadow` ke evidence — permission idealnya `600`/`640` milik `root:shadow`. Simpan ke `evidence/02_users_groups/`.

  

---

  

### 4.8 Inventory Privilege

  

Command:

```bash

getent group sudo wheel 2>/dev/null

sudo cat /etc/sudoers

sudo ls -la /etc/sudoers.d/

sudo find / -xdev -perm -4000 -type f 2>/dev/null   # SUID

sudo find / -xdev -perm -2000 -type f 2>/dev/null   # SGID

```

  

Interpretasi: `-xdev` wajib agar pencarian tidak menyeberang mount jaringan (mencegah proses menggantung). File SUID/SGID di luar dugaan = kandidat review untuk dibandingkan baseline CIS, bukan langsung disimpulkan berbahaya. Simpan ke `evidence/03_privileges/`.

  

---

  

### 4.9 Inventory Service & Process

  

Command:

```bash

systemctl list-units --type=service --state=running

systemctl list-unit-files --type=service

ps aux

ps -ef --forest

```

  

Interpretasi: bandingkan jumlah `running` vs total `unit-files` sebagai konteks, bukan patokan pass/fail. Process tidak dikenal → dicatat sebagai kandidat investigasi, **jangan langsung dimatikan** (lihat prinsip Competition Scenario di atas). Fallback non-systemd: `service --status-all` atau `chkconfig --list`. Simpan ke `evidence/04_services_processes/`.

  

---

  

### 4.10 Inventory Port & Koneksi

  

Command:

```bash

sudo ss -tulnp

ss -tuln

sudo ss -tanp

```

  

Interpretasi: port listening tanpa proses teridentifikasi (tanpa root, atau proses sudah berhenti) → dicek ulang, bukan langsung mencurigakan. Fallback: `sudo netstat -tulnp`, atau `sudo lsof -i -P -n`. Simpan ke `evidence/05_ports_connections/`.

  

---

  

### 4.11 Inventory Package

  

Command:

```bash

dpkg -l > evidence/06_packages/dpkg_list_$(date +%Y%m%d_%H%M).txt        # Debian/Ubuntu

rpm -qa | sort > evidence/06_packages/rpm_list_$(date +%Y%m%d_%H%M).txt  # RHEL/Rocky/Alma

```

  

Interpretasi: jumlah baris dicatat sebagai konteks (ratusan–ribuan tergantung image), bukan patokan pass/fail tunggal.

  

---

  

### 4.12 Inventory Cron & Scheduled Task

  

Command:

```bash

sudo crontab -l -u root 2>/dev/null

for u in $(cut -f1 -d: /etc/passwd); do echo "== $u =="; sudo crontab -l -u "$u" 2>/dev/null; done

cat /etc/crontab

ls -la /etc/cron.d /etc/cron.daily /etc/cron.hourly /etc/cron.weekly /etc/cron.monthly

systemctl list-timers --all

atq 2>/dev/null

```

  

Interpretasi: entri cron tanpa keterangan sumber = kandidat investigasi (bukan langsung dihapus — eksekusi hardening cron ada di modul lain). Simpan ke `evidence/07_scheduled_tasks/`.

  

---

  

### 4.13 Inventory Mount & Storage

  

Command:

```bash

df -hT

mount

lsblk -f

cat /etc/fstab

```

  

Interpretasi: entri `fstab` yang tidak muncul di `mount` dan bukan berlabel `noauto` = kandidat investigasi (bisa mount gagal, device lepas, atau memang sengaja belum di-mount). Simpan ke `evidence/08_storage_mounts/`.

  

---

  

### 4.14 Inventory Logging

  

> Jangan berasumsi sistem hanya pakai journald atau rsyslog — sebagian image memakai syslog-ng, kombinasi beberapa mekanisme, atau forwarding ke syslog remote.

  

Command:

```bash

ls -la /var/log

systemctl list-units --all --type=service | grep -iE 'journal|syslog|rsyslog|syslog-ng'

systemctl is-active systemd-journald 2>/dev/null

systemctl is-active rsyslog 2>/dev/null

systemctl is-active syslog-ng 2>/dev/null

journalctl --disk-usage 2>/dev/null

```

  

Interpretasi: minimal satu mekanisme logging harus teridentifikasi aktif dari hasil pemeriksaan **aktual**, bukan ditebak dari nama distro. Jika tidak ada satu pun aktif, tandai kandidat investigasi — bisa jadi konfigurasi khusus lab. Simpan ke `evidence/09_logs/`.

  

---

  

### 4.15 Evidence Collection & Hashing

  

Pola simpan:

```bash

<perintah> | tee evidence/<folder>/<nama>_$(date +%Y%m%d_%H%M).txt

```

  

Hashing (jalankan **paling terakhir**, setelah semua evidence lengkap):

```bash

find evidence -type f ! -name "SHA256SUMS.txt" -exec sha256sum {} \; > evidence/10_hashes/SHA256SUMS.txt

```

  

Verifikasi: `sha256sum -c evidence/10_hashes/SHA256SUMS.txt` harus mengembalikan `OK` untuk semua baris — dijalankan dari direktori kerja yang sama.

  

---

  

## 5. Baseline Summary & Kandidat Review

  

Isi `evidence/baseline_summary/baseline_summary.md` (manual, dari hasil 4.5–4.14):

  

```markdown

# Baseline Summary — <hostname> — <tanggal>

| Item | Nilai |

|---|---|

| Distro & versi | |

| Kernel | |

| UID_MIN aktif | |

| Total user non-sistem | |

| Total service running | |

| Total port listening | |

| Total package terpasang | |

| Mekanisme logging aktif | |

| Hash evidence dibuat? (Y/N) | |

| Snapshot terverifikasi? (Y/N) | |

```

  

**Kandidat review (baru dicatat, belum dieksekusi — eksekusi di modul hardening terkait):**

  

1. Service berjalan tapi tidak dikenal/tidak dibutuhkan → kandidat review, bukan langsung disable.

2. Banner login (`/etc/issue`, `/etc/motd`) belum memuat catatan legal.

3. Cron job tanpa komentar/keterangan sumber.

4. Port listening di luar daftar service resmi kompetisi.

5. File SUID/SGID di luar daftar standar distro.

  

---

  

### 5.1 Baseline Decision Tree

  

> Dipakai untuk setiap kandidat review di atas — bukan untuk memutuskan hardening (itu ranah Module 02+), tapi untuk memutuskan **apakah temuan ini perlu diinvestigasi lebih lanjut sekarang atau bisa ditunda**.

  

```

                        FINDING DITEMUKAN

                    (service/port/user/SUID/cron)

                               │

                               ▼

                  Apakah cocok dengan skenario

                  resmi lomba / sudah tercatat

                  di baseline sebagai "diketahui"?

                               │

                 ┌─────────────┴─────────────┐

                 │YA                        TIDAK

                 ▼                             ▼

        Catat sebagai baseline         INVESTIGASI

        normal, lanjut ke item          │

        berikutnya                     ▼

                                Apakah dibutuhkan

                                untuk fungsi sistem

                                / skenario lomba?

                                        │

                          ┌─────────────┴─────────────┐

                          │YA                        TIDAK

                          ▼                             ▼

                 Catat sebagai "known,          Catat sebagai kandidat

                 butuh disecure" →              disable/tindak lanjut →

                 eksekusi di modul               eksekusi di modul

                 hardening terkait                hardening terkait

                 (bukan dieksekusi                (bukan dieksekusi

                 sekarang)                        sekarang)

```

  

**Prinsip inti:** decision tree ini hanya menghasilkan *label*, tidak pernah eksekusi langsung (`systemctl stop`, `userdel`, dll). Semua eksekusi ada di modul hardening masing-masing setelah baseline selesai — lihat Competition Scenario poin 1.

  

---

  

### 5.2 Finding Interpretation Table

  

> Tabel referensi cepat. **Tidak semua temuan tidak biasa = serangan** — banyak yang normal tergantung skenario lomba. Gunakan bersama Decision Tree di atas.

  

| Temuan | Arti | Risiko | Tindakan |

|---|---|---|---|

| Service asing berjalan, tidak dikenal | Bisa jadi komponen skenario resmi (web app dinilai) atau residu image | Sedang — availability turun kalau salah matikan | Catat kandidat (5.1), verifikasi ke proctor/dokumentasi soal sebelum modul hardening service |

| Port terbuka tanpa service resmi terdaftar | Service listening tanpa keterangan pemilik | Sedang–Tinggi tergantung port (mis. 4444, 31337 = mencurigakan; 8080 = sering legit) | Cocokkan dengan 4.9 (proses pemilik port), baru putuskan di Module 02 |

| User dengan UID tinggi tidak dikenal | Bisa akun layanan legit (UID_MIN system) atau akun backdoor | Tinggi jika UID 0 selain root, atau user manusia tak terdaftar | Cross-check `/etc/passwd` vs daftar peserta/panitia resmi, jangan hapus di fase baseline |

| File SUID/SGID di luar standar distro | Bisa tool admin legit atau privilege escalation vector | Tinggi — SUID non-standar sering jadi jalur eskalasi | Bandingkan dengan baseline CIS/distro resmi, catat untuk audit permission di modul lanjutan |

| Cron job tanpa komentar sumber | Bisa maintenance script lama atau persistence mechanism attacker | Tinggi jika menjalankan binary di luar `/usr/bin` atau download dari internet | Baca isi command persis, cek apakah memanggil path mencurigakan, jangan hapus di fase baseline |

| Entri fstab tidak ter-mount | Device lepas, mount gagal, atau memang `noauto` disengaja | Rendah, kecuali menunjukkan storage eksternal tak terdokumentasi | Cocokkan dengan `lsblk` dan dokumentasi soal lomba |

  

---

  

## 6. Jebakan Umum (Competition Pitfalls)

  

| # | Jebakan | Akibat |

|---|---|---|

| 1 | Lupa `sudo` | Output kosong/`Permission denied` dikira sistem "bersih" |

| 2 | `>` berulang tanpa timestamp | Evidence lama tertimpa dan hilang |

| 3 | Langsung vonis service asing = malware | Availability turun tanpa bukti kuat |

| 4 | Lewat snapshot / lewat verifikasi restore | Tidak ada jalan pulang saat modul berikutnya salah |

| 5 | `find /` tanpa `-xdev` di mount jaringan | Proses menggantung, waktu lomba terbuang |

| 6 | Menyalin `/etc/shadow`/private key ke evidence | Kebocoran kredensial dari folder bukti sendiri |

| 7 | Hardcode UID `1000` untuk enumerasi user | Salah baca user non-sistem di distro yang beda `UID_MIN` |

| 8 | Hashing sebelum evidence lengkap | Hash tidak mewakili kondisi akhir baseline |

  

---

  

## 7. Rollback

  

Modul ini **tidak mengubah konfigurasi aktif** — hanya menulis ke folder kerja, backup terarah, evidence, dan metadata snapshot. Secara normal tidak ada yang perlu di-rollback pada sistem.

  

Jika terjadi kesalahan tidak sengaja (mis. salah target redirect `>` menimpa file sistem):

  

1. Hentikan aktivitas lanjutan segera.

2. Catat perintah persis yang salah dan waktunya.

3. Restore dari snapshot `pre-hardening-baseline-<tanggal>` — **hanya jika** mekanisme restore sudah pernah diverifikasi berfungsi.

4. Jika tidak ada snapshot atau restore gagal, laporkan ke proctor sebelum melanjutkan.

5. Setelah restore, ulangi Bagian 4–5 dari awal.

  

---

  

## 8. Checklist Selesai (Scoring-Oriented)

  

- [ ] Identitas sistem, jaringan, user/group, privilege, service/process, port, package, cron, mount, logging — semua tercatat di `evidence/`.

- [ ] Enumerasi user non-sistem memakai `UID_MIN` aktual, mengecualikan UID 65534.

- [ ] `SHA256SUMS.txt` ada dan `sha256sum -c` mengembalikan `OK` untuk semua entri.

- [ ] `baseline_summary.md` terisi lengkap tanpa sel kosong.

- [ ] Snapshot VM `pre-hardening-baseline-<tanggal>` terkonfirmasi, mekanisme restore terverifikasi.

- [ ] Backup konfigurasi terarah tersedia, **tanpa** file sensitif (`shadow`, private key, kredensial).

- [ ] Daftar kandidat review tercatat, belum dieksekusi.

- [ ] Tidak ada service dimatikan / user dihapus / konfigurasi diubah selama proses baseline.

  

**Lanjut ke:** Module 02 — Network Service Security.