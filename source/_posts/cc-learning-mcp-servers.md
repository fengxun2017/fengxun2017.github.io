---
title: MCP servers with Claude Code
date: 2026/2/4
categories:
- Claude
---

<center>
Claude Code in Action —— MCP servers with Claude Code，
(使用claude code输出的总结)
</center>

<!--more-->
***

# MCP 服务器与 Claude Code (MCP Servers with Claude Code)

> 课程来源: Anthropic Academy - Claude Code in Action
> 课程链接: https://anthropic.skilljar.com/claude-code-in-action/303239

## 课程简介

MCP（Model Context Protocol，模型上下文协议）服务器可以扩展 Claude Code 的能力。这些服务器可以在远程或本地运行，为 Claude 提供新的工具和功能。本课程介绍如何安装和使用 MCP 服务器，以增强开发工作流。

---

## 1. 什么是 MCP 服务器？ (What is an MCP Server?)

MCP 服务器是运行在本地或远程的服务器程序，它为 Claude Code 提供额外的工具和能力。

**核心价值：**
- 扩展 Claude Code 的原生功能
- 让 Claude 能够与外部工具和服务交互
- 将 Claude 从代码助手转变为全面的开发合作伙伴

---

## 2. 安装 Playwright MCP 服务器 (Installing the Playwright MCP Server)

### 安装命令

在终端中（**不在 Claude Code 内**）运行：

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

### 命令说明

这个命令做了两件事：
1. 将 MCP 服务器命名为 "playwright"
2. 提供在本地启动服务器的指令

### Playwright 的能力

Playwright 是最受欢迎的 MCP 服务器之一，它让 Claude 能够控制 Web 浏览器，为 Web 开发工作流打开了强大的可能性。

---

## 3. 管理权限 (Managing Permissions)

### 权限提示

首次使用 MCP 服务器工具时，Claude 会每次都请求权限。

### 预授权配置

如果想免除重复的权限提示，可以编辑配置文件进行预授权：

**`.claude/settings.local.json` 内容：**

```json
{
  "permissions": {
    "allow": ["mcp__playwright"],
    "deny": []
  }
}
```

**注意：** `mcp__playwright` 中的双下划线是必需的语法。

### 效果

配置后，Claude 可以直接使用 Playwright 工具，无需每次都请求权限。

---

## 4. 实际应用示例：改进组件生成 (Practical Example: Improving Component Generation)

### 工作流演示

Playwright MCP 服务器可以显著改进开发工作流。Claude 可以：

1. 打开浏览器并导航到应用程序
2. 生成测试组件
3. 分析视觉样式和代码质量
4. 根据观察结果更新生成提示词
5. 用改进后的提示词测试新组件

### 示例请求

```
"Navigate to localhost:3000, generate a basic component, review the styling, and update the generation prompt at @src/lib/prompts/generation.tsx to produce better components going forward."
```

### Claude 的操作流程

1. 使用浏览器工具与应用程序交互
2. 检查生成的输出
3. 修改提示词文件以鼓励更具创意和原创性的设计

---

## 5. 效果与收益 (Results and Benefits)

### 改进效果

使用 MCP 服务器后，Claude 生成的组件质量会显著提升：

| 改进前 | 改进后 |
|--------|--------|
| 通用紫色到蓝色渐变 | 暖色调日落渐变（橙色-粉色-紫色） |
| 标准 Tailwind 模式 | 海洋深度主题（青色-翡翠色-青色） |
| 对称设计 | 非对称设计和重叠元素 |
| 常规间距 | 创意间距和非传统布局 |

### 核心优势

**关键优势在于：** Claude 可以看到实际的视觉输出，而不仅仅是代码。这使它能够做出更明智的样式改进决策。

---

## 6. 探索其他 MCP 服务器 (Exploring Other MCP Servers)

Playwright 只是 MCP 服务器可能性的一种示例。MCP 生态系统包括用于以下用途的服务器：

| 类别 | 用途 |
|------|------|
| 数据库交互 | 查询、修改数据库 |
| API 测试和监控 | 测试和监控 API 端点 |
| 文件系统操作 | 高级文件管理和操作 |
| 云服务集成 | 与 AWS、Azure、GCP 等集成 |
| 开发工具自动化 | 自动化 IDE、Git 等工具 |

### 建议

考虑探索与你的特定开发需求一致的 MCP 服务器。它们可以将 Claude 从代码助手转变为可以与整个工具链交互的全面开发合作伙伴。

---

## 7. 总结

MCP 服务器是扩展 Claude Code 能力的强大方式：

1. **安装简单**：使用 `claude mcp add` 命令即可添加
2. **权限可控**：可配置预授权或每次确认
3. **生态丰富**：多种服务器满足不同需求
4. **开发效率提升**：让 Claude 能够看到实际运行效果，做出更明智的决策

通过合理使用 MCP 服务器，可以将 Claude Code 打造成一个真正全面的开发伙伴。
---
