---
title: 🤖 AI 编程心得笔记 - Claude Code 与 OpenCode 实战指南
date: 2026-02-06 10:00:00
updated: 2026-02-06 10:00:00
description: '深入解析 Claude Code 和 OpenCode 的使用技巧，包括 Agents、Hooks、Skills 等核心概念，以及常用命令和最佳实践，助你提升 AI 辅助编程效率'
keywords: 'AI编程,ClaudeCode,OpenCode,AI辅助编程,编程效率,代码审查,TDD,开发工具'
tags: 
  - AI编程
  - ClaudeCode
  - OpenCode
  - 效率工具
  - 开发工具
  - 最佳实践
categories:
  - 技术笔记
  - AI工具
author: Yi Chen
toc: true
comments: true
---

> 💡 本文记录使用 Claude Code 和 OpenCode 进行 AI 辅助编程的实战经验与心得体会。

## 🎯 写在前面

随着 AI 编程助手的快速发展，Claude Code 和 OpenCode 已经成为我日常开发的得力助手。这篇文章总结了使用过程中的核心概念、常用命令和最佳实践。

---

## 🧠 四大核心概念

理解这四个概念，是高效使用 AI 编程助手的关键：

| 概念 | 一句话理解 | 使用场景 |
|------|-----------|----------|
| **Agents** | 专家代理，决定"谁来执行" | 复杂任务 → `/plan`, 代码审查 → `/code-review`, 安全检查 → `/security` |
| **Hooks** | 自动触发器，决定"何时做什么" | 自动格式化、类型检查、保存上下文、提醒警告 |
| **Tools** | 工具函数，决定"用什么执行" | `Read`, `Edit`, `Bash`, `run-tests`, `security-audit` |
| **Skills** | 知识手册，决定"知道怎么做" | Django/Spring Boot/Go 开发指导，TDD/安全最佳实践 |

---

## 🚀 常用命令速查

### 规划与开发

```bash
/plan "功能描述"              # 创建实现计划（使用 planner agent）
/tdd "功能描述"               # TDD 工作流（先写测试）
/code-review                 # 代码审查
/security                    # 安全审查
/build-fix                   # 修复构建错误
```

### Go 开发专用

```bash
/go-review                  # Go 代码审查
/go-test                    # Go TDD
/go-build                   # Go 构建修复
```

### 测试与验证

```bash
/e2e                        # E2E 测试
/test-coverage             # 测试覆盖率分析
/verify                    # 验证循环
```

### 多智能体并行（Claude Code）

```bash
/multi-workflow "任务"      # 完整多智能体工作流
/multi-backend              # 后端并行开发
/multi-frontend             # 前端并行开发
/orchestrate               # 编排多个 agent
```

---

## ⚡ Hooks 自动触发清单

Hooks 是 AI 编程助手最强大的特性之一，能够在特定时机自动执行操作：

### PreToolUse（执行前）
- ✅ 阻止在 tmux 外运行 dev server
- ✅ 提醒使用 tmux（长时命令）
- ✅ git push 前提醒检查
- ✅ 阻止创建无关 .md 文件

### PostToolUse（执行后）
- ✅ **自动格式化** JS/TS（Prettier）
- ✅ **类型检查** TypeScript
- ✅ **警告** console.log
- ✅ PR 创建后显示 URL

### Stop（每次响应后）
- ✅ 检查 console.log
- ✅ 建议手动压缩上下文

### SessionStart/End（会话生命周期）
- ✅ 加载/保存上下文
- ✅ 评估可提取的模式

---

## 📚 推荐的 Skills（必用）

### 后端开发
- `django-patterns` - Django 架构模式
- `springboot-patterns` - Spring Boot 模式
- `golang-patterns` - Go 模式
- `postgres-patterns` - PostgreSQL 优化

### 工程实践
- `tdd-workflow` - TDD 工作流
- `security-review` - 安全审查
- `backend-patterns` - 后端通用模式
- `verification-loop` - 验证循环

### 高级功能
- `continuous-learning` - 持续学习（自动提取模式）
- `iterative-retrieval` - 迭代检索

---

## 🔧 OpenCode 特有功能

### 自定义 Tools

```typescript
run-tests({ coverage: true })      // 运行测试
check-coverage({ threshold: 80 })  // 检查覆盖率
security-audit({ severity: "high" }) // 安全扫描
```

### Plugin 事件（20+）

- `file.edited` - 文件编辑后
- `tool.execute.before/after` - 工具执行前后
- `session.idle` - 会话空闲
- `session.created/deleted` - 会话创建/结束

---

## 💡 我的最佳实践

### 1. 新项目启动流程

```bash
/plan "实现用户认证系统"     # 先规划
# 确认计划后
/tdd "实现登录功能"          # TDD 开发
/code-review                 # 审查代码
/security                    # 安全检查
```

### 2. 日常开发流程

```bash
# 修改代码（自动触发 hooks 格式化）
# ...
/code-review                 # 提交前审查
/test-coverage              # 检查覆盖率
/e2e                        # 运行 E2E 测试
```

### 3. 工具选择建议

- **需要完整功能** → Claude Code（33命令 + Hooks + Rules）
- **需要自定义工具** → OpenCode（run-tests, security-audit）
- **生产级项目** → 两者结合使用

---

## ⚠️ 注意事项

1. **Hooks 100% 触发**，Skills 是概率性的（50-80%）
2. **Skills 通用**，可在 Claude Code 和 OpenCode 间共享
3. **Agents 格式不同**，不能混用
4. **Commands 格式不同**，但功能类似
5. 使用 `/multi-*` 系列命令时，确保理解**并行化开销**
6. 定期运行 `/learn` 提取模式，积累团队知识

---

## 🔗 快速参考

配置位置：
- **Claude Code**: `~/.claude/` (agents, commands, hooks, rules, skills)
- **OpenCode**: `~/.opencode/ecc/` (opencode.json, prompts, tools, plugins)

文件结构：
- **Agents**: `~/.claude/agents/*.md` (14个)
- **Commands**: `~/.claude/commands/*.md` (33个)
- **Hooks**: `~/.claude/hooks/hooks.json`
- **Skills**: `~/.claude/skills/*/` (31个)
- **Rules**: `~/.claude/rules/{common,python,golang,typescript}/`

---

## 🎉 结语

AI 编程助手正在改变我们的开发方式。通过合理使用 Agents、Hooks、Tools 和 Skills，可以大幅提升开发效率和代码质量。希望这篇笔记对你有所帮助！

> 💬 **持续学习，不断进化** - AI 时代，保持学习的心态最重要。

---

*创建于 2026-02-06 | AI 编程实践笔记*
