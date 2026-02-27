---
name: get-file
description: "Download a file from NovaDB by name and save it to disk."
user-invocable: false
allowed-tools: novadb_cms_get_file
---

# Get File

Download a file from NovaDB by its file identifier and save it to disk. Returns metadata (file path, size, content type) instead of file content.

## Scope

**This skill ONLY handles:** Downloading a file from NovaDB by its file identifier and saving it to disk.

**For uploading files** → use `upload-file`

## When to Use

- An object contains a **BinRef** attribute → read Attr 11000 (identifier) + 11005 (extension), call `get-file`.
- Search results or fetched objects reference **binary objects** → read the referenced object, extract Attr 11000 + 11005, call `get-file`.

## Tools

1. `novadb_cms_get_file` — Download the file and save to disk

## Parameters

```json
{
  "name": "5fe618811cca585a2826a2da06e3ce1b.jpg"
}
```

- `name` — File identifier (hash) with extension (e.g. `5fe618811cca585a2826a2da06e3ce1b.jpg`). For newly uploaded files this is the `fileIdentifier` returned by the upload API. For existing binary objects, read attribute **11000** for the identifier and **11005** for the extension, then concatenate them.
- `filename` — (Optional) Dateiname ohne Pfad (z.B. `"report.jpg"`). Nur angeben, wenn ein exakter Dateiname benötigt wird. Standard: der File-Identifier (`name`) wird als Dateiname verwendet.

## Workflow

1. Call `novadb_cms_get_file` with the file identifier and optionally a `filename`
2. The file is saved to disk
3. Metadata is returned: file path, size in bytes, and content type

## Response

JSON metadata object:

```json
{
  "filePath": "report.jpg",
  "sizeBytes": 12345,
  "contentType": "image/jpeg"
}
```

## Common Patterns

- Use `filename` to save files with meaningful names instead of hashes
- To find the file identifier on an existing binary object: read attributes 11000 (identifier hash) and 11005 (extension), then use `<attr11000><attr11005>` as the name (e.g. `5fe618811cca585a2826a2da06e3ce1b.jpg`)
- When generating HTML or documents that reference images/files: download **every** referenced file via `get-file`
