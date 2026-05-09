# Memory Forensics — Xotira Tahlili

## Nima uchun Memory Forensics?

RAM xotirasi — jarayonlar, shifrlash kalitlari, parollar, tarmoq ulanishlari, va hatto disk da yo'q bo'lgan **fileless malware** izlarini o'z ichiga oladi.

## Memory Dump Olish

### Windows

```bash
# WinPmem (bepul)
winpmem_mini_x64.exe memory.raw

# Magnet RAM Capture (GUI, bepul)
# https://www.magnetforensics.com/resources/magnet-ram-capture/

# DumpIt (CLI)
DumpIt.exe /O memory.raw

# Task Manager (faqat jarayon dump)
# Details → jarayon → "Create dump file"

# ProcDump (Sysinternals)
procdump.exe -ma <PID> process.dmp
```

### Linux

```bash
# avml (bepul, keng qo'llab-quvvatlash)
sudo avml /tmp/memory.lime

# LiME (Loadable Kernel Module)
sudo insmod lime.ko "path=/tmp/memory.lime format=lime"

# /proc/mem orqali (ba'zi cheklovlar)
sudo dd if=/dev/mem of=/tmp/memory.raw bs=1M

# Virtualbox VM uchun
vboxmanage debugvm "VM Name" dumpvmcore --filename=memory.raw
```

---

## Volatility — Asosiy Vosita

### O'rnatish

```bash
# Volatility 3 (Python 3, tavsiya etiladi)
git clone https://github.com/volatilityfoundation/volatility3
cd volatility3
pip install -e .

# Volatility 2 (legacy, ko'p plugin)
pip2 install volatility
```

### Profil Aniqlash (Vol 2)

```bash
# OS profilini aniqlash
volatility -f memory.raw imageinfo
volatility -f memory.raw kdbgscan

# Topilgan profildan foydalanish
export PROFILE="Win10x64_19041"
volatility -f memory.raw --profile=$PROFILE [plugin]
```

### Volatility 3 Asosiy Buyruqlar

```bash
# Jarayonlar ro'yxati
python3 vol.py -f memory.raw windows.pslist

# Jarayon daraxti
python3 vol.py -f memory.raw windows.pstree

# Yashirin/noodatiy jarayonlar
python3 vol.py -f memory.raw windows.psscan
python3 vol.py -f memory.raw windows.cmdline

# DLL ro'yxati
python3 vol.py -f memory.raw windows.dlllist --pid 1234

# Tarmoq ulanishlari
python3 vol.py -f memory.raw windows.netstat

# Registry
python3 vol.py -f memory.raw windows.registry.hivelist
python3 vol.py -f memory.raw windows.registry.printkey \
  --key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Fayllar
python3 vol.py -f memory.raw windows.filescan
python3 vol.py -f memory.raw windows.dumpfiles --pid 1234

# Malware aniqlash
python3 vol.py -f memory.raw windows.malfind
python3 vol.py -f memory.raw windows.hollowprocesses

# Hisoblar
python3 vol.py -f memory.raw windows.hashdump
python3 vol.py -f memory.raw windows.lsadump
```

### Linux Memory Tahlil

```bash
# Jarayonlar
python3 vol.py -f memory.lime linux.pslist
python3 vol.py -f memory.lime linux.pstree

# Tarmoq
python3 vol.py -f memory.lime linux.netstat

# Bash tarixi (memory da!)
python3 vol.py -f memory.lime linux.bash

# Kernel modullar
python3 vol.py -f memory.lime linux.lsmod
```

---

## Amaliy: Malware Aniqlash

### 1. Shubhali Jarayonlarni Topish

```bash
# pslist vs psscan farqi — yashirin jarayonlar
python3 vol.py -f memory.raw windows.pslist > pslist.txt
python3 vol.py -f memory.raw windows.psscan > psscan.txt
# Farqni toping!

# Noodatiy parent-child munosabat
python3 vol.py -f memory.raw windows.pstree
# word.exe → cmd.exe → powershell.exe  ← SHUBHALI!
```

### 2. Malfind — Inject kod

```bash
python3 vol.py -f memory.raw windows.malfind

# Natija:
# PID: 1234 ProcessName: svchost.exe
# VAD: 0x400000 Protection: PAGE_EXECUTE_READWRITE  ← SHUBHALI
# MZ header topildi — PE fayl inject qilingan
```

### 3. Network Ulanishlar

```bash
python3 vol.py -f memory.raw windows.netstat

# Noodatiy portlar
# svchost.exe → 192.168.1.100:4444  ← Reverse shell?
# explorer.exe → 45.33.32.156:443   ← C2 aloqa?
```

### 4. Strings + YARA

```bash
# Strings chiqarish
strings -a memory.raw > strings.txt
grep -i "password\|http\|flag\|c2\|beacon" strings.txt

# YARA bilan skan
yara rules.yar memory.raw

# Volatility YARA
python3 vol.py -f memory.raw windows.yarascan \
  --yara-rules 'rule test { strings: $a = "malware" condition: $a }'
```

---

## CTF: Memory Forensics

```bash
# 1. Dastlab nimalar bor — tekshirish
python3 vol.py -f challenge.raw windows.info
python3 vol.py -f challenge.raw windows.pslist

# 2. Buyruqlar tarixi
python3 vol.py -f challenge.raw windows.cmdline
python3 vol.py -f challenge.raw windows.consoles

# 3. Clipboard
python3 vol.py -f challenge.raw windows.clipboard

# 4. Screenshot
python3 vol.py -f challenge.raw windows.screenshot

# 5. Fayllarni chiqarish
python3 vol.py -f challenge.raw windows.filescan | grep -i "flag\|secret\|txt"
python3 vol.py -f challenge.raw windows.dumpfiles --virtaddr 0x...

# 6. Shifrlash kalitlari
python3 vol.py -f challenge.raw windows.lsadump   # LSA secrets
python3 vol.py -f challenge.raw windows.hashdump  # NTLM hashlar
```

---

## Foydali Resurslar

```
MemLabs CTF: https://github.com/stuxnet999/MemLabs
Volatility Docs: https://volatility3.readthedocs.io
HackTheBox Sherlocks: Memory forensics challenges
TryHackMe: "Volatility" room
```
