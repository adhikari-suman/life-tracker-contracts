# life-tracker-contracts

The OpenAPI spec and the generated clients built from it. This repo decides the wire. The
backend and the frontends agree here, never by memory.

## Rules

- `openapi.yaml` is the source of truth. Change the API by editing it FIRST, then regenerating,
  then touching code — in that order.
- NEVER hand-write a request or response type in a frontend. Generate it from this spec.
- NEVER hand-edit a generated client. It is overwritten on the next build.
- Need a field that isn't in the spec? STOP and add it here. Do not invent it in a consumer — a
  plausible guess compiles, which is exactly why it is dangerous.

## Money crosses the wire as a string

`{ "amount": "12.34", "currency": "USD" }`. A JSON number is an IEEE 754 double in JavaScript, so
a numeric amount is corrupted before any client reads it. Holds in both directions, every repo.

## Editing the spec

- Keep it valid OpenAPI 3.0.3. A broken spec breaks every consumer's build at once.
- `operationId` is the generated method name — keep them stable and verb-shaped; renaming one is a
  breaking change to every client.
- `additionalProperties: false` on request bodies, so a typo'd field is rejected, not silently
  dropped.
- Errors are RFC 7807 (`Problem`). A domain exception maps to a status code in the backend's
  `@RestControllerAdvice`, never in the spec.

## Scope

Identity & Sharing only for now (auth, sessions, sharing). The Ledger surface is a separate,
later design — do not add Ledger endpoints or fields here until that is designed.
