---
title: Controlling context
date: 2026/2/2
categories:
- Claude
---

<center>
Claude Code in Action —— Controlling context
(使用claude code输出的总结)
</center>

<!--more-->
***

# 控制上下文 (Controlling context)

> 课程来源: Anthropic Academy - Claude Code in Action
> 课程链接: https://anthropic.skilljar.com/claude-code-in-action/303237

## 课程简介

在与 Claude 合作处理复杂任务时，您需要掌握多种技巧来引导对话、保持专注并提高效率。本课程介绍了 Claude Code 提供的上下文控制技术，帮助您在开发过程中保持高效的 AI 辅助工作流程。

---

## 1. 使用 Escape 打断 Claude (Interrupting Claude with Escape)

### 适用场景

当 Claude 开始走向错误的方向或试图一次性处理太多任务时，可以按 **Escape 键** 暂停 Claude 的响应，允许您重新引导对话。

### 实用示例

当您要求 Claude 为多个函数编写测试，而它开始为所有函数创建综合计划时，可以中断并要求它一次只专注于一个函数。

---

## 2. 结合 Escape 与记忆功能 (Combining Escape with Memories)

### 修复重复错误

当 Claude 在不同对话中反复犯同样的错误时，可以使用强大的组合技：

1. 按 **Escape** 停止当前响应
2. 使用 **# 快捷键** 添加关于正确方法的记忆
3. 使用更正后的信息继续对话

这可以防止 Claude 在未来的项目对话中重复同样的错误。

---

## 3. 回溯对话 (Rewinding Conversations)

### 问题背景

在长时间对话中，您可能会积累变得无关紧要或分散注意力的上下文。例如，当 Claude 遇到错误并花时间调试时，那种来回讨论可能对下一个任务没有帮助。

### 解决方案

按 **Escape 两次** 可以回溯对话，显示您发送的所有消息，允许您跳回 earlier 的某个点并从那里继续。

### 好处

- 保留有价值的上下文（如 Claude 对代码库的理解）
- 移除分散注意力或无关的对话历史
- 让 Claude 专注于当前任务

---

## 4. 上下文管理命令 (Context Management Commands)

### `/compact`

`/compact` 命令总结您的整个对话历史，同时保留 Claude 学到的关键信息。适用于：

- Claude 已经获得了关于项目的宝贵知识
- 您想继续执行相关任务
- 对话变长但包含重要上下文

### `/clear`

`/clear` 命令完全清除对话历史，提供一个全新的开始。适用于：

- 切换到完全不同的、不相关的任务
- 当前对话上下文可能会让 Claude 对新任务感到困惑
- 您想重新开始而不带任何先前上下文

---

## 5. 何时使用这些技术 (When to Use These Techniques)

这些对话控制技术在以下情况特别有价值：

| 场景 | 推荐技术 |
|------|----------|
| 长期对话，上下文变得混乱 | `/compact` 或双击 Escape |
| 任务切换，之前上下文可能分散注意力 | `/clear` |
| Claude 反复犯同样的错误 | Escape + # 添加记忆 |
| 复杂项目，需要专注于特定组件 | Escape 中断后重新聚焦 |

---

## 6. 总结

这些上下文控制技术不仅仅是便利功能——它们是保持有效 AI 辅助开发会话的重要工具：

- **Escape 键** - 打断并重新引导对话方向
- **双击 Escape** - 回溯对话历史
- **# 快捷键** - 添加记忆以纠正重复错误
- **`/compact`** - 压缩并保留关键上下文
- **`/clear`** - 完全重置对话

通过战略性地使用这些技术，您可以在整个开发工作流程中保持 Claude 的专注和高效率。

---
