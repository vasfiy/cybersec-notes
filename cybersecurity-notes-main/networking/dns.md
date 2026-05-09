# DNS — Domain Name System

## DNS nima?

DNS — internet "telefon kitobi". Inson o'qiydigan domen nomlarini (`google.com`) IP manzillarga (`142.250.185.46`) aylantiradi.

## DNS Qanday Ishlaydi?

```
Siz: google.com kiriting
  |
  ▼
1. Browser cache tekshiriladi
  |
  ▼
2. OS cache (/etc/hosts) tekshiriladi
  |
  ▼
3. Recursive Resolver (ISP yoki 8.8.8.8) so'raladi
  |
  ▼
4. Root Server (.) so'raladi → "com. uchun .com serveri bor"
  |
  ▼
5. TLD Server (.com) so'raladi → "google.com uchun bu server bor"
  |
  ▼
6. Authoritative Server so'raladi → "142.250.185.46"
  |
  ▼
Javob: 142.250.185.46
```

## DNS Yozuv Turlari (Record Types)

| Tur | Nomi | Misol |
|-----|------|-------|
| A | IPv4 manzil | `google.com → 142.250.185.46` |
| AAAA | IPv6 manzil | `google.com → 2607:f8b0::...` |
| CNAME | Alias | `www.example.com → example.com` |
| MX | Mail server | `example.com → mail.example.com` |
| TXT | Matn ma'lumot | SPF, DKIM, domenni tasdiqlash |
| NS | Name server | `example.com → ns1.example.com` |
| PTR | Teskari DNS | `142.250.185.46 → google.com` |
| SOA | Zone ma'lumoti | Zona boshlanishi |

## DNS Buyruqlari

```bash
# A yozuvini so'rash
dig google.com

# Muayyan yozuv turi
dig google.com MX
dig google.com TXT
dig google.com NS

# Muayyan DNS serverga so'rash
dig @8.8.8.8 google.com

# Teskari DNS (PTR)
dig -x 8.8.8.8

# Qisqa javob
dig +short google.com

# nslookup (Windows/Linux)
nslookup google.com
nslookup -type=MX google.com

# host buyrug'i
host google.com
host -t MX google.com
```

## /etc/hosts fayli

```bash
# Ko'rish
cat /etc/hosts

# Misol tarkib:
127.0.0.1   localhost
192.168.1.10  myserver.local
```

DNS dan oldin tekshiriladi — CTF va pentest da muhim!

## DNS ga Asoslangan Hujumlar

### 1. DNS Spoofing / Cache Poisoning
DNS javoblarini soxta IP bilan almashtirish.
```
Hacker → Soxta DNS javob → Victim noto'g'ri IP oladi
```

### 2. DNS Amplification (DDoS)
Kichik so'rov → Katta javob → Victim serverga yo'naltirish.

### 3. DNS Zone Transfer
Noto'g'ri sozlangan server barcha DNS yozuvlarini berishi mumkin:
```bash
# Zone transfer urinishi
dig axfr @ns1.example.com example.com

# fierce vosita bilan
fierce --domain example.com
```

### 4. Subdomain Takeover
Yo'q qilingan subdomainga yo'naltirish qolgan bo'lsa, uni egallash.

### 5. DNS Tunneling
DNS so'rovlari orqali ma'lumot o'tkazish (firewall bypass).

## DNS Recon (Razvedka)

```bash
# Subdomenlarni topish
sublist3r -d example.com
amass enum -d example.com
ffuf -w /usr/share/wordlists/dns.txt -u https://FUZZ.example.com

# DNS yozuvlarini yig'ish
dnsrecon -d example.com
dnsenum example.com

# theHarvester bilan
theHarvester -d example.com -b all
```

## DoH va DoT (Xavfsiz DNS)

| Protokol | Port | Tavsif |
|----------|------|--------|
| DNS | 53 | Oddiy, shifrlanmagan |
| DNS over HTTPS (DoH) | 443 | HTTPS orqali |
| DNS over TLS (DoT) | 853 | TLS orqali |

```bash
# DoH bilan so'rash
curl -s "https://dns.google/resolve?name=google.com&type=A"
```
