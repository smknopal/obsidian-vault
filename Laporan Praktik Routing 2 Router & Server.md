

## 1. Analisis Jaringan

- **LAN A:** 192.168.10.0/28 (Host: 192.168.10.2 - .14, Gateway: .1)
    
- **LAN B:** 192.168.20.0/28 (Host: 192.168.20.2 - .14, Gateway: .1)
    
- **WAN (Antar Router):** 10.10.10.0/30 (R1: .1, R2: .2)
    
- **Server Zone:** 192.168.30.0/28 (Server: 192.168.30.2, Gateway: .1)
    

## 2. Topologi & Kabel

- **Router to Router:** FastEthernet 0/1 ↔ FastEthernet 0/1 (**Kabel Cross-over**)
    
- **Router to Server:** FastEthernet 0/2 ↔ Server NIC (**Kabel Cross-over**)
    
- **Router to Switch:** FastEthernet 0/0 ↔ Switch (**Kabel Straight-through**)
    

---

## 3. Langkah Konfigurasi (CLI)

### A. Router 1 (Kantor A & Server)

Plaintext

```
enable
conf t

! Konfigurasi ke arah Switch LAN A
interface fa0/0
 ip address 192.168.10.1 255.255.255.240
 no shutdown

! Konfigurasi ke arah Router 2
interface fa0/1
 ip address 10.10.10.1 255.255.255.252
 no shutdown

! Konfigurasi ke arah Server
interface fa0/2
 ip address 192.168.30.1 255.255.255.240
 no shutdown
exit

! Konfigurasi Routing RIP v2
router rip
 version 2
 no auto-summary
 network 192.168.10.0
 network 10.10.10.0
 network 192.168.30.0
exit
```

### B. Router 2 (Kantor B)

Plaintext

```
enable
conf t

! Konfigurasi ke arah Switch LAN B
interface fa0/0
 ip address 192.168.20.1 255.255.255.240
 no shutdown

! Konfigurasi ke arah Router 1
interface fa0/1
 ip address 10.10.10.2 255.255.255.252
 no shutdown
exit

! Konfigurasi Routing RIP v2
router rip
 version 2
 no auto-summary
 network 192.168.20.0
 network 10.10.10.0
exit
```

---

## 4. Konfigurasi End-Device (PC & Server)

|**Perangkat**|**IP Address**|**Subnet Mask**|**Default Gateway**|
|---|---|---|---|
|**PC Kantor A**|192.168.10.2|255.255.255.240|192.168.10.1|
|**PC Kantor B**|192.168.20.2|255.255.255.240|192.168.20.1|
|**Server**|192.168.30.2|255.255.255.240|192.168.30.1|

---

## 5. Uji Koneksi & Verifikasi

1. **Ping PC ke Server:** Dari PC Kantor B, lakukan `ping 192.168.30.2`.
    
2. **Web Browser:** Dari PC mana pun, buka Web Browser dan akses `192.168.30.2`.
    
3. **Cek Tabel Routing:** Di Router, ketik `show ip route`. Pastikan ada kode **R** untuk jaringan tetangga.
    

---

## 6. Analisis Troubleshooting

- **Lampu Merah:** Salah jenis kabel (gunakan Cross-over untuk Router-Router/Router-Server) atau lupa perintah `no shutdown`.
    
- **Request Timed Out (RTO):** Biasanya karena salah mengisi **Default Gateway** di PC atau lupa mendaftarkan network di `router rip`.
    
- **Dampak Router Mati:** Jika salah satu router mati, komunikasi antar kantor akan terputus total karena router adalah satu-satunya gerbang penghubung.