---
name: Create and publish a Jinba toolset version
description: Create a toolset and a tool, test the draft, publish an immutable semantic version, and set it as the active published version.
api: openapi/jinba-toolbox-openapi.yml
operations: [createToolSet, createTool, testTool, publishVersion, setPublishedVersion]
---

# Create and publish a Jinba toolset version

Author a toolset of tools and ship an immutable, versioned release that agents (and the MCP endpoint) can execute.

## Auth
`Authorization: Bearer jtb_xxxxxxxxxxxx` (organization-scoped). Base URL: `https://toolbox-api.jinba.dev/v1`.

## Steps
1. **Create the toolset** — `createToolSet` (`POST /orgs/{orgId}/toolsets`) with `slug`, localized `name`, a `sandbox` config (`provider: e2b|daytona`, `language`, `packages`, `resources.timeout`), `visibility`, and `tags`.
2. **Add a tool** — `createTool` (`POST /orgs/{orgId}/toolsets/{slug}/tools`) with `slug`, `name`, `inputSchema` + `outputSchema` (JSON Schema), and `code`.
3. **Test the draft** — `testTool` (`POST /orgs/{orgId}/toolsets/{slug}/tools/{toolSlug}/test`) runs the draft code without publishing; confirm `success:true` and expected `output`.
4. **Publish a version** — `publishVersion` (`POST /orgs/{orgId}/toolsets/{slug}/versions`) with a semantic `version` (e.g. `1.3.0`) and `releaseNotes`. Versions are immutable.
5. **Activate** — `setPublishedVersion` (`PUT /orgs/{orgId}/toolsets/{slug}/published-version`) with `{ "version": "1.3.0" }` so `run` and MCP calls resolve to it.

## Rules
- Publish before exposing via MCP — the MCP/run path executes the published version, not the draft.
- Bump the semantic version on every change; never mutate an existing version.
- Errors are RFC 9457 Problem Details; a member role can create/manage tools but cannot manage other members or billing.
