Recon
===
🔙 [MENU README](./AD.md)

# ADRecon
Enviroment
```bash
https://github.com/sense-of-security/ADRecon
# AD Win : 192.168.177.17
# 有一般AD帳號甚至一般帳號就可以使用ADRecon情蒐
# 可以選擇Other user使用 Local Administrator情蒐
```
```bash
powershell.exe -nop -ep bypass
# -nop：-NoProfile啟動時不讀取任何配置文件
# -ep bypass : -ExecutionPolicy Bypass
# 允許執行未簽名的腳本和任何來源的腳本，指定了 PowerShell的執行策略

# in Domain
.\ADRecon.ps1 -OutputType HTML
# Domain.html
# Groups.html
# Users.html
# 滲透中較常使用

# Not in Domain 知道其中一個網域帳號
./adrecon.ps1 -DomainController 192.168.177.19 -OutputType ALL -Credential lpt.com\cpent
./adrecon.ps1 -DomainController 192.168.177.19 -OutputType HTML -Credential lpt.com\cpent
```
