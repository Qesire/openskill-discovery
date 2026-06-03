# Edge Cases

Boundary scenarios and handling protocols. Load this when any action encounters unexpected input or when the user asks about limitations and fallback behavior.

---

## 1. Empty Search Results

**Trigger:** Search yields zero matching skills across all methods.

```
1. Check if the query is too narrow (specialized terms, typos)
2. Broaden: use parent category terms (e.g., "container" instead of "docker")
3. Try Method B (repo search) with related keywords
4. Try Method C (web search) with different query formulations
5. Check curated repos manually for related skills
6. If still zero: "未找到匹配的技能。建议: [broader terms], 或自己创建一个 skill（参考 anthropics/skills 的 skill-creator）。"
7. Never fabricate results
```

---

## 2. Rate Limited

**Trigger:** `gh search code` returns rate limit error (429) or `curl` returns "API rate limit exceeded".

```
1. If using gh CLI: wait 60 seconds before retrying OR
2. Switch to Method C (web search via WebFetch) — no rate limit
3. If using curl (repo search): wait 60 seconds, retry once
4. Report to user: "GitHub API 频率限制，已切换到网页搜索/等待重试。"
5. If rate limited repeatedly: suggest user authenticate with gh auth login
```

---

## 3. No gh CLI / Not Authenticated

**Trigger:** `gh search code` returns "gh: command not found" or "You are not logged in."

```
1. gh not installed:
   - Fall back to Method B step 1 (curl repo search, no auth needed)
   - Fall back to Method C (web search, no auth needed)
   - Report: "gh CLI 未安装。已使用无需认证的搜索方法。建议安装 gh CLI 并认证以获得更好的搜索结果。"

2. gh installed but not authenticated:
   - Same fallback as above
   - Report: "gh CLI 未登录。已使用无需认证的搜索方法。运行 'gh auth login' 后可使用代码搜索。"
```

---

## 4. rg Not Installed

**Trigger:** Security scan `rg -n -e ...` returns "rg: command not found."

```
1. Fall back to grep -rn for the same patterns
2. Note: grep doesn't support lookahead in basic mode. Use grep -rnP for PCRE if available.
3. If neither rg nor grep -P is available: run individual grep -rn for simple patterns only
4. Report: "rg 未安装，已使用 grep 进行基本检查。安装 ripgrep 可获得更精确的安全扫描。"
5. List which patterns were skipped due to grep limitations
```

---

## 5. Duplicate Skill Names

**Trigger:** Multiple search results produce skills with the same `name` but different repositories.

```
1. Compare metadata: stars, recent activity, platform compatibility
2. Prefer official sources (anthropics/skills, openai/skills) over community repos
3. If both are equivalent: present both with a comparison table
4. Let user decide — do not auto-select
5. If installing one, note the alternative in case it's needed later

Report format:
"发现同名技能 [name]:
  - [A] from [repo] ([n] stars, updated [date]), 支持: [platforms]
  - [B] from [repo] ([n] stars, updated [date]), 支持: [platforms]
  请选择一个安装。"
```

---

## 6. Non-SKILL.md Repository

**Trigger:** Repo structure probing finds `skills/` or `.claude/skills/` directories but no `SKILL.md` files, or only `commands/` .md files.

```
1. Check if the repo uses commands/*.md (Claude Code legacy format) instead of skills/*/SKILL.md
2. If commands/ found: "该仓库使用旧版 commands/ 格式，而非 skills/SKILL.md。是否仍要尝试？"
3. If neither found: "该仓库不包含 Agent Skills 格式的技能文件。"
4. Do not attempt to convert commands/ to skills/ without user confirmation
```

---

## 7. MCP-Provided or Built-in Skill

**Trigger:** Skill is provided by an MCP server or is built into the platform, not installable as a file.

```
1. Check if the skill name matches known MCP-provided or built-in skills
2. If MCP: "该技能由 MCP 服务器提供，不需要手动安装。已在 [platform] 的 MCP 配置中启用即可。"
3. If built-in: "该技能是 [platform] 的内置技能，已自动可用，无需安装。"
4. Do not attempt to install MCP/built-in skills as files
```

---

## General Fallback Principle

When any tool or method is unavailable:

```
1. Report what's missing to the user (one sentence, not a wall of text)
2. Attempt the next-best method automatically
3. Note limitations of the fallback method
4. Always suggest the ideal setup (gh auth login, install rg) for future use
```