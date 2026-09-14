# Pending IRD updates (blocked on Notion free-block limit)

The Notion workspace has hit its free-block limit — no new content can be
added to any page (confirmed 2026-09-14 while working TIE-30/31/32; this is
the same blocker TIE-30 already hit once before). The decisions below are
**settled and implemented**; only the write into their Notion IRD is
pending. Once the workspace has space again (plan upgrade or freed blocks),
paste each section into the named IRD page and delete it from this file.

---

## → IRD-001 — Deploy Service Requirements (TIE-32)

**Add to "1. Decision", after item 5:**

6. `GET /overview` reads a single Redis cache key (`deploy-service:overview:v1`) with a 30-second TTL; any Redis failure or command timeout falls back to PostgreSQL immediately, with no retry.
7. `ci_run_id` uniqueness (DOP-001 AC-07) is enforced by a database-level UNIQUE constraint on `deploy_db`, not an application-only check — this is what makes idempotency safe under concurrent CI retries.

**Add as new "6.1 Overview Cache Policy", right after the "6. Release Flow" bullets (before "7. API"):**

| Setting | Value |
|---|---|
| Cache key | `deploy-service:overview:v1` |
| TTL | 30 seconds |
| On Redis failure/timeout | Fall back to PostgreSQL immediately, no retry |
| Invalidation | None on write — relies on the 30s TTL expiry (acceptable given the bound) |

**Add right after the "8. Data Model" table, before "9. Service Communication Boundary":**

`ci_run_id` carries a database-level `UNIQUE` constraint — this is the actual enforcement mechanism for AC-07 idempotency (see Decision #7), not just an application-level check.

Source: TIE-11 (implemented), TIE-12 (db-level idempotency).

---

## → IRD-003 — API Gateway, Infrastructure & Work Management (TIE-30)

**Add as new "3.2 Authentication Configuration (added per TIE-9)", right after "3.1 Timeouts":**

| Mechanism | Detail |
|---|---|
| JWT validation | HS256, secret from runtime env var `JWT_SECRET` |
| CI authentication | `X-API-Key` header, value from runtime env var `CI_API_KEY` |
| Deploy Service upstream | `DEPLOY_SERVICE_URL=http://deploy-service:3001` |
| Log Service upstream | `LOG_SERVICE_URL=http://log-service:3002` |

Credentials are never committed — `.env.gateway.example` documents variable names only; real values are generated per-environment (`infra/scripts/generate-secrets.sh`, per TIE-24 / IRD-003 §6).

**DOP-001 FR-4 check:** already states "API Gateway ... loads authentication configuration from environment variables" — verified, no DOP change needed.

Source: TIE-9 implementation plan and resolved Q&A.

---

## → IRD-003 — API Gateway, Infrastructure & Work Management (TIE-31)

**Add as new "6.1 PostgreSQL Least-Privilege Grants (added per TIE-6)", right after the "6. Infrastructure Requirements" bullets (before "7. Definition of Done"):**

- Revoke all database privileges from `PUBLIC`.
- Grant each service role `CONNECT` only to its own database.
- Explicitly revoke `CONNECT` from each other service's database.
- Revoke schema privileges from `PUBLIC`; grant the owning service role `USAGE` and `CREATE` on its own `public` schema for migrations.

Implemented in `infra/init/01-databases.sh`. Verified against a fresh PostgreSQL 16 container.

Source: TIE-6 implementation and verification.
