1. CEK USER SUSPICIOUS:
   awk -F: '($3 == 0) {print}' /etc/passwd    # cari UID 0 (root) selain root
   awk -F: '($2 == "") {print $1}' /etc/shadow # user tanpa password

2. CEK SUID BERBAHAYA:
   find / -perm -4000 -type f 2>/dev/null

3. CEK WORLD-WRITABLE:
   find / -perm -002 -type f 2>/dev/null

4. CEK PROSES MENCURIGAKAN:
   ps auxf                    # semua proses tree
   ss -tlnp                   # apa yang listening

5. CEK LOGIN TERAKHIR:
   last -a | head -20         # login sukses
   lastb -a | head -20        # login gagal (butuh root)

6. CEK CRON BACKDOOR:
   ls -la /etc/cron.*
   cat /etc/crontab
   for user in $(cut -f1 -d: /etc/passwd); do sudo crontab -u $user -l 2>/dev/null; done

7. CEK FILE YANG BARU DIMODIFIKASI:
   find / -mtime -1 -type f 2>/dev/null    # modifikasi 1 hari terakhir

8. CEK INSTALLED PACKAGES:
   dpkg -l | grep -i suspicious

9. CEK NETWORK CONNECTION AKTIF:
   ss -tnp                    # TCP established
   netstat -antp              # alternatif

10. MONITOR REAL-TIME:
    sudo tail -f /var/log/auth.log &
    sudo tail -f /var/log/syslog &
    htop
