# skill-discovery

[English](README.md)

从 GitHub 发现、校验、安全审计并安装 AI 代理技能，支持 OpenCode、Claude Code、Codex 三大平台。

## 功能

- **搜索** — 4 种搜索方法：GitHub CLI 代码搜索、无需认证的仓库搜索、网页搜索回退、精选仓库列表
- **校验** — 对照 [Agent Skills 规范](https://agentskills.io/specification) 检查 frontmatter，验证平台路径
- **安全审计** — 15 项安全检查：反弹 shell、管道执行 shell、凭证泄漏、eval、base64、环境变量导出等
- **安装** — 各平台安装命令，含插件检测（OpenCode、Claude Code、Codex）
- **跨平台** — 三平台完整字段支持矩阵，工具名称对照，迁移指南
- **审查** — 内置 [技能审查指南](references/skill-review-guide.md)，系统化审计方法

## 安装

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

## 使用

安装后，当你提出以下问题时 skill 会自动加载：

- "帮我找一个 docker 技能"
- "安装 obra/superpowers 里的 TDD 技能"
- "审计这个技能的安全性"
- "对照规范审查这个 SKILL.md"
- "验证一个 Claude 技能并安装到 OpenCode"

skill 会自动搜索、拉取、校验、安全扫描，并按你的平台完成安装。

## 结构

```
skill-discovery/
├── SKILL.md                          # 核心工作流（288 行）
├── references/
│   ├── platform-compatibility.md     # 字段矩阵、路径、插件结构
│   ├── search-methods.md             # 搜索策略、认证、速率限制
│   ├── security-scanning.md          # 15 项安全检查含风险评级
│   └── skill-review-guide.md         # 系统化审查方法论
└── .gitignore
```

SKILL.md 控制在 500 行以内 — 详细参考资料放在 `references/` 中，按需加载。

## 平台支持

| 平台 | 状态 | 文档 |
|------|------|------|
| OpenCode | 完整支持 | [opencode.ai/docs/skills](https://opencode.ai/docs/skills) |
| Claude Code | 完整支持 | [docs.anthropic.com/claude-code/skills](https://docs.anthropic.com/en/docs/claude-code/skills) |
| Codex | 完整支持 | [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) |

## 许可证

MIT — 详见 [SKILL.md frontmatter](SKILL.md)。