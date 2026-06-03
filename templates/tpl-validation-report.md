# Validation Report Template

Use this template when presenting SKILL.md validation results.

---

## Validation Report — [skill-name]

**Source:** [repo/url]
**Tested:** YYYY-MM-DD

### Spec Compliance

| Check | Status | Detail |
|-------|--------|--------|
| YAML frontmatter (`---`) | ✅ Pass / ❌ Fail | |
| `name` lowercase, hyphens only | ✅ Pass / ❌ Fail | |
| `name` matches directory | ✅ Pass / ❌ Fail | |
| `name` 1-64 chars, no `--` | ✅ Pass / ❌ Fail | |
| `description` 1-1024 chars | ✅ Pass / ❌ Fail | ([N] chars) |
| `description` trigger-focused | ✅ Pass / ❌ Fail | |
| `license` (optional) | ✅ Pass / ❌ Fail / ⬚ N/A | |
| `compatibility` ≤ 500 chars (optional) | ✅ Pass / ❌ Fail / ⬚ N/A | |

### Platform Path Verification

| Path Claim | Platform | Verification |
|------------|----------|--------------|
| | | ✅ / ❌ / ⚠ Not verified |

### Content Quality

| Check | Score | Note |
|-------|-------|------|
| Body < 500 lines | ✅ / ❌ ([N] lines) | |
| Step-by-step instructions | ✭✭✭✭✭ | |
| Examples present | ✭✭✭✭✭ | |
| Edge cases covered | ✭✭✭✭✭ | |
| Relative references used | ✅ / ❌ | |

### Overall

**Validation:** ✅ Pass / ❌ Fail with [N] issues
**Security risk rating:** Clean / Low / Medium / High
**Recommendation:** Install / Fix issues first

### Issues Found

| # | Severity | Description |
|---|----------|-------------|
| | major / minor | |

---

## Example

```
## Validation Report — docker-setup

### Spec Compliance
| Check | Status | Detail |
|-------|--------|--------|
| YAML frontmatter | ✅ Pass | Delimited by --- |
| name lowercase, hyphens only | ✅ Pass | docker-setup |
| name matches directory | ✅ Pass | skills/docker-setup/ |
| name 1-64 chars, no -- | ✅ Pass | 13 chars |
| description 1-1024 chars | ✅ Pass | 156 chars |
| description trigger-focused | ❌ Fail | Describes capabilities, not triggers |
| license | ⬚ N/A | Not declared |
| compatibility | ⬚ N/A | Not declared |

### Overall
**Validation:** ❌ Fail with 1 issue
**Recommendation:** Fix description to include trigger keywords before installing.
```