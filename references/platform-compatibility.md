# Platform Compatibility Reference

## Field Support Matrix

### Agent Skills Standard Fields

Per the [Agent Skills specification](https://agentskills.io/specification):

| Field | Claude Code | OpenCode | Codex | Notes |
|-------|------------|----------|-------|-------|
| `name` | ✓ required | ✓ required | ✓ required | Must match directory name. Claude Code: defaults to dir name if omitted. |
| `description` | ✓ required | ✓ required | ✓ required | 1-1024 chars. Describe triggers, not workflow summary. |
| `license` | ✓ optional | ✓ optional | ✓ optional | Short license name or bundled file reference |
| `compatibility` | ✓ optional | ✓ optional | ✓ optional | Max 500 chars. Environment requirements |
| `metadata` | ✓ optional | ✓ optional | ✓ optional | Arbitrary key-value map |
| `allowed-tools` | ✓ supported | ✗ ignored | ✗ ignored | Experimental. Space-separated tool list |

### Claude Code Extended Fields

These fields extend the standard for Claude Code-specific features. Not recognized by OpenCode or Codex.

| Field | Description | Claude | OpenCode | Codex |
|-------|-------------|--------|----------|-------|
| `name` | Display label (not invocation command). Command name = directory name. | ✓ | ✓ | ✓ |
| `description` | Truncated at 1,536 chars in skill listing | ✓ | ✓ | ✓ |
| `when_to_use` | Extra trigger context appended to description. Counts toward 1,536-char cap. | ✓ | ✗ | ✗ |
| `argument-hint` | Autocomplete hint, e.g. `[issue-number]` | ✓ | ✗ | ✗ |
| `arguments` | Named positional args for `$name` substitution | ✓ | ✗ | ✗ |
| `disable-model-invocation` | `true` = only user can invoke (`/name`). Blocks subagent preloading. | ✓ | ✗ | ✗ |
| `user-invocable` | `false` = hidden from `/` menu. Claude can still invoke. | ✓ | ✗ | ✗ |
| `allowed-tools` | Pre-approved tools while skill active. Space/comma or YAML list. | ✓ | ✗ | ✗ |
| `disallowed-tools` | Tools removed from Claude's pool while active. Clears on next message. | ✓ | ✗ | ✗ |
| `model` | Model override while skill active. Resets on next prompt. | ✓ | ✗ | ✗ |
| `effort` | Effort level: `low`, `medium`, `high`, `xhigh`, `max` | ✓ | ✗ | ✗ |
| `context` | Set to `fork` to run in isolated subagent context | ✓ | ✗ | ✗ |
| `agent` | Subagent type when `context: fork` (e.g. `Explore`, `Plan`) | ✓ | ✗ | ✗ |
| `hooks` | Hook event handlers scoped to skill lifecycle. MCP servers configured separately via `.mcp.json` or plugin manifest. | ✓ | ✗ | ✗ |
| `paths` | Glob patterns limiting when skill activates. Comma-separated or YAML list. | ✓ | ✗ | ✗ |
| `shell` | `bash` (default) or `powershell` for `` !`cmd` `` blocks | ✓ | ✗ | ✗ |
| `maxTurns` | Maximum turns when running as subagent | ✓ | ✗ | ✗ |

### Skill Placement Paths

| Platform | Global (user) | Project-local | Additional |
|----------|---------------|---------------|------------|
| **OpenCode** | `~/.config/opencode/skills/<name>/SKILL.md` | `.opencode/skills/<name>/SKILL.md` | Also discovers `.claude/skills/` and `.agents/skills/` for cross-compat |
| **OpenCode (nested)** | `~/.config/opencode/skills/<group>/<name>/SKILL.md` | `.opencode/skills/<group>/<name>/SKILL.md` | Discovery pattern `skills/**/SKILL.md`. Names currently flat; PR #27981 adds `group:name` prefixing. |
| **Claude Code** | `~/.claude/skills/<name>/SKILL.md` | `.claude/skills/<name>/SKILL.md` | Also `--add-dir` and enterprise managed paths |
| **Codex** | `~/.agents/skills/<name>/SKILL.md` | `.agents/skills/<name>/SKILL.md` | Also `/etc/codex/skills` (machine-wide). Invoke: `$skill-name` |
| **OpenCode Agents** | `~/.config/opencode/agents/<name>.md` | `.opencode/agents/<name>.md` | |
| **Claude Code Agents** | `~/.claude/agents/<name>.md` | `.claude/agents/<name>.md` | |
| **Codex Agents** | `~/.agents/agents/<name>.md` | `.agents/agents/<name>.md` | |

### Tool Name Equivalents

| Claude Code | OpenCode | Codex | Notes |
|------------|----------|-------|-------|
| Read, Bash, Grep, Glob, Edit, Write, Task, TodoWrite, Skill | Same name ✓ | Same name ✓ | Standard tool set |
| WebSearch | — | — | Claude Code only: web search via Anthropic backend |
| WebFetch | WebFetch | WebFetch | URL content fetching (all platforms) |
| Question | Question | AskUserQuestion | User prompting |
| LSP | LSP | — | Language Server Protocol support |
| NotebookRead, NotebookEdit | — | — | Claude Code only |

### Claude-Specific Warnings (when porting to OpenCode/Codex)

| Claude Field | Porting Advice |
|--------------|----------------|
| `hooks` | Use separate MCP config (`opencode.json` → `mcpServers`) |
| `disallowedTools` | Convert to platform permission rules |
| `permissionMode` | Convert to platform permission rules |
| `model` | Model selected by platform config (OpenCode: provider config) |
| `context: fork` | Use OpenCode subagents or `Task` tool delegation |
| `argument-hint`, `arguments` | Not supported; rewrite as inline instructions |

### Plugin Structures

| Platform | Plugin Indicator | How to Install |
|----------|-----------------|----------------|
| **OpenCode** | `.opencode/INSTALL.md` or `package.json` with `@opencode-ai/plugin` | Add `"plugin": ["<name>@git+<url>"]` to `opencode.json` |
| **Claude Code** | `.claude-plugin/plugin.json` | Clone to plugins dir, or use `/plugin install` |
| **Claude Code (skills-dir)** | `.claude-plugin/plugin.json` inside a `~/.claude/skills/<name>/` folder | Auto-loads as `<name>@skills-dir` |
| **Codex** | Plugin package structure | Install via Codex plugin system |