# Burp Suite — Web Xavfsizlik Testi

## Burp Suite nima?

Burp Suite — web ilovalarni xavfsizlik testidan o'tkazish uchun eng keng ishlatiladigan platforma. Community (bepul) va Professional versiyalari mavjud.

## Sozlash

### Proxy Sozlash
```
1. Burp Suite ishga tushiring
2. Proxy → Options → 127.0.0.1:8080
3. Brauzerda proxy: 127.0.0.1:8080
4. CA sertifikatini o'rnatish:
   - http://burp → CA Certificate yuklab olish
   - Brauzer → Settings → Certificates → Import
```

### FoxyProxy (Firefox)
```
1. FoxyProxy qo'shimchasini o'rnatish
2. Yangi proxy: 127.0.0.1:8080
3. "Burp" deb nomlash
4. Kerak bo'lganda yoqish
```

## Asosiy Modullar

### 1. Proxy — Interceptor
```
Proxy → Intercept → "Intercept is on"

Olingan so'rovda:
- Forward — yuborish
- Drop — rad etish
- Action → Send to Repeater/Intruder

HTTP History — barcha so'rovlarni ko'rish
```

### 2. Repeater — So'rovlarni Qayta Yuborish
```
So'rov ustida o'ng tugma → Send to Repeater (Ctrl+R)

Foydalanish:
- Parametrlarni o'zgartirib qayta yuborish
- Zaifliklarni qo'lda tekshirish
- SQL injection, XSS tekshirish
```

### 3. Intruder — Avtomatlashtirilgan Hujum
```
So'rov ustida o'ng tugma → Send to Intruder (Ctrl+I)

Attack turlari:
- Sniper — bitta payload, bitta pozitsiya
- Battering ram — bitta payload, ko'p pozitsiya
- Pitchfork — parallel payloadlar
- Cluster bomb — barcha kombinatsiyalar

Foydalanish:
- Brute force (login, parol)
- Directory bruteforce
- Parameter fuzzing
```

### 4. Scanner (Pro versiya)
```
Avtomatik zaiflik topish:
- SQL Injection
- XSS
- SSRF
- XXE
- va boshqalar
```

### 5. Decoder
```
Decoder → turli kodlashlarni qayta ishlash

URL decode: %3C → <
Base64 decode
HTML decode
Hex
```

### 6. Comparer
```
Ikkita javobni solishtirish
- Turli foydalanuvchilar javobi
- Login muvaffaqiyatli/muvaffaqiyatsiz
```

## Amaliy Misollar

### SQL Injection Tekshirish
```
1. Login formasini tutib oling
2. Repeater ga yuboring
3. username parametriga quyidagilarni sinab ko'ring:
   admin'--
   admin' OR '1'='1
   admin' OR 1=1--
4. Javob farqini kuzating
```

### XSS Tekshirish
```
1. Input maydonini topish
2. Quyidagilarni sinash:
   <script>alert(1)</script>
   <img src=x onerror=alert(1)>
   "><script>alert(1)</script>
3. Javobda skript bajarilishini tekshirish
```

### Directory Bruteforce
```
1. Target URL ni oching (masalan: http://10.10.1.1/)
2. So'rovni Intruder ga yuboring
3. URL oxiriga §§ qo'ying: GET /§test§ HTTP/1.1
4. Payloads → wordlist yuklang (common.txt, directory-list.txt)
5. Start Attack
6. Status 200 bo'lganlarini tekshiring
```

### Autentifikatsiya Bypass
```
1. Login so'rovini tutib oling
2. Repeater da sinash:
   - Admin username bilan turli parollar
   - SQL injection
   - Default parollar: admin/admin, admin/password

3. Intruder bilan brute force:
   - Sniper → password maydoniga wordlist
   - Cluster bomb → user + pass wordlist
```

## Burp Suite Shortcuts

| Shortcut | Vazifa |
|----------|--------|
| Ctrl+R | Repeater ga yuborish |
| Ctrl+I | Intruder ga yuborish |
| Ctrl+S | Scanner ga yuborish |
| Ctrl+F | Qidirish |
| Ctrl+Z | Orqaga |

## OWASP ZAP — Bepul Alternativa

```bash
# O'rnatish
sudo apt install zaproxy

# CLI bilan
zap-cli quick-scan --self-contained \
  --start-options '-config api.disablekey=true' \
  http://target.com
```
