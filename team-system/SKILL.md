---
name: claude-reference-team-system
description: >
  Claude TeamCreate/Task/SendMessage 团队协作系统参考。
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
  owner: "backend-dev"
})
```

### Step 3: 加入团队
```typescript
AgentTool({
  name: "backend-dev",
  team_name: "my-project",
  prompt: "..."
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

## Idle 通知机制

- 队友每轮结束后自动进入 idle 状态
- 消息队列自动投递，不丢失
- 不需要主动 poll 或定时检查

## 协调者模式

```typescript
process.env.CLAUDE_CODE_COORDINATOR_MODE = '1'
// 协调者获取 worker 可用工具列表，写入用户上下文
```

## OpenClaw 参考价值

1. Task 和 Team 绑定（任务归属清晰）
2. Idle 通知（不用 poll）
3. 消息队列 + 自动投递（不丢消息）
4. 按 ID 顺序选任务（早期任务 setup 上下文）
