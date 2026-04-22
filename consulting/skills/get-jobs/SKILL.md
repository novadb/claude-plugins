---
name: get-jobs
description: "List jobs for a branch with optional filters and pagination."
user-invocable: false
allowed-tools: job_query
---

# Get Jobs

List jobs for a branch with optional filters and pagination.

## Scope

**This skill ONLY handles:** Listing and filtering jobs for a branch with pagination support.

**For fetching a single job by ID** → use `get-job`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tools

1. `job_query` — List jobs

## Parameters

```json
{
  "branchId": 2100347,
  "definitionRef": 12345,
  "state": 2,
  "triggerRef": 100,
  "createdBy": "jdoe",
  "isDeleted": false,
  "take": 10,
  "continueToken": "opaque-token"
}
```

- `branchId` — Branch ID (int, optional — recommended)
- `definitionRef` — Filter by job definition ID (optional, int)
- `state` — Filter by state (optional, 0–5)
- `triggerRef` — Filter by trigger ID (optional, int)
- `createdBy` — Filter by creator username (optional)
- `isDeleted` — Filter by deleted status (optional)
- `take` — Page size (default 50)
- `continueToken` — Opaque continuation token from the previous response (optional)

## Job States (read-side)

| Value | State |
|-------|-------|
| 0 | New |
| 1 | Running |
| 2 | Succeeded |
| 3 | Error |
| 4 | KillRequest |
| 5 | RestartRequest |

## Pagination

The response includes a `continueToken` when more pages exist. Pass this in the next call. Omit for the first page.

## Response

Returns `CmsJobList` with `jobs[]` and optional `continueToken`.

## Common Patterns

### Pagination
Uses opaque `continueToken` for the next page.

### API Response (GET Jobs)
Returns `{ jobs: [...], continueToken?: "token" }`.
