---
name: get-job
description: "Fetch a single job by its ID (via filtered job_query)."
user-invocable: false
allowed-tools: job_query
---

# Get Job

Fetch a single job by its ID.

## Scope

**This skill ONLY handles:** Fetching a single job by its ID including state, definition, and timestamps.

**For listing all jobs on a branch** → use `get-jobs`
**For progress / logs / artifacts** → use `get-job-progress`, `get-job-logs`, `get-job-artifacts`

## Tools

1. `job_query` — Fetch jobs (filter by branchId + createdBy or state to narrow)

> **Note:** The new C# MCP has no single-get job tool. To retrieve one job, call `job_query` with filters (branchId plus any combination of state/definitionRef/createdBy) and pick the matching row by id.

## Parameters

```json
{
  "branchId": 2100347,
  "take": 50
}
```

- `branchId` — Branch ID (int, optional but recommended)
- `definitionRef` / `state` / `triggerRef` / `createdBy` / `isDeleted` — optional filters to narrow the result
- `take` — Page size (default 50)
- `continueToken` — opaque pagination token from a prior response

## Response

Returns `CmsJobList` with `jobs[]` and a `continueToken`. Find the job whose `id` matches.

## Job States (read-side)

| Value | State |
|-------|-------|
| 0 | New |
| 1 | Running |
| 2 | Succeeded |
| 3 | Error |
| 4 | KillRequest |
| 5 | RestartRequest |

## Common Patterns

### Job States
Read side uses 0–5. When **setting** state via `job_update`, only `0=KillRequest` and `1=RestartRequest` are valid.
