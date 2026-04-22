---
name: update-branch
description: "Update properties of an existing branch (work package)."
user-invocable: false
allowed-tools: work_package_update
---

# Update Branch

Update properties of an existing branch (work package).

## Scope

**This skill ONLY handles:** Updating properties (name, parent, type, state, due date, assignee, description) of an existing branch.

**For creating new branches** → use `create-branch`

## Tool

`work_package_update`

## Parameters

- `branchId` — Branch ID (int, required)
- `values` — JSON-encoded **string** of a CmsValue array for the fields to change
- `comment` — (optional) Audit trail comment

Read-then-merge semantics: only include fields you want to change — omitted fields remain untouched. Multi-value attributes, however, are **fully replaced** when specified.

## Branch Attribute IDs

| Attribute | ID | Type | Notes |
|-----------|------|------|-------|
| Name (EN) | 1000 | String | `language: 201`, `variant: 0` |
| Name (DE) | 1000 | String | `language: 202`, `variant: 0` |
| Parent branch | 4000 | ObjRef | `language: 0`, `variant: 0` |
| Branch type | 4001 | ObjRef | `language: 0`, `variant: 0` |
| Workflow state | 4002 | ObjRef | `language: 0`, `variant: 0` |
| Due date | 4003 | DateTime.Date | `language: 0`, `variant: 0`, ISO format |
| Assigned to | 4004 | String.UserName | `language: 0`, `variant: 0` |

## Example Call

```json
{
  "branchId": 2100500,
  "values": "[{\"attribute\":1000,\"language\":201,\"variant\":0,\"value\":\"Updated Name\"},{\"attribute\":4004,\"language\":0,\"variant\":0,\"value\":\"newuser\"}]",
  "comment": "Updated branch name and assignee"
}
```

Note: `values` is a JSON-encoded **string** — the new MCP expects it that way.

## Response

Returns `UpdateWorkPackageResult` with `createdValues` count and `transaction`.

## Common Patterns

### CmsValue Format
Every value entry in the JSON string follows: `{ attribute, language, variant, value }`
- `language`: 201=EN, 202=DE, 0=language-independent
- `variant`: 0=default
- Only include fields being changed; omitted fields remain unchanged.

### API Response (Update Branch)
Returns `{ createdValues, transaction }`.
