# CTF Resurslar va Strategiyalar

## CTF nima?

CTF (Capture The Flag) — kibersecurity musobaqasi. Maqsad: "Flag" topish (`flag{some_text_here}`).

## CTF Kategoriyalari

| Kategoriya | Tavsif | Asosiy vositalar |
|-----------|--------|------------------|
| Web | Web ilovalar zaifliklaril | Burp Suite, curl, ffuf |
| Pwn/Binary | Binary exploit | gdb, pwndbg, pwntools |
| Reverse Engineering | Dasturni tahlil | Ghidra, IDA, radare2 |
| Crypto | Kriptografiya | Python, CyberChef |
| Forensics | Raqamli sud ekspertizasi | Wireshark, Autopsy |
| Steganography | Yashirin ma'lumot | steghide, binwalk |
| OSINT | Ochiq manba razvedkasi | Google, Shodan |
| Misc | Aralash | — |

---

## Web CTF

```bash
# Directory topish
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
ffuf -u http://target.com/FUZZ -w wordlist.txt -mc 200

# Subdomain
ffuf -u http://FUZZ.target.com -w subdomains.txt -mc 200

# Fayllarni tekshirish
curl http://target.com/robots.txt
curl http://target.com/.git/config
curl http://target.com/sitemap.xml

# Cookie tahlil
# Base64 decode: eyJhZG1pbiI6ZmFsc2V9
echo "eyJhZG1pbiI6ZmFsc2V9" | base64 -d

# Source kodini ko'rish
curl -s http://target.com | grep -i "flag\|secret\|password\|comment"
```

---

## Forensics CTF

```bash
# Fayl turi aniqlash
file mystery_file
xxd mystery_file | head     # hex ko'rish

# Rasm ma'lumotlari
exiftool image.jpg
strings image.jpg | grep -i "flag\|ctf"

# Yashirin fayllar (steganography)
steghide extract -sf image.jpg
steghide info image.jpg

binwalk image.png              # ichidagi fayllar
binwalk -e image.png           # chiqarish

zsteg image.png                # PNG LSB steganography
stegsolve image.png            # grafik tahlil (Java)

# Audio steganography
sonic-visualiser audio.wav     # spektrogram
aubio track audio.wav

# PCAP tahlil
tshark -r capture.pcap
strings capture.pcap | grep -i "flag\|ctf"
tshark -r capture.pcap -Y "http" -T fields -e http.file_data

# ZIP/RAR parol
zip2john archive.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Disk image
autopsy disk.img               # grafik
fdisk -l disk.img              # bo'limlar
mount -o loop,offset=1048576 disk.img /mnt/
```

---

## Crypto CTF

```python
# CyberChef alternativasi (Python)

# Base64
import base64
base64.b64decode("SGVsbG8=")

# Hex
bytes.fromhex("48656c6c6f")

# ROT13
import codecs
codecs.encode("Hello", 'rot_13')

# Caesar cipher
def caesar(text, shift):
    return ''.join(chr((ord(c) - 65 + shift) % 26 + 65) 
                   if c.isupper() else chr((ord(c) - 97 + shift) % 26 + 97) 
                   if c.islower() else c for c in text)

# XOR
def xor(data, key):
    return bytes(a ^ b for a, b in zip(data, key * len(data)))

# RSA (n, e, c berilgan)
from Crypto.Util.number import long_to_bytes
# factordb.com yoki yafu bilan n ni factorlash
# p, q topilgandan keyin:
phi = (p-1) * (q-1)
d = pow(e, -1, phi)
m = pow(c, d, n)
print(long_to_bytes(m))
```

**Foydali saytlar:**
- https://factordb.com — RSA n factorization
- https://www.dcode.fr — Ko'p shifrlashlar
- https://gchq.github.io/CyberChef — Universal decoder

---

## Reverse Engineering

```bash
# Statik tahlil
file binary
strings binary | grep -i "flag\|ctf\|pass"
objdump -d binary | head -100
readelf -a binary

# Ghidra (GUI)
# Import → Analyze → Main funktsiyani toping

# gdb
gdb ./binary
(gdb) info functions
(gdb) disas main
(gdb) break main
(gdb) run
(gdb) x/s 0x400abc   # string ko'rish

# ltrace/strace
ltrace ./binary       # library calls
strace ./binary       # syscalls

# pwndbg (gdb extension)
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh
```

---

## OSINT CTF

```bash
# Domen ma'lumotlari
whois domain.com
nslookup domain.com
dig domain.com ANY

# Google dorks
site:example.com filetype:pdf
site:example.com "flag{"
cache:example.com
intitle:"index of" site:example.com

# Social media
sherlock username        # barcha platformalarda
holehe email@test.com    # email ro'yxatdan o'tgan saytlar

# Wayback Machine
curl "http://archive.org/wayback/available?url=example.com"

# Shodan
shodan search "example.com"
shodan host 192.168.1.1
```

---

## CTF Platformalari

| Platforma | Daraja | Tavsif |
|-----------|--------|--------|
| PicoCTF | Boshlang'ich | Maktab/universitet uchun |
| TryHackMe | Boshlang'ich-O'rta | Batafsil yo'l-yo'riq |
| HackTheBox | O'rta-Yuqori | Real muhit |
| CTFtime.org | Barcha | Musobaqalar taqvimi |
| OverTheWire | Boshlang'ich | Wargames |
| pwn.college | O'rta | Binary exploitation |
| CryptoHack | O'rta | Kriptografiya |

---

## CTF Foydali Vositalar

```bash
# O'rnatish (Kali Linux da ko'pchiligi bor)
sudo apt install binwalk steghide exiftool foremost \
  john hashcat gobuster ffuf

# Python kutubxonalar
pip install pwntools cryptography pycryptodome

# CyberChef (local)
# https://gchq.github.io/CyberChef/

# Online saytlar
# https://crackstation.net  — Hash cracking
# https://www.base64decode.org
# https://www.rapidtables.com/convert/number/hex-to-ascii.html
```

## Flag Formatlari

```
CTF{...}
flag{...}
FLAG{...}
picoCTF{...}
HTB{...}
THM{...}
```
