---
name: claude-code-leak-memory-system
description: >
  Claude Code memdir 记忆系统：MEMORY.md 入口、200行限制、相关性搜索、团队记忆。
  源码路径：src/memdir/
---

# Memory System — memdir

## 核心设计：MEMORY.md 入口

Claude Code 只认一个文件作为记忆入口：

```
~/.claude/memory/MEMORY.md
```

**限制**：
- 最多 200 行（MAX_ENTRYPOINT_LINES = 200）
- 最多 25KB（MAX_ENTRYPOINT_BYTES = 25,000）

```typescript
// src/memdir/memdir.ts
export const MAX_ENTRYPOINT_LINES = 200
export const MAX_ENTRYPOINT_BYTES = 25_000
```

**截断逻辑**：
1. 先按行截断（200行自然边界）
2. 再按字节截断（25KB 自然边界）

## 记忆分类（memoryTypes.ts）

```typescript
// 分类标签
TRUSTING_RECALL      // "可信任召回"——直接作为上下文
TYPES_SECTION_INDIVIDUAL  // "个人类型"——按类型组织
WHAT_NOT_TO_SAVE     // "不保存"——排除项
WHEN_TO_ACCESS       // "何时访问"——触发条件
MEMORY_FRONTMATTER_EXAMPLE  // frontmatter 示例
```

## 相关性搜索（findRelevantMemories.ts）

当需要检索记忆时：
1. 读取 MEMORY.md 入口
2. 提取 frontmatter 中的元信息
3. 按相关性匹配（不是全文搜索）

## 团队记忆（teamMemPaths.ts）

TEAMMEM feature flag 开启后：
- 团队共享记忆：`~/.claude/teams/{team-name}/memory/`
- 个人记忆：`~/.claude/memory/`
- 队友之间不共享个人记忆

## 与 OpenClaw 的差异

| 维度 | Claude Code | OpenClaw |
|------|------------|---------|
| 入口文件 | MEMORY.md（单一） | 多层（MEMORY.md + daily + L-Agent） |
| 大小限制 | 200行 / 25KB | 无硬限制 |
| 搜索方式 | 相关性匹配 | Native Vector |
| 团队记忆 | 独立目录隔离 | 共享桥接 |

## OpenClaw 参考价值

**借鉴思路**：
1. MEMORY.md 的 frontmatter 设计（分类标签）
2. 200行限制 → 强制记忆精炼（防止记忆膨胀）
3. WHAT_NOT_TO_SAVE 排除清单（隐私/临时数据不入记忆）

---

*源码：src/memdir/memdir.ts + memoryTypes.ts + findRelevantMemories.ts + teamMemPaths.ts*
