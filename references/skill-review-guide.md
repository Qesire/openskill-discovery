# Skill Review Guide

A systematic methodology for reviewing agent skills against platform specifications, best practices, and security requirements. Use this guide when auditing a skill for correctness, completeness, and quality.

---

## Review Framework

### Five Dimensions

| # | Dimensions | Core Question |
|---|-----------|---------------|
| 1 | **Spec Compliance** | Does the frontmatter follow the Agent Skills spec (`agentskills.io`)? |
| 2 | **Platform Accuracy** | Are paths, tool names, field support, and auth claims correct for each platform? |
| 3 | **Security** | Does the skill introduce supply-chain, exfiltration, or execution risks? |
| 4 | **Generality** | Does it work across platforms? Are assumptions explicit? Are edge cases handled? |
| 5 | **Design Quality** | Progressive disclosure, token budget, trigger clarity, structure, examples |

For each dimension, classify findings:

| Severity | Criteria | Action |
|----------|----------|--------|
| **Critical** | Functionally broken — wrong paths, false auth claims, unsupported platform claims | Must fix before use |
| **Major** | Significant inaccuracies — missing fields, incomplete checks, misleading claims | Fix before distribution |
| **Minor** | Improvements — wording, missing edge cases, missing optional features | Fix when convenient |

---

## Research Workflow

### Step 1: Gather Specifications

Always check these sources **first**, before making any claims:

| Source | URL | What it covers |
|--------|-----|----------------|
| Agent Skills Spec | `https://agentskills.io/specification` | Canonical field definitions, naming rules, directory structure |
| OpenCode Skills Docs | `https://opencode.ai/docs/skills` | OpenCode-specific: path discovery, permissions, recognized fields |
| Claude Code Skills Docs | `https://docs.anthropic.com/en/docs/claude-code/skills` | Claude extension fields, plugin structure, tool names, lifecycle |
| Claude Code Plugins | `https://docs.anthropic.com/en/docs/claude-code/plugins-reference` | Plugin manifests, directory layout, loader behavior |
| Codex Skills Docs | `https://developers.openai.com/codex/skills` | Codex paths (`.agents/skills/`), `$skill-name` invocation, config |

### Step 2: Check Reference Implementations

Study high-quality skills to validate conventions:

| Repository | Why It Matters |
|------------|---------------|
| `anthropics/skills` | Official reference for SKILL.md structure and trigger descriptions |
| `obra/superpowers` | Best example of rationalization countermeasures and pass/fail patterns |
| `ecc/agent-skills` | Multi-platform reference (7+ platforms), plugin architecture |
| `openai/skills` | Codex curated catalog, tiered quality system (`.system/`, `.curated/`) |

### Step 3: Read the Skill Source

Read the full SKILL.md. Note:
- Frontmatter (all fields present? values valid?)
- All referenced paths (do they match platform docs?)
- All commands (auth requirements, tool availability)
- Assumptions about the environment (`rg` available? `gh` authenticated? `jq` installed?)

---

## Concrete Check Items

### Spec Compliance

- [ ] YAML frontmatter delimited by `---`
- [ ] `name`: lowercase, hyphens only, 1-64 chars, no `--`, no leading/trailing `-`
- [ ] `name` matches parent directory name
- [ ] `description`: 1-1024 chars, describes triggers not workflow summary
- [ ] `license` (optional): valid short identifier
- [ ] `compatibility` (optional): max 500 chars
- [ ] Body < 500 lines (token budget)

### Platform Accuracy

**Paths** — the most common source of errors:

| What to Verify | Common Error |
|---------------|--------------|
| OpenCode skill paths | Using `<source>/<name>` nesting in install commands. Correct: `skills/<name>/SKILL.md`. Nested discovery works but names are flat. |
| Codex paths | Using `.codex/skills/` instead of `.agents/skills/`. Codex uses `.agents/` namespace. |
| Claude Code paths | Using `.claude/commands/` as primary. Skills go in `.claude/skills/<name>/`. |

**Auth claims** — verify these don't mislead:

| Claim to Check | Reality Check |
|----------------|---------------|
| "Works without auth" | `gh search code` requires auth. `gh search repos` requires auth for the CLI, not for the raw API. |
| "No API key needed" | GitHub Code Search API requires a token. Repo search API does not (with rate limits). |

**Tool names** — platform differences:

| Tool | Claude Code | OpenCode | Codex |
|------|------------|----------|-------|
| Web search | `WebSearch` | `WebFetch` (no search backend) | `WebFetch` |
| URL fetch | `WebFetch` | `WebFetch` | `WebFetch` |
| User prompt | `Question` | `Question` | `AskUserQuestion` |
| Notebooks | `NotebookRead`, `NotebookEdit` | — | — |

**Field support** — don't assume portability:

| Field | Standard? | Claude | OpenCode | Codex |
|-------|-----------|--------|----------|-------|
| `allowed-tools` | Experimental | ✓ | ✗ | ✗ |
| `argument-hint` | — | ✓ | ✗ | ✗ |
| `disable-model-invocation` | — | ✓ | ✗ | ✗ |
| `context: fork` | — | ✓ | ✗ | ✗ |
| `hooks` | — | ✓ | ✗ | ✗ |
| `paths` | — | ✓ | ✗ | ✗ |

**Plugin detection** — don't miss platform-specific install methods:

| Platform | Plugin Indicator | Install Method |
|----------|-----------------|----------------|
| OpenCode | `.opencode/INSTALL.md` or `package.json` with `@opencode-ai/plugin` | Add to `opencode.json` plugins array |
| Claude Code | `.claude-plugin/plugin.json` | `claude /plugin install` or manual copy |
| Codex | Plugin package structure | Codex plugin system |

### Security

Run the combined security scan (15 patterns — see `references/security-scanning.md`). Pay special attention to:

- **High-severity**: reverse shells, pipe-to-shell, base64+eval combos
- **Network**: curl/wget to non-standard domains (may be legitimate — review, don't just flag)
- **Credentials**: embedded tokens, API keys, SSH key references
- **File system**: `rm -rf /`, `cp` to system dirs, `chmod 777`
- **Persistence**: `authorized_keys`, system package installs, `git clone && execute`

When reviewing flagged items:
- Not all curl to non-GitHub domains is malicious (docs sites, package registries are normal)
- `rm -rf /tmp/something` is common cleanup — the regex should exclude `/tmp/`, `/home/`, `/var/tmp/`
- Warnings are for the user's awareness — explain the risk, let the user decide

### General Applicability

- [ ] Are platform-specific instructions clearly labeled?
- [ ] Is there a fallback path when required tools are unavailable (no `gh`, no `rg`)?
- [ ] Are edge cases mentioned (empty search results, rate limits, duplicate names)?
- [ ] Does the skill handle all 3 platforms if cross-platform is claimed?
- [ ] Are commands copy-pasteable with placeholders clearly documented?

### Design Quality

**Progressive disclosure**:
- [ ] SKILL.md body < 500 lines
- [ ] Large reference tables moved to `references/` files
- [ ] References linked with relative paths from SKILL.md

**Trigger description** (per Anthropic's guidance — skills "under-trigger"):
- [ ] Description starts with trigger conditions, not capability summary
- [ ] Specific symptoms and scenarios mentioned (not just abstract intent)
- [ ] Keywords included for matching

**Instruction structure** (per Superpowers/ECC patterns):
- [ ] Overview: 1-2 sentence core principle
- [ ] When to use / When NOT to use (side by side)
- [ ] Core workflow: numbered steps or flowcharts (flowcharts only for non-obvious decision points)
- [ ] Patterns/Examples: pass/fail comparisons with rationale
- [ ] Edge cases and common mistakes
- [ ] Anti-patterns: what to explicitly avoid
- [ ] Integration points: how this skill composes with others

**Token efficiency**:
- [ ] Each workflow step < 150 words
- [ ] No repeated patterns across examples
- [ ] No storytelling — just patterns

---

## Output Format

When presenting a review to the user, use this format:

### Review Summary

```
## 审查总结 (Review Summary)

### 严重问题 (Critical — broken functionality)
| # | 问题 (Issue) | 详情 (Detail) |
|---|-------------|---------------|

### 重要问题 (Major — significant inaccuracies)
| # | 问题 | 详情 |
|---|------|------|

### 次要问题 (Minor — improvements)
| # | 问题 | 详情 |
|---|------|------|

### 正面内容 (Positive aspects)
- item 1
- item 2
```

### Fix Plan

Structure fixes as phases:

```
### Phase N: [Theme]
**A) Fix description** — what changes, why
**B) ...**
```

### Verification

After fixes are applied, verify:
- [ ] All cross-references resolve (file paths in SKILL.md exist on disk)
- [ ] Zero remaining instances of old incorrect patterns (grep for `<source>`, old paths)
- [ ] SKILL.md body under 500 lines
- [ ] Frontmatter passes spec validation
- [ ] All platform paths match their respective documentation

---

## Common Findings Database

Patterns frequently discovered in skill reviews:

| Pattern | Root Cause | Fix |
|---------|-----------|-----|
| Codex paths use `.codex/` instead of `.agents/` | Codex rebranded/changed namespace | Replace all `.codex/skills` with `.agents/skills` |
| OpenCode paths use `<source>/<name>` nesting | Confusion with plugin structure or old conventions | Use flat `<name>/SKILL.md`. Note nested discovery + PR #27981. |
| "Works without auth" on `gh search code` | Assuming GitHub CLI works unauthenticated | Required: `gh auth login`. Fallback: web search via WebFetch. |
| Claude fields not documented | Skill predates Claude extended fields | Add `disable-model-invocation`, `user-invocable`, `arguments`, `context`, etc. |
| Security scan incomplete | Only checks curl, not wget or pipe-to-shell | Add all 15 patterns from security-scanning reference. |
| Description describes capability, not trigger | Following old documentation conventions | Rewrite: "Use when..." format with specific symptoms and keywords. |
| Missing plugin detection | Only checks one platform's plugin format | Check all 3: `.opencode/INSTALL.md`, `.claude-plugin/plugin.json`, Codex plugin |
| Tool name mapping assumes WebSearch = WebFetch | Not checking actual tool availability | Claude Code has both. Document as separate tools, not a mapping. |