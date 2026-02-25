# Agent Coordination

Rules for multiple agents working in parallel within the same epic.

## Model Selection — Cost Optimization

When spawning agents, specify the appropriate model to minimize token usage and cost.

| Task Complexity | Model | Use Cases |
|----------------|-------|-----------|
| **Simple** | `haiku` | File operations, simple edits, git operations, doc generation |
| **Medium** | `sonnet` | Feature implementation, test writing, API development, refactoring |
| **Complex** | `opus` | Architecture decisions, code review, security audits, orchestration |

### Examples

```typescript
// Simple task — use haiku
Task(agent: "doc-generator", model: "haiku", prompt: "Add JSDoc to utils.ts")
Task(agent: "git-guardian", model: "haiku", prompt: "Create commit")

// Medium task — use sonnet (default)
Task(agent: "nextjs-engineer", prompt: "Implement dashboard page")
Task(agent: "test-writer", prompt: "Write unit tests for UserService")

// Complex task — use opus
Task(agent: "code-architect", model: "opus", prompt: "Design API schema")
Task(agent: "code-reviewer", model: "opus", prompt: "Review PR for security")
```

### Default Model by Agent

| Agent | Default Model | Rationale |
|-------|---------------|-----------|
| `orchestrator` | opus | Coordinates complex workflows |
| `code-architect` | opus | Architectural decisions |
| `code-reviewer` | opus | Deep code analysis |
| `security-auditor` | opus | Security requires thoroughness |
| `git-guardian` | haiku | Straightforward git operations |
| `doc-generator` | haiku | Template-based generation |
| All others | sonnet | Balanced capability/cost |

---

## Parallelism Principle

Agents working on **different file scopes** never conflict. Assign each agent a clear ownership.

## Default File Scope Assignments

| Agent | Owns |
|-------|------|
| `drizzle-dba` | `app/packages/database/schema/`, `app/packages/database/migrations/` |
| `nestjs-engineer` | `app/api/src/modules/<feature>/` |
| `spring-engineer` | `app/services/enterprise/src/main/java/` |
| `nextjs-engineer` | `app/web/src/app/<route>/`, `app/web/src/components/` |
| `test-writer` | `**/*.test.ts`, `**/*.spec.ts`, `app/e2e/` |
| `doc-generator` | `**/*.md`, JSDoc in any file |

## Coordination Rules

1. Each agent only modifies files within its assigned scope
2. Shared types (`packages/types/`) — one agent creates, others import
3. If a shared file needs updating: **one agent handles it, others wait**
4. Each agent commits after each logical unit of work

## Sync Protocol

```bash
# Before starting work on a shared branch
git pull --rebase origin epic/<name>

# After completing a logical unit
git add <files-in-my-scope>
git commit -m "feat(scope): description"
git pull --rebase origin epic/<name>
git push
```

## Conflict Resolution

- Agents **never auto-resolve conflicts**
- On conflict: stop work, report the conflict to the orchestrator
- Human reviews and resolves; agent continues after resolution

