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
dir /s <FILE_NAME> 2> nul  #file
findstr /n /i /s <KEYWORD> *.* #content

### Linux 
find / -name <FILE_NAME> -ls 2> /dev/null  #file
find / -iname <FILE_NAME> 2> /dev/null  #file
grep -nir <KEYWORD> . #content
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

# D6000-V1.0.0.41_1.0.1.bin
Target
```bash
找到6組密碼 : 5組明碼、1組密文
```
```bash
binwalk -t -e DVRF_v03.bin

# 明碼
grep -nir passwd .

# 密文
find . -name passwd -ls 2> /dev/null


cat squashfs-root/usr/etc/passwd
# admin:$1$$iC.dUsGpxNNJGeOm1dFio/:0:0:root:/:/bin/sh
```
Crack
```bash
john squashfs-root/usr/etc/passwd
# 1234
```


# encrypted.bin
```bash
binwalk -t encrypted.bin
# Nothing
```
Entropy
```bash
binwalk -E encrypted.bin 

# When Error
pip install --upgrade matplotlib
rm -r ~/.cache/matplotlib/
sudo apt remove python3-numpy
pip3 install numpy


# Entropy Range : 1~10
# Entropy 的值越高，表示資訊的不確定性越大，文件內容越雜亂。
# Entropy 的值越低，表示資訊的不確定性越小，文件內容越有規律。

# 通常一篇英文文章大約在4-6
```
Dlink_firmware.bin
```bash
# 根據Entropy分析
# 基本上該檔案不是被加密就是被壓縮
# 但可以觀察到中間和尾段都有一段是稍微沒有那麼亂的
# 觀察Dlink_firmware.bin中間跟尾段的雜亂程度
hexdump -C -v Dlink_firmware.bin
# 可以看出這段都是00組成

```
encrypted.bin
```bash
# 換句話說
# encrypted.bin中間跟尾段有一段是以差不多的東西組成的
hexdump -C -v encrypted.bin


# 思考什麼情況會有這樣的結果呢
# 00在什麼情況下做完處理結果跟一開始差不多
# XOR真直表
  0 1 -> key
0 0 1 -> cipher
1
```
Steam XOR
```bash
# Dlink_firmware.bin 一般檔案的最高頻率部分
hexdump -C -v Dlink_firmware.bin | cut -d ' ' -f3-20 | sort | uniq -c | sort -nr | head -n 20
  2511 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
     4 00 00 00 01 00 00 00 00  00 00 3f 91 45 84 68 3b
     4 00 00 00 00 00 3f 91 45  84 68 3b de de a6 0f 23


# encrypted.bin 檔案的最高頻率部分
hexdump -C -v encrypted.bin | cut -d ' ' -f3-20 | sort | uniq -c | sort -nr | head -n 20
    111 88 44 a2 d1 68 b4 5a 2d  88 44 a2 d1 68 b4 5a 2d
     19 88 45 fb 8b 95 83 20 75  d2 44 a2 d0 01 96 84 1b
     16 b6 74 af 5a 6a b4 5a 2d  88 45 fb 8b 95 83 20 75


# Get XOR Key
88 44 a2 d1 68 b4 5a 2d
```
xcat
```bash
> https://github.com/mstrand/xcat.git

chmod +x xcat.py

./xcat.py -x <xor_key> encrypted.bin > decrypted.bin

./xcat.py -x 8844a2d168b45a2d ../encrypted.bin > ../decrypted.bin

# Now we can analyze
binwalk -t decrypted.bin
```

# encrypted.bin - XORTool
CTF XOR 速解
```bash
# install
python -m pip install xortool
git clone https://github.com/hellman/xortool.git
cd xortool
poetry build
pip install dist/xortool*.whl


# xortool
xortool enctypted.bin
xortool cipher -c " "
xortool encrypted.bin -c 00
# -c:指定最常出現的字(一般是空格)
# -m:指定最大嘗試的 key 長度


# decrypted
xortool enctypted.bin -l 8 -c 00
binwalk -t -e xortool_out/0.out

# key
cat xortool_out/filename-key.csv
python -c "print(b'\x88D\xa2\xd1h\xb4Z-'.hex())"
```
