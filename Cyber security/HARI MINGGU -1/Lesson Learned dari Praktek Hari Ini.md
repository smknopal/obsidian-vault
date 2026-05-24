
### Hal yang Bikin Salah Interpretasi
1. "Output kosong" - artinya TIDAK ADA hasil, bukan "tidak tau"
2. `head -30` memotong output - kalau butuh total, pakai `wc -l`

### Command Baru yang Perlu Di-include
- `systemctl list-unit-files --type=service --state=enabled | wc -l` (count)
- `sudo touch /var/log/btmp && sudo chmod 600 /var/log/btmp` (enable lastb)
- Bookmark: https://gtfobins.github.io (SUID abuse reference)

### Red Flag Checklist (print ini!)
- [ ] UID 0 selain root → BACKDOOR
- [ ] User password kosong di shadow → DANGER
- [ ] SUID di /tmp, /home → BACKDOOR
- [ ] Port listening aneh (4444, 6666, 31337) → REVERSE SHELL
- [ ] Cron job di /etc/cron.* dengan nama aneh → PERSISTENCE
- [ ] File baru modified di /etc, /bin, /sbin → TAMPER
