Pivot & Double Pivot
===
> 跳板
> via Port Fowarding / proxy /redirection Tunneling

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

# Enviroment
Parrot 
```bash
# eth0 : 192.168.0.18
# eth1 : 172.19.19.18
sudo ifconfig eth1 down
```
SSH
```bash
# eth0 : 192.168.0.70
# eth1 : 172.19.19.70
```
Wordpress
```bash
# eth0 : 192.168.0.24
# eth1 : 172.19.19.24
sudo ifconfig eth0 down

sudo nano /var/www/index.html
#172.19.19.24
```



# 4-Ways 
- [SSH port forwarding (SSH Tunneling)](./SSH%20port%20forwarding.md)
- [Port forwarding](./Port%20forwarding.md)
- [Meterpreter](./Meterpreter.md)
- [Chisel](./Chisel.md)