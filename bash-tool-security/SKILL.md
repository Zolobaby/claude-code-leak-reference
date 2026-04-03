---
name: claude-code-leak-bash-tool-security
description: >
  Claude Code BashTool 的安全设计：沙箱、权限、路径验证、破坏性命令警告。
  源码路径：src/tools/BashTool/
---

# BashTool Security Design

## 安全层次

### 1. 路径验证（pathValidation.ts）

**禁止的路径模式**：
- 系统目录（`/System`、`/usr`、`/bin`、`/sbin` 等 macOS 系统路径）
- 敏感目录（`~/.ssh`、`~/.aws`、`~/.gnupg`）
- 符号链接穿越（`../` 穿越检查）

**实现模式**：
```typescript
function isPathSafe(path: string): boolean {
  // 规范化路径（解析 .. 和符号链接）
  const normalized = realpathSync(path)
  // 黑名单检查
  return !BLACKLISTED_PATHS.some(p => normalized.startsWith(p))
}
```

### 2. 沙箱决策（shouldUseSandbox.ts）

```typescript
function shouldUseSandbox(command: string, mode: PermissionMode): boolean {
  // read-only 模式 → 强制沙箱
  if (mode === 'read-only') return true
  // 白名单命令 → 不沙箱
  if (SAFE_COMMANDS.has(command)) return false
  // 破坏性命令 → 强制沙箱
  if (DESTRUCTIVE_COMMANDS.has(command)) return true
  // 默认 → 沙箱
  return true
}
```

### 3. 权限模式（bashPermissions.ts）

| 模式 | 允许的操作 |
|------|---------|
| `read-only` | 文件读取、搜索、git read 操作 |
| `limited` | + 用户目录下的写操作 |
| `full` | + 系统命令（需要确认） |

### 4. 破坏性命令警告（destructiveCommandWarning.ts）

检测并警告：
- `rm -rf` / `rm -r /`
- `dd`（磁盘写入）
- `mkfs` / `fdisk`（磁盘操作）
- `kill -9`（强制终止）
- `chmod -R 777`

**警告格式**：
```
⚠️  This command will permanently delete files:
  rm -rf ./node_modules
  
  Are you sure? [y/N]
```

### 5. Bash 安全策略（bashSecurity.ts）

- **无用的 sudo 默认拒绝**：需要显式启用
- **后台进程监控**：防止 `nohup` 逃避
- **管道安全**：检查 stderr 是否被吞噬（`2>/dev/null` 高风险模式）

## 权限检查流程

```
命令输入
  ↓
路径验证（isPathSafe）
  ↓ 通过
命令类型检测（read-only / limited / full）
  ↓
沙箱决策（shouldUseSandbox）
  ↓
破坏性命令检测
  ↓ 无破坏性 或 用户确认
执行
```

## 与 OpenClaw 的差异

| 维度 | Claude Code | OpenClaw |
|------|------------|---------|
| 路径验证 | 完整 realpath 解析 | 目前无 |
| 沙箱 | 可配置 per-command | exec 安全模式 |
| 权限分级 | 3 档（read-only/limited/full） | 无分级 |
| 破坏性警告 | 内置 50+ 模式检测 | 基础检测 |

## OpenClaw 参考价值

**可以借鉴**：
1. `realpathSync` 解析符号链接后再验证（防止穿越）
2. 命令级沙箱开关（不是进程级）
3. 破坏性命令黑名单（扩展 `rm -rf` 等模式）
4. 权限模式分级（read-only agent 用 read-only bash）

---

*源码：src/tools/BashTool/bashSecurity.ts + pathValidation.ts + bashPermissions.ts + destructiveCommandWarning.ts*
