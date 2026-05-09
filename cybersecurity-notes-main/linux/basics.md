# Linux Asoslari — Kibersecurity uchun

## Nima uchun Linux?

Kibersecurity mutaxassislarining 90%+ ish quroli Linux. Kali Linux, Parrot OS, Ubuntu Server — barchasi Linux asosida.

## Fayl Tizimi Tuzilmasi

```
/               — Ildiz katalog
├── bin/        — Asosiy buyruqlar (ls, cp, mv)
├── sbin/       — Tizim buyruqlari (root uchun)
├── etc/        — Konfiguratsiya fayllari
├── home/       — Foydalanuvchi papkalari (/home/ali)
├── root/       — Root foydalanuvchi papkasi
├── var/        — O'zgaruvchan ma'lumotlar (loglar, veb)
├── tmp/        — Vaqtinchalik fayllar
├── opt/        — Qo'shimcha dasturlar
├── proc/       — Jarayonlar (virtual)
├── dev/        — Qurilmalar (/dev/sda, /dev/null)
├── usr/        — Foydalanuvchi dasturlari
└── lib/        — Kutubxonalar
```

## Ruxsatlar (Permissions)

```bash
ls -la
# -rwxr-xr-- 1 ali users 1234 Jan 1 myfile

# Format: [type][owner][group][others]
# type:  - (fayl), d (katalog), l (link)
# r = read (4), w = write (2), x = execute (1)

# Ruxsat o'zgartirish
chmod 755 file.sh    # rwxr-xr-x
chmod +x script.sh   # execute qo'shish
chmod u+w file.txt   # owner ga yozish qo'shish

# Egasini o'zgartirish
chown ali:users file.txt
chown -R ali /home/ali/  # rekursiv
```

### SUID/SGID/Sticky Bit
```bash
# SUID — fayl owner huquqi bilan ishlaydi (privesc uchun muhim!)
chmod u+s /usr/bin/program
# ls da: -rwsr-xr-x

# GUID
chmod g+s /shared/dir

# Sticky bit — faqat egasi o'chira oladi
chmod +t /tmp

# SUID fayllarni topish (privesc razvedkasi)
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

## Foydalanuvchilar va Guruhlar

```bash
# Joriy foydalanuvchi
whoami
id

# Barcha foydalanuvchilar
cat /etc/passwd

# Guruhlar
cat /etc/group

# Foydalanuvchi qo'shish
useradd -m newuser
passwd newuser

# Guruhga qo'shish
usermod -aG sudo username

# sudo imkoniyatlari
sudo -l          # nima ishlatsa bo'ladi
cat /etc/sudoers
```

## Jarayonlar

```bash
# Jarayonlar ro'yxati
ps aux
ps aux | grep nginx

# Real vaqtda
top
htop

# Jarayonni to'xtatish
kill PID
kill -9 PID    # majburan
killall nginx

# Fon jarayoni
command &
nohup command &

# Jarayonning fayllarini ko'rish
ls -la /proc/PID/
```

## Tarmoq Buyruqlari

```bash
# Interfeys ma'lumotlari
ip a
ip addr show
ifconfig

# Marshrutlash jadvali
ip route
netstat -rn

# Ochiq portlar
ss -tuln
netstat -tuln
netstat -tulnp   # jarayon nomi ham

# Ulanishlar
ss -tn
netstat -tn

# Firewall
iptables -L
ufw status
```

## Loglar

```bash
# Tizim loglari
tail -f /var/log/syslog
tail -f /var/log/auth.log    # Login urinishlari
cat /var/log/apache2/access.log
cat /var/log/nginx/access.log

# Systemd loglar
journalctl -f
journalctl -u nginx
journalctl --since "1 hour ago"

# Oxirgi loginlar
last
lastlog
who
w
```

## Muhim Fayllar (Xavfsizlik uchun)

```bash
/etc/passwd      — Foydalanuvchilar
/etc/shadow      — Shifrlangan parollar (root kerak)
/etc/sudoers     — Sudo ruxsatlar
/etc/crontab     — Vaqtli vazifalar
~/.bash_history  — Buyruqlar tarixi
~/.ssh/          — SSH kalitlar
/etc/hosts       — Lokal DNS
/etc/resolv.conf — DNS server sozlamalari
```

## Cron — Vaqtli Vazifalar

```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Hafta kuni (0-7, 0=Yakshanba)
│ │ │ └─── Oy (1-12)
│ │ └───── Kun (1-31)
│ └─────── Soat (0-23)
└───────── Daqiqa (0-59)
```

```bash
# Cron vazifalarini ko'rish
crontab -l
cat /etc/crontab
ls /etc/cron.*

# Cron tahrirlash
crontab -e

# Misol: har 5 daqiqada
*/5 * * * * /home/ali/script.sh
```
