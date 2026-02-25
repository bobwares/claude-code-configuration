# ADR: Turn Branch Creation in /execute Skill

**Status**: Accepted
**Date**: 2026-02-24
**Turn**: 1

## Context

The `/execute` skill runs the full pipeline (PRD + DDD + stack to working app). Previously, it created epic branches directly from main. The user requested that each `/execute` invocation create a dedicated `turn-{TURN_ID}` branch for better isolation and traceability.

## Decision

Add **Step 2e** to the `/execute` skill that creates a `turn-${TURN_ID}` branch immediately after resolving the turn ID and before any code execution.

**Sequence**:
1. Step 2a-d: Resolve turn ID, create directory, record time, write session_context.md
2. **Step 2e (NEW)**: Checkout main, pull, create `turn-${TURN_ID}` branch, push
3. Step 3+: Proceed with project initialization and code generation

**Epic branches** (Step 5b) are now created from the turn branch instead of main.

## Consequences

### Positive
- Full isolation: each `/execute` run has its own branch
- Traceability: branch name matches turn ID for easy correlation
- Rollback: entire execution can be discarded by deleting the turn branch
- Parallel runs: multiple `/execute` invocations won't conflict

### Negative
- Additional branch per execution (minor overhead)
- Requires network access to push the branch before proceeding

## Alternatives Considered

1. **Modify `turn-init.sh` hook**: Would create branch on every prompt, not just `/execute` — too broad
2. **Create dedicated hook for `/execute`**: More complex setup, harder to maintain
3. **Keep current behavior**: No isolation between executions — rejected per user request
