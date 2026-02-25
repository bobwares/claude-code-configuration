# ADR — Turn 11: Restructure Monorepo to app/ Directory

## Status

Accepted

## Context

The original monorepo structure placed application directories (`apps/`, `packages/`, `services/`) at the project root alongside configuration files. The user requested:

1. Application code should live under `app/` directory (`app/api`, `app/web`)
2. Makefile should stay at the project root
3. All other config files should move into `app/`

## Decision

**Restructure the monorepo to use `app/` as the application root:**

```
Before:                        After:
├── apps/                      ├── Makefile (stays)
│   ├── api/                   ├── app/
│   └── web/                   │   ├── api/
├── packages/                  │   ├── web/
├── package.json               │   ├── packages/
├── turbo.json                 │   ├── package.json
├── Makefile                   │   ├── turbo.json
└── docker-compose.yml         │   └── docker-compose.yml
```

**Key changes:**
- `apps/` → `app/` (flat, not `app/apps/`)
- `packages/` → `app/packages/`
- All pnpm/turbo config files → `app/`
- Makefile stays at root with `APP_DIR := app` variable

## Rationale

1. **Clear separation**: Application code is isolated from infrastructure files (`.claude/`, `ai/`, `docs/`)
2. **Makefile at root**: Developers expect `make` commands to work from project root
3. **Simpler paths**: `app/api` is cleaner than `apps/api` (singular)
4. **Workspace integrity**: pnpm workspace still works with updated `pnpm-workspace.yaml`

## Consequences

### Positive
- Cleaner project root — only Makefile, README, and CLAUDE.md visible
- Application code is self-contained in `app/`
- Easier to understand project structure at a glance

### Negative
- All path references in skills, agents, and context files needed updates (20+ files)
- Developers must remember to `cd app/` for direct pnpm commands
- Existing git history shows the move as deletions + additions

### Migration Notes
- Makefile now uses `cd $(APP_DIR) &&` prefix for all commands
- `pnpm-workspace.yaml` updated to `api`, `web`, `packages/*` (no `apps/` prefix)
- All skill pattern matching updated in `skill-rules.json`

## Related

- Turn 7: Original project-init scaffold (created `apps/` structure)
- Turn 8: Makefile generation (now updated for `app/` structure)
