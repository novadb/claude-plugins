---
name: create-job
description: "Create and start a new job on a branch."
user-invocable: false
allowed-tools: job_create, job_query
---

# Create Job

Create and start a new job on a branch.

## Scope

**This skill ONLY handles:** Creating and starting a new job on a branch.

**For monitoring after creation** → use `get-job-progress` or `get-job-logs`

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

> **Heads up:** The new C# MCP does not yet expose a job input file upload tool. Jobs that expect an input file (CSV import, etc.) cannot be started from this skill until the MCP adds that capability.

## Tools

1. `job_create` — Create the job
2. `job_query` — Fetch the created job (optional follow-up, filter by branchId)

## Parameters

```json
{
  "branchId": 2100347,
  "jobDefinitionId": 12345,
  "language": 201,
  "scopeIds": [100, 200],
  "objIds": [300, 400],
  "parameters": "[{\"name\":\"param1\",\"values\":[\"value1\"]}]"
}
```

- `branchId` — Branch ID (int, required)
- `jobDefinitionId` — Job definition ID (int, required)
- `language` — Language ID (int, required, e.g. 201 for EN)
- `scopeIds` — Scope object IDs (optional, int[])
- `objIds` — Object IDs to process (optional, int[])
- `parameters` — JSON-encoded **string** of `[{ name, values: [...] }]` (optional)

## Workflow

1. Call `job_create` with the parameters above.
2. The response returns the full `CmsJob` object with `id`, `state`, timestamps.
3. Optionally poll with `job_progress` / `job_log` / `job_query`.

## Response

Returns `CmsJob` with job id, state, definition, branch, timestamps.

## Common Patterns

### Job States (new numbering)
0=New, 1=Running, 2=Succeeded, 3=Error, 4=KillRequest, 5=RestartRequest.
Note: when **setting** state via `job_update`, only `0=KillRequest` and `1=RestartRequest` are valid values — these are different from the read-side enum.
