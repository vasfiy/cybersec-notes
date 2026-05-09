# Threat Intelligence — Tahdid Razvedkasi

## Threat Intelligence nima?

Kiberxavfsizlik tahdidlari haqida yig'ilgan, tahlil qilingan va amalda qo'llash mumkin bo'lgan ma'lumot. SOC Analitik tahdidni **oldindan** tushunib, **tezroq** munosabat bildirishi uchun kerak.

## Threat Intelligence Turlari

| Tur | Tavsif | Qo'llanma |
|-----|--------|-----------|
| **Strategic** | Umumiy tendentsiyalar, geopolitika | CISO, rahbariyat |
| **Operational** | Hujumchilar niyatlari, kampaniyalar | SOC boshlig'i |
| **Tactical** | TTPs (Tactics, Techniques, Procedures) | SOC Analitik |
| **Technical** | IOC: IP, hash, domen | Tier 1 Analitik |

---

## MITRE ATT&CK Framework

Haqiqiy hujumchilar taktika va texnikalarining katalogi. SOC da **tahdidni tushunish** va **detection qoidalar** yozish uchun ishlatiladi.

```
URL: https://attack.mitre.org/
```

### Taktikalar (14 ta)

```
TA0001 — Initial Access        (Dastlabki kirish)
TA0002 — Execution             (Kod bajarish)
TA0003 — Persistence           (Qolish)
TA0004 — Privilege Escalation  (Huquqni kengaytirish)
TA0005 — Defense Evasion       (Himoyadan qochish)
TA0006 — Credential Access     (Hisoblarni o'g'irlash)
TA0007 — Discovery             (Razvedka)
TA0008 — Lateral Movement      (Tarmoqda harakatlanish)
TA0009 — Collection            (Ma'lumot yig'ish)
TA0010 — Exfiltration          (Ma'lumotni chiqarish)
TA0011 — Command and Control   (C2)
TA0040 — Impact                (Zarar etkazish)
```

### Muhim Texnikalar (SOC uchun)

```
T1566     — Phishing (Initial Access)
T1059     — Command and Scripting Interpreter
T1053     — Scheduled Task/Job (Persistence)
T1547     — Boot/Logon Autostart (Persistence)
T1055     — Process Injection (Defense Evasion)
T1003     — OS Credential Dumping (Credential Access)
T1021     — Remote Services (Lateral Movement)
T1083     — File and Directory Discovery
T1041     — Exfiltration Over C2 Channel
T1486     — Data Encrypted for Impact (Ransomware)
```

### MITRE Navigator — Vizualizatsiya

```
https://mitre-attack.github.io/attack-navigator/

Foydalanish:
1. Hujumchi guruhini tanlash (APT28, Lazarus, ...)
2. Ularning TTPs ni ko'rish
3. Detection imkoniyatlarini baholash
```

---

## IOC — Indicators of Compromise

Hujum bo'lganligini bildiruvchi texnik belgilar.

### IOC Turlari

```
Network IOC:
- IP manzillar (C2 serverlar)
- Domenlar (malware domenlar)
- URL lar
- Email manbalari

Host IOC:
- File hash (MD5, SHA256)
- Fayl nomlari / yo'llar
- Registry kalitlar
- Mutex nomlari
- Jarayon nomlari

Behavioral IOC:
- TTPs (MITRE ATT&CK)
- Noodatiy process daraxt
- Trafik namunalar
```

### IOC ning Piramida Modeli (Pyramid of Pain)

```
    /\
   /  \  TTPs           ← Eng qiyin o'zgartirish (hujumchi uchun)
  /────\
 / Tools \              ← Qiyin
/──────────\
/ Network/Host \        ← O'rta
/──────────────\
/  Domain Names  \      ← Oson
/────────────────\
/   IP Addresses  \     ← Eng oson o'zgartirish
/──────────────────\
/    Hash Values    \   ← Trivial
```

---

## OSINT Manbalar

### Bepul Threat Intel Manbalari

```
Tahdid lentalar (Threat Feeds):
- https://otx.alienvault.com     — AlienVault OTX (bepul)
- https://threatfox.abuse.ch     — IOC bazasi
- https://urlhaus.abuse.ch       — Zararli URL lar
- https://bazaar.abuse.ch        — Malware namunalar
- https://phishtank.org          — Phishing URL lar
- https://feodotracker.abuse.ch  — C2 serverlar

IP/Domen tekshirish:
- https://www.virustotal.com     — Hash, IP, URL, domen
- https://www.shodan.io          — Internet qurilmalar
- https://censys.io              — Sertifikat + port ma'lumot
- https://urlscan.io             — URL tahlil
- https://www.hybrid-analysis.com — Malware sandox

Email:
- https://mxtoolbox.com         — Email header, SPF, DKIM
- https://www.emailrep.io       — Email obro' tekshirish
```

### VirusTotal API (Python)

```python
import requests

API_KEY = "your_vt_api_key"

def check_hash(file_hash):
    url = f"https://www.virustotal.com/api/v3/files/{file_hash}"
    headers = {"x-apikey": API_KEY}
    r = requests.get(url, headers=headers)
    data = r.json()
    
    stats = data["data"]["attributes"]["last_analysis_stats"]
    print(f"Malicious: {stats['malicious']}/{sum(stats.values())}")
    return stats

def check_ip(ip):
    url = f"https://www.virustotal.com/api/v3/ip_addresses/{ip}"
    headers = {"x-apikey": API_KEY}
    r = requests.get(url, headers=headers)
    return r.json()
```

---

## Threat Hunting

Tahdidlarni **proaktiv** qidirish — alert kutmasdan.

### Hunting Gipotezalar

```
"Agar APT28 bizni nishon olgan bo'lsa, ular PowerShell
 encoded buyruqlar ishlatadi"
→ PowerShell Event 4104 log → base64 string qidirish

"Agar ransomware tarqalayotgan bo'lsa, shadow copy o'chiriladi"
→ Event 7045 + vssadmin.exe + delete shadows

"Agar lateral movement bo'lsa, admin share ishlatiladi"
→ Event 5145 + \\ADMIN$ yoki \\C$
```

### Splunk Hunting So'rovlari

```
# PowerShell encoded buyruqlar
index=windows EventCode=4104
| where match(ScriptBlockText, "(?i)frombase64")
| table _time, ComputerName, UserID, ScriptBlockText

# Tun yarim ketidan jarayonlar
index=windows EventCode=4688
| eval hour=strftime(_time, "%H")
| where hour >= 22 OR hour <= 6
| table _time, ComputerName, NewProcessName, CommandLine

# C2 beacon aniqlash (doimiy interval)
index=network
| stats count, range(_time) as duration by src_ip, dest_ip, dest_port
| eval beacon_interval = duration / count
| where beacon_interval > 50 AND beacon_interval < 70

# Mass file encryption (Ransomware)
index=windows EventCode=4663
| stats count by ObjectName, SubjectUserName
| where count > 100
| eval ext = mvindex(split(ObjectName, "."), -1)
| where ext IN ("enc", "locked", "crypto", "ransomed")
```

---

## Threat Intelligence Platformalari (TIP)

```
Bepul:
- MISP (Open Source) — IOC almashish
- OpenCTI (Open Source) — CTI platforma

Tijorat:
- Recorded Future
- ThreatConnect
- Anomali ThreatStream
- Mandiant Advantage
```

### MISP — Malware Information Sharing Platform

```bash
# Docker bilan o'rnatish
git clone https://github.com/MISP/misp-docker
cd misp-docker
docker-compose up -d

# API bilan IOC qo'shish (Python)
from pymisp import PyMISP

misp = PyMISP("https://misp.local", "API_KEY", False)
event = misp.new_event(info="Phishing campaign 2025-01")
misp.add_named_attribute(event, "ip-dst", "192.168.1.100")
misp.add_named_attribute(event, "domain", "evil-phishing.com")
```

---

## SOC Analitik: Incident da TI qo'llash

```
1. Alert keldi → IOC larni ajratib oling
2. VT / OTX da IOC lar tekshiring
3. MITRE ATT&CK da texnikani aniqlang
4. Hujumchi guruhini aniqlang (attribution)
5. Boshqa tizimda xuddi shu IOC bormi — tekshiring
6. Threat hunt qoidasi yarating
7. Blok qoidasi qo'shing (firewall, SIEM)
8. Hisobot yozing
```
