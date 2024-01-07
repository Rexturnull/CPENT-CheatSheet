IOT
===
🔙 [MENU README](../README.md)

> 韌體拆解

# Enviroment
```bash
自己的Linux配上檔案
```
```bash
# 韌體檔案壓縮工具
# Parrot
git clone https://github.com/useidel/sasquatch.git
sh build.sh
# kali
git clone --quiet --depth 1 --branch "master" https://github.com/devttys0/sasquatch
cd sasquatch
wget https://github.com/devttys0/sasquatch/pull/47.patch && patch -p1 < 47.patch && sudo ./build.sh

# 分析二進制檔案工具
binwalk
```

# Find Something
```bash
### Windows 
dir /s <FILE_NAME> 2> nul 
findstr /n /i /s <KEYWORD> *.*

### Linux 
find / -name <FILE_NAME> -ls 2> /dev/null
grep -nir <KEYWORD> .
```


# Dlink_firmware.bin
Target
```bash
找到telnet的密碼
```
```bash
binwalk -t -e Dlink_firmware.bin
cd _Dlink_firmware.bin.extracted
grep -nir telnet . 
# etc/scripts/misc/telnetd.sh
```
```bash
cat etc/scripts/misc/telnetd.sh

#!/bin/sh
image_sign=`cat /etc/config/image_sign`
TELNETD=`rgdb -g /sys/telnetd`
if [ "$TELNETD" = "true" ]; then
        echo "Start telnetd ..." > /dev/console
        if [ -f "/usr/sbin/login" ]; then
                lf=`rgdb -i -g /runtime/layout/lanif`
                telnetd -l "/usr/sbin/login" -u Alphanetworks:$image_sign -i $lf &
        else
                telnetd &
        fi
fi
```
```bash
find / -name image_sign -ls 2> /dev/null

cat /home/kali/Desktop/IOT/_Dlink_firmware.bin.extracted/squashfs-root/etc/config/image_sign
# wrgn23_dlwbr_dir300b
```