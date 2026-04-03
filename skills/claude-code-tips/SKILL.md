---
name: claude-code-tips
description: Claude Code tips and tricks - productivity shortcuts, effective prompting, configuration, and workflow optimization.
metadata:
  priority: 10
  docs:
    - "https://docs.anthropic.com/claude-code"
  pathPatterns:
    - ".clauderc"
    - "CLAUDE.md"
  bashPatterns:
    - '\bclaude.?code\b'
    - '\b/claude\b'
  promptSignals:
    phrases:
      - "claude code tips"
      - "claude code tricks"
      - "productivity"
    anyOf:
      - "claude code"
      - "/help"
---

## Claude Code Tips

### Effective Prompting

```markdown
# Be specific about the outcome
❌ "Fix the bug"
✅ "Fix the null pointer in UserService.ts line 42 where user.profile is undefined"

# Break complex tasks into steps
❌ "Build a full auth system"
✅ "First, create the User model with email/password fields. Then we'll add JWT middleware."

# Provide context
❌ "Why is this slow?"
✅ "This React component re-renders on every keystroke even with useCallback. Profile the issue."
```

### Keyboard Shortcuts

```bash
# Command palette
Cmd+K - Open command palette

# Navigation
Cmd+P - Quick file open
Cmd+Shift+P - Command palette (same)

# Edits
Cmd+Enter - Accept suggestion
Cmd+Shift+Enter - Reject suggestion
Esc - Cancel operation

# General
Cmd+/ - Toggle sidebar
Cmd+B - Toggle terminal
```

### Project Configuration

```jsonc
// .clauderc.json
{
  "permissions": {
    "allow": [
      "Read: *",
      "Write: src/**",
      "Bash: npm test, npm run build"
    ],
    "deny": [
      "Bash: rm -rf",
      "Write: node_modules/**"
    ]
  },
  "promptHints": {
    "maxTokens": 4000,
    "temperature": 0.7
  }
}
```

### CLAUDE.md Best Practices

```markdown
<!-- CLAUDE.md -->
# Project Overview
This is a Next.js e-commerce platform with Stripe integration.

## Key Conventions
- Use TypeScript strict mode
- Prefer Server Components
- CSS Modules for styling

## Common Tasks
- `npm run dev` - Start development server
- `npm test` - Run unit tests
- `npm run lint` - Lint and fix

## Code Style
- 2 space indentation
- Single quotes in JS
- kebab-case for files

## Important Files
- src/app/ - Next.js App Router pages
- src/lib/ - Utilities and helpers
- prisma/schema.prisma - Database schema
```

### Workflow Optimization

```markdown
# Use slash commands for common operations
/claude:refactor this component to use hooks
/claude:explain this regex pattern
/claude:write tests for this function

# Chain operations
"Can you first check the tests pass, then if green merge to main and deploy to staging?"

# Ask for explanations
"Explain what this regex does" - Learn while working
```

### File Glob Patterns

```bash
# Find files
**/*.test.ts - All test files
**/components/** - All component directories
!**/node_modules/** - Exclude node_modules

# Multiple patterns
src/**/*.{ts,tsx} - TypeScript files in src
```

### Debugging Tips

```markdown
# When stuck, ask Claude to explain
"Walk me through how this function works"

# Use for code review
"Review this PR for potential bugs and suggest improvements"

# Trace execution
"Can you add console.log statements to trace the data flow?"
```

### Skill Usage

```markdown
# Skills enhance Claude's capabilities
- Available skills shown in help menu
- Skills auto-activate based on context
- Use /skill-name to explicitly invoke

# Create custom skills
skills/custom/SKILL.md - Your skill file
```

### Session Management

```bash
# New session for new problem
/exit - End current session

# Resume with context
# Just start a new session, Claude reads CLAUDE.md

# Clear context when needed
/new - Start fresh session
```

### Best Practices

1. **Be specific** - Exact file paths, line numbers
2. **Provide context** - Show relevant code
3. **Set boundaries** - CLAUDE.md for project rules
4. **Use skills** - Leverage available skills
5. **Iterate** - Build up complex features step by step
6. **Review changes** - Always check diffs before accepting
7. **Ask for explanations** - Learn while building
