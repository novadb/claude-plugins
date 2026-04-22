---
name: get-job-logs
description: "Preview and search logs produced by a job."
user-invocable: false
allowed-tools: job_log, job_logsearch
---

# Get Job Logs

Preview or grep the logs produced by a job.

## Scope

**This skill ONLY handles:** Reading job logs in-band (preview head/tail, or regex search).

**For job progress** → use `get-job-progress`
**For job artifacts** → use `get-job-artifacts`

> **Behavior change from the old MCP:** The old `novadb_cms_get_job_logs` downloaded the whole log file to disk. The new C# MCP does not. It offers two focused modes:
> - `job_log(head, tail)` — returns the first N + last M lines in one shot (preview mode).
> - `job_logsearch(pattern, contextLines)` — regex grep with `-C` context lines, capped at 500 output lines.
>
> There is no way to download the full log file. If large dumps are needed, that gap has to be filled on the C# MCP side.

## Preview Mode

```json
{
  "jobId": 12345,
  "head": 20,
  "tail": 20
}
```

Tool: `job_log`.

- `jobId` — Job ID (int, required)
- `head` — Lines from the start (default 20)
- `tail` — Lines from the end (default 20)

### Response

```json
{
  "totalLineCount": 12345,
  "lines": ["...","...","...","[omitted N lines]","...","..."],
  "truncated": true
}
```

`truncated: true` indicates there is more log between the head and tail chunks.

## Search Mode

```json
{
  "jobId": 12345,
  "pattern": "ERROR|WARN",
  "contextLines": 3
}
```

Tool: `job_logsearch`.

- `jobId` — Job ID (int, required)
- `pattern` — Regex (case-insensitive)
- `contextLines` — Lines before/after each match (default 3, grep `-C`)

### Response

```json
{
  "totalLineCount": 12345,
  "lines": ["142: line", "143: matching line", "144: line"],
  "matchCount": 17,
  "truncated": false
}
```

Output is capped at 500 lines; `truncated: true` means more matches exist than were returned.

## Common Patterns

### When to pick which
- Quick status check or stack trace at the end → `job_log` with `tail: 50+`.
- Finding a specific error across a long log → `job_logsearch`.

### Full download
Not supported. If you need everything, iterate `job_logsearch` with narrow patterns, or ask for `job_log` capability to be extended.
