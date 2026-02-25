# Stack Tokens

## `full-stack-nextjs-spring`
- **Evidence:** `ai/context/project_context.md` lists this as the active pattern name, and every `ai/agentic-pipeline/turns/*/manifest.json` records `activePatternName": "full-stack-nextjs-spring"`. It bundles Next.js frontend + Spring Boot backend + shadcn components + Vidstack video player + agentic orchestration.
- **Generated layers:**
  - `/ui`: Next.js App Router, Tailwind v4 `globals.css`, shadcn components installed under `ui/src/components/ui/`, re-exported via `ui/components.json`.
  - `/api`: Spring Boot directory layout (`api/src/main/java/com/picturemodel/{api,config,domain,infrastructure,service}`), Maven `pom.xml`, resource files (`application-*.yml`).
  - `/docs`: `docs/rebuild-guide.md`, `docs/plan-mode-driver.md`, `docs/tech-stack.md`, and `docs/schemas/` describing API contracts needed by both layers.
  - `/ai`: agentic pipeline log storage (`ai/agentic-pipeline/turns`), context definitions, skill refs (e.g., `.claude/skills/sql`).
- **Integration points:**
  - Next.js rewrites (`ui/next.config.ts`) proxy `/api/*` to `http://localhost:8080/api/*` for dev front/back integration.
  - JSON schemas (`docs/schemas`) feed both backend DTOs and `ui/src/types/api.ts`.
  - Shared enums (DriveType, ConnectionStatus, CrawlStatus, MetadataSource) drive naming conventions between `api/domain/enums` and `ui/src/types/api.ts`.
- **Naming conventions:**
  - Entities + tables: `RemoteFileDrive` → `remote_file_drives`, `Image` → `images`, `Tag` → `tags`, `CrawlJob` → `crawl_jobs`, `ImageMetadata` → `image_metadata`, `ImageTag` join table.
  - Controllers under `/api/controller` use kebab endpoint names (`/api/drives`, `/api/images`, `/api/files/{imageId}` etc). Frontend pages mimic those scopes (`/images/[id]`, `/browse/[id]`, `/storage`).
- **Token validation rules:**
  - Frontend must reuse the same dependency list from `docs/rebuild-guide.md` (React Query, Zustand, react-hook-form, Zod, STOMP/SockJS, Vidstack@1.12.13). `@vidstack/react` must stay pinned to 1.12.13 and be listed in `transpilePackages` (`ui/next.config.ts`).
  - Tailwind v4 CSS imports must use direct `node_modules` paths (`ui/src/app/globals.css`).
  - Back-end enums must all specify `columnDefinition = "VARCHAR(N)"` and include `MetadataSource.VIDEO` from day one (`api/src/main/java/com/picturemodel/domain/enums/MetadataSource.java`, `docs/plan-mode-driver.md` Turn 2 constraints).
  - `npx shadcn@latest add button card dialog dropdown-menu form input label` must run in a single command to populate `ui/src/components/ui/` (per Turn 1 instructions).

## `nextjs`
- **Description:** Generates the UI scaffold plus pages described in `ui/docs/file-map.md`.
- **Directories/files created:** `ui/package.json`, `url/next.config.ts`, `ui/src/app/layout.tsx`, `globals.css`, pages under `browse`, `images`, `storage`, `tags`, `settings`, reusable components and hooks (`components/`, `hooks/`, `store/`, `lib/`, `types/`).
- **Integration points:** React Query provider in `app/layout.tsx`, API client in `lib/api-client.ts`, WebSocket hook hooking into `use-websocket.ts`, and shared `ui/types/api.ts` matching `docs/schemas`.
- **Template variables:** `app/layout.tsx` wires `QueryClientProvider`, `ReactQueryDevtools`, and `globals.css` imports; `components/VideoPlayer.tsx` must import Vidstack CSS and wrap `<MediaPlayer>` (per `docs/plan-mode-driver.md` Turn 6).

## `spring`
- **Description:** Produces the backend Maven project with controllers, services, infrastructure, and configs.
- **Outputs:** `api/pom.xml`, `api/src/main/java/com/picturemodel/MediaViewerApplication.java`, packages `api/controller`, `api/service`, `api/domain`, `api/config`, `api/infrastructure/filesystem`, `api/api/dto/request/response`, `api/src/main/resources/application*.yml`.
- **Integration:** Controllers referenced by `docs/schemas`, services invoked from `api/controller`, asynchronous executors defined in `config/AsyncConfig.java`, `FileSystemProvider` implementations used by `CrawlerService` and `FileController`.
- **Validation rules:** `CrawlerService.startCrawl()` annotated with `@Async("crawlerExecutor")`, thumbnail sizes generated via Thumbnailator (per `docs/plan-mode-driver.md` Turn 3), and `open-in-view: true` set in base `application.yml` (per Turn 1 validation).

## `shadcn`
- **Description:** Controls component installation via `npx shadcn add …`, tracked in `ui/components.json`.
- **Generated files:** `ui/src/components/ui/` primitives (button, card, dialog, dropdown-menu, form, input, label) used by custom components (`DriveCard`, `ImageGrid`, `AddDriveDialog`).
- **Templates:** `docs/shadcn.md` enumerates CLI steps for Codex integration.

## `vidstack`
- **Description:** Video playback stack requiring `@vidstack/react` 1.12.13 plus CSS imports.
- **Integration:** `components/VideoPlayer.tsx` imports Vidstack CSS and `DefaultVideoLayout`, used by `ui/src/app/images/[id]/page.tsx`. The `docs/rebuild-guide.md` Turn 6 ensures the CSS import order.
- **Validation:** `ui/next.config.ts` must include `transpilePackages: ['@vidstack/react']` to keep Vidstack’s Proxy intact (see `docs/rebuild-guide.md` G4). The dependency list warns against the npm default 0.6.x release (`docs/rebuild-guide.md` G5).

## `ai` (agentic pipeline)
- **Description:** Stores execution memory, manifests, and turn metadata under `ai/agentic-pipeline/turns/`. Each turn emits a manifest, session context, pull request summary, and ADR.
- **Integration:** Planning commands read these files to continue the workflow; validation checklists reference them for reproducibility.
- **Format:** JSON manifests list `taskId`, `inputs`, `outputs`, `validations`, and `provenance`. Markdown session contexts enumerate loaded files and prompts.

## Unsupported tokens referenced in instructions
- `nestjs`, `drizzle`, and other tokens mentioned in the request are not present in the repo (no `nestjs` or `drizzle` references via `rg`). A future `/execute` implementation must either add templates for them or mark the tokens as invalid before generating code.
