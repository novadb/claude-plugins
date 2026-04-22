---
name: upload-file-continue
description: "[UNSUPPORTED in current MCP] Continue a chunked file upload with additional chunks."
user-invocable: false
allowed-tools: novadb_cms_upload_file_continue
---

# Upload File Continue

> ⚠️ **UNSUPPORTED IN THE CURRENT C# MCP.**
> This skill relies on `novadb_cms_upload_file_continue`, which is not exposed by the current NovaDB MCP (Noxum.Nova.AI.Mcp). Calling it will fail. The content below is preserved for the day chunked file upload support is added back; do not attempt to invoke.

Continue a chunked file upload that was started with `novadb_cms_upload_file`.

## Scope

**This skill ONLY handles:** Sending additional chunks for an in-progress file upload.

**For starting a new upload** → use `upload-file`
**For canceling an in-progress upload** → use `upload-file-cancel`

## Tools

1. `novadb_cms_upload_file_continue` — Send the next chunk

## Parameters

```json
{
  "filename": "chunk-file",
  "extension": "jpg",
  "commit": false,
  "token": "<upload-token>"
}
```

- `filename` — Dateiname ohne Pfad (z.B. `"chunk-file"`) (required). Nur Dateiname, kein Pfad.
- `uploadName` — Override-Name für den Upload (optional, Standard: `filename`)
- `extension` — File extension without dot (required)
- `commit` — `true` on the **final** chunk to complete the upload, `false` otherwise (required)
- `token` — Upload token returned by `novadb_cms_upload_file` (required)

## Workflow

1. Use the `token` from the initial `novadb_cms_upload_file` call
2. Call `novadb_cms_upload_file_continue` for each additional chunk
3. Set `commit: true` on the last chunk to finalize the upload

## Response

- `commit: false` → acknowledgment, continue sending chunks
- `commit: true` → `{ token, fileIdentifier }` (`fileIdentifier` is the hash for downloading via `get-file`)
