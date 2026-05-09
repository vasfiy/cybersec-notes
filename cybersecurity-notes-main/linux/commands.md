# Linux Buyruqlar — Tez Qo'llanma

## Fayl Boshqaruvi

```bash
# Ko'rish
ls -la              # batafsil ro'yxat
ls -lah             # o'lchamlar o'qiladigan formatda
find . -name "*.txt"
find / -user root -perm -4000   # SUID fayllar
locate filename

# Navigatsiya
pwd                 # joriy katalog
cd /home/user       # katalogga o'tish
cd ..               # yuqoriga
cd -                # oldingi katalog

# Yaratish/O'chirish
mkdir -p a/b/c      # rekursiv yaratish
touch file.txt      # fayl yaratish
cp file.txt /tmp/   # nusxa
cp -r dir/ /tmp/    # katalog nusxasi
mv file.txt new.txt # ko'chirish/nomlash
rm file.txt         # o'chirish
rm -rf dir/         # katalogni o'chirish (EHTIYOT!)

# Ko'rish
cat file.txt
less file.txt       # qulf bilan ko'rish (q - chiqish)
head -20 file.txt   # birinchi 20 qator
tail -20 file.txt   # oxirgi 20 qator
tail -f /var/log/syslog  # real vaqt

# Qidirish
grep "xato" file.txt
grep -r "password" /etc/    # rekursiv
grep -i "error" log.txt     # katta-kichik harfga e'tibor bermaslik
grep -n "GET" access.log    # qator raqami
grep -v "200" access.log    # teskari (200 bo'lmaganlar)

# Matni qayta ishlash
cat file.txt | sort
cat file.txt | sort | uniq   # takrorlanmaganlar
cat file.txt | wc -l         # qator soni
awk '{print $1}' file.txt    # birinchi ustun
sed 's/old/new/g' file.txt   # almashtirish
cut -d: -f1 /etc/passwd      # `:` bilan ajratib birinchi maydon
```

## Arxivlash

```bash
# tar
tar -czf archive.tar.gz /path/    # yaratish (gzip)
tar -xzf archive.tar.gz           # ochish
tar -tzf archive.tar.gz           # tarkibini ko'rish

# zip
zip -r archive.zip /path/
unzip archive.zip

# Parolsiz zip ochish
zip2john archive.zip > hash.txt
john hash.txt
```

## Tarmoq

```bash
# Ping
ping google.com
ping -c 4 google.com    # 4 marta

# Traceroute
traceroute google.com
mtr google.com          # real vaqt

# Port tekshirish
nc -zv 192.168.1.1 80
nc -zv 192.168.1.1 1-1000   # port range

# Fayl uzatish (Netcat)
# Qabul qiluvchi:
nc -lvp 4444 > received_file

# Yuboruvchi:
nc TARGET_IP 4444 < file_to_send

# wget/curl
wget https://example.com/file.zip
curl -O https://example.com/file.zip
curl -s https://api.example.com/data | python3 -m json.tool

# SSH
ssh user@192.168.1.10
ssh -p 2222 user@host           # boshqa port
ssh -i ~/.ssh/id_rsa user@host  # kalit bilan
ssh -L 8080:localhost:80 user@host  # tunnel

# SCP (fayl ko'chirish)
scp file.txt user@host:/tmp/
scp -r dir/ user@host:/tmp/
scp user@host:/tmp/file.txt .
```

## Tizim Ma'lumotlari

```bash
# Tizim
uname -a            # kernel va OS
hostname
cat /etc/os-release

# Disk
df -h               # bo'sh joy
du -sh /home/       # katalog o'lchami
lsblk               # disklar

# RAM
free -h
cat /proc/meminfo

# CPU
lscpu
cat /proc/cpuinfo | grep "model name" | head -1

# Uptime
uptime
w

# Muhit o'zgaruvchilari
env
echo $PATH
export MY_VAR="value"
```

## Foydalanuvchi Almashtirish

```bash
su username          # foydalanuvchiga o'tish
su -                 # root ga (to'liq muhit)
sudo command         # root huquqi bilan ishga tushirish
sudo -i              # root shell
sudo -u www-data cmd # boshqa foydalanuvchi bilan
```

## Skriptlash Asoslari

```bash
#!/bin/bash
# Skript boshlanishi

# O'zgaruvchi
NAME="Ali"
echo "Salom, $NAME"

# Shartli ifoda
if [ -f "/etc/passwd" ]; then
    echo "Fayl mavjud"
fi

# Sikl
for i in 1 2 3; do
    echo "Raqam: $i"
done

# Fayl orqali sikl
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Funktsiya
scan_host() {
    nmap -sV "$1"
}
scan_host 192.168.1.1
```

## One-Liner Foydali Buyruqlar

```bash
# Barcha ochiq portlar
ss -tuln | awk 'NR>1 {print $5}' | cut -d: -f2 | sort -n | uniq

# Tarmoqdagi qurilmalar
arp -a
nmap -sn 192.168.1.0/24

# Oxirgi 100 buyruq
history | tail -100

# Barcha sudo foydalanuvchilar
grep -Po '^sudo.+:\K.*$' /etc/group

# Python HTTP server (fayl ulashish)
python3 -m http.server 8080

# Base64 encode/decode
echo "text" | base64
echo "dGV4dA==" | base64 -d

# MD5/SHA hash
md5sum file.txt
sha256sum file.txt
echo -n "password" | sha256sum
```
