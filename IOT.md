IOT
===

韌體拆解
```bash
binwalk -t DVRF_v03.bin

# 韌體檔案壓縮工具
git clone https://github.com/useidel/sasquatch.git

binwalk -t -e DVRF_v03.bin

grep -nir telnet .

find . -iname passwd

# D6000 5+1組密碼  五組明碼 一組密文


# encrypted.bin
hexdump -C -v encrypted.bin

# Entropy 1~10
# English 4-6
# Steam XOR
binwalk -E encrypted.bin 

# XOR
  0 1 -> key
0 0 1 -> cipher
1

hexdump -C -v Dlink_firmware.bin | cut -d ' '-f3-20 | sort | uniq -c | sort -nr | head -n 20
# got key
hexdump -C -v encrypted.bin | cut -d ' '-f3-20 | sort | uniq -c | sort -nr | head -n 20

python3 xcat.py $KEY encrypted.bin > decrypted.bin



# CTF xortool
python3 -m pip install xortool
xortool encrypted.bin
xortool encrypted.bin -l 8 -c 00

python3 -c "print(b'....'.hex())"
```


