# Issue: Docker PostgreSQL Port 5432 Already Allocated

**Reported**: 2026-02-24T23:14:52Z
**Turn**: 9
**Status**: Resolved — Stopped conflicting container `mediaviewer-postgres`

---

## Problem Statement

Running `make all` fails because Docker cannot bind port 5432 for the PostgreSQL container. Another container is already using this port.

## Observed Symptoms

- `make all` command fails with exit code 1
- Error: `Bind for 0.0.0.0:5432 failed: port is already allocated`
- PostgreSQL container fails to start
- Network and volume are created successfully

## Environment

| Attribute | Value |
|-----------|-------|
| Branch | feature/execute-turn-branch |
| Node | v24.3.0 |
| OS | Darwin (macOS) |
| Docker | Running |
| Last commit | 33d3714 Initial Commit |

## Evidence Collected

### Error Details
```
Error response from daemon: driver failed programming external connectivity
on endpoint claude-code-configuration-postgres-1:
Bind for 0.0.0.0:5432 failed: port is already allocated
```

### Port Conflict Analysis
```
$ lsof -i :5432
COMMAND    PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
com.docke 2243 bobware  126u  IPv6 0xe31b29ed80e12eeb      0t0  TCP *:postgresql (LISTEN)

$ docker ps --filter "publish=5432"
NAMES                  PORTS                    STATUS
mediaviewer-postgres   0.0.0.0:5432->5432/tcp   Up 44 hours (healthy)
```

### Related Files
| File | Relevance |
|------|-----------|
| `docker-compose.yml` | Defines PostgreSQL service on port 5432 |
| `.env.example` | Contains DATABASE_URL with port 5432 |

---

## Diagnosis

### Root Cause Analysis

**Primary hypothesis**: Port conflict with existing Docker container

Another project's PostgreSQL container (`mediaviewer-postgres`) is already running and bound to port 5432. Docker cannot bind the same host port to multiple containers.

**Evidence**:
- `docker ps` shows `mediaviewer-postgres` using `0.0.0.0:5432->5432/tcp`
- Container has been running for 44 hours
- This is from a different project but shares the same default PostgreSQL port

### Alternative Hypotheses

| Rank | Hypothesis | Evidence | Likelihood |
|------|------------|----------|------------|
| 2 | Native PostgreSQL running | Could also bind 5432 | Low — lsof shows Docker, not native |
| 3 | Stale Docker network state | Previous failed starts | Low — fresh container name |

---

## Resolution Plan

### Option A: Use Different Port (Recommended)

Change this project to use port 5433 instead, avoiding conflict with existing services.

**Pros**:
- No disruption to other projects
- Both databases can run simultaneously

**Cons**:
- Need to update DATABASE_URL in .env

### Option B: Stop Conflicting Container

Stop the `mediaviewer-postgres` container to free port 5432.

**Pros**:
- No config changes needed

**Cons**:
- Disrupts the other project

### Option C: Share Existing Database

Use the existing PostgreSQL instance and just create a new database.

**Pros**:
- Resource efficient

**Cons**:
- Projects become coupled

---

## Recommended Fix (Option A)

### Fix Steps

1. **Update docker-compose.yml** — Change host port to 5433
   - File: `docker-compose.yml`
   - Change: `"5432:5432"` → `"5433:5432"`

2. **Update .env.example** — Update DATABASE_URL
   - File: `.env.example`
   - Change: port from 5432 to 5433

3. **Update Drizzle config** — Ensure DATABASE_URL matches
   - File: `packages/database/drizzle.config.ts`
   - No change needed (reads from env)

### Affected Files

| File | Change Type |
|------|-------------|
| `docker-compose.yml` | modify |
| `.env.example` | modify |

---

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Breaking existing .env files | Users copy from .env.example, so new setups will be correct |
| Port 5433 also in use | Check with `lsof -i :5433` before proceeding |

---

## Estimated Effort

| Phase | Effort |
|-------|--------|
| Verification | 30 seconds (check port 5433 availability) |
| Implementation | 2 minutes |
| Testing | 1 minute (run `make docker-up`) |
