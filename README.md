# Claude Code Leaked Source — Architecture Reference

> Extracted from the **Claude Code CLI leaked source** (March 31, 2026, via `.map` file on npm registry).
> 1,902 files, 512K+ lines of TypeScript, Bun runtime, React+Ink UI.

---

## What Is This?

This is a **structured reference** for AI agents (especially OpenClaw) to learn from Claude Code's architecture — without needing to read the entire 512K-line codebase.

Each submodule documents a key architectural pattern with:
- File paths to the original source
- Key code patterns and decisions
- Differences from OpenClaw's approach
- Specific reference value for implementation

---

## Modules

| Module | Topic | Key Insight |
|--------|-------|-------------|
| `agent-tool-patterns/` | Agent/Fork decision framework | Fork = cheap context sharing; Subagent = independent context |
| `bash-tool-security/` | BashTool security design | Path validation, sandboxing, destructive command detection |
| `memory-system/` | memdir memory system | MEMORY.md entry, 200-line cap, classification tags |
| `team-system/` | Multi-agent teamwork | Task+Team binding, idle notification, message queue |

---

## Source

**Original source**: `src/` directory of the leaked Claude Code TypeScript source

**Key files**:
- `src/tools.ts` — Tool registry
- `src/Tool.ts` — Tool type definitions
- `src/tools/AgentTool/prompt.ts` — Complete Agent tool prompt (Fork vs Subagent decisions)
- `src/tools/TeamCreateTool/prompt.ts` — Complete team collaboration workflow
- `src/memdir/memdir.ts` — Memory entry management
- `src/tools/BashTool/bashSecurity.ts` — Bash security design

---

## OpenClaw Usage

```
openclaw skills install Zolobaby/claude-code-leak-reference
```

Then reference the skill when solving related problems:
- Bash security → read `bash-tool-security`
- Agent patterns → read `agent-tool-patterns`
- Team coordination → read `team-system`
- Memory design → read `memory-system`

---

## License

This is a reference extraction from leaked source code published for educational purposes. Respect Anthropic's intellectual property.
