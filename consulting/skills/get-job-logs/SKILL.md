---
name: get-job-logs
description: "Fetch logs produced by a job."
user-invocable: false
allowed-tools: novadb_cms_get_job_logs
---

# Get Job Logs

Fetch logs produced by a job and save to disk.

## Scope

**This skill ONLY handles:** Fetching the logs produced by a job.

**For job progress** → use `get-job-progress`
**For job artifacts** → use `get-job-artifacts`

## Tools

1. `novadb_cms_get_job_logs` — Fetch job logs and save to disk

## Parameters

```json
{
  "jobId": "abc-123",
  "targetPath": "job-abc-123.txt"
}
```

- `jobId` — Job ID (string, required)
- `targetPath` — (Optional) Relative path, e.g. `"job-abc-123.txt"` or `"logs/job-abc-123.txt"`. Default: `job-<jobId>-logs.txt`. Subdirectories are created automatically. Absolute paths and path traversal (`../`) are rejected.

## Response

JSON metadata object:

```json
{
  "filePath": "job-abc-123.txt",
  "sizeBytes": 54321,
  "contentType": "application/octet-stream"
}
```

## Common Patterns

- Use `targetPath` to save logs with meaningful names
- Always provide a `targetPath` to control where the log file is saved
- Parent directories are created automatically if they don't exist
- Read the saved file to inspect log contents
