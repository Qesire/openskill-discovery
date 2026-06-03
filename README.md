# skill-discovery

[中文](README.zh-CN.md)

Find, validate, security-audit, and install AI agent skills from GitHub across OpenCode, Claude Code, and Codex.

## Features

- **Search** — 4 methods: GitHub CLI code search, unauthenticated repo search, web search fallback, curated repositories
- **Validate** — Check frontmatter against the [Agent Skills spec](https://agentskills.io/specification), verify platform paths
- **Security Audit** — 15 checks: reverse shells, pipe-to-shell, credentials, eval, base64, env dumping, and more
- **Install** — Per-platform install commands with plugin detection (OpenCode, Claude Code, Codex)
- **Cross-Platform** — Full field support matrix for all three platforms, tool name mappings, porting guidance
- **Review** — Built-in [skill review guide](references/skill-review-guide.md) for systematic audits

## Install

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

## Usage

Once installed, the skill loads automatically when you ask things like:

- "find me a docker skill"
- "install the TDD skill from obra/superpowers"
- "audit this skill for security issues"
- "review this SKILL.md against the spec"
- "validate and install a Claude skill for OpenCode"

The skill will search, fetch, validate, security-scan, and install the target skill for your platform.

## Structure

```
skill-discovery/
├── SKILL.md                          # Core workflow (288 lines)
├── references/
│   ├── platform-compatibility.md     # Field matrix, paths, plugin structures
│   ├── search-methods.md             # Search strategies, auth, rate limits
│   ├── security-scanning.md          # 15 security checks with risk levels
│   └── skill-review-guide.md         # Systematic review methodology
└── .gitignore
```

SKILL.md under 500 lines — detailed reference material in `references/`, loaded on demand.

## Platform Support

| Platform | Status | Documentation |
|----------|--------|---------------|
| OpenCode | Full | [opencode.ai/docs/skills](https://opencode.ai/docs/skills) |
| Claude Code | Full | [docs.anthropic.com/claude-code/skills](https://docs.anthropic.com/en/docs/claude-code/skills) |
| Codex | Full | [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) |

## License

MIT — see [SKILL.md frontmatter](SKILL.md).