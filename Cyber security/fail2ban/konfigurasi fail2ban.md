
pastikan buat salinan sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

edit jail.local
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 600
findtime = 600

cek status nya apakah nyambung ke sshd
sudo fail2ban-client status

cek aktivitas fail2ban
sudo tail -f /var/log/fail2ban.log