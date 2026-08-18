# Deploy Status Checker — Documentation

Document repository for the **Deploy Status Checker** mock project, ProOps2026 Week 2+.

| Property | Value |
|---|---|
| **Organization** | [proops2026-deploy-status-checker](https://github.com/proops2026-deploy-status-checker) |
| **Team** | Nguyen Hoang Thao Tien (`tien_nht`) + Chau Phuoc Duc (`duc_cp`) |
| **Started** | Day 9 — 2026-08-18 |
| **Program** | ProOps2026 · Week 2 |

## Team and roles

Roles rotate each sprint so both engineers practise both disciplines. Both members are Developers
at all times — the role below is what each person owns *in addition* to doing the work.

| Person | Role — Sprint 1 | Owns |
|---|---|---|
| **Nguyen Hoang Thao Tien** · `tien_nht` | Product Owner · Developer | DOP-001 and the IRDs. Decides what enters a sprint and — more importantly — what stays out. Final say on scope. |
| **Chau Phuoc Duc** · `duc_cp` | Scrum Master · Developer | The process. Builds the sprint backlog, assigns tasks, keeps the sprint moving, runs the review. |

**Sprint 2 onward:** the roles swap.

### What "Product Owner" means on this team

The PO is the one who says no. Two scope risks are already identified for this system — `log-service`
growing into a search engine, and Redis being added without a measurable reason. Any change to scope
goes through the PO and gets recorded in the DOP or the relevant IRD before it gets built.

### What "Scrum Master" means on this team

The SM owns flow, not content. Every task in the backlog is concrete, fits inside one day, and is
assigned to exactly one person by name — never "both". If a task is unclear or the load is unbalanced,
the SM fixes it before the sprint starts.

## What this system is

Deploy Status Checker tracks deployment state across multiple services. Each deploy record carries a
status, a timeline, and its associated logs. A dashboard shows live state per service and environment.

## Repositories in this organization

| Repository | Purpose |
|---|---|
| `docs` | This repo — architecture, sprint backlogs, design decisions |
| `api-gateway` | Single entry point; routing, JWT + API-key authentication |
| `deploy-service` | Owns deploy records and status transitions |
| `log-service` | Owns append-only log entries scoped to a deploy |

## Where the documents live

DOP and IRD documents live in **Notion** — see [INDEX.md](INDEX.md) for links.
Architecture and sprint planning live in this repo.
