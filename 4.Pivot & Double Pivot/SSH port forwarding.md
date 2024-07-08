SSH port forwarding
===
🔙 [MENU README](./Pivot%20&%20Double%20Pivot.md)

# Socket Pair
```bash

```

# Local
![](./SSH%20local%20port%20forwarding.png)
```bash
# local 開 port
```
```bash
# Parrot Command
sudo ssh -L *:80:192.168.0.24:80 administrator@192.168.0.70
# administrator / Infinit3
# Not Need Shadow Terminal
sudo ssh -nNTL *:80:192.168.0.24:80 administrator@192.168.0.70

# Connect
Parrot web connect to localhost:80
nc 127.0.0.1 80

# 去觀察各機器之間的連線狀況
sudo netstat -antp | grep :80
# :80  => 開在127.0.0.1
# *:80 => 開在0.0.0.0

# 192.168.0.70
sudo watch netstat -antp
```

# Remote
![](./SSH%20remote%20port%20forwarding.png)
```bash
# 在remote 開 port
# 由 Remote Forwarding 和一台對外機器
# 可以讓你從任何地方連回這個服務
```
```bash
# Setting SSH server(70)
# 基於安全考量， Remote Forwarding 預設都只能夠 bind 在 SSH Server 的 localhost 上
nano /etc/ssh/sshd_config # GatewayPorts yes
sudo service ssh restart

# Parrot Command
ssh administrator@192.168.0.70 -R *:8008:192.168.0.24:80

# Connect
Parrot web connect to  192.168.0.70:8008

# 去觀察各機器之間的連線狀況
sudo netstat -antp | grep :80
```
Windows
```bash
plink.exe -ssh kali@172.27.234.2 -R 445:172.25.170:445
```

# Dynamic(proxychains)
![](./SSH%20dynamic%20port%20forwarding.png)
```bash
# Parrot
whereis proxychains
cat /usr/bin/proxychains
# Proxy Server <===> ProxyChains
# 不斷追linking後可以發現proxychain的實現方式是用API Hooking(LD_PRELOAD)
# 再去追原始碼後可以發現API是透過LD_PRELOAD的方式hooking connect()
# 所以應用程式網路段用的不是用connect寫的則proxychains無法代理
sudo nano /etc/proxychains.conf
# sock4 127.0.0.1 9095
# sock5 127.0.0.1 9095

# 只要連至.18:9050(主機A)的網路流量都可以透過.70(主機B)轉發出去
ssh -D 9050 administrator@192.168.0.70
ssh -nNTCD 9050 administrator@192.168.0.70

# Connect
proxychains curl 192.168.0.24
# |S-chain|-<>-127.0.0.1:9050-<><>-192.168.0.24:80-<><>-OK
proxychains firefox # 192.168.0.24
```


# Local Double Pivot
![](./SSH%20local%20Double%20Pivot.png)
```bash
# -J : 指定一個跳板主機（Jump Host）以連接到目標主機
```
```bash
ssh -J administrator@192.168.0.70 administrator@192.168.0.10 -L *:80:192.168.0.24:80
# 192.168.0.70 : administrator / Infinit3
# 192.168.0.10 : administrator / toor


# Parrot web connect localhost:8888

# 去觀察192.168.0.24的連線狀況
# 可看到來自192.168.0.10的連線
sudo netstat -antp
```