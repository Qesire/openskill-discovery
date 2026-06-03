# Search Methods Reference

Phase-ordered search workflow with auth requirements and fallbacks. Each phase exits on success; proceed to the next only if the previous fails.

---

## Phase 0: Tool Availability Check

Before searching, determine what tools are available:

| Tool | Check command | If available | If not |
|------|--------------|--------------|--------|
| `gh` CLI + auth | `gh auth status` | Phase 2 (code search) works | Skip Phase 2; use Phase 3 + Phase 4 |
| `curl` | Always available via Bash | Phase 1, 3, 4 work | None (always available) |
| `jq` | `jq --version` | Parse JSON responses cleanly | Use `python3 -m json.tool` or manual grep |
| `rg` | `rg --version` | Security scan works fully | Security scan uses `grep -rnP` |

---

## Phase 1: Curated Repositories (always run first)

Search known high-quality skill repositories directly. No auth needed, no rate limits, highest-quality results.

| Repository | Contents | Best for |
|---|---|---|
| `anthropics/skills` | Official Anthropic: skill-creator, pdf, xlsx, frontend-design | Reference implementations |
| `openai/skills` | Official Codex curated catalog (~21k stars) | Codex-specific skills |
| `obra/superpowers` | TDD, debugging, brainstorming, code-review methodology | Development workflow skills |
| `addyosmani/agent-skills` | SDLC skills with Google engineering practices | Software engineering |
| `affaan-m/everything-claude-code` | 188 skills, 50 agents | Broad coverage |
| `alirezarezvani/claude-skills` | 246 skills across engineering, marketing, C-level | Non-engineering domains |
| `ecc/agent-skills` | 249 skills, 63 agents, multi-platform | Cross-platform reference |
| `OthmanAdi/planning-with-files` | Persistent markdown planning | Project management |
| `ChenLiu-1996/figures4papers` | Academic figure generation | Academic/LaTeX |

**How to use:**

1. Scan the table for repos matching the user's query domain
2. Probe the matching repo's skill directory: `gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1 | jq -r '.tree[].path' | grep -i 'SKILL\.md$'`
3. For each match, fetch the raw SKILL.md: `curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path>"`
4. If matches found → rank, validate, present. Exit.
5. If no matches → proceed to Phase 2.

---

## Phase 2: GitHub CLI Code Search (requires auth)

**Prerequisite:** `gh auth login` (any token scope works).

```bash
gh search code "filename:SKILL.md <query>" --limit 30
```

Each result includes `repository.full_name` and `path`. For top hits, fetch the raw file:

```bash
curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path_to_SKILL.md>"
```

**Auth required:** Yes. Code Search API requires authentication.
**Rate limit:** 9 req/min (authenticated).
**If unavailable:** Skip to Phase 3.

---

## Phase 3: Unauthenticated Repo Search + Tree Probing

Works partially without authentication — repo discovery is free, tree probing requires auth.

### Step 3a: Search repositories (no auth needed)

```bash
curl -s "https://api.github.com/search/repositories?q=<query>+skills&sort=stars&per_page=10" \
  | jq -r '.items[] | .full_name'
```

**Rate limit:** 10 req/min (unauthenticated).

### Step 3b: Probe each repo for skill directories (auth needed for gh api)

```bash
gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1 \
  | jq -r '.tree[].path' \
  | grep -iE '(skills|\.opencode/skills|\.claude/skills|\.codex/skills|\.agents/skills)/[^/]*/SKILL\.md$'
```

**Rate limit:** 5000 req/hr (authenticated).

---

## Phase 4: GitHub Web Search Fallback (no auth)

Use WebFetch on GitHub's web search as the last resort:

```
https://github.com/search?q=filename%3ASKILL.md+<query>&type=code
https://github.com/search?q=<query>+skills&type=repositories
```

Fetch results and extract repository names and paths from HTML. Works without authentication but requires HTML parsing.

**Pros:** No setup, code and repo search both work.
**Cons:** HTML parsing, less structured than API responses.

---

## Phase 5: Ranking and Selection

When multiple candidates are found across any phase, apply these ranking criteria in order:

| Priority | Factor | Weight |
|----------|--------|--------|
| 1 | Official/curated source | +3 (anthropics, openai, obra) |
| 2 | Stars | +2 (>500), +1 (>100), 0 (else) |
| 3 | Recent activity (commits < 3 months) | +1 |
| 4 | Platform compatibility (matches user's platform) | +1 |
| 5 | Validation score (all checks pass) | +1 |
| 6 | Security rating (Clean/Low) | +1, (Medium) 0, (High) -3 |

### Duplicate Names

If multiple skills share the same `name`:
1. Prefer official/curated sources
2. Check platform specificity
3. Present both with a comparison table if equally valid

### Empty Results Across All Phases

```
"未找到匹配的技能。
 建议:
 - 用更宽泛的关键词重试 (如 'container' 替代 'docker')
 - 在 curated repos 中手动查找
 - 参考 anthropics/skills 的 skill-creator 自行创建"
```

---

## Fetching Skill Content

Once candidates are selected, fetch the raw SKILL.md:

```bash
curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path>/SKILL.md"
```

The `HEAD` ref works universally across all repos.