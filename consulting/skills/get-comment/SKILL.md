---
name: get-comment
description: "Fetch a single comment (via filtered comment_query)."
user-invocable: false
allowed-tools: comment_query
---

# Get Comment

## Scope

**This skill ONLY handles:** Fetching a single comment by object or author filter.

**For listing/filtering comments** → use `get-comments`

> **Note:** The new C# MCP does not expose a single-comment getter. Use `comment_query` with enough filters to narrow to a single result.

## Tool

`comment_query`

## Parameters

```json
{
  "branchId": 2100347,
  "objectId": 12345,
  "user": "jdoe",
  "take": 50
}
```

- `branchId` — Branch ID (int, optional)
- `objectId` — Object the comment is on (int, optional)
- `user` — Comment author (optional)
- `continueToken` — Opaque pagination token (optional)
- `take` — Page size (default 50)

## Response

Returns `CmsCommentList` with `comments[]`. Each comment contains id, body (HTML with user mentions as `<span>`), branch reference, object reference, author, timestamps, deleted status.

## Common Patterns

### API Response (comment_query)
Returns a list even when filtered to a single comment. Take the first match.
