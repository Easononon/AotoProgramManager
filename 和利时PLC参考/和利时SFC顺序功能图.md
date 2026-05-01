---
title: 和利时SFC顺序功能图
category: concept
sources: [SFC基础培训-张星.pdf]
source_count: 1
keywords: [和利时, SFC, 顺序功能图, 步, 动作, 转换]
subject_terms: [SFC, 顺序控制, 步进]
aliases: [SFC基础, 顺序功能图]
tags: [SFC, 顺序控制, 和利时]
---

# SFC顺序功能图（SFC基础培训）

## SFC概述

SFC（Sequential Function Chart，顺序功能图）是一种面向顺序控制问题的图形化编程语言，特别适合描述离散制造、批次处理等**顺序执行**的控制逻辑。

## 核心概念

| 概念 | 说明 |
|------|------|
| **步（Step）** | 代表一个稳定的状态，用方框表示 |
| **动作（Action）** | 步对应的操作，可为LD/ST/FBD/CFC代码块 |
| **转换（Transition）** | 步之间的切换条件 |
| **有向连线** | 连接步与转换的箭头线 |

## SFC执行模型

```
[初始步] → (转换条件) → [步1] → (转换条件) → [步2] → ...
     ↑                                                           |
     └────────────────── (循环/跳转) ──────────────────────────┘
```

## 和利时SFC实现

在PowerPro中，SFC通过 **Iecsfc.lib** 库实现：

```st
// 必须添加 Iecsfc.lib 库
SFCActionControl(STEP := stepVar, ACTION := actionBlock);
```

### IEC步 vs 普通步

| 类型 | 说明 |
|------|------|
| **IEC步** | 必须关联SFCActionControl指令 |
| **普通步** | 允许任意LD/ST/FBD/CFC代码块 |

## 相关页面

- [[和利时LK大型PLC-指令系统]]
- [[IEC61131-3标准]]
