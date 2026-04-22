---
name: upload-file
description: "[UNSUPPORTED in current MCP] Start a file upload to NovaDB (single-chunk or first chunk of multi-chunk)."
user-invocable: false
allowed-tools: novadb_cms_upload_file
---

# Upload File

> ⚠️ **UNSUPPORTED IN THE CURRENT C# MCP.**
> This skill relies on `novadb_cms_upload_file`, which is not exposed by the current NovaDB MCP (Noxum.Nova.AI.Mcp). Calling it will fail. The content below is preserved for the day file-upload support is added back; do not attempt to invoke.

Start uploading a file to NovaDB. For single-chunk uploads, set `commit=true` to complete immediately.

## Scope

**This skill ONLY handles:** Starting a new file upload to NovaDB (single-chunk or first chunk of multi-chunk upload).

**For continuing a chunked upload** → use `upload-file-continue`
**For canceling an in-progress upload** → use `upload-file-cancel`

## Tools

1. `novadb_cms_upload_file` — Start the upload

## Parameters

```json
{
  "filename": "photo.jpg",
  "extension": "jpg",
  "commit": true
}
```

- `filename` — Dateiname ohne Pfad (z.B. `"photo.jpg"`) (required). Nur Dateiname, kein Pfad.
- `uploadName` — Override-Name für den Upload (optional, Standard: `filename`)
- `extension` — File extension **without dot** (e.g. `jpg`, `pdf`, `png`) (required)
- `commit` — `true` for single-chunk uploads (completes immediately), `false` for chunked uploads (required)

## Workflow

### Single-chunk upload (`commit: true`)

1. Call `novadb_cms_upload_file` with the filename and `commit: true`
2. Response returns `{ token, fileIdentifier }` — the `fileIdentifier` is the hash used to download the file via `get-file` (concatenate with extension, e.g. `5fe618811cca585a2826a2da06e3ce1b.jpg`)

### Chunked upload (`commit: false`)

1. Call `novadb_cms_upload_file` with the first chunk filename and `commit: false`
2. Response returns `{ token }` — use this token with `novadb_cms_upload_file_continue` for subsequent chunks
3. On the final chunk, call `novadb_cms_upload_file_continue` with `commit: true`

## Response

- `commit: true` → `{ token, fileIdentifier }` (`fileIdentifier` is the hash for downloading via `get-file`)
- `commit: false` → `{ token }` (upload token for continuing)
