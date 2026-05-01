---
title: 西门子SCL运动控制指令
category: concept
sources: [西门子S7-12001500 PLC SCL语言编程从入门到精通pdf.pdf]
source_count: 1
keywords: [运动控制, MC_Power, MC_MoveAbsolute, PTO, 步进电机, 伺服, 回原点]
subject_terms: [运动控制, 轴工艺对象, MC指令]
aliases: [运动控制, 轴指令, 定位]
tags: [运动控制, 西门子, MC指令]
---

# 西门子SCL运动控制指令

## 运动控制概述

S7-1200/1500实现运动控制的三种方式：
1. **PROFIdrive** - 基于PROFINET总线的驱动技术标准
2. **PTO** - 脉冲串输出（Pulse Train Output）
3. **模拟量** - ±10V速度给定信号

### PTO脉冲类型

| 类型 | 说明 |
|------|------|
| 脉冲+方向 | 一路发脉冲，一路指示方向 |
| 加计数+减计数 | 两路脉冲分别控制正/反转 |
| A相+B相 | 90°相位差，方向由超前/滞后决定 |
| A相+B相（四倍频）| 上升沿/下降沿均计数 |

## 运动控制指令

### MC_Power - 使能/禁用运动轴

```st
"Axis1_Power"(ENABLE := bEnable, 
              START_MODE := 0, 
              STOP_MODE := 0, 
              TO_AXIS := "Axis1",
              STATUS => wStatus,
              BUSY => bBusy,
              VALID => bValid);
```

参数说明：
- `ENABLE`：轴使能（1=使能）
- `STOP_MODE`：0=急停，1=立即停止，2=减速停止
- `TO_AXIS`：轴工艺对象

### MC_Home - 回原点

```st
"Axis1_Home"(EXECUTE := bHomeTrigger,
              MODE := 3,    // 3=主动回原点
              POSITION := 0.0,
              DONE => bHomeDone,
              BUSY => bBusy,
              ERROR => bError);
```

Mode模式：
- 0=绝对式直接回原点
- 1=相对式直接回原点
- 2=被动回原点（路过原点开关）
- 3=主动回原点（寻找原点开关）
- 6/7=绝对编码器调节

### MC_MoveAbsolute - 绝对定位

```st
"Axis1_MoveAbs"(EXECUTE := bMoveTrigger,
                DISTANCE := 100.0,    // 目标位置mm
                VELOCITY := 4.0,      // 速度mm/s
                DIRECTION := 0,        // 0=自动
                DONE => bDone);
```

⚠️ **注意**：执行前必须先执行MC_Home建立绝对坐标系

### MC_MoveRelative - 相对定位

```st
"Axis1_MoveRel"(EXECUTE := bRelTrigger,
                DISTANCE := 50.0,     // 相对移动距离mm
                VELOCITY := 10.0,      // 速度mm/s
                DONE => bDone);
```

### MC_MoveVelocity - 连续速度运行

```st
"Axis1_MoveVel"(EXECUTE := bVelTrigger,
                VELOCITY := -10.0,    // 负数=反向
                DIRECTION := 0,       // 0=由速度符号决定
                CURRENT := 0,
                DONE => bDone,
                IN_VELOCITY => bInVel);
```

### MC_MoveJog - 点动控制

```st
"Axis1_Jog"(JOG_FORWARD := btnFwd,   // 保持=运行
            JOG_BACKWARD := btnBwd,
            VELOCITY := 10.0,
            DONE => bDone);
```

### MC_Halt - 减速停止

```st
"Axis1_Halt"(EXECUTE := bStopTrigger, DONE => bHaltDone);
```

### MC_Reset - 复位错误

```st
"Axis1_Reset"(EXECUTE := bResetTrigger, DONE => bResetDone);
```

## 工艺对象参数

| 参数 | 说明 |
|------|------|
| 电机每转脉冲数 | 如1600（步进细分）|
| 电机每转位移 | 如4.0mm（丝杆导程）|
| 最大速度 | 如8.0mm/s |
| 启停速度 | 如2.5mm/s |
| 加速度 | 如3.0mm/s² |
| 硬件限位 | 左右限位开关 |
| 软件限位 | 位置软限制 |

## 相关页面

- [[西门子SCL核心语法]]
- [[IEC61131-3标准]]
