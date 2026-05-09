# TCP — Chuqur Tahlil

## TCP nima?

TCP (Transmission Control Protocol) — **ishonchli, tartibli va xatolarni tekshiradigan** ma'lumot uzatish protokoli. OSI modelining 4-qatlami (Transport).

## TCP vs UDP

| Xususiyat | TCP | UDP |
|-----------|-----|-----|
| Ishonchlilik | Ha (ACK) | Yo'q |
| Tartib | Ha | Yo'q |
| Tezlik | Sekinroq | Tezroq |
| Ulanish | Connection-oriented | Connectionless |
| Ishlatilishi | HTTP, SSH, FTP | DNS, VoIP, Video |

## TCP Header (20 byte minimum)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window            |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## 3-yo'nalishli Salom (3-Way Handshake)

Har qanday TCP ulanishi uch qadam bilan boshlanadi:

```
Mijoz                          Server
  |                              |
  |  ----  SYN (seq=100) ---->  |   "Ulanmoqchiman"
  |                              |
  |  <-- SYN-ACK (seq=200,     |   "OK, tayyorman"
  |       ack=101) ----         |
  |                              |
  |  ---- ACK (ack=201) ---->   |   "Boshlaylik"
  |                              |
  |         [Ma'lumot uzatiladi] |
```

### Flags tushuntirish:
- **SYN** — Sinxronizatsiya, ulanish boshlash
- **ACK** — Tasdiqlash
- **FIN** — Ulanishni yopish
- **RST** — Ulanishni majburan uzish
- **PSH** — Ma'lumotni darhol yuqoriga uzatish
- **URG** — Muhim ma'lumot

## 4-yo'nalishli Xayrlashuv (4-Way Termination)

```
Mijoz                          Server
  |                              |
  |  ---- FIN ----------------> |
  |  <--- ACK ----------------  |
  |  <--- FIN ----------------  |
  |  ---- ACK ----------------> |
  |                              |
  |      [Ulanish yopildi]       |
```

## TCP Port Skanerlash (Xavfsizlik)

`nmap` bilan TCP portlarni skanerlash:

```bash
# Oddiy skan
nmap 192.168.1.1

# SYN scan (yarim ochiq, stealth)
nmap -sS 192.168.1.1

# Barcha portlar
nmap -p- 192.168.1.1

# Version aniqlash
nmap -sV 192.168.1.1

# OS aniqlash
nmap -O 192.168.1.1
```

## TCP ga Asoslangan Hujumlar

### 1. SYN Flood (DDoS)
```
Hacker ko'p SYN paketlari yuboradi → server SYN-ACK javob beradi
→ Hacker ACK yubormaydi → Server xotirasi to'lib qoladi
```
**Himoya:** SYN cookies, rate limiting, firewall

### 2. TCP Session Hijacking
Mavjud TCP sessiyasiga kirish — sequence numberlarni taxmin qilish orqali.

### 3. Port Scanning
Ochiq portlarni aniqlash — Nmap, Masscan bilan.

## Wireshark bilan TCP Kuzatish

```
# TCP handshake filtri
tcp.flags.syn == 1

# Muayyan port
tcp.port == 80

# Muayyan IP dan
ip.src == 192.168.1.100

# HTTP trafik
http
```

## Amaliy: Netcat bilan TCP ulanish

```bash
# Server tomonda (port 1234 ni tinglash)
nc -lvp 1234

# Mijoz tomonda (ulanish)
nc 192.168.1.10 1234

# Fayl yuborish
nc -lvp 1234 > received.txt    # server
nc 192.168.1.10 1234 < file.txt  # mijoz
```

## Muhim TCP Portlar

| Port | Protokol | Tavsif |
|------|----------|--------|
| 20/21 | FTP | Fayl uzatish |
| 22 | SSH | Xavfsiz uzoq kirish |
| 23 | Telnet | Shifrlanmagan uzoq kirish |
| 25 | SMTP | Email yuborish |
| 53 | DNS | Domen nomlari (TCP+UDP) |
| 80 | HTTP | Veb trafik |
| 443 | HTTPS | Xavfsiz veb trafik |
| 3306 | MySQL | Ma'lumotlar bazasi |
| 3389 | RDP | Windows uzoq ish stoli |
