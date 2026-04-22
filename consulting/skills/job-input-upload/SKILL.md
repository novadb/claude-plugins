---
name: job-input-upload
description: "[UNSUPPORTED in current MCP] Upload input file for a job with chunked upload support."
user-invocable: false
allowed-tools: novadb_cms_job_input_upload, novadb_cms_job_input_continue, novadb_cms_job_input_cancel
---

# Job Input Upload

> ⚠️ **UNSUPPORTED IN THE CURRENT C# MCP.**
> This skill relies on `novadb_cms_job_input_upload`, `novadb_cms_job_input_continue`, and `novadb_cms_job_input_cancel`, none of which are exposed by the current NovaDB MCP (Noxum.Nova.AI.Mcp). Jobs that require an input file cannot be started from Claude until this capability is added back. The content below is preserved for that future state; do not attempt to invoke.

Upload input file for a job with chunked upload support.

## Scope

**This skill ONLY handles:** Uploading input files for jobs, with support for chunked uploads.

**For creating the job after upload** → use `create-job`
**For general file uploads (not job input)** → use `upload-file`

## Tools

1. `novadb_cms_job_input_upload` — Start the upload (first chunk)
2. `novadb_cms_job_input_continue` — Continue with subsequent chunks
3. `novadb_cms_job_input_cancel` — Cancel an in-progress upload

## Parameters

### Start upload
```json
{
  "filename": "input.csv"
}
```

- `filename` — Dateiname ohne Pfad (z.B. `"input.csv"`) (required). Nur Dateiname, kein Pfad.
- `uploadName` — Override-Name für den Upload (optional, Standard: `filename`)

### Continue upload
```json
{
  "token": "upload-token-from-previous-call",
  "filename": "next-chunk"
}
```

### Cancel upload
```json
{
  "token": "upload-token-to-cancel"
}
```

## Workflow

### Single-chunk upload

1. Call `novadb_cms_job_input_upload` with the filename
2. Response returns `{ token, name }` — use these as the `inputFile` parameter when creating a job

### Chunked upload (large files)

1. Call `novadb_cms_job_input_upload` with the first chunk — response returns `{ token }`
2. For each subsequent chunk, call `novadb_cms_job_input_continue` with the token and next chunk filename
3. Use the final `{ token, name }` as the `inputFile` parameter when creating a job

### Cancel

Call `novadb_cms_job_input_cancel` with the token to abort an in-progress upload.

## Response

- **Upload/Continue** — Returns `{ token, name }` for use with `novadb_cms_create_job`'s `inputFile` parameter
- **Cancel** — Returns confirmation of cancellation
