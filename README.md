# Claude Architecture Reference

> 基于 Claude Code 设计模式的多 Agent 系统架构参考

---

## 什么是这个？

这是一个面向 AI Agent 的**架构参考库**，提取自 Claude Code 的核心设计模式。

每个子模块包含：
- 原始设计文件路径
- 关键代码模式和决策逻辑
- 与 OpenClaw 的实现差异
- 对落地的具体参考价值

---

## 模块列表

| 模块 | 主题 | 核心价值 |
|------|------|---------|
| `agent-tool-patterns/` | Agent/Fork 决策框架 | Fork = 廉价上下文共享；Subagent = 独立上下文 |
| `bash-tool-security/` | BashTool 安全设计 | 路径验证、沙箱、破坏性命令检测 |
| `memory-system/` | memdir 记忆系统 | MEMORY.md 入口、200行限制、分类标签 |
| `team-system/` | 多 Agent 团队协作 | Task+Team 绑定、idle 通知、消息队列 |

---

## OpenClaw 安装

```bash
openclaw skills install Zolobaby/claude-reference
```

或者安装特定模块：
```bash
openclaw skills install Zolobaby/claude-reference/agent-tool-patterns
```

---

## 来源说明

本参考库基于 Claude Code 公开的设计模式和架构思路整理，供 AI Agent 学习参考。
