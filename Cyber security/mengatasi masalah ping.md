### Catatan Mengatasi Masalah Ping dan Koneksi Internet Terblokir Firewall

1. **Cek Status Firewall (UFW)**
    - Jalankan:
        
        ```text
        sudo ufw status verbose
        ```
        
    - Periksa apakah ada aturan yang memblokir koneksi keluar atau paket ICMP (ping).

2. **Izinkan Koneksi Keluar dan Ping di Firewall**
    - Set default policy untuk koneksi keluar menjadi allow:
        
        ```text
        sudo ufw default allow outgoing
        ```
        
    - Tambahkan aturan untuk mengizinkan paket ICMP (ping) dengan mengedit file `/etc/ufw/before.rules` dan tambahkan:
        
        ```text
        -A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
        ```
        
    - Reload UFW agar aturan baru diterapkan:
        
        ```text
        sudo ufw reload
        ```
        

3. **Cek Konfigurasi Jaringan VM atau Sistem**
    - Pastikan adapter jaringan VM diatur ke NAT atau Bridged Adapter agar bisa akses internet.
    - Cek IP dan gateway dengan:
        
        ```text
        ip a
        ip route
        ```
        
    - Pastikan ada IP valid dan default gateway.

4. **Cek DNS**
    - Pastikan file `/etc/resolv.conf` berisi nameserver yang valid, misalnya:
        
        ```text
        nameserver 8.8.8.8
        ```
        

5. **Tes Koneksi**
    - Coba ping gateway lokal:
        
        ```text
        ping <ip_gateway>
        ```
        
    - Coba ping IP publik Google:
        
        ```text
        ping 8.8.8.8
        ```
        
    - Coba ping nama domain Google:
        
        ```text
        ping google.com
        ```
        

6. **Restart Network Service Jika Perlu**
    - Jalankan:
        
        ```text
        sudo netplan apply
        ```
        
        atau
        
        ```text
        sudo systemctl restart NetworkManager
        ```
        

7. **Jika Masih Bermasalah**
    - Cek apakah ada firewall lain di host atau jaringan yang memblokir koneksi.
    - Coba restart VM dan host.
    - Ubah mode jaringan VM (misal dari NAT ke Bridged).

---

### Catatan Penting

- Jangan matikan firewall sepenuhnya sebagai solusi permanen karena akan membuka celah keamanan.
- Selalu sesuaikan aturan firewall agar koneksi penting tetap diizinkan.
- Pastikan konfigurasi jaringan VM dan DNS sudah benar agar koneksi internet lancar.