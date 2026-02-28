# Pull Request — Turn 11

## Summary

Restructured the monorepo to place all application code under `app/` directory while keeping the Makefile at root.

**Key changes:**
- Renamed `apps/` to `app/` (flat structure: `app/api`, `app/web`)
- Moved `packages/` to `app/packages/`
- Moved all config files (`package.json`, `turbo.json`, etc.) into `app/`
- Updated Makefile to run commands from `app/` directory
- Updated 20+ skill files, agent files, and context files with new paths

## Files Modified

| File | Change |
|------|--------|
| `app/` (directory) | Created — moved from `apps/` |
| `app/api/` | Moved from `apps/api/` |
| `app/web/` | Moved from `apps/web/` |
| `app/packages/` | Moved from `packages/` |
| `app/package.json` | Moved from root |
| `app/pnpm-workspace.yaml` | Moved + updated paths |
| `app/turbo.json` | Moved from root |
| `app/tsconfig.base.json` | Moved from root |
| `app/docker-compose.yml` | Moved from root |
| `Makefile` | Updated — added APP_DIR variable, cd to app/ |
| `.claude/skills/project-init/SKILL.md` | Updated paths to app/ structure |
| `.claude/skills/makefile-gen/SKILL.md` | Updated paths to app/ structure |
| `.claude/agents/*.md` | Updated 5 agent files with new paths |
| `.claude/context/*.md` | Updated 5 context files with new paths |
| `.claude/skills/*.md` | Updated 8 skill files with new paths |
| `.claude/hooks/skill-rules.json` | Updated all directory mappings |
| `.claude/templates/stack/tech-stack.template.md` | Updated paths |
| `CLAUDE.md` | Updated project structure + commands |

## New Directory Structure

```
project-root/
├── Makefile                    # Stays at root
├── app/                        # All application code and config
│   ├── api/                    # NestJS
│   ├── web/                    # Next.js
│   ├── packages/
│   │   ├── database/           # Drizzle
│   │   └── types/              # Shared types
│   ├── package.json
│   ├── pnpm-workspace.yaml
│   ├── turbo.json
│   ├── tsconfig.base.json
│   └── docker-compose.yml
├── .claude/                    # Pipeline config
├── ai/                         # Turn artifacts
└── docs/                       # Documentation
```

## Compliance Checklist

- [x] All path references updated across 20+ files
- [x] Makefile updated with APP_DIR variable
- [x] Skills updated (project-init, makefile-gen, etc.)
- [x] Agent files updated
- [x] Context files updated
- [x] skill-rules.json updated
- [x] CLAUDE.md updated
