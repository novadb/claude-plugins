---
name: get-job-progress
description: "Get current progress of a running job."
user-invocable: false
allowed-tools: job_progress
---

# Get Job Progress

Get the current progress of a running job.

## Scope

**This skill ONLY handles:** Fetching the current progress of a job.

**For full job details** → use `get-job`
**For job logs** → use `get-job-logs`

## Tools

1. `job_progress` — Fetch job progress

## Parameters

```json
{
  "jobId": 12345
}
```

- `jobId` — Job ID (int, required)

## Response

Returns `JobProgress`:

```json
{
  "progress": 0.42,
  "message": "Processing 42/100"
}
```

- `progress` — Fraction 0.0–1.0 (multiply by 100 to present as percentage).
- `message` — Human-readable status message.

## Common Patterns

### API Response (GET Job Progress)
Returns progress fraction plus a message. Poll while the job is running; stop once the read-side `state` of the job is 2 (Succeeded) or 3 (Error).
