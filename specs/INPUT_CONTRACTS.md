# Input Contracts

## Product Requirements Document (PRD)
- **Location:** `docs/PRD.md` is the canonical instance of a valid PRD for Media Viewer.
- **Format:** Markdown with top-level headings. The rebuild workflow assumes the following sections exist in order and cover the indicated content.
  1. `Executive Summary` — one paragraph describing the product’s purpose.
  2. `Product Vision` — aspirational third-person view of what the product solves.
  3. `User Personas` — table with Persona / Goals / Behaviors / Needs columns.
  4. `Core Use Cases` — bullet list of user-facing tasks (storage management, crawling, browsing, tagging, viewing).
  5. `Functional Requirements` — enumerated list of capabilities (drive CRUD, credential encryption, crawl control, metadata extraction, streaming).
  6. `Non-Functional Requirements` — architectural constraints (single-node, filesystem access, range streaming). 
  7. `Constraints` — platform constraints (JVM backend, Next.js UI, localized thumbnails, CORS). 
  8. `Assumptions`, `Risks`, and `Open Questions` — textual items guiding later decisions.

### Minimal PRD template
```
# Product Requirements Document (PRD)

## Executive Summary
Describe the problem scope and the high-level solution.

## Product Vision
Summarize what the product empowers users to do and why it matters.

## User Personas
| Persona | Goals | Behaviors | Needs |
| --- | --- | --- | --- |
| Example Persona | … | … | … |

## Core Use Cases
- Use case 1 describing drive management.
- Use case 2 describing media browsing/crawling.

## Functional Requirements
- List drives (LOCAL/SMB/SFTP/FTP).
- Encrypt credentials, manage crawl jobs, extract metadata, stream files.

## Non-Functional Requirements
- Runtime environment requirements (single-node, H2/PostgreSQL compatibility, CORS). 
- Media access expectations (filesystem/network, thumbnails stored locally).

## Constraints
- Backend must run on JVM (Spring Boot), UI must be Next.js App Router.

## Assumptions
- Single tenant, full filesystem access, metadata regen is acceptable.

## Risks
- Network crawl slowness, authentication gap, metadata mismatch.

## Open Questions
- Authentication strategy?
- Metadata standardization?
```

## Domain-Driven Design (DDD) input
- **NOTE:** A dedicated DDD input template is not present in the repo; the closest artifact is `docs/ENGINEERING_DESIGN_SPEC.md` (System Context, Architecture Overview, Domain Model, Data Model, API Specification, Deployment, etc.). For a new `/execute` pipeline to succeed, supply a Markdown document covering the same sections.
- **Expected sections:**
  1. `System Context` — describe the modular monolith, REST + WebSocket boundaries, and data flows.
  2. `Architecture Overview` — list layers (API, service, domain, infrastructure).
  3. `Domain Model` — enumerate aggregates (Drive, Image, Tag, CrawlJob) with key attributes, relationships, and enums (DriveType, ConnectionStatus, CrawlStatus, MetadataSource).
  4. `Data Model` — describe tables, columns, constraints, and indexes (mirroring `db/schema.sql`).
  5. `API Specification` — summarize REST endpoints, WebSocket topics, DTOs, and JSON schemas (see `docs/schemas/`).
  6. `Frontend Architecture` — list Next.js pages, global state providers, shadcn components, and responsive layouts.
  7. `Backend Architecture` — document services, thread pools, async executors, file system providers, and security (Jasypt, CORS, Jackson). 
  8. `Infrastructure & Deployment` — mention databases (H2 dev, PostgreSQL prod), thumbnail storage, Docker expectations, and observability hooks.

### Minimal DDD template
```
# Domain-Driven Design Brief

## System Context
Describe the UI, API, and storage boundaries (HTTP, STOMP over SockJS, JDBC).

## Architecture Overview
Describe the modular monolith layers and async execution for crawl/thumbnail jobs.

## Domain Model
- Drive (DriveType enum, status, credentials).
- Image (fileName, fileHash, metadataRelationship).
- Tag (name, color, usageCount).
- CrawlJob (status enum, counters, errors JSON).
- Enums: DriveType, ConnectionStatus, CrawlStatus, MetadataSource (must include VIDEO).

## Data Model
Describe the six tables (`remote_file_drives`, `tags`, `images`, `image_metadata`, `image_tags`, `crawl_jobs`), their PKs, FKs, unique constraints, and indexes.

## API Specification
List REST endpoints such as `/api/drives`, `/api/images`, `/api/files/{id}`, `/api/crawler`, `/api/tags`, `/api/system/status`, plus WebSocket topics `/topic/crawl/progress` and `/topic/drive/{driveId}/status`.

## Frontend Architecture
Summarize pages under `ui/src/app`, global providers (`QueryClientProvider`, `ReactQueryDevtools`), and reusable components/hook responsibilities (`use-drives`, `use-images`, `ImageGrid`, `VideoPlayer`).

## Backend Architecture
Summarize services (DriveService, CrawlerService, ThumbnailService, VideoMetadataExtractorService, WebSocketEventService), configs (AsyncConfig with three executors, CorsConfig, JacksonConfig, JasyptConfig, JpaConfig, WebSocketConfig), and file system providers.

## Infrastructure & Deployment
Explain database modes (H2 dev file at `api/data` and PostgreSQL prod), thumbnail local storage path, expectations for Docker Compose (missing but anticipated), and logging/observability choices.
```

- **Validation expectation:** Even without an automated parser, future `/execute` implementations should assert each section above exists and cite the same enums/paths as the existing documentation (`docs/schemas`, `db/schema.sql`, `api/*`, `ui/*`).
