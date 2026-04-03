---
name: claude-reference-bash-tool-security
description: >
  Claude BashTool 安全设计参考：沙箱、权限、路径验证、破坏性命令警告。
---

# BashTool Security Design

## 安全层次

### 1. 路径验证

**禁止的路径模式**：
- 系统目录（`/System`、`/usr`、`/bin`、`/sbin` 等）
- 敏感目录（`~/.ssh`、`~/.aws`、`~/.gnupg`）
- 符号链接穿越（`../` 穿越检查）

```typescript
// 规范化路径（解析 .. 和符号链接）
const normalized = realpathSync(path)
// 黑名单检查
return !BLACKLISTED_PATHS.some(p => normalized.startsWith(p))
```

### 2. 沙箱决策

```typescript
function shouldUseSandbox(command: string, mode: PermissionMode): boolean {
  if (mode === 'read-only') return true
  if (SAFE_COMMANDS.has(command)) return false
  if (DESTRUCTIVE_COMMANDS.has(command)) return true
  return true
}
```

### 3. 权限模式

| 模式 | 允许的操作 |
|------|---------|
| `read-only` | 文件读取、搜索、git read |
| `limited` | + 用户目录下的写操作 |
| `full` | + 系统命令（需要确认） |

### 4. 破坏性命令警告

检测并警告：`rm -rf`、`dd`、`mkfs`、`fdisk`、`kill -9`、`chmod -R 777` 等。

## 权限检查流程

```
命令输入 → 路径验证 → 命令类型检测 → 沙箱决策 → 破坏性检测 → 执行
```

## OpenClaw 参考价值

1. `realpathSync` 解析符号链接后再验证（防止穿越）
2. 命令级沙箱开关（不是进程级）
3. 破坏性命令黑名单扩展
4. 权限模式分级（read-only agent 用 read-only bash）
