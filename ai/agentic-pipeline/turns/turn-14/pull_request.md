# Pull Request: Turn 14

## Summary

- Created new `session-context-size` skill for capturing context window usage
- Skill generates reports with file sizes, skill counts, and context metrics
- Reports are saved to `./logs/context-{N}.md` with auto-incrementing numbers

## Files Changed

| Path | Action | Description |
|------|--------|-------------|
| `.claude/skills/session-context-size/SKILL.md` | Created | New skill for context usage reporting |

## Compliance Checklist

- [x] Follows skill YAML frontmatter format
- [x] Includes step-by-step instructions
- [x] Uses bash commands for file operations
- [x] Generates markdown report format
- [x] Auto-increments file numbers to avoid overwrites
