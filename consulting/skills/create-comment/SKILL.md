---
name: create-comment
description: "Create a new comment on an object."
user-invocable: false
allowed-tools: comment_create
---

# Create Comment

## Scope

**This skill ONLY handles:** Creating a new comment on an object in NovaDB.

**For updating existing comments** → use `update-comment`

Create a new comment on an object in NovaDB.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`comment_create` — Create the comment (returns the full comment object, no follow-up fetch needed)

## Parameters

```json
{
  "branchId": 2100347,
  "objectId": 12345,
  "message": "Great work!",
  "mentions": ["jdoe", "asmith"]
}
```

- `branchId` — Branch ID (int, required)
- `objectId` — Object ID to comment on (int, required)
- `message` — Comment text (string, required; plain text — the MCP renders it to HTML)
- `mentions` — (optional) Usernames to @-mention. The MCP prepends them to the message and renders them as clickable user refs.

## Differences from the Old MCP

- Old parameter `body` is now `message`. The old MCP required wrapping plain text in `<div>…</div>`; the new tool takes plain text and renders HTML for you.
- `mentions` replaces manual `<span>` mention elements.
- No separate follow-up fetch — the create call returns the full `CmsComment`.

## Response

Returns the full `CmsComment` including id, rendered HTML body, branch/object references, author, timestamps.

## Common Patterns

### API Response (comment_create)
Returns the full comment — no follow-up fetch required.
