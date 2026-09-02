# Architecture — Deploy Status Checker

See [DOP-001](https://app.notion.com/p/3cfb4aef823981edb9b9e6e434231dbe) for the full design. This diagram matches DOP-001 §4.

```mermaid
graph TD
  Client([Client / CI])

  subgraph frontend
    GW["api-gateway<br/>:3000"]
  end

  subgraph backend["backend - internal network"]
    DS["deploy-service<br/>:3001"]
    LS["log-service<br/>:3002"]
    PG[("PostgreSQL 16<br/>deploy_db - log_db")]
    RD[("Redis 7")]
  end

  Client -->|HTTP REST| GW
  GW -->|/deploys/*, /overview| DS
  GW -->|/deploys/:id/logs| LS
  DS --> PG
  LS --> PG
  DS -.->|cache, best-effort| RD
```

- Only `api-gateway` is reachable from outside the `backend` network.
- `deploy-service` and `log-service` never call each other directly.
- `deploy-service` owns `deploy_db`; `log-service` owns `log_db` — no cross-service database access.
- Redis is a cache only; every read path still works with Redis stopped (falls back to PostgreSQL).

Stack versions (locked, see DOP-001 §4): TypeScript / Node.js 24 / Express 5 / Prisma 6, PostgreSQL 16, Redis 7.
