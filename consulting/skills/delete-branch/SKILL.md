---
name: delete-branch
description: "Permanently delete a branch (work package). This cannot be undone."
user-invocable: false
allowed-tools: work_package_delete
---

# Delete Branch

Permanently delete a branch (work package). This cannot be undone.

## Scope

**This skill ONLY handles:** Permanently deleting an existing branch (work package).

**For finding the branch to delete** → use `get-branch` or `list-branches`

## Tool

`work_package_delete`

## Parameters

```json
{
  "branchId": 2100500,
  "comment": "No longer needed"
}
```

- `branchId` — Branch ID (int, required)
- `comment` — (optional) Audit trail comment

## Response

Returns `DeleteWorkPackageResult` with `transaction` info.

## Warning

Always confirm with the user before deleting a branch. Branch deletion is permanent and cannot be reversed.

## Common Patterns

### API Response (DELETE)
Returns deletion result with transaction ID.
