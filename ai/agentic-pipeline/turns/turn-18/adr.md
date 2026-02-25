# ADR - Turn 18

## Decision: Spring Boot Epic Structure

### Context
The Media Viewer PRD specifies a Spring Boot 3.2 backend (not the typical NestJS stack). The epic generator needed to adapt task assignments accordingly.

### Decision
- Assigned all backend tasks to `spring-engineer` (not nestjs-engineer)
- Used Spring Data JPA patterns (not Drizzle ORM)
- Included FileSystemProvider factory pattern for pluggable storage access
- Added WebSocket/STOMP configuration tasks for real-time updates

### Consequences
- Epic correctly maps to the actual tech stack
- `spring-engineer` agent will handle all Java/Spring code
- JUnit 5 + MockMvc for backend testing (not Vitest)
