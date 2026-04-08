---
title: Managing context on the Claude Developer Platform
date: 2026/2/25
categories:
- Claude
---

<center>
Managing context on the Claude Developer Platform
，内容为 Claude 开发者平台上下文管理功能的介绍。(使用claude code输出的总结)
</center>

<!--more-->
***

# Claude 开发者平台上下文管理 (Managing Context)

> 文章来源: Anthropic - Claude Blog
> 文章链接: https://claude.com/blog/context-management
> 发布日期: September 29, 2025

## 文章简介

本文介绍了 Claude 开发者平台推出的新型上下文管理能力：**上下文编辑（Context Editing）** 和 **记忆工具（Memory Tool）**。借助 Claude Sonnet 4.5 模型，这些功能使开发者能够构建能够处理长时间运行任务的 AI 代理，同时避免触及上下文限制或丢失关键信息。

---

## 1. 上下文管理解决的问题

### 上下文窗口的局限性

随着生产环境中的代理处理更复杂的任务并生成更多的工具结果，它们经常耗尽有效的上下文窗口——这迫使开发者在削减代理记录和降低性能之间做出选择。

上下文管理通过以下两种方式解决此问题：
- 确保仅相关数据保留在上下文中
- 跨会话保留有价值的见解

---

## 2. 上下文编辑 (Context Editing)

**上下文编辑** 自动清除上下文窗口中过时的工具调用和结果。当代理执行任务并累积工具结果时，上下文编辑会删除陈旧内容，同时保留对话流，从而有效延长代理的运行时间而无需手动干预。

### 主要优势
- **延长对话时间**：自动从上下文中移除过时的工具结果
- **提升性能**：Claude 只需关注相关上下文，提高模型表现
- **减少手动干预**：代理可以更长时间自主运行

---

## 3. 记忆工具 (Memory Tool)

**记忆工具** 使 Claude 能够通过基于文件的系统在上下文窗口之外存储和查询信息。Claude 可以在专用内存目录中创建、读取、更新和删除文件，这些文件存储在用户的基础设施中，可以在会话之间持久保存。

### 主要功能
- **跨会话持久化**：在对话之间保持信息
- **构建知识库**：代理可以随着时间推移积累知识
- **维护项目状态**：跨会话引用之前的进展
- **客户端操作**：完全通过工具调用在客户端执行

### 使用场景
- 保存调试见解和架构决策
- 存储研究关键发现
- 保存中间处理结果

---

## 4. 构建长时间运行的代理

Claude Sonnet 4.5 是世界上构建代理的最佳模型。这些功能为长时间运行的代理开启了新的可能性：

| 应用场景 | Context Editing 作用 | Memory Tool 作用 |
|----------|---------------------|------------------|
| **编程** | 清除旧的文件读取和测试结果 | 保存调试见解和架构决策 |
| **研究** | 清除旧的搜索结果 | 存储关键发现，构建知识库 |
| **数据处理** | 清除原始数据 | 存储中间结果 |

---

## 5. 性能改进

根据 Anthropic 内部对代理搜索的评估集测试：

| 方案 | 性能提升 |
|------|----------|
| 记忆工具 + 上下文编辑 | **+39%** |
| 单独使用上下文编辑 | **+29%** |

### Token 消耗优化

在 100 轮网络搜索评估中：
- 上下文编辑使代理能够完成因上下文耗尽而原本会失败的工作流
- **Token 消耗减少 84%**

---

## 6. 开始使用

这些功能现已在 Claude 开发者平台上公开发布beta版本，同时在 Amazon Bedrock 和 Google Cloud 的 Vertex AI 上也可使用。

### 相关文档
- [Context Editing 文档](https://docs.claude.com/en/docs/build-with-claude/context-editing)
- [Memory Tool 文档](https://docs.claude.com/en/docs/agents-and-tools/tool-use/memory-tool)
- [Cookbook 教程](https://platform.claude.com/cookbook/tool-use-memory-cookbook)

---

## 7. 总结

Claude 开发者平台的上下文管理功能为构建更强大的 AI 代理提供了关键支持：

1. **Context Editing** - 自动管理上下文窗口，移除过时内容，延长对话长度
2. **Memory Tool** - 跨会话持久化信息，构建知识库
3. **性能显著提升** - 最高 39% 的性能提升，84% 的 token 消耗减少

这些改进使开发者能够构建能够处理复杂、长时任务的 AI 代理，推动 AI 应用进入新的发展阶段。


**备注**：本文提到的 Memory Tool 和 Context Editing 功能，是 Claude Developer Platform（面向企业/开发者）中的功能，需要：
  - 使用平台 API
  - 或在 Amazon Bedrock / Google Vertex AI 上调用
  - 使用方法参考：[Cookbook 教程](https://platform.claude.com/cookbook/tool-use-memory-cookbook)

在 Claude Code CLI 中，可以使用：
- 在 CLAUDE.md 中编写项目级/用户级的重要规则，或使用claude-mem 这样的插件来实现信息持久化。
- 使用 /compace，/clear 来精简或清除上下文信息。