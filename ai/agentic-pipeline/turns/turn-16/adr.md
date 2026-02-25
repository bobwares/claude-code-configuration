# ADR: Turn 16 — Remove Memory Bank Integration

## Context

The memory bank (`.claude/memory/`) was designed to persist context across sessions, but analysis showed:
- Files were not being actively used (~1,900 tokens when loaded)
- Skills loaded memory but didn't provide clear value
- Added complexity without demonstrated benefit

## Decision

Remove all memory bank integration from skills, agents, and context files. The memory directory itself is preserved but no longer referenced by the pipeline.

## Consequences

**Positive:**
- Simpler skill implementations
- Reduced context token usage when skills run
- Clearer separation: turns handle provenance, not memory files

**Negative:**
- No automatic session-to-session context carryover
- Must rely on turn artifacts for historical context

**Mitigation:**
- `turns_index.csv` provides searchable history
- ADRs capture architectural decisions
- Memory bank can be re-enabled later if needed
