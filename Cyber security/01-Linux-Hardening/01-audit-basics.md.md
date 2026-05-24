Total User: 34 ( cat /etc/passwd | wc -l)

(awk -F: '($3 == 0) {print $1}' /etc/passwd)
hanya root

(grep -E "/bin/(bash|sh|zsh)$" /etc/passwd)
root, lks admin

(getent group sudo)
lksadmin

(sudo awk -F: '($2 == "") {print $1}' /etc/shadow)
output pass kosong

```markdown
- Total user: 34
- UID 0: root
- User dengan shell login: root, lks admin
- User sudo: lksadmin
- Password kosong: ya

### Red Flag Check
- [ ] Ada UID 0 selain root? → JIKA YA, investigate!
- [ ] NOPASSWD di sudoers? → JIKA YA, hapus!
- [ ] Password kosong? → JIKA YA, lock user!

### Command Penting (hafal!)
1. `awk -F: '($3==0)' /etc/passwd`
2. `getent group sudo`
3. `sudo grep NOPASSWD /etc/sudoers`
```

(ss -tlnp)
port 22

systemctl list-unit-files --type=service --state=enabled | head -30
tidak ada dir yang auto start

systemctl list-units --type=service --state=running | head -20

tidak ada

```bash
sudo ls -la /etc/cron.*/
sudo cat /etc/crontab
crontab -l 2>/dev/null
sudo crontab -l -u root 2>/dev/null
```
tidak ada yang aneh

## Eksperimen 2: Audit Service

### Port Listening
State       Recv-Q      Send-Q           Local Address:Port           Peer Address:Port     Process
LISTEN      0           4096                127.0.0.54:53                  0.0.0.0:*
LISTEN      0           4096                   0.0.0.0:22                  0.0.0.0:*
LISTEN      0           4096             127.0.0.53%lo:53                  0.0.0.0:*
LISTEN      0           4096                      [::]:22                     [::]:*

### Service Enabled (berapa banyak?)
Total: 55

### Cron Jobs Found
- /etc/crontab: 0
- Root crontab: 0

### Command Penting
1. `ss -tlnp` → port listening
2. `systemctl list-unit-files --state=enabled`
3. `sudo ls -la /etc/cron.*/`

(last -a | head -10)
lksadmin

(sudo lastb -a | head -10)
command not found

(sudo tail -30 /var/log/auth.log)
berisi hasil perintah yang terakhir dijalankan

```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
```
berisi list ip yang paling sering gagal

```bash
sudo find /etc /home /root /var/www -mtime -1 -type f 2>/dev/null
```
melihat file terakhir yang baru diubah

(find / -perm -4000 -type f 2>/dev/null)
cek file SUID

## Eksperimen 3: Audit Log

### Login Sukses Terakhir
lksadmin pts/0        Sun Apr 26 03:29 - still logged in    192.168.56.1

### Failed Login Attempts
tidak ada
### File Modified Last Day
tidak ada

### SUID Binaries Count
Total: 15

### Command Penting
1. `last -a` + `sudo lastb -a`
2. `sudo grep "Failed password" /var/log/auth.log`
3. `find / -perm -4000 -type f 2>/dev/null`
