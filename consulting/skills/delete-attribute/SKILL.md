---
name: delete-attribute
description: "Delete an attribute definition from NovaDB."
user-invocable: false
allowed-tools: object_delete
---

# Delete Attribute

## Scope

**This skill ONLY handles:** Deleting an existing attribute definition.

**For finding the attribute to delete** → use `get-attribute` or `search-attributes`

Soft-delete an attribute definition.

> **Note:** NovaDB object IDs start at 2²¹ (2,097,152). All IDs in examples below are samples — always use real IDs from your system.

## Tool

`object_delete`

## Parameters

```json
{
  "branchId": 2100347,
  "objectIds": [12345],
  "comment": "No longer needed"
}
```

- `branchId` — Numeric branch ID (int).
- `objectIds` — Array of attribute IDs to delete (int[]).
- `comment` — (optional) Audit trail.

## Response

Returns `DeleteObjectResult` with `deletedObjects` count and `transaction`.

## Warning

Deleting an attribute does NOT automatically remove it from forms. After deletion, you may need to update affected forms using the `set-form-fields` skill to remove references to the deleted attribute.

## Common Patterns

### API Response (DELETE)
Returns `{ deletedObjects, transaction }` confirming the deletion.
