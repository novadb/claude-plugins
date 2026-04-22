---
name: delete-comment
description: "Delete a comment by its ID."
user-invocable: false
allowed-tools: comment_delete
---

# Delete Comment

## Scope

**This skill ONLY handles:** Deleting an existing comment by its ID.

**For finding the comment to delete** → use `get-comment` or `get-comments`

Delete a comment by its ID.

## Tool

`comment_delete`

## Parameters

```json
{
  "commentId": 98765
}
```

- `commentId` — Comment ID (long, required)

## Response

Returns a short confirmation string (e.g. `"Deleted comment 98765"`).

## Warning

Always confirm with the user before deleting a comment. Comment deletion cannot be undone.

## Common Patterns

### API Response (DELETE)
Returns confirmation of deletion. Deletion cannot be undone.
