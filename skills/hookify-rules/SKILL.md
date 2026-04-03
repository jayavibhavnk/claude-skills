---
name: hookify-rules
description: Create Claude Code hooks to prevent unwanted behaviors - pattern matching, warnings, and blocks.
metadata:
  priority: 8
  docs:
    - "https://github.com/claude-code/claude-code-plugin-hookify"
  pathPatterns:
    - ".claude/**/*.md"
  bashPatterns:
    - '\bhookify\b'
  promptSignals:
    phrases:
      - "hook"
      - "prevent"
      - "block"
---

## Hookify Rules

### What are Hooks?

Hooks prevent unwanted behaviors by analyzing conversation patterns or explicit instructions.

### Rule Structure

```
.claude/hookify.dangerous-rm.local.md
```

```markdown
---
name: block-dangerous-rm
enabled: true
event: bash
pattern: rm\s+-rf
action: block
---

⚠️ **Dangerous rm command detected!**

This command could delete important files. Please:
- Verify the path is correct
- Consider using a safer approach
- Make sure you have backups
```

### Rule Fields

| Field | Options | Description |
|-------|---------|-------------|
| `name` | string | Unique identifier |
| `enabled` | true/false | Active state |
| `event` | bash, read, write, edit, stop, pretooluse, posttooluse | Trigger event |
| `pattern` | regex string | What to match |
| `action` | warn, block | What to do |

### Events

| Event | When it fires |
|-------|--------------|
| `bash` | Before shell commands |
| `read` | Before file reads |
| `write` | Before file writes |
| `edit` | Before edits |
| `stop` | When session stops |
| `pretooluse` | Before any tool |
| `posttooluse` | After any tool |

### Patterns

```markdown
# Single pattern
pattern: rm\s+-rf

# Multiple alternatives
pattern: (rm\s+-rf|git\s+reset\s+--hard)

# File path matching
pattern: \.env$

# Case insensitive
pattern: (?i)password\s*=\s*['"].*['"]
```

### Actions

**warn** - Shows message but allows execution:
```markdown
---
action: warn
---

⚠️ **Security warning**: Direct password in code detected.
Consider using environment variables instead.
```

**block** - Prevents execution:
```markdown
---
action: block
---

🚫 **Blocked**: Dangerous command detected.
```

### Examples

#### Prevent Destructive Git
```markdown
---
name: block-git-reset-hard
enabled: true
event: bash
pattern: git\s+reset\s+--hard
action: warn
---

⚠️ **git reset --hard detected!**

This discards all local changes. Are you sure?
Consider: git stash instead
```

#### Block console.log in Production
```markdown
---
name: no-console-log-prod
enabled: true
event: write
pattern: console\.log
action: warn
---

⚠️ **console.log detected**

Remove before committing to production.
Use structured logging instead.
```

#### Prevent env commits
```markdown
---
name: block-env-commit
enabled: true
event: bash
pattern: git\s+commit.*\.env
action: block
---

🚫 **.env files should never be committed!**

Add .env to .gitignore and use a secrets manager.
```
