# Wireshark — Tarmoq Tahlili

## Wireshark nima?

Wireshark — tarmoq paketlarini ushlash va tahlil qilish uchun grafik interfeys vositasi. Terminal alternativasi: `tcpdump`.

## O'rnatish

```bash
# Ubuntu/Debian
sudo apt install wireshark

# Wireshark ni root siz ishlatish
sudo usermod -aG wireshark $USER
# Tizimdan chiqib kiring va qayta kiring
```

## tcpdump — Terminal alternativasi

```bash
# Barcha trafikni ushlash
tcpdump -i eth0

# Faylga yozish
tcpdump -i eth0 -w capture.pcap

# pcap faylni o'qish
tcpdump -r capture.pcap

# HTTP trafik
tcpdump -i eth0 port 80

# Muayyan IP
tcpdump -i eth0 host 192.168.1.100

# DNS trafik
tcpdump -i eth0 port 53

# Verbose + mazmunli ko'rsatish
tcpdump -i eth0 -A -s 0 port 80

# SYN paketlari (port skanerni aniqlash)
tcpdump 'tcp[tcpflags] & tcp-syn != 0'
```

## Wireshark Filtrlari

### Display Filtrlari (ushlangandan keyin)

```
# Protokol
http
tcp
udp
dns
icmp
arp
ssl

# IP manzil
ip.addr == 192.168.1.1
ip.src == 192.168.1.1
ip.dst == 192.168.1.1

# Port
tcp.port == 80
udp.port == 53
tcp.dstport == 443

# HTTP
http.request.method == "POST"
http.response.code == 200
http.host contains "example"

# TCP Flags
tcp.flags.syn == 1
tcp.flags.ack == 1
tcp.flags.rst == 1

# Kombinatsiya
ip.addr == 192.168.1.1 && tcp.port == 80
http || dns
!(arp)
```

### Capture Filtrlari (ushlash paytida)
```
# Muayyan host
host 192.168.1.1

# Port
port 80
port 80 or port 443

# Subnet
net 192.168.1.0/24

# TCP SYN
tcp[tcpflags] & tcp-syn != 0
```

## Muhim Tahlil Texnikalari

### 1. HTTP ma'lumotlarini chiqarish
```
File → Export Objects → HTTP
→ Barcha HTTP fayllarni saqlash
```

### 2. TCP Stream Ko'rish
```
Paket ustida o'ng tugma → Follow → TCP Stream
→ To'liq suhbatni ko'rish
```

### 3. Statistika
```
Statistics → Conversations   — Kim kim bilan gaplashgan
Statistics → Protocol Hierarchy — Qaysi protokollar qancha
Statistics → IO Graph — Vaqt bo'yicha trafik
Analyze → Expert Information — Xatolar va ogohlantirishlar
```

### 4. Shifrlangan bo'lmagan parollarni topish
```
# HTTP Basic Auth
http.authorization

# FTP parol
ftp.request.command == "PASS"

# Telnet
Display filter: telnet
Follow TCP Stream
```

## CTF da Wireshark

```bash
# pcap fayldan ma'lumot chiqarish
tshark -r capture.pcap

# Muayyan protokol
tshark -r capture.pcap -Y "http" -T fields -e http.request.uri

# Barcha HTTP URL
tshark -r capture.pcap -Y "http.request" -T fields \
  -e ip.src -e http.host -e http.request.uri

# DNS so'rovlari
tshark -r capture.pcap -Y "dns.qry.type == 1" -T fields \
  -e dns.qry.name

# Fayllarni chiqarish
tshark -r capture.pcap --export-objects http,./extracted/

# Strings (oddiy usul)
strings capture.pcap | grep -i "flag\|password\|secret"
```

## Anomaliyalarni Aniqlash

```
# Port skan belgilari
- Ko'p SYN paketlari, kam SYN-ACK
- Bitta IP dan ko'p portlarga ulanish
- tcp.flags.syn == 1 && tcp.flags.ack == 0

# DDoS
- Ko'p IP lardan bitta maqsadga
- ICMP flood: icmp.type == 8

# ARP Spoofing
- Bir MAC ikkita IP e'lon qilishi
- arp.duplicate-address-detected

# DNS Tunneling
- DNS so'rovlarida g'ayrioddiy uzun nomlar
- dns.qry.name uzunligi > 50
```
