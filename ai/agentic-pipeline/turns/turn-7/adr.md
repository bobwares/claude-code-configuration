# ADR - Turn 7: Monorepo Structure with pnpm + Turbo

**Date**: 2026-02-24T22:43:45Z
**Agent**: AI Coding Agent (claude-opus-4-5)
**Status**: Accepted

## Context

The project needed a full-stack application structure to support Next.js frontend, NestJS backend, and shared database/types packages. A decision was needed on how to organize the codebase.

## Decision

Adopt a **pnpm workspaces + Turborepo** monorepo structure with the following layout:

```
apps/
  web/     # Next.js 15 App Router
  api/     # NestJS REST API
packages/
  database/  # Drizzle ORM + PostgreSQL
  types/     # Shared TypeScript types
services/    # (Reserved for Spring Boot)
```

## Rationale

1. **pnpm workspaces**: Efficient disk usage via content-addressable storage, strict dependency isolation
2. **Turborepo**: Fast incremental builds, task caching, simple configuration
3. **Separation of concerns**: Frontend, backend, and shared packages in dedicated directories
4. **Workspace references**: Apps can import `database` and `types` packages directly

## Alternatives Considered

### Option 1: Nx Monorepo
- **Pros**: More features, better caching, generators
- **Cons**: Heavier learning curve, more configuration
- **Rejected because**: Turbo is simpler and sufficient for this project size

### Option 2: Separate Repositories
- **Pros**: Full isolation, independent deployment
- **Cons**: Harder to share code, version coordination overhead
- **Rejected because**: Shared types and database schema benefit from co-location

## Consequences

### Positive
- Single repository for all related code
- Shared TypeScript configuration
- Easy cross-package imports with workspace protocol
- Fast builds with Turbo caching

### Negative
- Larger repository size over time
- All teams need to understand monorepo tooling

### Mitigations
- Use `.gitignore` to exclude build artifacts
- Document monorepo conventions in CLAUDE.md
