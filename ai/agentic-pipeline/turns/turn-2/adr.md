# ADR: File-Based Tech Stack Configuration

**Status**: Accepted
**Date**: 2026-02-24
**Turn**: 2

## Context

The `/execute` skill previously accepted tech stack as inline `+`-separated tokens (e.g., `stack=nextjs+nestjs+drizzle`). The user requested changing this to accept a file path instead, aligning with how PRD and DDD inputs work.

## Decision

Change the `stack` argument to accept a path to a `.tech-stack.md` file instead of inline tokens.

**New invocation format**:
```
/execute prd=docs/app.prd.md ddd=docs/app.ddd.md stack=docs/app.tech-stack.md
```

**File format**:
- Markdown with sections (Frontend, Backend, Database, AI)
- Lines starting with `- ` define technology tokens
- Lines starting with `#` are comments
- Inline comments allowed after `#`

## Consequences

### Positive
- Consistency: all three inputs are now file-based
- Reusability: tech stack files can be versioned and shared
- Documentation: comments in the file explain each choice
- Flexibility: easier to toggle technologies by commenting/uncommenting

### Negative
- Breaking change: old inline syntax no longer works
- Extra file: users must create a tech-stack.md file

## Alternatives Considered

1. **Support both formats**: More complex parsing, confusing UX
2. **YAML/JSON format**: Less readable, unfamiliar to users already writing markdown PRDs
3. **Keep inline tokens**: Rejected per user request
