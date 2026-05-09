# Disk Forensics — Disk Tahlili

## DFIR Asosiy Tamoyillari

```
1. Asl nusxani o'zgartirma — avval nusxa ol
2. Barcha amallarni hujjatlashtir
3. Chain of Custody — kim qachon nimaga tegdi
4. Hash bilan yaxlitlikni tasdiqlash (MD5/SHA256)
```

## Disk Image Olish

```bash
# dd — klassik usul
sudo dd if=/dev/sda of=/external/disk.img bs=4M status=progress

# ddrescue — zarar ko'rgan disk uchun
sudo ddrescue /dev/sda /external/disk.img /external/rescue.log

# dcfldd — hash bilan
sudo dcfldd if=/dev/sda of=/external/disk.img hash=sha256 \
  hashlog=/external/hash.log

# FTK Imager (Windows, GUI) — eng ko'p ishlatiladigan

# Hash tekshirish
md5sum disk.img > disk.img.md5
sha256sum disk.img > disk.img.sha256
```

## Autopsy — GUI Forensics

```bash
# O'rnatish
sudo apt install autopsy

# Ishga tushirish
autopsy
# Brauzerda: http://localhost:9999/autopsy

# Asosiy qadamlar:
# 1. New Case → Case ma'lumotlari
# 2. Add Data Source → disk image yoki papka
# 3. Ingest Modules tanlash:
#    - File Type Identification
#    - Keyword Search
#    - Email Parser
#    - Web Artifacts
#    - Recent Activity
# 4. Tahlil natijalarini ko'rish
```

## Fayl Tizimi Tahlili

```bash
# Disk image ni ulash (read-only)
sudo mount -o ro,loop disk.img /mnt/evidence

# Partitsiyali image
sudo fdisk -l disk.img
# Offset = bo'lim boshlanish sektori × 512

sudo mount -o ro,loop,offset=1048576 disk.img /mnt/evidence

# ext4 fayl tizimi
fsstat disk.img                    # umumiy ma'lumot
fls -r disk.img                    # fayl ro'yxati (o'chirilganlar ham)
istat disk.img <inode>             # fayl metadata

# NTFS (Windows)
sudo ntfs-3g -o ro disk.img /mnt/
```

## O'chirilgan Fayllarni Tiklish

```bash
# Foremost — fayl imzosi bo'yicha
foremost -i disk.img -o /tmp/recovered/

# Scalpel — konfiguratsiyalash mumkin
scalpel disk.img -o /tmp/scalpel_output/

# PhotoRec — rasm va ko'p format
photorec disk.img

# extundelete — ext3/ext4
extundelete disk.img --restore-all

# Recuva (Windows GUI)
```

## Artifact Tahlili — Windows

### Registry

```bash
# Registry fayllar joylashuvi (disk image da)
C:\Windows\System32\config\SAM       — Foydalanuvchi hisoblar
C:\Windows\System32\config\SYSTEM    — Tizim konfiguratsiya
C:\Windows\System32\config\SOFTWARE  — O'rnatilgan dasturlar
C:\Windows\System32\config\SECURITY  — Xavfsizlik siyosat
C:\Users\<user>\NTUSER.DAT           — Foydalanuvchi sozlamalari

# RegRipper bilan tahlil
rip.pl -r /mnt/NTUSER.DAT -f ntuser > output.txt
rip.pl -r /mnt/SAM -f sam

# Muhim Registry joylari
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run    — Autostart
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run    — User Autostart
HKLM\SYSTEM\CurrentControlSet\Services               — Xizmatlar
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList — Profillar
```

### Windows Prefetch

```bash
# Joylashuvi
C:\Windows\Prefetch\*.pf

# Nima beradi:
# - Bajarilib bo'lgan dasturlar (o'chirilgan bo'lsa ham!)
# - Oxirgi ishga tushirish vaqti
# - Ishlatilgan fayllar

# Tahlil
pecmd.py -f "NOTEPAD.EXE-1234ABCD.pf"
PECmd.exe -d "C:\Windows\Prefetch"  # (Windows)
```

### Browser Artifacts

```bash
# Chrome
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\
├── History          — URL tarixi (SQLite)
├── Downloads        — Yuklamalar
├── Cookies          — Cookie lar
├── Login Data       — Saqlangan parollar
└── Cache/           — Kesh fayllar

# Tahlil (SQLite)
sqlite3 History "SELECT url, title, visit_count, last_visit_time FROM urls ORDER BY last_visit_time DESC LIMIT 50;"

# hindsight (Python tool)
hindsight.py -i "C:\Users\user\AppData\Local\Google\Chrome\User Data" -o report

# Firefox
%APPDATA%\Mozilla\Firefox\Profiles\*.default\places.sqlite
```

### Windows Event Log (disk da)

```bash
# Joylashuvi
C:\Windows\System32\winevt\Logs\

# python-evtx bilan o'qish
evtx_dump.py Security.evtx > security_events.xml

# chainsaw — tez tahlil
chainsaw search Security.evtx -s sigma_rules/
chainsaw hunt Security.evtx -r rules/ --mapping mappings.yml

# Hayabusa — Windows log tahlil
hayabusa csv-timeline -d C:\Windows\System32\winevt\Logs\ -o timeline.csv
```

### LNK fayllar (Shortcut)

```bash
# Joylashuvi
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\

# Nima beradi: fayl ochilgan vaqt, asl yo'l, kompyuter nomi

# lnkparse
lnkparse file.lnk
```

### Prefetch, Shimcache, AmCache

```bash
# ShimCache — bajarilib bo'lgan dasturlar ro'yxati
# HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache

# AmCache — o'rnatilgan dasturlar, SHA1 hash bilan
C:\Windows\AppCompat\Programs\Amcache.hve

# AppCompatCacheParser
AppCompatCacheParser.exe -f SYSTEM --csv output.csv
```

---

## Linux Disk Forensics

```bash
# Joylashgan fayllar (ctime/mtime)
find /mnt/ -newermt "2025-01-01" -type f 2>/dev/null

# Yashirin fayllar
find /mnt/ -name ".*" -type f

# SUID/SGID fayllar
find /mnt/ -perm -4000 -o -perm -2000 2>/dev/null

# Bash tarixi
cat /mnt/home/user/.bash_history
cat /mnt/root/.bash_history

# Cron
cat /mnt/etc/crontab
ls /mnt/var/spool/cron/

# SSH kalitlari
ls -la /mnt/home/*/.ssh/
ls -la /mnt/root/.ssh/

# Journalctl (disk da)
journalctl --root=/mnt/ -n 100
```

---

## Timeline Yaratish

```bash
# mactime (Sleuth Kit)
fls -r -m / disk.img > body.txt
mactime -b body.txt -d > timeline.csv

# log2timeline / plaso
log2timeline.py plaso.dump /mnt/evidence/
psort.py -o l2tcsv plaso.dump > timeline.csv

# Autopsy — Timeline Analysis moduli (GUI)
```

---

## CTF: Disk Forensics

```bash
# 1. Fayl turi aniqlash
file disk.img

# 2. Strings qidirish
strings disk.img | grep -i "flag\|ctf\|password\|secret"

# 3. Binwalk
binwalk disk.img
binwalk -e disk.img   # ichidagi fayllarni chiqarish

# 4. Foremost
foremost -i disk.img -o recovered/

# 5. Mount qilish
sudo mount -o ro,loop disk.img /mnt/

# 6. SQLite bazalar
find /mnt/ -name "*.db" -o -name "*.sqlite" | xargs -I{} sqlite3 {} .tables

# 7. Fayllarni tekshirish
exiftool /mnt/suspicious_file.jpg
steghide extract -sf /mnt/image.jpg
```
