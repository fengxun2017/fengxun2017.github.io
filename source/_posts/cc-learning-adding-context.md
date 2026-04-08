---
title: Adding context
date: 2026/1/31
categories: 
- Claude
---

<center>
Claude Code in Action —— Adding context 
(使用claude code输出的总结)
</center>

<!--more-->
***

# 添加上下文 (Adding Context)

> 课程来源: Anthropic Academy - Claude Code in Action
> 课程链接: https://anthropic.skilljar.com/claude-code-in-action/303241

## 课程简介

在使用 Claude 进行编码项目时，上下文管理至关重要。您的项目可能有数十或数百个文件，但 Claude 只需要正确的信息即可有效地帮助您。过多的无关上下文实际上会降低 Claude 的性能，因此学会引导它关注相关文件和文档至关重要。

---

## 1. /init 命令 (The /init Command)

### 什么是 /init？

当您第一次在新项目中启动 Claude 时，运行 `/init` 命令。这会告诉 Claude 分析您的整个代码库并理解：

- **项目目的和架构** (Project's purpose and architecture)
- **关键命令和开发工作流程** (Key commands and development workflows)
- **编码模式和结构** (Coding patterns and structure)

### 工作流程

1. 用户运行 `/init` 命令
2. Claude 分析代码库
3. Claude 创建摘要并写入 `CLAUDE.md` 文件
4. 系统询问权限：
   - **Enter** - 逐个批准每个写入操作
   - **Shift+Tab** - 允许 Claude 在整个会话中自由写入文件

---

## 2. CLAUDE.md 文件 (The CLAUDE.md File)

### 主要用途

`CLAUDE.md` 文件有两个主要目的：

1. **引导 Claude 浏览代码库**
   - 指出重要命令
   - 说明架构
   - 定义编码风格

2. **自定义 Claude 的行为**
   - 给 Claude 特定或自定义的指令

### 核心机制

这个文件会包含在您向 Claude 发出的每个请求中，就像为项目设置了一个持久的系统提示。

---

## 3. CLAUDE.md 文件位置 (CLAUDE.md File Locations)

Claude 识别三个不同位置的 `CLAUDE.md` 文件：

### 📄 CLAUDE.md
- **位置**: 项目根目录
- **用途**: 通过 /init 生成，提交到源代码管理
- **范围**: 与其他工程师共享的项目级指令

### 📄 CLAUDE.local.md
- **位置**: 项目根目录
- **用途**: 不与其他工程师共享
- **范围**: 包含个人指令和 Claude 的自定义配置

### 📄 ~/.claude/CLAUDE.md
- **位置**: 用户主目录下的 .claude 文件夹
- **用途**: 用于机器上的所有项目
- **范围**: 全局指令，适用于所有项目

---

## 4. 添加自定义指令 (Adding Custom Instructions)

### Memory Mode（记忆模式）

您可以通过向 `CLAUDE.md` 文件添加指令来自定义 Claude 的行为。

#### 使用方法

使用 `#` 命令进入 "memory mode" - 这让您可以智能地编辑 `CLAUDE.md` 文件。

**示例:**
```
# Use comments sparingly. Only comment complex code.
```

Claude 会自动将这条指令合并到您的 `CLAUDE.md` 文件中。

#### 典型使用场景

- 修正 Claude 的不良习惯（如添加过多注释）
- 定义项目特定的编码规范
- 设置默认行为偏好

---

## 5. 使用 @ 符号引用文件 (File Mentions with '@')

### 基本用法

当您需要 Claude 查看特定文件时，使用 `@` 符号后跟文件路径。这会自动将该文件的内容包含在您的请求中。

### 示例

**用户输入:**
```
How does the auth system work? @auth
```

**Claude 的行为:**
1. 显示与 "auth" 相关的文件列表供选择
2. 将选定文件包含在对话中
3. 基于文件内容回答问题

### 好处

- **精准定位**: 直接引用需要的文件
- **节省时间**: 无需手动复制粘贴代码
- **保持焦点**: Claude 只看到相关上下文

---

## 6. 在 CLAUDE.md 中引用文件 (Referencing Files in CLAUDE.md)

### 持久性文件引用

您还可以在 `CLAUDE.md` 文件中直接使用 `@` 语法提及文件。这对于与项目多个方面相关的文件特别有用。

### 示例

**在 CLAUDE.md 中添加:**
```markdown
The database schema is defined in the @prisma/schema.prisma file.
Reference it anytime you need to understand the structure of data
stored in the database.
```

### 工作机制

当您以这种方式提及文件时，其内容会**自动包含在每个请求中**，因此 Claude 可以：
- 立即回答有关数据结构的问题
- 无需每次都搜索和读取架构文件
- 保持对核心项目文件的持续理解

---

## 7. 关键要点 (Key Takeaways)

### ✅ 上下文管理的重要性
- 过多的无关上下文会**降低性能**
- 只提供相关信息以获得最佳结果

### ✅ 三种 CLAUDE.md 文件
1. 项目级（CLAUDE.md）- 团队共享
2. 本地级（CLAUDE.local.md）- 个人使用
3. 全局级（~/.claude/CLAUDE.md）- 所有项目

### ✅ 两种上下文添加方式
- **临时**: 使用 `@文件路径` 在对话中引用
- **持久**: 在 CLAUDE.md 中使用 `@` 语法永久引用

### ✅ Memory Mode
- 使用 `#` 开头的消息快速添加项目指令
- Claude 会智能地将指令合并到合适位置

