```markdown
## 3 Pertanyaan Saat Dikasih Server

### 1. "Siapa yang bisa LOGIN?"
- cat /etc/passwd           → semua user
- awk -F: '($3==0)' /etc/passwd  → user UID 0 (root only!)
- getent group sudo         → siapa bisa sudo
- sudo cat /etc/sudoers     → cek NOPASSWD (DANGER!)
- sudo ls /etc/sudoers.d/   → config sudo custom

### 2. "Apa yang SEDANG JALAN?"
- ss -tlnp                  → port listening
- systemctl list-unit-files --state=enabled → auto-start service
- ps auxf                   → semua proses tree

### 3. "Di mana JEJAKNYA?"
- last -a                   → login sukses
- sudo lastb -a             → login gagal (brute force?)
- sudo tail /var/log/auth.log
- find / -mtime -1 -type f 2>/dev/null  → file baru diubah

---

## Pola Universal: Setiap Service = 3 Komponen

| Komponen | Lokasi |
|---|---|
| Config | /etc/<nama>/ |
| Service | systemctl <nama> |
| Log | /var/log/<nama>* atau journalctl -u <nama> |

**Ritual 4 langkah kalau konfigurasi service:**
1. sudo nano <config_file>
2. sudo systemctl restart <service>
3. sudo systemctl status <service>
4. sudo tail -f <log_file>  (verify works)
```
