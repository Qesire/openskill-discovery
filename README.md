# skill-discovery

> Find, validate, security-audit, and install AI agent skills from GitHub across OpenCode, Claude Code, and Codex.
>
> 从 GitHub 发现、校验、安全审计并安装 AI 代理技能，支持 OpenCode、Claude Code、Codex 三大平台。

---

## Features / 功能

- **Search / 搜索** — 4 search methods: GitHub CLI code search, unauthenticated repo search, web search fallback, curated repositories
- **Validate / 校验** — Check frontmatter against the [Agent Skills spec](https://agentskills.io/specification), verify platform paths
- **Security Audit / 安全审计** — 15 security checks: reverse shells, pipe-to-shell, credentials, eval, base64, env dumping, and more
- **Install / 安装** — Per-platform install commands + plugin detection (OpenCode, Claude Code, Codex)
- **Cross-Platform / 跨平台** — Full field support matrix for all three platforms, tool name mappings, porting guidance
- **Review / 审查** — Built-in [skill review guide](references/skill-review-guide.md) for auditing skills systematically

---

## Install / 安装

### OpenCode

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.config/opencode/skills/skill-discovery
```

### Claude Code

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.claude/skills/skill-discovery
```

### Codex

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.agents/skills/skill-discovery
```

---

## Usage / 使用

Once installed, the skill loads automatically when you ask things like:

| English | 中文 |
|---------|------|
| "find me a docker skill" | "帮我找一个 docker 技能" |
| "install the TDD skill from obra/superpowers" | "安装 obra/superpowers 里的 TDD 技能" |
| "audit this skill for security" | "审计这个技能的安全性" |
| "what's wrong with this SKILL.md?" | "这个 SKILL.md 有什么问题？" |

The skill will search, fetch, validate, security-scan, and install the target skill for your platform.

---

## Structure / 结构

```
skill-discovery/
├── SKILL.md                          # Core workflow / 核心工作流 (288 lines)
├── references/
│   ├── platform-compatibility.md     # Field matrix & plugin structures
│   ├── search-methods.md             # Search strategies with auth & rate limits
│   ├── security-scanning.md          # 15 security checks with risk levels
│   └── skill-review-guide.md         # Systematic review methodology
└── .gitignore
```

**SKILL.md** under 500 lines — detailed reference material in `references/`, loaded on demand (progressive disclosure).

---

## Platform Support / 平台支持

| Platform | Status | Docs |
|----------|--------|------|
| OpenCode | ✅ Full | [opencode.ai/docs/skills](https://opencode.ai/docs/skills) |
| Claude Code | ✅ Full | [docs.anthropic.com/claude-code/skills](https://docs.anthropic.com/en/docs/claude-code/skills) |
| Codex | ✅ Full | [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) |

---

## License / 许可证

MIT — see [SKILL.md frontmatter](SKILL.md).