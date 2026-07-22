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

**Identity & Sharing**, and the first slice of the **Ledger**:

- **Auth** — register, login, refresh, logout, email verification, password reset, and the JWKS
  public keys (`/.well-known/jwks.json`) that resource servers use to verify access tokens.
- **Sessions** — list active logins, revoke one, sign out everywhere.
- **Sharing** — the single anonymous Share Link, and named View Grants.
- **Ledger (accounts + transactions)** — create/list accounts (kind + fixed currency, with a
  computed balance), and record transactions as balanced movements (`from → to`).

Still to come in the Ledger: balance-based reporting (net worth, spending, income), cross-currency
transactions, splits and metadata, and the read endpoints a viewer calls. Do not invent Ledger
fields — design them here first.

## Conventions

- **Money is a string on the wire**: `{ "amount": "12.34", "currency": "USD" }`. A JSON number is
  a JavaScript double and is already wrong. Posting amounts are non-negative; a balance may be
  negative (`"-5.00"`).
- Time is RFC 3339 UTC. Ids are UUIDs. Errors are RFC 7807 `application/problem+json`.

## Regenerating clients

Each frontend owns its own generation step (e.g. `openapi-generator`, `orval`) pointed at this
`openapi.yaml`. The generated output lives in the consuming repo and is git-ignored there. This
repo holds the spec, not the clients.

## Decisions behind this spec

The shape here follows the design captured in the backend's ADRs — see
`../life-tracker-backend/docs/adr/0005`…`0008` and `../life-tracker-backend/docs/identity/CONTEXT.md`.
Two details were chosen while writing the spec and are flagged inline in `openapi.yaml`; both
are now ratified:

1. **Register auto-logs-in** — it opens a Session and returns an access + refresh token,
   exactly like `/auth/login`. Email verification, when added later, will gate what an
   unverified User may do, not whether they are signed in.
2. **Refresh-token delivery: both tokens in the body** — every client gets `accessToken` and
   `refreshToken` in the JSON and stores them itself; `/auth/refresh` reads the refresh token
   from the body. Uniform and simple to build against.

   > **Pre-production hardening (tracked):** a refresh token in JS-reachable storage on web is
   > an XSS exposure. Before the web client faces real users, move browser/Electron refresh
   > delivery to an httpOnly, Secure, SameSite cookie (native keeps the body). A deliberate
   > future spec change — do not ship web without it.
