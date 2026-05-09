# Python — Xavfsizlik Skriptlash

## Nima uchun Python?

SOC Analitik uchun Python — vaqtni tejash va takroriy ishlarni avtomatlash vositasi. Log tahlil, IOC tekshirish, alert boyitish — barchasi Python bilan osonlashadi.

## Muhit Sozlash

```bash
# Virtual muhit
python3 -m venv venv
source venv/bin/activate  # Linux
venv\Scripts\activate     # Windows

# Asosiy kutubxonalar
pip install requests python-dotenv colorama tabulate

# Xavfsizlik kutubxonalar
pip install scapy dnspython python-whois shodan pymisp
```

---

## 1. Log Tahlil Skriptlari

### SSH Brute Force Aniqlash

```python
#!/usr/bin/env python3
"""
/var/log/auth.log dan SSH brute force urinishlarini topish
"""
import re
from collections import Counter

LOG_FILE = "/var/log/auth.log"
THRESHOLD = 10  # bir IP dan necha marta

def parse_failed_logins(log_file):
    failed = []
    pattern = r'Failed password.*from (\d+\.\d+\.\d+\.\d+)'
    
    with open(log_file, 'r', errors='ignore') as f:
        for line in f:
            match = re.search(pattern, line)
            if match:
                failed.append(match.group(1))
    return failed

def detect_brute_force(log_file, threshold):
    failed_ips = parse_failed_logins(log_file)
    counts = Counter(failed_ips)
    
    print(f"{'IP Manzil':<20} {'Urinish':<10} {'Holat'}")
    print("-" * 45)
    
    for ip, count in counts.most_common(20):
        status = "⚠️  SHUBHALI" if count >= threshold else "OK"
        print(f"{ip:<20} {count:<10} {status}")

if __name__ == "__main__":
    detect_brute_force(LOG_FILE, THRESHOLD)
```

### Windows Event Log Tahlil

```python
#!/usr/bin/env python3
"""
Windows EVTX fayldan muvaffaqiyatsiz loginlarni chiqarish
"""
import sys
from evtx import PyEvtxParser  # pip install python-evtx

def analyze_failed_logins(evtx_file):
    parser = PyEvtxParser(evtx_file)
    results = []
    
    for record in parser.records_json():
        import json
        data = json.loads(record['data'])
        event_id = data.get('Event', {}).get('System', {}).get('EventID')
        
        if event_id == 4625:
            event_data = data.get('Event', {}).get('EventData', {})
            results.append({
                'time': record['timestamp'],
                'username': event_data.get('TargetUserName', 'N/A'),
                'ip': event_data.get('IpAddress', 'N/A'),
                'logon_type': event_data.get('LogonType', 'N/A')
            })
    
    return results

if __name__ == "__main__":
    evtx_file = sys.argv[1] if len(sys.argv) > 1 else "Security.evtx"
    logins = analyze_failed_logins(evtx_file)
    
    for login in logins[-20:]:  # oxirgi 20
        print(f"{login['time']} | {login['username']:<20} | {login['ip']}")
```

---

## 2. IOC Tekshirish Skriptlari

### VirusTotal Tekshirish

```python
#!/usr/bin/env python3
"""
IP, hash, yoki domen ni VirusTotal da tekshirish
"""
import requests
import sys

API_KEY = "your_vt_api_key_here"  # VT free API: 4 so'rov/daqiqa
BASE_URL = "https://www.virustotal.com/api/v3"

def check_ip(ip):
    url = f"{BASE_URL}/ip_addresses/{ip}"
    headers = {"x-apikey": API_KEY}
    r = requests.get(url, headers=headers)
    
    if r.status_code == 200:
        data = r.json()["data"]["attributes"]
        stats = data.get("last_analysis_stats", {})
        malicious = stats.get("malicious", 0)
        total = sum(stats.values())
        country = data.get("country", "N/A")
        
        print(f"IP: {ip}")
        print(f"Mamlakat: {country}")
        print(f"Natija: {malicious}/{total} engine XAVFLI dedi")
        return malicious > 0
    return None

def check_hash(file_hash):
    url = f"{BASE_URL}/files/{file_hash}"
    headers = {"x-apikey": API_KEY}
    r = requests.get(url, headers=headers)
    
    if r.status_code == 200:
        data = r.json()["data"]["attributes"]
        stats = data.get("last_analysis_stats", {})
        malicious = stats.get("malicious", 0)
        total = sum(stats.values())
        name = data.get("meaningful_name", "N/A")
        
        print(f"Hash: {file_hash}")
        print(f"Fayl nomi: {name}")
        print(f"Natija: {malicious}/{total} engine XAVFLI dedi")
        return malicious > 0
    return None

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Ishlatish: python3 vt_check.py <ip|hash|domain>")
        sys.exit(1)
    
    ioc = sys.argv[1]
    
    # IP yoki hash ekanini aniqlash
    if len(ioc) in [32, 40, 64]:
        check_hash(ioc)
    else:
        check_ip(ioc)
```

### Bulk IOC Tekshirish

```python
#!/usr/bin/env python3
"""
Fayldan IOC larni o'qib, barchasini VT da tekshirish
"""
import requests
import time
import csv
import sys

API_KEY = "your_vt_api_key"

def check_ioc(ioc_type, value):
    endpoints = {
        "ip": f"ip_addresses/{value}",
        "domain": f"domains/{value}",
        "hash": f"files/{value}",
        "url": f"urls/{value}"
    }
    
    url = f"https://www.virustotal.com/api/v3/{endpoints[ioc_type]}"
    headers = {"x-apikey": API_KEY}
    r = requests.get(url, headers=headers)
    
    if r.status_code == 200:
        stats = r.json()["data"]["attributes"].get("last_analysis_stats", {})
        return stats.get("malicious", 0), sum(stats.values())
    return None, None

def bulk_check(ioc_file):
    results = []
    
    with open(ioc_file) as f:
        iocs = [line.strip() for line in f if line.strip()]
    
    for ioc in iocs:
        ioc_type = "ip" if ioc.replace(".", "").isdigit() else "domain"
        malicious, total = check_ioc(ioc_type, ioc)
        
        status = "XAVFLI" if malicious and malicious > 0 else "Toza"
        results.append({
            "ioc": ioc, "type": ioc_type,
            "malicious": malicious, "total": total, "status": status
        })
        
        print(f"[{'!!' if status == 'XAVFLI' else '  '}] {ioc:<30} {status}")
        time.sleep(15)  # Free API: 4 req/min
    
    # CSV ga saqlash
    with open("ioc_results.csv", "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=results[0].keys())
        writer.writeheader()
        writer.writerows(results)
    
    print(f"\nNatija ioc_results.csv ga saqlandi")

if __name__ == "__main__":
    bulk_check(sys.argv[1] if len(sys.argv) > 1 else "iocs.txt")
```

---

## 3. Tarmoq Skanerlash

### Port Skanerlash (Nmap Wrapper)

```python
#!/usr/bin/env python3
"""
Oddiy port skanerlash skripti
"""
import socket
import concurrent.futures

def scan_port(host, port, timeout=1):
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(timeout)
            result = s.connect_ex((host, port))
            return port if result == 0 else None
    except:
        return None

def scan_host(host, port_range=(1, 1024)):
    print(f"Skanlanmoqda: {host} (portlar {port_range[0]}-{port_range[1]})")
    open_ports = []
    
    ports = range(port_range[0], port_range[1] + 1)
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=100) as executor:
        futures = {executor.submit(scan_port, host, p): p for p in ports}
        for future in concurrent.futures.as_completed(futures):
            result = future.result()
            if result:
                open_ports.append(result)
    
    open_ports.sort()
    for port in open_ports:
        try:
            service = socket.getservbyport(port)
        except:
            service = "unknown"
        print(f"  {port}/tcp  open  {service}")
    
    return open_ports

if __name__ == "__main__":
    import sys
    host = sys.argv[1] if len(sys.argv) > 1 else "127.0.0.1"
    scan_host(host)
```

---

## 4. Alert Boyitish (Enrichment)

```python
#!/usr/bin/env python3
"""
SIEM alertini qabul qilib, IOC larni boyitish
"""
import requests
import json
import ipaddress

def enrich_ip(ip):
    enrichment = {"ip": ip}
    
    # IP private tekshirish
    try:
        if ipaddress.ip_address(ip).is_private:
            enrichment["type"] = "private"
            return enrichment
    except:
        pass
    
    # VirusTotal
    vt_url = f"https://www.virustotal.com/api/v3/ip_addresses/{ip}"
    vt_headers = {"x-apikey": "YOUR_API_KEY"}
    r = requests.get(vt_url, headers=vt_headers)
    if r.status_code == 200:
        attrs = r.json()["data"]["attributes"]
        stats = attrs.get("last_analysis_stats", {})
        enrichment["vt_malicious"] = stats.get("malicious", 0)
        enrichment["country"] = attrs.get("country", "N/A")
        enrichment["asn"] = attrs.get("asn", "N/A")
        enrichment["org"] = attrs.get("as_owner", "N/A")
    
    # AbuseIPDB
    abuse_url = "https://api.abuseipdb.com/api/v2/check"
    abuse_headers = {"Key": "YOUR_ABUSEIPDB_KEY", "Accept": "application/json"}
    r = requests.get(abuse_url, headers=abuse_headers, params={"ipAddress": ip})
    if r.status_code == 200:
        data = r.json()["data"]
        enrichment["abuse_score"] = data.get("abuseConfidenceScore", 0)
        enrichment["abuse_reports"] = data.get("totalReports", 0)
    
    # Xavf darajasi
    malicious = enrichment.get("vt_malicious", 0)
    abuse_score = enrichment.get("abuse_score", 0)
    
    if malicious > 5 or abuse_score > 75:
        enrichment["risk"] = "HIGH"
    elif malicious > 0 or abuse_score > 25:
        enrichment["risk"] = "MEDIUM"
    else:
        enrichment["risk"] = "LOW"
    
    return enrichment

if __name__ == "__main__":
    ip = "1.2.3.4"
    result = enrich_ip(ip)
    print(json.dumps(result, indent=2))
```

---

## Foydali Kutubxonalar

```python
# Tarmoq
import socket          # TCP/UDP
from scapy.all import *  # Paket yaratish/tahlil
import requests        # HTTP

# Ma'lumot qayta ishlash
import re              # Regex
import json            # JSON
import csv             # CSV
from datetime import datetime

# Kriptografiya
import hashlib         # MD5, SHA256
import base64          # Encode/decode
from cryptography.fernet import Fernet  # Simmetrik shifrlash

# Tizim
import subprocess      # Buyruq bajarish
import os              # Fayl tizimi
import sys             # Tizim

# Log tahlil
import re
from collections import Counter, defaultdict
```

## Cheatsheet

```python
# Hash hisoblash
import hashlib
md5 = hashlib.md5(open("file", "rb").read()).hexdigest()
sha256 = hashlib.sha256(open("file", "rb").read()).hexdigest()

# IP range tekshirish
import ipaddress
network = ipaddress.ip_network("192.168.1.0/24")
ip = ipaddress.ip_address("192.168.1.50")
print(ip in network)  # True

# DNS so'rov
import socket
ip = socket.gethostbyname("google.com")
host = socket.gethostbyaddr("8.8.8.8")[0]

# Vaqt
from datetime import datetime
now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```
