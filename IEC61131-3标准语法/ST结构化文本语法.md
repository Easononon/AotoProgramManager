# ST（Structured Text）结构化文本 — IEC 61131-3 标准语法

> ST 是 IEC 61131-3 标准定义的高级文本编程语言，类似 Pascal  
> 适用于复杂逻辑、算法、数据处理；所有主流 PLC/DCS 平台均支持  
> 最后更新：2026-05-01

---

## 1. 基础语法规则

### 1.1 注释
```st
// 单行注释
(* 多行注释 *)
```

### 1.2 变量声明
```st
VAR
    MyInt    : INT := 10;        // 整型变量
    MyReal   : REAL;              // 实数
    MyBool   : BOOL;              // 布尔
    MyString : STRING(80);        // 字符串
    MyArr    : ARRAY[1..10] OF INT;  // 数组
    MyTime   : TIME;              // 时间类型
END_VAR
```

### 1.3 常量
```st
VAR CONSTANT
    MAX_COUNT : INT := 100;
    PI        : REAL := 3.14159;
END_VAR
```

### 1.4 类型别名
```st
TYPE
    MotorSpeed : INT (0..10000);  // 带范围约束
    PressData  : STRUCT
        value : REAL;
        unit  : STRING(10);
    END_STRUCT
END_TYPE
```

---

## 2. 运算符

| 类别 | 运算符 | 说明 |
|------|--------|------|
| 算术 | `+` `-` `*` `/` `MOD` `**` | 加减乘除，取模，幂 |
| 比较 | `=` `<>` `>` `<` `>=` `<=` | 等于，不等，大于，小于 |
| 逻辑 | `AND` `OR` `XOR` `NOT` | 与，或，异或，非 |
| 赋值 | `:=` | 赋值 |

> ⚠️ 注意：不同品牌对逻辑运算符有不同写法（`&` `&&` `.AND.`）

---

## 3. 控制语句

### 3.1 IF 语句
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

### 3.2 CASE 语句
```st
CASE state OF
    0:  // 空闲
        output := 0;
    1:  // 运行
        output := 100;
    2:  // 停止
        output := 0;
    ELSE
        output := -1;  // 未知状态
END_CASE;
```

### 3.3 FOR 循环
```st
FOR i := 1 TO 10 DO
    sum := sum + data[i];
END_FOR;

FOR i := 10 TO 1 BY -1 DO  // 逆向循环
    arr[i] := arr[i] * 2;
END_FOR;
```

### 3.4 WHILE 循环
```st
WHILE count < 100 DO
    count := count + 1;
    IF count > 50 THEN
        EXIT;  // 退出循环
    END_IF;
END_WHILE;
```

### 3.5 REPEAT 循环
```st
REPEAT
    sum := sum + 1;
UNTIL sum > 100
END_REPEAT;
```

---

## 4. 标准功能

### 4.1 定时器（IEC 标准）
| 功能块 | 说明 |
|--------|------|
| `TON` | 延时闭合（ON Delay）|
| `TOF` | 延时断开（OFF Delay）|
| `TP` | 脉冲（Pulse）|

```st
Timer1(IN := startSignal, PT := T#5s);
IF Timer1.Q THEN
    output := TRUE;
END_IF;
```

### 4.2 计数器
| 功能块 | 说明 |
|--------|------|
| `CTU` | 递增计数（Count Up）|
| `CTD` | 递减计数（Count Down）|
| `CTUD` | 双向计数 |

```st
Counter1(CU := countPulse, PV := 10);
IF Counter1.Q THEN
    alarm := TRUE;
END_IF;
```

### 4.3 边沿检测
| 功能块 | 说明 |
|--------|------|
| `R_TRIG` | 上升沿触发 |
| `F_TRIG` | 下降沿触发 |

```st
RisingEdge(CLK := inputSignal);
IF RisingEdge.Q THEN
    counter := counter + 1;
END_IF;
```

---

## 5. 功能块（Function Block）调用

### 5.1 声明
```st
VAR
    MyTimer : TON;       // TON 实例
    MyPID   : PID;       // PID 实例
END_VAR
```

### 5.2 调用
```st
MyTimer(IN := startSignal, PT := T#2s);
outputValue := MyTimer.Q;

MyPID(AutoMode := TRUE, SP := 100.0, PV := currentValue);
```

---

## 6. 功能（Function）调用

```st
// 内置函数
absValue := ABS(-10);       // 绝对值
sqrtVal  := SQRT(16.0);     // 平方根
maxVal   := MAX(a, b);     // 最大值
minVal   := MIN(a, b);     // 最小值
clamped  := LIMIT(0, value, 100);  // 限幅

// 自定义函数
result := MyFunction(input1, input2);
```

---

## 7. 字符串操作

```st
CONCAT(str1, str2);          // 字符串连接
LEFT(str, 5);                // 取左边5字符
RIGHT(str, 3);              // 取右边3字符
MID(str, 2, 5);             // 取中间（起始位置2，长度5）
LEN(str);                    // 字符串长度
INSERT(str1, str2, 3);       // 在位置3插入str2
DELETE(str, 2, 4);           // 删除从位置2开始的4字符
REPLACE(str1, str2, 3);      // 用str2替换str1从位置3开始
FIND(str1, str2);            // 查找str2在str1中的位置
```

---

## 8. 日期和时间

```st
timeNow  := CURRENT_TIME();    // 当前时间
dateNow  := CURRENT_DATE();    // 当前日期

// 时间常量表示
T#10s      // 10秒
T#1m30s    // 1分30秒
T#2h15m    // 2小时15分
TOD#12:30:00    // 当天时间 12:30:00
DATE#2026-05-01 // 日期
DT#2026-05-01-12:00:00 // 日期时间
```

---

## 9. 常见语法错误避免

| 问题 | 正确写法 |
|------|----------|
| 赋值用 `==` | 用 `:=` |
| 整数除法丢失精度 | 用 `REAL_TO_INT(x) / REAL_TO_INT(y)` |
| 数组越界 | 确保索引在声明范围内 |
| 未初始化变量 | 始终赋初值 |
| 逻辑运算符混用 | 统一用 `AND`/`OR`，不用 `&&`/`||` |

---

## 10. 代码结构示例

```st
PROGRAM MainProgram
VAR
    counter    : INT := 0;
    threshold : INT := 50;
    running    : BOOL := FALSE;
    alarm      : BOOL := FALSE;
    myTimer    : TON;
END_VAR

// 主逻辑
IF startButton AND NOT running THEN
    running := TRUE;
    myTimer(IN := FALSE);  // 复位定时器
END_IF;

IF running THEN
    myTimer(IN := TRUE, PT := T#5s);
    IF myTimer.Q THEN
        counter := counter + 1;
        alarm := (counter >= threshold);
        running := FALSE;
    END_IF;
END_IF;

output := counter;
END_PROGRAM
```

---

## 📝 各品牌 ST 语法差异（详情见"品牌差异对照表.md"）

| 差异点 | 西门子 | 施耐德 | 浙中控 | 横河 | 和利时 |
|--------|--------|--------|--------|------|--------|
| 关键字大小写 | 不敏感 | 不敏感 | 不敏感 | 敏感？ | 不敏感 |
| 变量声明关键字 | `VAR` | `VAR` | `VAR` | `VAR` | `VAR` |
| 定时器语法 | `TON` | `TON` | `TON` | `TON` | `TON` |
| 注释方式 | `//` `(*)` | `//` `(*)` | `//` `(*)` | `//` `(*)` | `//` `(*)` |
| 字符串连接 | `CONCAT()` | `CONCAT()` | `+` 操作符 | 不同 | 不同 |