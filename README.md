CPENT 
===
> vin_tang@uuu.com.tw

# Exam
1. 教材
   - ASPEN課程序號開通
   - 電子教材(一年期限)
   - 收到Email信箱
     ilabs Training (Access) Key
   - iLab靶機環境(180天)
     https://eccouncil.learnondemand.net/
2. 考試
   - 課程序號開通
   - Exam Dashboard便會開通(一年期限)
   - 預約考試後30天內需完成考試，沒考的連補考都沒有機會
   - 真的出事了，請洽恆逸補考卷
3. 考試時間
   - 需準備護照
   - 只能是平日上班日考
   - 考試選擇，選好不能改
     - 12 + 12 (間隔只要在30天內就可以)
     - 24
   - PenTest Report(7天內需繳交)
   - CPENT:70% 、 LPT:90%
   - 500*(分) * 5(大題) = 2500
   - Launch Exam -> OpenVPN -> Exam Range
   - 考試當天計時畫面需要掛著
   - 只能使用單螢幕考，MAC虛擬桌面也不可以，需要meeting可以錄影錄的到
4. 考試內容(5大題)
   - 選擇題
     - 大約55題選擇題，每大題約8-12題
     - 需要尋找的資訊包含:版本、金鑰、密碼、flag
   - 題型
     - AD (500)
     - Binary exploitation + IOT (500)
     - CTF (500)
     - OT (500)
     - Pivot & Double Pivot (500)
5. 證照延續
   - ECE : US120/3年
   - 年費 : US250/年
     可以開個apsen小號，讓年費維持US80/年

---



# Category
1. Penetration Test
   - [Scan](1.Penetration%20Test/Penetration%20Test%20-%20Scan%20.md)
   - [Enumeration & Service](1.Penetration%20Test/Penetration%20Test%20-%20Enumeration%20&%20Service.md)
   - [Privilege Escalation](1.Penetration%20Test/Penetration%20Test%20-%20Privilege%20Escalation.md)
2. [OT](./2.OT/OT.md)
3. [IOT](./3.IOT/IOT.md)
4. [Pivot & Double Pivot](./4.Pivot%20&%20Double%20Pivot/Pivot%20&%20Double%20Pivot.md)
5. PWN
6. AD Penetration Test

# Outline
- VA(Vulnerability Assessment)
- BAS(Brench and Attack Simulation)
- PT
- BUG Bounty
> hackerone 世界上最大的駭客公司 > Etheral Hacker獎金獵人
> hackerone hacktivity
```
PT ROI 投資報酬率 -> 0

Compliance-oriented Penetration Testing : compliance requirements
Red-team based Penetration Testing : adversarial goal-based assessment

APT41  caret
https://mitre-attack.github.io/caret/#/

Blind Testing
Dounle-blind Testing
```
- 檢測方法論(Open-Source Methodologies)
  - Information Gathering
    - scan : IP、Port、Service
    - Enumaration:設定、狀態、程式版本、漏洞(該開的沒開該觀的沒關)
    - 弱掃工具:拿版本跟CVE去對照
  - Scanning and Reconnaissance
  - Fingerprinting and Enumeration
  - Vulnerability Assessment
  - Exploit Research and Verification
  - Reporting

> AI Security : MITRE ATLAS

> 32bit ROP
> ESC8
> AutoRecon