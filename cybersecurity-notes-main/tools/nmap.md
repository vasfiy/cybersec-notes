# Nmap — Tarmoq Skaneri

## Nmap nima?

Nmap (Network Mapper) — tarmoqdagi qurilmalar, ochiq portlar, ishlaydigan xizmatlar va OS ni aniqlash uchun eng keng tarqalgan vosita.

> **Ogohlantiruv:** Faqat ruxsat berilgan tarmoqlarda ishlating!

## Asosiy Sintaksis

```bash
nmap [opsiyalar] [maqsad]

# Maqsad formatlari:
nmap 192.168.1.1           # Bitta IP
nmap 192.168.1.1-254       # IP range
nmap 192.168.1.0/24        # Subnet
nmap scanme.nmap.org       # Domen
nmap -iL targets.txt       # Fayldan
```

## Skan Turlari

```bash
# Ping skan (faqat hoslar)
nmap -sn 192.168.1.0/24

# TCP SYN skan (default, stealth) — root kerak
nmap -sS 192.168.1.1

# TCP Connect skan (root shart emas)
nmap -sT 192.168.1.1

# UDP skan (sekin, lekin muhim)
nmap -sU 192.168.1.1

# Version aniqlash
nmap -sV 192.168.1.1

# OS aniqlash — root kerak
nmap -O 192.168.1.1

# Script skan
nmap -sC 192.168.1.1

# Aggressive (OS + version + scripts + traceroute)
nmap -A 192.168.1.1
```

## Port Opsiyalari

```bash
# Muayyan portlar
nmap -p 80,443,22 192.168.1.1

# Port oraliq
nmap -p 1-1000 192.168.1.1

# Barcha portlar (0-65535)
nmap -p- 192.168.1.1

# Eng keng tarqalgan 100 port
nmap --top-ports 100 192.168.1.1

# Barcha portlar, tezroq
nmap -p- --min-rate 5000 192.168.1.1
```

## Tezlik va Vaqt

```bash
# T0 — Paranoid (eng sekin)
# T1 — Sneaky
# T2 — Polite
# T3 — Normal (default)
# T4 — Aggressive (CTF/lab uchun yaxshi)
# T5 — Insane (eng tez, aniqsiz)

nmap -T4 192.168.1.1
nmap -T2 192.168.1.1   # sekinroq, ko'rinmasroq
```

## NSE Scripts (Nmap Scripting Engine)

```bash
# Default skriptlar
nmap -sC 192.168.1.1

# Kategoriya bo'yicha
nmap --script=vuln 192.168.1.1        # zaifliklar
nmap --script=auth 192.168.1.1        # autentifikatsiya
nmap --script=discovery 192.168.1.1   # kashfiyot
nmap --script=brute 192.168.1.1       # brute force

# Muayyan skript
nmap --script=http-title 192.168.1.1
nmap --script=smb-vuln-ms17-010 192.168.1.1  # EternalBlue
nmap --script=ftp-anon 192.168.1.1           # Anonim FTP

# Skript bilan argument
nmap --script=http-brute --script-args=... 192.168.1.1
```

## Output Formatlari

```bash
# Oddiy matn
nmap -oN scan.txt 192.168.1.1

# XML
nmap -oX scan.xml 192.168.1.1

# Grepable
nmap -oG scan.gnmap 192.168.1.1

# Barcha formatlar
nmap -oA scan 192.168.1.1   # scan.nmap, scan.xml, scan.gnmap

# Verbose
nmap -v 192.168.1.1
nmap -vv 192.168.1.1
```

## Amaliy Misollar

```bash
# 1. Tez umumiy skan
nmap -T4 -A 192.168.1.1

# 2. Barcha portlarni tekshirish
nmap -p- -T4 192.168.1.1

# 3. Tarmoqda qurilmalarni topish
nmap -sn 192.168.1.0/24

# 4. Xizmat va versiyalarni aniqlash
nmap -sV --version-intensity 5 192.168.1.1

# 5. Zaiflik skani
nmap --script=vuln 192.168.1.1

# 6. SMB zaifliklarini tekshirish
nmap --script=smb-vuln* -p 445 192.168.1.1

# 7. Web server tekshiruvi
nmap --script=http-* -p 80,443 192.168.1.1

# 8. Stealth skan (IDS dan qochish)
nmap -sS -T2 -D RND:5 192.168.1.1
```

## Natijani Tahlil Qilish

```
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 8.2p1
80/tcp   open     http       Apache httpd 2.4.41
443/tcp  open     https      Apache httpd 2.4.41
3306/tcp closed   mysql
8080/tcp filtered http-proxy
```

- **open** — Port ochiq, xizmat ishlaydi
- **closed** — Port yopiq, javob beradi
- **filtered** — Firewall bloklab qo'ygan
- **open|filtered** — Aniq emas

## Masscan — Tezroq Alternativa

```bash
# Butun internet bo'yicha skan (lab uchun)
masscan 10.0.0.0/8 -p80 --rate=10000

# Nmap bilan birgalikda
masscan 192.168.1.0/24 -p1-65535 --rate=1000 -oG masscan.txt
# Keyin ochiq portlarga nmap bilan batafsil skan
```
