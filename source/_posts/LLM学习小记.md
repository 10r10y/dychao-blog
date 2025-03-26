---
title: LLM 学习小记
date: 2025-03-21 19:12:09
categories:
  - CSS
tags:
  - CSS
  - 前端开发
---

## 大模型API的理解

### 新旧API对比

- 老版API：text/completions
- 新版API：chat/completions

### chat/completions的核心概念

#### completions(补全)
文本转换成模型可理解的符号（tokenization：文本 -> token -> token IDs），再对符号进行补全

#### role类型
1. System - 系统角色（人设）
2. User - 用户
3. Assistant - 对用户的回应
4. Tool - 工具：用以调用插件和函数

#### token特点
1. token 不等于单词或文字
2. 不同语言token效率不同
3. 不同模型分词器不同

#### 补全参数

- Temperature：0~2，调整概率分布差异
  - 低温：强化高概率选项，降低选择多样性
  - 高温：选择分布平缓，增加选择多样性

- Top_p（核采样）：设定累计概率阈值

- Frequency Penalty：-2~2，降低已出现token再次出现概率，迫使模型选择新的表达

- Presence Penalty：-2~2，降低已出现主题的相关概率，迫使选择新主题

## 提示词工程

### 提示词类型
1. System Prompt - 系统提示词（优先级高于 User Prompt）
2. User Prompt - 用户提示词
3. Assistant Prompt - 模型输出

### 提示词工程流程

对于制造AI工具的人来说，提示词工程是一个有向无环图：

1. 用专业知识明确需求
2. 选择合适的模型
3. 运用提示词技巧表达需求
4. （按需）拆分提示词

![提示词工程流程](/images/prompt-engineering-flow.png)

说明：
1. 图中每个模块就是一个**Runnable**
2. **Runnable**之间的组合构成链（**Chain**），完整的链构成有向无环图
3. **Chain**可能并行或串行执行

## 选择合适的模型

### 选择步骤
1. 合规隐私要求：筛选符合条件的模型
2. 领域能力评估：构建评估指标
3. 验证与测试：评估并筛选最终模型
<!-- ![模型选择流程](/images/model-selection.png) -->

## 提示词编写指南

### 提示词载体与结构
- 载体：MD、XML、JSON、伪代码
- 结构：总分总、递进式、并列式、CO-STAR、LangGPT

### 编写步骤
1. 确定特定场景下的最佳实践
2. 判断最佳实践的复杂度
   - 简单：自然语言输出
   - 中等：MD格式输出
   - 复杂：选定提示词载体和结构（默认LangGPT）
3. 使用提示词生成工具
   - [月之暗面 Kimi × LangGPT 提示词专家](https://kimi.moonshot.cn/kimiplus/conpg00t7lagbbsfqkq0)
   - [OpenAI 商店 LangGPT 提示词专家](https://chatgpt.com/g/g-Apzuylaqk-langgpt-ti-shi-ci-zhuan-jia)
   - [302提示词专家](https://dash.302.ai/tools/list)
   - [Claude提示词工具](https://console.anthropic.com/)
4. 调试并优化输出的提示词

## 提示词技巧

### 0-shot
- 适用场景：简单任务
- 特点：无示例

### few-shot
- 适用场景：
  1. 需要特定格式输出
  2. 任务规则复杂或含糊
  3. 需要保持一致性的任务
  4. 涉及主观判断的任务
- 缺点：没有推理过程，不适合需要多步骤执行的任务

### CoT (Chain-Of-Thought) 思维链
要求模型展示推理步骤，随着步骤的明确，生成随机性降低

### ToT (Tree-Of-Though)
与传统 CoT 的区别：
- 不仅有思考，还能评估，并基于评估改进行动
- 不仅能顺序执行，还能前瞻、回溯、全局探索
- 脱离单纯提示词技巧的范畴，更接近workflow或agent概念





