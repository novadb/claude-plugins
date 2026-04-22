---
name: update-comment
description: "Update an existing comment's body text."
user-invocable: false
allowed-tools: comment_update
---

# Update Comment

## Scope

**This skill ONLY handles:** Updating the body text of an existing comment.

**For creating new comments** → use `create-comment`

Update an existing comment's body text.

## Tool

`comment_update` — Update the comment (returns the full updated comment)

## Parameters

```json
{
  "commentId": 98765,
  "message": "Updated comment text",
  "mentions": ["jdoe"]
}
```

- `commentId` — Comment ID (long, required)
- `message` — New comment text (string, required; plain text — the MCP renders HTML)
- `mentions` — (optional) Usernames to @-mention

## Differences from the Old MCP

- Old parameter `body` is now `message`. Plain text is accepted — no need to wrap in `<div>`.
- Update now returns the full `CmsComment` (previously 204 No Content).

## Response

Returns the updated `CmsComment` with rendered HTML body and all other comment fields.

## Common Patterns

### API Response (comment_update)
Returns the full updated comment — no follow-up fetch required.
