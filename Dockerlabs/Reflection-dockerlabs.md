# Reflection-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 ssh, 80 http
Users: balu, balulito, root
Vulnerability: Credentials exposed on web page, SUID /usr/bin/env
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashinani tizimga yuklab olib ishga tushursak bizga IP manzil berdi (`172.17.0.2`). Mashinaning IP manzilini olganimizdan so'ng, `nmap` buyrug'i orqali uning ochiq portlarini aniqlab olamiz.

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

Nmap natijasiga ko'ra 2 ta port ochiq ekan:
- 22 port — SSH service
- 80 port — Web service

Brauzerda IP manzil terib ishlab turgan sahifani ochdim. Bu sahifada zaifliklar haqida ma'lumot berilgan va mashq qilsa bo'ladigan vazifalar bor ekan. Eng pastida "Vazifalarni yakunlaganingizda bosing" degan tugma bor ekan — uni bosganimda credential berdi.

---

# 2. Initial Access

Topilgan credentialdan foydalanib SSH orqali userniga muvofaqiyatli ulandim. Bu mashinada flag qoldirilmagan ekan, shuning uchun keyingi qadam rootga o'tish bo'ldi.

---

# 3. Privilege Escalation

Kirgan foydalanuvchimda sudo huquqi yo'q ekan. SUID biti biriktirilgan fayllarni qidirganimda `/usr/bin/env` ga SUID biriktirilganini ko'rdim.

```bash
find / -perm -4000 2>/dev/null
```

```bash
/usr/bin/env /bin/sh -p
```

Bu buyruq orqali `root` huquqini qo'lga kiritdim.

---

**Mashina muvofaqiyatli yechildi!!!**
