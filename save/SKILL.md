---
name: save
description: Use when the user says "保存聊天记录", "保存对话", "save", or wants to save the current conversation as a Markdown document to Products/Notes/Conversation/. Triggers on any request to persist the current chat session.
---

# Save - 保存聊天记录

将当前会话总结为结构化 Markdown 文档，保存到 `Products/Notes/Conversation/`。

## 执行步骤

**1. 回顾会话，确定标题**

识别核心主题、重要决策、有价值的观点、执行过的操作，起一个简洁中文标题。

**2. 组织并写入文件**

文件名：`{YYYY-MM-DD}-{标题}.md`，保存到 `Products/Notes/Conversation/`。

文档结构：

```markdown
# {标题}

> 日期：{YYYY-MM-DD}

## 背景
[一两句话：本次对话的起因和上下文]

## 讨论要点

### {主题1}
[要点总结]

### {主题2}
[要点总结]

## 结论与决策
[结论、决策、行动项]
```

内容原则：保留有价值的观点和结论，去掉闲聊；结构化内容（表格、对比、代码）保留原始格式；优先记录用户的个人观点和决策。

写入后告知用户文件路径和内容概要。

## 红线

- **禁止** 逐条复制对话原文，必须总结提炼
- **禁止** 遗漏会话中的重要决策或结论
- **禁止** 添加会话中没有讨论过的内容
- **禁止** 省略用户表达的个人观点和立场
