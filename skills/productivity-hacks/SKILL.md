---
name: productivity-hacks
description: Boost developer productivity - keyboard shortcuts, workflow automation, and time-saving techniques.
metadata:
  priority: 7
  pathPatterns:
    - "**/*.md"
  promptSignals:
    phrases:
      - "productive"
      - "faster"
      - "workflow"
    anyOf:
      - "shortcut"
      - "效率"
      - "tips"
---

## Productivity Hacks

### Terminal Productivity

```bash
# Quick directory navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -la'

# Git shortcuts
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git log --oneline -10'
alias gco='git checkout'
alias gcb='git checkout -b'
alias gb='git branch'

# Find files fast
alias f='find . -type f -name'
alias ff='find . -type f -name "*$1*"'

# Kill process on port
killport() { lsof -ti:$1 | xargs kill -9; }

# Extract any archive
extract() {
  if [ -f "$1" ]; then
    case "$1" in
      *.tar.bz2) tar xjf "$1" ;;
      *.tar.gz) tar xzf "$1" ;;
      *.bz2) bunzip2 "$1" ;;
      *.rar) unrar x "$1" ;;
      *.gz) gunzip "$1" ;;
      *.tar) tar xf "$1" ;;
      *.tbz2) tar xjf "$1" ;;
      *.tgz) tar xzf "$1" ;;
      *.zip) unzip "$1" ;;
      *.Z) uncompress "$1" ;;
      *) echo "'$1' cannot be extracted via extract()" ;;
    esac
  else
    echo "'$1' is not a valid file"
  fi
}
```

### VS Code Shortcuts

| Shortcut | Action |
|----------|--------|
| `Cmd+Shift+P` | Command palette |
| `Cmd+P` | Quick file open |
| `Cmd+Shift+F` | Search in files |
| `Cmd+D` | Select next occurrence |
| `Cmd+Shift+L` | Select all occurrences |
| `Alt+Up/Down` | Move line up/down |
| `Cmd+Shift+K` | Delete line |
| `Cmd+/` | Toggle comment |
| `Cmd+B` | Toggle sidebar |
| `Ctrl+\`` | Toggle terminal |

### Git Workflows

```bash
# Quick amend (only if not pushed!)
git commit --amend --no-edit

# Squash last N commits
git rebase -i HEAD~3
# Change 'pick' to 'squash' for commits to combine

# Undo last commit, keep changes
git reset --soft HEAD~1

# Stash with message
git stash push -m "WIP: new feature"

# List stashes
git stash list

# Apply specific stash
git stash apply stash@{2}

# Clean up merged branches
git branch --merged main | xargs git branch -d
```

### Zsh Plugins (.zshrc)

```bash
# Enable plugins
plugins=(git node npm docker python)

# Aliases with autocomplete
alias gcb='git checkout -b'
compdef _git gcb='git checkout -b'

# History search
bindkey '^R' history-incremental-search-backward
```

### Time-Saving Patterns

```typescript
// Quick CRUD API generator
function generateCRUD(resource: string) {
  return `
router.get('/${resource}', async (req, res) => {
  const data = await db.${resource}.findMany();
  res.json(data);
});

router.post('/${resource}', async (req, res) => {
  const data = await db.${resource}.create({ data: req.body });
  res.status(201).json(data);
});
`;
}
```

### Pomodoro Technique

```markdown
## 25/5 Pomodoro

Work Session (25 min):
- Focus on ONE task
- No interruptions
- Track what you complete

Break (5 min):
- Stand up
- Stretch
- Check phone/messages

Long Break (15 min after 4 pomodoros):
- Walk around
- Coffee
- Reset

## Daily Tracking
| Time | Task | Pomodoros | Notes |
|------|------|-----------|-------|
| 9:00 | API endpoint | 2 | Done |
| 9:30 | Auth flow | 3 | Blocked on review |
```

### Focus Techniques

1. **Time blocking** - Schedule deep work in calendar
2. **Do not disturb** - Slack/email off during focus
3. **Two-minute rule** - If < 2 min, do it now
4. **Single tasking** - No context switching
5. **Exit strategy** - Define "done" before starting

### Keyboard-First Workflow

```
Alfred/Tray: Quick actions
↓↓
VS Code: Code editing
↓↓
Terminal: Git, build, run
↓↓
Browser: Documentation, PR review
```

### Automation Ideas

| Task | Tool |
|------|------|
| Deploy on push | GitHub Actions |
| Lint on save | Husky + lint-staged |
| Generate changelog | Conventional commits |
| Schedule tasks | Cron + scripts |
| Monitor errors | Sentry |
| Track time | Toggl |
