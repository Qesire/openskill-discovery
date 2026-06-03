# Search Methods Reference

Detailed methods for finding agent skills on GitHub with auth requirements, rate limits, and fallbacks.

---

## Method A: GitHub CLI Code Search (requires auth)

**Prerequisite**: `gh auth login` with a GitHub account. Any token scope works (even a no-scope PAT).

```bash
gh search code "filename:SKILL.md <query>" --limit 30
```

Each result includes `repository.full_name` and `path`. For top hits, fetch the raw file:

```bash
curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path_to_SKILL.md>"
```

**Auth required**: Yes. Code Search API (`GET /search/code`) requires authentication.
**Rate limit**: 9 requests/min (authenticated).
**Pros**: Directly finds SKILL.md files. Most targeted method.
**Cons**: Requires `gh` setup with auth.

---

## Method B: Unauthenticated Repo Search + Tree Probing

Search repositories without authentication, then probe each for skill directories.

### Step 1: Search repositories
```bash
curl -s "https://api.github.com/search/repositories?q=<query>+skills&sort=stars&per_page=10" \
  | jq -r '.items[] | .full_name'
```

If `gh` is authenticated, the CLI version works too:
```bash
gh search repos "<query>" --sort stars --limit 10
```

### Step 2: Probe each repo for skill directories
```bash
# For each repo from step 1:
gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1 \
  | jq -r '.tree[].path' \
  | grep -iE '(skills|\.opencode/skills|\.claude/skills|\.codex/skills|\.agents/skills)/[^/]*/SKILL\.md$'
```

**Auth required**: Step 1 (curl) — No. Step 2 (gh api) — Yes.
**Rate limit**: 10 req/min (unauthenticated) for repo search; 5000 req/hr (authenticated) for tree probing.
**Pros**: No auth needed for initial discovery. Can find skills in repos that aren't primarily skill collections.
**Cons**: Two-step process. Tree probing requires auth.

---

## Method C: GitHub Web Search (no auth, full access)

Use WebFetch to search GitHub's web interface:

1. **Code search**: `https://github.com/search?q=filename%3ASKILL.md+<query>&type=code`
2. **Repo search**: `https://github.com/search?q=<query>+skills&type=repositories`

Fetch the results page and extract repository names and paths. Works without any authentication but requires parsing HTML.

```bash
# Example: fetch code search results for "docker"
curl -s -H "Accept: text/html" \
  "https://github.com/search?q=filename%3ASKILL.md+docker&type=code"
```

**Auth required**: No.
**Rate limit**: GitHub's web rate limit (generous for browsing, ~10-30 req/min).
**Pros**: No setup needed. Both code and repo search work.
**Cons**: HTML parsing required. Less structured than API responses.

---

## Method D: Curated Repositories

Known high-quality skill repositories to search directly:

| Repository | Contents | Stars |
|---|---|---|
| `affaan-m/everything-claude-code` | 188 skills, 50 agents across categories | ~200+ |
| `anthropics/skills` | Official Anthropic skill-creator, pdf, xlsx, frontend-design | ~5k+ |
| `obra/superpowers` | TDD, debugging, brainstorming, code-review methodology | ~2k+ |
| `addyosmani/agent-skills` | SDLC skills with Google engineering practices | ~1k+ |
| `alirezarezvani/claude-skills` | 246 skills across engineering, marketing, C-level | ~500+ |
| `openai/skills` | Official Codex curated skills catalog | ~21k+ |
| `ecc/agent-skills` | 249 skills, 63 agents, multi-platform (7+ platforms) | ~500+ |
| `OthmanAdi/planning-with-files` | Persistent markdown planning methodology | ~200+ |
| `ChenLiu-1996/figures4papers` | Academic figure generation skills | ~100+ |

---

## Fetching Skill Content

Once you've identified a candidate skill, fetch the raw SKILL.md:

```bash
curl -s "https://raw.githubusercontent.com/<owner>/<repo>/HEAD/<path>/SKILL.md"
```

For skills in subdirectories, the `HEAD` ref works universally:
```
https://raw.githubusercontent.com/anthropics/skills/HEAD/skills/skill-creator/SKILL.md
```

---

## Selecting the Best Skill

When multiple search results match, use these criteria to rank and select:

### Ranking Factors (in priority order)

1. **Source reputation**: Official repos (anthropics/skills, openai/skills) > established community repos > personal repos
2. **Stars**: Higher stars indicate community validation
3. **Recent activity**: Commits in last 3 months indicate active maintenance
4. **Platform compatibility**: Check frontmatter `compatibility` field
5. **Validation score**: Passes all checks from [Validating a SKILL.md](#) section
6. **Security scan**: Clean or low-risk security scan results
7. **Documentation quality**: Has clear instructions, examples, and edge case handling

### Handling Duplicate Names

If multiple skills share the same `name`:
- Prefer the one from a curated/official repo
- Check which target platform each is primarily designed for
- If both are equally valid, present both and let the user decide

### Handling Empty Results

If no skills are found:
- Try broader search terms (e.g., "container" instead of "docker")
- Try Method B (repo search) with related keywords
- Try Method C (web search) with different query formulations
- Check curated repos manually for related skills
- Suggest the user create a custom skill using Anthropic's skill-creator as a template