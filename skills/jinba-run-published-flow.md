---
name: Run a published Jinba Flow workflow
description: Invoke a published Jinba Flow workflow synchronously or asynchronously via the Flow External API, passing named arguments and handling the result envelope.
api: openapi/jinba-flow-openapi.yml
operations: [runPublishedFlow]
---

# Run a published Jinba Flow workflow

Call a published natural-language workflow from any HTTP client.

## Auth
Each published flow has its own auto-generated API key. Send it as a Bearer token:
`Authorization: Bearer YOUR_API_KEY`. Base URL: `https://api.jinba.dev`.

## Steps
1. **Invoke** — `runPublishedFlow` (`POST /api/v2/external/flows/{flowId}/published-run`).
   Body: `args` is an array of `{ "name": ..., "value": ... }` pairs (values may be strings, numbers, booleans, objects, or arrays). Set `mode` to `sync` (block for the result) or `async` (return immediately with a run id).
2. **Read the result** — the response envelope is `{ status, result, stepOutputs }`. On failure, `status` is `failed` and `error` carries `{ name, value, traceback }`; per-step failures appear under `stepOutputs`.

## Rules
- Use `mode: async` for long-running workflows to avoid request timeouts.
- On `429`, honor `Retry-After` and back off exponentially.
- `401` = bad/missing key; `404` = flow not found, not published, or archived; `403` = key lacks access.
- Regenerating a flow's key invalidates the old key immediately — rotate consumers together.
