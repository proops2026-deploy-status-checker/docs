# Sprint 1 — Deploy Status Checker

> **Status: draft, awaiting Scrum Master review.** `duc_cp` owns this backlog. Both members must
> agree the assignments before Day 10 begins. Rebalance freely — the constraint is that every task
> stays under one day and belongs to exactly one person.

| Field | Value |
|---|---|
| **Duration** | Day 10 – Day 12 (3 days) |
| **Goal** | `deploy-service` records deploy events and answers the on-call query, end to end, in a container |
| **Scope** | `deploy-service` only — not the gateway, not `log-service` |
| **Product Owner** | Nguyen Hoang Thao Tien · `tien_nht` |
| **Scrum Master** | Chau Phuoc Duc · `duc_cp` |

Both members are Developers. Roles swap in Sprint 2.

## Acceptance criteria in scope

Covers **DOP-001 AC 1, 2, 3, 4, 5, 6, 8, 9**.

Explicitly **not** in Sprint 1:
- **AC 7** (log retrieval) — belongs to `log-service`, Sprint 2
- **AC 10** (overview survives Redis outage) — `/overview` and the Redis projection are Sprint 2

---

## Epic: deploy-service

### US-01 — As a CI pipeline, I want to report a deploy so the team has a record of it
Satisfies AC 1, AC 2, AC 8.

- [ ] `Tien` Scaffold Express + TypeScript, fail-fast config loader that exits non-zero on any missing required env var
- [ ] `Duc` Prisma schema and migration `001_init` for `deploys` — all six CHECK constraints plus `UNIQUE (service, environment, ci_run_id)`
- [ ] `Tien` `POST /deploys` handler with validation: `environment` enum, non-empty `version`, reject a `service` key in the body with `400 SERVICE_NOT_ACCEPTED`
- [ ] `Duc` Idempotency path: `X-Idempotency-Key` required; on unique conflict return `200` with the existing record instead of `201`
- [ ] `Duc` Integration test: create returns `201`, replay returns `200`, row count stays at 1
- [ ] `Tien` Unit test: body containing `service` returns `400`; missing idempotency key returns `400`

### US-02 — As a CI pipeline, I want to report the outcome so the record reflects reality
Satisfies AC 3, AC 4, AC 9.

- [ ] `Tien` State machine module with the transition table; illegal transition raises `409 ILLEGAL_TRANSITION`
- [ ] `Tien` `PATCH /deploys/:id` — the `UPDATE` must carry `AND service = <injected name>`; row count 0 returns `404`
- [ ] `Duc` Integration test: `STARTED → SUCCESS` sets `finished_at`; `SUCCESS → STARTED` returns `409` and leaves the row unchanged
- [ ] `Duc` Integration test: a foreign service name returns `404` and the target row is untouched

### US-03 — As an on-call engineer, I want to know what was deployed to an environment in a time window
Satisfies AC 5, AC 6. **This is the survival query — the reason the product exists.**

- [ ] `Duc` Migration `002_indexes`: `idx_deploys_env_started` plus the production partial index
- [ ] `Tien` `GET /deploys` — `since` required, window capped at 7 days, `limit` default 50 / max 200, violations return `400`
- [ ] `Tien` `GET /deploys/:id` — `200` or `404`
- [ ] `Duc` Integration test: filters by environment and window, newest first, and an empty window returns `200 []` — never `404`
- [ ] `Duc` Verify with `EXPLAIN ANALYZE` on seeded data that the query shows an index scan with **no `Sort` and no `Seq Scan`**

### Cross-cutting

- [ ] `Duc` `GET /health` returning `200` / `503` when PostgreSQL is unreachable, plus structured JSON logging carrying the request id
- [ ] `Tien` Dockerfile to the IRD-003 standard — multi-stage, pinned `node:24.19.0-alpine`, `USER node`, `HEALTHCHECK` — plus `.dockerignore`
- [ ] `Tien` `docker-compose.yml` with a PostgreSQL healthcheck and `deploy-service` gated on `condition: service_healthy`

---

## Load check

| Person | Tasks |
|---|---|
| `Tien` | 8 |
| `Duc` | 9 |

Seventeen tasks across three days for two people is tight. If the sprint slips, the Product Owner
defers the two Docker tasks first — they are prerequisites for Day 11, not for the acceptance
criteria. **The query path (US-03) is never the thing that gets cut.**

---

## Definition of Done

Sprint 1 is complete when: migrations create the `deploys` table with every CHECK and UNIQUE
constraint in place; `POST`, `PATCH`, `GET /deploys` and `GET /deploys/:id` all return the status
codes named in IRD-001, including `409` on an illegal transition and `200 []` on an empty window;
a CI retry provably creates no duplicate row; a foreign service name cannot alter another service's
deploy; every test above passes; `EXPLAIN ANALYZE` shows the survival query using an index with no
sort; and `docker compose up` brings PostgreSQL and `deploy-service` to a healthy state with one
command and no hardcoded configuration.
