---
title: Github integration
date: 2026/2/5
categories:
- Claude
---

<center>
Claude Code in Action —— Github integration
(使用claude code输出的总结)
</center>

<!--more-->
***

# GitHub 集成 (Github integration)

> 课程来源: Anthropic Academy - Claude Code in Action
> 课程链接: https://anthropic.skilljar.com/claude-code-in-action/303240

## 课程简介

Claude Code 提供了一个官方的 GitHub 集成，允许 Claude 在 GitHub Actions 中运行。该集成提供两个主要工作流：issues 和 pull requests 的 @mention 支持，以及自动 pull request 审查。

---

## 1. 设置集成 (Setting Up the Integration)

运行 `/install-github-app` 命令启动设置流程：

1. **安装 Claude Code 应用** - 在 GitHub 上安装 Claude Code 应用
2. **添加 API key** - 添加您的 API 密钥
3. **生成工作流文件** - 自动生成包含工作流文件的 pull request

合并 PR 后，工作流文件将存在于 `.github/workflows` 目录中。

---

## 2. 默认 GitHub Actions (Default GitHub Actions)

### Mention Action

在任何 issue 或 pull request 中使用 `@claude` 提及 Claude，Claude 将会：

- **分析请求** - 分析任务并创建执行计划
- **执行任务** - 全面访问代码库完成任务
- **回复结果** - 直接在 issue 或 PR 中回复结果

### Pull Request Action

创建 PR 时，Claude 自动：

- **审查代码** - 审查提议的更改
- **分析影响** - 分析修改的影响范围
- **发布报告** - 在 PR 上发布详细审查报告

---

## 3. 自定义工作流 (Customizing the Workflows)

合并初始 PR 后，可以自定义工作流文件以满足项目需求。

### 添加项目设置步骤

```yaml
- name: Project Setup
  run: |
    npm run setup
    npm run dev:daemon
```

### 自定义指令

```yaml
custom_instructions: |
  The project is already set up with all dependencies installed.
  The server is already running at localhost:3000. Logs from it
  are being written to logs.txt. If needed, you can query the
  db with the 'sqlite3' cli. If needed, use the mcp__playwright
  set of tools to launch a browser and interact with the app.
```

### MCP 服务器配置

```yaml
mcp_config: |
  {
    "mcpServers": {
      "playwright": {
        "command": "npx",
        "args": [
          "@playwright/mcp@latest",
          "--allowed-origins",
          "localhost:3000;cdn.tailwindcss.com;esm.sh"
        ]
      }
    }
  }
```

---

## 4. 工具权限 (Tool Permissions)

在 GitHub Actions 中运行 Claude 时，必须显式列出所有允许的工具。

```yaml
allowed_tools: "Bash(npm:*),Bash(sqlite3:*),mcp__playwright__browser_snapshot,mcp__playwright__browser_click,..."
```

### 重要提示

与本地开发不同，**GitHub Actions 中没有权限快捷方式**，每个 MCP 服务器的工具都必须单独列出。

---

## 5. 最佳实践 (Best Practices)

| 实践 | 说明 |
|------|------|
| 渐进式定制 | 从默认工作流开始，逐步自定义 |
| 提供上下文 | 使用自定义指令提供项目特定信息 |
| 明确权限 | 使用 MCP 服务器时明确工具权限 |
| 测试先行 | 先用简单任务测试工作流 |
| 按需配置 | 根据项目具体需求配置额外步骤 |

---

## 6. 总结

GitHub 集成将 Claude 从开发助手转变为自动化团队成员，可以：

- **处理任务** - 通过 @mention 在 issues 和 PR 中执行任务
- **审查代码** - 自动审查 pull request
- **集成工作流** - 在 GitHub 工作流中直接提供见解

通过战略性使用这些功能，您可以将 Claude 无缝集成到现有的 GitHub 开发流程中。

---
