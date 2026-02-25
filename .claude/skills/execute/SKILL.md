---
name: execute
description: One-command full app build. Provide PRD path, DDD path, and tech stack file — the pipeline does the rest. Usage: /execute prd=<path> ddd=<path> stack=<path-to-tech-stack.md>
---

# Execute — Full Pipeline Trigger

You are the entry point for the agentic-pipeline's one-command build mode.

The user has supplied everything you need. **Do not ask any clarifying questions.** Parse the arguments, validate inputs, and drive the complete pipeline from PRD + DDD + tech stack all the way to a working, verified, committed codebase.

---

## Step 0: Parse Arguments

Parse the invocation arguments. Accepted formats:

```
/execute prd=specs/my-app.prd.md ddd=specs/my-app.ddd.md stack=specs/tech-stack.md
/execute prd=my-app ddd=my-app stack=my-app
/execute specs/prd.md specs/ddd.md specs/tech-stack.md
```

Resolution rules:
- **PRD file**: If path ends in `.md` and exists, use directly. If bare name like `my-app`, check `.claude/prds/my-app.prd.md`, then `specs/my-app.prd.md`
- **DDD file**: If path ends in `.md` and exists, use directly. If bare name, check `.claude/ddd/<name>.ddd.md`, then `specs/<name>.ddd.md`
- **Tech stack file**: If path ends in `.md` and exists, use directly. If bare name, check `.claude/stacks/<name>.tech-stack.md`, then `specs/<name>.tech-stack.md`

If any required input is missing or a file does not exist, report exactly what is missing and stop. Example:
```
❌ Cannot execute: Tech stack file not found.
   Tried: .claude/stacks/my-app.tech-stack.md, specs/my-app.tech-stack.md
   Fix: Create the tech stack file first.
```

---

## Step 1: Read and Validate Inputs

Read all three input files completely.

**Read the PRD** and confirm it contains:
- A clear problem statement
- At least one user story or goal
- Success criteria

**Read the DDD document** and confirm it contains:
- At least one bounded context
- Domain entities or aggregates

**Read the tech stack file** and confirm it contains:
- At least one technology selection
- Valid technology tokens (see Tech Stack File Format below)

**If any document is missing critical sections**, report what is incomplete and stop. The user must fix the document before executing.

Print a validation summary:
```
✅ PRD validated: <title> — <one-sentence summary>
✅ DDD validated: <N> bounded contexts, <N> aggregates, <N> entities
✅ Tech stack: <list of selected layers>

Starting full pipeline...
```

---

## Step 2: Initialize Turn Lifecycle (Pre-Execution) — MANDATORY

Before any code execution, complete all pre-execution steps of the turn lifecycle protocol.

### Step 2a: Resolve TURN_ID

```bash
TURNS_INDEX="./ai/agentic-pipeline/turns_index.csv"

if [ -f "$TURNS_INDEX" ]; then
  # Read max turn_id from CSV and add 1
  TURN_ID=$(tail -n +2 "$TURNS_INDEX" | cut -d',' -f1 | sort -n | tail -1)
  TURN_ID=$((TURN_ID + 1))
else
  # First turn — initialize
  TURN_ID=1
  mkdir -p "./ai/agentic-pipeline"
  echo "turn_id,started_at,finished_at,elapsed_seconds,branch,commit_sha,task_summary" > "$TURNS_INDEX"
fi

CURRENT_TURN_DIRECTORY="./ai/agentic-pipeline/turns/turn-${TURN_ID}"
```

### Step 2b: Create Turn Directory

```bash
mkdir -p "${CURRENT_TURN_DIRECTORY}"
```

### Step 2c: Record TURN_START_TIME

```bash
TURN_START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

### Step 2d: Write session_context.md

Create file: `${CURRENT_TURN_DIRECTORY}/session_context.md`

Document all loaded context values in a markdown table:
- TURN_ID
- TURN_START_TIME (ISO 8601 UTC)
- TARGET_PROJECT (absolute path)
- CURRENT_TURN_DIRECTORY
- PRD path
- DDD path
- Tech stack
- Active branch
- Task description (execute pipeline for the specified app)

**This file MUST exist before proceeding to Step 2e.**

### Step 2e: Create Turn Branch

Create a dedicated branch for this execution run. This provides isolation and full traceability for all changes made during this `/execute` invocation.

```bash
# Ensure we're on an up-to-date main
git checkout main
git pull origin main

# Create and push the turn branch
git checkout -b turn-${TURN_ID}
git push -u origin turn-${TURN_ID}
```

This branch becomes the working branch for this execution. All epic branches in Step 5b will be created FROM this turn branch.

**The turn branch MUST exist before proceeding to Step 3.**

---

## Step 3: Initialize Project Structure

Invoke the `project-init` skill with the resolved tech stack.

This skill scaffolds the monorepo directory structure, creates config files, and sets up the workspace so specialist agents have a consistent foundation to build on.

Wait for `project-init` to complete before proceeding.

---

## Step 4: Parse DDD into Domain Model

Invoke the `ddd-parse` skill with the DDD document path.

This skill reads the DDD document and outputs a structured domain model at `.claude/domain/model.json` containing:
- Bounded contexts with their aggregates
- Entities and their fields
- Value objects
- Domain events
- Relationships between aggregates

Wait for `ddd-parse` to complete before proceeding.

---

## Step 5a: Parse PRD into Epics

Invoke the `spec-prd-parse` skill with the PRD name AND the domain model.

`spec-prd-parse` will read `.claude/domain/model.json` to:
- Map user stories to domain aggregates
- Generate database tasks that match the DDD entities
- Ensure API tasks align with domain service boundaries
- Generate frontend tasks that reflect the domain language

This produces `.claude/epics/<app-name>/epic.md` with a full, DDD-aware task breakdown.

Wait for `spec-prd-parse` to complete before proceeding.

---

## Step 5b: Execute Epics

For each epic produced in Step 5a:

1. **Branch**: `git checkout -b epic/<epic-name>` (from turn-${TURN_ID} branch)
2. **Spawn orchestrator**: provide the full epic task breakdown and domain model as context
3. The orchestrator will:
   - Execute all tasks in dependency order using specialist agents
   - Run verify-all after each phase
   - Fix any failures before proceeding
4. **Wait** for orchestrator to report completion
5. **Run final verify-all** — confirm all checks pass
6. **Spawn git-guardian** to commit, push, and create PR

---

## Step 6: Complete Turn Lifecycle (Post-Execution) — MANDATORY

After all epics complete (success or failure), complete all post-execution steps of the turn lifecycle protocol. **These steps are mandatory even if execution failed.**

### Step 6a: Record TURN_END_TIME

```bash
TURN_END_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

### Step 6b: Write pull_request.md

Create file: `${CURRENT_TURN_DIRECTORY}/pull_request.md`

Use template: `.claude/templates/pr/pull_request_template.md`

Include:
- Turn summary (3–5 bullets of what was accomplished)
- Start and end timestamps
- Tasks executed (table with task name, agent used)
- Files added/modified (tables with metadata header descriptions)
- Compliance checklist

### Step 6c: Write adr.md

Create file: `${CURRENT_TURN_DIRECTORY}/adr.md`

Apply ADR policy:
- If architectural decisions were made → Full ADR using `.claude/templates/adr/adr_template.md`
- If no architectural decisions → Write: `No architectural decision made this turn — [description of what was done].`

**This step is mandatory every turn. A turn without adr.md is incomplete.**

### Step 6d: Write manifest.json

Create file: `${CURRENT_TURN_DIRECTORY}/manifest.json`

Include:
- turnId
- tasks array with agent, inputs, outputs (with SHA-256 hashes)
- provenance with startedAt, finishedAt, tools used
- outputs array with file paths and SHA-256 hashes

Validate against: `.claude/templates/turn/manifest.schema.json`

SHA-256 computation:
```bash
# macOS
shasum -a 256 <file> | cut -d' ' -f1
# Linux
sha256sum <file> | cut -d' ' -f1
```

### Step 6e: Update turns_index.csv

```bash
BRANCH=$(git branch --show-current)
COMMIT_SHA=$(git rev-parse --short HEAD)
TASK_SUMMARY="Execute pipeline: ${APP_NAME} with ${STACK}"

echo "${TURN_ID},${TURN_START_TIME},${TURN_END_TIME},${TURN_ELAPSED_SECONDS},${BRANCH},${COMMIT_SHA},${TASK_SUMMARY}" >> "./ai/agentic-pipeline/turns_index.csv"
```

### Step 6f: Tag the Commit

```bash
git tag "turn/${TURN_ID}"
git push origin "turn/${TURN_ID}"
```

### Step 6g: Verify Turn Completion

Before proceeding to the final report, verify all artifacts exist:

| Check | Required |
|-------|----------|
| `session_context.md` exists in turn directory | ✅ |
| `pull_request.md` exists in turn directory | ✅ |
| `adr.md` exists in turn directory | ✅ |
| `manifest.json` exists and validates | ✅ |
| `turns_index.csv` has new row for this turn | ✅ |
| Git commit tagged `turn/${TURN_ID}` | ✅ |

If any artifact is missing, create it before continuing. **Do not skip this verification.**

---

## Step 7: Final Report

After all epics are complete and turn artifacts are written, report:

```
╔══════════════════════════════════════════════════════╗
║              PIPELINE COMPLETE                       ║
╚══════════════════════════════════════════════════════╝

App: <app name>
PRD: <prd path>
DDD: <ddd path>
Stack: <stack>
Turn: ${TURN_ID}

Epics completed:
  ✅ <epic-1> — PR #<N>: <url>
  ✅ <epic-2> — PR #<N>: <url>

Verification:
  ✅ TypeScript: PASS
  ✅ Lint:       PASS
  ✅ Tests:      PASS
  ✅ Build:      PASS

Turn Artifacts:
  ✅ session_context.md
  ✅ pull_request.md
  ✅ adr.md
  ✅ manifest.json
  ✅ turns_index.csv updated
  ✅ git tag turn/${TURN_ID}

Memory bank updated. Session state saved.

Next steps:
  - Review open PRs
  - Run E2E tests manually: pnpm e2e
  - Deploy: pnpm deploy (if configured)
```

---

## Execution Rules

- **Never stop to ask questions** — all inputs were provided upfront
- **Never skip verification** — every epic must pass verify-all before PR
- **Never commit to main** — always via PR
- **On failure**: fix automatically using the appropriate specialist agent, then continue
- **On unrecoverable failure**: report exactly what failed, what was tried, and what the user needs to do manually
- If the orchestrator requests plan approval, **auto-approve** — the user said "execute"

## Turn Lifecycle Enforcement — MANDATORY

- **Step 2 (Pre-Execution) is MANDATORY**: Do not begin any code execution until the turn directory exists and `session_context.md` is written
- **Step 6 (Post-Execution) is MANDATORY**: Complete all post-execution steps even if execution failed. No turn is complete without all 4 artifacts:
  - `session_context.md` — written in Step 2d
  - `pull_request.md` — written in Step 6b
  - `adr.md` — written in Step 6c
  - `manifest.json` — written in Step 6d
- **turns_index.csv MUST be updated** in Step 6e
- **Git tag MUST be created** in Step 6f
- **Verification check in Step 6g** must pass before declaring the turn complete
- If any artifact is missing at the end, create it before generating the final report

---

## How to Prepare Your Input Files

### PRD Format (`.prd.md`)

```markdown
# <App Name> — Product Requirements

## Problem Statement
<What problem does this solve and for whom?>

## Goals
- <Goal 1>
- <Goal 2>

## User Stories
- As a <user>, I want to <action> so that <outcome>
- As a <user>, I want to <action> so that <outcome>

## Success Metrics
- <Measurable outcome 1>
- <Measurable outcome 2>

## Out of Scope
- <What is explicitly not included>

## Technical Constraints
- <Any hard constraints: auth provider, compliance, performance>
```

### DDD Format (`.ddd.md`)

```markdown
# <App Name> — Domain Model

## Bounded Contexts

### <Context Name>
**Purpose**: <What domain concern this context owns>

#### Aggregates

##### <AggregateName>
**Root Entity**: <EntityName>
**Invariants**: <Business rules this aggregate enforces>

**Entities**:
- `<EntityName>`: <purpose>
  - `id`: UUID
  - `<field>`: <type> — <description>

**Value Objects**:
- `<ValueObjectName>`: <fields and constraints>

**Domain Events**:
- `<EventName>`: emitted when <trigger>

#### Domain Services
- `<ServiceName>`: <what cross-aggregate operation it performs>

## Relationships
- <ContextA> → <ContextB>: <relationship type and reason>
```

### Tech Stack File Format (`.tech-stack.md`)

```markdown
# <App Name> — Tech Stack

## Frontend
- nextjs          # Next.js 15 App Router (app/web/)
- shadcn          # shadcn/ui + Tailwind CSS

## Backend
- nestjs          # NestJS REST API (app/api/)
# - spring        # Java Spring Boot (services/enterprise/) — uncomment if needed

## Database
- drizzle         # Drizzle ORM + PostgreSQL (packages/database/)

## AI (optional)
# - ai            # Vercel AI SDK — uncomment if needed
```

**Valid tokens**:

| Token | What It Installs |
|-------|-----------------|
| `nextjs` or `next` | Next.js 15 App Router (app/web/) |
| `nestjs` or `nest` | NestJS REST API (app/api/) |
| `spring` or `springboot` | Java Spring Boot (services/enterprise/) |
| `drizzle` | Drizzle ORM + PostgreSQL (packages/database/) |
| `shadcn` or `ui` | shadcn/ui + Tailwind CSS (in app/web/) |
| `ai` or `vercel-ai` | Vercel AI SDK (in app/web/ and/or app/api/) |

**Parsing rules**:
- Lines starting with `#` are comments (ignored)
- Lines starting with `- ` define a technology token
- Inline comments after `#` are allowed: `- nextjs  # comment`
- Empty lines are ignored

### Full Example Invocation

```
/execute prd=specs/taskflow.prd.md ddd=specs/taskflow.ddd.md stack=specs/taskflow.tech-stack.md
```
