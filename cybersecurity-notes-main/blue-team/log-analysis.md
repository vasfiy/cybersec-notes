# Log Tahlili — SOC Analyst uchun

## Nima uchun Log Tahlili?

SOC Analitikining asosiy ishi — loglardan **anomaliya** va **hujum belgilarini** topish. "Log tahlil qila olmasang, SOC analitik bo'la olmaysan."

---

## Windows Event Loglar

### Log Joylashuvi

```
Event Viewer (GUI): Win+R → eventvwr.msc

Fayl joylashuvi:
C:\Windows\System32\winevt\Logs\
├── Security.evtx      — Xavfsizlik hodisalari (eng muhim)
├── System.evtx        — Tizim hodisalari
├── Application.evtx   — Ilova hodisalari
└── Microsoft-Windows-PowerShell%4Operational.evtx
```

### SOC uchun Eng Muhim Event IDlar

#### Autentifikatsiya
```
4624 — Muvaffaqiyatli login
  Logon Types:
  2  = Interactive (to'g'ridan to'g'ri)
  3  = Network (SMB, RPC)
  4  = Batch
  5  = Service
  7  = Unlock
  8  = NetworkCleartext (Cleartext parol!)
  9  = NewCredentials (RunAs)
  10 = RemoteInteractive (RDP)
  11 = CachedInteractive (offline)

4625 — Muvaffaqiyatsiz login
  Sub Status kodlari:
  0xC000006A — Parol noto'g'ri
  0xC0000064 — Username yo'q
  0xC0000234 — Hisob bloklangan
  0xC000006F — Vaqt cheklov

4634/4647 — Logout
4648       — Aniq credential bilan login (RunAs, sched task)
4672       — Admin huquqlar bilan login (SeDebugPrivilege, ...)
```

#### Hisob Boshqaruvi
```
4720 — Yangi foydalanuvchi yaratildi
4722 — Hisob yoqildi
4724 — Admin parolni reset qildi
4725 — Hisob o'chirildi
4726 — Foydalanuvchi o'chirildi
4728 — Domain Security guruhiga qo'shildi
4732 — Local Administrators guruhiga qo'shildi ⚠️
4756 — Universal guruhga qo'shildi
4740 — Hisob bloklandi ⚠️
```

#### Jarayon va Skript
```
4688 — Yangi jarayon yaratildi (Process Creation)
4689 — Jarayon tugatildi
4104 — PowerShell ScriptBlock Logging
4103 — PowerShell Module Logging
```

#### Xizmat va Task
```
7045 — Yangi xizmat o'rnatildi ⚠️
7034 — Xizmat kutilmagan to'xtadi
4698 — Scheduled Task yaratildi ⚠️
4702 — Scheduled Task yangilandi
```

#### Tarmoq ulashish
```
5140 — Network share ulandi
5145 — Network share ob'ektiga kirish tekshirildi
```

### PowerShell bilan Tahlil

```powershell
# Oxirgi 100 muvaffaqiyatsiz login
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 100 | Select-Object TimeCreated, Message

# Muayyan foydalanuvchi logini
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} | Where-Object { $_.Message -match "username_here" }

# Yangi xizmatlar (oxirgi 24 soat)
$since = (Get-Date).AddHours(-24)
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Id = 7045
    StartTime = $since
}

# Admin guruhiga qo'shilganlar
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4732
} | Select-Object -First 20 | Format-List

# PowerShell loglar
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} | Where-Object { $_.Message -match "base64\|invoke-expression\|iex\|downloadstring" }
```

---

## Linux Loglar

### Muhim Log Fayllari

```bash
/var/log/auth.log        — SSH, sudo, login (Debian/Ubuntu)
/var/log/secure          — SSH, sudo, login (RHEL/CentOS)
/var/log/syslog          — Umumiy tizim (Debian/Ubuntu)
/var/log/messages        — Umumiy tizim (RHEL/CentOS)
/var/log/kern.log        — Kernel loglar
/var/log/cron            — Cron vazifalar
/var/log/apache2/        — Apache veb server
/var/log/nginx/          — Nginx veb server
/var/log/fail2ban.log    — Brute force himoyasi
/var/log/audit/audit.log — Auditd (kengaytirilgan)
```

### Muhim Log Namunalar

```bash
# Muvaffaqiyatli SSH login
Jan 15 10:30:15 server sshd[1234]: Accepted password for ali from 192.168.1.100 port 54321 ssh2

# Muvaffaqiyatsiz SSH login
Jan 15 10:30:20 server sshd[1235]: Failed password for invalid user admin from 45.33.32.156 port 11835 ssh2

# Sudo ishlatish
Jan 15 10:35:00 server sudo: ali : TTY=pts/0 ; PWD=/home/ali ; USER=root ; COMMAND=/bin/bash

# Yangi foydalanuvchi
Jan 15 11:00:00 server useradd[5678]: new user: name=hacker, UID=1001, GID=1001, ...
```

### Tahlil Buyruqlari

```bash
# SSH brute force aniqlash
grep "Failed password" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head 20

# Muvaffaqiyatli loginlar
grep "Accepted" /var/log/auth.log | \
  awk '{print $1, $2, $3, $9, $11}' | tail -20

# Sudo foydalanish
grep "sudo" /var/log/auth.log | grep "COMMAND" | tail -20

# Noodatiy vaqtda loginlar (tun 22:00 - 06:00)
awk '/auth.log/ {
  match($3, /([0-9]{2}):/, h)
  if (h[1]+0 >= 22 || h[1]+0 <= 6) print
}' /var/log/auth.log

# Yangi foydalanuvchilar
grep "useradd\|new user" /var/log/auth.log

# Root loginlar
grep "session opened for user root" /var/log/auth.log
```

---

## Web Server Loglar

### Apache/Nginx Log Formati

```
192.168.1.100 - ali [15/Jan/2025:10:30:15 +0500] "GET /index.html HTTP/1.1" 200 1234 "-" "Mozilla/5.0"

Qismlar:
IP            = 192.168.1.100
User          = ali
Vaqt          = 15/Jan/2025:10:30:15 +0500
So'rov        = GET /index.html HTTP/1.1
Status        = 200
Hajm          = 1234 bytes
Referrer      = -
User-Agent    = Mozilla/5.0
```

### Hujum Belgilari

```bash
# SQL Injection urinishlari
grep -i "union\|select\|insert\|update\|delete\|drop\|'\|--" \
  /var/log/nginx/access.log | head -20

# Directory traversal
grep "\.\.\/" /var/log/nginx/access.log

# XSS urinishlari
grep -i "script\|onerror\|onload\|alert" /var/log/nginx/access.log

# Brute force (ko'p 401/403)
awk '$9 == 401 || $9 == 403 {print $1}' /var/log/nginx/access.log | \
  sort | uniq -c | sort -rn | head 10

# Scanner aniqlash (ko'p 404)
awk '$9 == 404 {print $1}' /var/log/nginx/access.log | \
  sort | uniq -c | sort -rn | head 10

# Top IP lar
awk '{print $1}' /var/log/nginx/access.log | \
  sort | uniq -c | sort -rn | head 20

# Top URL lar
awk '{print $7}' /var/log/nginx/access.log | \
  sort | uniq -c | sort -rn | head 20
```

---

## Splunk bilan Log Tahlili

```
# Muvaffaqiyatsiz loginlar soni IP bo'yicha
index=windows EventCode=4625
| stats count by IpAddress
| sort -count
| head 20

# Brute force aniqlash (5 daqiqada 10+ urinish)
index=windows EventCode=4625
| bucket _time span=5m
| stats count by _time, IpAddress
| where count >= 10

# Lateral movement — admin share
index=windows EventCode=5145
| where RelativeTargetName="*ADMIN$*" OR RelativeTargetName="*C$*"
| table _time, IpAddress, ShareName, SubjectUserName

# Yangi scheduled task lar
index=windows EventCode=4698
| table _time, ComputerName, SubjectUserName, TaskName, TaskContent

# PowerShell download urinishlari
index=windows EventCode=4104
| where match(ScriptBlockText, "(?i)downloadstring|webclient|invoke-webrequest|curl|wget")
| table _time, ComputerName, UserID, ScriptBlockText
```

---

## Tez Anomaliya Tekshiruv Ro'yxati

```
□ Ish vaqtidan tashqari loginlar (tun/dam olish kunlari)
□ Bir IP dan 10+ muvaffaqiyatsiz login
□ Yangi admin hisob yaratilishi
□ ADMIN$ yoki C$ share ulashish
□ PowerShell encoded buyruqlar
□ Yangi xizmat yoki scheduled task
□ Katta hajmli ma'lumot chiqishi
□ Noodatiy jarayon yaratilishi (cmd, powershell, wscript)
□ /tmp yoki AppData da bajariladigan fayllar
□ Tashqi IP ga DNS so'rovlari (uzun nomlar)
```
