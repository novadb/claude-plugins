---
name: update-job
description: "Update job state or retention settings."
user-invocable: false
allowed-tools: job_update
---

# Update Job

Update job state or retention settings.

## Scope

**This skill ONLY handles:** Updating job state (kill/restart) or retention settings.

**For full job details** → use `get-job`

## Tools

1. `job_update` — Update the job

## Parameters

```json
{
  "jobId": 12345,
  "state": 0,
  "retainUntil": "2026-12-31T23:59:59Z"
}
```

- `jobId` — Job ID (int, required)
- `state` — New state (optional, see valid transitions below)
- `retainUntil` — ISO 8601 date-time until which to retain the job (optional)

## Valid State Transitions

> **Breaking change from the old MCP:** The `state` codes accepted by `job_update` changed. The old TS MCP used `4 = KillRequest` / `5 = RestartRequest`; the new C# MCP uses `0 = KillRequest` / `1 = RestartRequest`. Do not pass `4` or `5` — they will either be silently ignored or produce an error.

Only two state values can be set via update:

| Value | State | Purpose |
|-------|-------|---------|
| 0 | KillRequest | Request to stop a running job |
| 1 | RestartRequest | Request to restart a job |

Note: the **read-side** state enum (returned by `job_query`) still uses the original full range: 0=New, 1=Running, 2=Succeeded, 3=Error, 4=KillRequest, 5=RestartRequest. Don't confuse the two — the write-side uses only 0/1.

## Response

Returns the updated `CmsJob`.

## Common Patterns

### Valid State Transitions (write-side)
Only `0` (KillRequest) and `1` (RestartRequest) can be set manually. Other read-side states are managed by the system.

### API Response (job_update)
Returns the updated `CmsJob`.
