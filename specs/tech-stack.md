# Media Viewer Tech Stack

This document describes the complete technology stack used in the Media Viewer application.

---

## Overview

| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | Next.js + React | 16.x + 19.x |
| Backend | Spring Boot | 3.2.x |
| Database | PostgreSQL | 16 |
| Language | TypeScript / Java | 5.7 / 21 |
| Styling | Tailwind CSS | 4.x |
| State | TanStack Query + Zustand | 5.x |

---

## Frontend (`ui/`)

### Core Framework

| Package | Version | Purpose |
|---------|---------|---------|
| `next` | ^16.1.6 | React framework with App Router |
| `react` | ^19.0.0 | UI library |
| `react-dom` | ^19.0.0 | React DOM renderer |
| `typescript` | ^5.7.3 | Type safety |

**Key Features:**
- App Router with server components
- Turbopack for fast development (`next dev --turbo`)
- Server Actions support
- Built-in image optimization

### State Management

| Package | Version | Purpose |
|---------|---------|---------|
| `@tanstack/react-query` | ^5.64.0 | Server state management, caching, and synchronization |
| `zustand` | ^5.0.3 | Client state management |

### UI Components

| Package | Version | Purpose |
|---------|---------|---------|
| `@radix-ui/react-dialog` | ^1.1.4 | Modal dialogs |
| `@radix-ui/react-dropdown-menu` | ^2.1.4 | Dropdown menus |
| `@radix-ui/react-select` | ^2.1.4 | Select inputs |
| `@radix-ui/react-tabs` | ^1.1.2 | Tab navigation |
| `@radix-ui/react-toast` | ^1.2.4 | Toast notifications |
| `@radix-ui/react-tooltip` | ^1.1.6 | Tooltips |
| `@radix-ui/react-scroll-area` | ^1.2.2 | Custom scrollbars |
| `@radix-ui/react-separator` | ^1.1.1 | Visual separators |
| `@radix-ui/react-slot` | ^1.1.1 | Component composition |
| `lucide-react` | ^0.468.0 | Icon library |

### Styling

| Package | Version | Purpose |
|---------|---------|---------|
| `tailwindcss` | ^4.0.0 | Utility-first CSS framework |
| `@tailwindcss/postcss` | ^4.0.0 | PostCSS integration |
| `class-variance-authority` | ^0.7.1 | Variant-based component styling |
| `clsx` | ^2.1.1 | Conditional class names |
| `tailwind-merge` | ^2.6.0 | Merge Tailwind classes without conflicts |

**Tailwind v4 Configuration:**
- CSS-based configuration via `@theme` directive in `globals.css`
- OKLCH color space for design tokens
- No `tailwind.config.js` required

### Real-time Communication

| Package | Version | Purpose |
|---------|---------|---------|
| `sockjs-client` | ^1.6.1 | WebSocket fallback |
| `@stomp/stompjs` | ^7.0.0 | STOMP protocol for WebSocket messaging |

### Media & Utilities

| Package | Version | Purpose |
|---------|---------|---------|
| `vidstack` | ^1.12.9 | Video player component |
| `react-virtuoso` | ^4.12.3 | Virtualized lists for large datasets |
| `date-fns` | ^4.1.0 | Date formatting and manipulation |

### Testing

| Package | Version | Purpose |
|---------|---------|---------|
| `vitest` | ^2.1.8 | Test runner |
| `@testing-library/react` | ^16.1.0 | React component testing |
| `@vitejs/plugin-react` | ^4.3.4 | React support for Vite/Vitest |

### Linting

| Package | Version | Purpose |
|---------|---------|---------|
| `eslint` | ^9.17.0 | Code linting |
| `eslint-config-next` | ^16.1.6 | Next.js ESLint configuration |
| `@eslint/eslintrc` | ^3.2.0 | ESLint configuration support |

---

## Backend (`api/`)

### Core Framework

| Dependency | Version | Purpose |
|------------|---------|---------|
| `spring-boot-starter-parent` | 3.2.1 | Spring Boot parent POM |
| `spring-boot-starter-web` | 3.2.x | REST API support |
| `spring-boot-starter-data-jpa` | 3.2.x | JPA/Hibernate ORM |
| `spring-boot-starter-validation` | 3.2.x | Bean validation |
| `spring-boot-starter-websocket` | 3.2.x | WebSocket support |

**Java Version:** 21 (LTS)

### Database

| Dependency | Version | Purpose |
|------------|---------|---------|
| `postgresql` | runtime | PostgreSQL JDBC driver |
| `h2` | runtime | In-memory database for testing |

### Security & Encryption

| Dependency | Version | Purpose |
|------------|---------|---------|
| `jasypt-spring-boot-starter` | 3.0.5 | Credential encryption |

### File System Providers

| Dependency | Version | Purpose |
|------------|---------|---------|
| `jsch` | 0.2.17 | SFTP file access |
| `smbj` | 0.13.0 | SMB/CIFS file access |
| `commons-net` | 3.10.0 | FTP file access |

### Media Processing

| Dependency | Version | Purpose |
|------------|---------|---------|
| `tika-core` | 2.9.1 | MIME type detection |
| `metadata-extractor` | 2.19.0 | EXIF and media metadata extraction |
| `thumbnailator` | 0.4.20 | Image thumbnail generation |

### API Documentation

| Dependency | Version | Purpose |
|------------|---------|---------|
| `springdoc-openapi-starter-webmvc-ui` | 2.3.0 | Swagger UI / OpenAPI docs |

### Development

| Dependency | Version | Purpose |
|------------|---------|---------|
| `lombok` | managed | Boilerplate reduction |

### Testing

| Dependency | Version | Purpose |
|------------|---------|---------|
| `spring-boot-starter-test` | 3.2.x | JUnit 5, Mockito, MockMvc |

---

## Database

### PostgreSQL 16

**Docker Image:** `postgres:16-alpine`

**Key Features:**
- UUID primary keys via `uuid-ossp` extension
- Timestamps with timezone (`TIMESTAMP WITH TIME ZONE`)
- CHECK constraints for enum-like values
- Automatic `updated_at` triggers

### Schema Tables

| Table | Purpose |
|-------|---------|
| `remote_file_drives` | Storage locations (local, SMB, SFTP, FTP) |
| `images` | Indexed media files |
| `image_metadata` | EXIF and extracted metadata |
| `tags` | User-defined tags |
| `image_tags` | Many-to-many image-tag relationships |
| `crawl_jobs` | Background indexing job tracking |

---

## Infrastructure

### Docker Compose Services

| Service | Image | Port |
|---------|-------|------|
| `postgres` | postgres:16-alpine | 5432 |
| `api` | Custom (Spring Boot) | 8080 |
| `ui` | Custom (Next.js) | 3000 |

### Runtime Requirements

| Tool | Minimum Version |
|------|-----------------|
| Node.js | 20.x |
| npm | 10.x |
| Java | 21 |
| Docker | 20.x |
| Docker Compose | 2.x |

### Build Tools

| Tool | Purpose |
|------|---------|
| Maven Wrapper (`mvnw`) | Java build automation |
| npm | Node.js package management |
| Concurrently | Parallel script execution |

---

## Testing Strategy

### Frontend Testing

```bash
npm run test:ui        # Run Vitest tests
npm run test:ui:watch  # Watch mode
```

- **Unit tests:** Vitest + React Testing Library
- **Component tests:** `@testing-library/react`
- **Coverage:** Vitest built-in coverage

### Backend Testing

```bash
npm run test:api       # Run Maven tests
./mvnw test            # Direct Maven
```

- **Unit tests:** JUnit 5 + Mockito
- **Integration tests:** `@SpringBootTest`
- **Controller tests:** `@WebMvcTest` + MockMvc

### E2E API Testing

```bash
# IntelliJ HTTP Client files in e2e/
e2e/drives.http
e2e/images.http
e2e/tags.http
e2e/crawler.http
e2e/system.http
```

---

## Development Commands

```bash
# Start all services
npm run dev

# Start individual services
npm run dev:ui     # Next.js on :3000
npm run dev:api    # Spring Boot on :8080

# Build
npm run build

# Testing
npm run test       # All tests
npm run test:ui    # Frontend only
npm run test:api   # Backend only

# Code quality
npm run lint       # ESLint
npm run typecheck  # TypeScript

# Database
npm run db:up      # Start PostgreSQL
npm run db:down    # Stop PostgreSQL

# Docker (full stack)
npm run docker:up
npm run docker:down
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser                              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Next.js 16 (ui/)                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ App Router  │  │ TanStack    │  │ Radix UI + Tailwind │ │
│  │ (RSC)       │  │ Query       │  │ Components          │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                          │                                   │
│               WebSocket (STOMP) + REST                      │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                 Spring Boot 3.2 (api/)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ REST        │  │ WebSocket   │  │ File System         │ │
│  │ Controllers │  │ Events      │  │ Providers           │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                          │                                   │
│                   Spring Data JPA                           │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    PostgreSQL 16                            │
│        UUID PKs │ Timestamps w/ TZ │ Check Constraints     │
└─────────────────────────────────────────────────────────────┘
```

---

## Version Summary

| Component | Version |
|-----------|---------|
| Next.js | 16.1.6 |
| React | 19.0.0 |
| TypeScript | 5.7.3 |
| Tailwind CSS | 4.0.0 |
| Spring Boot | 3.2.1 |
| Java | 21 |
| PostgreSQL | 16 |
| Node.js | 20+ |
