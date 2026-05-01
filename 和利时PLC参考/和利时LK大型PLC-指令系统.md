---
title: 和利时LK大型PLC指令系统
category: concept
sources: [LK大型可编程控制器指令手册.pdf]
source_count: 1
keywords: [和利时, LK, PowerPro, PLC, 指令系统, 标准库]
subject_terms: [和利时, PLC编程, 指令参考]
aliases: [和利时LK, LK-PLC, HollySys, Hollysys]
tags: [和利时, LK, PLC, 指令系统]
---

# 和利时LK大型PLC指令系统

## 概述

LK大型PLC是和利时集团推出的中高性能控制领域产品，编程软件为 **PowerPro**，支持 **ST/LD/FBD/SFC/CFC** 五种编程语言。

## 指令库结构

| 库名 | 类型 | 说明 |
|------|------|------|
| **Standard.lib** | 外部库 | IEC标准指令，随工程自动加载 |
| **Util.lib** | 内部开放库 | 模拟量/PID/高等数学运算 |
| **SysLibCallback.lib** | 外部库 | 系统回调指令（事件调用） |
| **HS_Check.lib** | 内部开放库 | 被除数零检查/边界检查 |
| **Iecsfc.lib** | 内部开放库 | SFC步动作控制 |
| **HS_AnalogConvert.lib** | 扩展 | 模拟量输入转工程量/工程量转输出 |
| **HS_Move.lib** | 扩展 | 数据传送 |
| **HS_Communication.lib** | 扩展 | 通讯指令 |
| **HS_SOE.lib** | 扩展 | SOE事件记录 |
| **HS_ScheduledTime.lib** | 扩展 | 预置输出定时 |
| **HS_RTC.lib** | 扩展 | 实时时钟 |
| **HS_Diagnosis.lib** | 扩展 | 诊断 |
| **HS_SetIPAddress.lib** | 扩展 | 修改IP地址 |
| **HS_PIDController.lib** | 扩展 | PID调节控制器 |
| **HS_LK850_Convert.lib** | 扩展 | LK850量程转换 |
| **HS_Timer.lib** | 扩展 | 保持型定时器 |

## Standard.lib 指令一览

### 定时器（IEC标准）

| 指令 | 说明 |
|------|------|
| `TON` | 通电延时定时器（Timer On Delay） |
| `TOF` | 断电延时定时器（Timer Off Delay） |
| `TP` | 脉冲定时器（Timer Pulse） |

```st
// LK ST语言定时器调用（与IEC61131-3一致）
Timer1 TON;
Timer1(IN := startSignal, PT := T#5s);
IF Timer1.Q THEN
    output := TRUE;
END_IF;

// 延时定时器
DelayTon(IN := startSignal, PT := T#3s, Q => output, ET => elapsed);
```

### 计数器

| 指令 | 说明 |
|------|------|
| `CTU` | 递增计数（Count Up） |
| `CTD` | 递减计数（Count Down） |
| `CTUD` | 递增/递减计数 |

```st
Counter1 CTU;
Counter1(CU := countUpSignal, PV := 10, Q => overflow, CV => currentValue);

Counter2 CTD;
Counter2(CD := countDnSignal, PV := 10, Q => zero, CV => currentValue);
```

### 边沿检测

| 指令 | 说明 |
|------|------|
| `R_TRIG` | 上升沿检测 |
| `F_TRIG` | 下降沿检测 |

```st
RisingEdge R_TRIG;
RisingEdge(CLK := inputSignal);
IF RisingEdge.Q THEN
    counter := counter + 1;
END_IF;
```

### 双稳态

| 指令 | 说明 |
|------|------|
| `RS` | 复位优先双稳态 |
| `SR` | 置位优先双稳态 |

### 字符串指令

`CONCAT` / `DELETE` / `FIND` / `INSERT` / `LEFT` / `LEN` / `MID` / `REPLACE` / `RIGHT` 用法与IEC61131-3标准一致。

### RTC 实时时钟

```st
// 读取/设置实时时钟
```

---

## Util.lib 应用指令库

### 模拟量报警

```st
// 上下限报警
Alarm1 LIMITALARM(IN := sensorValue, LIM_H := 100.0, LIM_L := 0.0, HYST := 0.5);

// 滞后报警
Hyst1 HYSTERESIS(IN := value, DV := 0.5);
```

### BCD码转换

```st
// BCD转整型
intVal := BCD_TO_INT(bcdWord);

// 整型转BCD
bcdWord := INT_TO_BCD(intVal);
```

### PID控制器

```st
// 比例控制器
P_Controller(ENABLE := TRUE, SP := 100.0, PV := currentValue, OUT => controlOutput);

// PID控制器
PID_Controller(ENABLE := TRUE, SP := setpoint, PV := processValue, OUT => output);
```

---

## 数据类型

| 类型 | 说明 | 范围/长度 |
|------|------|-----------|
| `BOOL` | 布尔 | TRUE/FALSE |
| `SINT` | 单整型 | -128~127 |
| `USINT` | 无符号单整型 | 0~255 |
| `INT` | 整型 | -32768~32767 |
| `UINT` | 无符号整型 | 0~65535 |
| `DINT` | 双整型 | ±2147483648 |
| `UDINT` | 无符号双整型 | 0~4294967295 |
| `REAL` | 实数 | ±3.4e38 |
| `TIME` | 时间 | T#0s ~ T#49d17h2m47s |
| `DATE` | 日期 | D#1970-01-01~ |
| `TOD` | 当日时间 | TOD#00:00:00~ |
| `DT` | 日期时间 | DT#1970-01-01-00:00:00~ |
| `STRING(n)` | 字符串 | 最大n字符 |

---

## 变量地址表示法

| 格式 | 说明 |
|------|------|
| `%IX0.0` | 输入位（I=输入，X=位）|
| `%QX0.0` | 输出位（Q=输出）|
| `%MB0` | 位存储区字节 |
| `%MW0` | 位存储区字 |
| `%MD0` | 位存储区双字 |
| `%IW0` | 输入区字 |
| `%QW0` | 输出区字 |

---

## 操作数语法

### 常量

```st
// 布尔常量
TRUE / FALSE / 1 / 0

// 时钟常量
T#18ms         // 18毫秒
T#100s12ms     // 100秒12毫秒
t#12h34m15s    // 12小时34分15秒

// 数字常量
14             // 十进制
2#1001_0011    // 二进制
8#67            // 八进制
16#AE           // 十六进制

// 实数常量
7.4
1.64e+009

// 字符串常量
'Abby and Craig'
```

### 变量访问

```st
// 数组访问
<字段名>[Index1, Index2]

// 结构访问
<结构名>.<变量名>

// 功能块变量
<实例名>.<变量名>

// 位访问（从0开始）
a.2 := b;   // 将变量a的第2位置b的值
```

---

## 类型转换（240个转换指令）

语法：`<TYPE1>_TO_<TYPE2>`

```st
// 布尔转其他类型
intVal := BOOL_TO_INT(TRUE);    // 结果=1
timeVal := BOOL_TO_TIME(TRUE);   // 结果=T#1ms

// 整型转实型
realVal := INT_TO_REAL(intVal);  // 整型→实数

// 实型转整型（四舍五入）
intVal := REAL_TO_INT(1.5);      // 结果=2

// 截断（去小数）
intVal := TRUNC(1.9);           // 结果=1

// 时间/日期转换
dtVal := DINT_TO_DT(seconds);   // DINT转日期时间
timeVal := DINT_TO_TIME(ms);     // 毫秒转时间
```

---

## 数学运算指令

```st
// 绝对值
result := ABS(-2);              // 结果=2

// 平方根
result := SQRT(10);             // 结果=3.162278

// 对数
result := LN(45);              // 自然对数
result := LOG(314.5);          // 常用对数

// 指数
result := EXP(2);              // 结果=e^2=7.389056

// 三角函数（弧度）
result := SIN(1.5);
result := COS(0.5);
result := TAN(0.5);

// 反三角函数
result := ASIN(0.5);           // 弧度
result := ACOS(0.5);
result := ATAN(0.5);

// 幂
result := EXPT(7, 2);         // 7^2=49
```

---

## 移位指令

```st
// 左移
result := SHL(16#45, 2);       // 左移2位

// 右移
result := SHR(16#45, 2);       // 右移2位

// 循环左移
result := ROL(16#45, 2);       // 循环左移

// 循环右移
result := ROR(16#45, 2);       // 循环右移
```

---

## 选择指令

```st
// 二选一
result := SEL(G := TRUE, IN0 := 3, IN1 := 4);   // G=TRUE选IN1=4

// 最大值
result := MAX(90, 60);         // 结果=90

// 最小值
result := MIN(90, 30);         // 结果=30

// 限幅
result := LIMIT(30, 90, 80);  // 限制在30~80之间，结果=80

// 多选一
result := MUX(0, 30, 40, 50, 60);  // K=0选30
```

---

## 比较指令

| 指令 | 说明 |
|------|------|
| `GT` | 大于 |
| `GE` | 大于等于 |
| `LT` | 小于 |
| `LE` | 小于等于 |
| `EQ` | 等于 |
| `NE` | 不等于 |

```st
result := 20 > 30;             // FALSE
result := 60 >= 40;            // TRUE
result := 40 <> 40;            // FALSE
```

---

## 与西门子SCL的差异

| 特性 | 和利时LK | 西门子SCL |
|------|---------|-----------|
| 变量声明 | `VAR ... END_VAR` | `VAR ... END_VAR` |
| 定时器语法 | `TON(Timer1)` 或函数调用 | 功能块实例调用 |
| 类型转换 | `INT_TO_REAL()` | `INT_TO_REAL()` |
| 字符串连接 | `CONCAT(s1,s2)` | `CONCAT(s1,s2)` |
| 位访问 | `var.2` | `#var.%X2` |
| 数学函数 | 基本一致 | 基本一致 |

## 相关页面

- [[IEC61131-3标准]]
- [[品牌差异对照表]]
- [[PLC知识库]]
