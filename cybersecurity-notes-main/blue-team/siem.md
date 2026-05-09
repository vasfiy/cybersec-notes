# SIEM — Security Information and Event Management

## SIEM nima?

SIEM — turli manbalardan loglarni yig'ib, korrelyatsiya qilib, xavfsizlik hodisalarini aniqlash tizimi.

```
Manbalar → [SIEM] → Korrelyatsiya → Alert → SOC Analitik
```

## Asosiy SIEM Tizimlari

| Tizim | Turi | Tavsif |
|-------|------|--------|
| Splunk | Tijorat | Eng keng tarqalgan |
| IBM QRadar | Tijorat | Enterprise |
| Microsoft Sentinel | Cloud | Azure asosida |
| Elastic SIEM | Open Source | ELK Stack |
| Wazuh | Open Source | OSSEC asosida |
| Graylog | Open Source | Log management |

## ELK Stack (Elasticsearch, Logstash, Kibana)

```
Loglar → Logstash (yig'ish/filter) → Elasticsearch (saqlash) → Kibana (vizual)
```

### O'rnatish (Docker)
```bash
docker run -d -p 9200:9200 \
  -e "discovery.type=single-node" \
  elasticsearch:8.0.0

docker run -d -p 5601:5601 \
  --link elasticsearch:elasticsearch \
  kibana:8.0.0
```

### Logstash config
```ruby
input {
  file {
    path => "/var/log/auth.log"
    start_position => "beginning"
  }
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{SYSLOGLINE}" }
  }
  date {
    match => [ "timestamp", "MMM d HH:mm:ss" ]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "security-logs-%{+YYYY.MM.dd}"
  }
}
```

## Log Manbalari

```
Linux:
/var/log/auth.log      — Autentifikatsiya (SSH, sudo)
/var/log/syslog        — Umumiy tizim
/var/log/apache2/      — Web server
/var/log/nginx/        — Web server
/var/log/fail2ban.log  — Brute force himoyasi

Windows Event IDs:
4624 — Muvaffaqiyatli login
4625 — Muvaffaqiyatsiz login
4648 — Aniq credential bilan login
4720 — Yangi hisob yaratildi
4732 — Guruhga qo'shildi
4740 — Hisob bloklandi
7045 — Yangi xizmat o'rnatildi
```

## Kibana Qidiruvlari (KQL)

```
# Muvaffaqiyatsiz loginlar
event.code: 4625

# Muayyan IP dan ulanishlar
source.ip: "192.168.1.100"

# SSH hujum
message: "Failed password" AND message: "ssh"

# Admin foydalanuvchi amallari
user.name: "administrator" AND event.category: "authentication"

# Port skaneri aniqlash
destination.port: [1 TO 1024] AND event.type: "connection"
```

## Wazuh — Open Source SIEM

```bash
# Agent o'rnatish (Linux)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" \
  | tee -a /etc/apt/sources.list.d/wazuh.list
apt-get install wazuh-agent

# Manager ga ulash
/var/ossec/bin/agent-auth -m MANAGER_IP

# Status tekshirish
systemctl status wazuh-agent
```

### Wazuh Qoidalar
```xml
<!-- /var/ossec/etc/rules/local_rules.xml -->
<group name="custom">

  <!-- Brute force aniqlash -->
  <rule id="100001" level="10" frequency="5" timeframe="30">
    <if_matched_sid>5716</if_matched_sid>
    <description>SSH Brute Force - 5 attempts in 30 seconds</description>
  </rule>

  <!-- Yangi foydalanuvchi -->
  <rule id="100002" level="12">
    <if_sid>5902</if_sid>
    <match>useradd</match>
    <description>New user account created</description>
  </rule>

</group>
```

## SOC Analitik Ko'nikmalar

```
Tier 1 — Alert Monitoring:
□ SIEM dashboard kuzatish
□ Alert triage
□ False positive aniqlash
□ Ticket yaratish

Tier 2 — Incident Investigation:
□ Deep log tahlil
□ Packet tahlil
□ IOC qidirish
□ Timeline yaratish

Tier 3 — Threat Hunting:
□ Proaktiv tahdid qidirish
□ Malware tahlil
□ Threat intelligence
□ YARA qoidalar yaratish
```

## Splunk Qidiruvlari (SPL)

```
# Muvaffaqiyatsiz loginlar
index=windows EventCode=4625 | stats count by src_ip | sort -count

# Top 10 shubhali IP
index=network | stats count by src_ip | sort -count | head 10

# DNS anomaliya
index=dns | stats count by query | where count > 100 | sort -count

# Port skan aniqlash
index=firewall | stats dc(dest_port) as port_count by src_ip 
| where port_count > 100

# Geolocation anomaliya
index=vpn action=success | iplocation src_ip 
| stats values(Country) as countries, dc(Country) as country_count 
  by user | where country_count > 2
```
