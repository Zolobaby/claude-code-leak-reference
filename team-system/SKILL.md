---
name: claude-code-leak-team-system
description: >
  Claude Code TeamCreate/Task/SendMessage 团队协作系统。
  源码路径：src/tools/TeamCreateTool/ + src/coordinator/
---

# Team System — 多 Agent 协作

## 核心概念

```
Team = TaskList（1:1 对应）
团队创建 → 自动创建对应任务列表
```

## 团队工作流

### Step 1: 创建团队
```typescript
TeamCreateTool({
  team_name: "my-project",
  description: "Working on feature X"
})
```
→ 创建：`~/.claude/teams/{team-name}/config.json`
→ 创建：`~/.claude/tasks/{team-name}/`（任务列表）

### Step 2: 创建任务
```typescript
TaskCreate({
  summary: "Implement login API",
  owner: "backend-dev"  // 分配给队友
})
// → 自动归属当前团队
```

### Step 3: 加入团队
```typescript
AgentTool({
  name: "backend-dev",
  team_name: "my-project",
  prompt: "You are the backend developer..."
})
```

### Step 4: 分配任务
```typescript
TaskUpdate({
  task_id: "xxx",
  owner: "backend-dev"
})
```

### Step 5: 协作通信
```typescript
SendMessageTool({
  to: "frontend-dev",
  message: "API spec is ready at /docs/api.md"
})
```

### Step 6: 优雅关闭
```typescript
SendMessageTool({
  to: "backend-dev",
  message: { type: "shutdown_request" }
})
```

## Idle 通知机制

**关键设计**：
- 队友每轮结束后自动进入 idle 状态
- 消息队列自动投递，不丢失
- 不需要主动 poll 或定时检查

```typescript
// 队友发来的消息 → 自动作为新 turn 到达
// 不需要任何主动获取操作
```

## 协调者模式（Coordinator Mode）

```typescript
// 开启协调者模式
process.env.CLAUDE_CODE_COORDINATOR_MODE = '1'

// 协调者获取的上下文
getCoordinatorUserContext(mcpClients, scratchpadDir)
// → 返回 worker 可用工具列表
// → 写入协调者的用户上下文
```

## 发现队友

读取团队配置文件：
```
~/.claude/teams/{team-name}/config.json
```

内容：
```json
{
  "members": [
    { "name": "backend-dev", "agentId": "...", "agentType": "general-purpose" }
  ]
}
```

**通信时使用 name，不使用 agentId**

## 任务列表协作规则

1. 每完成一个任务 → 检查 TaskList 找下一个
2. 按 ID 顺序优先（早期任务为后期任务设置上下文）
3. 发现阻塞任务 → 通知 team lead
4. 所有可用任务都被阻塞 → 帮助解决阻塞

## 与 OpenClaw 的差异

| 维度 | Claude Code | OpenClaw |
|------|------------|---------|
| 团队创建 | 内置 TeamCreateTool | 外部编排 |
| 任务分配 | 内置 Task 系统 | 飞书任务 |
| 消息投递 | 自动推送 | 需要轮询 |
| 成员发现 | 读配置文件 | 外部管理 |

## OpenClaw 参考价值

**可以借鉴的机制**：
1. Task 和 Team 绑定（任务归属清晰）
2. Idle 通知（不用 poll）
3. 消息队列 + 自动投递（不丢消息）
4. 按 ID 顺序选任务（早期任务 setup 上下文）

---

*源码：src/tools/TeamCreateTool/prompt.ts + src/coordinator/coordinatorMode.ts*
