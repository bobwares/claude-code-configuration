# Project Context

**INSTRUCTIONS**: This file persists across all sessions.

## Project Identity

**Name**: Media Viewer
**Purpose**: Self-hosted media indexing and browsing solution that connects to local or remote storage (LOCAL, SMB, SFTP, FTP), crawls media files, extracts metadata, generates thumbnails, and delivers them via a React + Next.js UI and Spring Boot API.
**Status**: Active Development
**Repository**: local

## Tech Stack

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| Frontend | Next.js App Router | 16.x | Port 3000 |
| UI Components | shadcn/ui + Tailwind CSS | v4 | Custom components |
| Backend API | Java/Spring Boot | 3.2.1 | Port 8080 |
| Database | PostgreSQL | 16 | Port 5432 |
| Database (Dev) | H2 | embedded | ./data/mediaviewer-dev |
| WebSocket | STOMP + SockJS | | Real-time updates |
| State Management | TanStack Query + Zustand | | Client-side |
| Testing | Vitest + Playwright | | Frontend |

## Architecture Overview

- Next.js frontend proxies /api to Spring Boot backend
- Spring Boot provides REST API + WebSocket endpoints
- PostgreSQL stores drive configs, images, metadata, tags, crawl jobs
- FileSystemProviders handle LOCAL/SMB/SFTP/FTP access
- Thumbnails stored locally in ./thumbnails
- Jasypt encrypts drive credentials

## Key URLs (Local Development)

| Service | URL |
|---------|-----|
| Next.js frontend | http://localhost:3000 |
| Spring API | http://localhost:8080 |
| Spring Swagger | http://localhost:8080/swagger-ui.html |
| H2 Console | http://localhost:8080/h2-console |
| PostgreSQL | localhost:5432 |

## Environment Setup

```bash
# 1. Start database
docker compose up -d postgres

# 2. Start API (from api/ directory)
./mvnw spring-boot:run

# 3. Start UI (from ui/ directory)
npm install
npm run dev
```

## Key Architectural Decisions

- ADR-001: Spring Boot chosen over NestJS for enterprise filesystem support
- ADR-002: H2 for development, PostgreSQL for production
- ADR-003: STOMP over SockJS for WebSocket (browser compatibility)
- ADR-004: Jasypt for credential encryption (vault password via env var)
- ADR-005: shadcn pattern for UI components (copy-paste, not npm)
