---
title: 西门子SCL核心语法
category: concept
sources: [西门子S7-12001500 PLC SCL语言编程从入门到精通pdf.pdf]
source_count: 1
keywords: [西门子, SCL, TIA Portal, S7-1200, S7-1500, 定时器, 计数器, PEEK, POKE]
subject_terms: [SCL语法, 指令参考, PLC编程]
aliases: [西门子SCL, SCL核心, TIA SCL]
tags: [西门子, SCL, 语法参考, 指令]
---

# 西门子SCL核心语法（基于《西门子S7-1200/1500 PLC SCL语言编程从入门到精通》）

## 1. 变量声明

### 1.1 基本类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `BOOL` | 布尔型（1位）| `VAR bFlag : BOOL := FALSE; END_VAR` |
| `INT` | 整型（16位）| `VAR nTemp : INT := 0; END_VAR` |
| `DINT` | 双整数（32位）| `VAR dwCount : DINT; END_VAR` |
| `REAL` | 实数（32位浮点）| `VAR fValue : REAL := 0.0; END_VAR` |
| `LREAL` | 双精度实数 | `VAR dfPi : LREAL := 3.14159; END_VAR` |
| `BYTE` | 字节（8位）| `VAR byData : BYTE; END_VAR` |
| `WORD` | 字（16位）| `VAR wAddr : WORD; END_VAR` |
| `DWORD` | 双字（32位）| `VAR dwVal : DWORD; END_VAR` |
| `STRING(n)` | 字符串，长度n | `VAR sName : STRING(80); END_VAR` |
| `TIME` | 时间类型 | `VAR tDelay : TIME := T#5s; END_VAR` |
| `DATE_AND_TIME` | 日期时间 | `VAR dtNow : DATE_AND_TIME; END_VAR` |

### 1.2 变量声明关键字

| 关键字 | 说明 | 作用域 |
|--------|------|--------|
| `VAR` | 普通变量 | 程序组织单元内部 |
| `VAR_TEMP` | 临时变量 | 仅在程序执行期间存在 |
| `VAR_INPUT` | 输入参数 | 从外部传入 |
| `VAR_OUTPUT` | 输出参数 | 传递给外部 |
| `VAR_IN_OUT` | 输入/输出参数 | 既传入又传出 |
| `VAR_GLOBAL` | 全局变量 | 整个程序 |
| `VAR_CONSTANT` | 常量 | 全局常量 |

### 1.3 数组

```st
VAR
    arData : ARRAY[1..10] OF INT;        // 一维数组
    ar2D   : ARRAY[1..3, 1..4] OF REAL;  // 二维数组
END_VAR

// 访问
arData[1] := 100;
ar2D[2, 3] := 3.14;
```

### 1.4 结构体

```st
TYPE MotorData
    STRUCT
        speed  : REAL;
        status : BOOL;
        id     : INT;
    END_STRUCT
END_TYPE

VAR
    m1 : MotorData;
END_VAR

m1.speed := 1500.0;
m1.status := TRUE;
```

---

## 2. 运算符与优先级

| 优先级 | 运算符 | 说明 |
|--------|--------|------|
| 高 | `NOT`, `**` | 逻辑非，幂 |
| ↑ | `*`, `/`, `MOD` | 乘除，取模 |
| | `+`, `-` | 加减 |
| | `>`, `<`, `>=`, `<=` | 比较 |
| | `=`, `<>` | 等于，不等于 |
| | `AND` | 逻辑与 |
| 低 | `OR`, `XOR` | 逻辑或，异或 |

---

## 3. 控制语句

### 3.1 IF语句

```st
IF temperature > 100 THEN
    alarm := TRUE;
    heating := FALSE;
ELSIF temperature < 50 THEN
    heating := TRUE;
ELSE
    alarm := FALSE;
END_IF;
```

### 3.2 CASE语句

```st
CASE state OF
    0: output := 0;     // 空闲状态
    1: output := 100;   // 运行状态
    2: output := 50;    // 暂停状态
    ELSE output := -1;   // 未知状态
END_CASE;
```

### 3.3 FOR循环

```st
FOR i := 1 TO 10 DO
    sum := sum + data[i];
END_FOR;

// 逆序循环
FOR i := 10 TO 1 BY -1 DO
    data[i] := data[i] * 2;
END_FOR;
```

### 3.4 WHILE循环

```st
WHILE count < 100 DO
    count := count + 1;
    IF count > 50 THEN
        EXIT;  // 退出循环
    END_IF;
END_WHILE;
```

### 3.5 赋值语句

```st
#result := #a + #b;           // 正常赋值
#byteVar.%X7 := TRUE;          // 片段访问（第7位）
#byteVar.%B0 := 16#FF;         // 字节访问
#dwordVar.%W0 := 16#1234;      // 字访问
```

### 3.6 AT覆盖访问（间接寻址）

```st
// 需要取消函数/函数块的优化属性
VAR
    statTestByte : Byte;
    statAtTestByte AT statTestByte : ARRAY[0..7] OF BOOL;
END_VAR

// 通过数组访问字节的某一位
statAtTestByte[0] := TRUE;  // 将statTestByte的第0位置1
```

---

## 4. 定时器指令

### 4.1 脉冲定时器 (TP)

```st
PulseTimer(IN := startSignal, PT := T#5s, Q => pulseOut, ET => elapsed);

IF pulseOut THEN
    output := TRUE;
END_IF;
```

### 4.2 延时接通定时器 (TON)

```st
DelayTon(IN := startSignal, PT := T#3s, Q => output, ET => timerET);
```

### 4.3 延时断开定时器 (TOF)

```st
DelayTof(IN := stopSignal, PT := T#2s, Q => output, ET => timerET);
```

---

## 5. 计数器指令

| 指令 | 说明 |
|------|------|
| `CTU` | 加计数（Count Up） |
| `CTD` | 减计数（Count Down） |
| `CTUD` | 加/减计数 |

```st
Counter1(CTU := countUpSignal, PT := 10, Q => overflow, CV => currentValue);
Counter2(CTD := countDnSignal, PT := 10, Q => zero, CV => currentValue);
```

---

## 6. 数学指令

```st
// 三角函数
result := SIN(angle);      // 正弦
result := COS(angle);      // 余弦
result := TAN(angle);      // 正切
result := SQRT(val);       // 平方根

// 统计函数
maxVal := MAX(a, b, c);   // 最大值
minVal := MIN(a, b, c);   // 最小值

// 类型转换
intVal := DINT_TO_INT(realVal);
realVal := INT_TO_REAL(intVal);
```

---

## 7. 读写存储器指令（间接寻址）

### 7.1 PEEK指令 - 读取

| area 值 | 存储区 |
|---------|--------|
| 16#81 | 输入映像区（I）|
| 16#82 | 输出映像区（Q）|
| 16#83 | 位存储区（M）|
| 16#84 | 数据块（DB）|
| 16#1 | 外设输入（仅S7-1500）|

```st
// 读取字节
tmpByte := PEEK(area:=16#84, dbNumber:=100, byteOffset:=10);

// 读取字
tmpWord := PEEK_WORD(area:=16#83, byteOffset:=100);

// 读取双字
tmpDword := PEEK_DWORD(area:=16#83, byteOffset:=90);

// 读取位
tmpBool := PEEK_BOOL(area:=16#83, byteOffset:=100, bitOffset:=7);
```

### 7.2 POKE指令 - 写入

```st
// 写入字节
POKE(area:=16#82, byteOffset:=10, value:=16#AB);

// 写入字
POKE(area:=16#82, byteOffset:=12, value:=3200);

// 写入位
POKE_BOOL(area:=16#82, byteOffset:=10, bitOffset:=5, value:=TRUE);
```

---

## 8. 模拟量处理指令

### 8.1 归一化 (NORM_X)

```st
// 将模拟量输入值(0~27648)映射到0~1之间
normalized := NORM_X(MIN:=0, IN:=analogInput, MAX:=27648);
```

### 8.2 标定 (SCALE_X)

```st
// 将0~1之间的值映射到工程值范围(-20~80℃)
scaledValue := SCALE_X(MIN:=-20.0, IN:=normalized, MAX:=80.0);
```

---

## 9. PWM脉宽调制

```st
// PWM 1硬件配置：脉冲周期1000μs，脉宽格式"千分之一"
CTRL_PWM(PWM:=1, ENABLE:=bEnable, OUTPUT_VALUE:=dutyCycle);
```

---

## 10. 常见错误与注意

1. **除法精度丢失**：整数除法前先转REAL
2. **数组越界**：索引从1开始（不是0）
3. **变量未初始化**：静态变量需赋初值
4. **OB1不支持SCL**：只能在FC/FB中使用SCL
5. **数据块优化属性**：PEEK/POKE/GET/PUT无法访问优化的数据块

## 相关页面

- [[IEC61131-3标准]]
- [[PLC知识库]]
- [[品牌差异对照表]]
