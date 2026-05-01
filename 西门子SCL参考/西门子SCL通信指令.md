---
title: 西门子SCL通信指令
category: concept
sources: [西门子S7-12001500 PLC SCL语言编程从入门到精通pdf.pdf]
source_count: 1
keywords: [通信, Modbus, S7通信, TCP, 串口, PUT, GET]
subject_terms: [通信协议, S7, Modbus-RTU, TCP通信]
aliases: [通信, S7通信, Modbus, TCP]
tags: [通信, 西门子, 协议]
---

# 西门子SCL通信指令

## 1. S7单边通信（GET/PUT）

### 服务器端设置

在CPU属性 **"防护与安全"→"连接机制"** 中勾选 **"允许来自远程对象的PUT/GET通信访问"**

⚠️ 优化的数据块不支持PUT/GET指令访问

### GET指令 - 读取远程数据

```st
GET(REQ := bTrigger,           // 上升沿触发
    ID := 16#100,              // 连接ID（硬件组态中的本地ID）
    NDR => bNDR,               // 新数据就绪
    ERROR => bError,
    STATUS => wStatus,
    ADDR_1 := P#DB100.DBB10.0 BYTE 10,  // 远程地址
    RD_1 := P#DB500.DBB0.0 BYTE 10);   // 本地存放地址
```

### PUT指令 - 写入远程数据

```st
PUT(REQ := bTrigger,
    ID := 16#100,
    DONE => bDone,
    ERROR => bError,
    STATUS => wStatus,
    ADDR_1 := P#DB100.DBB20.0 BYTE 10,  // 远程写入地址
    SD_1 := P#DB500.DBB10.0 BYTE 10);   // 本地数据源
```

---

## 2. Modbus-RTU通信

### 硬件配置

- CM 1241 RS232/RS422-485 通信模块
- CB 1241 RS485 通信板
- S7-1200 CPU固件 ≥ V4.1
- CM固件 ≥ V2.1

### Modbus_Comm_Load - 通信参数配置

```st
Modbus_Comm_Load(REQ := bFirstScan,    // 仅第1扫描周期执行
                PORT := 2,             // 端口硬件标识符
                BAUD := 9600,          // 波特率
                PARITY := 0,            // 0=无校验
                FLOW_CTRL := 0,
                RESP_TO := 1000,        // 主站等待时间ms
                MB_DB := "ModbusMaster".MB_DB);
```

### Modbus_Master - 主站读取

```st
Modbus_Master(REQ := bRequest,        // 上升沿触发
               MB_ADDR := 1,           // 从站地址
               MODE := 0,              // 0=读取
               DATA_ADDR := 40001,     // Modbus地址（保持寄存器）
               DATA_LEN := 1,          // 读取1个字
               DATA_PTR := "DB101".temperature,  // 存放地址
               DONE => bDone,
               ERROR => bError,
               STATUS => wStatus);
```

### Modbus_Slave - 从站

```st
Modbus_Slave(REQ := TRUE,
             MB_ADDR := 10,           // 本从站地址
             MB_HOLD_REG := "DB102_MB_HoldRegister".data,  // 保持寄存器
             DONE => bDone,
             ERROR => bError);
```

### Modbus地址模型

| 地址类型 | 地址范围 | 数据大小 |
|---------|---------|---------|
| 线圈 | 00001~09999 | 位 |
| 离散量输入 | 10001~19999 | 位 |
| 输入寄存器 | 30001~39999 | 字 |
| 保持寄存器 | 40001~49999 | 字 |

---

## 3. 点对点串口通信

### 发送 (Send_P2P)

```st
Send_P2P(REQ := bSendReq,      // 上升沿触发
         PORT := 2,
         DATA := "DB101_Global".sendData,
         LENGTH := 10,
         DONE => bSendDone,
         ERROR => bSendError);
```

### 接收 (Receive_P2P)

```st
Receive_P2P(EN_R := TRUE,
             PORT := 2,
             DATA := "DB101_Global".receiveData,
             LENGTH := 20,
             NDR => bNewData,
             ERROR => bRcvError,
             STATUS => wStatus);
```

---

## 4. TCP通信

### 建立连接 (TCON)

```st
TCON(ID := 1,                  // 连接ID
     CONNECT := "ConnDB".conn,  // 连接参数数据块
     DONE => bConnDone,
     BUSY => bConnBusy,
     ERROR => bConnError);
```

### 发送 (TSEND)

```st
TSEND(REQ := bSendReq,
      ID := 1,
      LEN := 20,
      DATA := "DB200".sendBuffer,
      DONE => bSendDone,
      ERROR => bSendError);
```

### 接收 (TRCV)

```st
TRCV(EN_R := TRUE,
     ID := 1,
     LEN := 20,
     DATA := "DB200".recvBuffer,
     NDR => bNewData,
     ERROR => bRcvError);
```

### 断开连接 (TDISCON)

```st
TDISCON(ID := 1, DONE => bDisconDone, ERROR => bDisconError);
```

---

## 5. 串口参数设置

### 波特率/校验/数据位/停止位

```st
Port_Config(REQ := bCfgReq,
            PORT := 2,
            BAUD := 115200,
            PARITY := 0,       // 0=无,1=奇,2=偶
            DATA_BITS := 8,
            STOP_BITS := 1,
            FLOW_CTRL := 0);
```

---

## 相关页面

- [[西门子SCL核心语法]]
- [[西门子SCL运动控制指令]]
- [[PLC知识库]]
