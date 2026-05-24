**A. Navigasi Linux Dasar:**
pwd, ls -la, cd, cat, nano, sudo
cp, mv, rm, mkdir, rmdir
find / -name "file"
grep "kata" file
ps aux | grep proses
netstat -tlnp  atau  ss -tlnp
systemctl status/start/stop/restart/enable

**B. User & Permission:**
whoami, id, who
useradd, usermod, userdel, passwd
chmod 644 file  (read-write owner, read group, read other)
chown user:group file
sudo visudo      # edit sudoers

**C. Network Basic:**
ip a               # lihat IP
ip route           # lihat routing
ping, traceroute, dig
ufw enable/disable/status/allow/deny

**D. Log Investigation:**
tail -f /var/log/auth.log       # live log auth
journalctl -u ssh               # log service
grep "Failed" /var/log/auth.log # cari gagal login
last                             # history login
