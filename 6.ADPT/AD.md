AD
===
🔙 [MENU README](../README.md)

# Tree 、 Forest
Tree
```
red.com
sub.red.com
==========================

只有名稱上只有延續關係
Domain與Domain之間各自獨立
沒有控制與管理的關係
```
Forest
```
red.com
sub.red.com
blue.com
==========================

Tree跟Tree之間組成Forest
```

# 管理者
Windows權限設計
```
Permission : 針對Object(檔案系統、登入檔)控制

Privilledge(Right) : 針對OS控制(程式執行，可以做什麼不能做什麼)
                     whoami /priv中的State就是你的Privilledge

==> 
所謂權限最大 : 
對於每個Object都可以Full Control
且擁有作業系統的每一個Right
```
Groups群組
```
Windows輸入active Directory Users and Computers
Builtin > Administrators
```
```
Administrators    (DC)
1. 電腦中權限最大的群組
==========================
Domain Admins     (Domain)
1. Domain中權限最大的群組
2. 在Domain Member的電腦上也是權限最大的帳號
3. Domain Admins權力來源是什麼 : 
   在電腦加入到Domain時Domain Admins會成為local administrators的成員
   net localgroup administrators
==========================
Enterprise Admins (Forest)
1. AD邏輯結構上最高權力
2. 必然存在於每個Domain當中DC Administrator的成員
3. 他控制了DC Administrators也就間接控制了Domain Administrators
4. Enterprise Admins只出現在Forest Domain(第一個Domain)
```
Windows群組原則
```
cmd輸入services.msc
找到Group Policy Client 這個就是群組原則
套用群組原則的當下他是開啟的就會寫入
套用群組原則的當下他是關閉的就無效
但不會因為你套用後再關閉這個服務而讓原則失效
```
思考一下:讓Domain Admin完全失效
```
1. 把Domain Admin踢出administrator群組
2. 把群組原則歸0
```
思考一下:DC上的Administrator跟Domain Admin有什麼差別
```
Administrator:就像是秘書長一樣可以決定很多事情，但出了DC就沒有用
Domain Admin :總統，擴及Domain

問題來了DC上的Administrator有沒有權利決定群組原則?
 只要可以控制DC Administrator我就可以控制Domain Admin
 可以想成你的政務要跟總統報就是要經過秘書長
```

# AD
NTDS
```
AD是邏輯存在的一個在DC上的服務，其存在屬於他的憲法
AD的資料庫 : ntds.dit，這個東西被刪掉AD就灰飛煙滅了
分成一個一個PARTITION

Schema         <= 定義整個AD資料結構
-------------
Configuration  <= 整個Forest不含Domain的設定
-------------
<Domain>       <= Domain所有的設定
-------------
<Domain>
-------------
...




Windows輸入ADSI Edit就可以看Partition
```
AD/DS
```
DNS      : 域名系統
Kerberos : 身份驗證協議
LDAP     : 輕量級目錄訪問協議
GC (TCP 3268) : 全域目錄
```

# Penetration
1. [Recon](./Recon.md)
2. [Kerberos](./Kerberos.md)


