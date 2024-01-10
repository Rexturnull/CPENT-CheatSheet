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
sudo ssh -L *:80:192.168.0.24:80 administrator@192.168.0.70
# administrator / Infinit3

# Parrot web connect localhost:80

# 去觀察各機器之間的連線狀況
sudo netstat -antp | grep :80

# 192.168.0.70
sudo watch netstat -antp
```

# Remote
![](./SSH%20remote%20port%20forwarding.png)
```bash
# remote 開 port

# SSH server(70)
nano /etc/ssh/sshd_config # GatewayPorts yes
sudo service ssh restart

ssh administrator@192.168.0.70 -L *:8008:192.168.0.24:80

# Parrot web connect 192.168.0.70:8008

# 去觀察各積氣之間的連線狀況
sudo netstat -antp | grep :80
```

# Dynamic(proxychains)
![](./SSH%20dynamic%20port%20forwarding.png)
```bash
# proxychains


# Parrot
whereis proxychains
cat /usr/bin/proxychains
# Proxy Server <===> ProxyChains  | API Hooking(6個API 範圍內才可以處理)  忘了什麼意思
sudo nano /etc/proxychains.conf
# sock4 127.0.0.1 9095
# sock5 127.0.0.1 9095

# 只要連至.18:9050(主機A)的網路流量都可以透過.70(主機B)轉發出去
ssh -D 9050 administrator@192.168.0.70

proxychains curl 192.168.0.24
# |S-chain|-<>-127.0.0.1:9050-<><>-192.168.0.24:80-<><>-OK
proxychains firefox #192.168.0.24
```


# Local Double Pivot
![](./SSH%20local%20Double%20Pivot.png)
