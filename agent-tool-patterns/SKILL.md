---
name: claude-reference-agent-tool-patterns
description: >
  Claude AgentTool 的完整提示词和 Fork/Subagent 决策框架。
---

# AgentTool Patterns — Fork vs Subagent

## 核心决策法则

> **不值得保留中间结果 → Fork**
> **值得保留独立上下文 → Subagent（指定 subagent_type）**

## Fork（廉价上下文共享）

```typescript
// Fork：继承父 Agent 的完整上下文
// 共享 prompt cache，开销极低
AgentTool({
  name: "ship-audit",
  description: "Branch ship-readiness audit",
  prompt: "Audit what's left before this branch can ship..."
})
// 不写 subagent_type = Fork
```

**使用场景**：
- 研究类任务（可并行的问题分解）
- 需要父上下文信息的分析
- 中间结果不需要持久化

**关键规则**：
- 不设置 `model`（不同模型无法复用父 cache）
- 短名字（1-2 个词，lowercase）
- **不偷看输出文件**——信任完成通知
- **不预测结果**——等通知到达再汇报

## Fresh Subagent（独立上下文）

```typescript
// Fresh：指定 subagent_type，从零开始
AgentTool({
  name: "migration-review",
  description: "Independent migration review",
  subagent_type: "code-reviewer",
  prompt: "Review migration 0042_user_schema.sql..."
})
```

**使用场景**：
- 需要独立判断的任务
- 破坏性或高风险操作
- 不依赖父上下文的探索

## 内置 Agent 类型

| 类型 | 工具权限 | 使用场景 |
|------|---------|---------|
| `general-purpose` | 全部 | 默认 |
| `Explore` | 只读（FileRead, Glob, Grep, WebSearch, WebFetch） | 调研、规划 |
| `code-reviewer` | 全工具 | 代码审查 |
| `test-runner` | 全工具 | 测试执行 |

## 后台任务机制

```typescript
// 显式后台运行
AgentTool({
  ...,
  run_in_background: true
})
// → 完成时自动推送通知到 conversation
```

## OpenClaw 参考价值

Fork 机制的核心：共享 cache → 比开新的 subagent 便宜得多，适合"研究完了告诉我"模式。
