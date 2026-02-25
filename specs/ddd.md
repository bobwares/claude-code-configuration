# Domain-Driven Design (DDD)

## System Context
Media Viewer is a modular monolith composed of two deployable processes: the Next.js 16.1.6 App Router UI (`ui/`) and the Spring Boot 3.2.1 API (`api/`). They communicate via HTTP REST (`/api/*` controllers) and STOMP-over-SockJS WebSockets (`/ws` endpoint configured in `api/config/WebSocketConfig.java`). The frontend proxies `/api` to `http://localhost:8080` (`ui/next.config.ts`) and consumes the REST schema definitions stored under `docs/schemas/` plus WebSocket topics `/topic/crawl/progress` and `/topic/drive/{driveId}/status` for real-time updates.

## Architecture Overview
The backend splits into API controllers (`api/src/main/java/com/picturemodel/api/`), services (`.../service/`), domain entities/enums/repositories (`.../domain/`), infrastructure providers (`.../infrastructure/filesystem`), and configuration classes (`/config`). Controllers are thin wrappers around services, services are transactional (except async crawls/thumbnails) and emit events via `WebSocketEventService`, and infrastructure provides pluggable filesystem access (Local/SMB/SFTP/FTP) via `FileSystemProviderFactory`. The frontend uses React Query + Zustand for data/state, `api-client.ts` for HTTP, custom hooks for CRUD, shadcn components for UI, and Tailwind CSS v4 for styling (`docs/tech-stack.md`).

## Domain Model
- **Drive (RemoteFileDrive):** Aggregate with attributes `id` (UUID), `name`, `type` (DriveType: LOCAL, SMB, SFTP, FTP), `connectionUrl`, `rootPath`, `encryptedCredentials`, `status` (ConnectionStatus), `autoConnect`, `autoCrawl`, timestamps, and relationships to `Image` and `CrawlJob`. (`api/src/main/java/com/picturemodel/domain/entity/RemoteFileDrive.java`).
- **Image:** Aggregate with `id`, `drive` FK, `fileName`, `filePath`, `fileSize`, `fileHash`, `mimeType`, `width`, `height`, `capturedAt`, `indexedDate`, `deleted` flag, and metadata/tags (`Image.java`).
- **ImageMetadata:** Key-value store per image with `metadataKey`, `metadataValue`, `source` from `MetadataSource` enum (EXIF, VIDEO, USER_ENTERED, AUTO_GENERATED) with column definition `VARCHAR(20)` to avoid hibernate check issues (`api/domain/entity/ImageMetadata.java`).
- **Tag:** Entity with `id`, `name` (unique), `color`, `createdDate`, Many-to-Many with Image through `image_tags` join table.
- **CrawlJob:** Tracks crawl runs per drive with status values `CrawlStatus` (PENDING, IN_PROGRESS, PAUSED, COMPLETED, FAILED, CANCELLED), counters (files_processed, files_added, etc.), `errors_json`, and `current_path_value`. Enforced via `api/domain/entity/CrawlJob.java` and `db/schema.sql`.
- **Enums:** DriveType, ConnectionStatus, CrawlStatus, MetadataSource as defined under `api/domain/enums/`. Each uses `@Enumerated(EnumType.STRING)` and explicit column definitions to align with `db/schema.sql` and prevent schema drift.

## Data Model
Six tables exist in `db/schema.sql` generated from the entities (Turn 24 manifest header referencing `.claude/skills/sql`):
1. `remote_file_drives` with CHECK constraints for `type` and `status` and encrypted credentials.
2. `tags` with unique `name` and optional `color`.
3. `images` storing metadata, file hashes, and boolean `deleted` plus indexes on `file_hash`, `indexed_date`, `file_name`, and `deleted`.
4. `image_metadata` with `(image_id, metadata_key)` unique constraint and `source` CHECK for MetadataSource values (must include VIDEO).
5. `image_tags` join table between images and tags.
6. `crawl_jobs` recording job stats, `errors_json` default `'[]'`, `status` per CrawlStatus, and foreign key to drives. Tables create indexes per `db/schema.sql` order reflecting FK dependencies.

## API Specification
- **Drive endpoints:** `/api/drives` (CRUD), `/api/drives/{id}/connect`, `/disconnect`, `/status`, `/test`, `/tree`. (`DriveController.java`).
- **Image endpoints:** `/api/images` search with filters (matches `docs/schemas/api-request-drive-images-query.json`), `/api/images/{id}`, `/api/images/{id}/metadata`, `/api/images/{id}/tags`, `DELETE /api/images/{id}` for soft delete.
- **File streaming:** `/api/files/{imageId}` full stream, `/api/files/{imageId}/thumbnail`, `/api/files/{imageId}/download`, with IO exception handling as described in Turn 4 instructions.
- **Crawler:** `/api/crawler/start`, `/crawler/jobs`, `/crawler/jobs/{id}/cancel`, `/crawler/jobs/{id}` etc., returning `CrawlJobDto`. Asynchronous processing via `@Async(