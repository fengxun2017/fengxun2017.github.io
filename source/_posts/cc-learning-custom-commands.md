---
title: Custom commands
date: 2026/2/3
categories:
- Claude
---

<center>
Claude Code in Action —— Custom commands
(使用claude code输出的总结)
</center>

<!--more-->
***

# 自定义命令 (Custom Commands)

> 课程来源: Anthropic Academy - Claude Code in Action
> 课程链接: https://anthropic.skilljar.com/claude-code-in-action/303234

## 课程简介

Claude Code 内置了以斜杠 `/` 开头的命令，但你也可以创建自己的**自定义命令**来自动化频繁执行的重复任务。本课程介绍如何设置和使用自定义命令，提高开发效率。

---

## 1. 创建自定义命令 (Creating Custom Commands)

### 设置步骤

要创建自定义命令，需要设置特定的文件夹结构：

1. 在项目目录中找到或创建 `.claude` 文件夹
2. 在其中创建一个名为 `commands` 的目录
3. 创建一个新的 Markdown 文件，文件名即为命令名（如 `audit.md`）

**关键点：**
- 文件名会成为命令名 → `audit.md` 创建 `/audit` 命令
- 创建命令文件后，需要重启 Claude Code 才能识别新命令

---

## 2. 示例：审计命令 (Example: Audit Command)

### 实际应用场景

以下是一个审计项目依赖漏洞的自定义命令示例：

**`commands/audit.md` 内容：**

```markdown
Run a security audit on the project's dependencies.

Steps:
1. Run `npm audit` to find vulnerable installed packages
2. Run `npm audit fix` to apply updates
3. Run tests to verify the updates didn't break anything
```

### 功能说明

这个审计命令执行三个步骤：
1. 运行 `npm audit` 查找有漏洞的已安装包
2. 运行 `npm audit fix` 应用更新
3. 运行测试验证更新没有破坏任何功能

---

## 3. 带参数的命令 (Commands with Arguments)

### 使用 `$ARGUMENTS` 占位符

自定义命令可以使用 `$ARGUMENTS` 占位符来接受参数，大大提高了命令的灵活性和可重用性。

### 示例：测试生成命令

**`commands/write_tests.md` 内容：**

```markdown
Write comprehensive tests for: $ARGUMENTS

Testing conventions:
* Use Vitests with React Testing Library
* Place test files in a __tests__ directory in the same folder as the source file
* Name test files as [filename].test.ts(x)
* Use @/ prefix for imports

Coverage:
* Test happy paths
* Test edge cases
* Test error states
```

### 调用方式

```bash
/write_tests the use-auth.ts file in the hooks directory
```

**参数不一定是文件路径**——可以是任何字符串，为 Claude 提供任务上下文和方向。

---

## 4. 关键优势 (Key Benefits)

| 优势 | 说明 |
|------|------|
| **自动化** | 将重复工作流转化为单一命令 |
| **一致性** | 确保每次都执行相同的步骤 |
| **上下文** | 为 Claude 提供特定的说明和约定 |
| **灵活性** | 使用参数让命令处理不同输入 |

---

## 5. 适用场景 (Use Cases)

自定义命令特别适用于以下项目特定工作流：

- 运行测试套件
- 部署代码
- 按照团队约定生成模板代码
- 常规代码审查或审计任务
- 项目特定的构建流程

---

## 6. 总结

自定义命令是 Claude Code 的强大功能：

1. **创建简单**：只需在 `.claude/commands/` 目录添加 Markdown 文件
2. **参数灵活**：使用 `$ARGUMENTS` 传递任意参数
3. **效率提升**：将重复任务自动化，减少手动操作
4. **团队协作**：统一工作流程和编码约定

通过合理使用自定义命令，可以显著提升开发效率和代码质量的一致性。

---
