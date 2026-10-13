---
name: skill-commander-intent
label: 命令意图识别
description: "Parse and classify user command intents from natural language input. Use when the user's message contains an implicit or explicit command, action request, or operational intent that needs to be structured into a recognizable command pattern with extracted parameters. Triggers when: (1) User asks to perform an action (create, update, delete, search, analyze, generate, etc.), (2) User's request maps to a system command or workflow, (3) User input needs intent classification before routing to appropriate tools, (4) Ambiguous user requests need clarification through intent analysis."
---

# 命令意图识别

## 概述

将用户的自然语言输入解析为结构化的命令意图，包括意图分类、参数提取和执行路径确定。帮助 Aily 准确理解用户想要执行的操作，并将请求路由到正确的工具或工作流。

## 意图识别工作流

处理每条用户消息时按以下步骤执行：

### 1. 意图检测

判断用户输入是否包含命令意图：
- **显性命令**：包含动词 + 对象（如"帮我生成报告"、"搜索最近文档"）
- **隐性命令**：以问题形式表达操作需求（如"今天的日程是什么？"→ 查询日程）
- **纯信息请求**：不涉及操作，仅寻求信息（如"什么是飞书多维表格？"）→ 不触发本技能

### 2. 意图分类

将检测到的意图归入以下核心类别之一：

| 类别 | 说明 | 典型动词 |
|------|------|---------|
| `create` | 创建新资源 | 创建、生成、新建、添加、制作 |
| `read` | 查询/读取信息 | 查看、搜索、查找、获取、列出、显示 |
| `update` | 修改现有资源 | 修改、更新、编辑、调整、更改 |
| `delete` | 删除资源 | 删除、移除、清空、取消 |
| `execute` | 执行特定操作 | 执行、运行、发送、导出、同步、通知 |
| `analyze` | 分析/处理数据 | 分析、统计、汇总、计算、对比 |
| `coordinate` | 协调/安排事项 | 安排、预约、协调、邀请、提醒 |

当意图涉及多个类别时，按主要操作确定主类别，次要操作为子意图。

### 3. 参数提取

从用户输入中提取结构化参数：

**核心参数类型：**
- `target`：操作对象（文档、表格、会议、任务、人等）
- `scope`：作用范围（时间范围、数量限制、特定集合）
- `criteria`：筛选条件（关键词、状态、优先级等）
- `content`：内容描述（标题、正文、备注等）
- `recipient`：接收方（发送给谁、邀请谁）
- `deadline`：时间约束（截止日期、提醒时间）

**提取规则：**
1. 显式参数：用户直接陈述的值（如"明天下午3点"）
2. 隐式参数：从上下文推断的值（如"这份文档"指当前讨论对象）
3. 缺失参数：用户未提供但执行必需的值 → 标记为待确认

### 4. 置信度评估

对每个识别出的意图和参数评估置信度：
- **高**：意图明确，参数完整 → 直接执行
- **中**：意图较明确，但存在歧义或缺失参数 → 执行前简要确认
- **低**：意图模糊或无法匹配已知类别 → 请求用户澄清

### 5. 执行路由

根据分类结果将请求路由到对应工具或工作流：
- `create` → 文档/表格/日程创建工具
- `read` → 搜索/查询工具
- `update` → 编辑/修改工具
- `delete` → 删除/取消工具
- `execute` → 对应执行工具
- `analyze` → 数据分析/报告生成工具
- `coordinate` → 日程协调/任务分配工具

## 歧义处理

当用户输入存在多种合理解读时：

1. **列出候选意图**：明确说明检测到的多种可能意图
2. **请求澄清**：请用户确认或补充关键信息
3. **默认策略**：若存在明显更可能的解读，先按高置信度意图执行，同时说明假设

**示例：**
> 用户："帮我处理一下这个"
> 分析：目标"这个"不明确，缺少操作类型
> 响应："您希望我对当前内容进行什么操作？可以创建新文档、生成摘要、或修改格式。"

## 输出格式

识别完成后，以结构化方式记录意图分析结果：

```
意图：[主类别]
目标：[操作对象]
参数：
  - [参数名]: [值]（[显式/隐式/缺失]）
置信度：[高/中/低]
执行路径：[工具/工作流名称]
```

## 资源引用

### references/command_patterns.md

常见命令模式参考，包含各业务领域的典型命令结构和参数组合。当用户请求涉及特定领域（飞书文档、日程管理、多维表格、任务管理等）时，读取此文件获取领域特定的意图识别模式。
