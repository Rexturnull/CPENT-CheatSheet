OT
===
🔙 [MENU README](../README.md#note)


# OT
```
SCADA <--Modbus--> Controller  <====> 現場
(量、測、調、控)    (RTU、DCS、PLC)    (機台、設備、設施)

量(Read)調(Write) : 類比訊號
測控              : 非類比訊號
```

# tcpdump
```bash
# 在 Modbus 通信中，通常是使用 TCP 協定的 502 Port
tcpdump tcp port 502 -vv -w modbus.cap
tcpdump -i ens3 port 502 -A -w OT_Dump.pcap
```



# Analyze
Setting
```
1. Transmission Control Protocol
2. "Right Click"
3. Protocal Preference
4. "Cancel"  Aynalze TCP sequence numbers
```
網卡型號和MAC地址
```
1. View
2. Name Resolution
3. "Enable" Resolve Network Address
```
Statstic
```
1. Endpoints : 網卡、IP數量
2. Conversation
3. Protocol Hierachy
```
Transaction Idntifier Filiter
```bash
mbtcp.trans_id=247
```

# Modbus
Modbus/TCP
```
1. Transaction Identifier：佔2個Byte，用於標示通訊的識別碼
2. Protocol Identifier   ：佔2個Byte，用於表示 Modbus 協定的版本
3. Length                ：佔2個Byte，表示後續資料的長度
4. Unit Identifier       ：佔1個Byte，用於識別 Modbus 設備或單元
```
Modbus
```
1. Function Code：佔1個Byte，表示 Modbus 操作的類型。
                  功能碼用於區分讀數據、寫數據、診斷等不同的 Modbus 操作
                  function多少出去就多少回來，如果不一致基本上就是error code
2. Data Field   ：(Click Function Code)這個字段的長度可變，根據Modbus操作的不同而異
                  它包含了具體的數據，例如讀取或寫入的數據
```


# ModbusAnalog
> Try to Analyze modbusAnalog
> And Try to answaer question in ModbusAnalogAns.txt
