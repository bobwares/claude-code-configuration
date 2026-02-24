# Tech Standards

Non-negotiable standards for all agents. These apply to all code in this project.

---

## TypeScript

- `"strict": true` in all tsconfig.json files
- No `any` — use `unknown` and narrow with type guards
- No `// @ts-ignore` without an explanatory comment
- Interfaces for object shapes; types for unions/intersections/mapped
- Explicit return types on all exported functions
- Types derived from source of truth: `typeof table.$inferSelect`, `z.infer<typeof schema>`

---

## React / Next.js

- Server Components by default — `'use client'` only when needed
- State display order: **Error → Loading (no data) → Empty → Success**
- Loading pattern: `if (loading && !data)` — **never** `if (loading)`
- Every list renders an empty state component
- Buttons are disabled + show a loading indicator during async operations
- No `useEffect` for data fetching
- `await params` in all Next.js 15 pages (params is a Promise)

---

## NestJS API

- Every endpoint has `@ApiOperation` and `@ApiResponse` Swagger decorators
- All DTOs use `nestjs-zod` (not `class-validator`)
- No business logic in controllers — controllers only route to services
- Services use injected Drizzle `db` — no raw SQL strings
- Throw NestJS exceptions (`NotFoundException`, `ConflictException`, etc.)

---

## Spring Boot API (Reactive — Project Reactor)

All Spring Boot applications use **Spring WebFlux** with **R2DBC** for fully non-blocking reactive APIs.

### Core Stack
- `spring-boot-starter-webflux` — reactive web layer
- `spring-boot-starter-data-r2dbc` — reactive data access
- `io.r2dbc:r2dbc-postgresql` — PostgreSQL R2DBC driver
- `io.projectreactor:reactor-test` — testing with StepVerifier

### API Layer (WebFlux)
- All controller methods return `Mono<T>` or `Flux<T>`
- Use `@RestController` with reactive return types
- `@Valid` on all request body parameters
- Never call `.block()` in production code

### Data Access (R2DBC)
- Use `@Table` annotation (not JPA `@Entity`)
- Repositories extend `ReactiveCrudRepository<T, ID>` or `R2dbcRepository<T, ID>`
- All database operations return `Mono<T>` or `Flux<T>`
- Use `@Transactional` on service methods (reactive transactions)
- Use `TransactionalOperator` for programmatic transaction control

### Web Clients
- Use `WebClient` for all external HTTP calls — never `RestTemplate`
- Configure with `WebClient.Builder` bean
- Handle errors with `.onStatus()` or `.onErrorResume()`

### General
- DTOs as Java Records — never expose R2DBC entities directly
- SLF4J for all logging — no `System.out.println`
- Constructor injection only — no `@Autowired` field injection
- Test reactive streams with `StepVerifier`

### Anti-Patterns
- `.block()` calls in production code
- Mixing JPA and R2DBC in the same module
- `RestTemplate` usage
- Blocking I/O inside reactive chains

---

## Database (Drizzle + PostgreSQL)

- UUID primary keys: `.primaryKey().defaultRandom()`
- All timestamps: `{ withTimezone: true }`
- Multi-step mutations always in `db.transaction()`
- Indexes defined in schema (not migration files)
- Export `type T = typeof table.$inferSelect` for every table
- Export `type NewT = typeof table.$inferInsert` for every table

---

## Security (All Layers)

- No secrets or credentials in source code — use environment variables
- `.env*` files are gitignored — never committed
- Validate user input at every entry point (DTO, Zod schema, Bean Validation)
- Authorization checked in service layer (not just route-level auth guard)
- Parameterized queries always (Drizzle and JPA handle this automatically)

---

## Testing

- Factory pattern for test data: `getMock[Entity](overrides?: Partial<Entity>)`
- Test behavior, not implementation details
- Unit tests co-located with source files (`*.test.ts`)
- E2E tests in `e2e/` directory using Playwright
- Minimum 80% line coverage on all service classes
- Write the failing test BEFORE implementing the fix (TDD)

---

## Git / Collaboration

- Conventional commits: `type(scope): description`
- All Claude commits include `Co-Authored-By: Claude <noreply@anthropic.com>`
- No commits directly to `main` — always via PR
- PR description includes: what changed, why, how to test
