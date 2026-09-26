# CLAUDE.md — DrawMeLove

Read `AGENTS.md` first — it is the source of truth for product rules, architecture, and agent working rules.

## Where things are
- `plan.md` — the build plan: phases, tasks with IDs/dependencies/acceptance criteria, sized for one autonomous agent per task.
- `docs/code-review-graph.md` — review pipeline + hotspots. Run it at every phase gate and PR.
- `docs/logs/` — every agent appends a work log here (`YYYY-MM-DD-<agent>-<task-id>.md`).
- `docs/` — architecture, realtime protocol, deployment, security docs (created by plan tasks).

## Commands
- Backend (after D1.1 installs the .NET 8 SDK): `dotnet build src/DrawMeLove.sln && dotnet test`
- Frontend: `cd client/drawmelove-web && npm ci && npm run typecheck && npm run build && npm test`

## Working rules (Claude-specific)
- Execute one `plan.md` task per branch/PR (`feature/<task-id>`); respect its `Depends` list.
- Use subagent teams for multi-task phases; log each agent's work to `docs/logs/`.
- When `plan.md` doesn't answer a question, follow `AGENTS.md` principles, pick the simplest option, and record the decision in the log.
