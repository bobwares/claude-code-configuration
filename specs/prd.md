# Product Requirements Document (PRD)

## Executive Summary
Media Viewer is a self-hosted media indexing and browsing solution that connects to local or remote storage (LOCAL, SMB, SFTP, FTP), crawls media files, extracts metadata, generates thumbnails, and delivers them via a React + Next.js UI and Spring Boot API. This repo implements the stack detailed in `docs/PRD.md` with a backend responsible for drives, metadata, and crawling plus a frontend that provides dashboards, browsing, and tagging experiences.

## Product Vision
Provide a fast, organized, visually rich catalog of large personal or team media libraries that surfaces crawl progress, metadata, and search without requiring heavy setup; the implementation shown here uses Next.js 16 App Router + Turbopack UI with TanStack Query/Zustand and a Spring Boot 3.2.1 API with WebSocket/SockJS real-time updates (`docs/tech-stack.md`).

## User Personas
| Persona | Goals | Behaviors | Needs |
| --- | --- | --- | --- |
| Media Curator | Organize huge photo/video collections | Configures multiple drives, watches crawls, tags assets | Reliable indexing, fast search, clear status widgets. |
| Photographer / Creator | Find specific assets quickly | Searches by filename/path, tags, and metadata | Accurate metadata, thumbnails, video playback. |
| Home / Small Office Admin | Centralize media across devices | Connects SMB/SFTP/FTP shares, verifies connectivity | Simple setup, drive health feedback, secure credentials. |

## Core Use Cases
- Add/manage storage devices (LOCAL, SMB, SFTP, FTP) with credential encryption and connectivity monitoring.
- Start/pause/cancel crawl jobs with incremental, EXIF, thumbnail options and real-time progress via `/topic/crawl/progress` WebSocket updates.
- Browse media by drives, directories, and tags with infinite scroll, filters, and search powered by React Query + Zustand.
- View images/videos with metadata, tags, and downloads/streaming (Vidstack video player with CSS imports) plus update user-entered metadata.
- Manage tags globally with CRUD operations and associate them with images.

## Functional Requirements
- CRUD storage devices, manage connections, run directory tests, emit status at `/api/drives` + `/api/drives/{id}` + `/api/drives/{id}/test` + `/api/drives/{id}/tree`.
- Encrypt credentials with Jasypt, use FileSystemProvider implementations (Local/SMB/SFTP/FTP) via `FileSystemProviderFactory`.
- Crawl drives via `CrawlerService`, store `CrawlJob` entries, track `files_processed`, `errors_json`, progress path, and emit STOMP events.
- Index images/videos into `images` table, extract EXIF via `ExifExtractorService`, video metadata via `VideoMetadataExtractorService`, store metadata in `image_metadata` table with `MetadataSource` (EXIF, VIDEO, USER_ENTERED, AUTO_GENERATED), soft delete support (`deleted` boolean).
- Thumbnail generation in sizes small/medium/large using `ThumbnailService`, store under `app.thumbnails.base-path` (./thumbnails).
- REST endpoints for `/api/images`, `/api/files/{imageId}`, `/api/crawler`, `/api/tags`, `/api/system/status`, matching JSON schemas (`docs/schemas/*.json`).
- UI features: Dashboard, drive browser, search results, image detail (Vidstack for videos), storage list, tags management, settings shell.

## Non-Functional Requirements
- Backend: Spring Boot 3.2.1, Java 21, H2 (dev) with Postgres (prod) compatibility; `spring.jpa.open-in-view: true` prevents LazyInitializationException (`api/src/main/resources/application.yml`).
- Frontend: Next.js 16.1.6 App Router + Turbopack, TypeScript 5.x strict, Tailwind CSS v4, shadcn primitives, Vidstack video player requiring `transpilePackages` in `ui/next.config.ts`.
- Media access: must operate with filesystem/network paths; thumbnails stored locally; databases (H2 dev path `./data/mediaviewer-dev`, PostgreSQL via `application-prod.yml`).
- WebSocket: STOMP+SockJS for progress updates and drive status.
- Real-time crawl and search via `/api` + WebSocket, while UI data layer uses TanStack Query + Zustand.

## Constraints
- Single-node deployment. No external queues; thread pools configured through `AsyncConfig` (task/crawler/thumbnail executors).
- No auth; single-tenant assumption.
- Filestreaming controllers must catch `IOException` (broken pipes) to avoid error noise (`docs/rebuild-guide.md` Turn 4). `MetadataSource` must include `VIDEO` from day 1 to allow video indexing.

## Assumptions
- Host machine has Node 20, npm 10, Java 21, Docker 24 for dev runs (`docs/rebuild-guide.md` Pre-Requisites).
- Credentials encrypted via Jasypt; vault password provided via `JASYPT_PASSWORD` env var.
- Crawls and thumbnails acceptable to rerun; no persistent job state beyond in-memory maps.

## Risks
- Network drive crawls can be slow or fail if credentials invalid; status shown via `/topic/drive/{driveId}/status`.
- No multi-user auth; the service is accessible on known ports.
- WebSocket protocol mismatches if client fails to subscribe or handle STOMP.

## Open Questions
- Should authentication/authorization be added in future?
- How to reconcile deleted files across crawls?
- What is the scaling expectation for extremely large media libraries?
- How should metadata key naming be standardized across crawlers and UI?
