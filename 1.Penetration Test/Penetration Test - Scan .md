Penetration Test - Scan
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

# IP Scan
Parrot Environment
```bash
ip route

# 172.19.19.0/24 dev eth1 proto kernel scope link src 172.19.19.18
# 192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.18
```
Nmap Parameter
```bash
-sT           TCP完整掃描(SYN、SYN+ACK、ACK)
              完整連線易留下紀錄 
              效率上實測比sS快很多
-sS           SYN半開放式掃描
-sP           ping ICMP 掃描(掃有開機的主機)
-sV           各種服務
-sU           UDP掃描
-sC           自動腳本
-sn           Ping scan 一開始想針對大範圍網段，確認同網段的主機哪些有開機，適合使用
-Pn           PingNo掃描之前不需要用ping命令，有些防火牆禁止ping命令
-O            作業系統
-A            更加詳細的輸出(作業系統與各種服務)
-n            不執行 DNS 解析，直接使用 IP 地址
-p-           0-65535所有訊息
-p 1-65535    PORT Nmap 為了掃描效率，只會掃描常見的 1000 個 Port
-p T:80       TCP
-P U:53       UDP
--script vuln 易遭受攻擊的漏洞
-T4           掃描速度如果網路環境穩定，參數建議調整為 -T4
              無論如何，一定要開啟增進掃描效率
-oG -         選項用於生成 grepable（可供 grep 使用）的輸出格式
```
Sudo vs UnSudo
```bash
nmap 192.168.0.7 -sn -n --packet-trace
# 觀察 : CONN 是使用 TCP 80 ,443 去偵測主機

sudo nmap 192.168.0.7 -sn -n --packet-trace
# 觀察 : 
# 這次是使用 SENT、RCVD去偵測主機
# 也就是這次有權限可以call raw socket使用ARP去偵測
# 防火牆是無法阻擋ARP的，ARP是網路的基本


# 加了sudo就無敵了嗎
sudo nmap scanme.nmap.org -sn -n --packet-trace
# 觀察 : 
# 這次使用ICMP type=8 , ICMP type=13
# CONN使用 80,443
# 換句話說，如果這些防火牆全部檔掉的話，就找不到這台機器


# 有沒有什麼掃描基本上沒問題的解決方法呢?
sudo nmap scanme.nmap.org -sn -PS22,80,445,3389 -n --packet-trace
# Windows : TCP 139,445,3389
# Linux   : TCP 22,80
# 通常來說，如果主機是正常的機器這些會開，可以找這些port來判斷這台機器的狀態
```
IP Scan Script
```bash
# Nmap
nmap 192.168.0.* -sn -oG -
nmap 192.168.0.* -sn -oG - | grep Up | cut -d' ' -f2 > ip_list

# Port
sudo nmap -n -sn -PS22,80,445,3389 192.168.0.1-254 -oG - | grep Up | cut -d' ' -f2 > ip_list

sudo nmap 192.168.0.* -n -sn -PS22,80,445,3389 -oG > ip_list

# Icmp
ping -c 1 192.168.0.7
ping -c 1 192.168.0.7 | grep "bytes from"
for ip in {1..254};do (ping -c 1 192.168.0.$ip | grep "bytes from" &);done
# & : 放入背景執行，這樣在每個IP的ping命令都在背景執行的同時，腳本可以繼續迭代下一個IP

#這個才是精隨，Icmp前的arp是檔不住的
cat /proc/net/arp 
cat /proc/net/arp | grep -v 00:00:00:00:00:00 | grep eth0 | cut -d' ' -f1
```

---

# Port Scan
Nmap Default
```bash
nmap 192.168.0.70
nmap 192.168.0.70 --top 1000
# 觀察 : 
# Nmap 預設掃描1000個port
# 可以到/usr/share/nmap/nmap-services去看是哪些

# 看看Nmap優先掃描的前三個port是什麼
nmap 192.168.0.70 --top 3 #23 80 443
nmap 192.168.0.70 --top 1 #80

# 全Port掃描
# - 之前沒填代表1
# - 之後沒填代表65536
nmap 192.168.0.70 -p -
```
hping3
```bash
# Ping的加強版，可以處理防火牆的問題
# 因為他不只利用icmp去執行ping這個動作
sudo hping3 192.168.0.7 -n -S -c 3 -p 80
```
Port Scan Script
```bash
# TCP
bash -c 'echo > /dev/tcp/192.168.0.70/22 && echo open'
bash -c 'for port in {1..100}; do (echo > /dev/tcp/192.168.0.70/$port && echo $port open &);done' 2> /dev/null


# UDP
sudo nmap -sU -p137-139 192.168.0.70  # netbios
# 觀察 :
# ICMP type=3 port unreachable的回應，通常表示該UDP端口是閉合的
# 因為UDP是一種無連接的協定，無法像 TCP 那樣進行三次握手等連接建立的過程
# 如果端口是閉合的，目標主機則通常會回應 port unreachable這樣的錯誤消息

# UDP 通常會開的Port
53,69,137,138,161,1900,5353  # 考試port:161須注意
```
RustScan
```bash
# 一個快速且暴力的TCP從1掃到65535的工具
# https://github.com/RustScan/RustScan/releases


# Install
wget https://github.com/RustScan/RustScan/releases/download/2.0.1/rustscan_2.0.1_amd64.deb
sudo dpkg -i rustscan_2.0.1_amd64.deb

# Command
rustscan -a 192.168.0.7 -u 5000 -t 8000 
rustscan -a 192.168.0.7 -u 5000 -t 8000 --scripts none
rustscan -a 192.168.0.7 -u 5000 -t 8000 --scripts -- -n -Pn -sVC -oG
# -u : Linux 預設的 soft limit 太低(預設為 1024)，會導致 RustScan 發揮不出原本應有的速度
```


---


# Service Scan
Service
```bash
nmap 192.168.0.7 -p47001
# Windows Remote Management（WinRM）
# WinRM 是一項 Microsoft 技術，用於管理和遠程執行 Windows 作業系統的操作
# 支援通過SOAP（Simple Object Access Protocol）
# 和XML（Extensible Markup Language）在HTTP協定上進行通信

# Windows
sudo nmap -n -p445,3389 192.168.0.8,20 -sVC
# Linux
sudo nmap -n -p22,80 192.168.0.24,70 -sVC
```
Service Script Scan
```bash
nmap 192.168.0.7 -p445 -sV -sC
# -sC  :  ls /usr/share/nmap/scripts
nmap 192.168.0.7 -p445 -script vulners
nmap 192.168.0.7 -sV -p445 -script smb-protocols
```

---

