# Panduan Setup VM Baru — Blue Team / Hardening (LKS Cybersecurity)

Checklist ini dipakai **setiap kali kamu setup VM baru** untuk latihan atau simulasi lomba, supaya boot cepat, konsisten, dan siap untuk tugas hardening/defense.

---

## 1. Kenapa Static IP > DHCP untuk Blue Team

Untuk peran hardening/blue team, **static IP adalah rekomendasi utama**, bukan DHCP. Alasannya:

| Aspek                            | Static IP                                      | DHCP                                                    |
| -------------------------------- | ---------------------------------------------- | ------------------------------------------------------- |
| Konsistensi untuk firewall rules | ✅ Alamat tetap, rules tidak perlu diubah       | ❌ Bisa berubah tiap lease renewal                       |
| Korelasi log/SIEM                | ✅ Mudah trace log ke host tertentu             | ❌ IP berubah bikin log historis rancu                   |
| Ketahanan terhadap gangguan DHCP | ✅ Tidak terpengaruh rogue DHCP/DHCP starvation | ❌ Rentan diserang (salah satu vektor attack umum)       |
| Kecepatan boot                   | ✅ Tidak nunggu proses DHCP handshake           | ⚠️ Bisa nambah delay kalau DHCP server lambat/tidak ada |
| Predictability saat lomba        | ✅ Kamu tahu persis IP tiap mesin               | ❌ Tidak ideal kalau dokumentasi topologi butuh IP tetap |

**Kesimpulan: pakai Static IP** untuk semua mesin yang berperan sebagai target hardening/defense di lomba. DHCP hanya masuk akal kalau environment lomba memang mengharuskannya (baca aturan teknis lomba dulu).

---

## 2. Konfigurasi Static IP (Netplan — default Ubuntu modern)

**Langkah 1 — Cek nama interface:**

```bash
ip a
```

Catat nama interface-nya (contoh: `ens33`, `eth0`, atau `enp0s3`).

**Langkah 2 — Edit file netplan:**

```bash
ls /etc/netplan/
sudo nano /etc/netplan/01-netcfg.yaml
```

**Langkah 3 — Isi konfigurasi static:**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:                      # ganti sesuai nama interface kamu
      dhcp4: no
      addresses:
        - 192.168.100.10/24     # sesuaikan dengan subnet lomba
      routes:
        - to: default
          via: 192.168.100.1
      nameservers:
        addresses: [192.168.100.1, 8.8.8.8]
```

**Langkah 4 — Terapkan konfigurasi:**

```bash
sudo netplan try      # test dulu, auto-rollback kalau gagal dalam 120 detik
sudo netplan apply    # kalau sudah yakin
```

> ⚠️ **Penting:** kalau kamu remote via SSH, selalu pakai `netplan try` dulu — bukan langsung `apply` — supaya kalau IP salah, konfigurasi otomatis rollback dan kamu tidak terkunci dari VM.

**Kalau pakai NetworkManager (Ubuntu Desktop):**

```bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.100.10/24
sudo nmcli con mod "Wired connection 1" ipv4.gateway 192.168.100.1
sudo nmcli con mod "Wired connection 1" ipv4.dns "192.168.100.1 8.8.8.8"
sudo nmcli con mod "Wired connection 1" ipv4.method manual
sudo nmcli con up "Wired connection 1"
```

---

## 3. Optimasi Boot Time (Aman untuk Blue Team)

Karena static IP tidak butuh proses DHCP handshake, `wait-online` biasanya jauh lebih cepat selesai secara natural. Tapi tetap lakukan ini:

**a. Perpendek timeout wait-online (JANGAN disable total)**

```bash
sudo systemctl edit systemd-networkd-wait-online.service
```

Isi:

```ini
[Service]
ExecStart=
ExecStart=/lib/systemd/systemd-networkd-wait-online --timeout=5
```

**b. Set interface yang tidak relevan supaya tidak ditunggu** Kalau ada interface tambahan (misal interface manajemen terpisah dari interface target), edit file `.network` terkait di `/etc/systemd/network/` dan tambahkan:

```ini
[Link]
RequiredForOnline=no
```

**c. Disable ModemManager (aman total untuk blue team)**

```bash
sudo systemctl disable ModemManager.service
sudo systemctl mask ModemManager.service
```

**d. Cek ulang setelah perubahan**

```bash
systemd-analyze blame
systemd-analyze critical-chain
```

---

## 4. Checklist Hardening Dasar Saat VM Baru Pertama Kali

Jalankan urutan ini **setiap kali** mulai dari VM baru/fresh install, sebelum masuk ke skenario lomba:

- [ ] **Update sistem**
    
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
    
- [ ] **Set static IP** (lihat bagian 2)
- [ ] **Sinkronisasi waktu** (krusial untuk korelasi log!)
    
    ```bash
    sudo apt install chrony -ysudo systemctl enable --now chronytimedatectl
    ```
    
- [ ] **Firewall baseline aktif**
    
    ```bash
    sudo ufw default deny incomingsudo ufw default allow outgoingsudo ufw allow sshsudo ufw enable
    ```
    
- [ ] **Hardening SSH**
    - Disable root login: `PermitRootLogin no` di `/etc/ssh/sshd_config`
    - Pertimbangkan disable password auth, pakai key-based auth
    - `sudo systemctl restart sshd`
- [ ] **Aktifkan logging terpusat/lengkap**
    
    ```bash
    sudo apt install auditd -ysudo systemctl enable --now auditd
    ```
    
- [ ] **Cek service yang jalan, matikan yang tidak perlu**
    
    ```bash
    systemctl list-units --type=service --state=running
    ```
    
- [ ] **Cek user account, hapus/kunci akun yang tidak perlu**
    
    ```bash
    cat /etc/passwd | grep -E '/bin/bash|/bin/sh'
    ```
    
- [ ] **Backup/snapshot VM setelah baseline siap** — ini titik aman untuk rollback kalau ada eksperimen yang gagal saat latihan.

---

## 5. Verifikasi Akhir Sebelum Dipakai Lomba

Reboot VM beberapa kali berturut-turut dan pastikan:

- [ ] Boot time konsisten cepat (catat waktunya dengan `systemd-analyze`)
- [ ] IP address tetap sama setiap boot
- [ ] Firewall tetap aktif otomatis setelah reboot
- [ ] Logging service (auditd, rsyslog) jalan otomatis
- [ ] Waktu sistem selalu akurat (chrony sync)
- [ ] Tidak ada service gagal start (`systemctl --failed`)

```bash
systemctl --failed
```

---

**Catatan akhir:** Simpan file ini dan jadikan checklist wajib tiap kali kamu setup ulang VM — baik untuk latihan mandiri maupun sebelum hari-H lomba.