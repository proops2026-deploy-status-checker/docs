# Document Index — Deploy Status Checker

The map of every governing document. Read the DOP first, then the IRD for the service you are working on.

Notion hub: [🚀 Mock Project — Deploy Status Checker](https://app.notion.com/p/3c0b4aef823981eaa3f4eb1190aa0b02)

## Product definition (Notion)

| Doc | Scope | Status | Link |
|---|---|---|---|
| DOP-001 | Whole product — problem, users, services, 10 acceptance criteria | Draft (Day 9) | [DOP-001](https://app.notion.com/p/3c0b4aef823981da852cd954bedee694) |

## Implementation requirements (Notion)

| Doc | Scope | Status | Link |
|---|---|---|---|
| IRD-001 | `deploy-service` — API contract, data model, indexes, state machine, tests | Draft (Day 9) | [IRD-001](https://app.notion.com/p/3c0b4aef82398132be03c5a71d7a1385) |
| IRD-002 | `log-service` — same, plus the cross-service reference rules | Draft (Day 9) | [IRD-002](https://app.notion.com/p/3c0b4aef8239817588c1d1fd167f12f9) |
| IRD-003 | `api-gateway` + infrastructure — routing, auth middleware, Compose, NFRs | Draft (Day 9) | [IRD-003](https://app.notion.com/p/3c0b4aef823981cd96cfdfa7b07fc72b) |
| IRD-004 | CI/CD pipeline spec | Due Day 12 | — |
| IRD-005 | Monitoring and logging spec | Due Day 16 | — |

> **Numbering note.** This `DOP-001` / `IRD-001…003` series belongs to the **mock project** and lives in
> its own Notion section. It is unrelated to the training-programme `DOP-001…005` / `IRD-001…010`
> series in the ProOps2026 workspace. Two parallel series, deliberately kept apart.

## Design artifacts (this repo)

| File | Contents | Status |
|---|---|---|
| [architecture.md](architecture.md) | Mermaid diagram · 10 settled decisions · ownership table · known blind spot | Settled at Q2–Q5 |
| [sprint-01.md](sprint-01.md) | Sprint 1 backlog — epics, user stories, tasks, Definition of Done | Draft for Scrum Master review |

## Rules for this document system

1. **Write the IRD before the code.** Starting to code without an IRD means guessing.
2. **One IRD per service or topic.** If an IRD passes ~150 lines, split it.
3. **The documents are the source of truth, not the chat history.** Update the IRD, then implement.
4. **Cross-service references store the ID only** — never a foreign key into another service's table.
5. **If a decision is not covered by the DOP or an IRD, stop and ask.** Do not guess.
