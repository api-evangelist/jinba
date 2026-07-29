---
name: Execute a Jinba Toolbox tool
description: Discover a toolset and execute one of its published tools via the Jinba Toolbox REST API, handling auth, RFC 9457 errors, and rate limits.
api: openapi/jinba-toolbox-openapi.yml
operations: [listToolSets, listTools, runTool, getRun]
---

# Execute a Jinba Toolbox tool

Use the Jinba Toolbox API to run a published tool inside its sandbox.

## Auth
Send an organization-scoped API key as a Bearer token on every request:
`Authorization: Bearer jtb_xxxxxxxxxxxx`. Base URL: `https://toolbox-api.jinba.dev/v1`.

## Steps
1. **Find the toolset** — `listToolSets` (`GET /orgs/{orgId}/toolsets`) and pick the target `slug`.
2. **Find the tool** — `listTools` (`GET /orgs/{orgId}/toolsets/{slug}/tools`); read each tool's `inputSchema` so you send valid arguments.
3. **Execute** — `runTool` (`POST /orgs/{orgId}/toolsets/{slug}/tools/{toolSlug}/run`) with a body `{ "input": { ... } }` matching the tool's `inputSchema`. Optionally set `version` to pin a published semantic version for reproducibility.
4. **Read the result** — the `RunResult` has `success`, `output`, `logs`, `durationMs`, and a `runId`. On `success:false`, inspect `error.name` / `error.message` / `error.traceback`.
5. **Inspect history if needed** — `getRun` (`GET /orgs/{orgId}/runs/{runId}`) to fetch full run details later.

## Rules
- Errors are RFC 9457 Problem Details (`application/problem+json`): `type`, `title`, `status`, `detail`.
- On `429 Too Many Requests`, honor `Retry-After` and back off exponentially.
- To run a tool in a public toolset without org context, use the public route `POST /public/{orgSlug}/{toolsetSlug}/run/{toolSlug}` instead.
- Never hard-code the API key; read it from an environment variable.
