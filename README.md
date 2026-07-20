# life-tracker-contracts

The wire contract for Life Tracker. `openapi.yaml` is the **single source of truth** for every
field name and shape that crosses between the backend and any frontend. Nothing here is
hand-agreed by memory; it is agreed *here*.

## The rule

- To change the API: **edit `openapi.yaml` first**, regenerate clients, then change the code — in
  that order.
- Frontends **generate** their request/response types from this spec. Never hand-write a wire
  type in a frontend repo.
- Never hand-edit a generated client; it is overwritten on the next build.
- Need a field that isn't here? **Stop and design it in the spec.** A plausible guess is worse
  than a question, because it compiles.

## Scope today

This first cut covers **Identity & Sharing** only:

- **Auth** — register, login, refresh, logout.
- **Sessions** — list active logins, revoke one, sign out everywhere.
- **Sharing** — the single anonymous Share Link, and named View Grants.

The **Ledger** surface (transactions, accounts, balances, and the read endpoints a viewer
actually calls) is **not here yet** — it is a separate, larger design. Do not invent Ledger
fields in this file.

## Conventions

- **Money is a string on the wire**: `{ "amount": "12.34", "currency": "USD" }`. A JSON number is
  a JavaScript double and is already wrong. (No money in the identity slice; the rule bites the
  moment the Ledger surface lands.)
- Time is RFC 3339 UTC. Ids are UUIDs. Errors are RFC 7807 `application/problem+json`.

## Regenerating clients

Each frontend owns its own generation step (e.g. `openapi-generator`, `orval`) pointed at this
`openapi.yaml`. The generated output lives in the consuming repo and is git-ignored there. This
repo holds the spec, not the clients.

## Decisions behind this spec

The shape here follows the design captured in the backend's ADRs — see
`../life-tracker-backend/docs/adr/0005`…`0008` and `../life-tracker-backend/docs/identity/CONTEXT.md`.
Two details were chosen while writing the spec (not yet ratified) and are flagged inline in
`openapi.yaml`:

1. **Register does not auto-login** — it returns the new `User`; the client then calls
   `/auth/login`.
2. **Refresh-token delivery is split by client** — httpOnly cookie for browser/Electron, response
   body for React Native. `accessToken` is always in the body.
