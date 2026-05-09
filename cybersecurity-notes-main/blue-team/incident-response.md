# Incident Response — Hodisaga Munosabat

## IR nima?

Incident Response (IR) — kiberhujum yoki xavfsizlik hodisasini aniqlash, tekshirish va yo'q qilish jarayoni.

## NIST IR Jarayoni (4 bosqich)

```
1. PREPARATION     — Tayyorgarlik
2. DETECTION       — Aniqlash va Tahlil
3. CONTAINMENT     — Izolyatsiya va Yo'q Qilish
4. RECOVERY        — Tiklash va O'rganish
```

---

## 1. Tayyorgarlik (Preparation)

```
□ IR jamoasi tuzilgan
□ Aloqa rejasi mavjud
□ Zaxira nusxalar mavjud
□ Loglar to'planmoqda (SIEM)
□ IDS/IPS o'rnatilgan
□ Incident Response rejasi yozilgan
□ Forensics vositalari tayyorlangan
```

---

## 2. Aniqlash va Tahlil (Detection & Analysis)

### Hujum ko'rsatkichlari (IoC — Indicators of Compromise)

```
Tarmoq IoC:
- Noma'lum IP/domenga ulanishlar
- Katta hajmli ma'lumot chiqishi (exfiltration)
- G'ayriodddiy vaqtda trafik (tunda)
- DNS so'rovlari noodatiy nomlar

Tizim IoC:
- Noma'lum jarayonlar
- Yangi admin hisoblar
- Scheduled task o'zgarishlari
- Kriptolangan fayllar (ransomware)
- /tmp yoki C:\Temp da bajariladigan fayllar
```

### Linux da Tezkor Tekshiruv

```bash
# Joriy ulanishlar
ss -tuln
netstat -antp

# So'nggi loginlar
last
lastlog
who

# Shubhali jarayonlar
ps aux --sort=-%cpu | head -20
ps aux | grep -v "root\|USER" | sort -k3 -rn

# Noma'lum cron
crontab -l
cat /etc/crontab
ls -la /etc/cron.*

# So'nggi o'zgartirilgan fayllar
find / -mtime -1 -type f 2>/dev/null | grep -v /proc
find /tmp /var/tmp -type f -newer /tmp 2>/dev/null

# SUID fayllar
find / -perm -4000 2>/dev/null

# Tarmoq ulanishlari
ss -antp | grep ESTABLISHED

# Ochiq fayllar
lsof -i
lsof +D /tmp
```

### Windows da Tezkor Tekshiruv

```powershell
# Jarayonlar
Get-Process | Sort-Object CPU -Descending | Select -First 20
tasklist /v

# Tarmoq ulanishlari
netstat -anb
Get-NetTCPConnection | Where-Object State -eq 'Established'

# So'nggi o'rnatilgan dasturlar
Get-WinEvent -LogName System | 
  Where-Object {$_.Id -eq 11724} | Select -First 20

# Startup
Get-CimInstance Win32_StartupCommand

# Scheduled tasks
Get-ScheduledTask | Where-Object State -eq 'Ready'

# Event loglar
Get-WinEvent -LogName Security -MaxEvents 100 | 
  Where-Object {$_.Id -in 4624,4625,4648} | 
  Format-List TimeCreated, Message
```

---

## 3. Izolyatsiya va Bartaraf Etish (Containment & Eradication)

```bash
# Tarmoqdan ajratish (Linux)
iptables -I INPUT -j DROP
iptables -I OUTPUT -j DROP

# Tarmoqdan ajratish (Windows)
netsh advfirewall set allprofiles state on
netsh advfirewall firewall add rule name="Block All" \
  protocol=ANY dir=in action=block

# Shubhali jarayonni to'xtatish
kill -9 PID
taskkill /F /PID xxxx

# Shubhali foydalanuvchini bloklash
passwd -l username         # Linux
net user username /active:no  # Windows

# Backdoor fayllarni topish
find /var/www -name "*.php" -newer /var/www/index.php
grep -r "eval(base64_decode" /var/www/
grep -r "system($_" /var/www/
```

---

## 4. Tiklash va O'rganish (Recovery & Lessons Learned)

```
Tiklash:
□ Toza zaxiradan tiklash
□ Patch qo'llash
□ Parollarni o'zgartirish
□ Sertifikatlarni yangilash
□ Monitoring kuchaytirish

O'rganish (Post-Incident Report):
□ Qanday kirildi?
□ Nima zarar ko'rildi?
□ Qancha vaqt davom etdi?
□ Qanday aniqlanadi?
□ Oldini olish uchun nima qilish kerak?
```

---

## DFIR Vositalari

```bash
# Memory dump
avml /tmp/memory.lime       # Linux
winpmem_mini.exe memory.raw # Windows

# Disk image
dd if=/dev/sda of=/external/disk.img bs=4M

# Volatility — memory tahlil
volatility -f memory.lime --profile=Linux imageinfo
volatility -f memory.lime --profile=Linux pslist
volatility -f memory.lime --profile=Linux netscan

# Log tahlil
grep "Failed password" /var/log/auth.log | wc -l
grep "Accepted" /var/log/auth.log | tail -20

# Fayl hash tekshirish
md5sum -c checksums.md5
sha256sum important_file.bin
```

## Common Attack Patterns

```
Lateral Movement:
- Pass the Hash
- Pass the Ticket (Kerberos)
- RDP / SSH orqali

Persistence:
- Cron job
- Systemd service
- Registry Run key (Windows)
- Scheduled Task

Exfiltration:
- DNS tunneling
- HTTPS orqali
- Cloud storage (Dropbox, Google Drive)
```
