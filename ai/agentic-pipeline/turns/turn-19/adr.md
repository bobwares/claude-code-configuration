# ADR: Turn-Level Skill and Agent Execution Trace

- **Turn**: 19
- **Date**: 2026-02-25T21:53:31Z
- **Status**: Accepted

## Context

Turn artifacts did not provide a single, explicit record of which skills and agents were executed. This made post-turn auditing harder, especially when comparing prompt requests, actual skills used, and specialist-agent participation.

## Decision

Introduce `execution_trace.json` as a turn artifact and require it to be updated by session end.

- `turn-init.sh` initializes `execution_trace.json` in each turn directory.
- Prompt slash commands are parsed into `requestedSkillsFromPrompt`.
- Session-end guidance now requires populating `skillsExecuted` and `agentsExecuted`.
- `pull_request.md` template and `manifest.schema.json` include execution trace fields.

## Consequences

- Every turn now has an explicit location for executed skills/agents.
- Provenance consumers can validate execution metadata from manifest and trace together.
- Slightly more post-execution bookkeeping is required.
