Markdown

````
# 🛡️ Linux Hardening: Misconfigurations & Audit Logging (Versi Penjelasan Lengkap)
**Peran:** Defender
**Konteks:** LKS Cyber Security 
**OS:** Debian/Ubuntu-based

---

## 3. Common Linux Misconfigurations
**Tujuan Utama:** Mencegah *Privilege Escalation* (mencegah *hacker* kelas teri yang sudah masuk jadi *user* biasa agar tidak bisa berevolusi menjadi `root`).

### A. Mengamankan File Kritis (/etc/passwd dan /etc/shadow)
Dua *file* ini adalah jantung dari akun Linux. `/etc/passwd` isinya daftar *user*, `/etc/shadow` isinya *password* yang dienkripsi.

**1. Set Pemilik File (chown)**
```bash
sudo chown root:root /etc/passwd /etc/shadow
````

> **Apa gunanya `chown`?** `chown` singkatan dari _Change Owner_. Perintah ini memastikan bahwa pemilik utama (Owner) dan grup (Group) dari kedua _file_ ini adalah `root`. Kalau _file_ ini diam-diam dimiliki oleh _user_ `www-data` (user web server), _hacker_ yang meretas web bisa langsung mengganti _password_ server!

**2. Set Izin File (chmod)**

Bash

```
sudo chmod 644 /etc/passwd
sudo chmod 600 /etc/shadow
```

> **Kenapa angkanya 644 dan 600?**
> 
> - `644 (/etc/passwd)`: `Root` boleh baca & edit (6). Orang lain cuma boleh baca (4). Kenapa orang lain boleh baca? Karena sistem butuh membaca _file_ ini untuk tahu nama-nama _user_ yang ada.
>     
> - `600 (/etc/shadow)`: **HANYA** `root` yang boleh baca & edit (6). Orang lain tidak boleh ngapa-ngapain (0). Ini krusial karena isinya adalah _hash password_.
>     

### B. Memburu SUID / SGID Biner yang Berbahaya

SUID adalah fitur Linux semacam "ID Card VIP sementara". Kalau sebuah program punya izin SUID, maka siapapun yang menjalankan program itu akan **mendadak meminjam kekuatan pemilik aslinya** (biasanya `root`).

**1. Cari semua file SUID:**

Bash

```
sudo find / -perm -4000 -type f 2>/dev/null
```

**2. Apa yang harus dicari dari hasil pencarian ini?** Kamu akan melihat daftar panjang. Jangan panik.

- **Yang NORMAL (Abaikan):** `/usr/bin/sudo`, `/usr/bin/passwd`, `/usr/bin/su`. Ini wajar karena _user_ biasa memang butuh akses sementara untuk ganti _password_ atau pindah akun.
    
- **Yang BERBAHAYA (Incaranmu):** Kalau kamu melihat aplikasi biasa seperti `/usr/bin/find`, `/usr/bin/nmap`, `/bin/bash`, `/usr/bin/vim`, atau `/usr/bin/cp` ada di daftar itu, **ITU ADALAH BACKDOOR!** _Hacker_ sengaja memberi SUID pada `vim`, supaya mereka bisa buka dan edit `/etc/shadow` pakai `vim` tanpa harus jadi `root`.
    

**3. Eksekusi: Cabut Izin SUID** Maksud dari "eksekusi" di sini adalah melumpuhkan bahayanya. Kita cabut "ID Card VIP" dari program yang mencurigakan tadi.

Bash

```
# Contoh: Jika kamu menemukan /usr/bin/vim punya SUID
sudo chmod u-s /usr/bin/vim
```

> **Apa gunanya `u-s`?** `User minus SUID`. Kita menghapus huruf `s` (izin khusus) dari program tersebut. Sekarang `vim` kembali jadi program normal yang tidak berbahaya.

---

## 4. Logging & Auditing Dasar (Persiapan SIEM)

**Tujuan Utama:** Memasang CCTV super detail di _file_ penting. Kalau ada yang mengotak-atik, kita punya bukti kuat.

### A. Aktifkan CCTV (Auditd)

Bash

```
sudo apt update && sudo apt install auditd -y
sudo systemctl enable --now auditd
```

### B. Pasang Rule (Aturan Pemantauan)

Kita beritahu CCTV ini untuk fokus mengawasi brankas _password_.

Bash

```
sudo auditctl -w /etc/shadow -p wa -k maling_shadow
```

> **Apa maksud Rule ini?**
> 
> - `-w /etc/shadow`: **Watch** (Awasi) _file_ `/etc/shadow`.
>     
> - `-p wa`: **Permissions Write & Attribute**. Rekam aktivitas HANYA jika ada yang mencoba menulis/mengedit (`w`) atau mengubah izin/hak cipta (`a`) _file_ tersebut. Kalau cuma sekadar membaca, abaikan saja.
>     
> - `-k maling_shadow`: **Key**. Ini seperti memberi _hashtag_ (tag) pada rekaman. Tujuannya biar kita gampang mencarinya nanti.
>     

### C. Cara Membaca Log (Penting untuk Threat Hunting)

_Hacker_ berhasil masuk dan mengubah _password_ diam-diam. Kita cari rekamannya menggunakan _hashtag_ yang kita buat:

Bash

```
sudo ausearch -k maling_shadow
```

**Cara membaca outputnya yang ruwet:** Nanti akan muncul teks panjang. Fokus pada kata-kata kunci ini saja di dalam blok teks tersebut:

- `time->`: Kapan kejadiannya? (contoh: `time->Wed Apr 29 23:45:12 2026`).
    
- `syscall=`: Apa yang dia lakukan? (Biasanya angka, atau tulisan `open`/`rename` yang menandakan dia membuka atau mengganti nama _file_).
    
- `success=`: Apakah dia berhasil? (`yes` = berhasil diedit, `no` = gagal karena kehalang _firewall_/_permissions_).
    
- `exe=`: Program apa yang dia pakai buat ngedit? (contoh: `exe="/usr/bin/nano"`, oh dia pakai Nano!).
    
- `uid=`: **Siapa pelakunya?** (Cari angka ini. Kalau `uid=1000`, berarti _user_ biasa yang melakukannya. Cocokkan angka 1000 ini di `/etc/passwd` untuk tahu nama _user_-nya).