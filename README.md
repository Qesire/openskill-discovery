# skill-discovery

[中文](README.zh-CN.md)

Find, validate, security-audit, and install AI agent skills from GitHub across OpenCode and Claude Code.

## Features

- **Search** — Priority-ordered search: curated repos → GitHub CLI code search → unauthenticated repo search → web search fallback
- **Validate** — Check frontmatter against the [Agent Skills spec](https://agentskills.io/specification), verify platform paths
- **Security Audit** — 15 checks: reverse shells, pipe-to-shell, credentials, eval, base64, env dumping, and more
- **Install** — Per-platform install commands with plugin detection (OpenCode, Claude Code)
- **Cross-Platform** — Full field support matrix, tool name mappings, porting guidance
- **Review** — Built-in [skill review guide](references/skill-review-guide.md) for systematic audits
- **Edge Cases** — 7 boundary scenarios with trigger→protocol in [references/edge-cases.md](references/edge-cases.md)
- **Templates** — Structured output templates for security reports and validation reports

## Install

### OpenCode

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.config/opencode/skills/skill-discovery
```

### Claude Code

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.claude/skills/skill-discovery
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
├── SKILL.md                          # Core workflow: dispatch table, hard rules, summaries
├── templates/
│   ├── tpl-security-report.md        # Structured security scan output
│   └── tpl-validation-report.md      # Structured validation checklist
├── references/
│   ├── search-methods.md             # Phase 0-5 ordered search workflow
│   ├── platform-compatibility.md     # Field matrix, paths, plugins, tool mappings
│   ├── security-scanning.md          # 15 security checks with risk levels
│   ├── skill-review-guide.md         # Systematic review methodology
│   └── edge-cases.md                 # 7 boundary scenarios with protocols
├── README.md
├── README.zh-CN.md
└── .gitignore
```

SKILL.md under 250 lines — detailed workflows and edge cases in `references/`, loaded on demand.

## Design Patterns

This skill incorporates structural patterns from the [note-merge](https://github.com/Qesire/note-merge) reference design:

| Pattern | Where |
|---------|-------|
| **Dispatch Table** | SKILL.md top — maps user intents to reference files |
| **Hard Rules** | Section 5 — 10 non-negotiable invariants |
| **Phase Structure** | search-methods.md — Phase 0→5 with exit conditions |
| **Edge Cases File** | references/edge-cases.md — 7 trigger→protocol scenarios |
| **Dependency Declaration** | Section 0 — tool table with fallbacks |
| **Output Templates** | templates/ — security report, validation report |
| **Platform Field Notes** | Frontmatter comments — which fields are platform-specific |
| **Recovery Protocol** | Hard Rule #6 — report install path + uninstall command |

## Platform Support

| Platform | Status | Documentation |
|----------|--------|---------------|
| OpenCode | Full | [opencode.ai/docs/skills](https://opencode.ai/docs/skills) |
| Claude Code | Full | [docs.anthropic.com/claude-code/skills](https://docs.anthropic.com/en/docs/claude-code/skills) |

## License

MIT — see [SKILL.md frontmatter](SKILL.md).