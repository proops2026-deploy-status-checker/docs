# Document Index — Deploy Status Checker

The map of every governing document. Read the DOP first, then the IRD for the service you are working on.

## Product definition (Notion)

| Doc | Scope | Status | Link |
|---|---|---|---|
| DOP-001 | Whole product — problem, users, services, acceptance criteria | ⬜ Not written | _(add Notion URL)_ |

## Implementation requirements (Notion)

| Doc | Scope | Status | Link |
|---|---|---|---|
| IRD-001 | `deploy-service` — API contract, data model, validation, tests | ⬜ Not written | _(add Notion URL)_ |
| IRD-002 | `log-service` — same structure + cross-service reference rule | ⬜ Not written | _(add Notion URL)_ |
| IRD-003 | `api-gateway` + infrastructure — routing, auth, Compose, NFRs | ⬜ Not written | _(add Notion URL)_ |

Later IRDs are added as new topics begin — IRD-004 CI/CD (Day 12), IRD-005 monitoring (Day 16).

## Design artifacts (this repo)

| File | Contents | Status |
|---|---|---|
| [architecture.md](architecture.md) | Mermaid diagram — services, storage, sync vs async flows | 🔄 Draft |
| [sprint-01.md](sprint-01.md) | Sprint 1 backlog — epics, user stories, tasks, DoD | ⬜ Not written |

## Rules for this document system

1. **Write the IRD before the code.** Starting to code without an IRD means guessing.
2. **One IRD per service or topic.** If an IRD passes ~150 lines, split it.
3. **The documents are the source of truth, not the chat history.** Update the IRD, then implement.
4. **Cross-service references store the ID only** — never a foreign key into another service's table.
