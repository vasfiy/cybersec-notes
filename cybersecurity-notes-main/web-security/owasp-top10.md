# OWASP Top 10 — Web Xavfsizlik Zaifliklar

## OWASP nima?

OWASP (Open Web Application Security Project) — web ilovalar xavfsizligini yaxshilashga bag'ishlangan notijorat tashkilot. Top 10 — eng xavfli web zaifliklar ro'yxati.

---

## A01 — Broken Access Control (Buzilgan Kirish Nazorati)

**Mohiyat:** Foydalanuvchi o'z huquqidan tashqariga chiqishi mumkin.

```
Misol:
https://example.com/user/123/profile   ← o'z profil
https://example.com/user/124/profile   ← boshqa odamning profili (IDOR)
```

**Tekshirish:**
- URL dagi ID larni o'zgartirish
- Admin sahifasiga kirish urinishi: `/admin`, `/dashboard`
- HTTP metod almashtirish: GET → POST → PUT

**Himoya:** Server tomonida har so'rovda ruxsatni tekshirish

---

## A02 — Cryptographic Failures (Kriptografik Muvaffaqiyatsizliklar)

**Mohiyat:** Ma'lumotlar noto'g'ri yoki umuman shifrlanmagan.

```
Muammolar:
- Parollar MD5/SHA1 bilan saqlangan (zaif)
- HTTP ishlatilishi (HTTPS o'rniga)
- Zaif SSL/TLS versiyalari
- Shifrlash kalitlari kodni ichida
```

**Himoya:** bcrypt/Argon2 parol hash, HTTPS, zamonaviy TLS

---

## A03 — Injection (Kiritma Hujumlari)

**Mohiyat:** Foydalanuvchi kiritgan ma'lumot kod sifatida bajariladi.

### SQL Injection
```sql
-- Login formasi:
Username: admin'--
Password: anything

-- Bajariladi:
SELECT * FROM users WHERE user='admin'--' AND pass='anything'
-- ' -- izoh belgisi, parol tekshirilmaydi
```

### OS Command Injection
```
Input: ; cat /etc/passwd
URL: https://example.com/ping?ip=8.8.8.8; cat /etc/passwd
```

### XXE (XML External Entity)
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>
```

**Himoya:** Prepared statements, input validation, WAF

---

## A04 — Insecure Design (Xavfsiz bo'lmagan Arxitektura)

**Mohiyat:** Arxitektura darajasida xavfsizlik hisobga olinmagan.

```
Misol:
- Parolni tiklashda "Sevimli raqam nima?" savoli
- Rate limiting yo'q (brute force mumkin)
- "Forgot password" da parolni email ga yuborish
```

---

## A05 — Security Misconfiguration (Noto'g'ri Xavfsizlik Sozlamasi)

```
Muammolar:
- Default parollar (admin/admin)
- Stack trace xato xabarlari (server ma'lumotlari)
- Keraksiz portlar ochiq
- Debug rejimi production da
- Directory listing yoqiq

Test:
curl https://example.com/phpinfo.php
curl https://example.com/.git/config
curl https://example.com/config.php.bak
```

---

## A06 — Vulnerable Components (Zaif Komponentlar)

```bash
# Versiyalarni tekshirish
whatweb https://example.com
wappalyzer brauzer qo'shimchasi

# CVE tekshirish
nmap -sV --script vuln 192.168.1.1

# npm zaifliklar
npm audit

# pip zaifliklar
pip-audit
```

---

## A07 — Authentication Failures (Autentifikatsiya Muvaffaqiyatsizliklari)

```
Muammolar:
- Brute force himoyasi yo'q
- Zaif parollar ruxsat beriladi
- Sessiya token URL da
- Chiqishda sessiya bekor qilinmaydi

Test:
- Default parollar: admin/admin, root/root
- Brute force: hydra, medusa
- Sessiya ID ni tekshirish
```

```bash
# Hydra bilan brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  192.168.1.1 http-post-form \
  "/login:user=^USER^&pass=^PASS^:Invalid credentials"
```

---

## A08 — Software and Data Integrity Failures

```
Muammolar:
- Ishonchsiz CDN dan skriptlar
- Imzosiz yangilanishlar
- Insecure deserialization

Misol:
<script src="http://malicious-cdn.com/jquery.js"></script>
```

---

## A09 — Security Logging Failures (Logging Muammolari)

```
Muammolar:
- Login muvaffaqiyatsizliklari loglanmaydi
- Loglar himoyalanmagan
- Anomaliyalar aniqlanmaydi

Yaxshi log:
[2025-01-01 10:30:15] WARN: Failed login attempt
  user=admin ip=192.168.1.100 attempt=5
```

---

## A10 — SSRF (Server-Side Request Forgery)

**Mohiyat:** Server boshqa manzilga so'rov yuborishga majburlanadi.

```
Zaifligi bor URL:
https://example.com/fetch?url=https://api.partner.com/data

Hujum:
https://example.com/fetch?url=http://169.254.169.254/
  (AWS metadata — credentials!)

https://example.com/fetch?url=http://localhost:8080/admin
  (Ichki admin panel)
```

**Himoya:** URL whitelist, metadata endpoint ga kirish bloklansin

---

## Tez Tekshiruv Ro'yxati

```
□ IDOR — ID larni o'zgartirish
□ SQL Injection — ' va " kiritish
□ XSS — <script>alert(1)</script>
□ Directory traversal — ../../../etc/passwd
□ SSRF — url= parametri
□ Default credentials — admin/admin
□ Sensitive files — /.git, /backup, /config
□ HTTP metodlar — OPTIONS, TRACE
□ Security headers — curl -I https://example.com
```
