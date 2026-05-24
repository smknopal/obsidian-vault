## 1. Rencana Pengalamatan (IP Address Plan)

Agar rapi, kita petakan dulu semua IP yang akan digunakan.

- **LAN 1 (PC 1 ke Router 1):** Network `192.168.10.0/28` (Gateway: `.1`)
    
- **LAN 2 (PC 2 ke Router 2):** Network `192.168.20.0/28` (Gateway: `.1`)
    
- **LAN 3 (PC 3 ke Router 3):** Network `192.168.30.0/28` (Gateway: `.1`)
    
- **WAN 1 (Router 1 ke Router 2):** Network `10.10.10.0/30`
    
- **WAN 2 (Router 2 ke Router 3):** Network `10.10.20.0/30`
    

---

## 2. Konfigurasi IP Address & Interface

Langkah pertama adalah menghidupkan port dan memasukkan IP Address di setiap router.

### Router 1 (Kiri)

Menghubungkan LAN 1 dan mengarah ke Router 2.

Plaintext

```
Router> enable
Router# configure terminal
Router# hostname R1

! Setup arah ke Switch 1 (LAN 1)
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.10.1 255.255.255.240
R1(config-if)# no shutdown
R1(config-if)# exit

! Setup arah ke Router 2 (WAN 1)
R1(config)# interface fa0/1
R1(config-if)# ip address 10.10.10.1 255.255.255.252
R1(config-if)# no shutdown
```

### Router 2 (Tengah)

Router ini paling sibuk karena berada di tengah, memiliki 3 jalur: ke LAN 2, ke Router 1, dan ke Router 3.

Plaintext

```
Router> enable
Router# configure terminal
Router# hostname R2

! Setup arah ke Switch 2 (LAN 2)
R2(config)# interface fa0/0
R2(config-if)# ip address 192.168.20.1 255.255.255.240
R2(config-if)# no shutdown
R2(config-if)# exit

! Setup arah ke Router 1 (WAN 1)
R2(config)# interface fa0/1
R2(config-if)# ip address 10.10.10.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit

! Setup arah ke Router 3 (WAN 2) - misal dicolok ke fa1/0
R2(config)# interface fa1/0
R2(config-if)# ip address 10.10.20.1 255.255.255.252
R2(config-if)# no shutdown
```

### Router 3 (Kanan)

Menghubungkan LAN 3 dan mengarah kembali ke Router 2.

Plaintext

```
Router> enable
Router# configure terminal
Router# hostname R3

! Setup arah ke Switch 3 (LAN 3)
R3(config)# interface fa0/0
R3(config-if)# ip address 192.168.30.1 255.255.255.240
R3(config-if)# no shutdown
R3(config-if)# exit

! Setup arah ke Router 2 (WAN 2)
R3(config)# interface fa0/1
R3(config-if)# ip address 10.10.20.2 255.255.255.252
R3(config-if)# no shutdown
```

---

## 3. Konfigurasi Routing RIP version 2

**Mengapa harus menggunakan RIPv2?** Jika menggunakan routing statis, kamu harus mendaftarkan rute yang tidak terhubung langsung secara manual satu per satu (misal: R1 harus disetting manual agar tahu cara ke LAN 3). Semakin banyak router, ini akan sangat merepotkan. Dengan protokol dinamis seperti RIP, kamu hanya perlu "mendaftarkan" jaringan yang menempel langsung di router tersebut. Router akan saling bertukar informasi secara otomatis. Versi 2 digunakan agar router membaca _subnet mask_ (`/28` dan `/30`) dengan benar, bukan sekadar menebak berdasarkan kelas IP.

### Konfigurasi di R1

Hanya kenalkan jaringan yang terhubung ke R1 (LAN 1 dan WAN 1).

Plaintext

```
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 192.168.10.0
R1(config-router)# network 10.10.10.0
R1(config-router)# no auto-summary
```

### Konfigurasi di R2

Kenalkan ketiga jaringan yang terhubung ke R2 (LAN 2, WAN 1, dan WAN 2).

Plaintext

```
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 192.168.20.0
R2(config-router)# network 10.10.10.0
R2(config-router)# network 10.10.20.0
R2(config-router)# no auto-summary
```

### Konfigurasi di R3

Hanya kenalkan jaringan yang terhubung ke R3 (LAN 3 dan WAN 2).

Plaintext

```
R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# network 192.168.30.0
R3(config-router)# network 10.10.20.0
R3(config-router)# no auto-summary
```

---

## 4. Setting IP Address di PC Client

Masuk ke menu **Desktop > IP Configuration** pada masing-masing PC. Pastikan Subnet Mask berakhiran `.240` karena kita memakai prefix `/28`.

- **PC 1**
    
    - IP Address: `192.168.10.2`
        
    - Subnet Mask: `255.255.255.240`
        
    - Default Gateway: `192.168.10.1`
        
- **PC 2**
    
    - IP Address: `192.168.20.2`
        
    - Subnet Mask: `255.255.255.240`
        
    - Default Gateway: `192.168.20.1`
        
- **PC 3**
    
    - IP Address: `192.168.30.2`
        
    - Subnet Mask: `255.255.255.240`
        
    - Default Gateway: `192.168.30.1`
        

---

## 5. Pengujian (Testing)

1. Buka PC 1, masuk ke **Command Prompt**.
    
2. Ketikkan `ping 192.168.30.2` (mencoba menghubungi PC 3).
    
3. **Wajar jika 1-2 baris pertama muncul pesan "Request Timed Out" (RTO)** karena jaringan sedang mempelajari rute MAC Address (_proses ARP_).
    
4. Tunggu sejenak, jika konfigurasi benar, baris berikutnya akan menampilkan pesan **Reply from 192.168.30.2**.