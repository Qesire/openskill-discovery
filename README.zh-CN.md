# skill-discovery

[English](README.md)

从 GitHub 发现、校验、安全审计并安装 AI 代理技能，支持 OpenCode 和 Claude Code。

## 功能

- **搜索** — 按优先级搜索：精选仓库 → GitHub CLI 代码搜索 → 无需认证的仓库搜索 → 网页搜索回退
- **校验** — 对照 [Agent Skills 规范](https://agentskills.io/specification) 检查 frontmatter，验证平台路径
- **安全审计** — 15 项安全检查：反弹 shell、管道执行 shell、凭证泄漏、eval、base64、环境变量导出等
- **安装** — 各平台安装命令，含插件检测（OpenCode、Claude Code）
- **跨平台** — 完整字段支持矩阵，工具名称对照，迁移指南
- **审查** — 内置 [技能审查指南](references/skill-review-guide.md)，系统化审计方法
- **边界情况** — 7 种边界场景，含触发条件与处理协议，见 [references/edge-cases.md](references/edge-cases.md)
- **模板** — 安全报告和校验报告的结构化输出模板

## 安装

### OpenCode

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.config/opencode/skills/skill-discovery
```

### Claude Code

```bash
git clone https://github.com/Qesire/openskill-discovery.git ~/.claude/skills/skill-discovery
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
├── SKILL.md                          # 核心工作流：调度表、硬性规则、摘要
├── templates/
│   ├── tpl-security-report.md        # 结构化安全扫描输出
│   └── tpl-validation-report.md      # 结构化校验检查清单
├── references/
│   ├── search-methods.md             # 阶段 0-5 顺序搜索工作流
│   ├── platform-compatibility.md     # 字段矩阵、路径、插件、工具映射
│   ├── security-scanning.md          # 15 项安全检查含风险评级
│   ├── skill-review-guide.md         # 系统化审查方法论
│   └── edge-cases.md                 # 7 种边界场景含处理协议
├── README.md
├── README.zh-CN.md
└── .gitignore
```

SKILL.md 控制在 250 行以内 — 详细工作流和边界情况放在 `references/` 中，按需加载。

## 设计模式

本 skill 融合了 [note-merge](https://github.com/Qesire/note-merge) 参考设计中的结构模式：

| 模式 | 位置 |
|------|------|
| **调度表** | SKILL.md 顶部 — 将用户意图映射到参考文件 |
| **硬性规则** | 第 5 节 — 10 条不可协商的不变性约束 |
| **阶段结构** | search-methods.md — 阶段 0→5，含退出条件 |
| **边界情况文件** | references/edge-cases.md — 7 种触发条件→处理协议 |
| **依赖项声明** | 第 0 节 — 工具表格含回退方案 |
| **输出模板** | templates/ — 安全报告、校验报告模板 |
| **平台字段注释** | 前端元数据注释 — 标注哪些字段是平台专属 |
| **恢复协议** | 硬性规则 #6 — 报告安装路径 + 卸载命令 |

## 平台支持

| 平台 | 状态 | 文档 |
|------|------|------|
| OpenCode | 完整支持 | [opencode.ai/docs/skills](https://opencode.ai/docs/skills) |
| Claude Code | 完整支持 | [docs.anthropic.com/claude-code/skills](https://docs.anthropic.com/en/docs/claude-code/skills) |

## 许可证

MIT — 详见 [SKILL.md frontmatter](SKILL.md)。