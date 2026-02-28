# Pull Request — Turn 8

## Summary

- Created new `/makefile-gen` skill for generating stack-aware Makefiles
- Generated Makefile for current project with all detected stack targets
- Skill auto-detects stack from project structure (nextjs, nestjs, drizzle, spring)
- Includes targets for tests, Docker, database, and local development

## Files Created

| File | Description |
|------|-------------|
| `.claude/skills/makefile-gen/SKILL.md` | New skill for Makefile generation |
| `Makefile` | Generated Makefile with 25+ targets |

## Makefile Targets

| Category | Targets |
|----------|---------|
| Setup | `install`, `setup`, `fresh`, `clean` |
| Dev | `dev`, `dev-web`, `dev-api` |
| Docker | `docker-up`, `docker-down`, `docker-logs`, `docker-reset` |
| Database | `db-generate`, `db-migrate`, `db-studio`, `db-reset` |
| Quality | `test`, `lint`, `typecheck`, `verify` |
| Build | `build`, `build-web`, `build-api`, `start-web`, `start-api` |

## Compliance Checklist

- [x] Skill has proper YAML frontmatter (name, description, triggers)
- [x] Makefile includes self-documenting help target
- [x] All targets have `## description` comments
- [x] ADR written (minimal — new skill, no architectural decisions)
- [x] Manifest generated

## Usage

```bash
make help          # Show all targets
make setup         # Install deps, start Docker, run migrations
make dev           # Start all dev servers
make verify        # Full quality check
```
