# Commands

## Core build & run targets (Makefile)

| Command | Description | Inputs / files read | Outputs / side effects |
| --- | --- | --- | --- |
| `make dev` | Launches the full stack (Spring Boot API + Next.js UI) concurrently. | `Makefile` variables `API_DEV_CMD` and `UI_DEV_CMD` reference `./mvnw -f api/pom.xml spring-boot:run` and `npm --prefix ui run dev`. | API on `:8080`, UI on `:3000`, same as `api/src/main/resources/application*.yml` and `ui/next.config.ts`. |
| `make api` | Runs `./mvnw -f api/pom.xml spring-boot:run` alone. | `api/pom.xml`, `api/src/main/resources/*.yml`. | Live API server for backend-only work. |
| `make ui` | Runs `npm --prefix ui run dev`. | `ui/package.json`, `ui/next.config.ts`, `ui/src`. | Next.js development server with Tailwind/shadcn UI. |
| `make install` | Bootstraps the UI dependencies exactly once. | `ui/package.json`, `ui/package-lock.json`, `ui/components.json`. | `node_modules` in `ui/`, needed for shadcn + Vidstack + React Query stack. |
| `make test` | Runs API tests (`./mvnw test`) and UI tests (`npm --prefix ui run test`). | `api/src` tests/services/controllers, `ui/src` components/hooks. | Ensures the validations listed in `docs/rebuild-guide.md` (Turn 2/3/5 etc) keep passing. |

## Maven & npm commands

| Command | Description | Notes |
| --- | --- | --- |
| `./mvnw -f api/pom.xml compile` | Pre-checks the backend during Turn 1 scaffolding. | Referenced directly by `docs/rebuild-guide.md` Turn 1 validation. |
| `./mvnw -f api/pom.xml test` | Runs unit/integration tests that guard entity constraints and services (`docs/rebuild-guide.md` Turns 2–3). | Includes Testcontainers PostgreSQL, H2, and DTO validation. |
| `./mvnw -f api/pom.xml spring-boot:run` | Starts the production-like backend; used by `make dev`, `make api`, and `start-backend.sh`. | Honors `application-dev.yml` and `application-prod.yml`. |
| `npm --prefix ui run dev` | Starts Next.js App Router dev server with Tailwind v4 and Turbopack. | Uses `ui/src/app/layout.tsx`, `ui/src/app/globals.css`, and `ui/next.config.ts`. |
| `npm --prefix ui run build` | Builds and bundles the UI; part of Turn 1 validation in `docs/rebuild-guide.md`. | Required before deploying or running `npm --prefix ui run start`. |
| `npm --prefix ui run lint` | Runs ESLint via `eslint.config.mjs`; referenced by `docs/rebuild-guide.md` known-gotcha table (G11). | Enforces import hygiene for shadcn primitives and utilities. |
| `npm --prefix ui run test` | Runs Vitest/unit tests for components/hooks. | Ensures utilities defined in `ui/src/lib/utils.ts` stay exported. |

## Project initialization & dependency setup

| Command | Description | Notes |
| --- | --- | --- |
| `npx create-next-app@latest ui --typescript --tailwind --app --no-src-dir --import-alias "@/*" --use-npm` | Bootstraps the UI scaffolding required by Turn 1 (`docs/plan-mode-driver.md § Turn 1`). | Creates `ui/src/app/`, `ui/package.json`, `ui/next-env.d.ts` before the rest of the rebuild. |
| `cd ui && npm install ...` | Installs the exact dependency set enumerated in `docs/rebuild-guide.md` (React Query, Zustand, react-hook-form, Zod, @stomp/stompjs, @vidstack/react@1.12.13, lucide-react, clsx, tailwind-merge, class-variance-authority, tw-animate-css). | This single command ensures the dependency graph matches the version lock recorded in `ui/package-lock.json` and avoids React 18–only Vidstack artifacts (G5). |
| `cd ui && npm install --save-dev vitest@^4 @testing-library/react@^16 @testing-library/jest-dom@^6 jsdom@^28 @vitejs/plugin-react@^5 @types/sockjs-client` | Adds the devtooling required by `docs/rebuild-guide.md` for Turn 5/6. | `ui/package-lock.json` records these versions for future installs. |
| `npx shadcn@latest add button card dialog dropdown-menu form input label` | Installs all required shadcn primitives in one command (Turn 1 specification). | Updates `ui/components.json` and copies files into `ui/src/components/ui/`. |

## Support scripts

| Script | Description | Notes |
| --- | --- | --- |
| `./start-backend.sh` | Runs `./mvnw clean`, `./mvnw package -DskipTests`, then `./mvnw spring-boot:run` from the repo root. | Provides the exact build+run sequence documented in `start-backend.sh` for video-enabled playback and H2 console availability. |
| `./test-video-detection.sh` | Hits `http://localhost:8080/api/images` (with filters) and formats results via `jq` to verify video detection before release. | Assumes the backend is running and produces JSON responses matching `docs/schemas/api-response-image-details.json`. |

## Procedural prompts and validations

- **Session start command (per `docs/rebuild-guide.md`):**
  > `I am starting Turn N of the Media Viewer rebuild.`
  > `Reference: docs/rebuild-guide.md (Turn N section)`
  > `Reference: docs/plan-mode-driver.md (Turn N pre-conditions)`
  > `Tech stack: docs/tech-stack.md`
  > `API contract: docs/ENGINEERING_DESIGN_SPEC.md`
  > `Do not invent new architectural decisions — follow the spec exactly.`

- **Validation commands (Turn 1 example):** `cd api && ./mvnw compile`, `cd ui && npm run build`, `./mvnw test`, `npm run lint` — these appear throughout `docs/rebuild-guide.md` as checkboxes that gate moving to the next turn.
