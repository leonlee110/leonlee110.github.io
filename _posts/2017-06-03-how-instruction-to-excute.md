---
title: 指令是怎样执行的
layout: page
categories: HardwareArchitecture
tags: cpu
---

一条指令执行的过程。

<!-- excerpt -->

## 1. Fetch Instruction
![fetch_instruction](/assets/cpu/fetch_instruction.png){:class="img-responsive center-block" width="600px"}
常见性能指标：
```
- L1I cache miss：在L1I cache中找不到指令，需要从内存或者硬盘中读取新的指令
- Predict Failure miss：分之预测失败，会在后续阶段flush已执行的指令
- TLB miss：地址转换规则不在TLB中，需要从内存中读取
- L1I cache bound：L1I取指令花费的时间
```

## 2. Decode
![decode_pipeline](/assets/cpu/decode_pipeline.png){:class="img-responsive center-block" width="600px"}


## 3. Allocation
```
- Data Dependence stall：由于数据依赖性导致的stall
- Name Dependence stall：由于寄存器依赖导致的stall
- Function Unit stall：由于可执行单元缺乏导致的stall
- Issue queue stall：由于issue queue缺乏导致的stall
```

## 4. Issue
```
- Issue queue
- RS
- Dependence Predict Failure：依赖预测失败
```

## 5. execute
![execute_pipeline](/assets/cpu/execute_pipeline.png){:class="img-responsive center-block" width="600px"}
```
- ALC：整形和逻辑预算单元
- AGU：地址转换预算单元
- FP：浮点计算单元
- IMLU/IDIV：乘法/除法预算单元
- BU：分支单元，执行控制流操作
- SMID：单指令多数据执行单元
```

## 6. Commit
