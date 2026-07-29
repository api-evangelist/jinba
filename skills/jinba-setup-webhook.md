---
name: Register and verify a Jinba Toolbox webhook
description: Register an organization webhook for tool-run and toolset events, send a test event, and verify the HMAC-SHA256 signature on incoming payloads.
api: openapi/jinba-toolbox-openapi.yml
operations: [createWebhook, testWebhook, listWebhooks]
---

# Register and verify a Jinba Toolbox webhook

Receive real-time events instead of polling.

## Auth
`Authorization: Bearer jtb_xxxxxxxxxxxx` (organization-scoped). Base URL: `https://toolbox-api.jinba.dev/v1`.

## Steps
1. **Register** — `createWebhook` (`POST /orgs/{orgId}/webhooks`) with `url` and an `events` array. Valid events: `tool.run.completed`, `tool.run.failed`, `toolset.published`, `member.added`, `member.removed`. Store the returned signing `secret` securely.
2. **Test** — `testWebhook` (`POST /orgs/{orgId}/webhooks/{id}/test`) to send a sample delivery to your endpoint.
3. **List / audit** — `listWebhooks` (`GET /orgs/{orgId}/webhooks`) to review registered endpoints and their `enabled` state.

## Verifying payloads
Every delivery carries `X-Webhook-Signature: sha256=<hex>` — an HMAC-SHA256 of the raw request body computed with the webhook secret. Recompute it and compare with a timing-safe equality check before trusting the payload envelope `{ event, timestamp, data }`.

## Rules
- Return `2xx` quickly, then process asynchronously.
- Deliveries may repeat — dedupe on `data.runId` or `timestamp`.
- Failed deliveries retry at 1 min / 5 min / 30 min; after 3 consecutive failures the webhook is auto-disabled and must be re-enabled.
