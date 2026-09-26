# AGENTS.md — DrawMeLove

DrawMeLove is an ephemeral, room-based, realtime collaborative canvas for sending emotions by drawing together.
Flow: open → blank canvas + room code → draw → share link → draw together live → everyone leaves → room disappears.

Work order for building the product: `plan.md`. Review process: `docs/code-review-graph.md`. Agent work logs: append to `docs/logs/`.

## 1. Non-negotiable product rules
- Canvas first: opening `/` immediately gives a blank canvas and a fresh room. No landing page, no login, no accounts.
- Rooms are ephemeral: a room is destroyed shortly (grace ~45s) after its last participant leaves. No persistence, no history.
- Room codes: 4 chars, alphabet `ABCDEFGHJKMNPQRSTUVWXYZ23456789` (no `I L O 0 1`), case-insensitive, crypto-random, collision-checked against active rooms.
- Shareable URL: `/r/{CODE}`. The copy button copies the full URL.
- Optional privacy: 4-digit room password, stored only as a PBKDF2 hash. Never log it, broadcast it, or put it in a URL.
- Mobile is a first-class screen: every feature must work on a phone with touch.

## 2. Architecture (fixed)
- Modular monolith, one process, four projects. No microservices.
  - `DrawMeLove.Domain` — Room, CanvasState, CanvasOperation, value objects. No dependencies.
  - `DrawMeLove.Application` — RoomManager, cleanup, validation, room access. Depends on Domain.
  - `DrawMeLove.Api` — REST controllers + SignalR hub. Depends on Application.
  - `DrawMeLove.Infrastructure` — password hashing, clock. Depends on Application/Domain.
- Active rooms + canvas live ONLY in memory (`ConcurrentDictionary` in RoomManager). PostgreSQL is NOT in the realtime path and NOT needed for the MVP.
- Frontend: `client/drawmelove-web` (React + TS + Vite). The canvas engine is plain TS classes, fully decoupled from React renders.

## 3. Realtime rules
- Transport: SignalR hub at `/hubs/draw`, SignalR groups = rooms (`room:{CODE}`), JSON protocol first.
- Sync by operations, never screenshots: `stroke | text | sticker | erase | undo | redo | clear`, with a server-authoritative `CanvasVersion`.
- Local-first rendering: pointer events render immediately; the network never blocks drawing. Batch points (~50ms chunks); never send every pointer event.
- New joiners receive a full snapshot (ops + version), then live ops. Reconnect = rejoin + version check + resync if behind.
- The server validates every operation: type, ids, colors, sizes, coordinates, point counts, text length, total message size. Never trust the client.
- SignalR groups are not authorization. Validate room access on every `JoinRoom` and every operation.

## 4. Security rules
- Rate-limit join + password endpoints per IP+room (4 digits = 10,000 combinations). Constant-time hash verification. No oracle hints.
- Friendly errors only; no stack traces to clients. Log room lifecycle and auth failures; never log passwords, hashes, tokens, or canvas content.

## 5. Dependency policy
- Frontend deps: `react`, `react-dom`, `react-router-dom`, `@microsoft/signalr`; dev: `vite`, `typescript`, `eslint`, `vitest`. Nothing else without a written reason.
- Backend: ASP.NET Core built-ins only (DI, logging, rate limiting, SignalR). No MediatR, AutoMapper, FluentValidation, Redis, queues, gRPC.
- No Redux. React state + tiny stores; canvas state lives in the engine.

## 6. Commands
- Backend: `dotnet build src/DrawMeLove.sln && dotnet test` (requires .NET 8 SDK — plan task D1.1 installs it).
- Frontend: `cd client/drawmelove-web && npm ci && npm run typecheck && npm run build && npm test`.

## 7. Working rules for agents
- Work from `plan.md`: exactly one task per branch `feature/<task-id>` → one PR. Never start a task before its `Depends` are merged.
- Keep diffs small. No drive-by refactors, no scope expansion, no new dependencies. If blocked, stop and log the blocker.
- Append a work log `docs/logs/YYYY-MM-DD-<agent>-<task-id>.md`: task, decisions, output, verification, blockers. Max 40 lines.
- After each phase, run the review pipeline in `docs/code-review-graph.md`. Blocking findings must be fixed before the next phase.
- Two-client behavior (phone + desktop) is part of every feature's acceptance.

## 8. Out of scope — do not build
Accounts, profiles, chat, feeds, likes, comments, persistent drawings, AI features, vector editing, microservices, Redis, message queues, gRPC, Kubernetes, a database for MVP.

## 9. Definition of Done (per feature)
Works on mobile touch and desktop mouse · works with 2+ connected clients · handles reconnect · validates server-side · no new dependencies · tests included and green · UI stays minimal · work logged.
