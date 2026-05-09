# OSI Model — 7 Qatlamli Model

## Nima uchun OSI kerak?

OSI (Open Systems Interconnection) modeli tarmoq aloqasini 7 qatlamga ajratib, har bir qatlam faqat o'z vazifasini bajaradi. Bu nosozliklarni tezda aniqlash va protokollarni tushunish uchun muhim.

## 7 Qatlam

```
7  Application   — Foydalanuvchi bilan to'g'ridan to'g'ri muloqot (HTTP, FTP, DNS, SMTP)
6  Presentation  — Ma'lumotni formatlash, shifrlash (SSL/TLS, JPEG, ASCII)
5  Session       — Sessiya ochish/yopish (NetBIOS, RPC)
4  Transport     — Ishonchli yetkazib berish, portlar (TCP, UDP)
3  Network       — Marshrutlash, IP manzillar (IP, ICMP, ARP)
2  Data Link     — MAC manzillar, kadrlar (Ethernet, Wi-Fi, Switch)
1  Physical      — Fizik signal, kabel (Mis kabel, Fiber, Radio)
```

## Har bir qatlam batafsil

### 7 — Application (Ilova)
- Foydalanuvchi ilovalar bilan aloqa
- **Protokollar:** HTTP, HTTPS, FTP, SSH, Telnet, DNS, SMTP, POP3, IMAP
- **Hujumlar:** Phishing, SQL Injection, XSS

### 6 — Presentation (Taqdimot)
- Ma'lumotni o'qiladigan formatga o'tkazish
- Shifrlash va dekodlash
- **Misol:** TLS handshake, Base64, Unicode

### 5 — Session (Sessiya)
- Ikki qurilma orasida sessiyani boshqarish
- Authentication ko'pincha shu darajada
- **Misol:** Login sessiyasi, RPC

### 4 — Transport (Transport)
- Port raqamlari (0–65535)
- **TCP** — ishonchli (ACK talab qiladi)
- **UDP** — tez, lekin ishonchsiz (DNS, VoIP, video)
- **Hujumlar:** Port scanning, SYN flood

### 3 — Network (Tarmoq)
- IP manzillar orqali marshrutlash
- **Protokollar:** IPv4, IPv6, ICMP, OSPF, BGP
- **Qurilmalar:** Router
- **Hujumlar:** IP spoofing, MITM

### 2 — Data Link (Ma'lumot havolasi)
- MAC manzillar (48-bit, masalan: `AA:BB:CC:DD:EE:FF`)
- **Protokollar:** Ethernet (802.3), Wi-Fi (802.11), ARP
- **Qurilmalar:** Switch, Bridge
- **Hujumlar:** ARP Spoofing, MAC flooding

### 1 — Physical (Fizik)
- Bitlarni elektr signal, yorug'lik yoki radioga aylantirish
- **Misol:** RJ-45, fiber optik, Wi-Fi antennasi
- **Qurilmalar:** Hub, kabel, NIC

## Xotira uchun (mnemonika)

**Yuqoridan pastga:** All People Seem To Need Data Processing  
**Pastdan yuqoriga:** Please Do Not Throw Sausage Pizza Away

## Kiberhujumlar qaysi qatlamda bo'ladi?

| Hujum turi | Qatlam |
|-----------|--------|
| Phishing | 7 — Application |
| SSL Stripping | 6 — Presentation |
| Session Hijacking | 5 — Session |
| SYN Flood (DDoS) | 4 — Transport |
| IP Spoofing | 3 — Network |
| ARP Spoofing | 2 — Data Link |
| Fizik kirish | 1 — Physical |

## Amaliy misol: `ping google.com` nima qiladi?

```
1. DNS so'rovi yuboriladi (Layer 7 — Application)
2. ICMP paketi yaratiladi (Layer 3 — Network)
3. Ethernet kadri yaratiladi (Layer 2 — Data Link)
4. Kabel orqali signal yuboriladi (Layer 1 — Physical)
5. Router marshrutlaydi (Layer 3)
6. Javob qaytadi
```
