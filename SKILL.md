---
name: skill-discovery
description: |
  Use when the user asks to search for skills/agents, find a skill for a specific task,
  install a skill, check skill quality, audit skill security, or adapt a Claude skill.
  Triggers on: skill, agent, install, find skill, search skill, validate skill,
  security audit, audit skill, Claude Code skill, Codex skill, OpenCode skill,
  skill quality, install agent, install from GitHub, find agent skill.
license: MIT
compatibility: opencode, claude-code, codex
---

# Skill Discovery & Management

## Overview

You are an AI coding agent. This skill teaches you how to find, validate, security-audit, and install agent skills from GitHub across OpenCode, Claude Code, and Codex.

**Skills** go in:
- OpenCode: `~/.config/opencode/skills/<name>/SKILL.md` (flat; nested `group/name` accepted but name is flat until PR #27981 adds `:` prefixing)
- Claude Code: `~/.claude/skills/<name>/SKILL.md`
- Codex: `~/.agents/skills/<name>/SKILL.md`
**Agents** go in:
- OpenCode: `~/.config/opencode/agents/<name>.md`
- Claude Code: `~/.claude/agents/<name>.md`
- Codex: `~/.agents/agents/<name>.md`

All platforms support project-local paths (`.opencode/skills/`, `.claude/skills/`, `.agents/skills/`).
OpenCode additionally discovers skills from `.claude/skills/` and `.agents/skills/` for cross-platform compatibility.

See [platform-compatibility.md](references/platform-compatibility.md) for the full path reference, field support matrix, and plugin structures.

When auditing or reviewing skills, use [skill-review-guide.md](references/skill-review-guide.md) for the systematic review methodology.

---

## 1. Finding Skills on GitHub

### Method A: GitHub CLI Code Search (requires auth)

```bash
gh search code "filename:SKILL.md <query>" --limit 30
```

Requires `gh auth login` (any token scope works). Rate limit: 9 req/min.

Each result gives `repository.full_name` and `path`. For top hits, fetch the raw file:

```bash
curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path_to_SKILL.md>"
```

### Method B: Repo Search + Tree Probing (repo search works without auth)

```bash
# Step 1: Search repositories (works unauthenticated via curl, 10 req/min)
curl -s "https://api.github.com/search/repositories?q=<query>+skills&sort=stars&per_page=10" \
  | jq -r '.items[] | .full_name'

# Or with gh CLI (requires auth):
gh search repos "<query>" --sort stars --limit 10

# Step 2: Probe each repo for skill directories (requires auth)
gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1 \
  | jq -r '.tree[].path' \
  | grep -iE '(skills|\.opencode/skills|\.claude/skills|\.codex/skills|\.agents/skills)/[^/]*/SKILL\.md$'
```

### Method C: GitHub Web Search (no auth)

Use WebFetch on GitHub's web search as a fallback when CLI auth isn't available:

```
https://github.com/search?q=filename%3ASKILL.md+<query>&type=code
```

### Method D: Curated Repositories

Known high-quality skill repositories:

| Repository | Contents |
|---|---|
| `anthropics/skills` | Official Anthropic: skill-creator, pdf, xlsx, frontend-design |
| `openai/skills` | Official Codex curated skills catalog (~21k stars) |
| `obra/superpowers` | TDD, debugging, brainstorming, code-review methodology |
| `addyosmani/agent-skills` | SDLC skills with Google engineering practices |
| `affaan-m/everything-claude-code` | 188 skills, 50 agents |
| `alirezarezvani/claude-skills` | 246 skills across engineering, marketing, C-level |
| `ecc/agent-skills` | 249 skills, 63 agents, multi-platform |
| `OthmanAdi/planning-with-files` | Persistent markdown planning |
| `ChenLiu-1996/figures4papers` | Academic figure generation skills |

See [search-methods.md](references/search-methods.md) for detailed instructions, rate limits, and selection criteria.

---

## 2. Validating a SKILL.md

Per the [Agent Skills specification](https://agentskills.io/specification):

**Required fields:**
- [ ] YAML frontmatter delimited by `---`
- [ ] `name`: lowercase, hyphens only, 1-64 chars, no consecutive hyphens (`--`), no leading/trailing `-`
- [ ] `name` matches the parent directory name
- [ ] `description`: 1-1024 chars, describes both what AND when to use

**Optional fields:**
- [ ] `license`: short license name or reference
- [ ] `compatibility`: 1-500 chars, environment requirements
- [ ] `metadata`: arbitrary key-value map
- [ ] `allowed-tools`: pre-approved tools list (experimental; Claude Code ✓; OpenCode ✗)

**Recommended:**
- [ ] Body < 500 lines (token budget)
- [ ] Relative paths for references (e.g., `references/REFERENCE.md`)
- [ ] Step-by-step instructions with examples and edge cases
- [ ] Description focuses on trigger conditions, not workflow summary

---

## 3. Security Scanning

Run security checks in the skill directory **before installation**.

**Quick combined scan** — run this single command for all checks:

```bash
rg -n \
  -e 'curl.*https?://(?!raw\.githubusercontent\.com|github\.com|api\.github\.com)' \
  -e 'wget.*https?://(?!raw\.githubusercontent\.com|github\.com|api\.github\.com)' \
  -e '\$\{?GITHUB_TOKEN\}?|\$\{?GH_TOKEN\}?' \
  -e 'OPENAI_API_KEY|ANTHROPIC_API_KEY|HUGGINGFACE_TOKEN' \
  -e '~/.ssh' \
  -e '\bchmod\s+[0-7]{3,4}\b' \
  -e '\brm\s+-rf\s+/(?!tmp/|home/|var/tmp/)' \
  -e '\beval\s+' \
  -e '\bbase64\s+-[dD]\b' \
  -e 'mkfifo|/dev/tcp|nc\s+-[e|l]' \
  -e 'curl.*\| *(ba)?sh|wget.*\| *(ba)?sh' \
  -e '\b(cat|read).*\.env\b' \
  -e '\b(npm install -g|pip3? install|apt-get install|brew install)\b' \
  -e 'authorized_keys' \
  -e '\b(env|printenv)\b' \
  -e 'git clone.*&&' \
  -e '\bcp\s+.*/(usr/bin|usr/local/bin|etc/)\b' \
  .
```

**Checks performed** (high-severity checks in **bold**):

| # | Check | What it catches |
|---|-------|-----------------|
| 1 | Network exfiltration (curl/wget) | Fetching from non-GitHub domains |
| 2 | Credential references | Embedded tokens and API keys |
| 3 | SSH key access | Reading `~/.ssh` private keys |
| 4 | File permissions | `chmod 777` etc. |
| 5 | Destructive operations | `rm -rf /` on system dirs |
| 6 | Code execution | Shell `eval` |
| 7 | Base64 obfuscation | Base64 decode of hidden payloads |
| 8 | **Reverse shells** | `mkfifo`, `/dev/tcp`, `nc -e` |
| 9 | **Pipe to shell** | `curl ... \| bash` supply-chain attacks |
| 10 | Sensitive file reading | `cat .env` |
| 11 | Unsolicited packages | `npm install -g`, `pip install` |
| 12 | SSH persistence | Writing to `authorized_keys` |
| 13 | Environment dumping | `env`, `printenv` |
| 14 | Clone-and-execute | `git clone ... && ...` |
| 15 | System directory writes | `cp` to `/usr/bin`, `/etc/` |

Report each match as `⚠ WARNING: [category] in [file]:[line]` with pattern description and the matched line.

After scanning, rate the risk: **Clean** (no warnings) / **Low** (1-2, none high-severity) / **Medium** (3-5, or any destructive/system) / **High** (reverse shells, base64+eval combos, pipe-to-shell).

See [security-scanning.md](references/security-scanning.md) for detailed explanations of each check.

---

## 4. Installing Skills

### Per-Platform Paths

| Platform | Skill path | Agent path |
|----------|-----------|------------|
| OpenCode | `~/.config/opencode/skills/<name>/SKILL.md` | `~/.config/opencode/agents/<name>.md` |
| Claude Code | `~/.claude/skills/<name>/SKILL.md` | `~/.claude/agents/<name>.md` |
| Codex | `~/.agents/skills/<name>/SKILL.md` | `~/.agents/agents/<name>.md` |

### Standard install (file copy)

```bash
# Skill
mkdir -p ~/.config/opencode/skills/<name>   # OpenCode
mkdir -p ~/.claude/skills/<name>             # Claude Code
mkdir -p ~/.agents/skills/<name>             # Codex
cp -r <skill_directory>/* <target_path>/

# Agent
mkdir -p ~/.config/opencode/agents           # OpenCode
mkdir -p ~/.claude/agents                    # Claude Code
mkdir -p ~/.agents/agents                    # Codex
cp <agent_file> <target_path>/<name>.md
```

### OpenCode Plugin install

If the repo contains `.opencode/INSTALL.md` or `package.json` with `@opencode-ai/plugin` in dependencies:

> This is an OpenCode plugin. Add to `~/.config/opencode/opencode.json`:
> ```json
> { "plugin": ["<name>@git+https://github.com/<owner>/<repo>.git"] }
> ```
> Then restart OpenCode.

### Claude Code Plugin install

If the repo contains `.claude-plugin/plugin.json`:

> This is a Claude Code plugin. Install it with:
> ```bash
> claude /plugin install github.com/<owner>/<repo>
> ```
> Or copy the plugin directory to your Claude Code plugins folder.

### Codex

Codex invokes skills with `$skill-name`. Skills can be disabled in `~/.codex/config.toml` via `[[skills.config]]` entries.

---

## 5. Platform Compatibility

See [platform-compatibility.md](references/platform-compatibility.md) for the complete field support matrix, tool name mappings, plugin structures, and porting guidance.

### Quick Field Support Summary

**All platforms**: `name` (required), `description` (required), `license`, `compatibility`, `metadata`

**Claude Code only**: `allowed-tools`, `disallowed-tools`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `when_to_use`, `model`, `effort`, `context` (fork), `agent`, `hooks`, `paths`, `shell`, `maxTurns`

**OpenCode only**: [skills.paths](https://opencode.ai/docs/skills/) and [skills.urls](https://opencode.ai/docs/skills/) in `opencode.json` for custom skill locations

### Tool Names

Read, Bash, Grep, Glob, Edit, Write, Task, TodoWrite, Skill, WebFetch — same names on all platforms.
Claude Code additionally has `WebSearch` (no equivalent in OpenCode/Codex).

---

## 6. Workflow Example

User: *"find me a docker skill"*

1. **Search**: `gh search code "filename:SKILL.md docker" --limit 30`
2. **Rank**: Sort results by repo stars, check curated repos list first
3. **Fetch top 5**: `curl` raw SKILL.md from each candidate
4. **Validate each**: check frontmatter fields against spec (section 2)
5. **Security-scan each**: run the 15 combined checks (section 3)
6. **Check for plugins**: look for `.opencode/INSTALL.md`, `.claude-plugin/plugin.json`, etc.
7. **Present results**: for each candidate, show:
   - Name, description, source repo, stars
   - Validation status (pass/fail with details)
   - Security rating (Clean/Low/Medium/High)
   - Supported platforms
   - Install methods per platform
8. **Install**: follow section 4 based on the target platform

### Edge Cases

- **Empty results**: Broaden terms, try Method B/C, check curated repos manually. Suggest creating a custom skill.
- **Duplicate names**: Prefer official sources, present both with comparison if equally valid.
- **Rate limited**: Switch to Method C (web search) or Method B (unauthenticated repo search).
- **No gh CLI**: Use Method B step 1 with curl (no auth) or Method C (web search).

---

## 7. Quick Reference

| Task | Command |
|---|---|
| Search skills (auth required) | `gh search code "filename:SKILL.md <query>" --limit 30` |
| Search repos (no auth with curl) | `curl -s "https://api.github.com/search/repositories?q=<query>&sort=stars&per_page=10"` |
| Probe repo structure | `gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1` |
| Fetch raw SKILL.md | `curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path>"` |
| Check for OpenCode plugin | `gh api repos/<owner>/<repo>/contents/.opencode/INSTALL.md` |
| Check for Claude plugin | `gh api repos/<owner>/<repo>/contents/.claude-plugin/plugin.json` |
| Security scan (combined) | `rg -n -e '...' -e '...' ... .` (15 patterns — see section 3) |
| Install skill (OpenCode) | `mkdir -p ~/.config/opencode/skills/<name> && cp -r <src>/* $_/` |
| Install skill (Claude Code) | `mkdir -p ~/.claude/skills/<name> && cp -r <src>/* $_/` |
| Install skill (Codex) | `mkdir -p ~/.agents/skills/<name> && cp -r <src>/* $_/` |
| Install agent (OpenCode) | `cp <src> ~/.config/opencode/agents/<name>.md` |
| Install agent (Claude Code) | `cp <src> ~/.claude/agents/<name>.md` |
| Install agent (Codex) | `cp <src> ~/.agents/agents/<name>.md` |