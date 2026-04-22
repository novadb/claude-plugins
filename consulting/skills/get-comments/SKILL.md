---
name: get-comments
description: "List comments with optional filters and pagination."
user-invocable: false
allowed-tools: comment_query
---

# Get Comments

## Scope

**This skill ONLY handles:** Listing and filtering comments with pagination support.

**For fetching a single comment by ID** → use `get-comment`
**For full-text search across comments** → use the `comment_search` tool directly

List comments with optional filters. Results are paginated.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`comment_query`

## Parameters

```json
{
  "branchId": 2100347,
  "objectId": 12345,
  "user": "jdoe",
  "take": 10,
  "continueToken": "opaque-token"
}
```

- `branchId` — (optional) Filter by branch ID (int)
- `objectId` — (optional) Filter by object ID (int)
- `user` — (optional) Filter by comment author
- `take` — (optional) Page size (default: 50)
- `continueToken` — (optional) Opaque continuation token from a previous response

All parameters are optional. Without filters, returns all comments.

## Pagination

The response includes a `continueToken` when more pages exist. Pass this token in the next call to fetch the next page.

## Response

Returns `CmsCommentList` with `comments[]` (id, HTML body, branch/object refs, author, timestamps, deleted status) and `continueToken`.

## Common Patterns

### Pagination
Uses opaque `continueToken` for next page.

### API Response (comment_query)
Returns array of comment objects with pagination. Check for `continueToken` in response for more results.
