# [Machine/Challenge Nomi] — HackTheBox Writeup

**Platforma:** HackTheBox  
**Tur:** Machine / Sherlock / Challenge  
**Qiyinlik:** Easy / Medium / Hard  
**OS:** Linux / Windows  
**Sana:** YYYY-MM-DD  
**IP:** 10.10.XX.XX  

---

## Qisqacha (TL;DR)

[1-2 jumlada nima qilinganini yozing — spoiler emas, faqat yo'nalish]

---

## Razvedka (Reconnaissance)

### Nmap Skan

```bash
nmap -sV -sC -p- --min-rate 5000 10.10.XX.XX -oA nmap/initial

# Natija:
PORT     STATE SERVICE VERSION
XX/tcp   open  ...
```

### Web Tekshiruv (agar web bo'lsa)

```bash
# Directory skan
gobuster dir -u http://10.10.XX.XX -w /usr/share/wordlists/dirb/common.txt

# Topilgan sahifalar:
/admin   (200)
/...
```

---

## Kirish (Initial Access)

**Zaiflik:** [CVE-XXXX-XXXX yoki zaiflik nomi]

```bash
# Exploit buyruqlari
...
```

**User Flag:** `[flag yoki ****]`

---

## Huquqni Kengaytirish (Privilege Escalation)

**Vektor:** [sudo, SUID, cron, kernel, ...]

```bash
# Tekshiruv
sudo -l
find / -perm -4000 2>/dev/null

# Exploit
...
```

**Root Flag:** `[flag yoki ****]`

---

## Xulosa

**O'rganilgan narsalar:**
- 

**MITRE ATT&CK TTPs:**
- [T1XXX — Texnika nomi]

**Foydali manbalar:**
- 

---
*HTB profil: [profil linki]*
