---
name: get-job-artifacts
description: "List job artifacts. (Single-artifact and ZIP download are unsupported in the current MCP.)"
user-invocable: false
allowed-tools: job_artifacts
---

# Get Job Artifacts

List artifacts produced by a job.

## Scope

**This skill ONLY handles:** Listing the artifacts (paths + download URLs) for a completed job.

**For job logs** → use `get-job-logs`
**For processed object IDs** → use `get-job-object-ids` (currently unsupported)

> ⚠️ **Partial support.**
> - ✅ Listing artifacts is available via the new `job_artifacts` tool.
> - ❌ Downloading a specific artifact or downloading all artifacts as a ZIP is **not** exposed by the current C# MCP (the old `novadb_cms_get_job_artifact` and `novadb_cms_get_job_artifacts_zip` tools are gone). Only the listing is functional here.

## Tool

`job_artifacts` — List all artifacts for a job.

## Parameters

```json
{
  "jobId": 12345
}
```

- `jobId` — Job ID (int, required). The job must have artifacts enabled and be complete.

## Response

Returns `JobArtifactList`:

```json
{
  "artifacts": [
    { "path": "output/report.csv", "downloadUrl": "https://.../artifacts/report.csv", "sizeBytes": 12345 }
  ]
}
```

The `downloadUrl` can be opened in a browser or fetched by the user with their own HTTP client — the MCP itself does not download artifacts anymore.

## Fallback for downloads

Until a download tool is added back to the C# MCP, fetch artifacts out-of-band using the `downloadUrl` returned in the listing (curl, browser, etc.). Do not attempt to call the old `novadb_cms_get_job_artifact` / `..._zip` tools — they no longer exist.

## Common Patterns

### API Response
Returns a list of artifact metadata. Actual file transfers happen outside Claude.
