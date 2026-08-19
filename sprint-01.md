# Sprint 1 — Deploy Status Checker

> **Status: draft, awaiting Scrum Master review.** `duc_cp` owns this backlog. Both members must
> agree the assignments before Day 10 begins. Rebalance freely — the constraint is that every task
> stays under one day and belongs to exactly one person.

| Field | Value |
|---|---|
| **Duration** | Day 10 – Day 16 · **1 week** · Tue 18 Aug → Mon 24 Aug 2026 |
| **Goal** | `deploy-service` records deploy events and answers the on-call query, end to end, in a container |
| **Scope** | `deploy-service` only — not the gateway, not `log-service` |
| **Product Owner** | Nguyen Hoang Thao Tien · `tien_nht` |
| **Scrum Master** | Chau Phuoc Duc · `duc_cp` |

Both members are Developers. Roles swap in Sprint 2.

## Acceptance criteria in scope

Covers **DOP-001 AC 1, 2, 3, 4, 5, 6, 8, 9**. AC 11 is schema-only in this sprint — see US-04.

Explicitly **not** in Sprint 1:
- **AC 7** (log retrieval) — belongs to `log-service`, Sprint 2
- **AC 10** (overview survives Redis outage) — `/overview` and the Redis projection are Sprint 2
- The `/overview` half of **AC 11** — it needs the projection, so Sprint 1 proves the rollback protocol at the API and data level only

---

## Epic: deploy-service

### US-01 — As a CI pipeline, I want to report a deploy so the team has a record of it
Satisfies AC 1, AC 2, AC 8.

- [ ] `Tien` Scaffold Express + TypeScript, fail-fast config loader that exits non-zero on any missing required env var
- [ ] `Duc` Prisma schema and migration `001_init` for `deploys` — all six CHECK constraints, `UNIQUE (service, environment, ci_run_id)`, the `rolled_back_from` self-FK and its not-self CHECK. Generate with `prisma migrate dev --create-only`, then hand-edit the SQL: Prisma cannot express CHECK constraints or partial indexes
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
- [ ] `Tien` `GET /deploys` — `since` required, window capped at 7 days, `limit` default 50 / max 200, **composite cursor** (`started_at` + `id`) with `next_cursor` in the response; violations return `400`
- [ ] `Tien` `GET /deploys/:id` — `200` or `404`
- [ ] `Duc` Integration test: filters by environment and window, newest first, and an empty window returns `200 []` — never `404`
- [ ] `Duc` Verify with `EXPLAIN ANALYZE` on seeded data that the query shows an index scan with **no `Sort` and no `Seq Scan`**

### US-04 — As an on-call engineer, I want a rollback to show both what failed and what is running now

> **DEFERRED TO SPRINT 2** by Product Owner decision — see Load check. The `rolled_back_from` column stays in migration `001_init` (free on an empty table); everything below moves.
Satisfies the data and endpoint half of AC 11.

- [ ] `Tien` Accept `rolled_back_from` on `POST /deploys`; validate it references a deploy of the same service and environment, else `400`
- [ ] `Tien` Reject a `status` field in the `POST` body — every deploy is born `STARTED`
- [ ] `Duc` Integration test: PATCH to `ROLLED_BACK` then POST the restored version — the query returns both rows in time order
- [ ] `Duc` Integration test: a rollback POST reusing the original idempotency key returns `200` with the original record and creates nothing — proves why the `<run_id>-rollback-<n>` convention is mandatory

### Cross-cutting

- [ ] `Duc` `GET /health` returning `200` / `503` when PostgreSQL is unreachable, plus structured JSON logging carrying the request id
- [ ] `Tien` Dockerfile to the IRD-003 standard — multi-stage, pinned `node:24.19.0-alpine`, `USER node`, `HEALTHCHECK` — plus `.dockerignore`
- [ ] `Tien` `docker-compose.yml` with a PostgreSQL healthcheck and `deploy-service` gated on `condition: service_healthy`

---

## Load check — the sprint is over capacity, and here is the cut

| Person | Tasks |
|---|---|
| `Tien` | 11 |
| `Duc` | 11 |

**Sprint length changed from 3 days to 1 week before the sprint started.** The original three-day plan
assumed three full implementation days. It does not survive contact with the calendar: Day 10 is spent
learning Docker Compose, and Day 12 opens the CI/CD topic and owes IRD-004. That left roughly a day and a
half of real build time for eighteen tasks — which is not a plan, it is a wish. Changing the length **before**
starting is planning; changing it after starting would be the thing Scrum forbids, because a sprint whose
end date moves stops measuring anything.

Twenty-two tasks across three days for two people would have been **roughly 3.7 tasks per person per day**.
The honest reading: US-04 arrived after this backlog was first drafted, when the Day 9 quality gate exposed
the rollback ambiguity and AC-11 was added to DOP-001. New scope after planning has to displace something,
not stack on top.

**Product Owner decision — defer US-04 to Sprint 2**, with one exception:

- The `rolled_back_from` column, its self-FK and its not-self CHECK **stay in migration `001_init`**. Adding a column to an empty table is free; adding it later to a populated one is a migration with a backfill. Exactly the argument that put idempotency in Sprint 1.
- The endpoint validation and the four rollback tests move to Sprint 2, alongside `/overview` — which AC-11 needs anyway.

That leaves **18 tasks**, `Tien` 9 and `Duc` 9. Across a one-week sprint that is **about 1.3 tasks per person
per day** — achievable even though training runs in parallel and no day is a full build day.

**Cut order if it still slips**, decided now rather than on the last morning:

1. **US-02** (`PATCH`, the outcome path) goes first. On-call can still answer the question the product exists
   for without it; they simply see every deploy as `STARTED`. Losing it costs AC 3, 4, 9.
2. The two Docker tasks would have been the earlier cut, but Day 10 does them anyway — they are prerequisites
   for every following day, not for any single acceptance criterion.
3. **US-03 is never what gets cut.** It is the reason the product exists.
---

## Definition of Done

Sprint 1 is complete when: migrations create the `deploys` table with every CHECK and UNIQUE
constraint in place; `POST`, `PATCH`, `GET /deploys` and `GET /deploys/:id` all return the status
codes named in IRD-001, including `409` on an illegal transition and `200 []` on an empty window;
a CI retry provably creates no duplicate row; a foreign service name cannot alter another service's
deploy; every test above passes; `EXPLAIN ANALYZE` shows the survival query using an index with no
sort; and `docker compose up` brings PostgreSQL and `deploy-service` to a healthy state with one
command and no hardcoded configuration.
