# AGENTS.md — agentic-pipeline

This file is the global project memory. Read it at the start of every session.

---

## Turn Lifecycle (New)

Every coding task is a **turn** — a structured 10-step protocol with full provenance tracking.

```
PRE-EXECUTION                   EXECUTION              POST-EXECUTION
─────────────────────────────   ─────────────────────  ──────────────────────────────────
Step 1: Resolve TURN_ID         Step 5: Execute tasks  Step 6: Record end time
Step 2: Create turn directory                          Step 7: Write pull_request.md
Step 3: Write session_context                          Step 8: Write adr.md (mandatory)
Step 4: Record start time                              Step 9: Write manifest.json
                                                       Step 10: Update turns_index.csv
```

Every turn produces **4 artifacts** in `./ai/agentic-pipeline/turns/turn-${TURN_ID}/`:

| Artifact | Purpose |
|----------|---------|
| `session_context.md` | Table of all loaded context variables |
| `pull_request.md` | Files changed, tasks executed, compliance checklist |
| `adr.md` | Architecture Decision Record (full or minimal — mandatory) |
| `manifest.json` | SHA-256 hashes of all output files (validated against schema) |

---

## MANDATORY: Automatic Turn Tracking

**Every prompt submission automatically creates a turn.** The `turn-init.sh` hook handles pre-execution:

```
User submits prompt
       ↓
turn-init.sh runs automatically:
  • Resolves TURN_ID (increments from last turn)
  • Creates ./ai/agentic-pipeline/turns/turn-${TURN_ID}/
  • Writes session_context.md with prompt and metadata
       ↓
Claude processes prompt
       ↓
Claude MUST complete post-execution (see below)
```

### Post-Execution: YOUR Responsibility

**After completing ANY response that modifies files**, you MUST:

1. **Write `pull_request.md`** to the turn directory:
   - Summary of changes (3-5 bullets)
   - Files modified (table with paths)
   - Compliance checklist

2. **Write `adr.md`** to the turn directory:
   - Full ADR if architectural decisions were made
   - Minimal: `No architectural decision made this turn — [description].`

3. **Write `manifest.json`** to the turn directory:
   - SHA-256 hashes of all modified files
   - Provenance (start time, end time, tools used)

4. **Update `turns_index.csv`**:
   ```bash
   echo "${TURN_ID},${START_TIME},${END_TIME},${ELAPSED},${BRANCH},${COMMIT_SHA},${SUMMARY}" >> ./ai/agentic-pipeline/turns_index.csv
   ```

### Skip Post-Execution ONLY When

- Pure questions with no file changes
- Clarification requests
- Error messages / diagnostics
- Reading files without modification

### Finding Current Turn

The turn directory was created by the hook. To find it:
```bash
ls -td ./ai/agentic-pipeline/turns/turn-* | head -1
```

Or read the most recent `session_context.md` for the TURN_ID.

---

## Governance Standards (New)

Applied to **every file** written or modified. See `.claude/context/context_governance.md`.

### Metadata Headers (mandatory on all source files)

```typescript
/**
 * App: MyApp
 * Package: apps/api/src/modules/tasks
 * File: tasks.service.ts
 * Version: 0.2.0
 * Turns: 1, 3
 * Author: AI Coding Agent (claude-opus-4-5)
 * Date: 2026-02-22T14:35:00Z
 * Exports: TasksService
 * Description: Task domain service — validates assignment rules and emits domain events
 */
```

### Semantic Versioning (per file)

| Change | Bump |
|--------|------|
| New file | `0.1.0` |
| Bug fix / refactor | PATCH: `0.1.1` |
| New feature | MINOR: `0.2.0` |
| Breaking change | MAJOR: `1.0.0` |

### Commit Format

```
AI Coding Agent Change:
- Implement TaskAssignmentService with domain validation
- Emit TaskAssigned event on assignee change
- Write 14 unit tests for assignment edge cases
- Document repository pattern decision in ADR turn-3
```

### Git Tagging

After every turn: `git tag turn/${TURN_ID} && git push origin turn/${TURN_ID}`

---

## Architecture Decision Records (New)

**Every turn requires an `adr.md`** — full or minimal. No exceptions.

- Full ADR: when architectural decisions are made (technology choices, API changes, pattern selections, infrastructure)
- Minimal: `No architectural decision made this turn — [what was done instead].`

Template: `.claude/templates/adr/adr_template.md`

Significant ADRs are also added to `.claude/memory/decisionLog.md`.

---

## Memory Bank

The memory bank lives in `.claude/memory/`. Always load with `/session-start`.

| File | Purpose | Update Frequency |
|------|---------|-----------------|
| `projectContext.md` | Project identity, stack, key URLs | Rarely (stable) |
| `activeContext.md` | Current branch, WIP, next step | Every session |
| `progress.md` | Epics/tasks status table | Every session |
| `decisionLog.md` | Key ADR summaries for long-term memory | On full ADRs |
| `conventions.md` | Project-specific patterns | As discovered |
| `sessionHistory.md` | Session summaries log | End of every session |

---

## Context Files (New)

7 context files are loaded at session start (auto-loaded by SessionStart hook):

| File | Purpose |
|------|---------|
| `context_container.md` | Base directories and path variable definitions |
| `context_session.md` | Turn timing and directory variables |
| `context_conventions.md` | Naming, templating, git conventions |
| `context_governance.md` | Mandatory coding standards |
| `context_orchestration.md` | Complete 10-step turn lifecycle |
| `context_adr.md` | ADR policy and decision matrix |
| `context_skills.md` | Available skills and invocation guide |

---

## Templates (New)

| Template | Path | Used By |
|----------|------|---------|
| ADR | `.claude/templates/adr/adr_template.md` | adr skill, session-end |
| Session context | `.claude/templates/contexts/session_context.md` | orchestration Step 3 |
| Pull request | `.claude/templates/pr/pull_request_template.md` | orchestration Step 7, session-end |
| Manifest schema | `.claude/templates/turn/manifest.schema.json` | orchestration Step 9 |
| Metadata header | `.claude/templates/governance/metadata_header.txt` | governance skill |
| Branch naming | `.claude/templates/governance/branch_naming.md` | governance skill |
| Commit message | `.claude/templates/governance/commit_message.md` | governance skill |

---

## Workflow: Spec → Code → Ship

```
1. /spec-prd-new       → Define what to build (PRD)
2. /spec-prd-parse     → Break PRD into DDD-aligned epics and tasks
3. /spec-epic-start    → Begin implementation (spawns orchestrator + turn lifecycle)
4. /spec-task-next     → Get next task
5. /verify-all         → Quality gate before every PR
6. /git-commit-push-pr → Ship it
```

### Or, one command:

```
/execute prd=docs/app.prd.md ddd=docs/app.ddd.md stack=nextjs+nestjs+drizzle+shadcn
```

---

## Agent Team

| Agent | Role | Model |
|-------|------|-------|
| `orchestrator` | Master coordinator — 5-phase workflow, delegates to specialists | opus |
| `code-architect` | System design, API contracts, database schema | opus |
| `nextjs-engineer` | Next.js 15 App Router, server components, server actions | sonnet |
| `nestjs-engineer` | NestJS modules, guards, interceptors, DTOs | sonnet |
| `spring-engineer` | Spring WebFlux, R2DBC, WebClient, reactive APIs | sonnet |
| `drizzle-dba` | Drizzle ORM schema, queries, migrations, PostgreSQL | sonnet |
| `ai-engineer` | Vercel AI SDK, streaming, tool calls, embeddings | sonnet |
| `code-reviewer` | Senior-engineer code review checklist | opus |
| `test-writer` | TDD, Vitest unit tests, Playwright E2E | sonnet |
| `verify-app` | Full quality gate (types + lint + test + build) | sonnet |
| `security-auditor` | OWASP top 10, secrets scanning, auth review | opus |
| `doc-generator` | JSDoc, README, OpenAPI/Swagger docs | haiku |
| `git-guardian` | Conventional commits, PR creation, branch management | haiku |
| `memory-bank` | Memory bank management, session state | haiku |

---

## Skills Quick Reference

**Domain skills** (auto-suggested by skill-eval hook):
`nextjs-patterns`, `nestjs-patterns`, `spring-patterns`, `drizzle-patterns`, `shadcn-patterns`, `vercel-ai-patterns`, `testing-patterns`, `react-ui-patterns`, `api-design-patterns`, `systematic-debugging`

**Governance skills** (always active):
`governance`, `adr`

**Workflow skills** (manual invocation):

| Skill | Command | Description |
|-------|---------|-------------|
| execute | `/execute prd=... ddd=... stack=...` | One-command: PRD+DDD+stack → working app |
| ddd-parse | `/ddd-parse docs/app.ddd.md` | Parse DDD doc into model.json |
| project-init | `/project-init nextjs+nestjs+drizzle` | Scaffold monorepo |
| spec-prd-new | `/spec-prd-new` | Start writing a new PRD |
| spec-prd-parse | `/spec-prd-parse <name>` | Generate epic from PRD |
| spec-epic-start | `/spec-epic-start <name>` | Launch epic |
| spec-task-next | `/spec-task-next` | Advance to next task |
| memory-init | `/memory-init` | Initialize memory bank |
| session-start | `/session-start` | Load context + orient |
| session-end | `/session-end` | Complete turn artifacts + update memory |
| verify-all | `/verify-all` | typecheck + lint + test + build |
| test-and-fix | `/test-and-fix` | Run tests, auto-fix |
| security-scan | `/security-scan` | OWASP review |
| git-commit-push-pr | `/git-commit-push-pr` | Commit + push + PR |
| git-quick-commit | `/git-quick-commit` | Fast checkpoint |
| fix-issue | `/fix-issue <description>` | Fix specific issue |

---

## Non-Negotiable Code Standards

See `.claude/rules/tech-standards.md` for full detail. `.claude/context/context_governance.md` for governance rules.

### TypeScript
- `"strict": true` in every tsconfig
- No `any` — use `unknown` + type guards
- Metadata header on every `.ts`/`.tsx` file

### React / Next.js
- Server Components by default; `'use client'` only when needed
- Always handle: loading → error → empty → success

### NestJS
- Every DTO validated with Zod via `nestjs-zod`
- Every endpoint documented with `@ApiOperation` + `@ApiResponse`

### Spring Boot
- `@Valid` on all controller inputs
- `@Transactional(readOnly = true)` on query methods
- DTOs as Java Records — never expose JPA entities directly

### Database
- UUID primary keys with `defaultRandom()`
- All timestamps `{ withTimezone: true }`
- All multi-step writes in `db.transaction()`

### Security
- No secrets in source code — ever
- Validate all user input at every entry point

### Git
- No direct commits to `main` — PRs only
- Branch: `<type>/<description>[-<task-id>]`
- Commit: `AI Coding Agent Change:` + 3-5 imperative bullets
- Tag every turn: `turn/${TURN_ID}`

---

## Project Structure

```
project-root/
├── ai/
│   ├── agentic-pipeline/          # Turn artifacts
│   │   ├── turns_index.csv        # Turn registry
│   │   └── turns/
│   │       └── turn-N/
│   │           ├── session_context.md
│   │           ├── pull_request.md
│   │           ├── adr.md
│   │           └── manifest.json
│   └── context/
│       └── project_context.md
├── apps/
│   ├── web/                        # Next.js 15 App Router
│   └── api/                        # NestJS REST API
├── services/
│   └── enterprise/                 # Java Spring Boot (optional)
├── packages/
│   ├── database/                   # Drizzle ORM
│   ├── types/                      # Shared TypeScript types
│   └── ui/                         # Shared UI components (optional)
├── .claude/
│   ├── agents/                     # 14 specialist agents
│   ├── skills/                     # 36 skills (domain + workflow + governance)
│   ├── context/                    # 7 context reference files
│   ├── hooks/                      # skill-eval, audit-log
│   ├── memory/                     # 6-file memory bank
│   ├── rules/                      # 3 rule files
│   └── templates/                  # ADR, PR, manifest, governance templates
├── CLAUDE.md                       # This file
├── USAGE.md                        # How to use the one-command pipeline
└── package.json                    # pnpm workspaces root
```

---

## Key Commands

```bash
pnpm dev              # Start all services (requires turbo)
pnpm dev:web          # Start Next.js only
pnpm dev:api          # Start NestJS only
pnpm build            # Production build
pnpm test             # Run all tests
pnpm lint             # Lint all packages
pnpm typecheck        # TypeScript check all packages
pnpm db:migrate       # Run Drizzle migrations
pnpm db:studio        # Open Drizzle Studio
```

## .claude Configuration Mirror

This section mirrors the runtime configuration stored under `.claude` so the global memory and tooling references stay in sync with the actual hooks, rules, permissions, skills, and agents that the pipeline uses.

### Skills (from `.claude/context/context_skills.md`)

- **Domain knowledge (auto-activated by the `skill-eval` hook when Claude sees related keywords/paths):**
  - `api-design-patterns`
  - `drizzle-patterns`
  - `nestjs-patterns`
  - `nextjs-patterns`
  - `react-ui-patterns`
  - `shadcn-patterns`
  - `spring-patterns`
  - `testing-patterns`
  - `vercel-ai-patterns`
  - `systematic-debugging`

- **Governance (always active for every turn):**
  - `governance` — enforces metadata headers, versioning, git standards, and other governance checks.
  - `adr` — drives architecture decision recording and post-execution ADR requirements.

- **Workflow (manually invoked via `/skill-name` in Claude Code):**

  | Skill | Command | Description |
  |-------|---------|-------------|
  | execute | `/execute prd=... ddd=... stack=...` | Single command that turns PRD + DDD + stack into working scaffold |
  | ddd-parse | `/ddd-parse docs/app.ddd.md` | Parse a DDD doc into `model.json` |
  | project-init | `/project-init nextjs+nestjs+drizzle` | Scaffold the monorepo |
  | spec-prd-new | `/spec-prd-new` | Create a fresh PRD |
  | spec-prd-parse | `/spec-prd-parse <name>` | Turn a PRD into epics/tasks |
  | spec-epic-start | `/spec-epic-start <name>` | Launch an epic workflow |
  | spec-task-next | `/spec-task-next` | Display/advance the current task |
  | memory-init | `/memory-init` | Initialize the six-file memory bank |
  | session-start | `/session-start` | Load context files, show status, and prep hooks |
  | session-end | `/session-end` | Wrap up, update memory, and close the turn |
  | verify-all | `/verify-all` | Run typecheck, lint, tests, and build |
  | test-and-fix | `/test-and-fix` | Run tests and auto-fix failures |
  | security-scan | `/security-scan` | Perform OWASP-style security review |
  | git-commit-push-pr | `/git-commit-push-pr` | Commit changes, push, and open PR |
  | git-quick-commit | `/git-quick-commit` | Fast checkpoint commit |
  | git-checkpoint | `/git-checkpoint` | Save current work (checkpoint) |
  | git-rollback | `/git-rollback` | Rollback to a previous checkpoint |
  | git-undo | `/git-undo` | Undo the last Claude commit |
  | context-prime | `/context-prime` | Reload the full project context |
  | fix-issue | `/fix-issue <description>` | Fix a specific issue |
  | diagnose-issue | `/diagnose-issue <problem>` | Diagnose a problem and create an issue doc |
  | mode | `/mode <mode-name>` | Switch Claude’s operating mode |

> Each skill is a directory under `.claude/skills/` containing a `SKILL.md` manifest with `name`, `description`, and `triggers`. The manifest validates before invocation to keep directory names, YAML names, and descriptions aligned.

### Agents (front matter from `.claude/agents/*.md`)

| Agent | Default model | Allowed tools | Summary |
| --- | --- | --- | --- |
| `orchestrator` | `claude-opus-4-5` | Task, Read, Bash, Glob, Grep | Master coordinator — never codes; always plans, delegates, verifies, ships, and updates memory. |
| `code-architect` | `claude-opus-4-5` | Read, Bash, Glob, Grep, Write, Edit | Designs system architecture, APIs, schemas, and module boundaries. |
| `code-reviewer` | `claude-opus-4-5` | Read, Glob, Grep, Bash | Opinionated senior review with checklist after implementation. |
| `security-auditor` | `claude-opus-4-5` | Read, Bash, Glob, Grep | OWASP-focused security review (auth, secrets, injection). |
| `ai-engineer` | `claude-sonnet-4-5` | Read, Write, Edit, Bash, Glob, Grep | Builds Vercel AI SDK, tool calls, embeddings, streaming UIs. |
| `drizzle-dba` | `claude-sonnet-4-5` | Read, Write, Edit, Bash, Glob, Grep | Drizzle + PostgreSQL schema, queries, migrations, transactions. |
| `nestjs-engineer` | `claude-sonnet-4-5` | Read, Write, Edit, Bash, Glob, Grep | NestJS modules, services, controllers, DTOs, guards, pipes. |
| `nextjs-engineer` | `claude-sonnet-4-5` | Read, Write, Edit, Bash, Glob, Grep | Next.js 15 App Router, server components, route handlers. |
| `spring-engineer` | `claude-sonnet-4-5` | Read, Write, Edit, Bash, Glob, Grep | Spring WebFlux + R2DBC reactive services, controllers, WebClient. |
| `test-writer` | `claude-sonnet-4-5` | Read, Write, Edit, Bash, Glob, Grep | TDD specialist—Vitest/JUnit/Playwright tests before fixes. |
| `doc-generator` | `claude-haiku-4-5` | Read, Write, Edit, Bash, Glob, Grep | Documents README, OpenAPI, JSDoc, changelogs. |
| `memory-bank` | `claude-haiku-4-5` | Read, Write, Edit, Bash | Manages session history, progress, and context updates. |
| `git-guardian` | `claude-haiku-4-5` | Bash, Read | Handles conventional commits, pushes, PRs; never touches `main`. |
| `verify-app` | `claude-sonnet-4-5` | Bash, Read, Task | Full quality gate (typecheck, lint, tests, builds) and spawns fix agents when needed. |

> Use the `Task` tool to spawn these agents with the appropriate prompt, scope, and model as defined in `.claude/agents/<agent>.md`. Follow the default scope assignments in `agent-coordination.md` when sharing work.

### Permissions (derived from `.claude/settings.json`)

- **Allow**: `Bash(git:*)`, `Bash(gh:*)`, `Bash(pnpm:*)`, `Bash(npm:*)`, `Bash(npx:*)`, `Bash(node:*)`, `Bash(tsc:*)`, `Bash(mvn:*)`, `Bash(gradle:*)`, `Bash(java:*)`, `Bash(docker compose:*)`, `Bash(docker:*)`, `Bash(psql:*)`, `Bash(ls:*)`, `Bash(find:*)`, `Bash(cat:*)`, `Bash(echo:*)`, `Bash(mkdir:*)`, `Bash(cp:*)`, `Bash(mv:*)`, `Bash(chmod:*)`, `Bash(date:*)`, `Bash(jq:*)`, `Bash(curl:*)`, `Bash(grep:*)`, `Bash(sed:*)`, `Bash(awk:*)`, `Bash(wc:*)`, `Bash(head:*)`, `Bash(tail:*)`, `Bash(sort:*)`, `Bash(uniq:*)`, `Bash(diff:*)`, `Bash(which:*)`, `Bash(pwd:*)`, `Bash(cd:*)`.
- **Deny**: `Bash(rm -rf /:*)`, `Bash(git push --force main:*)`, `Bash(git push -f main:*)`, `Bash(npm publish:*)`, `Bash(pnpm publish:*)`.
- **Ask**: `Bash(git push:*)`, `Bash(git commit:*)`, `Bash(sudo:*)`, `Read(.env*)`, `Read(**/.env*)`, `Read(~/.ssh/**)`.

> Permission enforcement is handled at the CLI layer (see `.claude/settings.json`). No hidden clusters of permissions exist outside this explicit allow/deny/ask list.

### Hooks (from `.claude/settings.json` + `.claude/hooks/`)

- **SessionStart**: Prints `=== AGENTIC-PIPELINE SESSION START ===`, lists the seven context files (`context_container`, `context_session`, `context_conventions`, `context_governance`, `context_orchestration`, `context_adr`, `context_skills`), reports the last/next turn, and states governance is active. Runs automatically when a session begins.
- **UserPromptSubmit**: Runs `bash "$(pwd)/.claude/hooks/turn-init.sh"` to create the turn directory/artifacts, then pipes the user prompt into `bash "$(pwd)/.claude/hooks/skill-eval.sh"` so domain skills can force-load when keywords or paths match.
- **PreToolUse**:
  - Hook `matcher: Edit|MultiEdit|Write` ensures the current branch is not `main` or `master`, aborting edits when the constraint fails.
  - Hook `matcher: Bash` runs `bash "$(pwd)/.claude/hooks/audit-log.sh"` before every Bash call to capture the command in the audit log.
- **PostToolUse** (Edit/MultiEdit/Write matcher):
  - Prettier auto-formats `.ts`, `.tsx`, `.js`, `.jsx`, `.json`, `.css`, `.md` files immediately after they are written.
  - When `package.json` changes, runs `pnpm install --silent` (or `npm install` fallback) in the modified directory.
  - When a `.test.ts`, `.test.tsx`, or `.spec.ts` file changes, kicks off `pnpm test --run --reporter=dot` and shows the tail end of the output.
  - When a `.ts` or `.tsx` file changes, runs `pnpm typecheck` and displays the first few lines that match `error` or `✓`.

### Rules (under `.claude/rules/`)

- `agent-coordination.md`: Defines agent-to-scope ownership, default models (haiku for simple tasks, sonnet for general work, opus for complex/critical reviews), sync protocol, progress-file communication, and conflict-resolution guardrails.
- `branch-operations.md`: Enforces branch naming (`epic/`, `feature/`, `fix/`, `chore/`, `docs/`, `refactor/`), forbids direct commits/pushes to `main`, requires PRs with passing CI, mandates pulling before pushing, and specifies the conventional commit format plus `Co-Authored-By: Claude`.
- `tech-standards.md`: Captures the full TypeScript/React/NestJS/Spring/Drizzle/Security/Testing/Git standards that must be obeyed before merging.

> These rule files are authoritative; update AGENTS.md whenever their scope or requirements change so the human-readable memory matches the actual enforcement logic.
