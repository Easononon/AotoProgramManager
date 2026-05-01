---
title: 和利时LM小型PLC指令系统
category: concept
sources: [LM小型可编程控制器指令手册.pdf]
source_count: 1
keywords: [和利时, LM, PowerPro, 小型PLC, 一体化PLC]
subject_terms: [和利时, 小型PLC, 指令参考]
aliases: [和利时LM, LM-PLC]
tags: [和利时, LM, 小型PLC]
---

# 和利时LM小型PLC指令系统

## 概述

LM小型PLC是和利时推出的新一代**小型一体化PLC**，使用与LK相同的 **PowerPro** 编程软件，指令系统与LK大型PLC高度兼容（基本一致）。

**LM vs LK 主要差异：**
- LK适用于中高性能控制领域（大型项目）
- LM适用于小型自动化项目（一体化结构）

## 编程语言

与LK相同，支持：
- **ST**（结构化文本）
- **LD**（梯形图）
- **FBD**（功能块图）
- **SFC**（顺序功能图）
- **CFC**（连续功能图）

## 核心指令（与LK基本通用）

### 定时器/计数器
| 指令 | 说明 |
|------|------|
| `TON` | 通电延时 |
| `TOF` | 断电延时 |
| `TP` | 脉冲 |
| `CTU` | 加计数 |
| `CTD` | 减计数 |
| `CTUD` | 加/减计数 |

### 边沿检测
| 指令 | 说明 |
|------|------|
| `R_TRIG` | 上升沿 |
| `F_TRIG` | 下降沿 |

### 数据类型
BOOL / SINT / USINT / INT / UINT / DINT / UDINT / REAL / TIME / DATE / TOD / DT / STRING 等，与LK完全一致。

## 与LK的差异（待补充）

LM与LK指令系统高度兼容，主要差异在于：
- 硬件规模和扩展能力不同
- LM为一体化CPU，LK为模块化
- 部分高性能指令（如HS_SOE、HS_Diagnosis等大型通讯扩展）可能LM不支持

## 相关页面

- [[和利时LK大型PLC-指令系统]]
- [[IEC61131-3标准]]
- [[品牌差异对照表]]
