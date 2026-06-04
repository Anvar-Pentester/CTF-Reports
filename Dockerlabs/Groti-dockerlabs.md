# Groti-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 SSH, 80 http, 3306 mysql
Users: grooti, root, mysql
Vulnerability: Leaked credentials, SUID /bin/bash
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashimiz nomi `Grooti`. Mashinani IP adresi berilgan, undan foydalanib quyidagi buyruq orqali ochiq portlarini aniqlab oldim.

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

Natija:
- 22 SSH
- 80 http
- 3306 mysql

80 portda ishlab turgan web sahifani brauzerda ochib ko'rdim. Bu yerda Groot ning sahifasi chiqib keldi. `My photos`, `My database` va `Ship invoices` deb nomlangan sahifalar bor ekan.

`My Photos` sahifasida `README.txt` va grootining rasmi bor ekan.

`README.txt` ichida `password1` paroli chiqdi, va oldida "Uni qayerga qo'yishni top" degan yozuv turibti.

`My database` ichida hech narsa yo'q. `My invoices` ichida esa fayl yo'llari jadval ko'rinishida joylashgan ekan.

Asosiy sahifani `devtools` qismini ochdim. Bu yerda "Men Grootiman... O'ylaymanki Rocket mening ma'lumotlar bazasiga kirdi..." deyilgan komentariya qolib ketgan.

---

# 2. Exploitation

`password1` paroli bor, va nmap natijasida mysql database borligini ko'rgan edim. Komentariyada `Rocket` esga olingan. Shulardan kelib chiqib `Rocket` orqali mysql ga kirishim mumkin deb tahmin qildim.

```bash
mysql -h 172.17.0.2 -u rocket -ppassword1
```

Database ga kirdim, u yerda 3 ta DB bor ekan:
- `files_secret` — bu DB da kerakli ma'lumot topish mumkin
- `information_schema` — default DB
- `performance_schema` — default DB

```bash
USE files_secret;
SELECT * FROM rutas;
```

`rutas` tablening qiymatida fayllarning pathi turgan ekan. `secret` faylining pathi meni grooti terminaliga olib keldi.

Bu sahifada SQL Injection sinab ko'rdim foyda bermadi. Raqam tanlash maydonida nechi raqamni kiritsam shu raqamdagi `password.txt` nomli faylni ko'chirib berdi.

Barcha raqamdagi fayllarni ko'chirayotganimda 16-raqamda qolganlaridan farq qiluvchi fayl ko'chirildi — bu zip fayl va hajmi katta edi.

Faylni zip dan ochmoqchi bo'lsam, parol bilan himoyalangan ekan. `john` orqali bu faylni avval hashini olib, keyin parolini topdim.

```bash
zip2john password16.txt > password.hash
john --wordlist=/usr/share/wordlists/rockyou.txt password.hash
```

Natijada faylni paroli topildi — `password1`. Topilgan parol orqali zip faylni ochdim. Ochilgan faylda parollar ro'yhati bor ekan.

---

# 3. Initial Access

Parollar ro'yhati topilgandan so'ng `hydra` orqali brute force qilib tizimga kirishga harakat qildim. Mashinada `Grooti` nomi ko'p joyda uchragani sababli, `grooti` foydalanuvchisi bo'lsa kerak deb tahmin qildim.

```bash
hydra -l grooti -P password16.txt ssh://172.17.0.2
```

Tahminim to'g'ri chiqdi. `grooti` foydalanuvchisi bor ekan va uning paroli `YoSoYgRo0t` ekan. Topilgan credentiallardan foydalanib tizimga muvofaqiyatli kirdim.

---

# 4. Privilege Escalation

Tizimga kirgandan so'ng `root` huquqini olish yo'llarini qidirdim. `/tmp` papkasida `malicious.sh` faylini topdim. Bu faylga yozish huquqim bor edi.

Men faylni ichini quyidagi kod bilan o'zgartirdim:

```bash
chmod u+s /usr/bin/bash
```

Bu komanda orqali `/usr/bin/bash` ga SUID bitini uladim.

```bash
ls -l /usr/bin/bash
```

SUID bog'langandan so'ng, quyidagi buyruq orqali `root` huquqini oldim:

```bash
/bin/bash -p
```

---

**Mashina muvofaqiyatli yechildi!**
