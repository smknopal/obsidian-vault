
```text
[SSH] /etc/ssh/sshd_config
  Port 2222
  PermitRootLogin no
  PasswordAuthentication no
  AllowUsers lksadmin
  → sudo systemctl restart ssh

[FIREWALL] ufw
  sudo ufw enable
  sudo ufw default deny incoming
  sudo ufw allow 2222/tcp
  sudo ufw status verbose

[PASSWORD POLICY] /etc/security/pwquality.conf
  minlen=12, dcredit=-1, ucredit=-1, ocredit=-1

[FAILLOCK] /etc/security/faillock.conf
  deny=5, unlock_time=600

[AUDIT] /var/log/auth.log + journalctl
  sudo tail -f /var/log/auth.log
  grep "Failed" /var/log/auth.log

[FIND BACKDOOR]
  find / -perm -4000 -type f      # SUID
  find / -perm -002 -type f       # world-writable
  ls -la /etc/cron.*              # cron jobs
  crontab -l; sudo crontab -l
```