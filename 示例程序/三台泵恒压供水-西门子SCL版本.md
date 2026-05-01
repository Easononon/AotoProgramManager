# 三台泵恒压供水 - 西门子SCL版本（TIA Portal）

```st
// ============================================================
// 三台泵恒压供水 - 恒压供水
// 西门子S7-1200/1500 TIA Portal SCL
// 适用：S7-1200 / S7-1500
// ============================================================

// ---- 全局数据块 DB1 ----
DATA_BLOCK DB1_PumpControl
    // 模拟量
    Setpoint    : REAL := 0.5;    // 设定压力 MPa
    Actual      : REAL := 0.0;     // 实际压力 %IW64（AI模块）
    Hysteresis  : REAL := 0.03;   // 滞环 MPa
    
    // 阈值计算（自动）
    P1_ON  : REAL;
    P1_OFF : REAL;
    P2_ON  : REAL;
    P2_OFF : REAL;
    P3_ON  : REAL;
    P3_OFF : REAL;
    
    // 泵状态
    Pump1Running : BOOL := FALSE;
    Pump2Running : BOOL := FALSE;
    Pump3Running : BOOL := FALSE;
    
    // 手自动
    Pump1Auto : BOOL := TRUE;
    Pump2Auto : BOOL := TRUE;
    Pump3Auto : BOOL := TRUE;
    
    // 故障
    Pump1Fault : BOOL := FALSE;
    Pump2Fault : BOOL := FALSE;
    Pump3Fault : BOOL := FALSE;
    
    // 运行计时
    Pump1Hour  : REAL := 0.0;
    Pump2Hour  : REAL := 0.0;
    Pump3Hour  : REAL := 0.0;
END_DATA_BLOCK

PROGRAM "PUMP_Auto"
    // ---- 本地变量 ----
    VAR
        tmP1Start, tmP2Start, tmP3Start : TON;
        tmP1Stop, tmP2Stop, tmP3Stop  : TON;
        tmHourCnt : TON;
    END_VAR
    
    // ---- 阈值更新 ----
    #DB1_PumpControl.P1_ON  := #DB1_PumpControl.Setpoint - #DB1_PumpControl.Hysteresis;
    #DB1_PumpControl.P1_OFF := #DB1_PumpControl.Setpoint + #DB1_PumpControl.Hysteresis;
    #DB1_PumpControl.P2_ON  := #DB1_PumpControl.Setpoint - 2.0 * #DB1_PumpControl.Hysteresis;
    #DB1_PumpControl.P2_OFF := #DB1_PumpControl.Setpoint + 2.0 * #DB1_PumpControl.Hysteresis;
    #DB1_PumpControl.P3_ON  := #DB1_PumpControl.Setpoint - 3.0 * #DB1_PumpControl.Hysteresis;
    #DB1_PumpControl.P3_OFF := #DB1_PumpControl.Setpoint + 3.0 * #DB1_PumpControl.Hysteresis;
    
    // ---- 启停逻辑 ----
    // 1#泵
    IF #DB1_PumpControl.Actual <= #DB1_PumpControl.P1_ON 
       AND #DB1_PumpControl.Pump1Auto AND NOT #DB1_PumpControl.Pump1Fault THEN
        #tmP1Start(IN := TRUE, PT := T#30s);
        IF #tmP1Start.Q THEN
            #DB1_PumpControl.Pump1Running := TRUE;
        END_IF;
    ELSIF #DB1_PumpControl.Actual >= #DB1_PumpControl.P1_OFF 
          AND #DB1_PumpControl.Pump1Running THEN
        #tmP1Stop(IN := TRUE, PT := T#5m);
        IF #tmP1Stop.Q THEN
            #DB1_PumpControl.Pump1Running := FALSE;
        END_IF;
    END_IF;
    
    // 2#泵
    IF #DB1_PumpControl.Actual <= #DB1_PumpControl.P2_ON 
       AND #DB1_PumpControl.Pump2Auto AND NOT #DB1_PumpControl.Pump2Fault THEN
        #tmP2Start(IN := TRUE, PT := T#30s);
        IF #tmP2Start.Q THEN
            #DB1_PumpControl.Pump2Running := TRUE;
        END_IF;
    ELSIF #DB1_PumpControl.Actual >= #DB1_PumpControl.P2_OFF 
          AND #DB1_PumpControl.Pump2Running THEN
        #tmP2Stop(IN := TRUE, PT := T#5m);
        IF #tmP2Stop.Q THEN
            #DB1_PumpControl.Pump2Running := FALSE;
        END_IF;
    END_IF;
    
    // 3#泵
    IF #DB1_PumpControl.Actual <= #DB1_PumpControl.P3_ON 
       AND #DB1_PumpControl.Pump3Auto AND NOT #DB1_PumpControl.Pump3Fault THEN
        #tmP3Start(IN := TRUE, PT := T#30s);
        IF #tmP3Start.Q THEN
            #DB1_PumpControl.Pump3Running := TRUE;
        END_IF;
    ELSIF #DB1_PumpControl.Actual >= #DB1_PumpControl.P3_OFF 
          AND #DB1_PumpControl.Pump3Running THEN
        #tmP3Stop(IN := TRUE, PT := T#5m);
        IF #tmP3Stop.Q THEN
            #DB1_PumpControl.Pump3Running := FALSE;
        END_IF;
    END_IF;
    
    // ---- 输出 ----
    "Pump1_Contactor" := #DB1_PumpControl.Pump1Running;  // Q0.0
    "Pump2_Contactor" := #DB1_PumpControl.Pump2Running;  // Q0.1
    "Pump3_Contactor" := #DB1_PumpControl.Pump3Running;  // Q0.2
    
    // ---- 运行计时 ----
    #tmHourCnt(IN := TRUE, PT := T#1h);
    IF #tmHourCnt.Q THEN
        IF #DB1_PumpControl.Pump1Running THEN #DB1_PumpControl.Pump1Hour += 1.0; END_IF;
        IF #DB1_PumpControl.Pump2Running THEN #DB1_PumpControl.Pump2Hour += 1.0; END_IF;
        IF #DB1_PumpControl.Pump3Running THEN #DB1_PumpControl.Pump3Hour += 1.0; END_IF;
    END_IF;
    
END_PROGRAM
```

---

## IO定义

| 地址 | 方向 | 说明 |
|------|------|------|
| %IW64 | AI | 压力传感器（0-1.0MPa对应0-27648） |
| %Q0.0 | DO | 1#泵接触器 |
| %Q0.1 | DO | 2#泵接触器 |
| %Q0.2 | DO | 3#泵接触器 |
| %Q0.3 | DO | 变频器使能 |
| %I0.0 | DI | 1#泵手/自动 |
| %I0.1 | DI | 2#泵手/自动 |
| %I0.2 | DI | 3#泵手/自动 |
| %I0.3 | DI | 1#泵故障 |
| %I0.4 | DI | 2#泵故障 |
| %I0.5 | DI | 3#泵故障 |
