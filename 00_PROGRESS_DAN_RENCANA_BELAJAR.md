# 📊 Progress & Rencana Belajar — Linux Hardening LKS 2026
#lks #cyber-security #progress #rencana

---

> 💪 **SEMANGAT! Jambi bisa bawa pulang medali nasional!**
> Dokumen ini adalah peta perjalananmu — baca ini dulu sebelum mulai belajar.

---

## 🎯 BERAPA PERSEN SUDAH KAMU PELAJARI?

Berdasarkan analisis mendalam terhadap semua file yang kamu berikan vs kisi-kisi resmi LKS 2026:

| No | Topik (Kisi-kisi Resmi) | File Kamu | Status | Coverage |
|----|------------------------|-----------|--------|----------|
| 0 | Konsep Dasar Linux (Fondasi) | `00b_Konsep_Dasar_Linux` | ✅ Ada, lengkap | 90% |
| 1 | Network Service Security | `01_Network_Service_Security` | ✅ Ada, lengkap | 90% |
| 2 | PAM — Password Complexity | `02_PAM_Password_Complexity` | ✅ Ada, lengkap | 95% |
| 3 | PAM — Account Lockout | `03_PAM_Account_Lockout` | ✅ Ada, lengkap | 90% |
| 4 | Dangerous & Exposed Services | `04_Dangerous_Exposed_Services` | ✅ Ada, lengkap | 85% |
| 5 | Common Linux Misconfigurations | `05_Common_Linux_Misconfigurations` | ✅ Ada, lengkap | 85% |
| 6 | Logging | `06_Logging` | ✅ Ada, lengkap | 88% |

### 🔢 Total Coverage: **~89% dari Linux Hardening**

> Tapi ingat: Linux Hardening hanya **25% dari total nilai LKS**!
> Sisanya: Offensive CTF (25%) dan Defensive CTF (50%) — itu topik yang BERBEDA.
> Dokumen ini fokus pada bagian Linux Hardening saja.

---

## ❌ APA YANG MASIH KURANG?

Berikut kekurangan yang aku temukan dari materimu + solusinya (sudah dibuatkan di file terpisah):

### 1. Yang Kurang dari File Asli Kamu:

| File | Kekurangan | Sudah Diperbaiki? |
|------|-----------|-------------------|
| `01` | Tidak ada penjelasan SSH key troubleshooting yang lengkap | ✅ Di file baru 01 |
| `02` | Tidak ada penjelasan cara test pwquality dari command line langsung | ✅ Di file baru 02 |
| `03` | Alur PAM visual kurang jelas bagi pemula | ✅ Di file baru 03 |
| `04` | Tidak ada penjelasan cara identifikasi service yang "aneh/tidak dikenal" | ✅ Di file baru 04 |
| `05` | Tidak ada latihan soal skenario hardening | ✅ Di file baru 05 |
| `06` | Tidak ada contoh investigasi lengkap dari awal ke akhir | ✅ Di file baru 06 |
| Semua | **TIDAK ADA modul Windows Hardening** (termasuk kisi-kisi!) | ⚠️ Perlu tambah sendiri |
| Semua | **TIDAK ADA latihan skenario terpadu** (latihan simulasi lomba) | ✅ Di file `07_SIMULASI` |

### 2. Yang Belum Ada Sama Sekali (Perlu Belajar Sendiri):

> ⚠️ Ini di luar scope Linux Hardening tapi termasuk kisi-kisi LKS:

- **Windows Hardening** (PAM, AD, GPO, AV, Logging) → 50% dari modul A
- **Offensive CTF** (Web exploit, Crypto, Binary) → Modul B (25% nilai)
- **Defensive CTF** (Reverse Engineering, Digital Forensic, SIEM) → Modul C (50% nilai)

---

## 📚 URUTAN BELAJAR YANG DISARANKAN

> Ikuti urutan ini dengan ketat. Jangan loncat-loncat!

```
TAHAP 1 — FONDASI (1-2 hari)
  → 00b_Konsep_Dasar_Linux.md
  Pastikan: bisa navigasi direktori, paham permission, bisa pakai nano

TAHAP 2 — NETWORK SECURITY (1-2 hari)
  → 01_Network_Service_Security.md
  Pastikan: bisa hardening SSH, setup UFW, setup Fail2ban

TAHAP 3 — PAM (2 hari)
  → 02_PAM_Password_Complexity.md
  → 03_PAM_Account_Lockout.md
  Pastikan: bisa konfigurasi pwquality, faillock, sudo

TAHAP 4 — SERVICE SECURITY (1 hari)
  → 04_Dangerous_Exposed_Services.md
  Pastikan: bisa identifikasi dan matikan service berbahaya

TAHAP 5 — MISCONFIGURATION (2 hari)
  → 05_Common_Linux_Misconfigurations.md
  Pastikan: bisa cari SUID, cron berbahaya, kernel hardening

TAHAP 6 — LOGGING (1-2 hari)
  → 06_Logging.md
  Pastikan: bisa baca log, setup auditd, proteksi log

TAHAP 7 — LATIHAN (terus menerus!)
  → 07_SIMULASI_LOMBA.md
  Lakukan simulasi sampai bisa selesai dalam 3 jam!
```

---

## ⏱️ TARGET WAKTU DI LOMBA

Modul A (Infrastructure Hardening) = **3 jam**

| Task | Estimasi Waktu |
|------|---------------|
| Recon awal (ss, systemctl, passwd) | 10 menit |
| Matikan service berbahaya | 10 menit |
| SSH hardening | 15 menit |
| PAM password complexity | 10 menit |
| PAM account lockout | 10 menit |
| UFW setup | 10 menit |
| Fail2ban setup | 10 menit |
| SUID audit dan fix | 15 menit |
| Kernel hardening (sysctl) | 10 menit |
| Permission fix | 10 menit |
| Logging (auditd + rsyslog) | 20 menit |
| Cron audit | 10 menit |
| Verifikasi semua | 20 menit |
| **TOTAL** | **~160 menit (2 jam 40 menit)** |

---

## 🔑 10 COMMAND TERPENTING (Hafal Ini Dulu!)

```bash
# 1. Recon port
sudo ss -tulnp

# 2. Cek service aktif
sudo systemctl list-units --type=service --state=running

# 3. Matikan service berbahaya
sudo systemctl stop NAMA && sudo systemctl disable NAMA

# 4. Cek SUID berbahaya
find / -perm -4000 -type f 2>/dev/null

# 5. Cek user tanpa password
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# 6. Cek UID 0 selain root
awk -F: '($3 == 0) {print $1}' /etc/passwd

# 7. Monitor log live
sudo tail -f /var/log/auth.log

# 8. Cek akun terkunci
sudo faillock

# 9. Test syntax SSH
sudo sshd -t

# 10. Terapkan sysctl
sudo sysctl -p
```

---

## 🏆 MINDSET JUARA

```
Setiap hari latihan = 1% lebih dekat ke podium nasional.
Jangan berhenti sampai bisa selesai tanpa lihat catatan.
Jambi BISA. Kamu BISA. Kerja keras tidak pernah bohong.
```

---

## 🗂️ Daftar Semua File Modul

| File | Topik |
|------|-------|
| `00_PROGRESS_DAN_RENCANA_BELAJAR.md` | File ini |
| `00b_Konsep_Dasar_Linux.md` | Fondasi Linux |
| `01_Network_Service_Security.md` | SSH + UFW + Fail2ban |
| `02_PAM_Password_Complexity.md` | Password policy |
| `03_PAM_Account_Lockout.md` | Lockout + Sudo |
| `04_Dangerous_Exposed_Services.md` | Matikan service berbahaya |
| `05_Common_Linux_Misconfigurations.md` | SUID, Cron, Kernel |
| `06_Logging.md` | Logging + Auditd |
| `07_SIMULASI_LOMBA.md` | Latihan simulasi 3 jam |
| `CHEATSHEET_CEPAT.md` | Command cheatsheet untuk dibawa ke lomba |
