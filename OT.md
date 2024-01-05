OT
===

```
SCADA <--Modbus--> Controller <====> 現場
(量、測、調、控)    (RTU、DCS、PLC)    (機台、設備、設施)

量(Read)調(Write) : 類比訊號
測控 : 非類比訊號
```
```
TCP層 > 右鍵 > Protocal Preference > Aynalze TCP sequence numbers 關掉

Statstics > Endpoints

502 Controller

Statstics > Conversation

Statstics > Protocol Hierachy

View > Name Resolution > Resolve Network Address


#Transaction Idntifier
mbtcp.trans_id=247


# Modbus function code
# function多少出去 就多少回來
# 如果不一致基本上就是 error code
```