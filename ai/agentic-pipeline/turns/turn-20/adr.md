# ADR - Turn 20

## Decision: Isolated Agent for Analysis Skill

### Context
User requested an analysis skill that clears context before running to save tokens.

### Decision
- Skill spawns an Explore subagent instead of running analysis in main conversation
- Explore agent has no access to conversation history (isolated context)
- Uses sonnet model for balanced capability/cost
- Output is static markdown (no follow-up required)

### Consequences
- Token-efficient: analysis doesn't consume main context window
- Repeatable: same analysis can be run multiple times cheaply
- Trade-off: agent can't reference prior conversation context
