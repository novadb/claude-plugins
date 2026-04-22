---
name: get-job-object-ids
description: "[UNSUPPORTED in current MCP] Fetch array of object IDs processed by a job."
user-invocable: false
allowed-tools: novadb_cms_get_job_object_ids
---

# Get Job Object IDs

> ⚠️ **UNSUPPORTED IN THE CURRENT C# MCP.**
> This skill relies on `novadb_cms_get_job_object_ids`, which is not exposed by the current NovaDB MCP (Noxum.Nova.AI.Mcp). Calling it will fail. The content below is preserved for the day this capability is added back; do not attempt to invoke.

Fetch array of object IDs processed by a job.

## Scope

**This skill ONLY handles:** Fetching the array of object IDs that were processed by a job.

**For full job details** → use `get-job`
**For job artifacts** → use `get-job-artifacts`

## Tools

1. `novadb_cms_get_job_object_ids` — Fetch processed object IDs

## Parameters

```json
{
  "jobId": "abc-123"
}
```

- `jobId` — Job ID (string, required)

## Response

Returns an array of object IDs that were processed by the job.

## Common Patterns

### API Response (GET Job Object IDs)
Returns an array of numeric object IDs processed by the job.
