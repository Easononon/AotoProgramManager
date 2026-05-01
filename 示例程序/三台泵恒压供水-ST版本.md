// ============================================================
// 三台泵恒压供水 - 通用ST版本（IEC 61131-3）
// 适用：西门子SCL / 和利时LK / 施耐德Unity Pro / 浙中控
// ============================================================

PROGRAM PumpAutoControl
VAR
    // ---- 模拟量参数 ----
    rSetpoint    : REAL := 0.5;     // 设定压力(MPa)，可根据需要修改
    rActual      : REAL := 0.0;     // 实际压力（由AI模块读取）%IW0
    rHysteresis  : REAL := 0.03;   // 滞环宽度(MPa)，防止震荡

    // ---- 启停阈值（基准=设定值±滞环）----
    // 第1台：最近设定值，轻微波动即启停
    // 第2台：压力较低时启动，压力恢复时最后停止
    // 第3台：最后启动，最先停止（压力高时卸载）
    rP1_ON   : REAL := rSetpoint - rHysteresis;      // P1启动阈值
    rP1_OFF  : REAL := rSetpoint + rHysteresis;      // P1停止阈值
    rP2_ON   : REAL := rSetpoint - 2.0*rHysteresis; // P2启动阈值
    rP2_OFF  : REAL := rSetpoint + 2.0*rHysteresis; // P2停止阈值
    rP3_ON   : REAL := rSetpoint - 3.0*rHysteresis; // P3启动阈值
    rP3_OFF  : REAL := rSetpoint + 3.0*rHysteresis; // P3停止阈值

    // ---- 泵状态标志 ----
    xPump1Running : BOOL := FALSE;
    xPump2Running : BOOL := FALSE;
    xPump3Running : BOOL := FALSE;
    xPump1Started : BOOL := FALSE;  // 已正式启动（经过延时）
    xPump2Started : BOOL := FALSE;
    xPump3Started : BOOL := FALSE;

    // ---- 泵手自动 ----
    xPump1Auto   : BOOL := TRUE;   // 自动模式
    xPump2Auto   : BOOL := TRUE;
    xPump3Auto   : BOOL := TRUE;

    // ---- 定时器 ----
    tmP1Start : TON;  // P1启动延时（防止频繁启停）
    tmP2Start : TON;
    tmP3Start : TON;
    tmP1Stop  : TON;  // P1停止延时（最小运行时间）
    tmP2Stop  : TON;
    tmP3Stop  : TON;

    // ---- 运行计时器（用于泵轮换）----
    nP1RunHour : REAL := 0.0;  // P1累计运行小时
    nP2RunHour : REAL := 0.0;
    nP3RunHour : REAL := 0.0;
    tmRunCnt   : TON;         // 运行计时基准（每小时更新一次累计）

    // ---- 控制逻辑中间变量 ----
    xNeedPump1 : BOOL := FALSE;  // P1需要运行
    xNeedPump2 : BOOL := FALSE;  // P2需要运行
    xNeedPump3 : BOOL := FALSE;  // P3需要运行
    xAnyPumpRunning : BOOL := FALSE;

    // ---- 故障标志 ----
    xPump1Fault : BOOL := FALSE;
    xPump2Fault : BOOL := FALSE;
    xPump3Fault : BOOL := FALSE;

    // ---- 输出映射 ----
    // Q0.0 = 1号泵接触器
    // Q0.1 = 2号泵接触器
    // Q0.2 = 3号泵接触器
    // Q0.3 = 变频器启动
    // Q0.4 = 报警蜂鸣器
END_VAR

// ============================================================
// 第一段：计算实际需要的泵数量
// ============================================================
xAnyPumpRunning := xPump1Running OR xPump2Running OR xPump3Running;

// ---- 启停逻辑（核心思想：压力低则开水，压力大则关水）----
IF rActual <= rP1_ON AND xPump1Auto AND NOT xPump1Fault THEN
    xNeedPump1 := TRUE;
ELSIF rActual >= rP1_OFF AND xPump1Running THEN
    xNeedPump1 := FALSE;
ELSE
    xNeedPump1 := xPump1Running;  // 保持当前状态
END_IF;

IF rActual <= rP2_ON AND xPump2Auto AND NOT xPump2Fault THEN
    xNeedPump2 := TRUE;
ELSIF rActual >= rP2_OFF AND rActual <= rP1_OFF AND xPump2Running THEN
    xNeedPump2 := FALSE;
ELSIF rActual >= rP1_OFF AND xPump2Running THEN
    xNeedPump2 := FALSE;  // 压力高时P2先停
ELSE
    xNeedPump2 := xPump2Running;
END_IF;

IF rActual <= rP3_ON AND xPump3Auto AND NOT xPump3Fault THEN
    xNeedPump3 := TRUE;
ELSIF rActual >= rP3_OFF AND xPump3Running THEN
    xNeedPump3 := FALSE;
ELSE
    xNeedPump3 := xPump3Running;
END_IF;

// ============================================================
// 第二段：启动延时（防止频繁启停，每次启动至少延时30秒）
// ============================================================
// 1号泵启动
tmP1Start(IN := xNeedPump1 AND NOT xPump1Running, PT := T#30s);
IF tmP1Start.Q AND xNeedPump1 THEN
    xPump1Running := TRUE;
END_IF;

// 2号泵启动
tmP2Start(IN := xNeedPump2 AND NOT xPump2Running, PT := T#30s);
IF tmP2Start.Q AND xNeedPump2 THEN
    xPump2Running := TRUE;
END_IF;

// 3号泵启动
tmP3Start(IN := xNeedPump3 AND NOT xPump3Running, PT := T#30s);
IF tmP3Start.Q AND xNeedPump3 THEN
    xPump3Running := TRUE;
END_IF;

// ============================================================
// 第三段：停止延时（最少运行5分钟后才能停止，保护泵）
// ============================================================
// 1号泵停止
tmP1Stop(IN := NOT xNeedPump1 AND xPump1Running, PT := T#5m);
IF tmP1Stop.Q THEN
    xPump1Running := FALSE;
END_IF;

// 2号泵停止
tmP2Stop(IN := NOT xNeedPump2 AND xPump2Running, PT := T#5m);
IF tmP2Stop.Q THEN
    xPump2Running := FALSE;
END_IF;

// 3号泵停止
tmP3Stop(IN := NOT xNeedPump3 AND xPump3Running, PT := T#5m);
IF tmP3Stop.Q THEN
    xPump3Running := FALSE;
END_IF;

// ============================================================
// 第四段：泵运行计时（用于轮换参考）
// ============================================================
tmRunCnt(IN := TRUE, PT := T#1h);  // 每小时复位一次
IF tmRunCnt.Q THEN
    IF xPump1Running THEN nP1RunHour := nP1RunHour + 1.0; END_IF;
    IF xPump2Running THEN nP2RunHour := nP2RunHour + 1.0; END_IF;
    IF xPump3Running THEN nP3RunHour := nP3RunHour + 1.0; END_IF;
END_IF;

// ============================================================
// 第五段：故障检测（超时无压力、电机过载等）
// ============================================================
// 3台泵都在运行但压力持续低于P3启动阈值30秒 → 可能有故障
IF xPump1Running AND xPump2Running AND xPump3Running AND rActual < rP3_ON THEN
    ;  // 可扩展：启动故障计时器，超时报警
END_IF;

// 任一泵故障时，不允许该泵启动（由外部故障信号触发）
// IF xPump1Fault THEN xPump1Running := FALSE; END_IF;

// ============================================================
// 第六段：输出赋值
// ============================================================
Pump1Start := xPump1Running;  // → Q0.0
Pump2Start := xPump2Running;  // → Q0.1
Pump3Start := xPump3Running;  // → Q0.2

// 蜂鸣器：3台泵全开且压力仍未达标（10分钟未到设定值）时报警
// 扩展：IF xPump1Running AND xPump2Running AND xPump3Running AND rActual < rSetpoint - 0.1 AND faultTimer.Q THEN Buzzer := TRUE; END_IF;

END_PROGRAM
