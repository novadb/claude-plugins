---
name: delete-job
description: "Delete a job by its ID."
user-invocable: false
allowed-tools: job_delete
---

# Delete Job

Delete a job by its ID.

## Scope

**This skill ONLY handles:** Deleting an existing job by its ID.

**For finding the job to delete** → use `get-job` or `get-jobs`

## Tools

1. `job_delete` — Delete the job

## Parameters

```json
{
  "jobId": 12345
}
```

- `jobId` — Job ID (int, required)

## Response

Returns a short confirmation string (e.g. `"Deleted job 12345"`).

## Common Patterns

### API Response (DELETE)
Returns confirmation of deletion.
