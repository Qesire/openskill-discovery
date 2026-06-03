---
name: skill-discovery
description: |
  Use when the user asks to search for skills/agents, find a skill for a specific task,
  install a skill, check skill quality, audit skill security, or adapt a Claude skill.
  Triggers on: skill, agent, install, find skill, search skill, validate skill,
  security audit, audit skill, Claude Code skill, Codex skill, OpenCode skill,
  skill quality, install agent, install from GitHub, find agent skill.
  Do NOT trigger on: general GitHub operations, general package installation,
  code review requests that don't mention skills.
license: MIT
compatibility: opencode, claude-code
metadata:
  uses_github: true
  prerequisites: "gh (optional, for code search), jq (optional, for JSON parsing), rg (optional, for security scanning)"
  platforms: "OpenCode, Claude Code"
  supported_search: "GitHub CLI, GitHub REST API, GitHub Web Search, Curated Repos"
# Claude Code-only fields; silently ignored by OpenCode:
allowed-tools: ["Read", "Bash", "Grep", "Glob", "WebFetch", "Task", "Edit", "Write"]
---

# Skill Discovery & Management

## Dispatch Table

| User intent | What to do | Load this reference |
|-------------|------------|---------------------|
| "find me a [topic] skill" / "搜索 [关键词] skill" | Search GitHub for matching skills | `references/search-methods.md` |
| "validate this skill" / "检查这个 SKILL.md" | Check frontmatter against Agent Skills spec | Section 2 + `references/platform-compatibility.md` |
| "security audit this skill" / "审计安全" | Run 15 security checks | Section 3 + `references/security-scanning.md` |
| "install this skill" / "安装这个 skill" | Copy to correct platform path | Section 4 + `references/platform-compatibility.md` |
| "review this skill" / "审查这个 skill" | Full systematic review | `references/skill-review-guide.md` |
| Unexpected behavior / edge case | Handle boundary scenarios | `references/edge-cases.md` |

---

## 0. Configuration — Dependencies

Read this before any action. Determine what's available and what needs fallback.

| Tool | Required? | Used for | If missing |
|------|-----------|----------|------------|
| `curl` | Yes | Fetching raw SKILL.md, unauthenticated repo search | Always available via Bash; no fallback needed |
| `jq` | No | Parsing JSON API responses | Use `python3 -m json.tool` or manual grep |
| `gh` CLI | No | Authenticated code search, repo probing | Fall back to Method B curl (repo search) or Method C (web search) |
| `rg` (ripgrep) | No | Security scanning with lookahead patterns | Fall back to `grep -rnP`; skip lookahead checks if PCRE unavailable |

**Sequential check:** tools → method selection (search-methods.md Phase 0) → execute → report.

---

## 1–4. Workflow Summaries

Each section below provides the core workflow. Detailed specifications, rate limits, and command variations are in the reference files.

### 1. Finding Skills on GitHub

**Priority order:**

1. **Curated repos** — No auth needed. Check the curated list first (free, high-quality).
2. **GitHub CLI code search** — Requires `gh auth login`. `gh search code "filename:SKILL.md <query>" --limit 30`.
3. **Repo search (curl)** — No auth needed. `curl "https://api.github.com/search/repositories?q=<query>&sort=stars&per_page=10"` then probe trees.
4. **Web search fallback** — No auth needed. Use WebFetch on `github.com/search?q=filename%3ASKILL.md+<query>&type=code`.

For detailed instructions, auth requirements, rate limits, and selection criteria: `references/search-methods.md`.

### 2. Validating a SKILL.md

Per the [Agent Skills specification](https://agentskills.io/specification):

- [ ] YAML frontmatter delimited by `---`
- [ ] `name`: lowercase, hyphens only, 1-64 chars, no `--`, no leading/trailing `-`. Matches directory name.
- [ ] `description`: 1-1024 chars, trigger-focused (not capability summary)
- [ ] `license` (optional): short identifier or file reference
- [ ] `compatibility` (optional): max 500 chars
- [ ] Body < 500 lines; relative paths for references; step-by-step with examples

Full field support matrix and platform-specific behaviors: `references/platform-compatibility.md`.

### 3. Security Scanning

Run **before installation**. Single combined scan:

```bash
rg -n \
  -e 'curl.*https?://(?!raw\.githubusercontent\.com|github\.com|api\.github\.com)' \
  -e 'wget.*https?://(?!raw\.githubusercontent\.com|github\.com|api\.github\.com)' \
  -e '\$\{?GITHUB_TOKEN\}?|\$\{?GH_TOKEN\}?' \
  -e 'OPENAI_API_KEY|ANTHROPIC_API_KEY|HUGGINGFACE_TOKEN' \
  -e '~/.ssh' -e '\bchmod\s+[0-7]{3,4}\b' -e '\brm\s+-rf\s+/(?!tmp/|home/|var/tmp/)' \
  -e '\beval\s+' -e '\bbase64\s+-[dD]\b' -e 'mkfifo|/dev/tcp|nc\s+-[e|l]' \
  -e 'curl.*\| *(ba)?sh|wget.*\| *(ba)?sh' -e '\b(cat|read).*\.env\b' \
  -e '\b(npm install -g|pip3? install|apt-get install|brew install)\b' \
  -e 'authorized_keys' -e '\b(env|printenv)\b' -e 'git clone.*&&' \
  -e '\bcp\s+.*/(usr/bin|usr/local/bin|etc/)\b' \
  .
```

Report each match as `⚠ [category] in [file]:[line] — [context]`. Rate risk: Clean / Low / Medium / High.

If `rg` not available, fall back to `grep -rnP`; skip lookahead checks if PCRE unavailable. Report skipped checks.

Full explanations of all 15 checks: `references/security-scanning.md`.

### 4. Installing Skills

Two scopes — ask the user which they want:

| Platform | Global (all projects) | Project-local (this repo only) |
|----------|----------------------|-------------------------------|
| OpenCode | `~/.config/opencode/skills/<name>/SKILL.md` | `.opencode/skills/<name>/SKILL.md` |
| Claude Code | `~/.claude/skills/<name>/SKILL.md` | `.claude/skills/<name>/SKILL.md` |

**Project-local recommended** inside a git repo — OpenCode walks from CWD up to the git worktree root to discover `.opencode/skills/`, `.claude/skills/`, and `.agents/skills/`. Works without git too (walks to `/`), but discovery is less efficient.

```bash
# Global install
mkdir -p ~/.config/opencode/skills/<name> && cp -r <skill_directory>/* $_/
mkdir -p ~/.claude/skills/<name> && cp -r <skill_directory>/* $_/

# Project-local install (run from repo root)
mkdir -p .opencode/skills/<name> && cp -r <skill_directory>/* $_/
mkdir -p .claude/skills/<name> && cp -r <skill_directory>/* $_/

# Agent (global)
cp <agent_file> ~/.config/opencode/agents/<name>.md
cp <agent_file> ~/.claude/agents/<name>.md

# Agent (project-local)
cp <agent_file> .opencode/agents/<name>.md
cp <agent_file> .claude/agents/<name>.md
```

Also detect plugin structures: `.opencode/INSTALL.md` (OpenCode plugin), `.claude-plugin/plugin.json` (Claude plugin). Report plugin install instructions when detected.

Full path reference, plugin structures, and per-platform install instructions: `references/platform-compatibility.md`.

---

## 5. Hard Rules

| # | Rule |
|---|------|
| 1 | Never claim `gh search code` works without auth — it requires `gh auth login` |
| 2 | Never use `.codex/skills/` paths — Codex uses `.agents/skills/`. If user mentions Codex, warn: "Codex path verification needed; check `developers.openai.com/codex/skills`" |
| 3 | Always security-scan before installing any skill |
| 4 | Always verify platform paths against documentation before installing — do not guess |
| 5 | Present all candidates with validation status and security rating; never auto-install without user confirmation |
| 6 | After installing, report what was installed and where — provide `rm -rf` uninstall command |
| 7 | When search returns zero results, try fallback methods (broader terms, Method B, Method C, curated check) before giving up |
| 8 | Curated repos take priority over raw search results when scores are equal |
| 9 | When porting a skill between platforms, warn about unsupported fields (Claude-only `allowed-tools`, `hooks`, `context: fork` are ignored by OpenCode) |
| 10 | All cross-references between SKILL.md and reference files must use relative paths (`references/xxx.md`) |
| 11 | Always ask the user whether to install globally (all projects) or project-locally (this repo only). For project-local, recommend a git repo for efficient discovery (OpenCode walks CWD→worktree root); warn if outside one but allow the install. |

---

## 6. Workflow Example

User: *"find me a docker skill"*

1. **Phase 0 — Check tools**: gh authenticated? curl available? rg available?
2. **Phase 1 — Curated repos first**: scan known repos for "docker" matches (free, high-quality)
3. **Phase 2 — Code search**: `gh search code "filename:SKILL.md docker" --limit 30` (if gh available)
4. **Phase 3 — Fallback if needed**: Repo search via curl → tree probing → web search
5. **Rank results**: Sort by repo reputation, stars, recent activity, validation score, security rating
6. **Fetch top 5**: `curl` raw SKILL.md from each candidate
7. **Validate each**: frontmatter check (Section 2)
8. **Security-scan each**: combined rg scan (Section 3)
9. **Check for plugins**: `.opencode/INSTALL.md`, `.claude-plugin/plugin.json`
10. **Present results**: table with name, description, repo, stars, validation, security rating, platforms
11. **Ask scope**: "安装为全局 skill（所有项目可用）还是项目 skill（仅当前项目）？"
    - If project-local, recommend a git repo; warn if outside one but allow
12. **Install**: follow Section 4 based on user's platform and scope choice

### Edge Cases (see `references/edge-cases.md`)

- **Empty results**: broaden terms, try Methods B/C, check curated repos
- **Rate limited**: switch to web search (Method C)
- **No gh CLI**: fall back to Method B curl (no auth) or Method C
- **No rg**: use `grep -rnP` for security scan
- **Duplicate names**: prefer official sources, present comparison
- **Non-SKILL.md repo**: check for commands/ format, report if incompatible
- **MCP/built-in skill**: report as already available, do not attempt file install
- **No scope specified**: ask "global or project-local?" before installing (see Hard Rule #11)

---

## 7. Reference Files

| Action | Load |
|--------|------|
| Search | `references/search-methods.md` |
| Validate | `references/platform-compatibility.md` |
| Security audit | `references/security-scanning.md` |
| Review | `references/skill-review-guide.md` |
| Edge cases | `references/edge-cases.md` |
| Config validation | `references/platform-compatibility.md` |

---

## 8. Quick Reference

| Task | Command |
|---|---|
| Search skills (auth) | `gh search code "filename:SKILL.md <query>" --limit 30` |
| Search repos (no auth) | `curl -s "https://api.github.com/search/repositories?q=<query>&sort=stars&per_page=10" \| jq -r '.items[].full_name'` |
| Probe repo structure | `gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1 \| jq -r '.tree[].path' \| grep -i 'SKILL\.md$'` |
| Fetch raw SKILL.md | `curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path>"` |
| Check for OpenCode plugin | `gh api repos/<owner>/<repo>/contents/.opencode/INSTALL.md` |
| Check for Claude plugin | `gh api repos/<owner>/<repo>/contents/.claude-plugin/plugin.json` |
| Security scan | `rg -n -e '...' -e '...' ... .` (15 patterns — see Section 3) |
| Install skill (OpenCode, global) | `mkdir -p ~/.config/opencode/skills/<name> && cp -r <src>/* $_/` |
| Install skill (OpenCode, project) | `mkdir -p .opencode/skills/<name> && cp -r <src>/* $_/` |
| Install skill (Claude, global) | `mkdir -p ~/.claude/skills/<name> && cp -r <src>/* $_/` |
| Install skill (Claude, project) | `mkdir -p .claude/skills/<name> && cp -r <src>/* $_/` |
| Install agent (OpenCode, global) | `cp <src> ~/.config/opencode/agents/<name>.md` |
| Install agent (OpenCode, project) | `cp <src> .opencode/agents/<name>.md` |