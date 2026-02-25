# Overview

## What this repo implements
Media Viewer is a full-stack rebuild of the self-hosted media catalog described in `docs/PRD.md`. The system is implemented as two independently deployable processes: a Spring Boot 3.2.1 backend (`api/`) that exposes REST, WebSocket, and file-streaming endpoints, and a Next.js 16.1.6 (App Router) frontend (`ui/`) that consumes those APIs via React Query, Zustand, and Tailwind v4 styles described in `docs/tech-stack.md` and `docs/ENGINEERING_DESIGN_SPEC.md`. The backend persists drives, images, metadata, tags, and crawl jobs (see `api/src/main/java/com/picturemodel/domain`) and exposes the JSON schemas in `docs/schemas/`. The frontend ships UI components, hooks, and pages wired through `ui/src/app`, `ui/src/components`, and the shadcn primitives tracked in `ui/components.json`.

## How to run the documented workflow
Rebuilding the app in a clean directory follows the eight-turn plan in `docs/rebuild-guide.md` and the pre-made decisions enumerated in `docs/plan-mode-driver.md`. The earliest turns cover scaffolding (`Turn 1`), domain services (`Turn 2`–`3`), API controllers (`Turn 4`), and the UI foundation plus pages (`Turn 5`–`7`). Each turn finishes with the validation checklist (e.g., `cd api && ./mvnw compile` and `cd ui && npm run build` after Turn 1). The common developer workflow is automated via `Makefile` targets (`make dev`, `make api`, `make ui`, `make install`, `make test`), and backend lifecycles can also be executed via `start-backend.sh` and inspected with `test-video-detection.sh`.

## Guarantees baked into the repository
- The frontend uses Tailwind v4 imports directly from `node_modules` and shadcn primitives so that Turbopack resolves CSS without a `tailwind.config.js` (`ui/src/app/globals.css`, `docs/rebuild-guide.md` Turn 1). `ui/next.config.ts` explicitly sets `transpilePackages: ['@vidstack/react']` to keep Vidstack’s Proxy-based runtime intact (`docs/rebuild-guide.md`, `docs/plan-mode-driver.md`).
- The backend enforces `spring.jpa.open-in-view: true` and column definitions on every enum field (`api/src/main/resources/application.yml`, `docs/plan-mode-driver.md` Turn 2) to prevent LazyInitializationException and schema drift when new enum values are introduced.
- `docs/plan-mode-driver.md` freezes the decision to use Next.js + Tailwind + shadcn on the UI side and Spring Boot + Hibernate + asynchronous crawlers on the API side so that every generated project follows the same stack tokens and validation checklist.
- The API contract is documented via `docs/schemas/` (25 JSON Schema files) and enforced through DTOs under `api/src/main/java/com/picturemodel/api/dto`.
- Every rebuild turn is recorded inside `ai/agentic-pipeline/turns/` (manifests, session_context, pull_request summaries, ADRs, triggers) so that the command-driven workflow knows precisely which files were produced and validated at each stage.

## Running the stack
- `make dev` (defined in `Makefile`) launches `./mvnw spring-boot:run` for `api/` and `npm --prefix ui run dev` for `ui/` with the environment values and ports described in `api/src/main/resources/*.yml` and `ui/next.config.ts`.
- `make test` runs backend tests (`./mvnw -f api/pom.xml test`) and frontend tests (`npm --prefix ui run test`) to keep the quality gates aligned with the validation checklists from `docs/rebuild-guide.md`.
- `make install` executes `npm --prefix ui install` to reproduce the pinned dependency graph (`ui/package.json`), including `@tanstack/react-query`, `@vidstack/react@1.12.13`, `lucide-react`, and shadcn’s CLI dependency.
- `start-backend.sh` encapsulates the exact build (`./mvnw clean package -DskipTests`) and run steps expected before executing HTTP-based smoke tests such as `test-video-detection.sh`.
