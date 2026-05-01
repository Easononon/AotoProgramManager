# 三台泵恒压供水 - 和利时LK版本（PowerPro）

```st
// ============================================================
// 三台泵恒压供水
// 和利时LK大型PLC / PowerPro编程软件
// ============================================================

// ---- 全局变量 GV ----
VAR_GLOBAL
    Setpoint    : REAL := 0.5;     // 设定压力 MPa
    Actual      : REAL := 0.5;     // 实际压力 %IW0
    Hysteresis  : REAL := 0.03;   // 滞环 MPa
END_VAR

PROGRAM PUMP_AUTO
VAR
    P1_ON, P1_OFF : REAL;
    P2_ON, P2_OFF : REAL;
    P3_ON, P3_OFF : REAL;
    
    Pump1Run : BOOL := FALSE;
    Pump2Run : BOOL := FALSE;
    Pump3Run : BOOL := FALSE;
    
    Pump1Auto : BOOL := TRUE;
    Pump2Auto : BOOL := TRUE;
    Pump3Auto : BOOL := TRUE;
    
    tmP1Start, tmP2Start, tmP3Start : TON;
    tmP1Stop, tmP2Stop, tmP3Stop : TON;
    tmHourCnt : TON;
END_VAR

    // ---- 计算阈值 ----
    P1_ON  := Setpoint - Hysteresis;
    P1_OFF := Setpoint + Hysteresis;
    P2_ON  := Setpoint - 2.0*Hysteresis;
    P2_OFF := Setpoint + 2.0*Hysteresis;
    P3_ON  := Setpoint - 3.0*Hysteresis;
    P3_OFF := Setpoint + 3.0*Hysteresis;
    
    // ---- 1#泵 ----
    IF Actual <= P1_ON AND Pump1Auto THEN
        tmP1Start(IN := TRUE, PT := T#30s);
        IF tmP1Start.Q THEN
            Pump1Run := TRUE;
        END_IF;
    ELSIF Actual >= P1_OFF AND Pump1Run THEN
        tmP1Stop(IN := TRUE, PT := T#5m);
        IF tmP1Stop.Q THEN
            Pump1Run := FALSE;
        END_IF;
    END_IF;
    
    // ---- 2#泵 ----
    IF Actual <= P2_ON AND Pump2Auto THEN
        tmP2Start(IN := TRUE, PT := T#30s);
        IF tmP2Start.Q THEN
            Pump2Run := TRUE;
        END_IF;
    ELSIF Actual >= P2_OFF AND Pump2Run THEN
        tmP2Stop(IN := TRUE, PT := T#5m);
        IF tmP2Stop.Q THEN
            Pump2Run := FALSE;
        END_IF;
    END_IF;
    
    // ---- 3#泵 ----
    IF Actual <= P3_ON AND Pump3Auto THEN
        tmP3Start(IN := TRUE, PT := T#30s);
        IF tmP3Start.Q THEN
            Pump3Run := TRUE;
        END_IF;
    ELSIF Actual >= P3_OFF AND Pump3Run THEN
        tmP3Stop(IN := TRUE, PT := T#5m);
        IF tmP3Stop.Q THEN
            Pump3Run := FALSE;
        END_IF;
    END_IF;
    
    // ---- 输出映射 ----
    %QX0.0 := Pump1Run;  // 1#泵
    %QX0.1 := Pump2Run;  // 2#泵
    %QX0.2 := Pump3Run;  // 3#泵
    %QX0.3 := Pump1Run OR Pump2Run OR Pump3Run;  // 变频器使能
    
END_PROGRAM
```

---

## IO定义（和利时LK）

| 地址 | 方向 | 说明 |
|------|------|------|
| %IW0 | AI | 压力传感器（4-20mA / 0-10V）|
| %QX0.0 | DO | 1#泵接触器 |
| %QX0.1 | DO | 2#泵接触器 |
| %QX0.2 | DO | 3#泵接触器 |
| %QX0.3 | DO | 变频器使能 |
| %IX0.0 | DI | 1#泵手/自动 |
| %IX0.1 | DI | 2#泵手/自动 |
| %IX0.2 | DI | 3#泵手/自动 |
| %IX0.3 | DI | 1#泵故障 |
| %IX0.4 | DI | 2#泵故障 |
| %IX0.5 | DI | 3#泵故障 |
