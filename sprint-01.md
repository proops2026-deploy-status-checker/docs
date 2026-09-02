# Sprint 1 — Deploy Status Checker

**Duration:** Day 10 – Day 12
**Goal:** Docker Compose brings up all three services and infra with one command; each service passes its health check; Deploy Service and Log Service implement the minimal API defined in IRD-001/IRD-002.
**Team:** `tien_nht` (Product Owner) · `duc_cp` (Scrum Master)

Governing docs: [DOP-001](https://app.notion.com/p/3cfb4aef823981edb9b9e6e434231dbe) · [IRD-001](https://app.notion.com/p/3cfb4aef8239813e8537da5ffed83e37) · [IRD-002](https://app.notion.com/p/3cfb4aef823981ac9238d5065dd21d0a) · [IRD-003](https://app.notion.com/p/3cfb4aef82398176ac29c560aacec3df)

---

### Epic: Infra & Container Platform
**User Story:** As a DevOps engineer, I want a Docker Compose stack with health-checked services on isolated networks, so the system runs reproducibly on any machine.
- [duc_cp] Write `infra/docker-compose.yml` — `frontend`/`backend` networks, one entry per service, `depends_on` + health checks (IRD-003 §5–§6)
- [duc_cp] Write `infra/init/01-databases.sh` — creates `deploy_db` + `log_db`, one least-privilege user each, `REVOKE CONNECT` cross-service
- [tien_nht] Write `.env.*.example` templates for postgres, gateway, deploy-service, log-service (no real secrets committed)

### Epic: API Gateway
**User Story:** As a client/CI, I want a single entry point that authenticates and routes requests, so backend services are never reachable directly.
- [tien_nht] Scaffold Express app + multi-stage Dockerfile (pinned `node:24-alpine`, non-root, `HEALTHCHECK`) — `api-gateway`
- [tien_nht] Implement JWT (users) + API key (CI) auth middleware and the routing table from IRD-003 §4

### Epic: Deploy Service
**User Story:** As CI, I want to report and query deploy status, so deploy history is centrally tracked (IRD-001).
- [duc_cp] Scaffold Express + Prisma app + Dockerfile — `deploy-service`
- [duc_cp] Implement `POST/PATCH /deploys`, `GET /deploys`, `GET /deploys/:id`, `GET /overview`, `GET /health` per IRD-001 §7
- [tien_nht] Enforce the release-flow state machine (IRD-001 §6) and `ci_run_id` idempotency

### Epic: Log Service
**User Story:** As CI, I want to submit and retrieve deployment logs, so on-call engineers can investigate failures (IRD-002).
- [tien_nht] Scaffold Express + Prisma app + Dockerfile — `log-service`
- [duc_cp] Implement `POST/GET /deploys/:id/logs` with cursor pagination and `GET /health` per IRD-002 §7

---

**Definition of Done (Sprint 1):**
- `docker compose up -d --build` from `infra/` brings up all 5 containers, every one reporting `(healthy)`.
- Only `api-gateway`'s port is reachable from the host; `deploy-service`, `log-service`, `postgres`, `redis` are not.
- Every endpoint listed above returns the status codes defined in its IRD.
- `deploy_user` cannot connect to `log_db` and vice versa.
- No secret is hardcoded in any Dockerfile, compose file, or committed `.env` file.
