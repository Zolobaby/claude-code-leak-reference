---
name: claude-reference
description: >
  Claude CLI 架构参考：基于 Claude Code 设计模式的多 Agent 系统参考。
  涵盖工具系统、子 Agent 模式、团队协作、记忆管理、Bash 安全等核心模块。
---

# Claude Architecture Reference

> 基于 Claude Code 设计模式的多 Agent 系统架构参考

---

## 快速定位

| 需求 | 文件 |
|------|------|
| 工具如何定义 | `src/tools.ts` + `src/Tool.ts` |
| Bash 安全/权限 | `src/tools/BashTool/` |
| 子 Agent 启动 | `src/tools/AgentTool/` |
| 团队协作 | `src/tools/TeamCreateTool/` + `src/coordinator/` |
| Memory 入口 | `src/memdir/` |
| 多 Agent 协调 | `src/coordinator/coordinatorMode.ts` |
| 系统提示词 | `src/constants/system.ts` + `src/utils/systemPrompt.ts` |
| MCP 集成 | `src/services/mcp/` + `src/tools/MCPTool/` |
| 内置 Skills | `src/skills/bundled/` |
| Feature Flag | `src/services/analytics/growthbook.js` |

---

## 核心子模块

### BashTool — 安全 Bash 执行
**路径**：`src/tools/BashTool/`
**关键文件**：
- `bashSecurity.ts` — 安全策略
- `bashPermissions.ts` — 权限模型
- `pathValidation.ts` — 路径验证
- `shouldUseSandbox.ts` — 沙箱决策

**参考价值**：Bash 执行的安全边界设计（路径验证、破坏性命令警告、沙箱策略）

### AgentTool — 子 Agent 启动
**路径**：`src/tools/AgentTool/`
**关键文件**：
- `prompt.ts` — Agent 工具的完整提示词（含 Fork vs Fresh 决策）
- `forkSubagent.ts` — Fork 机制实现
- `runAgent.ts` — Agent 运行逻辑
- `builtInAgents.ts` — 内置 Agent 类型定义

**参考价值**：Fork（廉价上下文共享）vs Subagent（独立上下文）的决策框架

### Team System — 团队协作
**路径**：`src/tools/TeamCreateTool/` + `src/coordinator/`
**关键文件**：
- `prompt.ts`（TeamCreateTool） — 团队工作流完整说明
- `coordinatorMode.ts` — 协调者模式实现
- `constants.ts` — 团队常量定义

**参考价值**：Task + Team 绑定、多成员通信、idle 通知机制

### Memory System — 记忆入口
**路径**：`src/memdir/`
**关键文件**：
- `memdir.ts` — 入口管理（MEMORY.md，200行/25KB 上限）
- `memoryTypes.ts` — 分类体系
- `findRelevantMemories.ts` — 相关性搜索
- `teamMemPaths.ts` — 团队记忆共享

**参考价值**：分层记忆入口设计

---

## 工具注册机制

```typescript
// 条件加载示例
const REPLTool = process.env.USER_TYPE === 'ant' ? require('./tools/REPLTool/') : null
const SleepTool = feature('PROACTIVE') || feature('KAIROS') ? require('./tools/SleepTool/') : null
```

Feature Flag 控制哪些工具被加载，不是全量打包。

---

## Agent 类型定义

```typescript
type AgentDefinition = {
  agentType: string        // e.g., "code-reviewer"
  whenToUse: string       // 使用场景描述
  tools?: string[]         // allowlist
  disallowedTools?: string[] // denylist
}
```

内置类型：`general-purpose`（全工具）、`Explore`（只读）、`code-reviewer`、`test-runner`

---

## Fork 决策法则

> "不值得保留中间结果 → Fork，值得保留独立上下文 → Subagent"

---

## 客户端证明机制

```typescript
// Bun HTTP 层在请求体中寻找占位符，替换为证明 token
const cch = feature('NATIVE_CLIENT_ATTESTATION') ? 'cch=00000;' : ''
// 最终头部：x-anthropic-billing-header: cc_version=...; cc_entrypoint=...; cch={真实token}
```

---

## 相关子 SKILL

- `claude-reference/bash-tool-security` — BashTool 安全设计详解
- `claude-reference/agent-tool-patterns` — Agent/Fork 模式详解
- `claude-reference/team-system` — 团队协作系统详解

---

*最后更新：2026-04-03*
