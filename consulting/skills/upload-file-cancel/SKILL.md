---
name: upload-file-cancel
description: "[UNSUPPORTED in current MCP] Cancel an in-progress chunked file upload."
user-invocable: false
allowed-tools: novadb_cms_upload_file_cancel
---

# Upload File Cancel

> ⚠️ **UNSUPPORTED IN THE CURRENT C# MCP.**
> This skill relies on `novadb_cms_upload_file_cancel`, which is not exposed by the current NovaDB MCP (Noxum.Nova.AI.Mcp). Calling it will fail. The content below is preserved for the day chunked file upload support is added back; do not attempt to invoke.

Cancel an in-progress chunked file upload and discard all uploaded data.

## Scope

**This skill ONLY handles:** Canceling an in-progress chunked file upload and discarding all uploaded chunks.

**For starting a new upload** → use `upload-file`

## Tools

1. `novadb_cms_upload_file_cancel` — Cancel the upload

## Parameters

```json
{
  "token": "<upload-token>"
}
```

- `token` — Upload token from `novadb_cms_upload_file` (required)

## Workflow

1. Call `novadb_cms_upload_file_cancel` with the upload token
2. The upload is cancelled and all uploaded chunks are discarded

## Response

Confirmation that the upload was cancelled.

## Common Patterns

### API Response (Cancel)
Returns confirmation that the upload has been canceled and all chunks discarded.
